# Using ArgoCD as network configuration deployments.

## Preamble

This is largely based off of the [network-configuration kubernetes operator](https://github.com/burnyd/k8s-network-config-operator)

The demo can be found within the [docs/demo.md](https://github.com/burnyd/argocd-network/blob/master/docs/demo.md) directory

Last week I read the [GitOps and Kubernetes book](https://www.manning.com/books/gitops-and-kubernetes) which is a pretty amazing book detailing the necessity for Gitops.  What is Gitops you ask?  The 1000ft view of Gitops is driving everything out of git.  When I say driving everything for an application out of git the idea is that you would make an application change or a change to anything and a product like [Argocd](https://argoproj.github.io/cd/) would then apply this to the kubernetes cluster without the devops engineer or application owner having to really do anything.  Weaveworks coined gitops first in 2018.

![Alt text](/images/gitops.jpg?raw=true "gitops")

This is nice because a use can pick and choose their own CI platform, unit tests etc.  Argo or other gitops platforms(flux, Jenkins x etc) can then worry about only pushing to the device which resides in the kubernetes cluster. Argocd its self for example is a operator.  It will continuously monitor a directory which will reside in a public git repo for kubernetes manifests.  Once there is a change / commit within that repo specifically within the folder in which argocd is following it will then either in a automated fashion or manual depending on the sync preferences within argo will then apply to the kubernetes cluster.  All of this can be used within the [argocd getting started guide.](https://argo-cd.readthedocs.io/en/stable/getting_started/).  Git because your single source of truth.  Git at this point is where all of your state is driven out of.  So you would generally have full deployments as well as when state has been changed.  EVERYTHING goes through git.

So this sounds nice how can we apply this to networking?  So in another repo there is a kubernetes config operator which the user can specify the way the network configuration looks and then the operator will simply push the config to the devices.  This is called the [k8s-network-config-operator](https://github.com/burnyd/k8s-network-config-operator) the general purposes of this is to take this concept and anytime a change happens within the configs/ directory and then push out the changes directly to the repo which argocd turns around and applies it to the kubernetes cluster which the operator apllies directly to the switches!

![Alt text](/images/argopush.jpg?raw=true "argo")

The idea here is that a change was made to the configs/ceos1.yaml file where vlans were added.  The devops / network engineer went ahead and either created a pull request or commit to the main branch to add vlans 111,222,333.  This wuold all reside in the repo.  Now argocd would pick this change up and apply it to the kubernetes api for this object which the operator tracks.  The operator notices that there is a change in the configurationa and pushes the change out to ceos1.  Argo can operate in the mode of automatic or sync mode where a human being can tell it to push or auto push on sync.

This makes it pretty attractive for most people in the networking space because you can decide which to push, when to push , git become your lifeline of all state which you have commit history to and lastly given that the operator will overwrite any other changes that are not through kubernetes git is always going to be what is the current running specification.
