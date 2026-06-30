# CAP Theorem, ACID/BASE & Consistency

---

## CAP Theorem

In distributed systems, you can only guarantee **two of three** at the same time:

```
C → Consistency       : Every node sees the same data at the same time
A → Availability      : System always responds to requests
P → Partition Tolerance: System keeps working even if network breaks between nodes
```

> **P is always required** — network partitions are inevitable in real life.
> The real choice is always **C vs A**.

---

## CP — Consistency + Partition Tolerance

When network partition occurs, **sacrifice availability to protect consistency.**

### Payment System — CP

**Scenario: Double Spend**

```
User balance: 100 TL
Two simultaneous requests:
  Request A → Istanbul node : withdraw 100 TL
  Request B → Ankara node   : withdraw 100 TL

Network partition — two nodes cannot see each other.
```

**Without Consistency (if AP was chosen):**
```
Istanbul node → balance is 100 TL ✓ → approve → balance: 0 TL
Ankara node   → balance is 100 TL ✓ → approve → balance: 0 TL

Network recovers, nodes try to sync...
  Actual balance: -100 TL 💀
  200 TL spent, only 100 TL existed.
```

**With CP:**
```
Network partition detected
→ Nodes cannot confirm each other
→ Block both transactions
→ Return: "Transaction cannot be processed right now"
→ User is frustrated but money is safe ✓
```

**Key insight:**
```
In a payment system:
  Wrong data   → catastrophic (money lost, legal liability)
  No response  → acceptable (user retries, nothing is lost)

Therefore: stop the transaction if unsure. Never approve with stale data.
```

**Normal vs Partition:**
```
Normal conditions  → both C and A are provided
During partition   → A is sacrificed, C is protected

Availability is only given up when absolutely necessary.
```

**Databases that choose CP:**
```
PostgreSQL, MySQL, CockroachDB, Google Spanner
```

---

## AP — Availability + Partition Tolerance

When network partition occurs, **sacrifice consistency to stay available.**

### Social Media Like — AP

```
User liked a post.

t=0ms   → Istanbul node: 1,000 likes ✓ (updated immediately)
t=0ms   → London node:     999 likes  (not yet notified)
t=0ms   → Tokyo node:      999 likes  (not yet notified)

t=200ms → London node: 1,000 likes ✓ (synced)
t=350ms → Tokyo node:  1,000 likes ✓ (synced)

Result: All nodes converge to 1,000 ✓
```

London user saw wrong data for 200ms — **but nobody noticed or cared.**

**Key insight:**
```
In a like system:
  Wrong data   → acceptable (off by 1 like for 200ms)
  No response  → catastrophic (app feels broken, user churns)

Therefore: always respond, sync in the background.
```

**Databases that choose AP:**
```
Cassandra, DynamoDB, CouchDB, Redis (with replication)
```

---

## Comparison

| | Payment System | Social Media Like |
|---|---|---|
| CAP Choice | CP | AP |
| Priority | Consistency | Availability |
| On failure | Stop transaction | Show stale data |
| Wrong data | Catastrophic | Tolerable |
| No response | Acceptable | Catastrophic |
| Database | PostgreSQL, MySQL | Cassandra, DynamoDB |

---

## Strong Consistency

Every read returns the most recently written value. All nodes agree before responding.

```
Strong Consistency:
  Write → wait for ALL nodes to confirm → then respond
  → Slow but always correct

Timeline:
  t=0ms  → Write to node A
  t=50ms → Sync to node B
  t=90ms → Sync to node C
  t=90ms → Respond to client (after all nodes confirmed)
```

**Where Strong Consistency is required:**
```
✗ Cannot tolerate stale data:
  - Bank balance
  - Payment approval
  - Flight seat reservation (two people cannot book the same seat)
  - "Last 1 item in stock" scenarios
  - Inventory deduction on purchase
```

---

## Eventual Consistency

"May be inconsistent right now, but **eventually** all nodes will converge to the same value."

```
Eventual Consistency:
  Write → respond immediately → sync nodes in background
  → Fast but briefly inconsistent
```

**How it works technically:**
```
1. Write → nearest node (fast)
2. Gossip Protocol → nodes tell each other "I learned this"
3. Conflict resolution → if conflict exists, resolve it:
   → Last Write Wins: most recent write wins
   → Vector Clock: tracks which write is more recent
```

**Where Eventual Consistency is acceptable:**
```
✓ Stale data is tolerable:
  - Like / comment counts
  - Follower counts
  - Product view counts
  - Search index updates
  - Approximate stock display ("In stock" vs exact count)
  - Social feed ordering
  - Notification delivery
```

**Real world examples:**
```
DNS propagation    → updated record takes minutes/hours to reach all servers
                     but eventually everyone sees the new IP

Amazon cart        → two devices add items simultaneously
                     briefly out of sync, eventually merged

Twitter like count → may show 999 or 1001 briefly
                     nobody notices or cares
```

---

## Strong vs Eventual Consistency

| | Strong Consistency | Eventual Consistency |
|---|---|---|
| Correctness | Always correct | Briefly incorrect |
| Speed | Slower | Faster |
| Availability | Lower | Higher |
| Use case | Payments, bookings | Likes, counts, feeds |
| CAP pair | CP | AP |

> Strong Consistency = everyone sees the truth at the same time → slow
> Eventual Consistency = everyone sees the truth eventually → fast

---

## ACID

ACID is the set of properties that guarantee reliable database transactions. Pairs naturally with CP + Strong Consistency.

```
A → Atomicity   : transaction is all-or-nothing
C → Consistency : data always moves from one valid state to another
I → Isolation   : concurrent transactions don't interfere with each other
D → Durability  : committed transaction is never lost, even on crash
```

### Atomicity — All or Nothing

```
Transfer 100 TL from Account A to Account B:
  Step 1: Deduct 100 TL from A
  Step 2: Add 100 TL to B

Without Atomicity:
  Step 1 succeeds, Step 2 fails (server crash)
  → 100 TL vanished from A, never arrived at B 💀

With Atomicity:
  If Step 2 fails → Step 1 is rolled back
  → Both steps happen, or neither does ✓
```

### Consistency — Valid State to Valid State

```
Rule: balance cannot go below 0

User has 100 TL, tries to spend 150 TL
→ Transaction violates the rule
→ DB rejects it, stays in valid state ✓
```

### Isolation — Concurrent Transactions Don't Interfere

```
User balance: 100 TL
Transaction A: withdraw 100 TL  (in progress)
Transaction B: withdraw 100 TL  (starts simultaneously)

Without Isolation:
  Both read 100 TL → both approve → balance: -100 TL 💀

With Isolation:
  B waits for A to complete
  A completes → balance: 0 TL
  B reads 0 TL → rejects ✓
```

### Durability — Committed = Permanent

```
Payment approved at 14:32:05
Server crashes at 14:32:06

Without Durability:
  Transaction lost on crash 💀

With Durability:
  Written to disk before confirming
  On restart → transaction is still there ✓
```

### ACID + CAP Together

```
Payment system:
  CAP  → CP  (consistency over availability)
  ACID → all four properties enforced
  DB   → PostgreSQL / MySQL

Like system:
  CAP  → AP  (availability over consistency)
  ACID → relaxed (BASE: Basically Available, Soft state, Eventual consistency)
  DB   → Cassandra / DynamoDB / Redis
```

---

## Decision Framework

When designing a system, ask:

```
1. What happens if a user sees stale data?
   → Catastrophic (money, seats, inventory) → CP + Strong Consistency + ACID
   → Tolerable (likes, counts, feeds)       → AP + Eventual Consistency

2. What happens if the system doesn't respond?
   → Catastrophic (social app, search)  → AP
   → Acceptable (payment, booking)      → CP

3. Do multiple operations need to be atomic?
   → Yes (transfer, purchase) → ACID transactions
   → No (like, view count)    → simple writes, no transaction needed
```

---

*Topics covered: CAP Theorem, CP vs AP, Strong Consistency, Eventual Consistency, ACID (Atomicity, Consistency, Isolation, Durability), real-world examples for payment systems and social media*

---

## BASE Principle

The philosophy adopted by systems that choose AP in CAP. The counterpart of ACID.

```
B → Basically Available   : system always responds, even during failures
A → Soft State            : data can change over time, may not be in sync
S → Eventually Consistent : all nodes eventually converge to the same value
```

---

### ACID vs BASE

```
ACID → "Be absolutely correct, even if it means being slow"
BASE → "Always respond, even if the answer is slightly stale"
```

| | ACID | BASE |
|---|---|---|
| Consistency | Strong | Eventual |
| Availability | Lower | Higher |
| Approach | Pessimistic | Optimistic |
| Use case | Payments, bookings | Social, analytics |

---

### Basically Available

System always responds — but the response may not always be the most up-to-date.

```
Node A crashed
→ ACID system: "Cannot respond without Node A, wait" ✗
→ BASE system: "Node A is gone, return stale data from Node B" ✓

User sees 999 likes, real count is 1,000
→ Acceptable
```

---

### Soft State

Data can change over time — even without external input.

```
Node B currently knows 999 likes
No new likes came in, but
Node A sent a sync → now 1,000

Data updated without external change
→ State is not fixed, it is "soft"
```

---

### Eventually Consistent

The most important part — already covered in detail above:

```
May be inconsistent right now
But eventually all nodes converge to the same value
```

---

### Real World Examples

```
DNS          → update a record, old IP may be returned for hours
               but eventually everyone sees the new IP

Shopping cart → add item from two devices simultaneously
               briefly out of sync, eventually merged

Facebook feed → different data centers may show slightly different content
               eventually synced
```

---

### When is BASE sufficient?

```
✓ BASE is sufficient:
  - Like, comment, follower counts
  - Feed ordering
  - Search results
  - Analytics data
  - Notifications

✗ BASE is not sufficient:
  - Money transfer
  - Ticket reservation
  - Inventory deduction on purchase
  - "Last 1 item in stock" scenarios
```

---

### Summary

```
ACID → Pessimist: "Something might go wrong, take precautions"
BASE → Optimist:  "Things will probably be fine, keep going"
```

> BASE is not a weakness — it is a **conscious trade-off.**
> Used in the right place, it makes your system both fast and scalable.
