# Development Enviroment

The minimal tools required for creating a development environment

Kubernetes Cluster, microK8s has been used for such purpouse.
Helm, for installing required tools in the cluster not supported by microk8s
ArgoCD, as gitops supporting tool

## macOs

### Steps

#### Required tools

*kubectl*

brew install kubernetes-cli

*helm*

brew install helm


#### Install k8s

https://ubuntu.com/tutorials/install-microk8s-on-mac-os#1-overview

https://microk8s.io/docs/addons

Install microk8s using homebrew

brew install ubuntu/microk8s/microk8s

Installed microk8s will create a VM for microk8s using multipass

To create a microk8s cluster (one node)

microk8s install --channel 1.21 --cpu  --mem --disk

microk8s status --wait-ready

Once installed to verity installation

multipass list

multipass info microk8s-vm

Install required features:
microk8s enable 

microk8s allows to execute kubctl commands in a very similar way

microk8s kubectl cluster-info

For developing purpouses is better using kubectl client. In order to fullfil this only is required to generate the required config file for kubectl

microk8s config > $HOME/.kube/config

Checking configuration

kubectl cluster-info


#### Install argoCD

helm install argo-cd --create-namespace --namespace argo-cd --values values.yaml argo/argo-cd --debug --dry-run


NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argo-cd-argocd-server -n argo-cd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-1-ssl-passthrough
      - Add the `--insecure` flag to `server.extraArgs` in the values file and terminate SSL at your ingress: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argo-cd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

https://www.arthurkoziel.com/setting-up-argocd-with-helm/

Dashboard

kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token

token=$(kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)

#### Install kubeedge

Download keadmin 

https://github.com/kubeedge/kubeedge/releases

Install


keadm init --advertise-address="192.168.1.50"

Status /var/log/kubeedge/cloudcore.log