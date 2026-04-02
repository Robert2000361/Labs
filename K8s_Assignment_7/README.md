# Kubernetes Advanced Scheduling Lab
## Scheduling Strategies, QoS, DaemonSets, and Static Pods

![Lab Architecture Placeholder](k8s-scheduling.png)

<p align="center">
  <b>Kubernetes Advanced Scheduling & Resource Management Lab</b>
</p>

This lab demonstrates how to implement and validate advanced **Kubernetes Scheduling & Resource Management** techniques at an intermediate-to-advanced level.

Instead of relying on default scheduling behavior, we explored how to precisely control **where Pods run**, **how they behave under pressure**, and **how system workloads are distributed across nodes**.

Key concepts covered in this lab:
- **Targeted Scheduling** (nodeName & nodeSelector)
- **Taints and Tolerations**
- **Node Affinity**
- **QoS Classes (Resource Management)**
- **DaemonSets (Infrastructure workloads)**
- **Static Pods (Control Plane-level management)**

The goal of this lab is to understand how Kubernetes makes intelligent scheduling decisions and how engineers can override or fine-tune those decisions for real-world production scenarios.

---

# Lab Objectives

By completing this lab, you will learn how to:

- Force Pods onto specific nodes using **nodeName** (bypassing the scheduler).
- Schedule Pods dynamically using **nodeSelector** with labels.
- Apply **Taints** to restrict scheduling and use **Tolerations** to allow exceptions.
- Implement **Node Affinity** with both required and preferred rules.
- Understand Kubernetes **QoS Classes** and how they affect Pod eviction priority.
- Deploy cluster-wide services using **DaemonSets**.
- Create and manage **Static Pods** directly via the Kubelet.

---

# Lab Environment

- **Cluster**: Minikube (Multi-node setup)
- **Primary Worker Node**: minikube-m02
- **Tools Used**: kubectl, SSH, YAML manifests

---

# Project Structure

```bash
K8s_Advanced_Scheduling_Lab
│
├── README.md
├── task1-nodeName.yaml
├── task2-nodeSelector.yaml
├── task3-taints-tolerations.yaml
├── task4-node-affinity.yaml
├── task5-qos-demo.yaml
├── task6-daemonset.yaml
└── task7-static-pod.yaml
```

Note: Most tasks were executed using imperative commands and YAML manifests to simulate real-world troubleshooting and deployment scenarios.

---

# 🚀 Lab Execution & Commands

## 1. Targeted Scheduling (nodeName & nodeSelector)

In this section, we bypassed the scheduler using hard-coded node names and used labels for dynamic selection.

### Task 1: Using nodeName (The Hard-Pin)

```bash
# Get the exact node name
kubectl get nodes

# Apply Pod with spec.nodeName: minikube-m02
# This bypasses the scheduler entirely
kubectl apply -f task1-nodeName.yaml

# Verification
kubectl get pod static-pinned -o wide
```

### Task 2: Using nodeSelector (Label-based)

```bash
# Add a custom label to the node
kubectl label node minikube-m02 env=lab

# Run a Pod that requires this label
kubectl apply -f task2-nodeSelector.yaml

# Test Failure: Remove the label and observe 'Pending' status
kubectl label node minikube-m02 env-
kubectl get pod selector-demo -o wide
```

---

## 2. Taints and Tolerations

We implemented the **"Lock and Key"** mechanism to repel unauthorized Pods from specific nodes.

```bash
# Apply a 'NoSchedule' Taint to the node
kubectl taint nodes minikube-m02 test=true:NoSchedule

# Attempt to run a normal Pod (will stay Pending)
kubectl run no-tol --image=nginx

# Apply a 'NoExecute' Taint to evict existing Pods
kubectl taint nodes minikube-m02 evict=now:NoExecute

# Verification of eviction
kubectl get pods -w

# Cleanup: Remove taints to allow future scheduling
kubectl taint nodes minikube-m02 test=true:NoSchedule-
kubectl taint nodes minikube-m02 evict=now:NoExecute-
```

---

## 3. Node Affinity (Hard vs. Soft Rules)

Node Affinity provides more expressive rules than nodeSelector using logical operators.

```bash
# Required (Hard Rule): Pod won't start without the label
kubectl label node minikube-m02 disktype=ssd
kubectl apply -f task4-node-affinity.yaml

# Preferred (Soft Rule): Pod starts even if label is missing
# (Modify YAML to use preferredDuringSchedulingIgnoredDuringExecution)
kubectl apply -f task4-node-affinity.yaml

# Verify affinity configuration
kubectl describe pod affinity-demo | grep -A15 Affinity
```

---

## 4. Resource Management & QoS Classes

Kubernetes assigns a QoS class based on resource requests and limits.

```bash
# Apply Pods for each class
kubectl apply -f task5-qos-demo.yaml

# Check QoS class for all Pods
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.qosClass}{"\n"}{end}'
```

---

## 5. DaemonSets (Cluster-wide Services)

Ensures a Pod runs on every node.

```bash
# Deploy the DaemonSet
kubectl apply -f task6-daemonset.yaml

# Confirm one Pod per node
kubectl get ds node-monitor
kubectl get pods -o wide | grep node-monitor

# View system DaemonSets
kubectl get ds -n kube-system
```

---

## 6. Static Pods (Node-level Management)

Static Pods are managed directly by the Kubelet.

```bash
# SSH into the worker node
minikube ssh -p minikube -n minikube-m02

# Create manifest directory
sudo mkdir -p /etc/kubernetes/manifests

# Create Static Pod manifest
sudo tee /etc/kubernetes/manifests/static-nginx.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: static-nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
EOF

# Exit node
exit

# Verify Pod
kubectl get pods -o wide | grep static-nginx
```

---

## 📊 Lab Evidence Tracking

All outputs were consolidated into a final proof file.

```bash
# Generate final report
cat k8s_proof.txt
```


# Key Learnings

## Targeted Scheduling
Using `nodeName` completely bypasses the scheduler, while `nodeSelector` allows dynamic placement based on labels. Misconfigured labels can leave Pods stuck in a Pending state.

## Taints & Tolerations
Think of it as a **lock-and-key mechanism**. Nodes repel unwanted Pods unless they explicitly tolerate the taint.

## Node Affinity
Provides more expressive and flexible scheduling than nodeSelector:
- **Required** = Hard rule (must match)
- **Preferred** = Soft rule (best effort)

## Resource Management (QoS)
Kubernetes assigns Pods into QoS classes:
- **Guaranteed** → Highest priority, least likely to be evicted
- **Burstable** → Medium priority
- **BestEffort** → First to be evicted under pressure

## DaemonSets
Used for infrastructure-level workloads (e.g., monitoring, logging agents). Ensures one Pod runs on every node automatically.

## Static Pods
Managed directly by the Kubelet from the node filesystem. Independent from the API Server, making them critical for control plane components.

---

# Final Result

This lab successfully demonstrated how to control and optimize Pod placement and behavior inside a Kubernetes cluster.

We moved from default scheduling to a fully controlled environment where:

- Pods can be forced or guided to specific nodes.
- Nodes can reject or accept workloads selectively.
- Scheduling logic can be fine-tuned using affinity rules.
- Resource usage directly impacts Pod survival (QoS).
- System services run reliably across all nodes (DaemonSets).
- Critical components can run independently of the control plane (Static Pods).

Mastering scheduling is essential for building resilient and production-ready Kubernetes systems.

