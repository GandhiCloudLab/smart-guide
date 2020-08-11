# Mounting Volume in POD and Deployment

The objective of this documentation is to explore the facts behind Mounting volume in POD and Deployment.

Here are some facts.

- When the image of the POD is changed by editing `POD` object, the POD `will not` get recreated.
- When the image of the POD is changed by editing `Deployment` object, the POD `will` get recreated.

- Volume `can't` be added to the existing `POD` object by editing POD object. 

- Volume `can` be added to the existing `Deployment` object by editing Deployment object. But the POD would be recreated.


## Mounting Volume in POD

### Create a POD 

Create a pod with name = `my-pod` and image = `redis`

### Edit the POD Image 

Edit the image of the pod with image = `redis:6.0`

Observation : The POD is not recreated. The same POD exists only the container is recreated.

Explanation : POD might contain more than one containers and it is not required to recreate the POD when the image of one container is changed.


### Add Volume to the POD

Try to add Volume (emptyDir/secret/configMap) to this POD (via kubectl edit) 

Ex:
```
  volumes:
  - name: redis-storage
    emptyDir: {}
```

Observation : You can't do this. You will get the below error.

```
spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
```

Explanation : You can edit only the below attributes of the POD. Volume attribute can't be updated.

- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`

### Mount Volume to the POD

Try to Mount Volume (emptyDir/secret/configMap) to the POD (via kubectl edit) 

Ex:
```
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
```

Observation : You can't do this.

Explanation : You can edit only the below attributes of the POD. Volume attribute can't be updated.

- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`


## Mounting Volume in Deployment

### Create a Deployment 

Create a deployment with name = `my-deployment` and image = `redis`

### Edit the Image of the Deployment

Edit the image of the `Deployment` with image = `redis:6.0`

Observation : The POD is recreated.

Explanation : The changes to the Deployment recreates the POD.


### Add Volume to the Deployment

Try to add Volume (emptyDir/secret/configMap) to this Deployment (via kubectl edit) 

Ex:
```
  volumes:
  - name: redis2-storage
    emptyDir: {}
```

Observation : POD is recreated with Volume.

Explanation : The changes to the Deployment recreates the POD.

### Mount Volume to the Deployment

Try to Mount Volume (emptyDir/secret/configMap) to the Deployment (via kubectl edit) 

Ex:
```
    volumeMounts:
    - name: redis2-storage
      mountPath: /data/redis2
```

Observation : POD is recreated with Volume and Volume mounts.

Explanation : The changes to the Deployment recreates the POD.


## Appendix

### Sample POD with Volume

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: redis
    image: redis:6.0
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```


### Sample Deployment with Volume

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis2
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis2
  template:
    metadata:
      labels:
        app: redis2
    spec:
      containers:
        - image: redis
          name: redis
          volumeMounts:
            - name: redis2-storage
              mountPath: /data/redis2
      volumes:
        - name: redis2-storage
          emptyDir: {}
```