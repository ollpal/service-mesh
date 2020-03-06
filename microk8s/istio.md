
# enable istio 

    microk8s.enable istio

# select mTLS or not

When prompted, choose whether to enforce mutual TLS authentication among sidecars. If you have a mixed
deployment with non-Istio and Istio enabled services or you’re unsure, choose No.

# enable auto injection of the istio side car

    microk8s.kubectl label namespace default istio-injection=enabled
