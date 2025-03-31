# Indexing Strategies for Large Datasets in MCP

This document explains the indexing strategies implemented in the MCP for handling large datasets (50+ lakh rows) efficiently.

## Issues Fixed

The following issues from the test script have been addressed:

1. **Column Casing Issue**: Fixed case sensitivity in queries to match the actual database schema (e.g., `CustomerID` instead of `customer_id`)
2. **PostgreSQL Function Error**: Removed dependency on the `obj_description` function which was causing the error: `function pg_catalog.obj_description(oid, smallint) does not exist`
3. **Missing Columns**: Updated the test script to use column names that actually exist in the database

## Indexing Strategies

### Why Use Indexes?
Indexes speed up data retrieval without scanning the entire table, which is critical when working with large datasets (50+ lakh rows).

### Types of Indexes

#### 1. Single-column index
- Creates an index on one column
- Best for columns frequently used in WHERE clauses
- Example:
  ```sql
  CREATE INDEX idx_name ON Customers ("LastName");
  ```

#### 2. Composite index
- Creates an index on multiple columns
- Best for queries that filter on multiple columns together
- Example:
  ```sql
  CREATE INDEX idx_name_email ON Customers ("LastName", "Email");
  ```

#### 3. Unique index
- Ensures values in the indexed columns are unique
- Also speeds up lookups
- Example:
  ```sql
  CREATE UNIQUE INDEX idx_unique_email ON Customers ("Email");
  ```

#### 4. Full-text index
- Optimizes searching through text data
- Example:
  ```sql
  CREATE INDEX idx_description_fulltext ON Products USING gin (to_tsvector('english', "Description"));
  ```

### When to Use Indexes

- **Column characteristics**:
  - Columns in `WHERE`, `JOIN`, or `ORDER BY` clauses
  - Primary keys and foreign keys
  - Columns with high cardinality (many unique values)

- **Query patterns**:
  - Frequently executed queries
  - Range queries on large datasets
  - Sorting operations on large result sets

### When to Avoid Indexes

- Small tables with few rows (full table scan may be faster)
- Columns with low cardinality (few distinct values)
- Tables with frequent insert/update operations
- Columns rarely used in WHERE clauses

## Implementation in MCP

The following index management features have been added to MCP:

1. **Create Index** (`createIndex`):
   - Create indexes on any table and columns
   - Support for unique indexes and different index types (btree, hash, gin, etc.)

2. **List Indexes** (`listIndexes`):
   - View all indexes in the database or for a specific table
   - Includes index type, column names, and uniqueness constraint

3. **Drop Index** (`dropIndex`):
   - Remove indexes that are no longer needed

4. **Recommend Indexes** (`recommendIndexes`):
   - Intelligent index recommendations based on table structure
   - Provides SQL commands to create suggested indexes
   - Includes explanations of why each index is recommended

## Performance Impact

Adding appropriate indexes can dramatically improve query performance:

1. **Without Index**:
   - Full table scan required
   - Linear performance degradation with table size
   - O(n) time complexity where n is the number of rows

2. **With Index**:
   - Logarithmic access time
   - O(log n) time complexity
   - Performance remains good even as data grows

## Example Usage in Python Client

```python
# Create an index
client.create_index(
    table_name="customer_churn_data",
    columns=["CustomerID"],
    unique=True
)

# List all indexes
indexes = client.list_indexes()

# Get index recommendations
recommendations = client.recommend_indexes("customer_churn_data")

# Drop an index
client.drop_index("idx_customer_churn_data_CustomerID")
```

## Best Practices

1. **Monitor index usage**:
   - Regularly check which indexes are being used
   - Drop unused indexes to save storage and improve write performance

2. **Balance read/write performance**:
   - More indexes improve read performance but slow down writes
   - Prioritize based on your application's read/write ratio

3. **Index column order matters**:
   - For composite indexes, place high-selectivity columns first
   - Match your index order with common query patterns

4. **Consider partial indexes** for very large tables:
   - Index only the subset of data that's frequently queried
   - Example: only index active users rather than all users

5. **Use covering indexes** to avoid table lookups:
   - Include all columns needed by the query in the index
   - Allows index-only scans for maximum performance
