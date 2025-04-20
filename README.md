# K8s_Homelab

## Hardware

- Raspberry Pi 5 16GB (Control plane)
- Raspberry Pi 5 8GB (Worker)
- OS on both nodes: Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-1020-raspi aarch64)

## Cluster info

- podSubnet: 10.200.0.0/16 (10.200.0.0-10.200.255.255)

- serviceSubnet: 10.96.0.0/12 (10.96.0.0-10.111.255.255)

- MetalLB IP pool for LoadBalancer services:
  - name: my-pool
  - ip range: 192.168.0.79-192.168.0.99

## Steps

[1. Basic cluster installation](/01_Installation/Installation.md)

[2. Set up an external Load balancer (MetalLB)](/02_Setup%20a%20Load%20balancer/Setup_a_Load_balancer.md)
