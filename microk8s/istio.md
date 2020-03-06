
# Settup istio

Instructions to setup MicroK8s for use with Istio.

    microk8s.enable istio

When prompted, choose whether to enforce mutual TLS authentication among sidecars. If you have a mixed
deployment with non-Istio and Istio enabled services or you’re unsure, choose No.

    https://istio.io/docs/setup/platform-setup/

    https://istio.io/docs/setup/install/istioctl/
    https://istio.io/docs/setup/install/multicluster/
    https://istio.io/docs/setup/additional-setup/config-profiles/
    https://istio.io/docs/setup/additional-setup/sidecar-injection/
    https://istio.io/docs/setup/additional-setup/cni/

    kubectl label namespace default istio-injection=enabled
