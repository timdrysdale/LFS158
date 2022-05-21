# EdX course notes on kubernetes

[EdxCourse](https://learning.edx.org/course/course-v1:LinuxFoundationX+LFS158x+1T2022/)


Let's learn how to install the latest Minikube release on Ubuntu Linux 20.04 LTS with VirtualBox v6.1 specifically.

NOTE: For other Linux OS distributions or releases, VirtualBox and Minikube versions, the installation steps may vary! Check the Minikube installation!

Verify the virtualization support on your Linux OS in a terminal (a non-empty output indicates supported virtualization):

$ grep -E --color 'vmx|svm' /proc/cpuinfo

Install the VirtualBox hypervisor. In a terminal run the following commands to add the recommended source repository for local OS, download and register the public key, update and install:

$ sudo bash -c 'echo "deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian eoan contrib" >> /etc/apt/sources.list'

$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -

$ sudo apt update

$ sudo apt install -y virtualbox-6.1

Install Minikube. We can download and install in a terminal the latest release or a specific release from the Minikube release page:

$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

$ sudo dpkg -i minikube_latest_amd64.deb

NOTE: Replacing /latest/ with a particular version, such as /v1.24.0/ will download that specified Minikube version.

Start Minikube. In a terminal we can start Minikube with the minikube start command, that bootstraps a single-node cluster with the latest stable Kubernetes version release. For a specific Kubernetes version the --kubernetes-version option can be used as such minikube start --kubernetes-version v1.22.0 (where latest is default and acceptable version value, and stable is also acceptable). More advanced start options will be explored later in this chapter:

Ach ... ended up with docker driver anyway - proceed!

```
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

```
 
```
$ minikube profile list
|----------|-----------|---------|--------------|------|----------|---------|-------|
| Profile  | VM Driver | Runtime |      IP      | Port | Version  | Status  | Nodes |
|----------|-----------|---------|--------------|------|----------|---------|-------|
| minikube | docker    | docker  | 192.168.49.2 | 8443 | v1.21.11 | Running |     1 |
|----------|-----------|---------|--------------|------|----------|---------|-------|

```

Example k8s profiles you can start
```
$ minikube start --kubernetes-version=v1.22.2 --driver=podman --profile minipod

$ minikube start --nodes=2 --kubernetes-version=v1.23.1 --driver=docker --profile doubledocker

$ minikube start --driver=virtualbox --nodes=3 --disk-size=10g --cpus=2 --memory=4g --kubernetes-version=v1.22.4 --cni=calico --container-runtime=containerd -p multivbox

$ minikube start --driver=docker --cpus=6 --memory=8g --kubernetes-version="1.23.2" -p largedock

$ minikube start --driver=virtualbox -n 3 --container-runtime=cri-o --cni=calico -p minibox

```

Use -p to specify which profile
```
minikube stop -p minibox
```

useful commands

```
minikube -p minibox ip
minikube -p minibox ip -n minibox-m02
minikube delete -p minibox
```
```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
$ minikube addons list

$ minikube addons enable metrics-server

$ minikube addons enable dashboard

$ minikube addons list

$ minikube dashboard 
```

APIs with Auth

Retrieve the token:

$ TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")

APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")

(this lists all APIS ... manually choose the one we want)
```
kubectl config view | grep https | cut -f 2- -d ":" | tr -d " "
APISERVER=https://192.168.49.2:8443
```

so...

```
TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")
APISERVER=https://192.168.49.2:8443
```

```
curl $APISERVER --header "Authorization: Bearer $TOKEN" --insecure

#alternative if you have info from the .kube/config file
curl $APISERVER --cert encoded-cert --key encoded-key --cacert encoded-ca
```

(no further notes copied into repo - see course - it is free)
