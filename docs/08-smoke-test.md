# Smoke Test

In this lab you will complete a series of tasks to ensure your Kubernetes cluster is functioning correctly.

## Check CoreDNS Nodes

CoreDNS pods should be running now

```
kubectl get pods -n kube-system -o wide | grep coredns
```
> output
```
coredns-64897985d-5wvq7                    1/1     Running   0             52m     10.142.0.193   worker-2       <none>           <none>
coredns-64897985d-lqj8r                    1/1     Running   0             52m     10.142.0.128   controller-2   <none>           <none>
```

### Core DNS Verification

Create a `busybox` deployment:

```
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

List the pod created by the `busybox` deployment:

```
kubectl get pods -l run=busybox
```

> output

```
NAME                      READY   STATUS    RESTARTS   AGE
busybox-bd8fb7cbd-vflm9   1/1     Running   0          10s
```

Execute a DNS lookup for the `kubernetes` service inside the `busybox` pod:

```
kubectl exec -ti busybox -- nslookup kubernetes
```

> output

```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

## Deployments

In this section you will verify the ability to create and manage [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

Create a deployment for the [nginx](https://nginx.org/en/) web server:

```
kubectl create deployment nginx --image=nginx
```

List the pod created by the `nginx` deployment:

```
kubectl get pods -l app=nginx
```

> output

```
NAME                    READY   STATUS    RESTARTS   AGE
nginx-dbddb74b8-6lxg2   1/1     Running   0          10s
```

## Services

In this section you will verify the ability to access applications remotely using [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/).

Create a service to expose deployment nginx on node ports.

```
kubectl expose deploy nginx --type=NodePort --port 80
```


```
PORT_NUMBER=$(kubectl get svc -l app=nginx -o jsonpath="{.items[0].spec.ports[0].nodePort}")
```

Test to view NGINX page

```
curl http://worker-1:$PORT_NUMBER
curl http://worker-2:$PORT_NUMBER
```

> output

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Logs

In this section you will verify the ability to [retrieve container logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/).

Retrieve the full name of the `nginx` pod:

```
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```

Print the `nginx` pod logs:

```
kubectl logs $POD_NAME
```

> output

```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/03/22 20:01:06 [notice] 1#1: using the "epoll" event method
2022/03/22 20:01:06 [notice] 1#1: nginx/1.21.6
2022/03/22 20:01:06 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/03/22 20:01:06 [notice] 1#1: OS: Linux 5.4.0-104-generic
2022/03/22 20:01:06 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/03/22 20:01:06 [notice] 1#1: start worker processes
2022/03/22 20:01:06 [notice] 1#1: start worker process 31
172.22.5.21 - - [22/Mar/2022:20:01:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.68.0" "-"
10.142.0.192 - - [22/Mar/2022:20:01:45 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.68.0" "-"
```

## Exec

In this section you will verify the ability to [execute commands in a container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/#running-individual-commands-in-a-container).

Print the nginx version by executing the `nginx -v` command in the `nginx` container:

```
kubectl exec -ti $POD_NAME -- nginx -v
```

> output

```
nginx version: nginx/1.21.6
```

## Clean Up

```
kubectl delete deployment nginx --force
kubectl delete pod busybox --force
```

Next: [Optional - MetalLB Loadbalancer](09-optional-metallb-loadbalancer.md)