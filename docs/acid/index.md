# ACID Principles

It ensures data remains accurate and valid, even in the event of system failures or concurrent errors.

<br/>

## Pillars

**Atomicity (All or Nothing)**: Transactions are treated as indivisible units. If one part of a transaction fails, the entire transaction is canceled (rolled back) and no changes are applied to the database
[READ MORE](./atomocity.md)

**Consistency (Valid State)**: A transaction can only bring the database from one valid and consistent state to another. All data must follow predefined rules, constraints, and triggers
[READ MORE](./consistency.md)

**Isolation (Independent Execution)**: When multiple transactions run concurrently, they do not interfere with each other. Each transaction acts as if it is the only one operating on the system, preventing data corruption from overlapping operations
[READ MORE](./isolation.md)

**Durability (Permanent Changes)**: Once a transaction is successfully completed (committed), its changes are permanently saved, even in the event of power loss, crashes, or system errors
[READ MORE](./durability.md)

<br/>

## Common Use Cases

Primarily encounter ACID-compliant databases in industries that require rigorous transaction handling.

Notable applications include:

- **Banking & Finance**: Processing fund transfers so that money is not duplicated or lost.
- **E-Commerce**: Managing inventory counts and customer orders during checkout.
- **Healthcare**: Storing and updating patient records reliably.
