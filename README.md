# Example with Argo Rollouts and HPA and Istio

* Argo Rollouts version  v1.7.1-CR-24605
* Istio 1.19

## Install Argo Rollouts

```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/download/v1.7.1/install.yaml
```

## Install Istio 

```
helm repo add istio https://istio-release.storage.googleapis.com/ch
arts
helm repo update
```

```
kubectl create namespace istio-system
helm install istio-base istio/base -n istio-system --version 1.19.0
helm install istiod istio/istiod -n istio-system --version 1.19.0 --wait
```

```
kubectl create namespace istio-ingress
helm install istio-ingress istio/gateway -n istio-ingress --version 1.19.0 --wait
```

## Test Istio

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/tags/1.19.0/samples/httpbin/httpbin.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/tags/1.19.0/samples/httpbin/httpbin-gateway.yaml
```

Find ingress host and port

```
export INGRESS_NAME=istio-ingress
export INGRESS_NS=istio-ingress
export INGRESS_HOST=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
export TCP_INGRESS_PORT=$(kubectl -n "$INGRESS_NS" get service "$INGRESS_NAME" -o jsonpath='{.spec.ports[?(@.name=="tcp")].port}')
```

Test application access

```
curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST:$INGRESS_PORT/status/200"
curl -s -I -HHost:httpbin.example.com "http://$INGRESS_HOST:$INGRESS_PORT/headers"
```

## Deploy rollout

```
cd manifests
kubectl apply -f .
```

Verify with

```
kubectl argo rollouts get rollout demo-ag-prod-primary

```




