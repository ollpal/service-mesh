    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.4/samples/bookinfo/platform/kube/bookinfo.yaml

    kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"

    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.4/samples/bookinfo/networking/bookinfo-gateway.yaml

    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
    export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
￼￼￼
    export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.4/samples/bookinfo/networking/destination-rule-all-mtls.yaml
