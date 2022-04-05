# Bootstrapping the Kubernetes Workers

In this section you will setup the Kubernetes Workers for `worker-1` & `worker-2` using kubeadm. 

References: 
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#install-workers
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes

## Initialize Workers

On `worker-1` & `worker-2`

```
sudo kubeadm join 172.22.5.30:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --ignore-preflight-errors=NumCPU,Mem
```

> Note: we are adding `--ignore-preflight-errors=NumCPU,Mem` 

## Verify Worker Nodes

On `controller-1` 

```
kubectl get nodes
```

> output 

```
NAME           STATUS   ROLES                  AGE   VERSION
controller-1   Ready    control-plane,master   30m   v1.23.5
controller-2   Ready    control-plane,master   29m   v1.23.5
worker-1       Ready    <none>                 35s   v1.23.5
worker-2       Ready    <none>                 25s   v1.23.5
```

> Note: status is Ready since pod networking is setup

## Verify Pod Networking for Worker Nodes

We should now see two more calico pods running on our two new worker nodes

```
kubectl get pods -n kube-system -o wide | grep calico
```

> output

```
calico-kube-controllers-56fcbf9d6b-tq922   1/1     Running   0             22m     10.142.0.194   controller-2   <none>           <none>
calico-node-85nb7                          1/1     Running   0             2m48s   172.22.5.22    worker-2       <none>           <none>
calico-node-d2gcj                          1/1     Running   0             22m     172.22.5.11    controller-1   <none>           <none>
calico-node-hk8k6                          1/1     Running   0             22m     172.22.5.12    controller-2   <none>           <none>
calico-node-q6pv8                          1/1     Running   0             2m58s   172.22.5.21    worker-1       <none>           <none>
```

Next: [Smoke Test](08-smoke-test.md)