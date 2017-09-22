# k8s_nginx_lb
From k8s deployment a loadblance nginx cluster.
# k8s_env,master_count:1, pod_count:2.
  *  **master_configuration:**
  ```
  # yum -y install etcd kubernetes-master
  ```
  ```
  /etc/etcd/etcd.conf
  ETCD_NAME=default
  ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
  ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
  ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"

  /etc/kubernetes/apiserver
  KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
  KUBE_API_PORT="--port=8080"
  KUBELET_PORT="--kubelet-port=10250"
  KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"
  KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
  KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
  KUBE_API_ARGS=""
  ```
  run etcd、kube-apiserver、kube-controller-manager、kube-scheduler service，and change to enabled.
  ```
  # for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do systemctl restart $SERVICES;systemctl enable       $SERVICES;systemctl status $SERVICES ; done
  ```
  define flannel network
  ```
  # etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'
  ```
  *  **node_configuration:**
  ```
  # yum -y install flannel kubernetes-node
  ```
  ```
  /etc/sysconfig/flanneld
  FLANNEL_ETCD="http://<master_ip>:2379"
  FLANNEL_ETCD_KEY="/atomic.io/network"

  /etc/kubernetes/config
  KUBE_LOGTOSTDERR="--logtostderr=true"
  KUBE_LOG_LEVEL="--v=0"
  KUBE_ALLOW_PRIV="--allow-privileged=false"
  KUBE_MASTER="--master=http://<master_ip>:8080"

  /etc/kubernetes/kubelet
　　node1:
  KUBELET_ADDRESS="--address=0.0.0.0"
  KUBELET_PORT="--port=10250"
  KUBELET_HOSTNAME="--hostname-override=<node1_ip>"
  KUBELET_API_SERVER="--api-servers=http://<master_ip>:8080"
  KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
  KUBELET_ARGS=""
　　node2:
  KUBELET_ADDRESS="--address=0.0.0.0"
  KUBELET_PORT="--port=10250"
  KUBELET_HOSTNAME="--hostname-override=<node2_ip>"
  KUBELET_API_SERVER="--api-servers=http://<master_ip>:8080"
  KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
  KUBELET_ARGS=""
  ```
  all node run kube-proxy,kubelet,docker,flanneld service，and change to enabled.
  ```
  # for SERVICES in kube-proxy kubelet docker flanneld;do systemctl restart $SERVICES;systemctl enable $SERVICES;systemctl status $SERVICES; done
  ```

# deployment nginx cluster
