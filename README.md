# Example with Argo Rollouts and HPA and Istio

* Argo Rollouts version  v1.7.1-CR-24605
* Istio 1.19

## Install Argo Rollouts

```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-
rollouts/releases/download/v1.7.1/install.yaml
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

## Deploy

```
cd manifests
kubectl apply -f .
```

Verify with

```
kubectl argo rollouts get rollout demo-ag-prod-primary
```