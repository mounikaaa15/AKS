
# OOMKilled Production Troubleshooting Guide (DevOps Perspective)

## What is OOMKilled?

OOM stands for **Out Of Memory**.

Every application (or container) is given a limited amount of RAM.

Example:

```text
Payment Service

Memory Limit = 2 GB
```

If the application uses:

```text
500 MB
800 MB
1.2 GB
1.8 GB
2.1 GB
```

Linux says:

> You crossed your memory limit.

It immediately kills the process.

Kubernetes sees that the process died and automatically starts a new pod.

---

# Important Mindset

When you see:

```text
OOMKilled
```

**Never think**

> Increase memory.

Instead ask:

> Why did this application need so much memory?

Finding the answer is the real job of a DevOps Engineer.

---

# Production Investigation Flow

```text
Alert Received
      │
      ▼
Confirm OOMKilled
      │
      ▼
Find which process used memory
      │
      ▼
Check logs
      │
      ▼
Check monitoring graphs
      │
      ▼
Check deployments
      │
      ▼
Check traffic
      │
      ▼
Identify Root Cause
      │
      ▼
Temporary Mitigation
      │
      ▼
Permanent Fix
```

---

# Step 1: Confirm it is OOMKilled

```bash
kubectl describe pod <pod-name>
```

Example

```text
Last State:
Reason: OOMKilled
Exit Code: 137
```

Now you know the pod died because of memory.

---

# Step 2: Which Process Used Memory?

Most containers run only one main application.

Example

```text
Container

PID 1

↓

Java Application
```

or

```text
Container

↓

Python
```

or

```text
Container

↓

NodeJS
```

Check running processes

```bash
kubectl exec -it <pod-name> -- ps aux
```

Example

```text
USER      PID   %MEM   COMMAND

root        1    82%   java
root       15     1%   sh
```

Now you know:

Java process is consuming memory.

---

# Step 3: Check Live Memory Usage

```bash
kubectl exec -it <pod-name> -- top
```

Example

```text
PID    MEMORY

1      1.8 GB

20     25 MB

35     18 MB
```

Now you know exactly which process is using RAM.

For Linux servers

```bash
top
```

or

```bash
htop
```

or

```bash
ps aux --sort=-%mem
```

---

# Step 4: What Was the Application Doing?

Logs tell the story.

Example

```bash
kubectl logs <pod-name> --previous
```

Suppose the last logs are

```text
Starting report generation...

Reading report.csv...

Loaded 5 million records...
```

Now you know memory increased while generating a report.

Another example

```text
POST /upload

Uploading movie.mp4

Size = 8 GB
```

Another example

```text
Refreshing Cache...
```

Logs explain what happened immediately before the crash.

---

# Step 5: Check Monitoring

Monitoring answers one question:

How did memory behave?

## Pattern 1

```text
500 MB

700 MB

900 MB

1200 MB

1500 MB

1800 MB

OOM
```

Memory continuously increases.

Possible cause:

* Memory leak

---

## Pattern 2

```text
400 MB

500 MB

600 MB

2 GB

OOM
```

Sudden spike.

Possible causes:

* Huge traffic
* Large file upload
* Report generation
* Batch processing

---

## Pattern 3

```text
500 MB

700 MB

500 MB

650 MB

550 MB
```

Healthy.

Memory goes up and comes back down.

Garbage collection is working.

---

# Step 6: Check Traffic

Example

```text
Users

100

↓

10000
```

Memory increased.

This is expected.

Possible solutions

* Scale replicas
* Increase memory (if justified)

Not a memory leak.

---

# Step 7: Check Deployments

Ask

> What changed?

Example

```text
10:00 AM

Deployment

↓

10:05 AM

OOMKilled
```

Possible conclusion

The new release introduced the problem.

Temporary solution

Rollback.

Developers investigate.

---

# Step 8: Check Scheduled Jobs

Example

```text
2:00 AM

Nightly Report
```

Memory becomes

```text
500 MB

↓

2.5 GB

↓

OOMKilled
```

Only happens at 2 AM.

Likely cause

Batch processing.

---

# Step 9: Check Node Memory

Sometimes the application is fine.

The node itself has no memory.

```bash
kubectl top nodes
```

Example

```text
Node Memory

95%
```

The pod may be killed because the entire node is under memory pressure.

---

# Common Causes of OOMKilled

---

## 1. Memory Leak

### Symptoms

Memory continuously increases.

```text
500

700

900

1200

1500

1800

OOM
```

### Example

```java
List<Customer> customers = new ArrayList<>();

while(true){
    customers.add(new Customer());
}
```

Memory is never released.

### DevOps Action

* Confirm memory trend
* Collect logs
* Restart pod (temporary)
* Open bug for developers

### Developer Action

* Heap dump
* Find leaked objects
* Fix code
* Deploy

---

## 2. Traffic Spike

Example

```text
100 Users

↓

10000 Users
```

Memory increases because more requests are processed.

### DevOps Action

* Check traffic metrics
* Scale replicas
* Increase memory only if workload requires it

---

## 3. Memory Limit Too Small

Application needs

```text
3 GB
```

Pod limit

```text
2 GB
```

OOM occurs every day.

### Solution

Increase memory limit.

---

## 4. Large File Processing

Bad approach

```text
Read entire 10 GB file

↓

Store in memory

↓

OOM
```

Better approach

```text
Read 1000 rows

↓

Process

↓

Delete

↓

Read next 1000 rows
```

Developer changes application logic.

---

## 5. Cache Growing Forever

Bad

```text
Cache

↓

Never remove old data
```

Good

```text
Cache Size = 1000

Old data removed automatically
```

Developers add cache limits.

---

## 6. Bad Deployment

Yesterday

```text
500 MB
```

Today

```text
2 GB
```

Nothing changed except deployment.

### DevOps Action

Rollback.

Developers investigate.

---

## 7. Batch Jobs

Every night

```text
2 AM

Generate Reports
```

Memory spikes.

### Solution

Optimize batch processing.

Increase memory only if workload genuinely requires it.

---

## 8. Large Database Queries

Bad

```sql
SELECT *
FROM Orders;
```

Returns

```text
50 million rows
```

Application loads everything into memory.

Better

* Pagination
* Streaming
* Smaller batches

---

## 9. Too Many Threads or Workers

One worker

```text
100 MB
```

100 workers

```text
100 × 100 MB

=

10 GB
```

Developers reduce worker count or optimize concurrency.

---

# Temporary Production Mitigation

As a DevOps Engineer, your responsibility is to keep the service available while the root cause is investigated.

Possible temporary actions:

* Restart the pod
* Delete the unhealthy pod (Kubernetes recreates it)
* Roll back to a previous deployment
* Scale replicas
* Increase memory temporarily (only if justified)

These actions reduce customer impact but **do not fix the underlying issue**.

---

# Permanent Fix

The permanent fix depends on the root cause.

| Root Cause             | Permanent Fix                                  |
| ---------------------- | ---------------------------------------------- |
| Memory leak            | Fix code and release new version               |
| Traffic spike          | Scale application or optimize performance      |
| Memory limit too small | Increase memory requests and limits            |
| Large file             | Stream or process in chunks                    |
| Large database query   | Pagination or streaming                        |
| Cache growth           | Add cache eviction policy                      |
| Too many workers       | Reduce concurrency or optimize worker usage    |
| Batch job              | Optimize processing or allocate more resources |
| Bad deployment         | Fix regression and redeploy                    |

---

# Commands Every DevOps Engineer Should Know

## Verify OOMKilled

```bash
kubectl describe pod <pod-name>
```

---

## View Previous Logs

```bash
kubectl logs <pod-name> --previous
```

---

## Pod Memory Usage

```bash
kubectl top pod
```

---

## Node Memory Usage

```bash
kubectl top node
```

---

## Running Processes

```bash
kubectl exec -it <pod-name> -- ps aux
```

---

## Live Process Memory

```bash
kubectl exec -it <pod-name> -- top
```

---

## Linux Memory Usage

```bash
top
```

```bash
htop
```

```bash
ps aux --sort=-%mem
```

---

# Senior DevOps Mindset

A junior engineer sees:

```text
OOMKilled
```

and asks:

> How much memory should I add?

A senior DevOps engineer asks:

* Was it really OOMKilled?
* Which process used the memory?
* What was the application doing?
* Did memory grow gradually or spike suddenly?
* Did traffic increase?
* Was there a deployment?
* Was a scheduled job running?
* Is the node under memory pressure?
* Is this a code issue or a capacity issue?

Only after answering these questions do they decide on the appropriate action.

---

# Golden Rule

**Never treat OOMKilled as the problem.**

OOMKilled is only the **symptom**.

The real job is to discover **why the application needed more memory**.

Only after identifying the root cause should you decide whether to:

* Fix the application
* Scale the application
* Increase memory
* Roll back a deployment
* Optimize the workload

This approach is how production teams handle OOMKilled incidents in real-world environments.
