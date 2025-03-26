# MCP Server 

This enhanced version of the Microservice Control Panel (MCP) has been optimized to handle large datasets (50+ lakh rows) with efficient data extraction capabilities and indexing strategies.

## Core Enhancements

### 1. Pagination Support
- Efficiently handle large datasets by retrieving data in manageable chunks
- Avoid memory issues when working with millions of rows

### 2. Dynamic Data Extraction
- Query specific tables, columns, and rows based on user input
- No hardcoded table names - completely flexible query system

### 3. Advanced Indexing Strategies
- Create, manage, and recommend database indexes
- Dramatically improve query performance for large datasets

## Indexing Strategies

### Why Use Indexes?
Indexes speed up data retrieval without scanning the entire table, which is critical for tables with millions of rows.

### Types of Indexes Supported:
- **Single-column index**: Creates an index on one column
- **Composite index**: Creates an index on multiple columns for complex queries
- **Unique index**: Ensures values in the indexed columns are unique
- **Full-text index**: Optimizes text searches within large text fields

### When to Use:
- Frequently searched columns
- Columns used in `WHERE`, `JOIN`, or `ORDER BY` clauses
- Primary keys and foreign keys

### When to Avoid:
- Small tables (indexes have overhead)
- Columns with low cardinality (few distinct values)
- Tables with frequent write operations

### Example:
```sql
-- Create a simple index on CustomerID
CREATE INDEX idx_customer_id ON Customers ("CustomerID");

-- Create a composite index for filtering by multiple columns
CREATE INDEX idx_gender_age ON customer_churn_data ("Gender", "Age");
```

## API Endpoints

### Database Inspection

#### Get All Tables
```
POST /query
Content-Type: application/json

{
  "type": "getTables"
}
```
- **Description**: Retrieves all tables in the database
- **Response**: Object with array of table names, estimated row counts, and sizes

#### Get Table Columns
```
POST /query
Content-Type: application/json

{
  "type": "getTableColumns",
  "tableName": "customer_churn_data"
}
```
- **Description**: Retrieves all columns for a specific table
- **Response**: Array of column information including name, data type, and constraints

#### Get Table Statistics
```
POST /query
Content-Type: application/json

{
  "type": "getTableStats",
  "tableName": "customer_churn_data"
}
```
- **Description**: Retrieves statistics for a specific table
- **Response**: Object with table name, row count estimation, total size, and column information

#### Get Table Sample
```
POST /query
Content-Type: application/json

{
  "type": "getTableSample",
  "tableName": "customer_churn_data",
  "limit": 5
}
```
- **Description**: Retrieves a sample of rows from a specific table
- **Parameters**:
  - `tableName` (required): The name of the table
  - `limit` (optional): Number of rows to return (default: 5)
- **Response**: Array of row objects

### Query Execution

#### Execute Basic Query
```
POST /query
Content-Type: application/json

{
  "type": "query",
  "query": "SELECT * FROM customer_churn_data LIMIT 10",
  "params": []
}
```
- **Description**: Executes a custom SQL query
- **Parameters**:
  - `query` (required): SQL query string
  - `params` (optional): Array of parameter values for prepared statements
- **Response**: Array of query results

#### Execute Paginated Query
```
POST /query
Content-Type: application/json

{
  "type": "paginatedQuery",
  "query": "SELECT * FROM customer_churn_data",
  "page": 1,
  "pageSize": 100
}
```
- **Description**: Executes a SQL query with pagination for large datasets
- **Parameters**:
  - `query` (required): SQL query string (without LIMIT/OFFSET)
  - `page` (optional): Page number (default: 1)
  - `pageSize` (optional): Number of rows per page (default: 100)
- **Response**: Object containing:
  - `data`: Array of results
  - `pagination`: Pagination metadata (total rows, page, total pages, etc.)

#### Execute Filtered Query
```
POST /query
Content-Type: application/json

{
  "type": "filteredQuery",
  "tableName": "customer_churn_data",
  "columns": ["CustomerID", "Age", "Gender"],
  "filters": {
    "Age": {"operator": ">", "value": 30},
    "Gender": {"operator": "=", "value": "Female"}
  },
  "page": 1,
  "pageSize": 100
}
```
- **Description**: Executes a query with column and filter selection
- **Parameters**:
  - `tableName` (required): The table to query
  - `columns` (optional): Array of column names to retrieve (default: all columns)
  - `filters` (optional): Object with column filters, where each filter has an operator and value
  - `page` (optional): Page number (default: 1)
  - `pageSize` (optional): Number of rows per page (default: 100)
- **Response**: Object containing data array and pagination metadata

### Advanced Data Access (For Large Datasets)

#### Get Specific Row by Number
```
GET /data/row/:tableName/:rowNumber
```
- **Description**: Retrieves a specific row by its row number using efficient direct access
- **URL Parameters**:
  - `tableName`: The name of the table
  - `rowNumber`: The row number to retrieve (1-based index)
- **Response**: Object containing the specific row data

#### Binary Search on Column
```
POST /data/search
Content-Type: application/json

{
  "tableName": "customer_churn_data",
  "column": "Age",
  "value": 30,
  "operator": ">"
}
```
- **Description**: Performs an efficient binary search on a column for a specific value
- **Parameters**:
  - `tableName` (required): The table to search
  - `column` (required): Column name to search on
  - `value` (required): Value to search for
  - `operator` (optional): Comparison operator (=, >, <, >=, <=, LIKE) (default: =)
- **Response**: Array of matching rows

### Index Management

#### Create Index
```
POST /index/create
Content-Type: application/json

{
  "tableName": "customer_churn_data",
  "columns": ["Age", "Gender"],
  "options": {
    "unique": false,
    "type": "btree",
    "name": "idx_age_gender"
  }
}
```
- **Description**: Creates a new index on the specified columns
- **Parameters**:
  - `tableName` (required): The table to create an index on
  - `columns` (required): Array of column names to index
  - `options` (optional): Object with index options:
    - `unique` (boolean): Whether the index enforces uniqueness
    - `type` (string): Index type (btree, hash, gin, etc.)
    - `name` (string): Custom name for the index

#### List Indexes
```
GET /index/list?tableName=customer_churn_data
```
- **Description**: Lists all indexes for a table, or all indexes in the database if no table is specified
- **Query Parameters**:
  - `tableName` (optional): Filter indexes by table name
- **Response**: Array of index information

#### Drop Index
```
POST /index/drop
Content-Type: application/json

{
  "indexName": "idx_age_gender"
}
```
- **Description**: Drops an existing index
- **Parameters**:
  - `indexName` (required): The name of the index to drop

#### Get Index Recommendations
```
POST /query
Content-Type: application/json

{
  "type": "recommendIndexes",
  "tableName": "customer_churn_data"
}
```
- **Description**: Recommends indexes for a table based on its structure and data distribution
- **Parameters**:
  - `tableName` (required): The table to analyze
- **Response**: Array of recommended indexes with explanations

## Client Integration Examples

### JavaScript
```javascript
// Sample JavaScript client to query MCP API
async function queryMCP() {
  const response = await fetch('http://localhost:3000/query', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      type: 'getTables'
    }),
  });
  
  const data = await response.json();
  console.log(data);
}
```

### Python
```python
# Sample Python client to query MCP API
import requests

def query_mcp():
    response = requests.post('http://localhost:3000/query', json={
        'type': 'getTables'
    })
    return response.json()

# Get a specific row by number - direct access
def get_row_by_number(table_name, row_number):
    response = requests.get(f'http://localhost:3000/data/row/{table_name}/{row_number}')
    return response.json()

# Perform a binary search
def binary_search(table_name, column, value, operator='='):
    response = requests.post('http://localhost:3000/data/search', json={
        'tableName': table_name,
        'column': column,
        'value': value,
        'operator': operator
    })
    return response.json()
```

## Using MCP with AI Applications

When integrating MCP with AI applications, the following patterns work best:

### 1. Efficient Data Retrieval for Training Datasets
```python
def fetch_training_data(table_name, filters=None, limit=10000):
    """Fetch data in chunks to avoid memory issues with large training datasets"""
    page_size = 1000
    page = 1
    all_data = []
    has_more = True
    
    while has_more and len(all_data) < limit:
        response = requests.post('http://localhost:3000/query', json={
            'type': 'filteredQuery',
            'tableName': table_name,
            'filters': filters,
            'page': page,
            'pageSize': page_size
        })
        result = response.json()
        
        all_data.extend(result['data'])
        page += 1
        has_more = result['pagination']['hasNext']
        
        if len(all_data) >= limit:
            all_data = all_data[:limit]
            break
            
    return all_data
```

### 2. AI-Driven Index Recommendations
```python
def get_smart_index_recommendations(table_name, query_patterns):
    """Get index recommendations based on AI analysis of query patterns"""
    recommendations = requests.post('http://localhost:3000/query', json={
        'type': 'recommendIndexes',
        'tableName': table_name
    }).json()
    
    # Match recommendations with query patterns
    optimal_indexes = []
    for pattern in query_patterns:
        for rec in recommendations['data']:
            if any(col in pattern['columns'] for col in rec['columns']):
                optimal_indexes.append({
                    'columns': rec['columns'],
                    'priority': pattern['frequency']
                })
    
    return sorted(optimal_indexes, key=lambda x: x['priority'], reverse=True)
```

### 3. Natural Language to SQL Query Processing
```python
def natural_language_query(query_text):
    """Example of how to integrate MCP with a natural language to SQL processor"""
    # This would normally be handled by an LLM or other NLP system
    # This is a simplified example
    
    # Detect specific row requests
    import re
    row_match = re.search(r"row (?:number\s*)?(\d+)", query_text.lower())
    
    if row_match:
        row_number = int(row_match.group(1))
        # Extract table name - simplified example
        if "customer_churn_data" in query_text:
            return get_row_by_number("customer_churn_data", row_number)
    
    # Otherwise perform a filtered query with NLP-extracted parameters
    # Code would integrate with your NLP system here
    
    # Fallback to basic query
    return requests.post('http://localhost:3000/query', json={
        'type': 'query',
        'query': "SELECT * FROM customer_churn_data LIMIT 10"
    }).json()
```

## Error Handling

All API endpoints follow a consistent error response pattern:

```json
{
  "type": "endpoint_type",
  "data": null,
  "error": "Error message describing what went wrong"
}
```

Common HTTP status codes:
- 200: Success
- 400: Bad Request (missing parameters, invalid input)
- 404: Not Found (table, index, or row not found)
- 500: Server Error (database issues, query syntax errors)

## Performance Considerations

1. For tables with millions of rows:
   - Always use pagination
   - Utilize direct row access (`/data/row/:tableName/:rowNumber`) when possible
   - Create appropriate indexes on frequently queried columns

2. For complex queries:
   - Use the `binarySearchRows` endpoint for finding specific values
   - Create composite indexes on columns used together in filters
   - Consider using filtered queries instead of raw SQL for better optimization

3. For write-heavy operations:
   - Be selective with indexes as they slow down INSERT/UPDATE operations
   - Use batch operations when possible

## Security Notes

- MCP does not include authentication/authorization by default
- For production use, implement proper security measures:
  - API authentication (JWT, OAuth, etc.)
  - Rate limiting
  - Input validation and sanitization
  - Network security (firewall rules, VPN, etc.)

## Testing

### Unit Tests
Run the test suite to verify index functionality:
```
npm test
```

This will execute all index management tests, including:
- Listing existing indexes
- Creating and dropping indexes
- Testing query performance with and without indexes
- Implementing complex indexing strategies

### Performance Testing
Run the customer churn data test to see performance improvements:
```
npm run test:performance
```

This includes:
1. Table statistics retrieval
2. Structural analysis
3. Paginated query performance
4. Filtered query performance
5. Index performance comparison

## Getting Started

1. Start the server:
```
npm start
```

2. Use the Python test client to interact with the MCP:
```python
from app import MCPClient

client = MCPClient()
tables = client.get_all_tables()
