Rancher Local Path Provisioner
==============================

> Install Rancher Local Path Provisioner

Local Path Provisioner provides a way for the Kubernetes users to utilize the local storage in each node. Based on the user configuration, the Local Path Provisioner will create hostPath based persistent volume on the node automatically. It utilizes the features introduced by Kubernetes Local Persistent Volume feature, but make it a simpler solution than the built-in local volume feature in Kubernetes.

_***Note:*** PersistentVolumeClaims accessModes are restricted to_ `ReadWriteOnce`

Requirements
------------

Kubernetes should be installed and running. 

Role Variables
--------------

Please see the [default variable file](defaults/main.yml) for descriptions of all variables. 


Dependencies
------------

* Kubernetes cluster 1.16+

Example Playbook
----------------

A sample playbook running this roll with default config follows:

    - hosts: kube-master[0]
      roles:
         - name: Install local path provisioner
           role: local-path-provisioner

Example Local volume 
------------------------

``` 
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-hdd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-path-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-hdd
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: local-path-pvc
```

License
-------

BSD 3-Clause
