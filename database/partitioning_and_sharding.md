# Partitioning vs Sharding

Partitioning and sharding are techniques for splitting data into smaller pieces so databases can handle large amounts of data more efficiently.

They sound similar, and people sometimes use the words loosely, but there is an important difference.

## Short Version

**Partitioning** means splitting one large table or dataset into smaller parts.

**Sharding** means splitting data across multiple database servers.

Simple rule:

> Partitioning is usually inside one database.  
> Sharding is usually across multiple databases or machines.

## Partitioning

Partitioning is when a large table is divided into smaller logical pieces called partitions.

For example, imagine an `orders` table with millions of rows:

```text
orders
```

You can partition it by year:

```text
orders_2024
orders_2025
orders_2026
```

The database may still let you query it as one table:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01';
```

Internally, the database can look only at the relevant partition instead of scanning the whole table.

## Common Partitioning Types

### Range Partitioning

Data is split by ranges.

Example:

```text
orders from 2024 -> partition_2024
orders from 2025 -> partition_2025
orders from 2026 -> partition_2026
```

Good for:

- Time-based data
- Logs
- Orders
- Analytics tables

### List Partitioning

Data is split by specific values.

Example:

```text
country = 'US' -> partition_us
country = 'TR' -> partition_tr
country = 'DE' -> partition_de
```

Good for:

- Region-based data
- Category-based data
- Tenant grouping

### Hash Partitioning

Data is split using a hash function applied to a column value.

Example:

```text
hash(user_id) % 4
```

The result (0, 1, 2, or 3) determines which partition the row goes to:

```text
result 0 -> partition_a
result 1 -> partition_b
result 2 -> partition_c
result 3 -> partition_d
```

This spreads rows across partitions more evenly regardless of the underlying data distribution.

Good for:

- Large tables where data should be distributed evenly
- Avoiding one partition becoming too large

### Composite Partitioning

Partitioning strategies can be combined. This is called composite or sub-partitioning.

Example: partition an `orders` table by year first (range), then hash within each year partition by `customer_id`.

```text
partition_2026 -> sub-partition by hash(customer_id) % 4
```

This is useful when a single strategy does not distribute data evenly enough, or when queries filter on multiple dimensions.

## Why Use Partitioning?

Partitioning can improve performance and manageability.

Benefits:

- Faster queries when filters match the partition key
- Easier deletion or archiving of old data
- Smaller indexes per partition
- Better maintenance for very large tables

Example:

If you often query recent orders, partitioning by `order_date` can help the database skip old partitions.

## Sharding

Sharding is a type of horizontal scaling where data is split across multiple database servers.

Instead of one huge database server holding all users, you might split users like this:

```text
Shard 1: users with id 1 - 1,000,000
Shard 2: users with id 1,000,001 - 2,000,000
Shard 3: users with id 2,000,001 - 3,000,000
```

Or with hashing:

```text
hash(user_id) % 3
```

Each shard contains only part of the data.

## What Is a Shard Key?

A shard key decides where data should live. It is chosen upfront and determines which shard receives a given row.

Example:

```text
user_id
```

If the app shards by `user_id`, all data for a user can be routed to the correct shard.

A good shard key should:

- Spread data evenly
- Match common query patterns
- Avoid hot spots
- Be stable and not change often

Choosing a poor shard key has real consequences. For example, sharding by `country` when 80% of your users are in the US means one shard handles the vast majority of traffic while the others sit mostly idle. This is called a hot spot and can negate the benefits of sharding entirely.

## Sharding Example

Imagine a social media app with 500 million users.

One database server may not be enough for all user data. So you split users across multiple databases:

```text
db_shard_1 -> users A-F
db_shard_2 -> users G-M
db_shard_3 -> users N-Z
```

When the app needs user data, it first decides which shard contains that user, then queries that database.

Note that unlike partitioning — which is often handled transparently by the database — sharding usually requires routing logic in the application or a middleware layer. The application (or a proxy) must know which shard to talk to before issuing a query.

## Why Use Sharding?

Sharding is used when one database server is no longer enough.

Benefits:

- Handles more data
- Handles more traffic
- Spreads load across multiple machines
- Allows horizontal scaling

Good for:

- Very large user bases
- High-traffic applications
- Multi-tenant SaaS platforms
- Messaging systems
- Large-scale ecommerce

## Main Difference

| Topic | Partitioning | Sharding |
|---|---|---|
| Meaning | Split data into smaller parts | Split data across servers |
| Scope | Usually inside one database | Across multiple databases or machines |
| Goal | Improve query performance and manageability | Scale beyond one database server |
| Complexity | Lower | Higher |
| Transparency to app | Usually invisible; the DB handles routing | Usually requires app-level or middleware routing |
| Example | Split `orders` by month | Split users across multiple DB servers |

## Important Detail

Sharding is often considered a form of partitioning.

More specifically:

> Sharding is horizontal partitioning across multiple machines.

So all sharding is partitioning, but not all partitioning is sharding.

## Common Problems With Sharding

Sharding can solve scale problems, but it adds complexity.

Challenges:

- **Harder joins across shards.** If two related records live on different shards, joining them requires fetching data from multiple machines and merging it in the application or middleware.
- **Harder transactions across shards.** Atomic transactions spanning multiple shards require distributed transaction protocols, which are complex and can hurt performance.
- **Hot spots.** A poorly chosen shard key can cause one shard to receive far more traffic than others. For example, sharding by `country` in an app where most users are in one country concentrates load on a single shard.
- **Rebalancing is difficult.** As data grows, you may need to add shards and redistribute data. Moving data between shards without downtime is non-trivial.
- **More complex backups and deployments.** Operating multiple database servers means more moving parts to monitor, back up, and upgrade.

## Simple Analogy

Partitioning is like organizing one large filing cabinet into labeled drawers.

Sharding is like putting different filing cabinets in different offices.

With partitioning, the data is split inside the same database system. With sharding, the data is split across multiple database systems.

## Interview Version

Partitioning means splitting a large table into smaller parts, usually inside the same database. For example, an orders table can be partitioned by month or year.

Sharding means splitting data across multiple database servers. Each server stores only part of the data. This helps when one database server cannot handle all the traffic or storage.

The key difference is that partitioning is mainly for performance and manageability inside a database, while sharding is mainly for horizontal scaling across machines. Partitioning is also typically transparent to the application, whereas sharding usually requires the application or a middleware layer to handle routing.

## When to Use Which?

Use partitioning when:

- One database can still handle the workload
- Tables are very large
- Queries often filter by date, region, or category
- You want easier archiving or maintenance

Use sharding when:

- One database server is not enough
- Traffic or data volume is extremely high
- You need horizontal scaling
- You can handle the added operational complexity

## Practical Advice

Start with normal indexing and good schema design first.

Then consider partitioning when tables become very large.

Use sharding only when you truly need to scale beyond a single database server, because sharding adds significant complexity.
