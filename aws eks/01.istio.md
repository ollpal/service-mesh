Info from https://aws.amazon.com/blogs/opensource/getting-started-istio-eks/

    export ISTIO_VERSION=1.5.0
    curl -L https://git.io/getLatestIstio | sh -
    cd istio-1.5.0

    bin/istioctl verify-install

This command installs the default profile on the cluster defined by your Kubernetes configuration. The default profile is a good starting point for establishing a production environment, unlike the larger demo profile that is intended for evaluating a broad set of Istio features.

    bin/istioctl manifest apply

    kubectl label namespace default istio-injection=enabled

Sample app

    kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml


    export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
    export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

browse to http://$GATEWAY_URL/productpage

    kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
    kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
    kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml