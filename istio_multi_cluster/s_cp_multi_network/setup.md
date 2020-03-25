## Shared Control Plane with multiple networks

### Istio release version

istio-1.5-alpha.3dfe32d6be43ec36cceb3d58f9a50ec427d5e57f

### Create two or more K8s clusters

I am creating GKE clusters by following this Platform setup instruction: https://istio.io/docs/setup/platform-setup/gke/

```bash
kubectl config get-contexts
CURRENT   NAME                                                CLUSTER                                             AUTHINFO                                            NAMESPACE
          gke_carolyntestprj-262400_us-central1-c_cluster-1   gke_carolyntestprj-262400_us-central1-c_cluster-1   gke_carolyntestprj-262400_us-central1-c_cluster-1
*         gke_carolyntestprj-262400_us-central1-c_cluster-2   gke_carolyntestprj-262400_us-central1-c_cluster-2   gke_carolyntestprj-262400_us-central1-c_cluster-2
```

### Preparation

- Certificate Authority

Install the certificates in each cluster:

```bash
kubectl create namespace istio-system
kubectl create secret generic cacerts -n istio-system \
    --from-file=samples/certs/ca-cert.pem \
    --from-file=samples/certs/ca-key.pem \
    --from-file=samples/certs/root-cert.pem \
    --from-file=samples/certs/cert-chain.pem
```

- Cross-cluster control plane access

Used `istio-ingressgateway` gateway shared with data traffic.

- Cluster & Network naming

Set cluster context:
```bash
$ export MAIN_CLUSTER_CTX=$(kubectl config view -o jsonpath='{.contexts[0].name}')
$ export REMOTE_CLUSTER_CTX=$(kubectl config view -o jsonpath='{.contexts[1].name}')
```

Set cluster name:
```bash
$ export MAIN_CLUSTER_NAME=main0
$ export REMOTE_CLUSTER_NAME=remote0
```

Set cluster network:

```bash
$ export MAIN_CLUSTER_NETWORK=network1
$ export REMOTE_CLUSTER_NETWORK=network2
```

### Deployment

#### Main cluster

Create main cluster's config: istio-main-cluster.yaml


```bash
istioctl --context=${MAIN_CLUSTER_CTX} manifest apply -f istio-main-cluster.yaml
```

Verify pods are running:
```bash
kubectl --context=${MAIN_CLUSTER_CTX} -n istio-system get pod
NAME                                   READY   STATUS    RESTARTS   AGE
istio-ingressgateway-7ffb6d8fb-sbpsz   1/1     Running   0          5m46s
istiod-fc46bdb6-9qzrm                  1/1     Running   0          6m4s
prometheus-cc8fc4f6b-l6j9d             2/2     Running   0          5m46s
```

Set remote endpoint to access main cluster:

```bash
export ISTIOD_REMOTE_EP=$(kubectl --context=${MAIN_CLUSTER_CTX} -n istio-system get svc istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

#### Remote cluster

Create remote cluster's config: istio-remote0-cluster.yaml

```bash
istioctl --context ${REMOTE_CLUSTER_CTX} manifest apply -f istio-remote0-cluster.yaml
```

Verify pods are running:

```bash
kubectl --context=${REMOTE_CLUSTER_CTX} -n istio-system get pod
```

### Cross-cluster LB

- Config ingress gateways

```bash
kubectl --context=${MAIN_CLUSTER_CTX} apply -f cluster-aware-gateway.yaml
kubectl --context=${REMOTE_CLUSTER_CTX} apply -f cluster-aware-gateway.yaml
```

- Config cross-cluster service registries

```bash
istioctl x create-remote-secret --context=${REMOTE_CLUSTER_CTX} --name ${REMOTE_CLUSTER_NAME} | \
kubectl apply -f - --context=${MAIN_CLUSTER_CTX}
```



