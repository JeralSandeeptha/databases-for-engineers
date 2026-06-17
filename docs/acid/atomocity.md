# Atomocity

- **Atomicity** It indictates that a transaction is treated as a `single`, `indivisible` `unit of work`
- The "all-or-nothing" rule
- Either all operations within the transaction succeed, or none of them are applied, preventing partial updates that could corrupt data
- All queries in transactions must succeed
- If any query fails all other successful quieres must be invalid
- If the database went down `prior to a commit` of a transaction, all the successful queries in the transactions should rollback
- `Commit` phase is the phase that actually supports to Atomocity
- Lack of atomocity leads to `Inconsistency` which fails `Consistency`

<br/>

## Key Concepts

- **Indivisibility**: You cannot execute half of a transaction and leave the rest pending.
- **Rollback Mechanism**: If any individual step in a transaction fails, the database automatically undoes (or rolls back) all preceding steps.
- **No Partial States**: The database never ends up in a broken or halfway-updated state.
