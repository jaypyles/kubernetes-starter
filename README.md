# Summary

## Startup

- Create two virtual machines using VirtualBox, using a host-only network adapter configuration

- Enable the DHCP server on the host only network

  - `sudo systemctl restart networking && sudo dhclient` to assign an address using dhcp

## Creating a Cluster

### Master Node

- Disable swap

```bash
sudo swapoff -a

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

- Init the cluster

```bash
sudo kubeadm init --apiserver-advertise-address=<HOST_ONLY_ADAPTER_IP> --pod-network-cidr=192.168.0.0/16
```

- Apply a network configuration

```bash
# apply flannel network config
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

- Get the join token

```bash
kubeadm token create --print-join-command
```

### Worker Node

- Join cluster

```bash
#join the cluster
sudo kubeadm join <MASTER_IP> --token <TOKEN> --discovery-token-ca-cert-hash <HASH>
```

## Troubleshooting

Using Debian 11 or Ubuntu 22.04, the following steps are required to enable systemd cgroup

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo service containerd restart
sudo service kubelet restart
```

This is done manually for now to avoid cert errors:

```bash
sudo rm -rf $HOME/.kube || true
sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# K3S setup

Install k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

Get join token
```bash
cat /etc/rancher/k3s/k3s.yaml
```

Join the cluster on the worker node

```bash
export URL="https://<MASTER_IP>:6443"
export TOKEN="<<TOKEN>>"
curl -sfL https://get.k3s.io | K3S_URL=$URL K3S_TOKEN=$TOKEN sh -
```


