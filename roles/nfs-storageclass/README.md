Local Path Provisioner StorageClass
==============================

> Creates storage classes that utilize the Rancher Local Path Provisioner

Simple utility role to create storage classes that use a preinstalled Rancher Local Path Provisioner.

Requirements
------------

Kubernetes should be installed and running. Rancher Local Path Provisioner should be installed and running.

Role Variables
--------------

See the [default variable file](defaults/main.yml) for descriptions of all variables. 


Dependencies
------------

* Kubernetes cluster >1.18.0.

Example Playbook
----------------

A sample playbook running this roll with default config follows:

    - hosts: kube-master[0]
      roles:
         - name: Install local path provisioner
           role: local-storageclass
           vars:
             local_storageclass_name: proteus-fast-work

License
-------

BSD 3-Clause
