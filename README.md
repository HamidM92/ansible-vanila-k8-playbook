# Kubernetes Node Preparation using Ansible (AWX Ready)

## 📌 Overview

This repository provides an automated approach to prepare RPM-based Linux servers (Rocky / RHEL / CentOS) for a **vanilla Kubernetes (kubeadm)** installation using **Ansible**.

The playbook is designed to be used with:

* Ansible AWX / Ansible Automation Platform
* Execution Environments (containerized Ansible runtime)
* Static or dynamic inventories

---

## 🎯 Purpose

Preparing Kubernetes nodes manually is error-prone and inconsistent.
This automation ensures:

* Standardized configuration across all nodes
* Faster provisioning
* Idempotent and repeatable deployments
* Alignment with Kubernetes best practices

---

## ⚙️ What This Automation Does

The playbook performs the following steps:

### 1. Disable SELinux

* Sets SELinux to permissive mode (runtime + persistent)
* Required for Kubernetes networking compatibility

---

### 2. Disable Swap

* Disables swap immediately
* Removes swap entries from `/etc/fstab`

📌 Kubernetes requires swap to be disabled for proper scheduling.

---

### 3. Disable Firewalld

* Stops and disables firewalld service

📌 In production, consider replacing this with properly defined firewall rules.

---

### 4. Enable IPv4 Forwarding

* Configures:

  ```
  net.ipv4.ip_forward = 1
  ```
* Applies sysctl settings without reboot

📌 Required for pod-to-pod networking.

---

### 5. Load Required Kernel Modules

* Configures persistent modules:

  * `overlay`
  * `br_netfilter`
* Loads modules at runtime using `modprobe`

📌 Required for container networking and CNI plugins.

---

### 6. Install Container Runtime (containerd)

* Installs `containerd`
* Enables and starts the service

📌 containerd is the recommended runtime for Kubernetes.

---

### 7. Configure Containerd (Systemd Cgroup)

The playbook ensures:

* `/etc/containerd` directory exists
* `config.toml` is generated (or overwritten)
* `SystemdCgroup = true` is enforced

📌 This is critical for compatibility with kubelet.

---

### 8. Install Kubernetes Components

* Adds Kubernetes repository dynamically
* Installs:

  * `kubelet`
  * `kubeadm`
  * `kubectl`

---

### 9. Enable Kubelet

* Enables and starts kubelet service

---

## 🔧 Kubernetes Version Configuration

The Kubernetes version is controlled via the variable:

```yaml
k8s_version: "v1.35"
```

### 🔁 Customizing Version

To install a different version, update this value:

```yaml
k8s_version: "v1.34"
```

📌 This affects:

* Repository URL
* Installed package versions

---

## 🌐 Network & DNS Requirements

Before running this automation, ensure:

### ✅ DNS Configuration

All nodes (masters & workers) must have:

* Proper `/etc/resolv.conf`
* Reachable DNS servers
* Ability to resolve external domains

Example:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

---

### ✅ Connectivity Requirements

Nodes must be able to:

* Reach Kubernetes repo:

  ```
  pkgs.k8s.io
  ```
* Reach container repositories
* Resolve DNS properly

---

### ⚠️ Enterprise Note

In restricted environments (e.g. banking / enterprise):

* Use internal YUM repositories
* Configure HTTP proxy if required
* Avoid public internet dependency

---

## 🤖 Ansible Design Logic

This playbook follows **idempotent design principles**:

### ✔️ State-driven configuration

Instead of checking conditions, the playbook:

* Enforces desired state
* Rewrites configs if needed
* Ensures consistency across nodes

---

### ✔️ Containerd configuration strategy

Regardless of system state:

* Directory is created if missing
* Config is always generated fresh
* Required parameters are enforced

This avoids:

* Partial configs
* Drift between nodes
* Hidden inconsistencies

---

### ✔️ Minimal dependencies

Only required collection:

```yaml
collections:
  - ansible.posix
```

Everything else uses `ansible.builtin` modules.

---

## 🚀 Usage (AWX)

1. Create a **Project** pointing to this repository
2. Create **Inventory** (group: `k8s_nodes`)
3. Add **Machine Credential** (SSH)
4. Create **Job Template**:

   * Playbook: `k8.yaml`
   * Inventory: your inventory
   * Execution Environment: your custom EE

---

## 📦 Execution Environment

This playbook is designed to run inside a custom EE built using:

* `ansible-builder`
* Required collections
* Python dependencies

---

## 🧪 Validation After Run

On each node, verify:

```bash
lsmod | grep overlay
lsmod | grep br_netfilter
systemctl status containerd
systemctl status kubelet
```

---

## 🏁 Summary

This automation provides:

* Fully prepared Kubernetes nodes
* Consistent configuration
* AWX-ready execution
* Production-aligned best practices

---

## 👨‍💻 Author Notes

* Designed for platform engineering workflows
* Optimized for AWX / Execution Environments
* Built with enterprise environments in mind

---
