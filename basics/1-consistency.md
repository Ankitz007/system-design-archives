# Consistency

## Definition

Consistency refers to the guarantee that all nodes or replicas in a distributed system agree on the same data value at any given time, or see updates in a predictable order. It dictates the "freshness" and uniformity of data that clients read.

> [!IMPORTANT]
> It's about answering a seemingly simple question: **"If I write data here, when and how will it be readable elsewhere?"**

## Importance

Consistency is crucial in distributed systems to ensure that all clients see the same data at the same time, especially in scenarios where multiple nodes can read and write data concurrently. Inconsistent data can lead to confusion, errors, and unexpected behavior in applications.

## The Challenges

Distributed systems face several obstacles due to their reliance on multiple nodes and inherently unreliable networks. Key challenges include:

- **Network Failures**: Messages may be delayed, lost, or arrive out of order, causing discrepancies in data across nodes.
- **Node Crashes**: Unexpected node failures can result in the loss of recent updates, complicating recovery and consistency.
- **Latency**: Communication delays can lead to nodes temporarily holding different versions of data, impacting real-time consistency.

These challenges highlight the complexity of maintaining consistency in distributed environments.

## Types of Consistency Models

Consistency models describe how distributed systems manage data across multiple nodes. They provide a contract between the system and the programmer, defining how data should behave when read, written, or updated. For example:

- **Strong consistency** guarantees any read reflects the most recent write, regardless of which node is accessed.
- **Eventual consistency** guarantees all nodes will converge to the same state over time but allows temporary discrepancies between them.

These models help navigate the trade-offs between consistency, availability, and performance. Selecting the right model ensures the system meets technical demands and user expectations, especially under adverse conditions like network partitions or heavy loads.

Let’s explore four commonly used consistency models: **Strong Consistency**, **Sequential Consistency**, **Causal Consistency**, and **Eventual Consistency**. Understanding the nuances will help you choose the right model for your application’s needs.

### 1. Strong Consistency

Strong consistency, also known as **linearizability**, guarantees that all clients see the latest data immediately after a write. Every operation occurs instantaneously at some point between its invocation and completion, behaving as if there were a single copy of the data. This approach guarantees a unified, real-time view of data across all nodes, making it easier to reason about correctness in distributed systems.

#### Key Techniques

- **Consensus protocols** (e.g., Raft, Paxos): A leader coordinates writes by reaching a quorum among nodes to ensure correctness, even during failures.
- **Real-time order (linearizability)**: Operations are serialized, and their real-time invocation order is respected, meaning a write completed at one node is immediately visible to all subsequent reads across the system.

#### Trade-offs

**Pros**:

- Guarantees correctness by eliminating stale reads and preventing anomalies like double-spending.
- Simplifies application logic with consistent data views.

**Cons**:

- High latency due to quorum-based operations required for synchronization.
- Reduced availability during network partitions, as writes may block until consensus is achieved.

#### Use Cases

- **Financial transactions and ledgers**: Banking and payment applications where real-time correctness and strict ordering of operations are critical to prevent anomalies such as double-spending or inconsistent balances.
- **Distributed coordination**: Systems like leader election and distributed locks require strict synchronization to ensure tasks or resources are managed consistently across nodes.
- **Configuration and metadata management**: Distributed systems often rely on shared configuration or metadata (e.g., system settings, quotas, or state information). Strong consistency ensures updates to these values are immediately synchronized across all nodes, preventing conflicting states.

### Real-World Example

- **Google Spanner**: A globally distributed database that uses a combination of atomic clocks and consensus algorithms to provide strong consistency across data centers, ensuring that all nodes see the same data at the same time.
- **Zookeeper**: A distributed coordination service that uses a consensus protocol to ensure strong consistency for configuration management and leader election, making it suitable for applications requiring strict synchronization.
- **Distributed databases**: Systems like CockroachDB and Google Spanner use strong consistency to ensure that all replicas of a piece of data are synchronized, providing a single source of truth for applications.
- **Distributed file systems**: Systems like HDFS or Ceph may use strong consistency for metadata operations to ensure that file system operations are atomic and consistent across all nodes.

### 2. Sequential Consistency

Sequential consistency ensures that all operations occur in a logical order. The execution results are as if all operations were executed individually, maintaining the order in which each client issues operations. Unlike strong consistency, sequential consistency does not enforce real-time ordering, allowing for performance optimizations while preserving predictable behavior.

#### Key Techniques

- **Global ordering**: All clients observe operations in the same sequence, ensuring no client sees events out of order.
- **Relaxed synchronization**: Operations do not need to appear instantaneous, reducing overhead compared to strong consistency.

#### Trade-offs

**Pros**:

- Lower synchronization overhead compared to strong consistency, as strict real-time guarantees are not required.
- Well-suited for scenarios where the sequence of operations matters more than immediate visibility.

**Cons**:

- Slower than eventual consistency due to global ordering constraints.
- Lack of real-time guarantees can cause anomalies in time-sensitive applications.

#### Use Cases

- **Gaming systems**: Ensures actions happen in the correct order for all players, such as moves in turn-based games, even if operations aren’t synchronized in real-time.
- **Collaborative editing**: Guarantees ordered application of updates in shared workspaces, like document editing platforms, ensuring all collaborators’ edits appear in the correct sequence.

#### Real-World Example

- **Distributed databases**: Some databases, like Amazon DynamoDB with specific configurations, can provide sequential consistency for certain operations, ensuring that all clients see updates in the same order.
- **Distributed file systems**: Systems like Google File System (GFS) may provide sequential consistency for file operations, ensuring that all clients see file updates in the same order, even if they are not immediately visible.
- **Distributed messaging systems**: Systems like Apache Kafka can provide sequential consistency for message delivery, ensuring that all consumers see messages in the order they were produced, even if they are not immediately visible to all consumers.
- **Distributed caches**: Systems like Redis can provide sequential consistency for certain operations, ensuring that all clients see updates in the same order, even if they are not immediately visible to all clients.

### 3. Causal Consistency

Causal consistency ensures that operations with a cause-and-effect relationship are observed in the correct order across nodes. This model strikes a balance between consistency and performance, making it suitable for collaborative applications or systems that do not require strict real-time guarantees.

#### Key Techniques

- **Dependency tracking**: Mechanisms like vector clocks or versioned timestamps are used to capture and respect causal relationships between operations.
- **Partial ordering**: Causally related events are applied in sequence, while unrelated operations are not constrained by global ordering.

#### Trade-offs

**Pros**:

- Provides intuitive behavior for collaborative or real-time systems.
- Balances consistency and performance effectively.

**Cons**:

- More complex than eventual consistency due to the need for dependency tracking.
- Slightly higher latency compared to eventual consistency.
- Does not enforce total ordering across the entire system.

#### Use Cases

- **Collaborative platforms**: Applications like Google Docs or shared workspaces, where causally related updates (e.g., editing text and applying formatting) must be applied in the correct order.
- **Messaging apps**: Ensures messages are delivered in the order they were sent, maintaining logical coherence in conversations without requiring global synchronization.

#### Real-World Example

- **Amazon DynamoDB**: Offers a causal consistency option, allowing applications to read data in a way that respects the causal relationships between writes, making it suitable for collaborative applications.
- **Google Docs**: Uses causal consistency to ensure that edits made by one user are visible to others in the correct order, even if they are not immediately synchronized.
- **Distributed databases**: Some databases, like Riak and Couchbase, can provide causal consistency for certain operations, ensuring that all clients see updates in the correct order based on their causal relationships.
- **Distributed caches**: Systems like Redis can provide causal consistency for certain operations, ensuring that all clients see updates in the correct order based on their causal relationships.

### 4. Eventual Consistency

Eventual consistency allows for temporary inconsistencies between nodes, with a guarantee that all nodes will eventually converge to the same state. This model is often employed in AP systems, sacrificing immediate consistency to ensure the system remains responsive during network failures or partitions. In practice, eventual consistency means that updates propagate asynchronously across replicas, leading to temporary discrepancies. However, the system guarantees that all nodes eventually synchronize to the same state.

#### Key Techniques

- **Asynchronous replication**: Writes propagate in the background, allowing low-latency updates but causing temporary discrepancies.
- **Gossip protocols**: Nodes communicate in a peer-to-peer fashion to share updates, ensuring all nodes converge to the same data.
- **Anti-entropy**: Techniques that use data structures like Merkle trees to detect and reconcile differences between replicas.
- **Merge conflict resolution**: Resolves conflicting writes using strategies like last-write-wins, vector clocks, or custom logic.

#### Session Guarantees

Session guarantees aim to improve user experience by ensuring that a client’s actions (writes) are consistently visible to users, even when the system is in an inconsistent state. These guarantees include:

- **Read Your Writes (RYW)**: Once a client performs a write, all subsequent reads within the same session will reflect that write. This method ensures that users see their updates, even when the system is not fully consistent across all nodes.
- **Monotonic Reads**: Once a client sees a particular value, subsequent reads will never show an older value, preventing backward jumps in the data.
- **Writes Follow Reads**: After reading a value, all subsequent operations will see any writes by that client, ensuring logical consistency within the session.

#### Trade-offs

**Pros**:

- High availability and low latency, even during partitions.
- Scales well for high-demand, read-heavy systems.

**Cons**:

- Temporary inconsistencies may lead to stale or conflicting reads.
- Application logic must handle potential data conflicts.

#### Use Cases

- **Web caching**: Ensures high-speed data access with tolerable temporary inconsistencies, such as serving stale session data or cached web pages.
- **Product recommendations**: Provides personalized suggestions without requiring real-time consistency, allowing recommendation systems to sync across nodes gradually.
- **Inventory counts**: Tracks stock across distributed warehouses or regions, tolerating brief inconsistencies while ensuring eventual convergence to accurate totals.

#### Real-World Example

- **Amazon DynamoDB**: Uses eventual consistency as its default model, allowing for high availability and low latency while ensuring that all replicas eventually converge to the same state.
- **Apache Cassandra**: Employs eventual consistency to provide high availability and partition tolerance, allowing for low-latency writes and reads while ensuring that all replicas eventually synchronize.
- **Riak**: A distributed database that uses eventual consistency to ensure high availability and partition tolerance, allowing for low-latency writes and reads while ensuring that all replicas eventually synchronize.

## Techniques to Achieve Consistency

To make a distributed system **more consistent**, several techniques can be employed to ensure data correctness, synchronization, and ordering across nodes. Here are the **main techniques**:

### 1. **Consensus Algorithms**

Ensure all nodes agree on a single value or sequence of events.

- **Examples:** Paxos, Raft, Zab (used by ZooKeeper)
- **Use:** Replication logs, leader election, maintaining consistency among replicas

✅ *Used in*: etcd, ZooKeeper, CockroachDB, Consul

### 2. **Write-Ahead Logging (WAL) / Replication Logs**

Changes are logged before being applied, allowing recovery and consistent replication.

- **Primary Use:** Database durability and replica synchronization
- **Combined With:** Raft or Paxos for ordering and durability

✅ *Used in*: PostgreSQL, CockroachDB, Kafka (log-based replication)

### 3. **Quorum-Based Protocols**

Require a majority (or quorum) of replicas to agree before committing a read or write.

- **Technique:** Ensure overlapping read/write sets to maintain consistency
- **Formulas:**

  - `R + W > N` (for quorum reads/writes in Dynamo-style systems)

✅ *Used in*: Cassandra, DynamoDB, Riak

### 4. **Synchronized Clocks / Logical Timestamps**

Use timestamps to order operations and avoid inconsistencies.

- **Techniques:**

  - **Physical Clocks:** e.g., Google's TrueTime API (Spanner)
  - **Logical Clocks:** Lamport clocks, vector clocks
  - **Hybrid Clocks:** Combine physical + logical time (e.g., HLC)

✅ *Used in*: Google Spanner (TrueTime), CRDTs, Causal consistency systems

### 5. **Versioning / Vector Clocks**

Track the history of objects to resolve conflicts and maintain causal relationships.

- **Use Case:** Detecting and resolving divergent versions
- **Conflict Resolution:** "Last write wins," manual merge, CRDTs

✅ *Used in*: Amazon Dynamo, Riak, CRDT-based systems

### 6. **Leader-Based Replication**

Designate a single leader to handle all writes, ensuring total order.

- **Followers replicate** the leader’s log to stay consistent.
- **Stronger consistency** than leaderless or peer-based models.

✅ *Used in*: ZooKeeper, Raft-based systems like etcd and CockroachDB

### 7. **Synchronous Replication**

Wait for all (or majority) replicas to acknowledge writes before confirming success.

- **Benefit:** Guarantees consistency across replicas
- **Downside:** Higher latency, potential for reduced availability

✅ *Used in*: traditional RDBMS clusters, Google Spanner

### 8. **Conflict-Free Replicated Data Types (CRDTs)**

Special data structures that can converge automatically even under eventual consistency.

- **Use Case:** Collaborative editing, offline sync
- **Benefit:** Strong eventual consistency without coordination

✅ *Used in*: Riak, Redis (experimental), Automerge, Yjs

## References

- [Consistency models in distributed systems](https://systemdr.substack.com/p/consistency-models-in-distributed?utm_source=%2Finbox%2Fsaved&utm_medium=reader2)
- [Navigating consistency in distributed systems](https://hazelcast.com/blog/navigating-consistency-in-distributed-systems-choosing-the-right-trade-offs/)
