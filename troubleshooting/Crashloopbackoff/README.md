| Rank   | Reason                                     | Frequency         | DevOps Action                                                                    |
| ------ | ------------------------------------------ | ----------------- | -------------------------------------------------------------------------------- |
| **1**  | **Memory Leak**                            | ⭐⭐⭐⭐⭐ Very Common | Collect logs, analyze memory trend, restart pod (temporary), developers fix code |
| **2**  | **Memory Limit Too Low**                   | ⭐⭐⭐⭐⭐ Very Common | Verify actual memory usage, increase memory if workload genuinely needs it       |
| **3**  | **Traffic Spike / High Load**              | ⭐⭐⭐⭐ Common       | Scale replicas, enable autoscaling, monitor memory                               |
| **4**  | **Large File or Large Payload Processing** | ⭐⭐⭐⭐ Common       | Identify API/job, recommend streaming or chunk processing                        |
| **5**  | **Bad Deployment (Regression)**            | ⭐⭐⭐ Common        | Roll back deployment, compare with previous version                              |
| **6**  | **Large Database Query**                   | ⭐⭐⭐ Common        | Identify query, recommend pagination or batching                                 |
| **7**  | **Unbounded Cache Growth**                 | ⭐⭐⭐ Common        | Verify cache behavior, recommend cache size limits                               |
| **8**  | **Batch Jobs / CronJobs**                  | ⭐⭐ Less Common    | Check scheduled jobs, allocate resources or optimize job                         |
| **9**  | **Too Many Threads / Workers**             | ⭐⭐ Less Common    | Reduce concurrency or optimize worker configuration                              |
| **10** | **Node Memory Pressure**                   | ⭐⭐ Less Common    | Check node memory usage, redistribute workloads or add nodes                     |
