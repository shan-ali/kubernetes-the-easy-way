# Installing the Client Tools

First identify a system from where you will perform administrative tasks, such as creating certificates, kubeconfig files and distributing them to the different VMs.

If you are on a Linux laptop, then your laptop could be this system. In my case I chose the `controller-1` node to perform administrative tasks. Whichever system you chose make sure that system is able to access all the provisioned VMs through SSH to copy files over.

## Access all VMs

Generate Key Pair on `controller-1` node

```
ssh-keygen
```

Leave all settings to default.

View the generated public key ID at:

```
cat .ssh/id_rsa.pub
```
>output
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD......8+08b ubuntu@controller-1
```

Move public key of controller-1 to all other VMs. Access each VM using the following multipass command

```
multipass shell controller-2
```

Then add public key from controller-1 to the authorized keys for all hosts (including controller-1)

```
cat >> ~/.ssh/authorized_keys <<EOF
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD......8+08b ubuntu@controller-1
EOF
```

Next: [Network Loadbalancer](04-network-loadbalancer.md)