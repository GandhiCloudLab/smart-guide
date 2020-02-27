# External IP in Service

External-IP in the Openshift 3.11 can be assigned to the service.

## Tags

Openshift, Kubernetes, External IP

## Assigning External IP to a service

Get the service listed.
```
$ oc get svc

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
g-app-store-service   ClusterIP   172.30.135.105   <none>        8080/TCP   9d

```
The external ip is empty for now.

Here the external IP can be patched using the below command. 
```
oc patch svc g-app-store-service -p '{"spec":{"externalIPs":["123.333.323.339"]}}'
```

Then the application can be referenced using the external ip and port.

## Default IP Range for the external IPs

During the installation of the Openshift 3.11 and below (???), the default IP Range for the external IPs can be set. If it was not set rightly, it may assign some unused internal ips to external ips. 

In this case though the external ip assigned the service cannot be accessed via ExternalIP.

## Reference

https://docs.openshift.com/container-platform/3.11/dev_guide/expose_service/expose_internal_ip_service.html#manually-assign-ip-service

