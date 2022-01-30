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
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address 0.0.0.0 &
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

![Alt text](/images/initial-login.jpg?raw=true "initial")


## Create the argo app deployment.

```
argocd app create network-config --repo https://github.com/burnyd/argocd-network.git \
--path configs \
--dest-server https://kubernetes.default.svc \
--dest-namespace networkconfig \
--sync-policy automated
```

At this point we can see that Argo is monitoring our deployment of the ceos opperator and ready to deploy if there is a change!

![Alt text](/images/init-app.jpg?raw=true "initial-app")

![Alt text](/images/init-overview.jpg?raw=true "initial-overview")

The interesting part here as well is that we can see that argo is effectively watching https://github.com/burnyd/argocd-network.git with every text/kubernetes yaml within the configs directory as the argocd application is supposed to do.

So at this point we can go ahead and make a small change to configs/ceos1.yaml and add vlans to it.

Current vlans

```
!
vlan 100,200,300,400,500,600
!
```

```
ceos1#show vlan brief
VLAN  Name                             Status    Ports
----- -------------------------------- --------- -------------------------------
1     default                          active
100   VLAN0100                         active
200   VLAN0200                         active
300   VLAN0300                         active
400   VLAN0400                         active
500   VLAN0500                         active
600   VLAN0600                         active
```

Changing the config at configs/ceos1.yaml and configs/ceos2.yaml to add vlans.

```
vlan 100,200,300,400,500,600,111,222,333
```


We can see within argo it pushes it directly to the kubernetes cluster object netdevs in our scenario.

![Alt text](/images/push.jpg?raw=true "push")

Now the most interesting of all.. I did not have to touch any comand lines, apis or toolings.  This happened entirely out of git!  So within the argo sync message fb1c45e is my commit id.  It matches up with what git has.

![Alt text](/images/git-app.jpg?raw=true "git-app")

Checking the switches / ceos we can see that they did infact get the vlans from the operator.

```
ceos1#show vlan brief
VLAN  Name                             Status    Ports
----- -------------------------------- --------- -------------------------------
1     default                          active
100   VLAN0100                         active
111   VLAN0111                         active
200   VLAN0200                         active
222   VLAN0222                         active
300   VLAN0300                         active
333   VLAN0333                         active
400   VLAN0400                         active
500   VLAN0500                         active
600   VLAN0600                         active
```