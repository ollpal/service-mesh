https://istio.io/docs/setup/install/multicluster/gateways/

```bash
cd istio-1.4.6
```

# Deploy the Istio control plane in each cluster

```bash
bin/istioctl --context az manifest apply \
    -f install/kubernetes/operator/examples/multicluster/values-istio-multicluster-gateways.yaml

kubectl --context az create secret generic cacerts -n istio-system \
    --from-file=samples/certs/ca-cert.pem \
    --from-file=samples/certs/ca-key.pem \
    --from-file=samples/certs/root-cert.pem \
    --from-file=samples/certs/cert-chain.pem
```

# Setup DNS

```bash
export ISTIO_DNS_IP=$(kubectl --context az get svc -n istio-system istiocoredns -o jsonpath='{.spec.clusterIP}')

kubectl --context az apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  istio.server: |
    global:53 {
        errors
        cache 30
        forward . ${ISTIO_DNS_IP}:53
    }
EOF

kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
```

# Configure application services

## Deploy the sleep service in az

```bash
kubectl create --context az namespace ttt
kubectl label --context az namespace ttt istio-injection=enabled
kubectl apply --context az -n ttt -f samples/sleep/sleep.yaml

export SLEEP_POD=$(kubectl get --context az -n ttt pod -l app=sleep \
    -o jsonpath='{.items..metadata.name}')

echo $SLEEP_POD
```

## Export the cluster2 gateway address:

```bash
export CLUSTER2_GW_ADDR=$(kubectl get --context=aws-2 svc --selector=app=istio-ingressgateway \
    -n istio-system -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}')
echo $CLUSTER2_GW_ADDR
```

## Create a service entry for the httpbin service in az.

```bash
kubectl apply --context az -n ttt -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ttt
spec:
  hosts:
  # must be of form name.namespace.global
  - httpbin.bar.global
  # Treat remote cluster services as part of the service mesh
  # as all clusters in the service mesh share the same root of trust.
  location: MESH_INTERNAL
  ports:
  - name: http1
    number: 8000
    protocol: http
  resolution: DNS
  addresses:
  # the IP address to which httpbin.bar.global will resolve to
  # must be unique for each remote service, within a given cluster.
  # This address need not be routable. Traffic for this IP will be captured
  # by the sidecar and routed appropriately.
  - 240.0.0.4
  endpoints:
  # This is the routable address of the ingress gateway in cluster2 that
  # sits in front of sleep.foo service. Traffic from the sidecar will be
  # routed to this address.
  - address: ${CLUSTER2_GW_ADDR}
    ports:
      http1: 15443 # Do not change this port value
EOF
```

# Call httpbin

```bash
kubectl exec --context az $SLEEP_POD -n ttt -c sleep -- curl -I httpbin.bar.global:8000/headers
```
