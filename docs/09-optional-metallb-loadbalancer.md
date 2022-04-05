# MetalLB (Optional Step)

"Kubernetes does not offer an implementation of network load balancers (Services of type LoadBalancer) for bare-metal clusters. The implementations of network load balancers that Kubernetes does ship with are all glue code that calls out to various IaaS platforms (GCP, AWS, Azure…). If you’re not running on a supported IaaS platform (GCP, AWS, Azure…), LoadBalancers will remain in the “pending” state indefinitely when created."

"MetalLB aims to redress this imbalance by offering a network load balancer implementation that integrates with standard network equipment, so that external services on bare-metal clusters also “just work” as much as possible."

Reference: https://metallb.universe.tf/

This loadbalancer will be used in conjunction with the next steps to setup an Ingress Controller on our cluster.

## Install by Manifest

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml
```

This will deploy MetalLB to your cluster, under the metallb-system namespace. The components in the manifest are:

- The `metallb-system/controller` deployment. This is the cluster-wide controller that handles IP address assignments.
- The `metallb-system/speaker` daemonset. This is the component that speaks the protocol(s) of your choice to make the services reachable.
- Service accounts for the controller and speaker, along with the RBAC permissions that the components need to function.

```
kubectl get all -n metallb-system
```
>output
```
NAME                             READY   STATUS    RESTARTS   AGE 
pod/controller-57fd9c5bb-tcjc9   1/1     Running   0          102s
pod/speaker-87nkz                1/1     Running   0          102s
pod/speaker-9jmrh                1/1     Running   0          102s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE 
daemonset.apps/speaker   2         2         2       2            2           kubernetes.io/os=linux   102s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           102s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-57fd9c5bb   1         1         1       102s
```

Reference: https://metallb.universe.tf/installation/

## MetalLB ConfigMap

"MetalLB requires a pool of IP addresses in order to be able to take ownership of the ingress-nginx Service. This pool can be defined in a ConfigMap named config located in the same namespace as the MetalLB controller. This pool of IPs must be dedicated to MetalLB's use, you can't reuse the Kubernetes node IPs or IPs handed out by a DHCP server."

```
cat > metallbconfigmap.yaml <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.22.0.0-10.22.0.5
EOF

kubectl apply -f metallbconfigmap.yaml
```
Reference: https://kubernetes.github.io/ingress-nginx/deploy/baremetal/

Next: [Optional - Ingress Controller](10-optional-ingress-controller.md)