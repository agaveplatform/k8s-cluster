Kubernetes Cluster Default Admin RBAC
=============================

Grants a service account cluster admin role. 

> This role should only be used after completing a Kubernetes install.

Requirements
------------

* An existing Kubernetes deployment > 1.18.10.  
 

Role Variables
--------------

`kubernetes_cluster_admin_service_accounts`: list of service account names to which cluster admin status will be granted

Dependencies
------------

- collections.helm
- collections.kubernetes

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - role: kubernetes-cluster-admin
           vars:
             kubernetes_cluster_admin_service_accounts: [ "default" ]

License
-------

BSD 3-Clause

Author Information
------------------

Rion Dooley <deardooley@gmail.com>