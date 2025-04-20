# External Load Balancer

## Simple test app

- I first deployed a simple [nginx test app deployment](/02_Setup%20a%20Load%20balancer/nginx-test-app.yaml) to be able to test the different service types.

- After having the app up and running I applied a [simple service](/02_Setup%20a%20Load%20balancer/nginx-service.yaml), without specifying a type which defaults it to a ClusterIP service. I then tested the port-forward command to temporarily expose the service on the node: `kubectl port-forward service/nginx-test-service 8080:80 -n nginx-test`.

- To instead expose the service on a given port on all nodes, I deleted the previous service and applied the [NodePort service](/02_Setup%20a%20Load%20balancer/nginx-service-nodeport.yaml). This makes the nginx test app available to devices in the local network by going to the node IP and specifying the nodePort configured in the service yaml file, for example: `192.168.0.90:30005`.

## Service types summary

[k8s docs - Service type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

- Not specifying a service type in our service manifests will default it to a ClusterIP.
This type is only accessible from within the cluster, and we'd have to expose it to the outside using an Ingress or Gateway.
- The NodePort type can be used to expose a service to the outside on a static port on each node's IP.
- LoadBalancer - this type will expose the service by using an external load balancer, which must be provided either by a cloud provider or manually by us, since Kubernetes does not include a built-in load balancer.
For this reason, when applying a LoadBalancer service such as our [LoadBalancer example](/02_Setup%20a%20Load%20balancer/nginx-service-loadbalancer.yaml), its EXTERNAL-IP will remain at \<pending\> until an external Load balancer, such as MetalLB, is configured.
  - See service status using the command `kubectl get svc -n nginx-test` (we need to specify the namespace since the service is not in the default one)

## External Load Balancer setup

### Installing MetalLB

[MetalLB docs - Installation](https://metallb.universe.tf/installation/)
[MetalLB docs - Configuration](https://metallb.universe.tf/configuration/)

1. Apply manifest: `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml`

2. We then must give MetalLB an IP address pool from which to take addresses and assign to our LoadBalancer services. We do this by applying an IPAddressPool such as [this one](/02_Setup%20a%20Load%20balancer/metalLB-ip-pool.yaml).

3. Next, we tell MetalLB how we want it to announce the IP addresses that it has assigned to our services. In my case I did it the easiest way - by applying an L2Advertisement manifest, that is associated to the IPAddressPool we created above. This will make MetalLB simply respond to ARP requests on the local network, giving clients the machine's MAC address.
So, apply [metalLB-ip-advertisement.yaml](/02_Setup%20a%20Load%20balancer/metalLB-ip-advertisement.yaml/).

4. We now have a working external load balancer configured.

### Troubleshooting

- I initially didn't get the L2 advertisement to work, and experimented with different configuration changes, such as

  - echo "net.ipv4.conf.all.arp_ignore=1" | sudo tee -a /etc/sysctl.conf

  - echo "net.ipv4.conf.all.arp_announce=2" | sudo tee -a /etc/sysctl.conf

  - echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter

- I later changed these values back to default

- The thing that seems to have been the issue was that I needed to enable promiscuous mode on the network interface since I'm using WiFi on my raspberry Pi's.

  - I made promiscuous mode persistent by adding a [.service](/02_Setup%20a%20Load%20balancer/promisc-on-start.service) file in `/etc/systemd/system` (see solution #1 [here](https://superuser.com/questions/1804774/persistent-promiscuous-mode-in-debian-12)).

- For help with troubleshooting I used [metallb troubleshooting docs](https://metallb.universe.tf/troubleshooting/#using-wifi-and-cant-reach-the-service)
