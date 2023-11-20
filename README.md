# Multi node k3s cluster in spot instances

## First create the number of spot instances you want

Lets assume we are creating 1 master node and 2 agent nodes

## SSH into one instance which should be master and do the following

### Name the host with explict name to avoid name colision
```sh
sudo -i

hostnamectl set-hostname master1

echo master1 > /etc/hostname
```

### Now install K3S cluster

```sh
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--tls-san x.x.x.x" sh -s -
```

> enter public ip address of the EC2 instance on `--tls-san`

### Make your cluster full Control Plane node

```sh
cat <<EOF > /etc/rancher/k3s/config.yaml
> cluster-init: true
> EOF
```
> This may not work even with `sudo` so have to do that command in interractive sudo shell `sudo -i`

### After doing that restart your k3s cluster

```sh
sudo systemctl restart k3s
```

- it should look like this when you do `kubectl get nodes`
```
NAME      STATUS   ROLES                       AGE   VERSION
master1   Ready    control-plane,etcd,master   13m   v1.27.7+k3s2
```

### Take note of the node-token of the master node
```sh
sudo cat /var/lib/rancher/k3s/server/node-token
```

## SSH into other two instances and do the following

### Name the host with explict name to avoid name colision
```sh
sudo -i

hostnamectl set-hostname agent1

echo agent1 > /etc/hostname
```
> here change the hostname in increment or any way you see fit

### Install K3S agent in worker instances

```sh
curl -sfL https://get.k3s.io | K3S_URL=https://<master-node-public-ip>:6443 K3S_TOKEN=<master-node-token> sh -
```

- After installing agent in both instances go to master node and do `kubectl get nodes`, it should look like this

```
NAME      STATUS   ROLES                       AGE   VERSION
agent1    Ready    <none>                      92s   v1.27.7+k3s2
agent2    Ready    <none>                      10s   v1.27.7+k3s2
master1   Ready    control-plane,etcd,master   23m   v1.27.7+k3s2
```

## Now  your multi node K3S cluster is ready
