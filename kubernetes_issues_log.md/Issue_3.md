# 🧾 Kubernetes Issues Log (Interview & Project Notes)

> A detailed record of all real issues I faced while practicing Kubernetes — including causes, fixes, and learnings.  
> This document is meant for **DevOps interviews** and **project troubleshooting references**.  
> Every entry follows a clear, story-based format for easy recall during discussions.

---

## 📘 Table of Contents
| No | Issue Title | Description |
|----|--------------|--------------|
| 1️⃣ | [Service not connecting to pods due to different namespace](#1️⃣-issue-service-not-connecting-to-pods-due-to-different-namespace) | Service and pods were deployed in different namespaces, leading to no endpoints. |

---

## 🧱 1️⃣ Issue: Service not connecting to pods due to different namespace

---

### 🧩 2️⃣ Environment / Context  
- **Cluster type:** Kubeadm cluster on AWS EC2  
- **Nodes:** 1 control-plane + 2 worker nodes  
- **Kubernetes version:** v1.29  
- **Container runtime:** containerd  
- **Cloud/OS:** AWS EC2 / Ubuntu 22.04  

---

### ⚠️ 3️⃣ Problem Description  
While deploying a Spring Boot application using a ReplicaSet and Service, the Service failed to route traffic to the pods.

**Commands executed:**
```bash
kubectl get svc
```
**Output:**
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        5d8h
springsvc    NodePort    10.104.67.211   <none>        80:32425/TCP   5m17s
```

Checked the pods:
```bash
kubectl get all -n prod
```
**Output:**
```
pod/replica1-5hrm5   1/1   Running   0   7m51s
pod/replica1-k6h25   1/1   Running   0   7m51s
pod/replica1-pcwcw   1/1   Running   0   7m51s
```

When describing the Service:
```bash
kubectl describe svc springsvc
```
**Output:**
```
Endpoints:
```
(no endpoints listed)

As a result, accessing the app at  
`http://<EC2-Public-IP>:32425` returned no response.

---

### 🔍 4️⃣ Investigation Steps  
1. Verified pod and service selectors:
   ```bash
   kubectl describe svc springsvc
   ```
   Found that the **selector** matched (`app=springapp`).

2. Checked the namespace of the pods:
   ```bash
   kubectl get pods -n prod
   ```
   Pods existed in the **`prod` namespace**.

3. Checked the namespace of the service:
   ```bash
   kubectl get svc
   ```
   Service was in the **`default` namespace**.

4. Realized that **Services are namespace-scoped** —  
   a service in one namespace cannot reach pods in another namespace.

---

### 🧠 5️⃣ Root Cause  
The **Service was created in the default namespace**, while the ReplicaSet and pods were created in the **`prod` namespace**.  
Because Services cannot cross namespace boundaries, it found **no matching pods**, leaving **no endpoints** attached.

---

### 🛠️ 6️⃣ Resolution Steps  
1. Deleted the old Service:
   ```bash
   kubectl delete svc springsvc
   ```

2. Updated the Service YAML to include the correct namespace:
   ```yaml
   metadata:
     name: springsvc
     namespace: prod
   ```

3. Reapplied the YAML:
   ```bash
   kubectl apply -f rs1.yaml
   ```

4. Verified again:
   ```bash
   kubectl describe svc springsvc -n prod
   ```
   **Output:**
   ```
   Endpoints: 10.36.0.2:8080, 10.36.0.1:8080, 10.44.0.1:8080
   ```

✅ Service now successfully routed to all 3 pods.

---

### ✅ 7️⃣ Key Learnings
- Services and pods **must be in the same namespace** for endpoint discovery.  
- `kubectl describe svc` helps verify if endpoints are correctly linked.  
- Namespaces act as **logical isolation boundaries** — resources in different namespaces do not interact automatically.  
- Always specify the namespace explicitly in YAML manifests.

---

### 💡 8️⃣ Prevention / Automation
- Always specify `namespace:` in all Kubernetes manifests.  
- Use `kubectl get all --all-namespaces` to verify resource placement.  
- Maintain consistent naming patterns (e.g., `springapp-prod`).  
- Integrate a CI/CD check to validate namespace consistency before deployment.

---

### 🗣️ 9️⃣ Interview Summary (How to Explain)
> “While deploying a Spring Boot app, my Service couldn’t reach the pods even though labels matched.  
> I found that my ReplicaSet and pods were in the `prod` namespace, but my Service was accidentally created in the `default` namespace.  
> Since Services are namespace-scoped, it couldn’t discover the pods.  
> I fixed it by reapplying the Service in the same namespace, after which the endpoints appeared and traffic flowed correctly.  
> This taught me the importance of namespace alignment and scoping in Kubernetes.”

---

### 🧩 Notes for Future Logs
When logging your next issue, copy this structure and replace details as needed:  
1. Clear issue title  
2. Environment context  
3. Actual commands + outputs  
4. Investigation + reasoning  
5. Fix + learnings  
6. Interview summary  

---

**Author:** Balaji Besta  
**Purpose:** Real-world troubleshooting log for Kubernetes practice and DevOps interview preparation.  
**Last Updated:** November 9, 2025


---

## 🧾 Practical Verification

Below is a real execution screenshot captured during the troubleshooting session on my Kubernetes cluster.

![Service Namespace Practice Proof](Command-Outputs/Issue_3.png)

**Description:**
- The screenshot shows successful creation of the Service in the `prod` namespace.
- Verified endpoints (`10.36.0.2`, `10.36.0.1`, `10.44.0.1`) confirm that the service correctly linked to 3 running pods.
- Confirms real, hands-on execution — not a copy-paste exercise.

---

**Author:** Balaji Besta  
**Last Updated:** November 9, 2025
