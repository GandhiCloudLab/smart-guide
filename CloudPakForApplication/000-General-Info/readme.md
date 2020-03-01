# Cloud Pak for Applications General Information


## 1. Version related info

[Ann Robinson]
Cloud Pak for Applications v4.0.1 was released Friday, February 14th 2020.

This fixpack adds support for the recent release of Red Hat OpenShift 4.3 on Intel, so Cloud Pak for Applications v4.0.1 installs and runs on both OpenShift 4.2 and OpenShift 4.3 (Intel).

[Alan Little]
cp4apps is not supported on OCP 4.3,  CP4apps 4.0.1 was a fixpack that enabled OCP 4.3.  So that is expected. The next version of CP4Apps will support OCP 4.3 as well.

## 2. Disable Transformation Advisor 
To select to not install Transformation Advisor when installing CP4Apps v4, in the config.yaml configuation file under the Transformation Advisor settings modify the config setting to be

```
 config: ""
```
as noted in the knowledge center - https://www.ibm.com/support/knowledgecenter/SSCSJL_4.x/install-conf-files.html#config

## 3. Air gap installation
[26-Feb-2020]

Has anyone performed air gap installation of cp4a? Provided that you have a VM to pass access to internet through a proxy.

[Alan Little]
Airgap is not yet supported