# Ingress-Nginx Setup

- [Installation Guide using Helm](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

- Replace [nginx-service-loadbalancer](/02_Setup%20a%20Load%20balancer/nginx-service-loadbalancer.yaml) with [nginx-service-for-ingress](/03_Setup%20an%20Ingress%20Controller/ingress-nginx/nginx-service-for-ingress.yaml) instead.

- Then apply [nginx-ingress-test](/03_Setup%20an%20Ingress%20Controller/ingress-nginx/nginx-ingress-test.yaml), using the nginx ingressClass that was automatically created when installing ingress-nginx using Helm.

  - [K8s Ingress docs](https://kubernetes.io/docs/concepts/services-networking/ingress/)

- Any device in the local network can now access the nginx test app by visiting `http://test.mydomain.local/`