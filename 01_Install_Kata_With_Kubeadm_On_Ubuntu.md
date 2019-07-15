## How install KataContainer with Kubeadm on Ubuntu

### 1. Install containerd

Confirm the containerd is exist or not.
```
$ sudo systemctl status containerd            
Unit containerd.service could not be found.
$ sudo apt-get install containerd
....
$ sudo systemctl status containerd                                                       ✔  793  10:46:32 
  ● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2019-07-15 10:46:30 JST; 24s ago
       Docs: https://containerd.io
    Process: 710 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 720 (containerd)
      Tasks: 15
     CGroup: /system.slice/containerd.service
             └─720 /usr/bin/containerd
  
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.131068353+09:00" level=info msg="Get image filesystem path "/va
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.132353950+09:00" level=error msg="Failed to load cni during ini
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134534543+09:00" level=info msg="loading plugin "io.containerd.
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134600451+09:00" level=info msg="Start subscribing containerd e
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134667526+09:00" level=info msg="Start recovering state"
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134769729+09:00" level=info msg="Start event monitor"
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134786199+09:00" level=info msg="Start snapshots syncer"
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134795330+09:00" level=info msg="Start streaming server"
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134822857+09:00" level=info msg=serving... address="/run/contai
  7月 15 10:46:30 chengchen-ubuntu containerd[720]: time="2019-07-15T10:46:30.134840173+09:00" level=info msg="containerd successfully booted
$ sudo systemctl enable containerd
$ sudo cat /etc/containerd/config.toml
```

### 2. Install Kata container

### 3. Install Kubeadm

### 4. Run Kubeadm
```
$ sudo kubeadm init --cri-socket /run/containerd/containerd.sock --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.15.0
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
	[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
	[ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
``` 

Solve the `[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist`
```
$ sudo modprobe br_netfilter 
```
[Refer Link](https://github.com/weaveworks/weave/issues/2789)

Solve the `[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1`
```
$ sudo bash -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
$ sudo sysctl -p
```
[Refer Link](https://askubuntu.com/questions/783017/bash-proc-sys-net-ipv4-ip-forward-permission-denied)

Solve the swap
```
sudo swapoff -a
```

Then Init the Kubeadm again. It is **WORKED**!!!
```
$ sudo kubeadm init --cri-socket /run/containerd/containerd.sock --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.15.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [chengchen-ubuntu kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.107]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [chengchen-ubuntu localhost] and IPs [192.168.1.107 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [chengchen-ubuntu localhost] and IPs [192.168.1.107 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.503733 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.15" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node chengchen-ubuntu as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node chengchen-ubuntu as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 8yz7nr.o3idp6dd5xqtxgqz
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.107:6443 --token 8yz7nr.o3idp6dd5xqtxgqz \
    --discovery-token-ca-cert-hash sha256:1ad017b6273a11810ed75eb57bc0bc81e680a8dcc7f1124e05f386f5ec76513b
``` 

Confirm Kubernetes cluster status.
```
$ export KUBECONFIG=/etc/kubernetes/admin.conf
$ sudo -E kubectl get nodes
$ sudo -E kubectl get pods
```

### 5. Install a Pod Network
A pod network plugin is needed to allow pods to communicate with each other.

+ Install the `flannel` plugin by following the Using `kubeadm` to Create a Cluster guide, starting from the **Installing a pod network** section.

+ Create a pod network using flannel

> Note: There is no known way to determine programmatically the best version (commit) to use. See https://github.com/coreos/flannel/issues/995.

```
$ sudo -E kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.extensions/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created
```

+ Wait for the pod network to become available

```
# number of seconds to wait for pod network to become available
$ timeout_dns=420
$ while [ "$timeout_dns" -gt 0 ]; do
    if sudo -E kubectl get pods --all-namespaces | grep dns | grep Running; then
        break
    fi

    sleep 1s
    ((timeout_dns--))
 done
```

+ Check the pod network is running

```
$ sudo -E kubectl get pods --all-namespaces | grep dns | grep Running && echo "OK" || ( echo "FAIL" && false )
kube-system   coredns-5c98db65d4-84xqm                   1/1     Running   0          22m
kube-system   coredns-5c98db65d4-sl2sm                   1/1     Running   0          22m
OK
```

### Allow pods to run in the master node
By default, the cluster will not schedule pods in the master node. To enable master node scheduling:
```
$ sudo -E kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Create an untrusted pod using Kata Containers
By default, all pods are created with the default runtime configured in CRI containerd plugin.

If a pod has the `io.kubernetes.cri.untrusted-workload` annotation set to `"true"`, the CRI plugin runs the pod with the Kata Containers runtime.

+ Create an untrusted pod configuration
```
$ cat << EOT | tee nginx-untrusted.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-untrusted
  annotations:
    io.kubernetes.cri.untrusted-workload: "true"
spec:
  containers:
  - name: nginx
    image: nginx
    
EOT
```

### When you want to reset the kubeadm
The kubeadm resetting is very difficult that I spend whole day to solve some problems.
```
$ sudo kubeadm reset
$ sudo rm -rf ~/.minikube ~/.kube /etc/kubernetes /var/lib/kubelet /var/lib/etcd /var/lib/cni /etc/cni/net.d/*
$ sudo systemctl stop kubelet 
$ sudo ip link delete cni0
$ sudo ip link delete flannel1
```

### About config.toml
If you deleted the config.toml, you can recovery it by this command.
```
$ sudo containerd config default > /etc/containerd/config.toml
```

Don't forgot update the setting.

```
$ sudo cat /etc/containerd/config.toml                                      
root = "/var/lib/containerd"
state = "/run/containerd"
oom_score = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216

[debug]
  address = ""
  uid = 0
  gid = 0
  level = ""

[metrics]
  address = ""
  grpc_histogram = false

[cgroup]
  path = ""

[plugins]
  [plugins.cgroups]
    no_prometheus = false
  [plugins.cri]
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    enable_selinux = false
    sandbox_image = "k8s.gcr.io/pause:3.1"
    stats_collect_period = 10
    systemd_cgroup = false
    enable_tls_streaming = false
    max_container_log_line_size = 16384
    [plugins.cri.containerd]
      snapshotter = "overlayfs"
      no_pivot = false
      [plugins.cri.containerd.default_runtime]
        runtime_type = "io.containerd.runtime.v1.linux"
        runtime_engine = ""
        runtime_root = ""
      [plugins.cri.containerd.untrusted_workload_runtime]
        runtime_type = "io.containerd.runtime.v1.linux"  # <- Change here
        runtime_engine = "/usr/bin/kata-runtime"         # <- Change here
        runtime_root = ""
    [plugins.cri.cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
    [plugins.cri.registry]
      [plugins.cri.registry.mirrors]
        [plugins.cri.registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]
    [plugins.cri.x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
  [plugins.diff-service]
    default = ["walking"]
  [plugins.linux]
    shim = "containerd-shim"
    runtime = "runc"
    runtime_root = ""
    no_shim = false
    shim_debug = false
  [plugins.opt]
    path = "/opt/containerd"
  [plugins.restart]
    interval = "10s"
  [plugins.scheduler]
    pause_threshold = 0.02
    deletion_threshold = 0
    mutation_threshold = 100
    schedule_delay = "0s"
    startup_delay = "100ms"
```