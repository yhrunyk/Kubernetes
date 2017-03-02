# Kubernetes
File used for kubernetes

- create service with type "NodePort"

kubectl expose deployment/oms --type="NodePort" --port 8080


# Kubernetes

Deployment on CentOS 7

We will need 4 servers, running on CentOS 7 64 bit with minimal install. All components are available directly from the CentOS extras repository which is enabled by default.

Prerequisites

1. Disable iptables on each node to avoid conflicts with Docker iptables rules:

$ systemctl stop firewalld

$ systemctl disable firewalld

2. Install NTP and make sure it is enabled and running:

$ yum -y install ntp

$ systemctl start ntpd

$ systemctl enable ntpd

3. Setting up the Kubernetes Master

The following steps should be performed on the master.

- Install etcd and Kubernetes through yum:

$ yum -y install etcd kubernetes

- Configure etcd to listen to all IP addresses inside /etc/etcd/etcd.conf. Ensure the following lines are uncommented, and assign the following values:

ETCD_NAME=default

ETCD_DATA_DIR="/var/lib/etcd/default.etcd"

ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"

ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"

- Configure Kubernetes API server inside /etc/kubernetes/apiserver. Ensure the following lines are uncommented, and assign the following values:

KUBE_API_ADDRESS="--address=0.0.0.0"

KUBE_API_PORT="--port=8080"

KUBELET_PORT="--kubelet_port=10250"

KUBE_ETCD_SERVERS="--etcd_servers=http://127.0.0.1:2379"

KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_API_ARGS=""

- Start and enable etcd, kube-apiserver, kube-controller-manager and kube-scheduler:

systemctl restart etcd kube-apiserver kube-controller-manager kube-scheduler
  
systemctl enableetcd kube-apiserver kube-controller-manager kube-scheduler

- Define flannel network configuration in etcd. This configuration will be pulled by flannel service on minions:

$ etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'

- At this point, we should notice that nodes' status returns nothing because we haven’t started any of them yet:

$ kubectl get nodes
NAME             LABELS              STATUS
