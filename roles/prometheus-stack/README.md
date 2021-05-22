Kubernetes Prometheus Stack Role 
==============================

> Install Prometheus + Grafana in a Kubernetes cluster

Installs via the official Prometheus Stack chart. Default auth is enabled

Requirements
------------

Kubernetes should be installed and running. You will need an admin service account to run this.

Role Variables
--------------

Edit `templates/custom-values.yml` as needed.

Dependencies
------------

* Helm 3
* Kubernetes cluster 1.18+

Example Playbook
----------------

A sample playbook running this roll with default config follows:

    - hosts: kube-master[0]
      roles:
         - { role: prometheus-stack }

Viewing Apps
------------

```
kubectl --namespace monitoring port-forward svc/prometheus-grafana 3000:80
```

View the Grafana dashboard at [http://localhost:3000/]()

```
kubectl --namespace monitoring port-forward svc/prometheus-operated 9090
```

View the Prometheus dashboard at [http://localhost:9090/]()


License
-------

BSD 3-Clause
