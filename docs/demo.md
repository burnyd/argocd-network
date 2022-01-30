# Demo

## Preamble

This is largely based off of the [network-configuration kubernetes operator](https://github.com/burnyd/k8s-network-config-operator)

## Create the kind cluster or use your own.
```
kind create cluster
```

## Apply the k8s-network-config-operator operator into the cluster

```
kubectl apply -k https://github.com/burnyd/k8s-network-config-operator/manifests/.
```

## Apply the two ceos devices I have with the cluster.
```
kubectl apply -f https://raw.githubusercontent.com/burnyd/k8s-network-config-operator/master/manifests/ceos/ceos1.yaml

kubectl apply -f https://raw.githubusercontent.com/burnyd/k8s-network-config-operator/master/manifests/ceos/ceos2.yaml
```

## Install the argocd application inside of the cluster
The ArgoCD install needs to be installed in the cluster I tested this with kind this can be installed on a very minimal environment https://argo-cd.readthedocs.io/en/stable/getting_started/

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Install the argocd binary.
https://argo-cd.readthedocs.io/en/stable/cli_installation/
```
➜  argocd-network git:(master) which argocd
/usr/local/bin/argocd
```

## Create a portfoward proxy on your local machine if you want access to the gui

This all allow for any connections on the localhost to 8080 to forward to the kubernetes service for argocd.
```
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```

connect to the gui at 127.0.0.1:8080

## Get the initial password

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

username: admin
password: generated password from above.

## Loginto the argo server so any argocd binary requests to talk to the application.

```
argocd login 127.0.0.1:8080
```

## Create the argo app deployment.

```
argocd app create network-config --repo https://github.com/burnyd/argocd-network.git \
--path configs \
--dest-server https://kubernetes.default.svc \
--dest-namespace networkconfig \
--sync-policy automated
```