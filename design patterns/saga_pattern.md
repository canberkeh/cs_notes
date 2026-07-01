# Saga Pattern

The **Saga pattern** is a way to manage a business process that spans multiple services or databases without using one big distributed transaction.
It is commonly used in **microservices**.

## The Problem

Imagine an ecommerce checkout flow:

1. Create an order.
2. Reserve inventory.
3. Charge the customer.
4. Arrange shipment.

In a monolithic app with one database, this could be one transaction:

```sql
BEGIN;
CREATE ORDER;
RESERVE INVENTORY;
CHARGE PAYMENT;
COMMIT;
```

But in microservices, each service often owns its own database:

```text
Order Service      -> orders database
Inventory Service  -> inventory database
Payment Service    -> payments database
Shipping Service   -> shipping database
```

There is no single database transaction across all of them.

So the question becomes:

> What happens if step 1 and step 2 succeed, but step 3 fails?

The Saga pattern helps solve this.

## Core Idea

A saga breaks one large business transaction into a sequence of smaller local transactions.
Each service performs its own local transaction. If something fails, the saga runs **compensating actions** to undo the previous successful steps.

Example:

```text
1. Create order
2. Reserve inventory
3. Charge payment
4. Ship order
```

If payment fails, the system may run compensation:

```text
1. Cancel order
2. Release reserved inventory
```

## Important Term: Compensation

A **compensating transaction** is an action that logically reverses a previous action.

Examples:

| Original action | Compensation |
|---|---|
| Create order | Cancel order |
| Reserve inventory | Release inventory |
| Charge payment | Refund payment |
| Book hotel | Cancel hotel booking |
| Send confirmation | Send cancellation notice |

Compensation does not always mean deleting data. Usually it means changing the state.

For example:

```text
Order status: CREATED -> CANCELLED
```

This is safer than deleting the order row.

## Coordination Styles: Choreography vs. Orchestration

A saga needs *something* to decide when a step runs and when compensation should kick in. There are two common approaches.

### Choreography

There is no central coordinator. Each service listens for events from other services, does its own work, and publishes a new event.

```text
Order Service      -> "OrderCreated" event
Inventory Service  -> "StockReserved" event
Payment Service     -> "PaymentProcessed" event
Shipping Service    -> "OrderShipped" event
```

If a step fails, that service publishes a failure event (e.g. `PaymentFailed`), and any service that needs to compensate listens for it and reacts on its own.

**Pros:** simple to start with, loosely coupled, no extra coordinating component.
**Cons:** as the flow grows, it becomes hard to see the whole process anywhere — the logic is scattered across services. Risk of circular event dependencies.

### Orchestration

A central **orchestrator** service tells each participant what to do and handles failures/compensation itself.

```text
              ┌───────────────────┐
              │  Saga Orchestrator │
              └───────────────────┘
               │        │        │
           Order    Inventory  Payment
           Service    Service   Service
```

**Pros:** the flow lives in one place, easier to understand, test, and monitor. Compensation logic is centralized.
**Cons:** the orchestrator is an extra component to build and can become a single point of failure or a "god service" if not kept thin.

There is no universally "correct" choice — choreography suits small, stable flows; orchestration tends to scale better for complex, multi-step business processes.

## Example Saga Flow

Successful flow:

```text
Order Service:    create order
Inventory Service: reserve items
Payment Service:   charge customer
Shipping Service:  create shipment
Order Service:     mark order as confirmed
```

Failure flow (payment fails after inventory was reserved):

```text
Order Service:      create order
Inventory Service:  reserve items
Payment Service:    charge customer  -> FAILS

-- compensation runs in reverse order --

Inventory Service:  release reserved items
Order Service:       mark order as cancelled
```

Note that compensation typically runs in the **reverse order** of the steps that already succeeded — you undo the most recent thing first.

## Idempotency

In a distributed system, messages and requests can be retried or delivered more than once — a network timeout doesn't always mean the original call failed, so services often retry "just in case."

This means every step in a saga (and every compensation) needs to be **idempotent**: running it twice must produce the same result as running it once.

```text
Charge payment $50   -> should not become two $50 charges if retried
Release inventory    -> should not add stock twice if the compensation event arrives twice
```

A common way to achieve this is attaching a unique idempotency key (e.g. `sagaId + stepName`) to each operation, and having the receiving service check "have I already done this?" before acting.

## Eventual Consistency

Unlike a single ACID transaction, a saga does **not** give you immediate, atomic consistency across services. While the saga is running — and especially while compensation is running after a failure — the system is in a **temporarily inconsistent state**. For example, an order might briefly show as `CREATED` while inventory hasn't been reserved yet.

The saga guarantees the system will *eventually* reach a consistent state (either "all steps succeeded" or "all steps were compensated") — but not that it's consistent at every instant in between. This trade-off is usually acceptable for business processes like orders or bookings, but it needs to be a conscious design decision, not an afterthought.

## Practical Considerations

- **Observability matters.** Since a saga's steps are spread across services, you need logging/tracing (e.g. a `sagaId` attached to every event or log line) to answer "where is this saga stuck?" without it, debugging failures is painful.
- **Compensation isn't always a clean rollback.** Once money has moved or an email has been sent, "undoing" it means issuing a refund or a cancellation notice — the inverse action, not literally erasing what happened.
- **Not every step needs compensation.** Read-only or side-effect-free steps can simply be skipped when building the saga.

## When to Use It

- A business process spans multiple services/databases.
- A two-phase-commit-style distributed transaction isn't practical (too slow, doesn't scale, or services don't support it).
- Eventual consistency is acceptable for the use case (e.g. orders, bookings, reservations).
