**ACID** describes four properties that make database transactions reliable:

- **Atomicity**
- **Consistency**
- **Isolation**
- **Durability**

Suppose a transaction transfers $100 from account A to account B:

```
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';

COMMIT;
```

### Atomicity

The transaction happens completely or not at all.

If the second update fails, the first update is rolled back. The database never completes only half of the transfer.

```
Both updates succeed → commit
Any update fails     → roll back both
```

### Consistency

A successful transaction preserves the database’s defined rules and invariants.

Examples include:

- Primary keys remain unique
- Foreign keys reference existing rows
- Check constraints remain true
- Application invariants, such as conservation of transferred money, are maintained

The database enforces declared constraints, while the transaction’s application logic must preserve business-level invariants.
Example: Bank balance cannot go -ve. DB rejects transactions that try this.

This is different from **consistency in CAP**, which usually means that distributed operations behave like a single, current copy of the data.

### Isolation

Concurrent transactions should not interfere in ways prohibited by the selected isolation level.

Without sufficient isolation:

```
Transaction 1 reads balance = 500
Transaction 2 reads balance = 500
Transaction 1 subtracts 100
Transaction 2 subtracts 200
```

One update might overwrite the other, causing a lost update.

Common isolation levels include:

```
Read uncommitted
Read committed
Repeatable read
Serializable
```

`Serializable` provides the strongest conventional guarantee: concurrent transactions behave as though they ran one at a time. Weaker levels permit more concurrency but may allow anomalies.

### Durability

After the database reports a successful commit, the result survives failures such as a process crash or power loss, within the system’s stated failure model.

Databases typically provide durability using:

- Write-ahead logs
- Journaling
- FUA or storage-cache flushes
- Replication
- Recovery processing after restart

In short:

```
Atomicity    All or nothing
Consistency  Preserve defined invariants
Isolation    Control interference between concurrent transactions
Durability   Committed changes survive failure
```