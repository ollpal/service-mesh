
    kubectl apply -f dns.yaml
    kubectl apply -f proxy.yaml

Test with

    export SOURCE_POD=$(microk8s.kubectl get pod -l app=sleep -o jsonpath=\{.items..metadata.name\})
    microk8s.kubectl exec $SOURCE_POD -c sleep -i -t -- sh -c "HTTPS_PROXY=$HTTPS_PROXY HTTP_PROXY=$HTTP_PROXY curl https://www.google.com"
