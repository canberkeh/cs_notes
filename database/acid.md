# ACID in Databases

ACID is a set of rules that helps databases keep data correct, even when many
users are using the system at the same time or when something goes wrong.

ACID stands for:

- Atomicity
- Consistency
- Isolation
- Durability

You usually hear about ACID when learning about database transactions.


## What Is a Transaction?

A transaction is a group of database operations that should be treated as one
unit of work.

Real-world example: transferring money.

```text
Move $100 from Alice to Bob.
```

That sounds like one action, but the database may need to do multiple steps:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE owner = 'Alice';

UPDATE accounts
SET balance = balance + 100
WHERE owner = 'Bob';
```

These two updates must succeed together or fail together. If Alice loses $100
but Bob never receives it, the database has created a serious problem.

That is why transactions matter.
ACID is a set of guarantees that define how transactions should behave.


## Basic Transaction Example

In SQL, a transaction often looks like this:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE owner = 'Alice';

UPDATE accounts
SET balance = balance + 100
WHERE owner = 'Bob';

COMMIT;
```

If something fails, the database can undo the transaction:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE owner = 'Alice';

-- Something goes wrong here.

ROLLBACK;
```

`COMMIT` means "save these changes permanently."

`ROLLBACK` means "undo everything in this transaction."


## A Is for Atomicity

Atomicity means a transaction is all-or-nothing.

Either every part of the transaction succeeds, or none of it is saved.

### Real-World Example: Bank Transfer

Imagine Alice sends $100 to Bob.

Bad result:

```text
Alice balance: -$100
Bob balance: unchanged
```

That means the system took money from Alice but failed to give it to Bob.

Atomicity prevents that.

With atomicity, the database says:

```text
If I cannot complete the whole transfer, I will undo the whole transfer.
```

### Python-Style Example

```python
def transfer_money(db, from_account, to_account, amount):
    try:
        db.begin()

        db.execute(
            "UPDATE accounts SET balance = balance - ? WHERE id = ?",
            [amount, from_account],
        )
        db.execute(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?",
            [amount, to_account],
        )

        db.commit()
    except Exception:
        db.rollback()
        raise
```

The important idea is not the exact database library. The idea is this:

```text
commit if everything works
rollback if anything fails
```


## C Is for Consistency

Consistency means a transaction must move the database from one valid state to
another valid state.

The database should not accept changes that break rules.

### Real-World Example: Online Store Inventory

Imagine an online store has this rule:

```text
Product stock cannot be negative.
```

If a product has 3 items left, the system should not allow 5 people to buy it.

Bad result:

```text
Product: Keyboard
Stock: -2
```

That is inconsistent data.

The database can protect consistency with constraints:

```sql
CREATE TABLE products (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    stock INTEGER NOT NULL CHECK (stock >= 0)
);
```

Now the database refuses invalid stock values.

### Banking Example: Money Transfer

A bank has this rule:

​```text
The total money in the system must never change.
​```

If Aylin sends $100 to Mehmet:

​```text
Aylin:  $500 → $400
Mehmet: $300 → $400
Total:  $800 → $800 ✅
​```

If the system crashes after Aylin's account is debited but before Mehmet's is credited:

​```text
Aylin:  $500 → $400
Mehmet: $300 → $300
Total:  $800 → $700 ❌
​```

That is inconsistent. The database can protect against this with a constraint:

​```sql
ALTER TABLE accounts
ADD CONSTRAINT positive_balance CHECK (balance >= 0);
​```

Now the database refuses any transaction that would leave an account below zero.

## I Is for Isolation

Isolation means transactions should not interfere with each other in unsafe
ways.

This matters when multiple users are using the system at the same time.

### Real-World Example: Buying the Last Ticket

Imagine there is 1 concert ticket left.

Two users click "Buy" at almost the same time:

```text
User A sees 1 ticket left.
User B sees 1 ticket left.
User A buys the ticket.
User B also buys the ticket.
```

Bad result:

```text
Tickets left: -1
```

Isolation helps prevent both transactions from acting as if they are the only
one running.

The database can make one transaction wait for the other, or it can reject one
of the purchases.

### Python-Style Example

```python
def buy_ticket(db, event_id, user_id):
    try:
        db.begin()

        ticket_count = db.query_one(
            "SELECT tickets_left FROM events WHERE id = ? FOR UPDATE",
            [event_id],
        )

        if ticket_count == 0:
            raise ValueError("Sold out")

        db.execute(
            "UPDATE events SET tickets_left = tickets_left - 1 WHERE id = ?",
            [event_id],
        )
        db.execute(
            "INSERT INTO purchases (event_id, user_id) VALUES (?, ?)",
            [event_id, user_id],
        )

        db.commit()
    except Exception:
        db.rollback()
        raise
```
With this approach, one user gets the ticket and the other receives a
"Sold out" error — the stock never drops below zero.

`FOR UPDATE` locks the row until the transaction finishes.
Other transactions that try to read or update the same row must wait.
This prevents two users from buying the same last ticket simultaneously.

The exact syntax depends on the database, but the concept is common.

## D Is for Durability

Durability means that once a transaction is committed, the data should survive
crashes, restarts, and power failures.

### Real-World Example: Payment Confirmation

Imagine you pay for an order and the website says:

```text
Payment successful.
```

Right after that, the server crashes.

Durability means the system should not forget your payment after it restarts.

Once the database commits the transaction, the payment record should still be
there.

### Another Example: Hospital Records

If a doctor updates a patient's medication and the transaction commits, that
update must survive a crash.

Bad result:

```text
Doctor saves new medication.
System says saved.
Server restarts.
Medication update disappears.
```

Durability prevents that kind of data loss.

Databases achieve durability by writing transactions to a commit log on disk
before confirming success. If the system crashes, the log is replayed on
restart to restore the committed state.


## ACID Summary Table

| Letter | Name | Meaning | Real-World Example |
| --- | --- | --- | --- |
| A | Atomicity | All operations succeed or none do | Money is not removed from Alice unless Bob receives it |
| C | Consistency | Data rules remain valid | Stock cannot become negative |
| I | Isolation | Transactions do not interfere unsafely | Two people cannot both buy the last ticket |
| D | Durability | Committed data survives failures | A confirmed payment remains after a server crash |


```text
Atomicity: half-finished work
Consistency: invalid data
Isolation: users interfering with each other
Durability: saved data disappearing
```

### Mistake 1: Thinking Atomicity Means Fast

Atomicity does not mean a transaction is fast. It means the transaction is
all-or-nothing.

### Mistake 2: Thinking Consistency Means "Data Looks Organized"

Consistency means data follows rules and constraints.

Examples:

- Email cannot be null.
- Account balance cannot go below zero.
- Short code must be unique.
- Order must belong to an existing user.

### Mistake 3: Ignoring Concurrent Users

Your code may work with one user but fail with many users at once.

Example:

```text
One item left.
Two users buy it at the same time.
Both orders succeed.
```

That is an isolation problem.

### Mistake 4: Saying "Saved" Before Commit

Do not tell the user something was saved if the transaction has not committed.

If the commit fails, the user may believe the operation succeeded when it did
not.


## Practice Exercise

Imagine you are building a food delivery app.

A user places an order. The system must:

1. Create the order.
2. Charge the customer.
3. Reduce restaurant inventory.
4. Assign a delivery driver.

Answer these questions:

1. What could go wrong if atomicity is missing?
2. What consistency rules should exist?
3. What isolation problems could happen during a busy lunch rush?
4. What data must be durable after the order is confirmed?

---

**1. What could go wrong if atomicity is missing?**

```text
The customer might be charged but the order might not be created.
The inventory might be reduced but the driver might never be assigned.
The order might be created but the customer might never be charged.
Any of the four steps could succeed while the others fail.
```

**2. What consistency rules should exist?**

```text
Order total must equal the sum of item prices.
Customer balance cannot go below zero after charge.
Restaurant inventory cannot go below zero.
An order must belong to a customer that exists.
An order must belong to a restaurant that exists.
An order cannot be assigned to a driver that does not exist.
Delivery address cannot be null.
```

**3. What isolation problems could happen during a busy lunch rush?**

```text
Two customers order the last item at the same time.
Both see 1 item left, both orders succeed, inventory drops to -1.

Two drivers are assigned to the same order at the same time.
One order ends up with two drivers.

A customer's balance is read by two transactions at the same time.
Both charge the customer, the balance drops below zero.
```

**4. What data must be durable after the order is confirmed?**

```text
The order and its items.
The charge to the customer.
The reduced inventory.
The assigned driver.
The delivery address.
The confirmation timestamp.
```

Atomicity:
All order operations must either succeed together or fail together.

Consistency:
Business invariants must always remain valid.
Inventory cannot become negative.
A paid order must have a valid payment record.
Order totals must match item totals.

Isolation:
Concurrent orders during peak traffic must not interfere with each other.
Race conditions could cause overselling inventory,
double charging customers,
or assigning multiple drivers to the same order.

Durability:
Once an order is confirmed,
the order record, payment record,
inventory updates, and driver assignment
must survive crashes, restarts, and power failures.