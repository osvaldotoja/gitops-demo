# gitops-demo

Using ArgoCD in a declarative way

# HOWTO

Create a cluster using kind.

```sh
kind create cluster --name gitops-demo
```

Install argocd, declare a private repository alongside a private ssh key.

```
kubectl create namespace argocd
kustomize build argocd/setup | kubectl apply -n argocd  -f -
```


Wait for argocd to be up and running.

```sh
ARGOCDSERVER_POD=$(kubectl get pod -n argocd  -o custom-columns=:metadata.name | grep argocd-server)
kubectl wait pod/${ARGOCDSERVER_POD} -n argocd --for condition=Ready --timeout=120s
```

Deploy the gitops repository.

```sh
kustomize build apps/staging | kubectl apply -f -
```


# Using GitOps

Check the deployment

```
$ kubectl describe deployment staging-nginx-deployment | grep Image
    Image:        nginx:1.7.9
```

Make a change on the repository. For example, update to a new nginx version.

```sh
# linux
sed -i 's|1.7.9|1.16.1|' addons/base/nginx/deployment.yaml

# macos sed ...
sed -i.bk 's|1.7.9|1.16.1|' addons/base/nginx/deployment.yaml
rm addons/base/nginx/deployment.yaml.bk
```


Commit, push and wait for changes to be applied onto the cluster

```sh
kubectl get pods -w
```

Expected output:

```sh
$ kubectl describe deployment staging-nginx-deployment | grep Image
    Image:        nginx:1.16.1
```

# clean up

```sh
kind delete cluster --name gitops-demo
```