# Cloud Pak for Applications v4.0.1 Installation

Given below the steps to install the Cloud Pak for applications v4.0.1 on top of RHOCP 4.2/4.3 cluster.

Most of the steps mentioned below to be done from your local system. Switch to infra node, when it is explicitly mentioned.

## References

The detailed steps about this installation is available in the knowledge center at the location https://www.ibm.com/support/knowledgecenter/SSCSJL_4.x/install-icpa-cli.html

Another version of the installation steps are available in https://github.ibm.com/IBMCloudPak4Apps/icpa-install

## Prerequisites

RHOCP 4.2/4.3 cluster up and running. 

## Steps to install

### 1. Prerequisite for Transformation Advisor

The pre-requisite for the Transformation Advisor is available in the below link. Make sure it is done.

[Transformation Advisor](TA-Prerequisite-install.md)

### 2. Get the entitlement key

*Note: there is change in the way we get the entitlement key for IBMers and Partners*

#### IBMers

IBMers can get the entitlement key of the cloud pak for application from the below URL for the internal consumption.

https://github.ibm.com/CloudpakOpenContent/cloudpak-entitlement

#### Partners

Partners after you order IBM Cloud Pak for Applications, an entitlement key for the cloud pak software is associated with your MyIBM account.

Get the entitlement key that is assigned to your ID

1. Log in to [MyIBM Container Software Library](https://myibm.ibm.com/products-services/containerlibrary)  External link icon with the IBMid and password that are associated with the entitled software.

2. In the Entitlement keys section, select Copy key to copy the entitlement key to the clipboard.

### 3. Extract the installation configuration from the installer image

1. Replace the <<entitlement_key>> value and run the below export commands to set the entitled registry information.

*Note: there is change in the registry user for IBMers and Partners*

#### IBMers
```
export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=ekey
export ENTITLED_REGISTRY_KEY=<<entitlement_key>>
```

#### Partners
```
export ENTITLED_REGISTRY=cp.icr.io
export ENTITLED_REGISTRY_USER=cp
export ENTITLED_REGISTRY_KEY=<<entitlement_key>>
```

2. Docker login 

Do the docker login to access the entitled registry by running the below command

```
docker login "$ENTITLED_REGISTRY" -u "$ENTITLED_REGISTRY_USER" -p "$ENTITLED_REGISTRY_KEY"

```

3. Create data directory

Create a "data" directory by running the below command

```
mkdir data
```

4. Extract the installation configuration

Extract the installation configuration files by running the below command 

```
docker run -v $PWD/data:/data:z -u 0 \
           -e LICENSE=accept \
           "$ENTITLED_REGISTRY/cp/icpa/icpa-installer:4.0.1" cp -r data/* /data
```

### 4. Login to OCP cluster

Login to your OCP cluster using the below command. You need to substitute the value for <your_cluster_hostname>.

```
oc login https://<your_cluster_hostname> -u <username> -p <password>
```

### 5. Run the installer

Run the installer using the below command. 

```
docker run -v ~/.kube:/root/.kube:z -u 0 -t \
           -v $PWD/data:/installer/data:z \
           -e LICENSE=accept \
           -e ENTITLED_REGISTRY -e ENTITLED_REGISTRY_USER -e ENTITLED_REGISTRY_KEY \
           "$ENTITLED_REGISTRY/cp/icpa/icpa-installer:4.0.1" install
```

It may take more than 20 minutes time to complete the installation. At the end of the installation you will get the necessary URLs to access Kabanero, TA and Tekton.

## Troubleshooting

### 1. TA installation fails with TA issues.

if you have forgotten to do the Prerequisite for Transformation advisor and installed the cloud pak for application and it is showing the error in TA. You can do the followings.

1. Do the Prerequisite for Transformation advisor steps given in [Transformation Advisor](TA-Prerequisite-install.md)

2. Continue the step `*5. Run the installer*`

### 2. Installation fails after Retrying... 

The installation may fail when any of the step taking long time to complete. You may get error like this.
```
  Retrying... (39 of 41)
  Retrying... (40 of 41)
failed

msg: 
non-zero return code
```

Mostly this is due to timeout issue. To resolve this, wait for few minutes and run the step  `*5. Run the installer*` again.

### 3. Un-Installing Cloud pak for App

To uninstall the cloud pak for app run the `docker run -v ..."`cmd with uninstall option.
```
docker run -v ~/.kube:/root/.kube:z -u 0 -t \
           -v $PWD/data:/installer/data:z \
           -e LICENSE=accept \
           -e ENTITLED_REGISTRY -e ENTITLED_REGISTRY_USER -e ENTITLED_REGISTRY_KEY \
           "$ENTITLED_REGISTRY/cp/icpa/icpa-installer:4.0.1" uninstall
```

### 4. Got the error `"collections.kabanero.io" is invalid: spec.versions: Invalid value:`

When CP4A is installed using the image `cp.icr.io/cp/icpa/icpa-installer:4.0.0` got the below error.

```
    CustomResourceDefinition.apiextensions.k8s.io "collections.kabanero.io"
    is invalid: spec.versions: Invalid value:
    []apiextensions.CustomResourceDefinitionVersion{apiextensions.CustomResourceDefinitionVersion{Name:"v1alpha1",
    Served:true, Storage:true,
    Schema:(*apiextensions.CustomResourceValidation)(0xc0312d48b0),
    Subresources:(*apiextensions.CustomResourceSubresources)(nil),
    AdditionalPrinterColumns:[]apiextensions.CustomResourceColumnDefinition(nil)}}:
    per-version schemas may not all be set to identical values (top-level
    validation should be used instead)
```

This issue is due to mismatch in CRD versions between cp4a 4.0.0 and cp4a 4.0.1. 

So uninstalled cp4a 4.0.0 and installed with `cp.icr.io/cp/icpa/icpa-installer:4.0.1` image.