# Sample Deployment files for Route

Sample Deployment files to deploy application with route in Openshift.

## Tags

Openshift, Kubernetes, Route, Router

## Steps

1. Download the yaml files available in the `src` folder

2. Run the below oc apply command to deploy the app in OCP.

```
oc apply -f 01-namespace.yaml
oc apply -f 02-deployment.yaml
oc apply -f 03-service.yaml
oc apply -f 04-route.yaml
```

3. Access the deployed application using the below url.

```
http://g-app-store-route-g-app-store-pro.apps.<IP_ADDRESS>.com
```

4. You can also change the image name in `02-deployment.yaml` file with your application image and test.