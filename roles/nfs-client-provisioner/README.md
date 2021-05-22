NFS Client Provisioner
==============================

> Installs [nfs-client-provisioner](https://artifacthub.io/packages/helm/ckotzbauer/nfs-client-provisioner) helm chart

The [nfs-client-provisioner](https://artifacthub.io/packages/helm/ckotzbauer/nfs-client-provisioner) is an automatic provisioner for Kubernetes that uses your already configured NFS server, automatically creating Persistent Volumes.

This chart installs a custom storage class into a K8S cluster using the Helm package manager. It also installs a NFS client provisioner into the cluster which dynamically creates persistent volumes from single NFS share.

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
         - name: Install nfs client provisioner
           role: nfs-client-provisioner

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
