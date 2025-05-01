# Setup Cert Manager and a LetsEncrypt Wildcard Certificate using DNS challenge (dns01)

## Pre-Requisites

- A K8s cluster with a working ingress controller, like [ingress nginx](/03_Setup%20an%20Ingress%20Controller/ingress-nginx/Ingress-Nginx_Setup.md)
- A domain name registered on Cloudflare, or at least pointing to Cloudflare nameservers.
- A cloudflare API token, used to enable cert manager to make DNS entry changes on our domain. We need this in order to [prove ownership of our domain](https://cert-manager.io/docs/tutorials/acme/dns-validation/).

  - The API token can be generated on your Cloudflare Profile page.
  - [Cert manager docs](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/#api-tokens) specifies the required permissions we need to set on our API token.

- A local DNS server with a record resolving the domain name, and all its subdomains, to the external IP address of our ingress controller loadbalancer service (in my case 192.168.0.89). For example:

  - example.domain > 192.168.0.89
  - *.example.domain > 192.168.0.89

### Wildcard DNS record when using PI-Hole as a local DNS

[See this reddit post](https://www.reddit.com/r/selfhosted/comments/1eenxzp/wildcard_dns_record_in_pihole/)

- To avoid having to create a DNS record for every single subdomain we want to be able to resolve on our local network, we can log into the PI-Hole machine and create a file in `/etc/dnsmasq.d/[filename].conf`. Inside this file we simply add a wildcard entry, for example: `address=/example.domain/192.168.0.89`. This will resolve any subdomain, such as `test.example.domain` to our ingress service loadbalancer IP.

## Setup Cert Manager

- Install it using Helm by following the [cert manager documentation](https://cert-manager.io/docs/installation/helm/#installing-with-helm).

### Create a ClusterIssuer

- In our K8s cluster, we need an [Issuer/ClusterIssuer](https://cert-manager.io/docs/concepts/issuer/) resource that will represent a certificate authority (CA), such as Let's Encrypt, which is able to generate the certificates.

- In order to use the dns01 challenge method to prove ownership of our domain, we also need to specify a K8s secret containing our Cloudflare API token. So, we start by creating the secret:

  - Since I want to create the secret using a manifest, I first need to [base64 encode the token](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-config-file/#create-the-config-file). Running Linux, I used the command: `echo -n 'my-token-value' | base64 -w 0`.

  - Then we insert the output in [our manifest](/04_Setup%20cert%20manager/token-secret.yaml) and apply it in our cluster. In the manifest we also [make sure](https://github.com/cert-manager/cert-manager/issues/263) to put the secret in the same namespace where cert manager is running to enable our ClusterIssuer resource to find it.

  - Create the secret: `kubectl apply -f token-secret.yaml`.

  - Having done this, we now can refer to our token secret in our [ClusterIssuer manifest](/04_Setup%20cert%20manager/cluster-issuer.yaml), and then create it: `kubectl apply -f cluster-issuer.yaml`.

  - We now have our ClusterIssuer ready to use.

### Create a wildcard certificate

- To request a wildcard certificate (a certificate that will work on any subdomain for our domain name), we define a manifest for a [Certificate K8s resource](/04_Setup%20cert%20manager/wildcard-certificate.yaml).

- And create it: `kubectl apply -f wildcard-certificate.yaml`.

- The certificate status can be viewed using: `kubectl get certificate -n default`.

## Use the Certificate in our Ingress

- Having the certificate created, we can use it in our [Ingress resource](/04_Setup%20cert%20manager/nginx-ingress.yaml), referring to our Certificate secret named in our [Certificate manifest](/04_Setup%20cert%20manager/wildcard-certificate.yaml) (secretName).

  - Notice I now put the Ingress in the default namespace since that's where I put the Certificate resource. I also moved the rest of my nginx test deployment to the default namespace.

- And finally we create the ingress: `kubectl apply -f nginx-ingress.yaml`.

- If everything works, from any device on our local network we should now be able to browse to https://nginx-test.example.domain and see that the connection is secure, using a certificate that is verified by Let's Encrypt!
