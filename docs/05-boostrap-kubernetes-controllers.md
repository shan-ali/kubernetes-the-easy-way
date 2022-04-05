# Bootstrapping the Kubernetes Control Plane

In this section you will setup the Kubernetes Control Plane for `controller-1` & `controller-2` using kubeadm. 

References: 
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node

## Initialize the First Control Plane

On `controller-1`

```
sudo kubeadm init \
  --control-plane-endpoint "172.22.5.30:6443"\
  --upload-certs\
  --pod-network-cidr "10.142.0.0/24"\
  --ignore-preflight-errors=NumCPU,Mem
```

> Note: 
> - `--control-plane-endpoint` : set to the address or DNS and port of the load balancer
> - `--upload-certs` : uploads the certificates that should be shared across all the control-plane instances to the cluster
> - `--pod-network-cidr` : the pod network to be used by the CNI (Calico in our case)
> - `--ignore-preflight-errors` : we will ignore the minumum requirements for NumCPU & Mem. Alternatively we can allocate more to our Multipass hosts. 

You should see the below output. Please keep note of the different `kubeadm join` commands for later.

>output
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/conf

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 172.22.5.30:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --control-plane --certificate-key <certificate-key>

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.22.5.30:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

## Join the Second Control Plane

We will use the first `kubeadm join` command from the output above to join `controller-2` as a control plane node to the cluster

On `controller-2`

```
sudo kubeadm join 172.22.5.30:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --control-plane --certificate-key <certificate-key> --ignore-preflight-errors=NumCPU,Mem
```

> Note: we are adding `--ignore-preflight-errors=NumCPU,Mem` 

## Configure kubectl

In order to use kubectl without passing the admin kubeconfig we need to add it to our home directory for any host that we want to use kubectl.

On `controller-1` & `controller-2`

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Verify Control Plane Nodes

On `controller-1` 

```
kubectl get nodes
```

> output 

```
NAME           STATUS     ROLES                  AGE     VERSION
controller-1   NotReady   control-plane,master   19m     v1.23.5
controller-2   NotReady   control-plane,master   2m28s   v1.23.5
```
> Note: status is NotReady since we have not configured pod networking yet

## Verify Control Plane Pods

When using kubeadm, all of the kubernetes components will run as pods on the Control Plane. Lets take a look at them

```
kubectl get pods -n kube-system -o wide
```
> output
```
NAME                                   READY   STATUS    RESTARTS        AGE     IP            NODE           NOMINATED NODE   READINESS GATES
coredns-64897985d-5lwzk                0/1     Pending   0               7m7s    <none>        <none>         <none>           <none>
coredns-64897985d-mnwtm                0/1     Pending   0               7m7s    <none>        <none>         <none>           <none>
etcd-controller-1                      1/1     Running   0               7m22s   172.22.5.11   controller-1   <none>           <none>
etcd-controller-2                      1/1     Running   0               5m52s   172.22.5.12   controller-2   <none>           <none>
kube-apiserver-controller-1            1/1     Running   0               7m20s   172.22.5.11   controller-1   <none>           <none>
kube-apiserver-controller-2            1/1     Running   0               5m56s   172.22.5.12   controller-2   <none>           <none>
kube-controller-manager-controller-1   1/1     Running   1 (5m40s ago)   7m20s   172.22.5.11   controller-1   <none>           <none>
kube-controller-manager-controller-2   1/1     Running   0               5m56s   172.22.5.12   controller-2   <none>           <none>
kube-proxy-75n8h                       1/1     Running   0               7m6s    172.22.5.11   controller-1   <none>           <none>
kube-proxy-qzz7m                       1/1     Running   0               5m57s   172.22.5.12   controller-2   <none>           <none>
kube-scheduler-controller-1            1/1     Running   1 (5m41s ago)   7m20s   172.22.5.11   controller-1   <none>           <none>
kube-scheduler-controller-2            1/1     Running   0               5m56s   172.22.5.12   controller-2   <none>           <none>
```
> Note: coredns pods are in a Pending state and are not running. This is because there is no pod networking configured at the moment. We will configure this in the next lab. 

## Verify etcd 

Check if etcd pods are running

```
kubectl get pods -n kube-system -o wide | grep etcd
```
> output
```
etcd-controller-1                          1/1     Running   7 (23m ago)    18h   172.22.5.11    controller-1   <none>           <none>
etcd-controller-2                          1/1     Running   0              18h   172.22.5.12    controller-2   <none>           <none>
```

Check the status of the etcd cluster

```
kubectl exec -n kube-system etcd-controller-1 -- etcdctl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key member list -w table
```
> output
```
+------------------+---------+--------------+--------------------------+--------------------------+------------+
|        ID        | STATUS  |     NAME     |        PEER ADDRS        |       CLIENT ADDRS       | IS LEARNER |
+------------------+---------+--------------+--------------------------+--------------------------+------------+
| 710f2742e7b07dfe | started | controller-1 | https://172.22.5.11:2380 | https://172.22.5.11:2379 |      false |
| f8a112e8a2842edf | started | controller-2 | https://172.22.5.12:2380 | https://172.22.5.12:2379 |      false |
+------------------+---------+--------------+--------------------------+--------------------------+------------+
```

Next: [Configure Pod Networking](06-configure-pod-networking.md)