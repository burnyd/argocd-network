#Install kind

#Apply the ceos stuff

#Install the argocd application inside of the cluster
https://argo-cd.readthedocs.io/en/stable/getting_started/
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

#Install the argocd binary.
https://argo-cd.readthedocs.io/en/stable/cli_installation/

➜  argocd-network git:(master) which argocd
/usr/local/bin/argocd

# Create a portfoward proxy on your local machine if you want access to the gui
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

connect to the gui at 127.0.0.1:8080

# Get the initial password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
username: admin
password: generated password from aboce.

# Loginto the argo server.
argocd login 127.0.0.1:8080

# Create the argo app deployment.

argocd app create network-config --repo https://github.com/burnyd/argocd-network.git \
--path configs \
--dest-server https://kubernetes.default.svc \
--dest-namespace networkconfig \
--sync-policy automated