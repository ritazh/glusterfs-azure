## GlusterFS

[GlusterFS](http://www.gluster.org) is an open source scale-out filesystem. This solution references [installation guide for Centos](https://wiki.centos.org/HowTos/GlusterFSonCentOS) and [installation guide for RHEL](https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.2/html/installation_guide/installing_red_hat_storage_server_on_red_hat_enterprise_linux_layered_install) to provision a GlusterFS server cluster on Azure. It also provides examples to how to mount GlusterFS volumes locally and how to use GlusterFS volumes from Kubernetes containers.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fritazh%2Fglusterfs-azure%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

# Usage

Deploy the Gluster cluster on Azure using the `Deploy button` ☝️ or using the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

From Azure CLI, populate `azuredeploy.parameters.json` with the following parameters before starting the deployment.

```
az group deployment create --resource-group <RESOURCEGROUP> --template-file azuredeploy.json --parameters azuredeploy.parameters.json

```
Below is the list of template parameters:

| Name   | Required | Description | Default |
|:--- |:--- |:---|:---|
| adminUsername | :heavy_check_mark: | SSH user name | |
| adminPassword | :heavy_check_mark: | SSH user password | |
| hostOs | :heavy_check_mark: | OS to install on the host system. Allowed values: `Centos`, `Ubuntu`, and `RHEL` | `Centos` |
| nodeNumber | :heavy_check_mark: | Number of nodes in the gluster file system. Allowed values: `2`, `4`, `6`, and `8` | `2` |
| diskNumber | :heavy_check_mark: | Number of disks per node. Allowed values: `2`, `3` and `4` | `2` |
| rhsmUsername | | [RHEL only] Name of the virtual network to be used | |
| rhsmPassword | | [RHEL only] Name of the virtual network to be used | |
| rhsmPoolId | | [RHEL only] Name of the virtual network to be used | |
| virtualNetworkName | :heavy_check_mark: | Name of the virtual network to be used | |
| virtualNetworkNewOrExisting | :heavy_check_mark: | Create a new or use existing Virtual Network. Allowed values: `new` and `existing` | `new` |
| vmNamePrefix | :heavy_check_mark: | VM name prefix, a number will be appended for each node | |
| volumeName | :heavy_check_mark: | Gluster file system volume name | `gfsvol` |


# How it works

## Verify Gluster Volumes
To verify the new gluster volume is created and running, run the following on one of the gluster nodes:

```
# gluster volume info
 
Volume Name: gfsvol
Type: Replicate
Volume ID: 2bff68ee-xxxx-xxxx-xxxx-bbd6b3fcfa51
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: <NODE1>:/datadrive/brick
Brick2: <NODE2>:/datadrive/brick
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
```

## Mount Gluster Volumes

To mount the new gluster volume to a new host, install the GlusterFS native client from [here](http://gluster.readthedocs.io/en/latest/Administrator%20Guide/Setting%20Up%20Clients/). 

Use the private ip or node name of one of the gluster hosts to mount to a local mount directory
```
# mount -t glusterfs HOSTNAME-OR-IPADDRESS:/VOLNAME MOUNTDIR

# mount -t glusterfs 10.0.0.10:/gfsvol /mnttest/

```
## Verify Mounted Volume

To verify the size of the new gluster volume, run `df -h` you should see the local mount directory with the same size as the total size of all the managed disks you have created. 

```
# df -h
...
10.0.0.10:/gfsvol  2.0T   81M  1.9T   1% /mnttest
..
```

# Mount Gluster Volumes from Kubernetes Pod

If you want access gluster volumes from Kubernetes pods, one way is to provision the gluster cluster in the same vnet as your Kubernetes cluster.

## Create Gluster Cluster using existing vnet

Assuming you already have a K8s cluster running, update the `azuredeploy.parameters.json` file to provision a gluster cluster in the same vnet and same resource group as your k8s cluster.

```json

	"virtualNetworkName": {
	  "value": "k8s-vnet-xxxxxxxx"
	},
	"virtualNetworkNewOrExisting": {
	  "value": "existing"
	},

```
From Azure CLI,

```
az group deployment create --resource-group <KUBERNETES-CLUSTER-RESOURCEGROUP> --template-file azuredeploy.json --parameters azuredeploy.parameters.json

```

Now that you have a gluster cluster and a kubernetes cluster running in the same vnet, use this [documentation](https://github.com/kubernetes/examples/blob/master/staging/volumes/glusterfs/README.md) to allow containers to use GlusterFS volumes.

 ```
$ kubectl create -f _output/glusterfs-endpoints.json 
endpoints "glusterfs-cluster" created
$ kubectl create -f _output/glusterfs-service.json 
service "glusterfs-cluster" created
$ kubectl create -f _output/glusterfs-pod.json
```

Once the pod is running, let's verify the gluster volume has been mounted:

```
$ kubectl exec -it glusterfs -- /bin/bash
root@glusterfs:/# df -h         
...
10.0.0.10:gfsvol  2.0T   81M  1.9T   1% /mnt/gluster

```
