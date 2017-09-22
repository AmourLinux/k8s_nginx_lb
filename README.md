# k8s_nginx_lb
From k8s deployment a loadbalance nginx cluster.
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
  *  **create pod**
  ```
  # kubectl create -f nginx-rc.yaml
  ```
  *  **create service**
  ```
  # kubectl create -f nginx-service-nodeport.yaml
  ```
  *  **view pod state**
  ```
  # kubectl get pods
  NAME                     READY     STATUS    RESTARTS   AGE
  nginx-controller-534rm   1/1       Running   1          3h
  nginx-controller-c3fzv   1/1       Running   1          3h
  ```
  *  **view pod desc**
  ```
  # kubectl describe pod nginx-controller-534rm
  ...
  ```
  *  **view service state**
  ```
  # kubectl get service
  NAME                     CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
  kubernetes               10.254.0.1     <none>        443/TCP          3h
  nginx-service-nodeport   10.254.2.144   <nodes>       8000:32177/TCP   3h
  ```
  *  **view service desc**
  ```
  # kubectl describe service nginx-service-nodeport
  ...
  ```
  
*  **We can use <node_ip>:port access nginx cluster , and achieve loadbalance:**
*  **In this test. node_ip is node1_ip or node2_ip , port is 32177(by #kubectl get service get).**
*  **By service_nodeport achieve nginx loadbalance cluster.**
*  **By rc achieve pod dynamic stretching.**

  
