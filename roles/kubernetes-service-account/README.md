Kubernetes Service Account Role
=============================
> Bootstraps a namespace with service account, psp, quotas, and rbac   

Creates a service account in a given namespace of the same name. If the namespace does not exist, 
it is created. Quota, ResourceLimit, and PSP are set for the namespace and service account. A valid 
kubeconfig file will be dumped to stdout for the service account.

Requirements
------------
* An existing Kubernetes deployment > 1.18.0.
* PSP enabled in the cluster

Role Variables
--------------

See [defaults/main.yml]() for a documented list of all variables.

Dependencies
------------

- community.kubernetes.k8s

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - role: kubernetes-service-account
           vars:
             kubernetes_service_account_namespace: staging 

License
-------

BSD 3-Clause

