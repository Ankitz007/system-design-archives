# Availability

## Definition

The guarantee that the distributed system is operational and responsive to client requests. Every request sent to a non-failing node receives a non-error response within a reasonable time, though the response might not reflect the absolute latest data state (depending on consistency guarantees).

## Importance

High availability is critical for user satisfaction and service-level objectives (SLOs). Many applications (e.g., social media, online banking, SaaS platforms) must be accessible 24/7. Low availability can cause lost revenue and trust. In architecture, availability drives designs like multi-region deployments, failover strategies, and graceful degradation. Unlike reliability, which is about correctness, availability is about liveness: even if data is slightly out-of-date, an available system still responds to user requests.

## Availability in Sequence vs Parallel

If a service consists of multiple components prone to failure, the service's overall availability depends on whether the components are in sequence or in parallel.

### Sequence

Overall availability decreases when two components are in sequence:

```math
Availability_{Total} = Availability(A) \times Availability(B)
```

For example, if a service has two components A and B, and A has 99.9% availability while B has 99.5%, the overall availability is:

```math
Availability_{Total} = 0.999 \times 0.995 = 0.994505
```

This means that the service is only available 99.45% of the time, which is significantly lower than the availability of either component alone.

### Parallel

Overall availability increases when two components are in parallel:

```math
Availability_{Total} = 1 - (1 - Availability(A)) \times (1 - Availability(B))
```

For example, if a service has two components A and B, and A has 99.9% availability while B has 99.5%, the overall availability is:

```math
Availability_{Total} = 1 - (1 - 0.999) \times (1 - 0.995) = 0.999995
```

This means that the service is available 99.9995% of the time, which is significantly higher than the availability of either component alone. This is because if one component fails, the other component can still handle requests, ensuring that the service remains available.

## Challenges and Trade-offs

Achieving high availability in distributed systems involves mitigating failures and maintenance downtime:

- **Redundancy and Failover**: Like reliability, availability uses redundant components (servers, databases, network paths). For instance, deploying services in multiple Availability Zones (AZs) in AWS means that if one AZ fails, others serve traffic. However, redundancy increases cost and complexity.
- **Maintenance and Upgrades**: Systems must allow rolling upgrades or hot swaps so that taking a node down for maintenance doesn’t bring the service offline. Kubernetes, for example, can drain pods one at a time to avoid downtime.
- **Consistency vs Availability**: Per the CAP theorem, partitioned systems must choose consistency or availability. Many distributed databases (e.g., Cassandra, DynamoDB) choose to remain available at the cost of immediate consistency (they are AP systems). Others (e.g., Google Spanner, MongoDB) choose consistency (and hence become unavailable during some partitions) to avoid stale reads.
- **Latency and Load Balancing**: Ensuring availability under load often requires load balancers and auto-scaling. But under extremely high load (e.g., DDOS, traffic spikes), even well-scaled systems may slow or fail. Architectures must account for peak loads.
- **Complexity of Distributed Coordination**: Mechanisms that improve availability (like consensus groups) themselves have failure modes. For example, a split-brain scenario in leader election can momentarily make parts of the system unavailable until resolved.

## Techniques to Improve Availability

Improving the **availability** of a distributed system involves reducing downtime and ensuring the system continues to function even when components fail. Here are key techniques to achieve higher availability:

### 1. **Redundancy**

- Deploy multiple instances of services (horizontal scaling).
- Use replicated data storage (e.g., master-slave or multi-master replication).
- Avoid single points of failure.

### 2. **Failover and Recovery Mechanisms**

- Automatic failover to backup nodes or systems.
- Health checks and watchdog services to detect and replace unhealthy components.
- Graceful degradation (partial service still works if full service is down).

### 3. **Geographic Distribution**

- Deploy services across multiple regions or availability zones.
- Use anycast IPs or geo-aware DNS to route users to the nearest healthy server.

### 4. **Load Balancing**

- Distribute requests across multiple servers to avoid overloading.
- Use smart load balancers that detect and avoid unhealthy nodes.

### 5. **Data Replication and Partitioning**

- Use consistent and partition-tolerant data stores (e.g., Cassandra, DynamoDB).
- Apply quorum-based reads/writes for higher consistency without total downtime.

### 6. **Graceful Handling of Failures**

- Use circuit breakers and retries with backoff to avoid cascading failures.
- Implement timeouts and rate limiting.

### 7. **Monitoring and Alerting**

- Real-time system monitoring to detect issues early.
- Alerting mechanisms for quick human intervention if needed.

### 8. **Versioning and Safe Deployments**

- Use blue-green or canary deployments to minimize impact from faulty updates.
- Rollback mechanisms in case of deployment failures.

### 9. **Stateless Services**

- Design services to be stateless when possible to allow easy replication and replacement.
- Store state in reliable external systems (e.g., distributed caches, databases).

### 10. **Chaos Engineering**

- Intentionally inject failures to test resilience (e.g., using Netflix’s Chaos Monkey).
- Helps ensure the system behaves correctly under failure conditions.
