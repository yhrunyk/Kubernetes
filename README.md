# Kubernetes
File used for kubernetes

- create service with type "NodePort"

kubectl expose deployment/oms --type="NodePort" --port 8080


# Kubernetes

Deployment on CentOS 7

We will need 4 servers, running on CentOS 7 64 bit with minimal install. All components are available directly from the CentOS extras repository which is enabled by default.

Prerequisites

1) Disable iptables on each node to avoid conflicts with Docker iptables rules:

$ systemctl stop firewalld

$ systemctl disable firewalld

2) Install NTP and make sure it is enabled and running:

$ yum -y install ntp

$ systemctl start ntpd

$ systemctl enable ntpd

3) Setting up the Kubernetes Master

The following steps should be performed on the master.

4) Install etcd and Kubernetes through yum:

$ yum -y install etcd kubernetes

5) Configure etcd to listen to all IP addresses inside /etc/etcd/etcd.conf. Ensure the following lines are uncommented, and assign the following values:

ETCD_NAME=default

ETCD_DATA_DIR="/var/lib/etcd/default.etcd"

ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"

ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"

6) Configure Kubernetes API server inside /etc/kubernetes/apiserver. Ensure the following lines are uncommented, and assign the following values:

KUBE_API_ADDRESS="--address=0.0.0.0"

KUBE_API_PORT="--port=8080"

KUBELET_PORT="--kubelet_port=10250"

KUBE_ETCD_SERVERS="--etcd_servers=http://127.0.0.1:2379"

KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_API_ARGS=""

7) Start and enable etcd, kube-apiserver, kube-controller-manager and kube-scheduler:

$ systemctl restart etcd kube-apiserver kube-controller-manager kube-scheduler
  
$ systemctl enableetcd kube-apiserver kube-controller-manager kube-scheduler

8) Define flannel network configuration in etcd. This configuration will be pulled by flannel service on minions:

$ etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'

9) At this point, we should notice that nodes' status returns nothing because we haven’t started any of them yet:

$ kubectl get nodes
NAME             LABELS              STATUS

# Setting up Kubernetes Minions (Nodes)

The following steps should be performed on minion1, minion2 and minion3 unless specified otherwise.

1) Install flannel and Kubernetes using yum:

$ yum -y install flannel kubernetes

2) Configure etcd server for flannel service. Update the following line inside /etc/sysconfig/flanneld to connect to the respective master:

FLANNEL_ETCD="http://192.168.50.130:2379"

3) Configure Kubernetes default config at /etc/kubernetes/config, ensure you update the KUBE_MASTER value to connect to the Kubernetes master API server:

KUBE_MASTER="--master=http://192.168.50.130:8080"

4)  Configure kubelet service inside /etc/kubernetes/kubelet as below:

minion1:

KUBELET_ADDRESS="--address=0.0.0.0"

KUBELET_PORT="--port=10250"

- change the hostname to this host’s IP address

KUBELET_HOSTNAME="--hostname_override=192.168.50.131"

KUBELET_API_SERVER="--api_servers=http://192.168.50.130:8080"

KUBELET_ARGS=""

minion2:

KUBELET_ADDRESS="--address=0.0.0.0"

KUBELET_PORT="--port=10250"

- change the hostname to this host’s IP address

KUBELET_HOSTNAME="--hostname_override=192.168.50.132"

KUBELET_API_SERVER="--api_servers=http://192.168.50.130:8080"

KUBELET_ARGS=""

minion3:

KUBELET_ADDRESS="--address=0.0.0.0"

KUBELET_PORT="--port=10250"

- change the hostname to this host’s IP address

KUBELET_HOSTNAME="--hostname_override=192.168.50.133"

KUBELET_API_SERVER="--api_servers=http://192.168.50.130:8080"

KUBELET_ARGS=""

5) Start and enable kube-proxy, kubelet, docker and flanneld services:

$ systemctl restart kube-proxy kubelet docker flanneld

$ systemctl enable kube-proxy kubelet docker flanneld

6) On each minion, you should notice that you will have two new interfaces added, docker0 and flannel0. You should get different range of IP addresses on flannel0 interface on each minion, similar to below:

minion1:

$ ip a | grep flannel | grep inet

inet 172.17.45.0/16 scope global flannel0

minion2:

$ ip a | grep flannel | grep inet

inet 172.17.38.0/16 scope global flannel0

minion3:

$ ip a | grep flannel | grep inet

inet 172.17.93.0/16 scope global flannel0

7) Now login to Kubernetes master node and verify the minions’ status:

$ kubectl get nodes

NAME             LABELS                                  STATUS

192.168.50.131   kubernetes.io/hostname=192.168.50.131   Ready

192.168.50.132   kubernetes.io/hostname=192.168.50.132   Ready

192.168.50.133   kubernetes.io/hostname=192.168.50.133   Ready

You are now set. The Kubernetes cluster is now configured and running. We can start to play around with pods.
