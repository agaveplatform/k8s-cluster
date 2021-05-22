# Kubernetes Cert-Manager Issuers
> Creates default ClusterIssuers for Cert-Manager 
 
This folder will deploy three issuers to an existing Kubernetes cluster with Cert-Manager installed:
1. **acme-prod**: generateds valid top level ssl certs from Let's Encrypt using HTTP01 protocol. 
2. **acme-staging**: generates certs from Let's Encrypt staging server. These are effectively self-signed.
3. **ca-issuer**: Uses the kubernetes cluster ca to sign certs.

## Requirements 

The following need to be present for this role:

* Kubernetes should be installed and running. 
* Cert-manager already installed

## Role Variables

See [defaults/main.yml]() for a documented list of all variables.

## Dependencies

* Kubernetes cluster 1.16+

## Example Playbook

A sample playbook running this roll with default config follows:

    - hosts: kube-master[0]
      roles:
         - { role: cert-manager-issuers }

## License

BSD 3-Clause
