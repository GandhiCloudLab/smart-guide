# Prerequisite  for Transformation Advisor

Transformation advisor is installed along with the Cloud Pak for Applications. Before installing the same make sure that the prerequisite for Transformation Advisor is available.

The detailed documentation is available in IBM Knowledge Center https://www.ibm.com/support/knowledgecenter/SSCSJL_4.x/install-prerequisites-ta.html


## Step 1. Verify PV exists

### Find PV

1. Login to OCP cluster

Login to your OCP cluster using the below command.

```
oc login https://<your_cluster_hostname> -u <username> -p <password>
```

2. Run the below command to see whether any PV exists.

```
oc get pv
```

3. Describe the listed PVs and see if it is for TA.

```
oc describe pv <<pv_name>>
```

4. If a PV exists with the below condition then you can continue further. If it doesn't then skip to the next step (Step 2. Verify and Install NFS Server). 

```
Status : UnBound
Capacity:   > 8Gi
Access Modes:    RWX
```
### Update NFS folder permission

5. Find the value of the nfs.path attribute. It could be like this.

```
path: /nfsshare/ta
```

6. Login to the infra node of your cluster with ssh.

```
ssh -v root@aaa.bbb.ccc.ddd
```

7. Run the below command to upgrade the rights to the nfs folder.

```
chmod -R 777 /nfsshare/ta
```
8. Exit from the infra node by executing the below command

```
exit
```

*....Prerequisite done and you can go back and install the cloud pak for application.....*

## Step 2. Verify and Install NFS Server

*You would need NFS server for Transformation Advisor (ta). If you have existing NFS server, you can skip this step and goto next step.*

If you don't have existing nfs server, you can setup one quickly with following instructions.

1. Run the below commands.

```
yum install -y nfs-utils
mkdir /nfsshare
chmod -R 755 /nfsshare
chown nfsnobody:nfsnobody /nfsshare
systemctl enable rpcbind
systemctl enable nfs-server
systemctl start rpcbind
systemctl start nfs-server
```

2. Export the NFS share folder by doing below steps.

    2.1) Open vi editor

    ```
    vi /etc/exports
    ```

    2.2) Add the below line, save and exit the vi editor.

    ```
    /nfsshare *(rw,sync,no_subtree_check,no_root_squash,no_all_squash)
    ```

3. Restart nfs-Server by running the below commands.

```
systemctl restart rpcbind
systemctl restart nfs-server
```

## Step 3. Verify Share folder exists for Transformation Advisor

Lets assume your NFS share is `/nfsshare`.

Check whether you have any share(folder) avaialble for TA like `/nfsshare/ta`, `/nfsshare/ta_share` or etc.

If exists, note the path of share folder name. (Lets assume it is `/nfsshare/ta`)

If it doesn't exists then do the following to create a folder.
```
mkdir /nfsshare/ta
chmod 755 /nfsshare/ta
```

## Step 4. Find the NFS Server IP

Note down the IP address of the NFS Server. (Lets assume it is `111.222.333.444`)

## Step 5. Create PV

We need to create a PV in your cluster by doing below.

Given below the PV Content.

```
apiVersion: v1
kind: PersistentVolume
metadata:
 name: tanfspv
 labels:
   pvc_for_app: "ta"
spec:
 capacity:
   storage: 8Gi
 accessModes:
 - ReadWriteMany
 nfs:
   path: <<NFS_SERVER_TA_PATH>>
   server: <<NFS_SERVER_IP>>
 persistentVolumeReclaimPolicy: Recycle
 ```

1. Replace the server ip <<NFS_SERVER_IP>> with the NFS server ip, we noted above. (Our Assumption value `111.222.333.444`)

2. Replace the <<NFS_SERVER_TA_PATH>> with the NFS share folder name. (Our Assumption value `/nfsshare/ta`)

3. Copy the PV content into a file called pv.yaml

4. Run the below command to create the pv in cluster.
```
oc create -f pv.yaml
```

*....Prerequisite done and you can go back and install the cloud pak for application.....*
