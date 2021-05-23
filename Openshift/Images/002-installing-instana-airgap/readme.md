# 002-installing-instana-airgap

This article explains about how to install Instana through Operator in Openshift Cluster locaed in DMZ.

In DMZ, the images refered in the instana operators can't be downloaded from docker.io. This article explains about how to use private registry to solve this probelm.


There are 2 images used in Instana.

- Instana Operator image 
- Instana Agent image 


### 1. Assumption

Let us assume that there is a private registry created and added to the Openshift Cluster in the DMZ network.

Lets consider `myregistry.mydomain.io` is private registry name.


### 2. Download Instana Agent Operator files

Run the below command

```
git clone https://github.com/instana/instana-agent-operator.git
```

### 3. Prefix Private registry in Instana Agent image 

1. From the cloned directory, open the file `deploy/instana-agent.customresource.yaml`



The image name of the instana agent is `instana/agent:latest`. If we use this as it is, it will pull the image from `docker.io`, so we need to prefix that with the private registry to pull it from private registry.

To change the docker image name in the instana operator, we need to add `agent.image` in the CRD.

Reference : https://www.instana.com/docs/setup_and_manage/host_agent/on/kubernetes/#operator-configuration


Add the below text under the `spec:` in the file `deploy/instana-agent.customresource.yaml`.

Note: In earlier steps, we have taken `myregistry.mydomain.io` as a private registry.

```
agent.image: myregistry.mydomain.io/instana/agent:latest
```

The resulted file could look like this.

```
  ....

  spec:
    agent.image: myregistry.mydomain.io/instana/agent:latest
    agent.zone.name: my-zone # (optional) name of the zone of the host
    agent.key: replace-me # replace with your Instana agent key
    agent.endpoint.host: ingress-red-saas.instana.io # the monitoring ingress endpoint

  ....

```

### 4. Prefix Private registry in Instana Operator image

1. Open the file `https://github.com/ppc64le/build-scripts/blob/master/i/instana-agent-operator/instana-agent-operator_rhel8.2.sh`

2. Comment the below line at line number 67, as we have already downloaded.
```
git clone https://github.com/instana/instana-agent-operator.git
```

3. Need to prefix the private registry for the instana operator too.

Replace the below line at line number 77
```
docker build -f src/main/docker/Dockerfile.jvm -t instana/instana-agent-operator .
```
by
```
docker build -f src/main/docker/Dockerfile.jvm -t myregistry.mydomain.io/instana/instana-agent-operator .
```

### 5. Download Instana Agent image and retag

Need to download the agent image from docker.io and retag it.

Run the below command from bastion server.

```
docker pull instana/agent:latest
docker tag instana/agent:latest myregistry.mydomain.io/instana/agent:latest
```

Run the below command to verify the tag is ok.

```
docker images | grep instana
```

The output could be like this.
```
Jeyas-MacBook-Pro:test jeyagandhi$ docker images | grep instana
instana/agent                                latest    f43f02526f06   2 weeks ago    808MB
myregistry.mydomain.io/instana/agent         latest    f43f02526f06   2 weeks ago    808MB
```

### 6. Build Instana Operator image

1. Build the Instana Operator image as usual by executing the `instana-agent-operator_rhel8.2.sh` file.

This could have creted an instana operator image `myregistry.mydomain.io/instana/instana-agent-operator`.

2. Run the below command to verify.

```
docker images | grep instana
```

The output could be like this.
```
Jeyas-MacBook-Pro:test jeyagandhi$ docker images | grep instana
myregistry.mydomain.io/instana/instana-agent-operator   latest    771330101ac0   2 weeks ago    403MB
instana/agent                                   latest    f43f02526f06   2 weeks ago    808MB
myregistry.mydomain.io/instana/agent            latest    f43f02526f06   2 weeks ago    808MB
```

### 7.Push both the images to private registry

Need to push the images to private registry.

Run the below command from bastion server.

```
docker push myregistry.mydomain.io/instana/agent:latest
docker push myregistry.mydomain.io/instana/instana-agent-operator:latest
```

### 8. Continue the regular install process

Now run the CatalogSource for Instana to add instana into operator hub and follow regular process. The instana would be installed from myregistry.mydomain.io

