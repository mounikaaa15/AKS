Below is a more detailed explanation of how to troubleshoot when **users cannot access a Kubernetes application**, including what each command checks, why it is used, expected output, unexpected output, and the next troubleshooting action.

# 31. Users cannot access the application. How would you troubleshoot?

A structured troubleshooting approach is required because user access issues can occur at multiple layers:

**User → DNS → Azure Load Balancer → Ingress Controller → Kubernetes Service → Endpoints → Pods → Application**

The goal is to identify the layer where communication is failing.

---

## 1. Check Application Pods

### Command

```bash
kubectl get pods -n <namespace>
```

### Purpose

This verifies whether the application containers are running correctly. If pods are not healthy, users will not be able to access the application even if networking is configured correctly.

### Expected Output

Example:

```
NAME                         READY   STATUS    RESTARTS   AGE
app-deployment-7d8f9c-x2abc  1/1     Running   0          5h
app-deployment-7d8f9c-y3def  1/1     Running   0          5h
```

Meaning:

* `READY 1/1` → Container is running and ready to receive traffic.
* `STATUS Running` → Pod is active.
* `RESTARTS 0` → Application has not crashed.

### Unexpected Outputs

#### Case 1: CrashLoopBackOff

Example:

```
NAME                         READY   STATUS             RESTARTS
app-deployment-x123          0/1     CrashLoopBackOff   10
```

Possible causes:

* Application startup failure
* Incorrect configuration
* Missing environment variables
* Database connection failure
* Invalid application code

Next step:

```bash
kubectl logs <pod-name>
```

---

#### Case 2: ImagePullBackOff

Example:

```
app-deployment-x123   0/1   ImagePullBackOff
```

Possible causes:

* Container image does not exist
* Wrong image tag
* Authentication failure with container registry

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Failed to pull image
```

---

#### Case 3: Pending

Example:

```
app-deployment-x123   0/1   Pending
```

Possible causes:

* No available worker node resources
* CPU/memory shortage
* Node selector mismatch
* Taints/tolerations issue

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```
FailedScheduling
```

---

# 2. Inspect Pod Details

### Command

```bash
kubectl describe pod <pod-name>
```

### Purpose

Provides detailed information about:

* Container status
* Events
* Volume mounts
* Environment variables
* Health probes
* Scheduling issues

---

### Expected Output

Example:

```
State: Running

Ready: True

Conditions:
  Type              Status
  Ready             True
```

Events:

```
Normal Started container app
Normal Pulled image successfully
```

---

### Unexpected Output Examples

## Application failing readiness probe

Example:

```
Readiness probe failed:
HTTP probe failed with statuscode: 503
```

Impact:

* Pod may be running
* But Kubernetes will not send user traffic to it

Solution:

* Check application health endpoint
* Verify application dependencies

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
```

---

## Container killed due to memory issue

Example:

```
Last State:
Reason: OOMKilled
```

Meaning:

Application exceeded its memory limit.

Solution:

Increase resources:

```yaml
resources:
 limits:
   memory: 2Gi
```

---

# 3. Review Application Logs

### Command

```bash
kubectl logs <pod-name>
```

For multiple containers:

```bash
kubectl logs <pod-name> -c <container-name>
```

### Purpose

Checks whether the application itself is functioning.

---

### Expected Output

Example:

```
INFO Application started successfully
INFO Listening on port 8080
INFO Connected to database
```

Application is healthy.

---

### Unexpected Output Examples

## Database connection failure

```
Connection refused to database server
```

Possible causes:

* Database unavailable
* Incorrect credentials
* Network issue

---

## Application startup failure

```
Exception: Missing configuration property DATABASE_URL
```

Possible causes:

* Missing ConfigMap
* Missing Secret
* Incorrect environment variables

---

## Port binding issue

```
Server failed to bind port 8080
```

Possible causes:

* Application configured for another port
* Service pointing to wrong container port

---

# 4. Verify Kubernetes Service

### Command

```bash
kubectl get svc -n <namespace>
```

### Purpose

A Service provides a stable network endpoint for pods.

It verifies:

* Service exists
* Service type
* Service port
* Cluster IP

---

### Expected Output

Example:

```
NAME            TYPE        CLUSTER-IP      PORT(S)
app-service     ClusterIP   10.96.10.25     80/TCP
```

Meaning:

* Service exists
* It exposes port 80
* It forwards traffic to application pods

---

### Unexpected Output

## Service missing

```
No resources found
```

Cause:

* Deployment exists but Service was not created

---

## Wrong port

Example:

Service:

```
PORT 80
TARGET PORT 9090
```

Pod:

```
Container listens on 8080
```

Result:

Traffic fails because Service forwards to the wrong port.

---

# 5. Check Service Configuration

### Command

```bash
kubectl describe svc <service-name>
```

### Purpose

Checks:

* Selector
* Ports
* Endpoints

---

### Expected Output

Example:

```
Selector:
app=myapplication

Port:
80/TCP

TargetPort:
8080

Endpoints:
10.244.1.5:8080
10.244.2.6:8080
```

---

### Unexpected Output

Example:

```
Endpoints: <none>
```

Meaning:

Service cannot find any pods.

Possible causes:

## Label mismatch

Service:

```yaml
selector:
 app: frontend
```

Pod:

```yaml
labels:
 app: backend
```

The service will not route traffic.

Fix:

Make labels and selectors match.

---

# 6. Verify Endpoints

### Command

```bash
kubectl get endpoints
```

### Purpose

Checks whether the Service has backend pods available.

---

### Expected Output

Example:

```
NAME            ENDPOINTS
app-service     10.244.1.10:8080,10.244.2.10:8080
```

Traffic path:

```
Service
   |
   |
Pod IP:8080
```

---

### Unexpected Output

Example:

```
app-service     <none>
```

Possible causes:

* Pods are not ready
* Readiness probe failing
* Service selector mismatch

---

# 7. Check Ingress Configuration

### Command

```bash
kubectl get ingress
```

### Purpose

Ingress exposes applications outside the Kubernetes cluster.

It checks:

* Hostname
* Backend service
* External routing

---

### Expected Output

Example:

```
NAME        HOSTS              ADDRESS
app-ingress app.company.com    20.x.x.x
```

---

### Unexpected Output

## No ingress

```
No resources found
```

Possible cause:

* Ingress not deployed

---

## Wrong backend service

Example:

Ingress:

```
Backend:
wrong-service:80
```

But actual service:

```
app-service
```

Result:

Users receive:

```
404 Not Found
```

---

# 8. Check Ingress Controller

Even if ingress exists, the controller may not be running.

Check:

```bash
kubectl get pods -n ingress-nginx
```

Expected:

```
ingress-controller   Running
```

Unexpected:

```
CrashLoopBackOff
```

Possible causes:

* Controller configuration issue
* Certificate problem
* Cloud load balancer issue

---

# 9. Check Network Policies

### Command

```bash
kubectl get networkpolicy
```

### Purpose

Network policies control traffic between pods.

---

### Expected

No policy blocking application traffic.

Example:

```
No resources found
```

or correctly configured policies.

---

### Unexpected

Example:

```
deny-all-policy
```

Result:

* Pods are running
* Service exists
* But traffic is blocked

Solution:

Allow required communication:

```
Ingress Controller
       |
       |
 Application Pods
```

---

# 10. Check Azure Load Balancer Configuration

For AKS environments:

Traffic path:

```
User
 |
DNS
 |
Azure Load Balancer
 |
Ingress Controller
 |
Service
 |
Pod
```

Check:

* Public IP exists
* Health probes are passing
* Backend pool is healthy
* NSG rules allow traffic

---

### Expected

Azure Load Balancer:

```
Backend health:
Healthy
```

---

### Unexpected

Example:

```
Backend health:
Unhealthy
```

Possible causes:

* Health probe port incorrect
* Ingress controller unavailable
* NSG blocking traffic

---

# 11. Validate DNS Resolution

### Command

```bash
nslookup app.example.com
```

Expected:

```
Name:
app.example.com

Address:
20.x.x.x
```

Unexpected:

```
NXDOMAIN
```

Meaning:

* DNS record missing
* Incorrect hostname

---

# 12. Test Application Internally

Run from inside cluster:

```bash
kubectl run test --image=curlimages/curl -it --rm -- sh
```

Then:

```bash
curl http://app-service:80
```

Expected:

```
HTTP/1.1 200 OK
```

Unexpected:

```
Connection refused
```

Meaning:

* Service issue
* Application not listening
* Network policy issue

---

# Complete Troubleshooting Decision Flow

```
User cannot access application
             |
             v
Check DNS
             |
             v
Check Azure Load Balancer
             |
             v
Check Ingress
             |
             v
Check Service
             |
             v
Check Endpoints
             |
             v
Check Pods
             |
             v
Check Application Logs
```

## Final Interview Answer Summary

"When users cannot access an application running on Kubernetes, I troubleshoot layer by layer. First, I verify pod health using `kubectl get pods` and inspect failures using `kubectl describe pod` and `kubectl logs`. Then I validate the Kubernetes Service configuration and ensure endpoints are correctly mapped to healthy pods. Next, I check Ingress rules and the ingress controller to confirm external routing. If the application is hosted on AKS, I verify Azure Load Balancer health probes, backend pools, NSG rules, and network policies. This helps identify whether the issue is caused by the application, Kubernetes networking, ingress, or cloud infrastructure."
