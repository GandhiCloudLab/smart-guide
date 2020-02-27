# Sample Deployment files Route

Sample Deployment files to deploy application with route in Openshift.

## Tags

Primary : 

Openshift, Kubernetes, Route, Router

Secondary : 

## Steps

1. Download the yaml files available in the `src` folder

2. Run the follwoing oc apply command to deploy the app in openshift.

```
oc apply -f 01-namespace.yaml
oc apply -f 02-deployment.yaml
oc apply -f 03-service.yaml
oc apply -f 04-route.yaml
```

3. Now you can access the applicaiton using the below url in your browser.

```
http://g-app-store-route-g-app-store-pro.apps.
```
Add the below argument sections 
```
    args:
      - '--loglevel=4'
```

under the image section.
```
    image: openshift3/ose-haproxy-router:v3.11
    imagePullPolicy: IfNotPresent
```

Also increase the livenessProbe and readinessProbe with larget timeouts
```
        livenessProbe:
          failureThreshold: 10
          httpGet:
            host: localhost
            path: /healthz
            port: 1936
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30

        readinessProbe:
          failureThreshold: 10
          httpGet:
            host: localhost
            path: healthz/ready
            port: 1936
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
```

#### Another way to enable Debug log 

There is an another way to increase the debug log like the below.

```
$ oc set env dc/router ROUTER_SYSLOG_ADDRESS=<dest_ip:dest_port>  ROUTER_LOG_LEVEL=<level>
```
You can refer the same in https://docs.openshift.com/container-platform/3.11/admin_guide/router.html

#### View Logs in router pod 

Run the below command to know the router pod.
```
$ oc get pods -n default | grep route

router-5-8lknf                                                1/1     Running     0          8h
```

Run the below command to tail the logs
```
$ oc get pods -n default | grep route
```

## routes.json 

The route related informations are found in the router at `/var/lib/haproxy/router/routes.json`.

If you have any issues in your route and it is not working you can have a look at this json to understand if any issue here.

Connect to the interactive shell of the router pod.
```
$ oc exec -it router-5-8lknf -n default sh
```

Open and see the `router.json` file and obeserve for any error. 
```
    "ohc:ohc-patient-viewer": {
      "Name": "my-app-store",
      "Namespace": "aaa",
      "Host": "my-app-store-aaa.apps.23.2.1.4.xip.io",
      "Path": "",
      "TLSTermination": "",
      "Certificates": null,
      "VerifyServiceHostname": false,
      "Status": "",
      "PreferPort": "myhttp",
      "InsecureEdgeTerminationPolicy": "",
      "RoutingKeyName": "bbad49c76bc973a079eef8acbb34a45d",
      "IsWildcard": false,
      "Annotations": {
        "openshift.io/host.generated": "true"
      },
      "ServiceUnits": {
        "aaa/my-app-store": 100
      },
      "ServiceUnitNames": null,
      "ActiveServiceUnits": 1,
      "ActiveEndpoints": 0
    },
```

The above piece of the file shows the router details of an router. The contents `"ServiceUnitNames": null` and `"ActiveEndpoints": 0` indicates that there is some issue with your service to expose through the HA Proxy / Router. 

You may need to check the service or image of that service pod.


## Invalid Host Header Error

When accessing a service through the route there was an error called "Invalid Host Header". 

1. Checked whether the other services are working with route to make sure the Router and OCP configurations are OK. The other services are working fine.

2. Able to access the service by doing the service port forwarding. http://localhost:8082
```
kubectl port-forward svc/my-app-stor 8082:8080
```

3. Able to access the pod by doing the service port forwarding. http://localhost:8084
```
kubectl port-forward pod/my-app-store-asdfsad-sdf-df 8084:8080
```

4. The routes.json was showing `"ActiveEndpoints": 0`, so the service is not accessible behind HA proxy.

5. Suspected that there could be probelm with the image used in the service. Replaced the image used in the Deployment.yaml file with another image and redeployed in OCP. It works. So the issue could be related to image only.

6. Got the access to see the Dockerfile of the image. It is Vue Js application. The Dockerfile shows that Vue Js applicaiton is directly converted as a Docker image. It is the root cause of the probelm.

7. Bundling the Vue Js application with out wrapping with some webserver like NGINX will be working as a service in Kubernetes. It can be accessed via NodePort. 

8. To access any application via HA Proxy, it requires the webserver. So wrap the VueJs application with NGINX as mentioned below, the application started working.

```
# Dockerfile
FROM nginx:1.17
COPY ./nginx.conf /etc/nginx/nginx.conf
WORKDIR /code
COPY ./dist .
EXPOSE 8080:8080
CMD ["nginx", "-g", "daemon off;"]
```

Reference : https://blog.openshift.com/deploy-vuejs-applications-on-openshift/

9. Even after wrapping with the NGINX there could be a "Invalid Host header" issue. As mentioned in the below thread, need to configure `"disableHostCheck: true"` if required.

https://github.com/vuejs-templates/webpack/issues/1205

