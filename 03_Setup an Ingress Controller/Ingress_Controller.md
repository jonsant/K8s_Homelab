# Ingress

## What and why?

- Exposing our services to the outside of the cluster can be done using NodePort and LoadBalancer. But if we need multiple services in our cluster, and want to be able to route incoming requests to the service using the request path, we can use an Ingress. The Ingress will sit in front of multiple services and expose them under the same IP address. While the LoadBalancer operates at OSI layer 4 (transport), the Ingress operates at layer 7 (application) which allows for more intelligent routing and load balancing decisions.

## Ingress vs Service

- Ingress is a separate K8s resource which allows for things such as load balancing, routing based on HTTP/HTTPS level, SSL/TLS termination. Again, the Ingress will sit in front of our K8s services.

- For our Ingress to work, our cluster needs to be configured with an Ingress Controller.

- Read more [here](https://kubernetes.io/docs/concepts/services-networking/ingress/), [here](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/), and [here](https://www.cortex.io/post/understanding-kubernetes-services-ingress-networking)

