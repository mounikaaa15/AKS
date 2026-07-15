# AKS Networking in Real-Time Production

In Azure Kubernetes Service (AKS), **networking decides how Pods, Nodes, Services, and external users communicate**.

When you create an AKS cluster, you must choose a network model:

1. **Azure CNI**
2. **Azure CNI Overlay**
3. **Kubenet (legacy in many scenarios)**

The main difference is **where Pod IP addresses come from and how traffic flows**.

---

# 1. Azure CNI (Traditional / Direct VNet Integration)

## Concept

With **Azure CNI**, every Kubernetes Pod gets an IP address directly from your Azure Virtual Network (VNet).

Meaning:

```
Azure VNet
|
|-- Node 10.0.1.4
|
|-- Pod 10.0.1.5
|
|-- Pod 10.0.1.6
|
|-- Database 10.0.2.10
```

Pods behave like normal Azure resources.

They can communicate directly with:

* Other Pods
* Virtual Machines
* Azure SQL
* Storage accounts
* Private endpoints
* On-premises networks through VPN/ExpressRoute

---

# Real Production Example

Imagine a banking application.

Architecture:

```
Users
 |
Application Gateway
 |
AKS Cluster
 |
+----------------+
| Node           |
|                |
| Pod: Frontend  | 10.0.1.20
| Pod: API       | 10.0.1.21
+----------------+
 |
 |
Azure SQL
10.0.2.50
```

The API Pod directly talks to Azure SQL:

```
API Pod
10.0.1.21
     |
     |
Azure SQL
10.0.2.50
```

No extra network translation is required.

---

## Advantages

✅ Better performance
✅ Simple routing
✅ Easy integration with Azure services
✅ Good for enterprise environments

---

## Disadvantage

The problem:

Every Pod consumes a VNet IP.

Example:

```
Subnet:
10.0.1.0/24

Available IPs:
254
```

If you have:

```
100 Nodes

Each node:
30 Pods
```

You need:

```
100 x 30 = 3000 IP addresses
```

Your subnet can run out of IPs.

---

## Used in Production When:

* Enterprise applications
* Large private networks
* Need direct access to Azure resources
* Hybrid cloud environments

Example:

```
AKS
 |
ExpressRoute
 |
Company Data Center
```

---

# 2. Azure CNI Overlay (Modern Recommended Approach)

## Concept

Azure CNI Overlay separates:

* Node IP network
* Pod IP network

Nodes get Azure VNet IPs.

Pods get private IPs from a separate Pod CIDR.

Example:

```
Azure VNet

Node Network:

10.0.0.0/16


Pod Network:

192.168.0.0/16
```

---

Real example:

```
Azure VNet

Node 1
10.0.1.4
 |
 |
 +---- Pod A
 |     192.168.1.5
 |
 +---- Pod B
       192.168.1.6


Node 2
10.0.1.5
 |
 |
 +---- Pod C
       192.168.2.5
```

Pods do not consume Azure VNet IPs.

---

# Traffic Flow Example

A frontend Pod calls an API Pod:

```
Frontend Pod
192.168.1.5

       |
       |
       v

API Pod
192.168.2.5
```

Azure networking creates an overlay tunnel:

```
Pod Network
192.168.x.x

        |
        |
Overlay

        |
        |

Node Network
10.0.x.x
```

The nodes carry the traffic.

---

# Production Example

Large e-commerce application:

```
Internet
 |
Application Gateway
 |
AKS
 |
--------------------------------
Frontend Pods (500)
Backend Pods (1000)
Payment Pods (50)
--------------------------------
 |
Azure SQL
Redis
Storage
```

Instead of needing thousands of VNet IP addresses:

```
Node IPs:
10.0.x.x

Pod IPs:
192.168.x.x
```

---

## Advantages

✅ Saves Azure VNet IP addresses
✅ Supports very large clusters
✅ Easier subnet planning
✅ Recommended for many new AKS deployments

---

## Disadvantage

Some advanced networking scenarios need extra configuration.

Example:

Directly connecting external systems to Pod IPs is not the same as traditional Azure CNI.

---

## Used in Production When:

* Large microservice platforms
* Many Pods
* Multiple teams sharing clusters
* Need IP scalability

Example:

```
100 Nodes

Each node:
100 Pods

Total:
10,000 Pods
```

Azure CNI Overlay handles this better.

---

# 3. Kubenet (Legacy Networking)

## Concept

Kubenet uses:

* Azure VNet IPs for Nodes
* Separate internal network for Pods

But unlike Azure CNI Overlay, Kubernetes manages routing.

Example:

```
Azure VNet

Node:

10.0.1.5


Pod:

10.244.1.10
```

The Pod IP is not a real Azure VNet IP.

Azure does not know about the Pod directly.

---

Traffic:

```
User
 |
Load Balancer
 |
Node
 |
Kubernetes Routing
 |
Pod
```

---

# Real Production Example

Small application:

```
Users
 |
Azure Load Balancer
 |
AKS Node

10.0.1.5

 |
 |
Pod

10.244.1.20
```

The node performs routing/NAT to send traffic to the Pod.

---

## Advantages

✅ Uses fewer VNet IP addresses
✅ Simple for small clusters
✅ Older AKS deployments commonly used it

---

## Disadvantages

❌ More routing complexity
❌ Limited compared with Azure CNI
❌ Not preferred for new large deployments

---

# Production Comparison

| Feature                      | Azure CNI  | Azure CNI Overlay | Kubenet           |
| ---------------------------- | ---------- | ----------------- | ----------------- |
| Pod IP source                | Azure VNet | Separate Pod CIDR | Separate Pod CIDR |
| Pod gets Azure IP            | Yes        | No                | No                |
| IP consumption               | High       | Low               | Low               |
| Performance                  | Excellent  | Excellent         | Good              |
| Scalability                  | Medium     | High              | Medium            |
| Recommended for new clusters | Yes        | Yes               | Usually no        |
| Enterprise usage             | High       | Very high         | Decreasing        |

---

# Real Company Architecture Example

## Modern Production AKS

```
                 Internet
                    |
                    |
          Azure Application Gateway
                    |
                    |
                 AKS Cluster

        Azure CNI Overlay Networking

        +-----------------------+
        | Node Pool             |
        |                       |
        | Frontend Pods         |
        | API Pods              |
        | Worker Pods           |
        +-----------------------+

                    |
          Private Endpoint

                    |
              Azure SQL
              Redis Cache
              Storage
```

---

# How to choose in real projects?

### Choose Azure CNI when:

```
Need:
- Pod has direct Azure IP
- Hybrid connectivity
- Strict enterprise networking
```

Example:
Banking, healthcare, government.

---

### Choose Azure CNI Overlay when:

```
Need:
- Thousands of Pods
- Large microservices
- Efficient IP usage
```

Example:
E-commerce, SaaS platforms.

---

### Choose Kubenet when:

```
Need:
- Small existing clusters
- Legacy AKS environments
```

For new production systems, Azure CNI or Azure CNI Overlay are usually preferred.
