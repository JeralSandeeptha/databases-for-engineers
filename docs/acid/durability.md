# Durability

`Durability` `guarantees that once a transaction is successfully completed and committed, its changes are permanently saved` in a non volatile system. This means the data survives subsequent system failures, power outages, or crashes

<br/>

## Durability Techniques

These `three items` are specific mechanism types used by database engines to ensure Durability (saving data permanently) while keeping system performance high.

○ WAL - Write ahead log
○ Asynchronous snapshot
○ AOF - Append only file

### WAL

**What it is**: A technique where changes are written to a permanent log before they are applied to the actual database pages.

**How it works**: When a transaction happens, the database writes the action to a sequential, disk-based log file (the WAL). Because sequential writing is incredibly fast, the database can safely confirm the transaction is durable without waiting for the slower process of updating the main database files.

**Commonly used in**: PostgreSQL, MySQL (InnoDB Redo Log), and SQLite

### Asynchronous snapshot

**What it is**: A technique that takes a full "point-in-time" photo of the entire database state at scheduled intervals in the background.

**How it works**: The database runs normally in memory. Periodically (e.g., every 5 minutes), a background process copies the current memory state to a permanent file on disk. It is "asynchronous" because the user does not have to wait for the snapshot to finish to complete their transaction.

**Trade-off**: High performance, but if the system crashes between snapshots, any data modified since the last snapshot is lost.

**Commonly used in**: Redis (RDB snapshots).

### AOF

**What it is**: A log-based technique that records every single write operation received by the server.

**How it works**: Instead of taking massive snapshots of the data, the database constantly appends raw commands (like SET, INSERT, or DEL) to the end of a continuous file. If the system crashes, the database simply replays this entire file from top to bottom to rebuild the exact state.

**Commonly used in**: Redis (AOF mode)
