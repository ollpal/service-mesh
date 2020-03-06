# expose kiali 
kubectl patch svc -n istio-system kiali --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

# get kiali port
kubectl get svc kiali  -n istio-system  -o jsonpath='{ .spec.ports[0].nodePort }{"\n"}'

# admin / admin to login (default)