# Eventual Consistency

`Eventual consistency` is a `data consistency model used in distributed databases where all replicas will eventually show the same data, provided no new updates are made`. It sacrifices immediate consistency to achieve high availability and fast performance across a distributed network

<br/>

Distributed databases replicate data across multiple servers (often globally) to prevent downtime and reduce latency.

**The Challenge**: Updating every single replica around the world at the exact same millisecond slows down system performance

**The Solution**: Eventual consistency allows a server to respond to a user immediately after a local update, while asynchronously copying that update to all other replicas in the background

<br/>

## How It Works (The Timeline)

**The Update**: A user updates their profile picture on Server A.

**The Lag**: For a brief window, Server B and Server C still show the old picture.

**The Sync**: The database actively propagates the update across the network.

**Consistency**: After a short period (milliseconds to seconds), all servers sync up and hold the identical, newest data.