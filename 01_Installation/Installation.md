# Preparation & Installation
[kubeadm Docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

## 1. Disable swap on both machines
- `sudo swapoff -a`
## 2. Install a container runtime on each node (containerd)
- Enable IPv4 packet forwarding [k8s docs](https://kubernetes.io/docs/setup/production-environment/container-runtimes#prerequisite-ipv4-forwarding-optional)

- Install containerd [docker docs](https://docs.docker.com/engine/install/ubuntu/)
    - Set up docker's apt repository
    - Install containerd (sudo apt-get install containerd.io)
- Make sure both kubelet and the container runtime (containerd) use the same cgroup driver
    - ```containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sed 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml```
    - `sudo systemctl restart containerd`
    - `sudo systemctl status containerd` (make sure it's running)
- Repeat on both nodes

## 3. Install kubeadm, kubelet and kubectl on each node
-  [Follow doc instructions (currently step 1-4, step 5 can be skipped)](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)

## 4. Initialize the control plane node 
- [K8s docs - Initializing your control-plane node](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node)
- `sudo kubeadm init --pod-network-cidr=10.200.0.0/16 --apiserver-advertise-address=192.168.0.12`
- pod-network-cidr = Internal cluster network
- apiserver-advertise-address = The address that is advertised to other nodes where to find the API server. Can be skipped if the machine only has one network interface.
- Run the commands produced by the output from the previous command ('To start using your cluster...')
- `kubectl get node -o wide` Make sure INTERNAL-IP got set to the same as the apiserver-advertise-address
- Copy and run the command to join a worker node from the output produced by the kubeadm init command above.

## 5. Install a Container Network Interface (CNI)
- We must deploy a CNI to enable our pods to communicate with each other. Cluster DNS (CoreDNS pods) won't start until we've done this. [K8s docs - Installing a Pod network add-on](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)
- I'll use [Flannel](https://github.com/flannel-io/flannel)
- [Version i got working (0.25.1)](https://github.com/flannel-io/flannel/releases/tag/v0.25.1)
- Download kube-flannel.yml to the control-plane node and make edits:
    - Change the network to be the cluster CIDR we defined earlier (10.200.0.0/16) at: net-conf.json > Network `"Network": "10.200.0.0/16"`
    - Specify the network interface to be used (in my case wlan0), under containers > args `--iface=wlan0`

- See the [final kube-flannel.yml](/01_Installation/kube-flannel.yml)
- Make sure the linux-modules package is installed on the machine. For example, when running Ubuntu with kernel version 6.8.0-1020-raspi, run the following: `sudo apt install linux-modules-6.8.0-1020-raspi`

    - To see kernel version: `uname -r`
- Apply the kube-flannel.yml on the control-plane node: `kubectl apply -f kube-flannel.yml`
- Display all pods: `kubectl get pod -A`

## Basic Cluster running!
- We now have a working cluster with pods that are able to communicate with each other!

