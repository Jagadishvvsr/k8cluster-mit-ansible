# 📌 Overview

The playbook provisions a Kubernetes cluster with:

* Fully configured Control Plane
* Multiple Worker Nodes
* Container runtime (containerd)
* CNI networking (Calico)
* Automated node join & validation

The architecture follows a standard Kubernetes design:

* Control Plane → Manages cluster state
* Worker Nodes → Run application workloads (Pods)

# 🧱 Cluster Architecture
* Control Plane Components
  * API Server
  * Controller Manager
  * Scheduler
  * etcd (cluster state store)
* Worker Node Components
  * kubelet
  * kube-proxy
  * container runtime (containerd)
  * Pods (application workloads)

# ⚙️ Roles & Responsibilities
**🔹 bootstrap

Prepares all nodes for Kubernetes compatibility:

* Disables swap
* Configures kernel modules (overlay, br_netfilter)
* Enables network settings for Kubernetes:
    * net.bridge.bridge-nf-call-iptables
    * IPv4/IPv6 forwarding
* Ensures iptables can see container traffic

** 📍 Executed on: All nodes

# 🔹 containerd

Installs and configures the container runtime:

* Installs containerd

* Generates default config:

          containerd config default 
* Updates:
  * SystemdCgroup = true (required for Kubernetes)
*Configures pause image (via variables)

** 📍 Executed on: All nodes

# 🔹 k8_components

Installs Kubernetes binaries:

   * Adds Kubernetes repository & GPG keys
   * Installs:
         * kubelet
         * kubeadm
         * kubectl
* Holds package versions (prevents unintended upgrades)
* Enables kubelet service

** 📍 Executed on: All nodes

# 📦 Version management:

group_vars/all/k8components.yaml

# 🔹 control_plane

Initializes the Kubernetes control plane:

* Runs:
      ** kubeadm init
* Uses variables:
     * Control_Plane_Private_IP
     * PodCidr
* Configures kubeconfig for the user
* Generates join token for worker nodes

** 📍 Executed on: Control plane nodes

# 🔹 cni

Configures cluster networking using Calico:

* Installs Tigera operator
* Applies custom Calico CR
* Waits for cluster readiness before applying networking

** 📍 Executed on: Control plane nodes

# 🔹 worker_nodes

Joins worker nodes to the cluster:
* Uses generated join_command
* Ensures idempotency:
  ** creates: /etc/kubernetes/kubelet.conf

**📍 Executed on: Worker nodes

# 🔹 pod_check

Validates cluster health:
* Checks pods in:
    ** kube-system namespace
* Ensures cluster is fully operational

** 📍 Executed on: Control plane

# 📂 Project Structure
.
├── roles/  \
│   ├── bootstrap/ \
│   ├── containerd/
│   ├── k8_components/
│   ├── control_plane/
│   ├── cni/
│   ├── worker_nodes/
│   └── pod_check/
├── group_vars/
│   └── all/
│       ├── k8components.yaml
│       ├── controlplane.yaml
│       ├── calicoversion.yaml
│       └── pauseImage.yaml
├── inventory/
│   └── hosts.ini
└── playbook.yaml

# 🧩 Key Features

✔ Modular role-based design
✔ Idempotent execution (safe re-runs)
✔ Centralized version management
✔ Multi-OS support (Debian & RedHat)
✔ Automated networking (Calico)
✔ Production-style kubeadm setup

# 📦 Requirements
* Ansible (>= 2.9 recommended)
* Kubernetes collection:
          ansible-galaxy collection install kubernetes.core
* SSH access to all nodes
* Proper inventory configuration

# 🚀 How to Run

     ** ansible-playbook -i inventory/hosts.ini playbook.yaml
     
# 📊 Variables

All configurable parameters are centralized:

* k8components.yaml → Kubernetes version
* controlplane.yaml → Control plane settings
* calicoversion.yaml → Calico version
* pauseImage.yaml → Container runtime pause image

# ✅ Output
After successful execution:

* Control plane initialized
* Worker nodes joined
* Networking configured
* Cluster validated (kube-system pods running)

# 💡 Use Cases
* Learning Kubernetes internals
* Dev/Test cluster provisioning
* Infrastructure automation practice
* CI/CD infrastructure setup
