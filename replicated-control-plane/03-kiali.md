# Visualizing Your Mesh

https://istio.io/docs/tasks/observability/kiali/

## Create a secret

```bash
KIALI_USERNAME=$(read '?Kiali Username: ' uval && echo -n $uval | base64)
KIALI_PASSPHRASE=$(read -s "?Kiali Passphrase: " pval && echo -n $pval | base64); echo

kubectl --context cluster-1 create namespace istio-system

cat <<EOF | kubectl --context cluster-1 apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

## Install kiali

```bash
istio-1.4.6/bin/istioctl --context cluster-1 manifest apply --set values.kiali.enabled=true
istio-1.4.6/bin/istioctl --context cluster-1 dashboard kiali
```

## Delete kiali

```bash
kubectl --context cluster-1 -n istio-system \
    delete all,secrets,sa,configmaps,deployments,ingresses,clusterroles,clusterrolebindings,customresourcedefinitions \
        --selector=app=kiali
```
￼￼￼
