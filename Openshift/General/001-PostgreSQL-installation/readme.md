# Installing PostgreSql through Openshift 3.11 catalog

## Tags

Openshift, Kubernetes, Postgresql, Storage Class, PVC

## Problem 

When installing PostgreSql through Openshift 3.11 Catalog the storage was in pending state. 

## Root cause

No default Storage class exists and not able to bind any storage class.

## Solution

1. Find the storage class available using the below.

```
$ oc get sc

```

2. create a new PVC using the available storage class.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    template: postgresql-persistent-template
  name: postgresql
  namespace: aaaaaa
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: nfs-storage-1
```

3. Delete the installed Postgresql and create new one from catalog. 

4. The created PVC get bounded and postgresql start working.


## Note

1. It is good to have default storage class created for the cluster. Also It can be patched via the below command.
```
kubectl patch storageclass <your-class-name> -p ‘{“metadata”: {“annotations”:{“storageclass.kubernetes.io/is-default-class”:“true”}}}’
```
