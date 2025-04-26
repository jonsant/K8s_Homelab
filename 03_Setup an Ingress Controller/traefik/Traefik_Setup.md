# Traefik Setup

## Install Traefik

### Install Helm

- I wanted to use the Helm 'packet manager' to simplify the installation, so I first followed the Helm documentation to install it [via script](https://helm.sh/docs/intro/install/#from-script).

### Install Traefik using Helm chart

- Then to install Traefik I simply followed [Traefik's documentation](https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart) to first add the Traefik repo to Helm, update it, and finally installing Traefik.

- I noticed the Dashboard was not enabled by default (couldn't reach it on localhost when port fortwarding to the the Traefik pod), so I created this [traefik-values.yaml](/03_Setup%20an%20Ingress%20Controller/traefik-values.yaml) file and executed `helm upgrade --install traefik traefik/traefik -f fix-traefik-dashboard.yaml`. All this after reading [this github issue](https://github.com/traefik/traefik-helm-chart/issues/85).
  - Note, however that the port has since been changed to `8080` instead of 9000, as shown in [this example from Traefik](https://github.com/traefik/traefik-helm-chart/blob/master/EXAMPLES.md). We can also see this locally by 'describing' our traefik pod: `kubectl describe pod traefik-xyz`

- In order to test reaching the dashboard, I had to port forward using the address=0.0.0.0 argument to allow me to accessing the dashboard from a web browser from a client on my local network (outside of the cluster network), in my case I did: `kubectl port-forward traefik-xyz --address=0.0.0.0 8080:8080`

  - Then, from a client on my home network, I visited this URL: `http://192.168.0.12:8080/dashboard/#/` and was able to see the dashboard.
