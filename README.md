# Agave Infrastructre-as-Code Kubernetes Cluster Repo  

> Provisions production ready [Kubernetes cluster](https://kubernetes.io) for the [Agave Platform](https://agaveplatform.org/) on the [IU Jetstream Cloud](https://jetstream-cloud.org)

This is the [Agave Platform's](https://agaveplatform.org/) infrastructure-as-code (IaC) repo for [Kubernetes](https://kubernetes.io) and all support services needed to operate the platform. It focuses on spinning up a cluster per project and relies upon [Helm 3](https://helm.sh) to manage individual cluster services. 

___Deployment of the Agave Platform itself is handled independently of this repo via [Agave's Helm charts](https://github.com/agavaeplatform/helm-charts).___  

## Prerequisites

* Mac or Linux deployment host. (In theory, it is possible to run via Docker on Windows, but this has not been tested.)
* Python 3.6+ and virtualenv
* Git
* [Terraform](https://terraform.io)
* Allocation on the Indiana University region of the [Jetstream Cloud](https://jetstream-cloud.org).


## Dependencies

* [Andrea Zonca's fork](https://github.com/zonca/jetstream_kubespray) fork of the official [Kubespray](https://github.com/kubernetes-sigs/kubespray) project is included as a Git submodule of this repo.


### Preparation

Check out repo, create a virtual environment, install Python and Ansible Galaxy requirements 

```shell
# check out the repo
git clone git@github.com:agaveplatform/k8s-cluster.git
cd k8s-cluster

# create a virutal environment
python3 -mvenv .venv
source .venv/bin/activate

# Install Python requirements
pip install -r requirements.txt

# Install Ansible Galaxy requirements
ansible-galaxy install -r galaxy.yml
```

## Usage
Use of this repository is meant to be idempotent. You should be able to run both Terraform and the Ansible playbooks repeatedly without negatively impacting the system. While hte infrastructure and cluster are mutable in practice, the use of this repo should leave them conceptually immutable, thus changes made outside this repo that alter the state of the base infrastructure footprint (nodes, network, images, k8s version, prometheus config, etc.) will be rolled back to their original state when the automation in this repo is rerun.  

Multiple clusters can be managed out of this repository through the creation of project directories within the inventory folder. A sample `develop` project is provided as an example. When getting started, it is best to copy the `inventory/develop` directory, edit the `cluster.tfvars` and `group_vars/all/agave.yml` files with project names that suit your purpose, and use that as your default inventory. This process can be repeated for as many projects as you would like to create without conflict.  

### Provisioning 

Provision and deploy a new Kubernetes cluster.

> _You must have your Jetstream Openstack environment set up to provision a new cluster. If this is your first time using Jetstream or its OpenStack APIs, you will have to [request API access](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/39682057/Using+the+Jetstream+API). Once your request is approved, you can follow the instructions for [Setting up openrc.sh](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/39682064/Setting+up+openrc.sh) to configure your local environment to access the Jetstream Openstack APIs._


```shell

cd inventory/develop

# Source the openstack.rc file for your Jetstream allocation. Enter the password when prompted. 
# This will neveer leave your local computer.
source path/to/openrc.sh 

# Initialize Terraform modules and new plan
./terraform_init.sh

# Update cluster.tfvars with a unique cluster_name and network_name for your project.

# Now run the plan to provision the k8s infrastructure on Jetstream 
./terraform_apply.sh
```

At this point you have provisioned the VM, networking, routers, ip addresses, security groups, and storage needed for your cluster on Jetstream. 

___NOTE: Always run Terraform from your inventory project directory___  

### Spraying the cluster

Now we are ready to install Kubernetes on the provisioned nodes.  

```bash
# Once complete, return to the root project directory and spray the new hosts to lay down Kubernetes
cd ../../
ansible-playbook -i inventory/develop/hosts playbooks/kubespray/k8s-cluster.yml --become
```

You will now have a working k8s cluster running on Jetsream. If you would like to access the cluster from your local host, a kubectl config file pointing at your cluster will be present on your local file system at `inventory/develop/artifacts/admin.conf`. 

Next, it's time to deploy Agave's support infrastructure to handle things like logging, metrics collection, cert management, storage provisioning, etc. 

```shell
# Run the top level site.yml playbook to orchestrate all the other playbooks needed to set things up
ansible-playbook -i inventory/develop/hosts playbooks/site.yml
```

Once completed, you should be able to access and view your cluster services. We recommend using [Lens](https://k8slens.dev/) as a convenient way of interacting with your cluster. You can use your existing `inventory/develop/artifacts/admin.conf` to add a new cluster to Lens. 

### Scaling Up

To scale up the cluster worker count, edit your `inventory/develop/cluster.tfvars` file, updating the `number_of_k8s_nodes` value to the number of desired worker nodes.  

> Generally speaking, it's not a great idea to change the master or etcd topology as that gives kubespray fits. It's better to just tear the cluster down and provision a new one if you need a different topology. 

```bash
cd  inventory/develop

# edit the number_of_k8s_nodes value to a greater value

# Apply the new Terraform plan
./terraform_apply.sh
```  

Once complete, run the kubespray scale playbook to join the new nodes to the cluster.  

```bash
# return to the project root directory
cd ../../

# run the scale playbook
ansible-playbook -i inventory/develop/hosts playbooks/kubespray/k8s-scale.yml --flush-cache --become  
```

### Scaling Down

To scale down the cluster worker count, find the last N worker nodes to remove using kubectl. The following command lists
the last 2 nodes.

```bash
kubectl --kubeconfig='inventory/develop/artifacts/admin.conf' \
        get nodes -o Name -l node-role.kubernetes.io/master!= | sed 's#node/##' | tail -n 2
```

Once you have the list of node names to remove, run the `playbooks/kubespray/k8s-remove-node.yml`, passing a comma-separated list of those hosts as an argument. Assuming the nodes returned from the above command were `agavek8s-k8s-node-1` and `agavek8s-k8s-node-2`, the following command will remove them from the k8s cluster.

```bash
ansible-playbook -i inventory/develop/hosts playbooks/kubespray/k8s-remove-node.yml --become -e node=agavek8s-k8s-node-1,agavek8s-k8s-node-2
```  

> Generally speaking, it's not a great idea to change the master or etcd topology as that gives kubespray fits. It's better to just tear the cluster down and provision a new one if you need a different topology. 

```bash
cd  inventory/develop

# edit the number_of_k8s_nodes value, reducing the number by the number of nodes removed. 
# In the example above, this would be 2.

# Apply the new Terraform plan
./terraform_apply.sh
```

Once Terraform completes, the resources will be released from Openstack and no longer consuming your allocation.

```bash
cd  inventory/develop

# edit the number_of_k8s_nodes value to a greater value

# Apply the new Terraform plan
./terraform_apply.sh
```  

Once complete, run the kubespray scale playbook to join the new nodes to the cluster.

### Cleanup

You can tear down the cluster simply by destroying the Terraform plan you used to provision the Jetsream cluster initially.

```bash
cd inventory/develop

# Destroy the Terraform play
./terraform_destroy.sh  
```  

## Architecture

### Cloud Infrastructure
The Jetstream (Openstack) cluster at Indiana University is used as the IaaS provider for our cloud infrastructure. We interact with it using Terraform. For deployments under 10 nodes, this works fine. For larger deployments, errors may be experienced spinning up nodes, attaching networking, and mounting storage. Terraform is idempotent in nature, so rerunning the plan or, at worst, destroying and reapplying the plan usually overcomes this problem. Additionally, limiting the number of parallel threads Terraform uses to at most 3 can improve reliability when running against the Jetstream Openstack APIs. 

#### DNS
Jetstream does not offer Octavia LBaaS or external DNS integration, so DNS needs to be managed independently of Openstack through. In practice this is only an issue when deploying the Agave Helm charts as Kubernetes internal DNS is sufficient for all cluster admin services. 

#### VM
`m1.large` Ubuntu 20.04 instances are used for the master and worker nodes in the cluster by default. This can be edited in the `cluster.tfvars` The resulting VM will have 10 cores, 30GB of memory, and a 60GB root partition. An additional 100GB Cinder block device partitioned as an ext4 disk and mounted at `/extra/var/lib/docker`.  

#### Networking
All VM instances are open to TCP and UDP ingress and egress on all ports within their private network. TCP is open on ports 80, 443, and 6443 to the outside world. Port 22 is open on the master nodes by default. This should be closed for a production deployment.  

#### Storage
Cinder is used to provide expanded local host storage for each VM. Manilla is used as an externally managed shared (NFS) file system. No NVME or SSD is available within Jetstream at this time.  


### Kubernetes
Kubernetes is deployed across the VMs using Kubespray. Kubespray is highly configurable, and works well on bare metal and cloud infrastructures alike. The following sections describe the specific configure we utilize on Jetstream for our purposes.  

> ***NOTE:** Detailed information about Kubespray can be found in the Kubespray [getting-started.md](submodules/zonca/docs/getting-started.md) guide.*  

#### Authentication
Authentication to the Kubernetes API is handled through issuance of custom service accounts. A custom deployment account is created for each deployment environment. ie. production, staging, development, feature-xyz, etc. 

#### CSI
We use two storage plugins in our default deployment to provide local as well as NFS persistent volumes through the CSI interface. 
* The core CSI plugin included with Kubernetes provides NFS support. 
* The Rancher Local Path Provisioner provides full CSI support for local storage on each node.

#### CNI
Kubespray ships with Flannel as the default networking plugin. In practice, Flannel tends to drop a lot of ssh traffic on Jetstream, so we instead use [Weave](https://github.com/weaveworks/weave) as our CNI plugin. 

#### Ingress
The Nginx Ingress Controller is installed as the default Ingress controller by Kubespray. It integrates with cert-manager to provide automatic ssl for all Ingress and Services Kubernetes resources.  

#### Access Control
Role Based Access Controls and Pod Security Policies are both enabled within the cluster. Every workload deployed to the cluster runs under its own service account with restrictive RBAC and PSP specific to the workload. It is intended that distinct namespaces be created for each workload to preserve cluster security and workload isolation. 


### Cluster admin services
A full suite of administration services is deployed on top of the vanilla Kubernetes deployment created by Kubespray. We categorize them by purpose and describe each in turn in the following sections.

#### Authentication
* [rancher-local-path-provisioner](https://github.com/rancher/local-path-provisioner) provides fully lifecycle CSI support for local storage. 

#### Monitoring
* [Prometheus](https://prometheus.io/docs/introduction/overview/) is an open-source systems monitoring and alerting toolkit

#### Logging
* [Loki](https://grafana.com/oss/loki/) is a horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by Prometheus.
* [Grafana](https://grafana.com/oss/grafana/) allows you to query, visualize and alert on metrics and logs no matter where they are stored.

#### Certificate management
* [Cert Manager](https://cert-manager.io/) by Jetstack provides X.509 certificate management within Kubernetes. This repo will configure interaction with Let's Encrypt and generation of cluster-wide PKI for MTLS and secure intra cluster communication.

#### Databases
* [KubeDB](https://kubedb.com) AppsCode simplifies and automates routine database tasks such as provisioning, patching, backup, recovery, failure detection, and repair for various popular databases on private and public clouds.


License
-------

BSD 3-Clause
