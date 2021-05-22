# Ansible NFS Persistent Volume Role

> Creates a persistent volume and claim to a NFS export 

Creates a new Persistent Volume (PV) and Persistent Volume Claim (PVC) in a Kubernetes cluster to expose a NFS export as a volume in Kubernetes Pods. If the PV exists, it will be updated to match the configuration of the playbook calling the role. If the PVC exists, it will be updated to mach the playbook.

***If the PV and PVC are already in use and attached to a running pod, this role will fail. All claims need to be detached prior to updating the PV and PVC.***


Requirements
------------

* Kubernetes cluster
* Kubectl on the inventory host
* NFS server with exports to the Kubernest nodes

> See the [nfs-server](../nfs-server/README.md) and [nfs-client](../nfs-client/README.md) roles used by the [playbooks/storage/nfs.yml](../../playbooks/storage/nfs.yml) playbook for examples configuring a NFS server for the cluster.

Role Variables
--------------

| Name | Description | Default |
| ----- | --------------| --------|
| `nfs_persistent_volume_namespace` | namespace to which the pvc will be added. | default |
| `nfs_persistent_volume_mount_path` | export on the nfs server to be mounted. | /datasets/evaluations |
| `nfs_persistent_volume_nfs_server` | host or ip address to nfs server. | |
| `nfs_persistent_volume_name` | name of the pv to be created/updated. | cluster-nfs |  

Dependencies
------------

The following Ansible collections are used:

* community.kubernetes module  

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: kube-master[0]
      roles:
         - role: nfs-persisten-volume
         	 nfs_persistent_volume_namespace: jdoe

License
-------

BSD 3-Clause