- [Introduction](#introduction)
- [Data intensive applications](#data-intensive-applications)
- [Storage and Retrieval](#storage-and-retrieval)
  - [Data Structures That Power Your Database](#data-structures-that-power-your-database)
  - [Indexes](#indexes)
    - [Hash Indexes](#hash-indexes)

# Introduction
This notes are based on the book [Designing Data-Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781663728289/) by Martin Kleppmann

# Data intensive applications

A data-intensive application is typically built from standard building blocks that provide commonly needed functionality. For example, many applications need to:
- Store data so that they, or another application, can find it again later (**databases**)
- Remember the result of an expensive operation, to speed up reads (**caches**)
- Allow users to search data by keyword or filter it in various ways (**search indexes**)
- Send a message to another process, to be handled asynchronously (**stream processing**)
- Periodically crunch a large amount of accumulated data (**batch processing**)

# Storage and Retrieval

At a high level, databases need to do two things: **store data** and **retrieve data**. Its important to know how databases handle this, in order to better decide the appropriate database for your application.

## Data Structures That Power Your Database

Consider the following database implemented in bash as an example:
```bash
#!/bin/bash

db_set () {
    echo "$1,$2" >> database
}

db_get () {
    grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

- **High write performance**: The `db_set` function appends a new key-value pair to the end of the file. This is very fast because appending is a simple operation.

> [!NOTE]
> Normal databases need to take more into consideration (concurrency, disk space reclaiming, handling errors, etc), but the idea is the same.

- **Low read performance**: The `db_get` function reads the entire file and then filters out the key-value pair that it needs. This is slow because it has to read the entire file, which has _O(n)_ complexity.

In order to increase the read performance we will need a data structure called **index**. But this will decrease the write performance, because we need to maintain a new data structure every time we write a new key-value pair.


## Indexes
An index is an _additional_ structure that is derived from the primary data. Many DBs allow you to modify indexes without modifying the primary data, only the performance of queries.

Maintaining indexes incurs in overhead, specially on writes. This is an important trade-off, well chosen indexes speed up read queries, but slow down writes. This is why DBs don't index everything by default.

### Hash Indexes

