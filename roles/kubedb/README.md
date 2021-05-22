KubeDB Ansible Role
=========

> Deploys the KubeDB operator to a kubernetes cluster.

This role deploys a KubeDB operator to Kubernetes via Helm. All database plugins are currently supported. 

Requirements
------------

* A valid kubernetes cluster.
* Helm installed in the cluster.

Role Variables
--------------

See [defaults/main.yml]() for a documented list of all variables. 


Example Playbook
----------------

    - hosts: kube-master[0]
      become: yes
			
			roles:
         - role: kubedb

License
-------

BSD 3-Clause

KubeDB is an open source project.
