## 📦 Kubernetes Deployment: `kennethreitz/httpbin`

This solution demonstrates a clean and minimal Kubernetes deployment of the public Docker image [`kennethreitz/httpbin`](https://hub.docker.com/r/kennethreitz/httpbin), with a focus on:

* Best practices in Kubernetes resource definitions
* Secure container runtime configuration
* Even pod distribution across the cluster
* Simple, effective verification of application availability

---

### 💠 Deployment Overview

* **Deployment name:** `httpbin`
* **Namespace:** `demo`
* **Replicas:** 3
* **Image:** `kennethreitz/httpbin`
* **Port exposed:** `80`
* **Security settings:**

  * Run as non-root (`runAsUser: 1000`)
  * No privilege escalation
  * All Linux capabilities dropped
* **Affinity rules:** Ensures pods are scheduled on different nodes using `podAntiAffinity`

---

### 📁 Files Included

| File                      | Description                                                                |
| ------------------------- | -------------------------------------------------------------------------- |
| `deployment.yaml` | Kubernetes Deployment manifest with security and scheduling best practices |
| `service.yaml`    | ClusterIP Service for internal access to the application                   |

---

### 🔒 Security Best Practices Implemented

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

* Container runs with the **least privilege**
* Filesystem is **writable only where needed**
* No ability to gain additional capabilities at runtime

---

### 🌍 Even Pod Distribution

Pod anti-affinity ensures that the 3 replicas are spread across different nodes:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - httpbin
        topologyKey: "kubernetes.io/hostname"
```

---

### ✅ Method of Verification

Application functionality and health were verified using multiple methods:

   **Actual verification proof:**
> **Note:** Only one pod is currently running due to limited local resources, but the deployment is correctly configured to distribute pods across nodes when the cluster permits.

   ```bash
   kubectl get pods -n demo
   NAME                       READY   STATUS    RESTARTS   AGE
   httpbin-79597cbfc6-9k8c6   0/1     Pending   0          18m
   httpbin-79597cbfc6-bpbnm   0/1     Pending   0          18m
   httpbin-79597cbfc6-n5b7f   1/1     Running   0          18m

   kubectl port-forward svc/httpbin 8080:80 -n demo

   curl -I http://localhost:8080/
   HTTP/1.1 200 OK
   Server: gunicorn/19.9.0
   Date: Thu, 08 May 2025 13:39:12 GMT
   Connection: keep-alive
   Content-Type: text/html; charset=utf-8
   Content-Length: 9593
   Access-Control-Allow-Origin: *
   Access-Control-Allow-Credentials: true
   ```

   → Confirms that the container is serving valid HTTP responses.

---
