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

### Worker Node

- Join cluster

```bash
#join the cluster
sudo kubeadm join <MASTER_IP> --token <TOKEN> --discovery-token-ca-cert-hash <HASH>
```
