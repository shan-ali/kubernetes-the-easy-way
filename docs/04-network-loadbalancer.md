### Provision a Network Load Balancer

Login to `loadbalancer` instance.

```
sudo apt-get update && sudo apt-get install -y haproxy
```

```
cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg 
frontend kubernetes
    bind 172.22.5.30:6443
    option tcplog
    mode tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode tcp
    balance roundrobin
    option tcp-check
    server controller-1 172.22.5.11:6443 check fall 3 rise 2
    server controller-2 172.22.5.12:6443 check fall 3 rise 2
EOF
```

```
sudo service haproxy restart
```

Next: [Boostrap Kubernetes Controllers](05-boostrap-kubernetes-controllers.md)