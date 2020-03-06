# expose kiali from the cluster via a node port

    kubectl patch svc -n istio-system kiali --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

# get kiali port

    kubectl get svc kiali  -n istio-system  -o jsonpath='{ .spec.ports[0].nodePort }{"\n"}'

# use webpage

admin / admin to login (default)

webpage is $GATEWAY_URL:[kiali port]