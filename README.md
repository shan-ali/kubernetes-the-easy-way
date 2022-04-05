# Kubernetes The Easy Way On Multipass

This tutorial walks you through setting up Kubernetes using `kubeadm` on a local machine using Multipass. 
> This tutorial is for educational purposes only! Don't run this in production :)

## Cluster Details

Kubernetes The Hard Way guides you through bootstrapping a highly available Kubernetes cluster with end-to-end encryption between components and RBAC authentication.

* [Kubernetes](https://github.com/kubernetes/kubernetes) 1.23.4
* [Docker Container Runtime](https://docs.docker.com/) 20.10.13
* [Calico](https://projectcalico.docs.tigera.io/about/about-calico) 3.22

## Labs

* [Prerequisites](docs/01-prerequisites.md)
* [Provisioning Compute Resources](docs/02-compute-resources.md)
* [Installing the Client Tools](docs/03-client-tools.md)
* [Cluster Control Plane Loadbalancer](docs/04-network-loadbalancer.md)
* [Bootstrapping the Kubernetes Control Plane](docs/05-bootstrapping-kubernetes-controllers.md)
* [Deploy Calico - Pod Networking Solution](docs/06-configure-pod-networking.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/07-bootstrapping-kubernetes-workers.md)
* [Smoke Test](docs/08-smoke-test.md)
* [Optional - MetalLB Loadbalancer](09-optional-metallb-loadbalancer.md)
* [Optional - Ingress Controller](docs/10-optional-ingress-controller.md)

