# Kubernetes Lab 4  
## emptyDir Behavior, Blue-Green Deployment, and Troubleshooting

This lab demonstrates how Kubernetes handles **ephemeral storage using `emptyDir` volumes**, how to perform a **Blue‑Green deployment**, and how to **troubleshoot configuration issues** in a deployment.

The application used in this lab is a simple **logger application** that writes timestamps continuously into a file inside a mounted volume.

---

# Lab Objectives

By completing this lab you will learn how to:

- Deploy an application using Kubernetes **Deployments**
- Use **emptyDir volumes** for temporary storage
- Understand the difference between:
  - Container restart
  - Pod recreation
- Implement **Blue‑Green Deployment**
- Troubleshoot deployment issues using:
  - `kubectl logs`
  - `kubectl exec`
- Switch application traffic using **Service selectors**

---

# Project Structure

```
Lab-4-Bonus
│
├── README.md
│
├── manifests
│   ├── v1
│   │   └── logger-deployment.yaml
│   │
│   ├── v2
│   │   └── logger-deployment.yaml
│   │
│   └── service
│       └── logger-service.yaml
│
└── screenshots
    ├── 01-v1-running.png
    ├── 02-emptydir-before-restart.png
    ├── 03-container-restart.png
    ├── 04-pod-recreation.png
    ├── 05-switch-service-to-v2.png
    ├── 06-v2-error.png
    ├── 07-v2-fixed.png
    └── 08-v2-output.png
```

---

# Step 1 — Create Namespace

```bash
kubectl create namespace staging
```

Verify:

```bash
kubectl get ns
```

---

# Step 2 — Deploy Logger v1

```
manifests/v1/logger-deployment.yaml
```

```bash
kubectl apply -f manifests/v1/logger-deployment.yaml
```

```bash
kubectl get pods -n staging
```

The application writes timestamps to:

```
/log/output.txt
```

---

# Step 3 — Verify Logging

```bash
kubectl exec -n staging deploy/logger -- cat /log/output.txt
```

Example output:

```
Fri Mar 13 14:57:16 UTC 2026
Fri Mar 13 14:57:26 UTC 2026
Fri Mar 13 14:57:36 UTC 2026
```

---

# Step 4 — Test Container Restart

```bash
kubectl exec -n staging deploy/logger -- kill 1
```

```bash
kubectl exec -n staging deploy/logger -- cat /log/output.txt
```

Result:

```
emptyDir survives container restart
```

---

# Step 5 — Test Pod Recreation

```bash
kubectl get pods -n staging
```

```bash
kubectl delete pod <pod-name> -n staging
```

```bash
kubectl exec -n staging deploy/logger -- cat /log/output.txt
```

Result:

```
emptyDir is recreated when a Pod is recreated
```

---

# Step 6 — Create Service

```
manifests/service/logger-service.yaml
```

```bash
kubectl apply -f manifests/service/logger-service.yaml
```

```bash
kubectl get svc -n staging
```

---

# Step 7 — Deploy Logger v2

```
manifests/v2/logger-deployment.yaml
```

```bash
kubectl apply -f manifests/v2/logger-deployment.yaml
```

```bash
kubectl get pods -n staging
```

Example output:

```
Fri Mar 13 14:57:56 UTC 2026 - Host: logger-v2-84868b8d84
```

---

# Step 8 — Switch Service to v2

```yaml
spec:
  selector:
    app: logger
    version: v2
```

```bash
kubectl apply -f manifests/service/logger-service.yaml
```

```bash
kubectl describe svc logger-service -n staging
```

---

# Step 9 — Troubleshooting

```bash
kubectl logs -n staging <v2-pod>
```

Error:

```
/bin/sh: can't create /wrong/log/output.txt: nonexistent directory
```

---

# Step 10 — Fix the Issue

```
mountPath: /log
/log/output.txt
```

```bash
kubectl apply -f manifests/v2/logger-deployment.yaml
```

---

# Step 11 — Verify Fix

```bash
kubectl get pods -n staging
```

```bash
kubectl exec -it -n staging <v2-pod> -- sh
```

```bash
cat /log/output.txt
```

Example:

```
Fri Mar 13 14:58:06 UTC 2026 - Host: logger-v2-84868b8d84
Fri Mar 13 14:58:16 UTC 2026 - Host: logger-v2-84868b8d84
```

---

# Key Learnings

## emptyDir Behavior

| Event | Result |
|------|-------|
| Container restart | Data remains |
| Pod recreation | Data deleted |

---

## Blue‑Green Deployment

Traffic switched from:

```
version=v1
```

to:

```
version=v2
```

using a **Service selector update**.

---

## Troubleshooting Tools

```
kubectl logs
kubectl exec
kubectl describe
kubectl get pods
```

---

# Final Result

- Demonstrated **emptyDir lifecycle behavior**
- Performed **Blue‑Green deployment**
- Identified and fixed a **logging path issue**
- Verified that `v2` writes both **timestamp and hostname**
