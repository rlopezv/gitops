# Content
Repository for GitOps Implementation with Kubeedge


# Development Environment

The minimal tools required for creating a development environment for testing GitOps implementation in a k8s environment.

- **Kubernetes Cluster**, *microK8s* has been used for such purpouse.
- **Helm**, for installing required tools in the cluster not included in microk8s
- **ArgoCD**, as CD tool

#### References 

- [**microk8s**](https://microk8s.io/)
- [**Helm**](https://helm.sh/)
- [**ArgoCD**](https://argo-cd.readthedocs.io/en/stable/)

Two development environments for creating the required Kubernetes Cluster has been used:

- macOs (amd64)
- Ubuntu 20.04 LTS (amd64)


# Environment Installation

## macOs

The macOs installation includes instructions to install the tools required to create the cluster using microk8s and inteect with it (helm, kubectl).

Brew (the missing package manager) is required for installation. 


### Install brew

Follow the instructions included in  [brew reference](https://brew.sh/)


### Steps

Using *brew*


#### Required tools

Installation using brew

Check brew status

```sh
brew info
```


*kubectl*

```sh
brew install kubernetes-cli
```

*helm*

```sh
brew install helm
```

#### Install k8s


*References:*

- https://ubuntu.com/tutorials/install-microk8s-on-mac-os#1-overview

- https://microk8s.io/docs/addons


Install microk8s using homebrew

```sh
brew install ubuntu/microk8s/microk8s
```

Installed microk8s will create a VM for microk8s using multipass

To create a microk8s cluster (one node)

```sh
microk8s install --channel 1.19 --cpu  --mem --disk
```

```sh
microk8s status --wait-ready
```

Once installed, verity installation

```sh
multipass list
```

```sh
multipass info microk8s-vm
```

Install required features: 

```sh
microk8s enable 
```

microk8s allows to execute kubctl commands in a very similar way

```sh
microk8s kubectl cluster-info
```

For developing purpouses is more common using the kubectl client. For this is omñy required to generate the config file.

```sh
microk8s config > $HOME/.kube/config
```

Checking configuration

```
kubectl cluster-info
```
  

## Ubuntu 20.04 LTS

Snap will be used for installation

```sh
sudo apt update
sudo apt install snapd
```

#### References

- [snap](https://snapcraft.io/docs)

### Install kubectl

Use snap for installation

```sh
sudo snap install kubectl --classic
```

### Install Helm

```sh
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Install microk8s

References:
- https://microk8s.io/docs/getting-started


#### Install microk8s using snap

List available channels

```
sudo snap info microk8s
```

After checking compatibity with kubeedge version  

```
sudo snap install microk8s --classic --channel=1.22/stable
```

Enable basic services

```
microk8s enable dns dashboard storage
```


Check status  
```
microk8s status
```

For developing purpouses is more common using the kubectl client. So the following command is used to generate the reuired config file.

```
microk8s config > $HOME/.kube/config
```

Checking configuration

```
kubectl cluster-info
```


## Install argoCD

Add helm repo for argoCD

ArgoCD can be installed using helm. The values (values.yml) used for it are available in *infra/argocd*. 

```sh
helm repo add argo https://argoproj.github.io/argo-helm
```

 The values used for it are available in this repository in infra/charts/argocd

```sh
hhelm install argo-cd --create-namespace --namespace argo-cd --values values.yaml --version 4.3.1 argo/argo-cd --debug --dry-run
```

NOTES:
In order to access the server UI the following options are avaialble:

1. Port fowarding
    ```sh
    kubectl port-forward service/argo-cd-argocd-server -n argo-cd 18080:443
    ```

    To check the result open the browser on http://localhost:18080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-1-ssl-passthrough
      - Add the `--insecure` flag to `server.extraArgs` in the values file and terminate SSL at your ingress: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by executing:


```sh
kubectl -n argo-cd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**Alternative**

You can create yout own chart for managing argoCD installation as described in:

https://medium.com/devopsturkiye/self-managed-argo-cd-app-of-everything-a226eb100cf0
https://medium.com/@andrew.kaczynski/gitops-in-kubernetes-argo-cd-and-gitlab-ci-cd-5828c8eb34d6


###  Dashboard

The Kubernetes Dashboard allows a basic monitoring and managemente of the cluster. 

Once enabled in microk8s.
```
microk8s enable dashboard
```

Several options for accessing the dashboard

1. Using port fowarding
```sh
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8080:443
```

To obatin the access token

```sh
kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token

token=$(kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
```

## Installing Edge Runtime


Since Ubuntu and CentOs are described as the supported runtimes, it's required to install Ubuntu on edge nodes.

The edgeNodes are *Raspberry pi 4 Model B*. Other devices supporting Ubuntu can be used.

### Prepare nodes

These instructions are for for ubuntu 20.04.04 LTS (arm64).

Create sd boot for rpi

https://www.raspberrypi.com/documentation/computers/getting-started.html#using-raspberry-pi-imager


Once installed is required to change some parameters of the OS kernel in rpis to allow kubeedge execution.

**Enable remote ssh**
Check config

https://roboticsbackend.com/install-ubuntu-on-raspberry-pi-without-monitor/

Look for system-boot partition in Ubuntu SD Card

Configure wifi access

In network-config add content

```yaml
version: 2
ethernets:
  eth0:
    dhcp4: true
    optional: true
wifis:
  wlan0:
    dhcp4: true
    optional: true
    access-points:
      "YOUR_WIFI_NAME":
        password: "YOUR_WIFI_PASSWORD"
```

In /user-data check ssh is enabled

```yaml
...
# On first boot, set the (default) ubuntu user's password to "ubuntu" and expire user passwords
chpasswd:
  expire: true
  list:
  - ubuntu:ubuntu

# Enable password authentication with the SSH daemon
ssh_pwauth: true
...
```
Username/password: ubuntu/ubuntu

**Enable cgroup**

modify /boot/firmware/cmdline.txt 

add cgroup_enable=memory cgroup_memory=1

The content should be something like

```
console=serial0,115200 console=tty1 root=PARTUUID=738a4d67-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```
**Upgrade OS**

Get list of avilable updates

```sh
sudo apt update
```

Upgrade OS
```sh
sudo apt upgrade
```

After this reboot

**Change hostname**

Set edgeNodeName

```sh
sudo hostnamectl set-hostname <edgeNodeName>
```

Add new hostNmae to it

```sh
vi /etc/hosts
```

After this Reboot

**Install docker**

Using docker script

```sh
curl -sSL https://get.docker.com | sh
```

Add user to docker group
```sh
sudo usermod -aG docker ${USER}
```

Reboot
```sh
sudo reboot
```


**Install kubeedge**

Reference:

- https://kubeedge.io/en/docs/setup/keadm/


**Install keadmin**

- https://github.com/kubeedge/kubeedge/releases

In the example keadm v1.9.2.

*On cloudside*

For amd64 (Ubuntu 20)

```sh
wget https://github.com/kubeedge/kubeedge/releases/download/v1.9.2/keadm-v1.9.2-linux-amd64.tar.gz 
```

*On edgeside*

For arm64 (Ubuntu 20)

```sh
wget https://github.com/kubeedge/kubeedge/releases/download/v1.9.2/keadm-v1.9.2-linux-arm64.tar.gz 
```

```sh
tar xvf keadm-v1.9.2-linux-arm64.tar.gz

mv <extracted> /usr/local/bin/keadm

chmod +x /usr/local/bin/keadm
```

**Install CloudCore**

Cloudside with k8s cluster access

```sh
keadm init --kube-config ${HOME}/.kube/config --advertise-address "KUBEDGE_CLOUDCORE_ADDRESS"
```

Obtain token to enroll edge nodes

```sh
keadm gettoken --kube-config ${HOME}/.kube/config
```


Status 
```sh
tail -f /var/log/kubeedge/cloudcore.log
```

Rebooting
```
pkill cloudcore
nohup cloudcore > cloudcore.log 2>&1 &
```

Restart cloudcore (Cloud side)

```
ps aux | grep cloudcore
kill -9 PID
<PATH>/cloudcore &
```

If registered as service

Create a file names cloudcore.service in 

```
[Unit]
Description=cloudcore.service

[Service]
Type=simple
ExecStart=/usr/local/bin/cloudcore
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Start service

```
systemctl start cloudcore
```

Enable automatic start updocker rm -f $(docker ps -a -q) of service

```
systemctl enable cloudcore
```


Check service status

```
systemctl --type=service | grep cloudcore
```

**Install EdgeCore**

Edgeside in nodes


Enroll

```
keadm join --cloudcore-ipport=KUBEDGE_CLOUDCORE_ADDRESS:10000 --edgenode-name=<NODE-NAME> --token=<TOKEN> --kubeedge-version=<>

```

Check status

```
journalctl -u edgecore.service -b
```

Restart edgecore (Edge side)
```
sudo systemctl restart edgecore
```

### ArgoCD rest API

**SWAGGER-API**
$ARGOCD_SERVER/swagger-ui

**Obtain token**

```sh
curl $ARGOCD_SERVER/api/v1/session -d $'{"username":"admin","password":"password"}'
```

**Use API**

```sh
curl $ARGOCD_SERVER/api/v1/applications -H "Authorization: Bearer $ARGOCD_TOKEN" 
```


## Enable Metrics

In order to enable metrics is required to make some actions.

Obtain K8S certificates

### Reference 
  - https://microk8s.io/docs/services-and-ports



## Enable metrics


Generate certificates.

### Cloudside

The cluster CA is required for enabling comunication between cloudcore and edgecore.

This are usually located in **/etc/kubernetes/pki** in microk8s these are located in **/var/snap/microk8s/current/certs/**

CA file and key

- K8SCA_FILE, /etc/kubernetes/pki/ca.crt
- K8SCA_KEY_FILE, /etc/kubernetes/pki/ca.key

In microk8s

- K8SCA_FILE, /var/snap/microk8s/current/certs/ca.crt
- K8SCA_KEY_FILE, /var/snap/microk8s/current/certs/ca.key

It's also required to declare the CLOUDCOREIPs.

Copy [certgen.sh](infra/kubeedge/certgen.sh) to /etc/kubeedge

To generate the certificates required for allowing remote loging

```sh
## Set working directory
cd /etc/kubeedge

# Declare vars
export CLOUDCOREIPS="[servers]"
export K8SCA_FILE=/var/snap/microk8s/current/certs/ca.crt
export K8SCA_KEY_FILE=/var/snap/microk8s/current/certs/ca.key

# Generate certificates
./certgen.sh stream

```


## Enable routing

Once these certificates are generated it's required to modify the iptables and the keubedge configuration files on both cloudside and edgeside.

On cloudside

Modify **/etc/kubeedge/config/cloudcore.yaml**

Enable stream

```txt
cloudStream:
  # change
  enable: true
```

```sh
export CLOUDCOREIPS="[servers]"
iptables -t nat -A OUTPUT -p tcp --dport 10350 -j DNAT --to $CLOUDCOREIPS:10003
```


On edge side (every edge node)

Modify **/etc/kubeedge/config/edgecore.yaml**

Enable stream

```txt
edgeStream:
  # change
  enable: true
  # Check value of cloudserver
  server: [CLOUDCOREIP]:10004 
```

```sh
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

Restart cloudside and edge side

```sh
# On cloudside
systemctl restart cloudcore
```


```sh
# On edgeside (every edge node)
systemctl restart edgecore
```

Once done, tt will be possible to access node statistics throw the common interfaces.

```sh
# On cloud sice
curl "http://[NODE_IP]:10350/stats/summary?only_cpu_and_memory=true"
```

```sh
# On edge sice
curl "http://localhost:10350/stats/summary?only_cpu_and_memory=true"
```

```json
{
 "node": {
  "nodeName": "edgenode01",
  "systemContainers": [
   {
    "name": "kubelet",
    "startTime": "2022-10-22T16:17:01Z",
    "cpu": {
     "time": "2022-10-22T18:45:07Z",
     "usageNanoCores": 36490670,
     "usageCoreNanoSeconds": 252662549983
    },
    "memory": {
     "time": "2022-10-22T18:45:07Z",
     "usageBytes": 30613504,
     "workingSetBytes": 30613504,
     "rssBytes": 27066368,
     "pageFaults": 22869,
     "majorPageFaults": 0
    }
   },
   {
    "name": "runtime",
    "startTime": "2022-10-20T16:47:00Z",
    "cpu": {
     "time": "2022-10-22T18:45:13Z",
     "usageNanoCores": 13118760,
     "usageCoreNanoSeconds": 1786750198925
    },
    "memory": {
     "time": "2022-10-22T18:45:13Z",
     "usageBytes": 115150848,
     "workingSetBytes": 51351552,
     "rssBytes": 36360192,
     "pageFaults": 19054431,
     "majorPageFaults": 297
    }
   },
   {
    "name": "pods",
    "startTime": "2022-10-20T16:47:31Z",
    "cpu": {
     "time": "2022-10-22T18:45:03Z",
     "usageNanoCores": 0,
     "usageCoreNanoSeconds": 0
    },
    "memory": {
     "time": "2022-10-22T18:45:03Z",
     "availableBytes": 3977527296,
     "usageBytes": 0,
     "workingSetBytes": 0,
     "rssBytes": 0,
     "pageFaults": 0,
     "majorPageFaults": 0
    }
   }
  ],
  "startTime": "2022-10-20T16:47:01Z",
  "cpu": {
   "time": "2022-10-22T18:45:02Z",
   "usageNanoCores": 62053022,
   "usageCoreNanoSeconds": 9409379988290
  },
  "memory": {
   "time": "2022-10-22T18:45:02Z",
   "availableBytes": 2643247104,
   "usageBytes": 2401181696,
   "workingSetBytes": 1334280192,
   "rssBytes": 165142528,
   "pageFaults": 76527,
   "majorPageFaults": 165
  },
  "network": {
   "time": "2022-10-22T18:45:02Z",
   "name": "eth0",
   "rxBytes": 680926297,
   "rxErrors": 0,
   "txBytes": 43422651,
   "txErrors": 0,
   "interfaces": [
    {
     "name": "wlan0",
     "rxBytes": 0,
     "rxErrors": 0,
     "txBytes": 0,
     "txErrors": 0
    },
    {
     "name": "eth0",
     "rxBytes": 680926297,
     "rxErrors": 0,
     "txBytes": 43422651,
     "txErrors": 0
    }
   ]
  },
  "fs": {
   "time": "2022-10-22T18:45:02Z",
   "availableBytes": 22324531200,
   "capacityBytes": 31064162304,
   "usedBytes": 7425003520,
   "inodesFree": 1805511,
   "inodes": 1933312,
   "inodesUsed": 127801
  },
  "runtime": {
   "imageFs": {
    "time": "2022-10-22T18:45:02Z",
    "availableBytes": 22324531200,
    "capacityBytes": 31064162304,
    "usedBytes": 1395364102,
    "inodesFree": 1805511,
    "inodes": 1933312,
    "inodesUsed": 127801
   }
  },
  "rlimit": {
   "time": "2022-10-22T18:45:17Z",
   "maxpid": 4194304,
   "curproc": 204
  }
 },
 "pods": []
}
```



## Modify metrics-server deployment

Since edgenodes are not really k8s nodes it's required to customize the metrics-server used for accessing node metrics.

In this case, version 5.2 of metrics server has been used.

```sh
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.2/components.yaml -O deploy.yaml
```

And customized with the following values:


The file is avaliable in [deploy.yaml](infra/metrics-server/deploy.yaml)

Enable hostnetwork mode

```yml
spec:
  template:
    spec:
      hostNetwork: true
```

Skip tls insecure

```yml
spec:
  template:
    spec:
      containers:
      - args:
        - --kubelet-insecure-tls
```

Node selector the service must be deployed in master node
```yml
spec:
  template:
    spec:
      nodeSelector:
        beta.kubernetes.io/arch : amd64
```


Once modified it can be applyed using 
```sh
kubectl apply -f deploy.yaml
```

Once completed the following info should be available

```
$ kubectl top nodes
NAME             CPU(cores)   CPU%        MEMORY(bytes)   MEMORY%     
edgenode01       48m          1%          1272Mi          34%         
edgenode02       41m          1%          1169Mi          31%         
rpi3             97m          2%          403Mi           49%         
ubuntu-desktop   1688m        21%         9959Mi          41%         
edgenode03       <unknown>    <unknown>   <unknown>       <unknown>  
```

## Observing

It will use Grafana and Prometheus included in microk8s distribution.

```sh
microsk8s enable prometheus
```

Once enabled these can be accesed through port fowarding

Set port-forwarding to enable external access

**PrometheusUI**
```sh
$sh microk8s kubectl port-forward -n monitoring service/prometheus-k8s --address 0.0.0.0 9090:9090
```
```txt
Forwarding from 0.0.0.0:9090 -> 9090
```
**Grafana UI**
```sh
$sh microk8s kubectl port-forward -n monitoring service/grafana --address 0.0.0.0 3000:3000
```txt
Forwarding from 0.0.0.0:3000 -> 3000
```

The metrics consumed in prometheus are configured to obtain data from port 10250.

The edgenodes metrics are avaliable on port 10350, so it's required to modify the endpoints used for retreaving node statistics.




## Install mesh (Optional)

In order to allow access to containers running in edge nodes is required to install Kubeedge EdgeMesh.

Reference:

- https://github.com/kubeedge/edgemesh

- https://edgemesh.netlify.app/



thisisunsafe

