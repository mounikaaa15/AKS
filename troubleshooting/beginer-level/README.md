Here are Kubernetes interview questions from beginner to advanced level covering:

Pods
ReplicaSets
Deployments
Services
ConfigMaps
Secrets
Namespaces

I have arranged them in the order interviewers usually ask: basic → intermediate → production-level scenarios.

Beginner Level Kubernetes Interview Questions
1. What is Kubernetes?

Answer:

Kubernetes is an open-source container orchestration platform used to automate:

Container deployment
Scaling
Networking
Self-healing
Rolling updates
Resource management

It manages containers across a cluster of worker nodes.

Pods Interview Questions
2. What is a Pod in Kubernetes?

Answer:

A Pod is the smallest deployable unit in Kubernetes.

A Pod represents one or more containers that:

Share the same network namespace
Share storage volumes
Have the same lifecycle

Example:

Pod
 |
 |-- Application Container
 |
 |-- Sidecar Container
3. Why does Kubernetes use Pods instead of directly running containers?

Answer:

Because Kubernetes needs an abstraction layer.

Pods provide:

Container grouping
Shared networking
Shared storage
Lifecycle management
4. Can a Pod contain multiple containers?

Answer:

Yes.

Example:

Payment Pod

Container 1:
Application

Container 2:
Logging Agent

Container 3:
Monitoring Agent

Containers inside a pod:

Run together
Start together
Stop together
5. How do you create a Pod?

Example:

apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
  - name: nginx
    image: nginx

Command:

kubectl apply -f pod.yaml
6. How do you check Pod status?
kubectl get pods

Detailed:

kubectl describe pod <pod-name>
7. Explain different Pod phases.

Answer:

Phase	Meaning
Pending	Pod scheduled but container not started
Running	Containers running
Succeeded	Completed successfully
Failed	Container failed
Unknown	Node communication issue
ReplicaSet Interview Questions
8. What is ReplicaSet?

Answer:

ReplicaSet ensures that a specified number of identical Pods are always running.

Example:

replicas: 3

Means:

ReplicaSet

 |
 |-- Pod 1
 |
 |-- Pod 2
 |
 |-- Pod 3
9. Why do we need ReplicaSets?

ReplicaSets provide:

High availability
Pod replacement
Self-healing

Example:

If one pod crashes:

Before:

3 Pods

After failure:

2 Pods

ReplicaSet creates:

New Pod
10. Difference between Pod and ReplicaSet?
Pod	ReplicaSet
Runs containers	Manages Pods
Single instance	Multiple replicas
No self healing	Self healing
11. Can we update application versions using ReplicaSet?

No.

ReplicaSets do not support:

Rolling updates
Rollbacks

For that we use Deployment.

Deployment Interview Questions
12. What is Deployment?

Answer:

Deployment manages ReplicaSets and provides:

Rolling updates
Rollbacks
Version control
Scaling

Architecture:

Deployment

    |
    |
 ReplicaSet

    |
    |
  Pods
13. Deployment vs ReplicaSet?
Deployment	ReplicaSet
Manages ReplicaSets	Manages Pods
Supports rollback	No rollback
Supports updates	No update strategy
Production recommended	Internal component
14. Explain Rolling Update strategy.

Example:

Current:

Version 1

Pod Pod Pod

Update:

Version 2

New Pod
Old Pod
Old Pod

Gradually:

New Pod
New Pod
New Pod

No downtime.

15. How do you update an image in Deployment?

Command:

kubectl set image deployment/app \
app=nginx:1.28
16. How do you rollback a Deployment?
kubectl rollout undo deployment/app

Check history:

kubectl rollout history deployment/app
17. What happens when a Pod inside Deployment crashes?

Flow:

Pod crashes

     |

ReplicaSet detects

     |

Creates new Pod

     |

Traffic restored
Service Interview Questions
18. What is Kubernetes Service?

Answer:

Service provides stable networking for Pods.

Pods are temporary.

Their IP changes.

Service provides:

Stable IP
DNS name
Load balancing
19. Why do we need Services?

Example:

Without Service:

User

 |
Pod IP

Problem:

Pod restarts.

IP changes.

With Service:

User

 |

Service IP

 |

Pods
20. Types of Kubernetes Services?
ClusterIP

Default.

Internal communication.

Example:

Backend ---> Database
NodePort

Exposes application externally.

Example:

User

 |
Node IP:30080

 |
Service

 |
Pod
LoadBalancer

Cloud provider load balancer.

Example:

AWS:

ELB

 |
Service

 |
Pods
21. How does Service find Pods?

Using labels.

Example:

Service:

selector:
 app: payment

Pod:

labels:
 app: payment

Matching labels connect them.

ConfigMap Interview Questions
22. What is ConfigMap?

ConfigMap stores non-sensitive configuration.

Examples:

Database hostname

Application mode

Log level

Environment variables
23. Why not store configuration inside container images?

Because:

Same image can run everywhere
Environment-specific config changes
Easier deployment

Example:

Image

+

ConfigMap

=

Application
24. How can applications consume ConfigMaps?
Environment variables

Example:

envFrom:
- configMapRef:
    name: app-config
Mounted files

Example:

/etc/config/application.properties
Secrets Interview Questions
25. What is Kubernetes Secret?

Secret stores sensitive information:

Examples:

Passwords
API keys
Certificates
26. Difference between ConfigMap and Secret?
ConfigMap	Secret
Non-sensitive data	Sensitive data
Plain text	Base64 encoded
App settings	Passwords
27. Are Kubernetes Secrets encrypted?

Default:

No.

They are only base64 encoded.

Production:

Enable:

Encryption at rest
External secret managers

Examples:

Cloud secret services
Vault solutions
28. How do Pods consume Secrets?

Environment variable:

envFrom:
- secretRef:
    name: database-secret

or volume:

/etc/secrets/password
Namespace Interview Questions
29. What is Namespace?

Namespace provides logical isolation inside a Kubernetes cluster.

Example:

Cluster

 |
 |-- Development Namespace
 |
 |-- Testing Namespace
 |
 |-- Production Namespace
30. Why use Namespaces?

Benefits:

Resource isolation
Access control
Separate teams
Environment separation
31. Namespace vs Cluster?
Namespace	Cluster
Logical separation	Physical Kubernetes environment
Inside cluster	Collection of nodes
Intermediate Production-Level Questions
32. A Pod is running but application is not accessible. How do you troubleshoot?

Answer:

Check:

1. Pod status
kubectl get pods
2. Logs
kubectl logs pod-name
3. Service
kubectl get svc
4. Endpoints
kubectl get endpoints
5. Labels
kubectl describe svc service-name
33. Pod is stuck in Pending state. Reasons?

Possible causes:

No available nodes
Insufficient CPU/memory
Wrong node selector
PVC unavailable
Scheduling constraints

Debug:

kubectl describe pod pod-name
34. Pod is CrashLoopBackOff. What does it mean?

Container starts:

Start

 |

Crash

 |

Restart

 |

Crash again

Common reasons:

Application error
Wrong environment variables
Missing secrets
Wrong command

Debug:

kubectl logs pod-name
35. How do you perform zero downtime deployment?

Use:

Deployment
RollingUpdate strategy
Readiness probes

Example:

strategy:
 type: RollingUpdate
Advanced Production Questions
36. Explain Kubernetes Deployment lifecycle.
Deployment created

        |

ReplicaSet created

        |

Pods created

        |

Service exposes Pods

        |

Rolling updates create new ReplicaSet

        |

Old ReplicaSet scaled down
37. What happens when you change a Deployment image?

Example:

Before:

Deployment

ReplicaSet v1

Pods v1

After:

Deployment

ReplicaSet v2

Pods v2

Old ReplicaSet remains for rollback.

38. How do you limit resources for Pods?

Using:

resources:

 requests:
   cpu: 250m
   memory: 256Mi

 limits:
   cpu: 500m
   memory: 512Mi
39. Difference between requests and limits?
Requests	Limits
Minimum guaranteed	Maximum allowed
Scheduler uses it	Runtime enforcement
40. How do you secure Kubernetes workloads?

Production practices:

Run containers as non-root
Use RBAC
Use NetworkPolicies
Encrypt Secrets
Scan images
Restrict privileges
Use namespaces
Scenario-Based Senior Interview Questions
41. Your production Deployment has 10 replicas. One node goes down. What happens?

Answer:

Node becomes unavailable
Kubernetes detects missing Pods
Scheduler creates replacement Pods
Service routes traffic to healthy Pods
42. How do you deploy a critical application with zero downtime?

Answer:

Use:

Deployment
Multiple replicas
Readiness probes
Rolling updates
PodDisruptionBudget
Horizontal scaling
43. How would you debug Service not routing traffic?

Checklist:

Service exists?
        |
Selector correct?
        |
Endpoints created?
        |
Pods healthy?
        |
Container port correct?
        |
Network policy blocking?

Commands:

kubectl get svc

kubectl get endpoints

kubectl describe svc app-service
44. Why should Pods not be created directly in production?

Because Pods:

Cannot self-heal
Cannot scale
Cannot rollback

Production uses:

Deployment
     |
ReplicaSet
     |
Pods
45. Explain the complete production request flow.

Example:

User

 |

LoadBalancer

 |

Ingress

 |

Service

 |

Deployment

 |

ReplicaSet

 |

Pod

 |

Container

 |

Application
