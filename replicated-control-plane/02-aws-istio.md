https://istio.io/docs/setup/install/multicluster/gateways/

```bash
cd istio-1.4.6
```

# Deploy the Istio control plane in each cluster

```bash
bin/istioctl --context cluster-1 manifest apply \
    -f install/kubernetes/operator/examples/multicluster/values-istio-multicluster-gateways.yaml

kubectl --context cluster-1 create secret generic cacerts -n istio-system \
    --from-file=samples/certs/ca-cert.pem \
    --from-file=samples/certs/ca-key.pem \
    --from-file=samples/certs/root-cert.pem \
    --from-file=samples/certs/cert-chain.pem

bin/istioctl --context cluster-2 manifest apply \
    -f install/kubernetes/operator/examples/multicluster/values-istio-multicluster-gateways.yaml

kubectl --context cluster-2 create secret generic cacerts -n istio-system \
    --from-file=samples/certs/ca-cert.pem \
    --from-file=samples/certs/ca-key.pem \
    --from-file=samples/certs/root-cert.pem \
    --from-file=samples/certs/cert-chain.pem
```

# Setup DNS

```bash
kubectl --context cluster-1 get svc -n istio-system istiocoredns -o jsonpath='{.spec.clusterIP}'
```

```bash
kubectl --context cluster-1 edit -n kube-system configmap coredns
```

    global:53 {
        errors
        cache 30
        forward . <cluster-ip>:53
    }

# Configure application services

## Deploy the sleep service in cluster1.

```bash
kubectl create --context=cluster-1 namespace foo
kubectl label --context=cluster-1 namespace foo istio-injection=enabled
kubectl apply --context=cluster-1 -n foo -f samples/sleep/sleep.yaml

export SLEEP_POD=$(kubectl get --context=cluster-1 -n foo pod -l app=sleep \
    -o jsonpath='{.items..metadata.name}')

echo $SLEEP_POD
```

## Deploy the httpbin service in cluster2.

```bash
kubectl create --context=cluster-2 namespace bar
kubectl label --context=cluster-2 namespace bar istio-injection=enabled
kubectl apply --context=cluster-2 -n bar -f samples/httpbin/httpbin.yaml
```

## Export the cluster2 gateway address:

```bash
export CLUSTER2_GW_ADDR=$(kubectl get --context=cluster-2 svc --selector=app=istio-ingressgateway \
    -n istio-system -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}')
echo $CLUSTER2_GW_ADDR
```

## Create a service entry for the httpbin service in cluster1.

```bash
kubectl apply --context=cluster-1 -n foo -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-bar
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
  - 240.0.0.2
  endpoints:
  # This is the routable address of the ingress gateway in cluster2 that
  # sits in front of sleep.foo service. Traffic from the sidecar will be
  # routed to this address.
  - address: ${CLUSTER2_GW_ADDR}
    ports:
      http1: 15443 # Do not change this port value
EOF
```

# Version-aware routing to remote services

If the remote service has multiple versions, you can add labels to the service entry endpoints.

```bash
kubectl apply --context=cluster-1 -n foo -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-bar
spec:
  hosts:
  # must be of form name.namespace.global
  - httpbin.bar.global
  location: MESH_INTERNAL
  ports:
  - name: http1
    number: 8000
    protocol: http
  resolution: DNS
  addresses:
  # the IP address to which httpbin.bar.global will resolve to
  # must be unique for each service.
  - 240.0.0.2
  endpoints:
  - address: ${CLUSTER2_GW_ADDR}
    labels:
      cluster: cluster2
    ports:
      http1: 15443 # Do not change this port value
EOF
```

￼￼
You can then create virtual services and destination rules to define subsets of the
httpbin.bar.global service using the appropriate gateway label selectors. The instructions are the
same as those used for routing to a local service. See multicluster version routing for a complete
example.

https://istio.io/blog/2019/multicluster-version-routing/

# Version Routing in a Multicluster Service Mesh

## Deploy version v1 of the bookinfo application in cluster1

```bash
kubectl create --context=cluster-1 namespace bookinfo
kubectl label --context=cluster-1 namespace bookinfo istio-injection=enabled

kubectl apply --context=cluster-1 -n bookinfo -f bookinfo-productpage-v1.yaml
kubectl apply --context=cluster-1 -n bookinfo -f bookinfo-details-v1.yaml
kubectl apply --context=cluster-1 -n bookinfo -f bookinfo-reviews-v1.yaml
```

## Deploy bookinfo v2 and v3 services in cluster2

```bash
kubectl create --context=cluster-2 namespace bookinfo
kubectl label --context=cluster-2 namespace bookinfo istio-injection=enabled

kubectl apply --context=cluster-2 -n bookinfo -f bookinfo-ratings-v1.yaml
kubectl apply --context=cluster-2 -n bookinfo -f bookinfo-reviews-v2.yaml
kubectl apply --context=cluster-2 -n bookinfo -f bookinfo-reviews-v3.yaml
```

## Access the bookinfo application

Create the bookinfo gateway in cluster1:

```bash
kubectl apply --context=cluster-1 -n bookinfo \
    -f istio-1.4.6/samples/bookinfo/networking/bookinfo-gateway.yaml

kubectl --context cluster-1 -n bookinfo get gateway
```

Determine the ingress IP and port

```bash
export INGRESS_HOST=$(kubectl --context cluster-1 -n istio-system \
    get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
export INGRESS_PORT=$(kubectl --context cluster-1 -n istio-system \
    get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl --context cluster-1 -n istio-system \
    get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')

echo $INGRESS_HOST $INGRESS_PORT $SECURE_INGRESS_PORT

export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
echo $GATEWAY_URL
```

Point your browser to http://$GATEWAY_URL/productpage.

```bash
curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
```

You should see the productpage with reviews, but without ratings, because only v1 of the reviews
service is running on cluster-1 and we have not yet configured access to cluster-2.

## Create a service entry and destination rule on cluster1 for the remote reviews service

As described in the
[setup instructions](https://istio.io/docs/setup/install/multicluster/gateways/#setup-dns), remote
services are accessed with a .global DNS name. In our case, it’s reviews.default.global, so we
need to create a service entry and destination rule for that host. The service entry will use the
cluster2 gateway as the endpoint address to access the service. You can use the gateway’s DNS name,
if it has one, or its public IP, like this

```bash
export CLUSTER2_GW_ADDR=$(kubectl --context=cluster-2 -n istio-system \
    get svc --selector=app=istio-ingressgateway \
        -o jsonpath="{.items[0].status.loadBalancer.ingress[0].hostname}")

echo $CLUSTER2_GW_ADDR

kubectl apply --context=cluster-1 -n bookinfo \
    -f <(sed "s/\${\?CLUSTER2_GW_ADDR}\?/$CLUSTER2_GW_ADDR/" bookinfo-reviews-serviceentry.yaml)
kubectl apply --context=cluster-1 -n bookinfo \
    -f bookinfo-reviews-destinationrule-global.yaml
```

The address 240.0.0.3 of the service entry can be any arbitrary unallocated IP. Using an IP
from the class E addresses range 240.0.0.0/4 is a good choice. Check out the
[gateway-connected multicluster](https://istio.io/docs/setup/install/multicluster/gateways/#configure-the-example-services)
example for more details.

Note that the labels of the subsets in the destination rule map to the service entry endpoint
label (cluster: cluster2) corresponding to the cluster2 gateway. Once the request reaches the
destination cluster, a local destination rule will be used to identify the actual pod labels
(version: v1 or version: v2) corresponding to the requested subset.

## Create a destination rule on both clusters for the local reviews service

```bash
kubectl apply --context=cluster-1 -n bookinfo -f bookinfo-reviews-destinationrule-cluster-1.yaml
kubectl apply --context=cluster-2 -n bookinfo -f bookinfo-reviews-destinationrule-cluster-2.yaml
```

## Create a virtual service to route reviews service traffic

At this point, all calls to the reviews service will go to the local reviews pods (v1) because
if you look at the source code you will see that the productpage implementation is simply making
requests to http://reviews:9080 (which expands to host reviews.default.svc.cluster.local, the
local version of the service. The corresponding remote service is named reviews.default.global,
so route rules are needed to redirect requests to the global host.

Note that if all of the versions of the reviews service were remote, so there is no local
reviews service defined, the DNS would resolve reviews directly to reviews.default.global. In
that case we could call the remote reviews service without any route rules.

Apply the following virtual service to direct traffic for user jason to reviews versions v2 and
v3 (50/50) which are running on cluster2. Traffic for any other user will go to reviews version
v1.

```bash
kubectl apply --context cluster-1 -n bookinfo -f bookinfo-reviews-virtualservice.yaml
```

# Summary

In this article, we’ve seen how to use Istio route rules to distribute the versions of a service
across clusters in a multicluster service mesh with a replicated control plane model. In this
example, we manually configured the .global service entry and destination rules needed to provide
connectivity to one remote service, reviews. In general, however, if we wanted to enable any
service to run either locally or remotely, we would need to create .global resources for every
service. Fortunately, this process could be automated and likely will be in a future Istio
release.
