## 🧱 Issue 2: Pod stuck in *ImagePullBackOff* state due to invalid or private image name

---

### 🧩 Environment / Context
- **Cluster type:** kubeadm (self-managed)
- **Nodes:** 1 control-plane + 2 worker nodes (AWS EC2)
- **Kubernetes version:** v1.31.13
- **Container runtime:** containerd
- **Namespace:** prod

---

### ⚠️ Problem Description
After applying the manifest file:

```bash
kubectl apply -f pod4.yaml
```

Output:
```
pod/nginxpod4 created
```

Then checking the pod status:
```bash
kubectl get po -n prod
```

Output:
```
NAME        READY   STATUS             RESTARTS       AGE
nginxpod    1/1     Running            0              71m
nginxpod2   1/1     Running            0              60m
nginxpod3   1/1     Running            1 (6m3s ago)   18m
nginxpod4   0/1     ImagePullBackOff   0              24s
```

---

### 📄 YAML Manifest (`pod4.yaml`)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginxpod4
  namespace: prod
  labels:
    env: prod
spec:
  containers:
    - name: nginx
      image: mongospring
      ports:
      - containerPort: 80
```

---

### 🔍 Investigation Steps
Described the pod to get detailed events:
```bash
kubectl describe po nginxpod4 -n prod
```
Found:
```
Warning  Failed     Failed to pull image "mongospring": failed to resolve reference "docker.io/library/mongospring:latest": pull access denied, repository does not exist or may require authorization
```

---

### 🧠 Root Cause
The image name **`mongospring`** does not exist in Docker Hub’s public registry.  
Kubernetes attempted to pull the image from `docker.io/library/mongospring:latest`, but since the image was **invalid or private**, the node couldn’t access it.

---

### 🛠️ Resolution Steps

#### ✅ Option 1: Use a valid public image
Replace the image name in `pod4.yaml`:
```yaml
image: nginx:latest
```
Then reapply:
```bash
kubectl apply -f pod4.yaml
```

#### 🔐 Option 2: If using a private image
1. Create a Docker registry secret:
   ```bash
   kubectl create secret docker-registry regcred      --docker-server=<registry-url>      --docker-username=<username>      --docker-password=<password>      --docker-email=<email>
   ```
2. Reference it in your pod YAML:
   ```yaml
   imagePullSecrets:
     - name: regcred
   ```
3. Reapply the pod file.

---

### ✅ Key Learnings
- `ImagePullBackOff` means Kubernetes **failed to pull the image** from the registry.  
- Always verify the image name and path before deployment.  
- For private images, use **imagePullSecrets** for authentication.  
- Use `kubectl describe pod` to find the exact cause of pull errors.

---

### 💡 Prevention / Automation
- Run `docker pull <image-name>` locally to verify image existence.  
- Maintain a private registry secret for authenticated image pulls.  
- Use CI/CD validation to check image accessibility before deploy.  

---

### 🗣️ Interview Summary
> “While practicing Kubernetes, I faced a pod stuck in *ImagePullBackOff* state.
> After describing the pod, I found the image name was incorrect (`mongospring`).
> Kubernetes couldn’t find it in Docker Hub and failed to pull it.
> I fixed the issue by using a valid image and learned how Kubernetes handles image pulling and registry authentication.”

---
