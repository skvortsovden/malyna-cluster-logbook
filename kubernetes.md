# kubernetes

## installing k3s on raspberry pi cluster

### enable cgroups
kubernetes requires `control groups (cgroups)` to manage resources. 

Edit the boot file: 

```
sudo vim /boot/firmware/cmdline.txt
```

Append this to the end of the existing line (do not create a new line):

```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

Reboot all nodes.

### install k3s

1. install k3s on the master node (malyna):
```
curl -sfL https://get.k3s.io | sh -
```

2. get token for nodes to join the cluster:
```
sudo cat /var/lib/rancher/k3s/server/node-token
```
3. install ks3 on worker nodes (kalyna and lohyna) using the token from step 2:
```
curl -sfL https://get.k3s.io | K3S_URL=https://malyna:6443 K3S_TOKEN=<NODE_TOKEN> sh -
```


result:

```
devops@malyna:~ $ sudo kubectl get nodes
NAME     STATUS   ROLES           AGE    VERSION
kalyna   Ready    <none>          3h4m   v1.34.5+k3s1
lohyna   Ready    <none>          3h3m   v1.34.5+k3s1
malyna   Ready    control-plane   3h7m   v1.34.5+k3s1
```