# SQL vs NoSQL Databases

SQL and NoSQL are two broad families of databases. The main difference is how they model data and how strictly they enforce structure.

## SQL Databases

SQL databases are relational databases. They store data in tables, with rows and columns.

Example `users` table:

| id | name | email |
|---:|---|---|
| 1 | Maya | maya@example.com |
| 2 | Leo | leo@example.com |

Example `orders` table:

| id | user_id | total |
|---:|---:|---:|
| 101 | 1 | 49.99 |
| 102 | 2 | 19.99 |

The relationship is clear: `orders.user_id` points to `users.id`.

Common SQL databases include:

- PostgreSQL
- MySQL
- SQLite
- SQL Server
- Oracle Database

SQL databases are best when your data has a clear structure and relationships matter.

Good uses for SQL:

- Banking systems
- Inventory systems
- Ecommerce orders
- Accounting
- User accounts and permissions
- Systems needing strong consistency

Example SQL query:

```sql
SELECT users.name, orders.total
FROM users
JOIN orders ON users.id = orders.user_id;
```

That query says: "Give me users and their orders by matching related rows."

## NoSQL Databases

NoSQL databases are non-relational. They do not always use tables, rows, and joins. They are often designed for flexibility, scale, or speed with certain kinds of data.

Common NoSQL types:

- Document databases: store data as JSON-like documents (MongoDB, CouchDB)
- Key-value stores: store data as simple key → value pairs (Redis, DynamoDB)
- Wide-column databases: store data in rows with flexible columns (Cassandra, HBase)
- Graph databases: store data as nodes and relationships (Neo4j)

A document database might store a user like this:

```json
{
  "id": 1,
  "name": "Maya",
  "email": "maya@example.com",
  "orders": [
    { "id": 101, "total": 49.99 },
    { "id": 103, "total": 12.50 }
  ]
}
```

Instead of splitting users and orders into separate tables, the related data can be nested together in one document.

NoSQL databases are best when the shape of your data changes often, or when you need to handle very large amounts of data with high read/write throughput.

Good uses for NoSQL:

- Real-time analytics
- Chat apps
- Logs and events
- Content feeds
- Caching
- Large distributed systems
- Flexible JSON-like data

## Core Difference

SQL databases usually prioritize structure, relationships, and consistency.

NoSQL databases usually prioritize flexibility, scalability, and speed for specific access patterns.

| Question | SQL | NoSQL |
|---|---|---|
| Data structure | Tables | Documents, keys, graphs, columns |
| Schema | Usually strict | Often flexible |
| Relationships | Strong support with joins | Often embedded or manually linked |
| Query language | SQL | Varies by database |
| Best for | Structured, relational data | Flexible or massive-scale data |
| Consistency | Usually strong | Varies; sometimes eventual |
| Scaling | Often vertical or read replicas | Often horizontal/distributed |

## Simple Analogy

SQL is like a well-organized spreadsheet system with rules.

NoSQL is like a set of flexible file cabinets where each file can have a different shape depending on what you need.

## When to Use SQL

Use SQL when:

- Your data relationships are important.
- You need transactions, accuracy, and consistency.
- Your data model is fairly predictable.
- You need complex queries and reporting.

Example: a banking app should probably use SQL because correctness matters deeply.

## When to Use NoSQL

Use NoSQL when:

- Your data shape changes frequently.
- You need huge scale across many servers.
- You mostly retrieve data in a few predictable ways.
- You are storing nested, document-like data.

Example: a social media feed or event logging system may work well with NoSQL.

## A Common Misconception

It is not "SQL is old, NoSQL is new."

Modern SQL databases like PostgreSQL are extremely powerful and can even store JSON. Modern NoSQL databases can also support transactions and indexes. The boundary is blurrier than people make it sound.

A good default choice for many apps is:

**Start with PostgreSQL unless you have a clear reason not to.**

It handles structured and unstructured data, supports JSON columns, and
gives you transactions and strong consistency out of the box.
