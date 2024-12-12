# Example with Argo Rollouts and HPA and Istio

* Argo Rollouts version  v1.7.1-CR-24605
* Istio 1.19

This repository replicates Argo Rollout issues [2507](https://github.com/argoproj/argo-rollouts/issues/2507) and [3681](https://github.com/argoproj/argo-rollouts/issues/3681)


## Get a Kubernetes cluster

[Create a K8s cluster](https://istio.io/latest/docs/setup/platform-setup/gke/)

Example for GKE

```
export PROJECT_ID=`gcloud config get-value project` && \
  export M_TYPE=n1-standard-2 && \
  export ZONE=us-west2-a && \
  export CLUSTER_NAME=${PROJECT_ID}-${RANDOM} && \
  gcloud services enable container.googleapis.com && \
  gcloud container clusters create $CLUSTER_NAME \
  --cluster-version latest \
  --machine-type=$M_TYPE \
  --num-nodes 4 \
  --zone $ZONE \
  --project $PROJECT_ID
```

## Install Argo Rollouts

```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/download/v1.7.1/install.yaml
```

## Install Istio 

Install Istio [with istioctl](https://cloud.google.com/kubernetes-engine/docs/tutorials/secure-services-istio)

```
export ISTIO_VERSION=1.19.0
curl -L https://istio.io/downloadIstio | TARGET_ARCH=$(uname -m) sh -
cd istio-1.19.0/
export PATH=$PWD/bin:$PATH
istioctl install --set profile="default" -y
watch kubectl get pods -n istio-system
```


## Test Istio

```
kubectl label namespace default istio-injection=enabled
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/tags/1.19.0/samples/httpbin/httpbin.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/refs/tags/1.19.0/samples/httpbin/httpbin-gateway.yaml
```

Find ingress host and port

```
export INGRESS_NAME=istio-ingressgateway
export INGRESS_NS=istio-system
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

Show loadBalancer URL
```
echo $INGRESS_HOST:$INGRESS_PORT
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

Visit $INGRESS_HOST:$INGRESS_PORT in your browser to look at the Rollout

Promote with 

```
kubectl argo rollouts promote demo-ag-prod-primary
```

Promote a second time and you will see 503 errors in the application


## Workaround 

The problem doesn't seem to exist if you use [host level splitting](https://argo-rollouts.readthedocs.io/en/stable/features/traffic-management/istio/#host-level-traffic-splitting)
The same example as found in the [workaround](workaround) folder works just file

## Proper solution

The downtime cause is the combinations of using destination rule labels
that are also part of [ephemeral metadata](https://argo-rollouts.readthedocs.io/en/stable/features/ephemeral-metadata/). 

The folder [solution](solution) is the same example and it works without downtime



