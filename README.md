# CICD for Kubeneretes
To follow the [CICD for K8s](https://lordpangan.com/2022/04/24/cicd-for-kubernetes/) write-up, here are the pre-requisites.
## Setup Cluster
Setups K8s cluster and install cluster apps.
```
### Clone the repo
git clone git@github.com:lordpangan/dev-k8s.git

### Setup the cluster
kind create cluster --name cluster1 --config simple-cluster.yaml

### Install ingress and argocd, wait for argocd to finish deployment
kubectl kustomize cluster-init/overlay/setup-ingress-argocd | k apply -f -
kubectl wait --namespace argocd --for=condition=ready pod --selector=app.kubernetes.io/name=argocd-server --timeout=90s

### Install argo-workflows and kubevela apps
kubectl apply -f cluster-init/cluster-apps/argo-workflow.yaml
kubectl apply -f cluster-init/cluster-apps/kubevela.yaml
kubectl apply -f cluster-init/cluster-apps/reloader.yaml
```

## Deploy sample-app
Next deploy the app that we are going to use for the CICD demo.
```
# Install the trait used for our app
kubectl apply -f apps/cluster-config.yaml
kubectl apply -f apps/qa-sample-app.yaml
kubectl apply -f apps/prod-sample-app.yaml
```

## Run CI Workflow and CD

### Add required Secrets
Add the regcred(docker secret) and git-token to argo namespace.
```
# Create 'git-token;' secret commiting to app repo
kubectl -n argo create secret generic git-token --from-literal=git-token=$GIT_TOKEN

# Create 'regcred' secret for pushing to docker repo
kubectl --namespace argo \
    create secret \
    docker-registry regcred \
    --docker-username=$DOCKER_USER \
    --docker-password=$DOCKER_USER \
    --docker-email=$$DOCKER_EMAIL
```

### Run the CI Workflow
```
argo -n argo submit workflows/ci.yaml
```

### Run the CD
```
argo -n argo submit workflows/cd.yaml
```