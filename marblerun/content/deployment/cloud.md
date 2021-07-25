# Cloud deployment

This guide walks you through setting up Marblerun on different CSP offerings individually.

## Azure confidential computing VMs

[Azure confidential computing services](https://azure.microsoft.com/en-us/solutions/confidential-compute/) are generally available and provide access to VMs with Intel SGX enabled in [DCsv2 VM instances](https://docs.microsoft.com/en-us/azure/virtual-machines/dcv2-series).
The description below uses a VM running Ubuntu 18.04.

### Prerequisites

* [Update and install EGo](https://github.com/edgelesssys/ego#install)
* [Update and install the Azure DCAP client](https://docs.microsoft.com/en-us/azure/confidential-computing/quick-create-portal#3-install-the-intel-and-open-enclave-packages-and-dependencies)

### Deploy Marblerun

You can run Marblerun standalone on your Azure DCsv2 VM, see our [standalone guide](content/deployment/standalone.md).
Alternatively, you can install a Kubernetes cluster, probably the simplest option would be [minikube](https://minikube.sigs.k8s.io/docs/start/), see our [Kubernetes guide](content/deployment/kubernetes.md) on how to install Marblerun in minikube.

## Azure Kubernetes Services (AKS)

Azure Kubernetes Service (AKS) offers a popular deployment technique relying on
Azure's cloud resources. AKS hosts Kubernetes pods in Azure confidential compute
VMs and exposes the underlying confidential compute hardware.

This section describes the workflow to create an AKS cluster with confidential
compute VMs and deploy Marblerun.

### Prerequisites

Follow the instructions on the [AKS Confidential Computing Quick Start guide](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-get-started)
to provision an AKS cluster with Intel SGX enabled worker nodes.

### Deploy Marblerun

See our [Kubernetes guide](content/deployment/kubernetes.md) on how to install Marblerun in your AKS cluster.
