# 🛠️ Lab 2: Pod Troubleshooting and Static Pods
---

## 📖 Scenario
The cluster's control plane is managed by another team. You are responsible for workloads on the worker nodes. A node has been rebooted, and you need to ensure that a critical logging agent is running on it regardless of the cluster's state. You also need to practice basic troubleshooting on a misconfigured pod.

---

## 📝 Lab Description
The student will learn about **static pods** by placing a manifest on a control-plane node. They will then be given a broken pod manifest to debug, focusing on **pod lifecycle events** and **storage issues** without using PVCs.

---
# Tasks to Implement

- [ ] **Task 1:** SSH into the control-plane node (or use `minikube ssh`). Identify the directory for static manifests (e.g., `/etc/kubernetes/manifests`).

- [ ] **Task 2:** Create a static pod manifest for a "logger" container that runs `sleep 3600`. Verify that the pod runs on the control-plane node itself.

- [ ] **Task 3:** Troubleshoot a broken pod: You are given the following YAML (`fail-pod.yaml` in this repo). It should run but fails. Identify why and fix it.

- [ ] **Task 4:** Use `kubectl describe`, `kubectl logs`, and check the Pod Lifecycle to diagnose the "CrashLoopBackOff" or error state. Fix the issue.

- [ ] **Task 5:** Modify the fixed pod to include ephemeral storage by writing a file to `/data` inside the container and verifying it exists.

---

## 📂 Project Structure
```text
.
├── fail-pod.yaml          # The broken pod manifest to be fixed
├── logger-static.yaml     # Manifest for the static pod task
└── README.md              # Lab documentation and instructions