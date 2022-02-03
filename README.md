# gitops
Repository for GitOps Implementation with Kubeedge


# Development Environment

The minimal tools required for creating a development environment to test GitOps.

- Kubernetes Cluster, microK8s has been used for such purpouse.
- Helm, for installing required tools in the cluster not supported by microk8s
- ArgoCD, as gitops supporting tool

Two development environments for creating the required Kubernetes Cluster has been used:

- macOs (amd64)
- Ubuntu 20.04 LTS (amd64)
## macOs

The macOs installation includes instructions to install the tools required to create the cluster using microk8s and inteect with it (helm, kubectl).

### Steps

#### Required tools

Installation using brew:

- *kubectl*

    ```
    brew install kubernetes-cli
    ```

- *helm*
    ```
    brew install helm
    ```

#### Install k8s

References:

- https://ubuntu.com/tutorials/install-microk8s-on-mac-os#1-overview

- https://microk8s.io/docs/addons

Install microk8s using homebrew

```
brew install ubuntu/microk8s/microk8s
```

Installed microk8s will create a VM for microk8s using multipass

To create a microk8s cluster (one node)

```
microk8s install --channel 1.19 --cpu  --mem --disk
```

```
microk8s status --wait-ready
```

Once installed to verity installation

```
multipass list
```

```
multipass info microk8s-vm
```

Install required features:
```
microk8s enable 
```

microk8s allows to execute kubctl commands in a very similar way

```
microk8s kubectl cluster-info
```

For developing purpouses is more common using the kubectl client. For this is omñy required to generate the config file.

```
microk8s config > $HOME/.kube/config
```

Checking configuration

```
kubectl cluster-info
```


## Ubuntu 20.04 LTS

Install kubectl using snap

```
sudo snap install kubectl --classic
```

Install helm
```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

#### Install k8s

References:
- https://microk8s.io/docs/getting-started


**Install microk8s using snap**

List available channels

```
sudo snap info microk8s
```

After checking compatibity with kubeedge version  

```
sudo snap install microk8s --classic --channel=1.19/stable
```

Enable basic services

```
microk8s enable dns dashboard storage
```


Check status  
```
microk8s status
```

For developing purpouses is more common using the kubectl client. For this is omñy required to generate the config file.

```
microk8s config > $HOME/.kube/config
```

Checking configuration

```
kubectl cluster-info
```




#### Install argoCD

Add helm repo for argoCD

helm repo add argo https://argoproj.github.io/argo-helm

ArgoCD can be installed using helm. The values used for it are
```
helm install argo-cd --create-namespace --namespace argo-cd --values values.yaml argo/argo-cd --debug --dry-run
```

NOTES:
In order to access the server UI the following options are avaialble:

1. Port fowarding
    ```
    kubectl port-forward service/argo-cd-argocd-server -n argo-cd 18080:443
    ```

    To check the result open the browser on http://localhost:18080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-1-ssl-passthrough
      - Add the `--insecure` flag to `server.extraArgs` in the values file and terminate SSL at your ingress: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:


```
kubectl -n argo-cd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Better approach

https://medium.com/devopsturkiye/self-managed-argo-cd-app-of-everything-a226eb100cf0


Dashboard

The Kubernetes Dashboard allows a basic monitoring and managemente of the cluster. Once enabled in microk8s.

1. Using fport fowarding
```
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443
```

To obatin the access token

```
kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token

token=$(kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
```

#### Install kubeedge

Reference:

- https://kubeedge.io/en/docs/setup/keadm/

Download keadmin 

https://github.com/kubeedge/kubeedge/releases



Install ClcoudCore

```
keadm init --advertise-address="KUBEDGE_CLOUDCORE_ADDRESS"

# Status /var/log/kubeedge/cloudcore.log
```

Install EdgeCore

```
keadm join --cloudcore-ipport=KUBEDGE_CLOUDCORE_ADDRESS:10000 --token=<TOKEN>

Status /var/log/kubeedge/cloudcore.log
```

Restart cloudcore (Cloud side)

```
ps aux|grep cloudcore
kill -9 PID
<PATH>/cloudcore &
```

Restart edgecore (Edge side)
```
sudo systemctl restart edgecore
```






