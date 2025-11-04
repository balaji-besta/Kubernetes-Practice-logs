

# 🧾 Kubernetes Issues Log (Interview & Project Notes)

Use this document to record all real issues you encounter while practicing Kubernetes. Each entry helps you recall what happened, how you fixed it, and what you learned — perfect for interviews or project discussions.

---

## 🧱 1️⃣ Issue Title
**Example:** Pod stuck in *Terminating* state due to *NotReady* node

---

## 🧩 2️⃣ Environment / Context
- **Cluster type:** (e.g., kubeadm, Minikube, EKS, GKE, etc.)
- **Nodes:** (e.g., 1 control-plane + 2 worker nodes)
- **Kubernetes version:**
- **Container runtime:** (e.g., containerd, Docker)
- **Cloud/OS:** (e.g., AWS EC2, Ubuntu 22.04)

---

## ⚠️ 3️⃣ Problem Description
Describe exactly what you observed.

**Example:**
```bash
kubectl get pods
````

**Output:**

```
nginx-demo   1/1   Terminating   1 (14h ago)   21h
```

The pod remained stuck in *Terminating* for a long time.

---

## 🔍 4️⃣ Investigation Steps

List the commands and checks you performed to debug the issue.

**Example:**

```bash
kubectl describe pod nginx-demo
```

**Found:**

```
Warning  NodeNotReady  node-controller  Node is not ready
```

Then checked:

```bash
kubectl get nodes
```

**Output:**

```
ip-172-31-2-114    Ready      control-plane
ip-172-31-40-68    NotReady   <none>
ip-172-31-44-133   NotReady   <none>
```

---

## 🧠 5️⃣ Root Cause

Explain what caused the issue.

**Example:**
Both worker nodes were stopped in AWS EC2, so the kubelet on those nodes wasn’t reporting to the control-plane. The pod’s node became unreachable, leaving the pod stuck in *Terminating*.

---

## 🛠️ 6️⃣ Resolution Steps

Show the exact commands and actions you took to fix the issue.

**Example:**

```bash
kubectl delete pod nginx-demo --grace-period=0 --force
kubectl delete node ip-172-31-40-68
kubectl delete node ip-172-31-44-133
```

Later rejoined worker nodes using:

```bash
kubeadm token create --print-join-command
```

---

## ✅ 7️⃣ Key Learnings

* Always check node status when pods are stuck.
* Understand how node unavailability affects pod lifecycle.
* `kubectl delete --grace-period=0 --force` can safely remove unreachable pods.
* Learned internal behavior of Kubernetes termination and finalizers.

---

## 💡 8️⃣ Prevention / Automation

* Create a script to auto-detect pods stuck in *Terminating* for >10m.
* Regularly monitor node health with `kubectl get nodes`.
* Use single-node cluster (remove taints) for cost-efficient labs.
* Add alerts for node heartbeat failures.

---

## 🗣️ 9️⃣ Interview Summary (How to explain)

> "While practicing Kubernetes, I faced an issue where a pod was stuck in *Terminating* state.
> After checking, I found both worker nodes were *NotReady* because I had stopped the EC2 instances.
> The control-plane couldn’t reach those nodes to finish the pod termination. I fixed it by force-deleting the pod and removing the unreachable nodes.
> This taught me how Kubernetes manages pod lifecycle and node communication."

---


---
