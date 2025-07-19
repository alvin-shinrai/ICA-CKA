
# Kubernetes Cluster Architecture, Installation and Configuration - Complete CKA Guide

## Table of Contents

1. [Kubernetes Cluster Architecture]
    - [Control Plane Components]
    - [Node Components]
    - [Add-on Components]
2. [Role-Based Access Control (RBAC)]
    - [RBAC Components]
    - [Users and Service Accounts]
    - [Roles and ClusterRoles]
    - [RoleBindings and ClusterRoleBindings]
    - [Testing RBAC]
3. [Infrastructure Preparation]
    - [System Requirements]
    - [Container Runtime]
    - [Network Configuration]
    - [Firewall and Security]
4. [Cluster Installation with kubeadm]
    - [Installing kubeadm, kubelet, and kubectl]
    - [Initializing the Control Plane]
    - [Joining Worker Nodes]
    - [Installing CNI Plugin]
5. [Cluster Lifecycle Management]
    - [Upgrading Clusters]
    - [Backing Up etcd]
    - [Certificate Management]
    - [Node Management]
6. [High Availability Control Plane]
    - [External etcd Topology]
    - [Stacked etcd Topology]
    - [Load Balancer Configuration]
7. [Extension Interfaces]
    - [Container Network Interface (CNI)]
    - [Container Storage Interface (CSI)]
    - [Container Runtime Interface (CRI)]
8. [Custom Resource Definitions (CRDs) and Operators]
    - [Creating CRDs]
    - [Installing Operators]
    - [Operator Lifecycle Manager]
9. [Helm and Kustomize]
    - [Helm Basics]
    - [Kustomize Basics]
    - [Installing Cluster Components]
10. [CKA Exam Tips and Common Scenarios](https://claude.ai/chat/735036d1-82e1-464d-a05f-d24734c36c31#cka-exam-tips-and-common-scenarios)
11. [Troubleshooting Commands](https://claude.ai/chat/735036d1-82e1-464d-a05f-d24734c36c31#troubleshooting-commands)

---

## Kubernetes Cluster Architecture

### Control Plane Components

The control plane manages the cluster and makes global decisions about the cluster.

**API Server (kube-apiserver)**

- Central management entity and front-end for the cluster
- Exposes Kubernetes API
- All communication goes through API server

```bash
# Check API server status
kubectl get componentstatuses
systemctl status kubelet
journalctl -u kubelet
```

**etcd**

- Distributed key-value store
- Stores all cluster data
- Single source of truth for cluster state

```bash
# Check etcd health
kubectl get pods -n kube-system | grep etcd
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  endpoint health
```

**Controller Manager (kube-controller-manager)**

- Runs controller processes
- Node controller, Replication controller, Endpoints controller, etc.

```bash
# Check controller manager
kubectl get pods -n kube-system | grep controller-manager
kubectl logs -n kube-system kube-controller-manager-master
```

**Scheduler (kube-scheduler)**

- Assigns Pods to nodes based on resource requirements
- Considers constraints, affinity rules, data locality

```bash
# Check scheduler
kubectl get pods -n kube-system | grep scheduler
kubectl logs -n kube-system kube-scheduler-master
```

### Node Components

Components that run on every node, maintaining running pods.

**kubelet**

- Primary node agent
- Communicates with API server
- Manages Pod lifecycle

```bash
# Check kubelet status
systemctl status kubelet
journalctl -u kubelet --since "1 hour ago"
```

**kube-proxy**

- Network proxy running on each node
- Maintains network rules for service communication

```bash
# Check kube-proxy
kubectl get pods -n kube-system | grep kube-proxy
kubectl logs -n kube-system kube-proxy-xyz
```

**Container Runtime**

- Software responsible for running containers
- Docker, containerd, CRI-O

```bash
# Check container runtime
crictl version
crictl pods
crictl images
```

### Add-on Components

**CoreDNS**

- DNS server for service discovery

```bash
kubectl get pods -n kube-system | grep coredns
kubectl get configmap coredns -n kube-system
```

**CNI Plugin**

- Provides Pod networking

```bash
ls /etc/cni/net.d/
kubectl get pods -n kube-system | grep -E "calico|flannel|weave"
```

---

## Role-Based Access Control (RBAC)

RBAC regulates access to Kubernetes resources based on the roles of individual users.

### RBAC Components

**Four main RBAC objects:**

1. **Role/ClusterRole** - Define permissions
2. **RoleBinding/ClusterRoleBinding** - Grant permissions to subjects
3. **User** - Human users (external to Kubernetes)
4. **ServiceAccount** - Application accounts (Kubernetes objects)

### Users and Service Accounts

**Create ServiceAccount:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
```

```bash
# Create ServiceAccount
kubectl create serviceaccount my-service-account

# Get ServiceAccount details
kubectl get serviceaccount my-service-account -o yaml
kubectl describe serviceaccount my-service-account
```

**ServiceAccount with custom Secret:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-sa
  namespace: default

---
apiVersion: v1
kind: Secret
metadata:
  name: my-sa-token
  namespace: default
  annotations:
    kubernetes.io/service-account.name: my-sa
type: kubernetes.io/service-account-token
```

### Roles and ClusterRoles

**Role** (namespace-scoped):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]          # Core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
```

**ClusterRole** (cluster-wide):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "nodes"]
  verbs: ["get", "watch", "list"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["*"]              # All verbs
```

**Advanced Role with specific resources:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-manager
  namespace: production
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
  resourceNames: ["allowed-config", "allowed-secret"]  # Specific resources only
```

### RoleBindings and ClusterRoleBindings

**RoleBinding** (grants Role in specific namespace):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRoleBinding** (grants ClusterRole cluster-wide):

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
- kind: User
  name: admin-user
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: admin-sa
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRole bound to specific namespace:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-role-in-namespace
  namespace: development
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole          # Can bind ClusterRole to namespace
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Testing RBAC

**Check permissions:**

```bash
# Check what you can do
kubectl auth can-i "*" "*"
kubectl auth can-i create deployments
kubectl auth can-i get pods --namespace=default

# Check what specific user can do
kubectl auth can-i get pods --as=jane
kubectl auth can-i create deployments --as=system:serviceaccount:default:my-sa

# Check in specific namespace
kubectl auth can-i get pods --as=jane --namespace=production
```

**Impersonate user:**

```bash
# Test as specific user
kubectl get pods --as=jane
kubectl get deployments --as=system:serviceaccount:default:my-sa

# Test with groups
kubectl get pods --as=jane --as-group=system:authenticated
```

**Common RBAC commands:**

```bash
# Create Role
kubectl create role pod-reader --verb=get,list,watch --resource=pods

# Create ClusterRole
kubectl create clusterrole deployment-reader --verb=get,list,watch --resource=deployments

# Create RoleBinding
kubectl create rolebinding jane-pod-reader --role=pod-reader --user=jane

# Create ClusterRoleBinding
kubectl create clusterrolebinding jane-cluster-admin --clusterrole=cluster-admin --user=jane

# Create ServiceAccount and bind
kubectl create serviceaccount my-sa
kubectl create rolebinding my-sa-binding --role=pod-reader --serviceaccount=default:my-sa
```

---

## Infrastructure Preparation

### System Requirements

**Minimum requirements per node:**

- 2 GB RAM
- 2 CPUs
- 20 GB disk space
- Network connectivity between all nodes
- Unique hostname, MAC address, and product_uuid

**Pre-installation checks:**

```bash
# Check system resources
free -h
nproc
df -h

# Check network connectivity
ping <other-node-ip>

# Check unique identifiers
cat /sys/class/dmi/id/product_uuid
ip link show
hostname
```

### Container Runtime

**Install containerd:**

```bash
# Install containerd
wget https://github.com/containerd/containerd/releases/download/v1.7.8/containerd-1.7.8-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.8-linux-amd64.tar.gz

# Install runc
wget https://github.com/opencontainers/runc/releases/download/v1.1.9/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

# Install CNI plugins
wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.3.0.tgz

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable systemd cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Start containerd
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

### Network Configuration

**Disable swap:**

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

**Configure kernel modules:**

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

**Set sysctl parameters:**

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

### Firewall and Security

**Required ports:**

**Control Plane:**

- 6443: Kubernetes API server
- 2379-2380: etcd server client API
- 10250: kubelet API
- 10259: kube-scheduler
- 10257: kube-controller-manager

**Worker Nodes:**

- 10250: kubelet API
- 30000-32767: NodePort services

```bash
# Configure firewall (example for Ubuntu/ufw)
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10259/tcp
sudo ufw allow 10257/tcp
sudo ufw allow 30000:32767/tcp
```

---

## Cluster Installation with kubeadm

### Installing kubeadm, kubelet, and kubectl

**Ubuntu/Debian:**

```bash
# Update package index
sudo apt-get update

# Install packages needed for apt repository
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Add Kubernetes apt repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubelet, kubeadm and kubectl
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Enable kubelet
sudo systemctl enable --now kubelet
```

**CentOS/RHEL:**

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

### Initializing the Control Plane

**Basic initialization:**

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

**Advanced initialization:**

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.1.10 \
  --apiserver-cert-extra-sans=master.example.com \
  --pod-network-cidr=192.168.0.0/16 \
  --service-cidr=10.96.0.0/12 \
  --kubernetes-version=v1.28.0 \
  --cri-socket=unix:///var/run/containerd/containerd.sock
```

**Configuration file approach:**

```yaml
# kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.1.10
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock

---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.28.0
apiServer:
  advertiseAddress: 192.168.1.10
  certSANs:
  - master.example.com
  - 192.168.1.10
controlPlaneEndpoint: "master.example.com:6443"
networking:
  serviceSubnet: "10.96.0.0/12"
  podSubnet: "192.168.0.0/16"
  dnsDomain: "cluster.local"
etcd:
  local:
    dataDir: "/var/lib/etcd"
```

```bash
sudo kubeadm init --config=kubeadm-config.yaml
```

**Post-installation setup:**

```bash
# Configure kubectl for regular user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Verify installation
kubectl get nodes
kubectl get pods -n kube-system
```

### Joining Worker Nodes

**From control plane, get join command:**

```bash
kubeadm token create --print-join-command
```

**On worker nodes:**

```bash
sudo kubeadm join 192.168.1.10:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

**Manual token creation (if needed):**

```bash
# Create token
kubeadm token create

# Get CA cert hash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

### Installing CNI Plugin

**Install Calico:**

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

**Install Flannel:**

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

**Verify CNI installation:**

```bash
kubectl get pods -n kube-system
kubectl get nodes
# Nodes should show Ready status after CNI is installed
```

---

## Cluster Lifecycle Management

### Upgrading Clusters

**Check available versions:**

```bash
apt update
apt-cache madison kubeadm
# or for yum: yum list --showduplicates kubeadm --disableexcludes=kubernetes
```

**Upgrade control plane (first master):**

```bash
# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.28.1-00
sudo apt-mark hold kubeadm

# Verify version
kubeadm version

# Plan upgrade
sudo kubeadm upgrade plan

# Apply upgrade
sudo kubeadm upgrade apply v1.28.1

# Drain node
kubectl drain master --ignore-daemonsets --delete-emptydir-data

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.28.1-00 kubectl=1.28.1-00
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Uncordon node
kubectl uncordon master
```

**Upgrade additional control plane nodes:**

```bash
sudo kubeadm upgrade node
# Then follow same kubelet/kubectl upgrade steps
```

**Upgrade worker nodes:**

```bash
# On worker node
sudo apt-mark unhold kubeadm
sudo apt-get update && sudo apt-get install -y kubeadm=1.28.1-00
sudo apt-mark hold kubeadm

sudo kubeadm upgrade node

# From control plane
kubectl drain worker1 --ignore-daemonsets --delete-emptydir-data

# On worker node
sudo apt-mark unhold kubelet kubectl
sudo apt-get update && sudo apt-get install -y kubelet=1.28.1-00 kubectl=1.28.1-00
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# From control plane
kubectl uncordon worker1
```

### Backing Up etcd

**Direct etcd backup:**

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  snapshot save /backup/etcd-snapshot-$(date +%Y%m%d_%H%M%S).db

# Verify backup
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /backup/etcd-snapshot-*.db
```

**Restore etcd backup:**

```bash
# Stop API server
sudo systemctl stop kubelet
sudo docker stop $(sudo docker ps -f name=k8s_kube-apiserver* -q)

# Restore snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --name master \
  --initial-cluster master=https://192.168.1.10:2380 \
  --initial-advertise-peer-urls https://192.168.1.10:2380 \
  --data-dir /var/lib/etcd-backup

# Update etcd configuration to use backup directory
sudo mv /var/lib/etcd /var/lib/etcd.old
sudo mv /var/lib/etcd-backup /var/lib/etcd

# Start services
sudo systemctl start kubelet
```

### Certificate Management

**Check certificate expiration:**

```bash
kubeadm certs check-expiration
```

**Renew certificates:**

```bash
# Renew all certificates
sudo kubeadm certs renew all

# Renew specific certificate
sudo kubeadm certs renew apiserver

# Update kubeconfig files
sudo kubeadm init phase kubeconfig all
```

**Manual certificate generation:**

```bash
# Generate new CA
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -subj "/CN=kubernetes-ca" -out ca.crt

# Generate client certificate
openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj "/CN=admin/O=system:masters" -out client.csr
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365
```

### Node Management

**Add new node:**

```bash
# Generate new token
kubeadm token create --print-join-command

# On new node, join cluster
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

**Remove node:**

```bash
# Drain node
kubectl drain worker2 --ignore-daemonsets --delete-emptydir-data --force

# Delete node
kubectl delete node worker2

# On the node being removed
sudo kubeadm reset
sudo rm -rf /etc/kubernetes/
sudo rm -rf /var/lib/kubelet/
sudo rm -rf /var/lib/etcd/
```

---

## High Availability Control Plane

### External etcd Topology

**etcd cluster setup (on separate nodes):**

```bash
# On each etcd node
export ETCD_NAME=etcd1  # etcd2, etcd3 for other nodes
export ETCD_IP=192.168.1.20  # Different IP for each node

etcd --name ${ETCD_NAME} \
  --initial-advertise-peer-urls https://${ETCD_IP}:2380 \
  --listen-peer-urls https://${ETCD_IP}:2380 \
  --advertise-client-urls https://${ETCD_IP}:2379 \
  --listen-client-urls https://${ETCD_IP}:2379,https://127.0.0.1:2379 \
  --initial-cluster etcd1=https://192.168.1.20:2380,etcd2=https://192.168.1.21:2380,etcd3=https://192.168.1.22:2380 \
  --initial-cluster-state new \
  --initial-cluster-token my-etcd-token
```

**kubeadm config with external etcd:**

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.28.0
controlPlaneEndpoint: "loadbalancer.example.com:6443"
etcd:
  external:
    endpoints:
    - https://192.168.1.20:2379
    - https://192.168.1.21:2379
    - https://192.168.1.22:2379
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```

### Stacked etcd Topology

**First control plane node:**

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.28.0
controlPlaneEndpoint: "loadbalancer.example.com:6443"
apiServer:
  certSANs:
  - "loadbalancer.example.com"
  - "192.168.1.100"
  - "192.168.1.101" 
  - "192.168.1.102"
```

```bash
sudo kubeadm init --config=kubeadm-config.yaml --upload-certs
```

**Additional control plane nodes:**

```bash
sudo kubeadm join loadbalancer.example.com:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <certificate-key>
```

### Load Balancer Configuration

**HAProxy configuration:**

```
global
    daemon

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend kubernetes-apiserver
    bind *:6443
    mode tcp
    option tcplog
    default_backend kubernetes-apiserver

backend kubernetes-apiserver
    mode tcp
    option tcp-check
    balance roundrobin
    server master1 192.168.1.100:6443 check fall 3 rise 2
    server master2 192.168.1.101:6443 check fall 3 rise 2
    server master3 192.168.1.102:6443 check fall 3 rise 2
```

**NGINX configuration:**

```
stream {
    upstream kubernetes {
        server 192.168.1.100:6443;
        server 192.168.1.101:6443;
        server 192.168.1.102:6443;
    }
    
    server {
        listen 6443;
        proxy_pass kubernetes;
    }
}
```

---

## Extension Interfaces

### Container Network Interface (CNI)

**CNI configuration structure:**

```bash
ls /etc/cni/net.d/
cat /etc/cni/net.d/10-calico.conflist
```

**Sample CNI configuration:**

```json
{
  "cniVersion": "0.3.1",
  "name": "calico",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "nodename": "node1",
      "mtu": 1440,
      "ipam": {
        "type": "calico-ipam"
      },
      "policy": {
        "type": "k8s"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    }
  ]
}
```

### Container Storage Interface (CSI)

**CSI driver components:**

- CSI Controller Plugin (StatefulSet)
- CSI Node Plugin (DaemonSet)
- CSI Driver object

**Example CSI Driver installation:**

```yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: example.csi.driver
spec:
  attachRequired: true
  podInfoOnMount: true
  volumeLifecycleModes:
  - Persistent
  - Ephemeral
```

**Check CSI drivers:**

```bash
kubectl get csidrivers
kubectl get csinodes
kubectl describe csinode <node-name>
```

### Container Runtime Interface (CRI)

**CRI implementations:**

- containerd
- CRI-O
- Docker (via dockershim - deprecated)

**CRI commands with crictl:**

```bash
# Configure crictl
cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
EOF

# CRI operations
crictl version
crictl info
crictl pods
crictl ps
crictl images
crictl pull nginx:latest
crictl rmi nginx:latest
```

---

## Custom Resource Definitions (CRDs) and Operators

### Creating CRDs

**Simple CRD:**

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: webapps.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer
                minimum: 1
                maximum: 10
              image:
                type: string
          status:
            type: object
            properties:
              availableReplicas:
                type: integer
    subresources:
      status: {}
      scale:
        specReplicasPath: .spec.replicas
        statusReplicasPath: .status.availableReplicas
  scope: Namespaced
  names:
    plural: webapps
    singular: webapp
    kind: WebApp
    shortNames:
    - wa
```

**Custom Resource instance:**

```yaml
apiVersion: example.com/v1
kind: WebApp
metadata:
  name: my-webapp
  namespace: default
spec:
  replicas: 3
  image: nginx:1.20
```

**CRD with validation:**

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        required: ["spec"]
        properties:
          spec:
            type: object
            required: ["engine", "version"]
            properties:
              engine:
                type: string
                enum: ["mysql", "postgresql", "mongodb"]
              version:
                type: string
                pattern: '^[0-9]+\.[0-9]+(\.[0-9]+)?$'
              storage:
                type: string
                pattern: '^[0-9]+Gi$'
              replicas:
                type: integer
                minimum: 1
                maximum: 5
                default: 1
          status:
            type: object
            properties:
              phase:
                type: string
                enum: ["Pending", "Running", "Failed"]
              message:
                type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

### Installing Operators

**Manual operator deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-operator
  namespace: webapp-system
spec:
  replicas: 1
  selector:
    matchLabels:
      name: webapp-operator
  template:
    metadata:
      labels:
        name: webapp-operator
    spec:
      serviceAccountName: webapp-operator
      containers:
      - name: operator
        image: webapp-operator:v1.0.0
        command:
        - webapp-operator
        env:
        - name: WATCH_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: OPERATOR_NAME
          value: "webapp-operator"
```

**Operator RBAC:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webapp-operator
  namespace: webapp-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: webapp-operator
rules:
- apiGroups: ["example.com"]
  resources: ["webapps"]
  verbs: ["*"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["*"]
- apiGroups: [""]
  resources: ["services", "configmaps"]
  verbs: ["*"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: webapp-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: webapp-operator
subjects:
- kind: ServiceAccount
  name: webapp-operator
  namespace: webapp-system
```

### Operator Lifecycle Manager

**Install OLM:**

```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.25.0/install.sh | bash -s v0.25.0
```

**Check OLM installation:**

```bash
kubectl get csv -A
kubectl get catalogsource -A
kubectl get subscription -A
```

---

## Helm and Kustomize

### Helm Basics

**Install Helm:**

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

**Basic Helm operations:**

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search charts
helm search repo nginx
helm search hub prometheus

# Install chart
helm install my-nginx bitnami/nginx
helm install my-release bitnami/apache --set replicaCount=3

# List releases
helm list
helm list -A

# Get release information
helm get values my-nginx
helm get manifest my-nginx
helm status my-nginx

# Upgrade release
helm upgrade my-nginx bitnami/nginx --set service.type=NodePort

# Rollback release
helm rollback my-nginx 1

# Uninstall release
helm uninstall my-nginx
```

**Custom values file:**

```yaml
# values.yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.20"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
  nodePort: 30080

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
```

```bash
helm install my-app bitnami/nginx -f values.yaml
```

### Kustomize Basics

**Basic kustomization.yaml:**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml
- configmap.yaml

namePrefix: prod-
nameSuffix: -v1

commonLabels:
  environment: production
  team: platform

images:
- name: nginx
  newTag: 1.20-alpine

replicas:
- name: webapp-deployment
  count: 5

configMapGenerator:
- name: app-config
  files:
  - app.properties
  - database.conf

secretGenerator:
- name: app-secrets
  literals:
  - username=admin
  - password=secret123
```

**Directory structure:**

```
app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── development/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── production/
        ├── kustomization.yaml
        └── patch.yaml
```

**Base kustomization:**

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml

commonLabels:
  app: webapp
```

**Production overlay:**

```yaml
# overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../base

namePrefix: prod-

replicas:
- name: webapp
  count: 5

patches:
- patch.yaml

images:
- name: webapp
  newTag: v1.2.0
```

**Apply with kubectl:**

```bash
# Apply base
kubectl apply -k base/

# Apply overlay
kubectl apply -k overlays/production/

# View generated YAML
kubectl kustomize overlays/production/
```

### Installing Cluster Components

**Install metrics-server with Helm:**

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm install metrics-server metrics-server/metrics-server \
  --set args="{--kubelet-insecure-tls}"
```

**Install ingress-nginx with Helm:**

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --set controller.service.type=NodePort
```

**Install cert-manager with Kustomize:**

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

```bash
kubectl apply -k .
```

---

## CKA Exam Tips and Common Scenarios

### Most Important Topics for CKA:

1. **RBAC** - Creating roles, bindings, testing permissions
2. **kubeadm** - Cluster installation and upgrades
3. **etcd backup/restore** - Cluster data protection
4. **High availability** - Multi-master setups
5. **Troubleshooting** - Component issues and logs

### Common Exam Scenarios:

**Scenario 1: Create RBAC for developer**

```bash
# Create namespace
kubectl create namespace development

# Create ServiceAccount
kubectl create serviceaccount developer -n development

# Create Role
kubectl create role pod-manager \
  --verb=get,list,watch,create,update,patch,delete \
  --resource=pods \
  -n development

# Create RoleBinding
kubectl create rolebinding developer-binding \
  --role=pod-manager \
  --serviceaccount=development:developer \
  -n development

# Test permissions
kubectl auth can-i get pods --as=system:serviceaccount:development:developer -n development
```

**Scenario 2: Backup and restore etcd**

```bash
# Backup
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key

# Restore (in exam, you may need to restore to different location)
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
  --data-dir /var/lib/etcd-from-backup
```

**Scenario 3: Upgrade cluster with kubeadm**

```bash
# Check current version
kubectl get nodes

# Plan upgrade
sudo kubeadm upgrade plan

# Upgrade control plane
sudo kubeadm upgrade apply v1.28.1

# Upgrade kubelet
sudo apt-get update && sudo apt-get install -y kubelet=1.28.1-00
sudo systemctl restart kubelet
```

---

## Troubleshooting Commands

### Cluster Component Status:

```bash
# Check all components
kubectl get componentstatuses
kubectl get nodes
kubectl get pods -n kube-system

# Check specific components
systemctl status kubelet
journalctl -u kubelet --since "1 hour ago"
systemctl status containerd

# Check logs
kubectl logs -n kube-system kube-apiserver-master
kubectl logs -n kube-system kube-controller-manager-master
kubectl logs -n kube-system kube-scheduler-master
```

### RBAC Troubleshooting:

```bash
# Check permissions
kubectl auth can-i "*" "*"
kubectl auth can-i get pods --as=jane
kubectl auth can-i create deployments --as=system:serviceaccount:default:my-sa

# List RBAC objects
kubectl get roles,rolebindings -A
kubectl get clusterroles,clusterrolebindings
kubectl describe rolebinding my-binding
```

### Certificate Issues:

```bash
# Check certificate expiration
kubeadm certs check-expiration

# Check certificate details
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

# Verify certificate chain
openssl verify -CAfile /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/apiserver.crt
```

### etcd Troubleshooting:

```bash
# Check etcd health
ETCDCTL_API=3 etcdctl endpoint health \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key

# Check etcd status
ETCDCTL_API=3 etcdctl endpoint status \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
  --write-out=table
```

### Node Troubleshooting:

```bash
# Check node status
kubectl describe node <node-name>
kubectl get events --sort-by=.metadata.creationTimestamp

# Check container runtime
crictl version
crictl info
crictl pods
systemctl status containerd

# Check disk space and resources
df -h
free -h
top
```

---

_This guide covers all essential Kubernetes cluster architecture, installation, and configuration topics for the CKA exam. Focus on hands-on practice with kubeadm, RBAC, and etcd operations._