# microk8s cluster

This document describes how to create an install a microk8s cluster.

This section describes how to install in hosts running Ubuntu 20.04 LTS.

## Ubuntu 20.04 LTS

If one of the nodes is a Raspberry Pi is required to enable cgroups

**Enable cgroup**

modify /boot/firmware/cmdline.txt 

add cgroup_enable=memory cgroup_memory=1

The content should be something like

```
console=serial0,115200 console=tty1 root=PARTUUID=738a4d67-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```


Snap will be used for installation


```sh
sudo apt update
sudo apt install snapd
```

#### References

- [snap](https://snapcraft.io/docs)


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



Check status  
```
microk8s status
```

For developing purpouses is more common using the kubectl client. So the following command is used to generate the reuired config file.

```
microk8s config > $HOME/.kube/config
```


## Creating the cluster

On the *master node* execute

```
sudo microk8s.add-node
```

This will return the required join command to be executed on the leaf nodes

```
microk8s join 192.168.1.50:25000/42bf6b4f28e7ddd94266875c1e59d859/794eb9fcae04
```

It's important that master node address and port are accesible from the leaf nodes.


On the *edge node* execute the join command

```
microk8s join 192.168.1.50:25000/42bf6b4f28e7ddd94266875c1e59d859/794eb9fcae04
Contacting cluster at 192.168.1.50
Waiting for this node to finish joining the cluster. .. .. .. .. .. .. .. .. .. ..  
```

After a while the node will joint the cluster

```
ubuntu-desktop    Ready      <none>   20m   v1.22.16-3+496586ad04d1d5
microk8s-worker   NotReady   <none>   16s   v1.22.16-3+024782121eb186
```

Calico will be enabled as CNI provider.


```
kubectl get all -A -o wide
NAMESPACE     NAME                                             READY   STATUS    RESTARTS        AGE     IP              NODE              NOMINATED NODE   READINESS GATES
kube-system   pod/dashboard-metrics-scraper-58d4977855-zm6tq   1/1     Running   0               22m     10.1.232.131    ubuntu-desktop    <none>           <none>
kube-system   pod/kubernetes-dashboard-869949b85-7575r         1/1     Running   0               22m     10.1.232.133    ubuntu-desktop    <none>           <none>
kube-system   pod/hostpath-provisioner-566686b959-f2z4s        1/1     Running   1 (6m23s ago)   22m     10.1.232.132    ubuntu-desktop    <none>           <none>
kube-system   pod/coredns-7f9c69c78c-txwts                     1/1     Running   0               24m     10.1.232.130    ubuntu-desktop    <none>           <none>
kube-system   pod/metrics-server-85df567dd8-bxq4q              1/1     Running   0               22m     10.1.232.134    ubuntu-desktop    <none>           <none>
kube-system   pod/calico-node-7mplj                            1/1     Running   0               4m54s   192.168.1.50    ubuntu-desktop    <none>           <none>
kube-system   pod/calico-kube-controllers-7d499c7fb5-75dc4     1/1     Running   0               4m42s   10.1.232.135    ubuntu-desktop    <none>           <none>
kube-system   pod/calico-node-wp29n                            1/1     Running   0               4m54s   192.168.1.125   microk8s-worker   <none>           <none>

NAMESPACE     NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
default       service/kubernetes                  ClusterIP   10.152.183.1     <none>        443/TCP                  25m   <none>
kube-system   service/kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   24m   k8s-app=kube-dns
kube-system   service/metrics-server              ClusterIP   10.152.183.127   <none>        443/TCP                  24m   k8s-app=metrics-server
kube-system   service/kubernetes-dashboard        ClusterIP   10.152.183.219   <none>        443/TCP                  24m   k8s-app=kubernetes-dashboard
kube-system   service/dashboard-metrics-scraper   ClusterIP   10.152.183.188   <none>        8000/TCP                 24m   k8s-app=dashboard-metrics-scraper

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS    IMAGES                          SELECTOR
kube-system   daemonset.apps/calico-node   2         2         2       2            2           kubernetes.io/os=linux   25m   calico-node   docker.io/calico/node:v3.19.1   k8s-app=calico-node

NAMESPACE     NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                  IMAGES                                            SELECTOR
kube-system   deployment.apps/coredns                     1/1     1            1           24m   coredns                     coredns/coredns:1.8.0                             k8s-app=kube-dns
kube-system   deployment.apps/dashboard-metrics-scraper   1/1     1            1           24m   dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.6               k8s-app=dashboard-metrics-scraper
kube-system   deployment.apps/hostpath-provisioner        1/1     1            1           23m   hostpath-provisioner        cdkbot/hostpath-provisioner:1.1.0                 k8s-app=hostpath-provisioner
kube-system   deployment.apps/kubernetes-dashboard        1/1     1            1           24m   kubernetes-dashboard        kubernetesui/dashboard:v2.3.0                     k8s-app=kubernetes-dashboard
kube-system   deployment.apps/metrics-server              1/1     1            1           24m   metrics-server              k8s.gcr.io/metrics-server/metrics-server:v0.5.0   k8s-app=metrics-server
kube-system   deployment.apps/calico-kube-controllers     1/1     1            1           25m   calico-kube-controllers     docker.io/calico/kube-controllers:v3.17.3         k8s-app=calico-kube-controllers

NAMESPACE     NAME                                                   DESIRED   CURRENT   READY   AGE     CONTAINERS                  IMAGES                                            SELECTOR
kube-system   replicaset.apps/coredns-7f9c69c78c                     1         1         1       24m     coredns                     coredns/coredns:1.8.0                             k8s-app=kube-dns,pod-template-hash=7f9c69c78c
kube-system   replicaset.apps/dashboard-metrics-scraper-58d4977855   1         1         1       22m     dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.6               k8s-app=dashboard-metrics-scraper,pod-template-hash=58d4977855
kube-system   replicaset.apps/hostpath-provisioner-566686b959        1         1         1       22m     hostpath-provisioner        cdkbot/hostpath-provisioner:1.1.0                 k8s-app=hostpath-provisioner,pod-template-hash=566686b959
kube-system   replicaset.apps/kubernetes-dashboard-869949b85         1         1         1       22m     kubernetes-dashboard        kubernetesui/dashboard:v2.3.0                     k8s-app=kubernetes-dashboard,pod-template-hash=869949b85
kube-system   replicaset.apps/metrics-server-85df567dd8              1         1         1       22m     metrics-server              k8s.gcr.io/metrics-server/metrics-server:v0.5.0   k8s-app=metrics-server,pod-template-hash=85df567dd8
kube-system   replicaset.apps/calico-kube-controllers-7c5c869448     0         0         0       25m     calico-kube-controllers     docker.io/calico/kube-controllers:v3.17.3         k8s-app=calico-kube-controllers,pod-template-hash=7c5c869448
kube-system   replicaset.apps/calico-kube-controllers-7d499c7fb5     1         1         1       4m42s   calico-kube-controllers     docker.io/calico/kube-controllers:v3.17.3         k8s-app=calico-kube-controllers,pod-template-hash=7d499c7fb5
```

Enable basic services

```
microk8s enable dns dashboard storage
```

Enable Ingress (nginx)
```
microk8s enable ingress
```

Enable Prometheus
```
microk8s enable prometheus
```

Set port-forwarding to enable external access

**PrometheusUI**
```sh
$ microk8s kubectl port-forward -n monitoring service/prometheus-k8s --address 0.0.0.0 9090:9090
```
```txt
Forwarding from 0.0.0.0:9090 -> 9090
```
**Grafana UI**
```sh
$ microk8s kubectl port-forward -n monitoring service/grafana --address 0.0.0.0 3000:3000
```
```txt
Forwarding from 0.0.0.0:3000 -> 3000
```

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
