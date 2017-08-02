Glusterfs-azure

Using this [installation guide for Centos](https://wiki.centos.org/HowTos/GlusterFSonCentOS) and this [installation guide for RHEL](https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.2/html/installation_guide/installing_red_hat_storage_server_on_red_hat_enterprise_linux_layered_install) as reference

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fritazh%2Fglusterfs-azure%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

# Usage


# How it works


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
