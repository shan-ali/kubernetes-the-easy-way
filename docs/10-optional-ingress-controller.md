# Ingress Controller (Optional Step)

"In order for the Ingress resource to work, the cluster must have an ingress controller running."Ingress controllers are not shipped with K8s by default and therefore we will need to install it. We will be using the nginx ingress controller.

The Ingress resource "exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource."

Reference: 
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/ 
- https://kubernetes.io/docs/concepts/services-networking/ingress/

## Install by Manifest

Install the controller in the ingress-nginx namespace.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/cloud/deploy.yaml
```

Check if controller is running.

```
kubectl get pods --namespace=ingress-nginx
```
>output
```NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-x7nrt        0/1     Completed   0          107s
ingress-nginx-admission-patch-xzgc4         0/1     Completed   1          107s
ingress-nginx-controller-5b6f946f99-sb4d2   1/1     Running     0          107s
```

## Check if MetalLB set External IP

"As soon as MetalLB sets the external IP address of the ingress-nginx LoadBalancer Service, the corresponding entries are created in the iptables NAT table and the node with the selected IP address starts responding to HTTP requests on the ports configured in the LoadBalancer Service"

```
kubectl -n ingress-nginx get svc
```
>output
```
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.98.168.107   10.22.0.0     80:31701/TCP,443:30577/TCP   3m55s
ingress-nginx-controller-admission   ClusterIP      10.96.36.75     <none>        443/TCP                      3m55s
```

Reference: https://kubernetes.github.io/ingress-nginx/deploy/baremetal/

## Local Testing

Let's create a simple web server and the associated service:

```
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
```

Then create an ingress resource. The following example uses an host that maps to localhost:

```
kubectl create ingress demo-localhost --class=nginx \
  --rule=demo.localdev.me/*=demo:80
```

Now, forward a local port to the ingress controller:

```
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80 &
```
> Note: 
>- adding `&` will run the port-forwarding in the background
>- you will need to do ctrl+c to get out of the process (it will continue to run in the background)

At this point, if you access http://demo.localdev.me:8080/, you should see an HTML page telling you "It works!" (Needs to be accessed from our VM).

```
curl http://demo.localdev.me:8080/
```
>output
```
Handling connection for 8080
<html><body><h1>It works!</h1></body></html>
```

Kill the port-forwarding process that is running in the background

```
pkill -f "port-forward"
```

Reference: https://kubernetes.github.io/ingress-nginx/deploy/#local-testing

## Online Testing

If your Kubernetes cluster is a "real" cluster that supports services of type LoadBalancer, it will have allocated an external IP address or FQDN to the ingress controller.

You can see that IP address or FQDN with the following command:

```
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
```

It will be the EXTERNAL-IP field. If that field shows <pending>, this means that your Kubernetes cluster wasn't able to provision the load balancer (generally, this is because it doesn't support services of type LoadBalancer).

Once you have the external IP address (or FQDN), set up a DNS record pointing to it. We can add this by adding to the `/etc/hosts` file like below

```
10.22.0.0 www.demo.io
```

Create the ingress resource 
```
kubectl create ingress demo --class=nginx \
  --rule="www.demo.io/*=demo:80"
```

You should then be able to see the "It works!" page when you connect to http://www.demo.io/ (Needs to be accessed from our VM since we dont have an external LB). 

```
curl http://www.demo.io
```

Reference: https://kubernetes.github.io/ingress-nginx/deploy/#online-testing 

## Clean Up

```
kubectl delete ingress demo-localhost
kubectl delete deployment demo
kubectl delete service demo
kubectl delete ingress demo
```