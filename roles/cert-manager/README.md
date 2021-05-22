# Kubernetes Cert-Manager 
> Deploy Cert-Manager to Kubernetes via Helm
 
This folder will deploy a cert-manager to your kubernetes cluster via Helm. A self-signed issuer will be created as the default ClusterIssuer.

## Requirements 

The following need to be present for this role:

* Kubernetes should be installed and running. 

## Role Variables

See [defaults/main.yml]() for a documented list of all variables.

## Dependencies

* Kubernetes cluster 1.16+
* Helm 3

## Example Playbook

A sample playbook running this roll with default config follows:

    - hosts: kube-master[0]
      roles:
         - role: cert-manager
           vars:
             cert_manager_namespace: cert-manager
             cert_manager_version: v1.1.0


## License

BSD 3-Clause
