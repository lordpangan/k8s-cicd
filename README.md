# CICD for Kubeneretes

## Setup Cluster
To follow our CICD for K8s write-up here are the requirements.
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
```

## Deploy sample-app
Next deploy the app that we are going to use for the CICD demo.
```
### Install the trait used for our app
kubectl apply -f https://raw.githubusercontent.com/lordpangan/kubevela-manifests/main/traits/gateway-ing-url-rewrite.yaml
### Install the sample-app
kubectl apply -f apps/sample-app.yaml
```

## Run CI Workflow
Add the regcred(docker secret) and git-token to argo namespace.
```
### Create 'regcred' secret for pushing to docker repo
kubectl --namespace argo \
    create secret \
    docker-registry regcred \
    --docker-username=$DOCKER_USER \
    --docker-password=$DOCKER_USER \
    --docker-email=$$DOCKER_EMAIL

### Create 'git-token;' secret commiting to app repo
kubectl -n argo create secret generic git-token --from-literal=git-token=$GIT_TOKEN
```

Run the CI Workflow
```
argo -n argo submit workflows/ci.yaml
```

