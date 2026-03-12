# ☸️ Lab 1: My First Pod and Service

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)

---

## 📖 Scenario
You have just joined the platform team. Your first task is to manually deploy a simple **"hello world"** web application to a Kubernetes cluster to verify that your access and the cluster itself are working correctly. You need to deploy it and expose it internally to another team member for testing.

---

## 📝 Lab Description
The student will create and manage a single **Nginx pod**. They will use both **imperative** and **declarative** approaches to understand the difference. They will then expose this pod using a **Service** to make it accessible within the cluster.

---

## 🚀 Tasks to Implement

- [ ] **Task 1:** Using imperative commands, run an Nginx pod.
- [ ] **Task 2:** Using a declarative YAML file, create an Nginx pod with a custom `index.html` file that says *"Hello from My First Lab"*.
- [ ] **Task 3:** Use `kubectl` commands to list the pod, view its logs, and describe its status. Check the Pod Lifecycle phases as the pod starts.
- [ ] **Task 4:** Expose the declarative pod internally using a **ClusterIP service**.
- [ ] **Task 5:** Execute an `exec` command into the pod to verify the custom `index.html` content and use `curl` to test the service endpoint from within the cluster.

---
## 📂 Project Structure

```text
.
├── index.html          
├── pod.yaml            
├── service.yaml        
└── README.md         