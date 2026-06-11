# HashMap: A Comprehensive Guide

A HashMap is a fundamental data structure used in computer science to store data in key-value pairs. It provides highly efficient performance for searching, inserting, and deleting data.

## Core Concepts

- **Key:** A unique identifier used to look up a specific value.
- **Value:** The actual data associated with a key (values do not have to be unique).
- **Hash Function:** An algorithm that takes a key and converts it into a numerical index. This index determines where the value is stored in memory.

## How It Works (Hashing)

```
[ Key ] ---> ( Hash Function ) ---> [ Index / Bucket ] ---> [ Value ]
```

**Insertion:** When you add a pair (e.g., `"John" → 28`), the HashMap passes the key `"John"` through its hash function. The function outputs a specific memory slot index (e.g., slot 5). The value `28` is stored there.

**Retrieval:** When you look up `"John"`, the HashMap runs the key through the exact same function, gets index 5, and instantly grabs the value `28`.

> [!NOTE]
> **Collisions:** If two different keys generate the exact same index, it is called a collision. HashMaps handle this automatically by linking the multiple values together at that index using a secondary data structure like a Linked List or a Balanced Tree.

## Key Performance Benefits

**Speed:** Searching, adding, or removing data takes `O(1)` constant time on average. This means operations take the exact same fraction of a second whether you have 10 items or 10 million items.

**Flexibility:** Unlike standard arrays that limit you to using sequential numbers (`0, 1, 2...`) as indices, a HashMap allows you to use words, IDs, or custom objects as keys.

## Common Language Equivalents

While the structure is identical, different programming languages call it by different names:

| Language   | Name                      |
|------------|---------------------------|
| Java       | `HashMap`                 |
| Python     | `dict` (Dictionary)       |
| JavaScript | `Map` or `Object {}`      |
| C++        | `std::unordered_map`      |
| C#         | `Dictionary<TKey, TValue>`|

## Basic Implementation Examples

### Python (Dictionary)

```python
# Create a dictionary
user_map = {}

# Insert data
user_map["John"] = 28
user_map["Alice"] = 32

# Retrieve data
print(user_map["John"])  # Output: 28

# Delete data
del user_map["Alice"]
```

### Java (HashMap)

```java
import java.util.HashMap;

public class Main {
    public static void main(String[] args) {
        // Create a HashMap
        HashMap<String, Integer> userMap = new HashMap<>();

        // Insert data
        userMap.put("John", 28);
        userMap.put("Alice", 32);

        // Retrieve data
        System.out.println(userMap.get("John")); // Output: 28

        // Delete data
        userMap.remove("Alice");
    }
}
```
