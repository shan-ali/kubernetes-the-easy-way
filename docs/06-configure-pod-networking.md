# Provisioning Pod Network

We chose to use CNI - [calico](https://projectcalico.docs.tigera.io/about/about-calico) as our networking option.

We will be following the official documentation for installing calico [on-premises deployments](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)

References:
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network
- https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises

## Deploy Calico Network

on `controller-1`

Download the Calico networking manifest for the Kubernetes API datastore.
```
curl https://projectcalico.docs.tigera.io/archive/v3.22/manifests/calico.yaml -O
```

If you are using pod CIDR `192.168.0.0/16`, skip to the next step. If you are using a different pod CIDR with kubeadm, no changes are required - Calico will automatically detect the CIDR based on the running configuration. For other platforms, make sure you uncomment the `CALICO_IPV4POOL_CIDR` variable in the manifest and set it to the same value as your chosen pod CIDR.

We already specified this when we started our cluster with `--pod-network-cidr "10.142.0.0/24"`. Therefore we do not need to make any modifications to the `calico.yaml` file. 

Apply the manifest using the following command.

```
kubectl apply -f calico.yaml
```

## Verification

List the registered Kubernetes nodes from the `controller-1` node:

```
kubectl get pods -n kube-system -o wide | grep calico
```

> output

```
calico-kube-controllers-56fcbf9d6b-tq922   1/1     Running   0             18m   10.142.0.194   controller-2   <none>           <none>
calico-node-d2gcj                          1/1     Running   0             18m   172.22.5.11    controller-1   <none>           <none>
calico-node-hk8k6                          1/1     Running   0             18m   172.22.5.12    controller-2   <none>           <none>
```

Now if we list our nodes their status will be `Ready`
```
kubectl get nodes
```

> output

```
NAME           STATUS   ROLES                  AGE   VERSION
controller-1   Ready    control-plane,master   50m   v1.23.5
controller-2   Ready    control-plane,master   32m   v1.23.5
```


Next: [Boostrap Kubernetes Workers](07-boostrap-kubernetes-workers.md)
