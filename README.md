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

 The values used for it are available in this repository in 
```sh
helm install argo-cd --create-namespace --namespace argo-cd --values values.yaml argo/argo-cd --debug --dry-run
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


## Install mesh

In order to allow access to containers running in edge nodes is required to install Kubeedge EdgeMesh.

Reference:

- https://github.com/kubeedge/edgemesh

- https://edgemesh.netlify.app/



**Configure CloudSide**

Install using helm chart

```
helm install edgemesh \
--set server.nodeName=desktop-ubuntu \
--set "server.advertiseAddress={192.168.1.50}" \
https://raw.githubusercontent.com/kubeedge/edgemesh/main/build/helm/edgemesh.tgz
```



**Configure EdgeMesh**

Perform the following steps to configure EdgeMesh on your edge node.

1. Edit /etc/nsswitch.conf.

    ```
    vi /etc/nsswitch.conf
    ```


2. Add the following content to this file:

    ```
    hosts: dns files mdns4_minimal 
    ```

3. Save the file and run the following command to enable IP forwarding:

    ```
    sudo echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
    ```

4. Verify ip fowarding:

    ```
    sudo sysctl -p | grep ip_forward
    ```

5. Expected result:

    ```
    net.ipv4.ip_forward = 1
    ```

**Ubuntu 20.04 Jetson Nanp**

https://qengineering.eu/install-ubuntu-20.04-on-jetson-nano.html

https://sleeplessbeastie.eu/2021/12/06/how-to-install-gitlab-runner-on-raspberry-pi/

Install gitlab-runner (armd64)

```sh
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-arm64

# Give it permission to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab Runner user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as a service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start

# Add user to docker group

sudo usermod -aG docker gitlab-runner

```


Failed to delete directory /etc/kubeedge/: unlinkat /etc/kubeedge/ca/rootCA.crt: permission denied
Failed to delete directory /var/lib/kubeedge/: unlinkat /var/lib/kubeedge/edgecore.db: permission denied
Failed to delete directory /var/lib/edged: open /var/lib/edged: permission denied
Failed to delete directory /var/lib/dockershim: unlinkat /var/lib/dockershim/sandbox/11f60987807c7668adc539343755ff3dc6c407a613da96abfbb352084942c9b2: permission denied

RBAC (kubernetes-dashboard)
kubectl -n kubernetes-dashboard create token admin-user

thisisunsafe

## Enable metrics


Generate certificates.

### Cloudside

Copy [certgen.sh] to /etc/kubeedge

Declare Kubernetes CA file and key

- K8SCA_FILE, /etc/kubernetes/pki/ca.crt
- K8SCA_KEY_FILE, /etc/kubernetes/pki/ca.key

In microk8s

- K8SCA_FILE, /var/snap/microk8s/current/certs/ca.crt
- K8SCA_KEY_FILE, /var/snap/microk8s/current/certs/ca.key

And cloudcore ip server

```sh
## Set working directory
cd /etc/kubeedge

# Declare vars
export CLOUDCOREIPS="192.168.1.50"
export K8SCA_FILE=/var/snap/microk8s/current/certs/ca.crt
export K8SCA_KEY_FILE=/var/snap/microk8s/current/certs/ca.key

# Generate certificates
./certgen.sh stream

```