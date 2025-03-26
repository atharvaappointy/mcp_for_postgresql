# MCP Server - Enhanced for Large Datasets

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

### 4. Natural Language Processing
- Process queries written in plain English
- Support for conversational CRUD operations
- Automatic intent detection and parameter extraction

### 5. Advanced Search Capabilities  
- Smart column-based search with optimized query execution
- Multi-column search with dynamic filtering
- ID-based record retrieval with primary key optimization

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

### CRUD Operations

#### Create (Insert Data)
```
POST /query
Content-Type: application/json

{
  "type": "query",
  "query": "INSERT INTO customer_churn_data (CustomerID, Age, Gender) VALUES ($1, $2, $3)",
  "params": [123456, 35, "Female"]
}
```
- **Description**: Creates new records in the database
- **Parameters**:
  - `query` (required): SQL INSERT statement
  - `params` (optional): Array of parameter values for prepared statements
- **Response**: Status information about the operation

#### Read (Query Data)
```
POST /query
Content-Type: application/json

{
  "type": "filteredQuery",
  "tableName": "customer_churn_data",
  "columns": ["CustomerID", "Age", "Gender"],
  "filters": {
    "Age": {"operator": ">", "value": 30}
  },
  "page": 1,
  "pageSize": 100
}
```
- **Description**: Retrieves data based on specified filters and columns
- **Response**: Array of matching records with pagination metadata

#### Update (Modify Data)
```
POST /query
Content-Type: application/json

{
  "type": "query",
  "query": "UPDATE customer_churn_data SET Age = $1, Gender = $2 WHERE CustomerID = $3",
  "params": [36, "Male", 123456]
}
```
- **Description**: Updates existing records in the database
- **Parameters**:
  - `query` (required): SQL UPDATE statement
  - `params` (optional): Array of parameter values for prepared statements
- **Response**: Status information about the operation

#### Delete (Remove Data)
```
POST /query
Content-Type: application/json

{
  "type": "query",
  "query": "DELETE FROM customer_churn_data WHERE CustomerID = $1",
  "params": [123456]
}
```
- **Description**: Removes records from the database
- **Parameters**:
  - `query` (required): SQL DELETE statement
  - `params` (optional): Array of parameter values for prepared statements
- **Response**: Status information about the operation

### Batch Operations

#### Batch Insert
```
POST /query
Content-Type: application/json

{
  "type": "query",
  "query": "INSERT INTO customer_churn_data (CustomerID, Age, Gender) VALUES ($1, $2, $3), ($4, $5, $6), ($7, $8, $9)",
  "params": [123456, 35, "Female", 123457, 42, "Male", 123458, 28, "Female"]
}
```
- **Description**: Inserts multiple records in a single operation
- **Response**: Status information about the operation

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

## Advanced Search API

### Find By ID
```
POST /search/id
Content-Type: application/json

{
  "tableName": "customer_churn_data",
  "idColumn": "CustomerID",
  "idValue": 123456
}
```
- **Description**: Retrieves a single record by its ID using optimized primary key search
- **Parameters**:
  - `tableName` (required): The name of the table
  - `idColumn` (required): The column containing the ID
  - `idValue` (required): The ID value to search for
- **Response**: The matching record or an error

### Smart Column Search
```
POST /search/column
Content-Type: application/json

{
  "tableName": "customer_churn_data",
  "columnName": "Age",
  "searchValue": 30,
  "options": {
    "operator": ">",
    "fuzzyMatch": false,
    "caseSensitive": false,
    "page": 1,
    "pageSize": 100
  }
}
```
- **Description**: Performs an intelligent search on a specific column
- **Parameters**:
  - `tableName` (required): The name of the table
  - `columnName` (required): The column to search on
  - `searchValue` (required): The value to search for
  - `options` (optional): Additional search options
    - `operator`: Comparison operator (=, >, <, LIKE, etc.)
    - `fuzzyMatch`: Whether to use fuzzy matching for text
    - `caseSensitive`: Whether the search should be case-sensitive
    - `page`: Page number for pagination
    - `pageSize`: Page size for pagination
- **Response**: Matching records with metadata

### Multi-Column Search
```
POST /search/multi
Content-Type: application/json

{
  "tableName": "customer_churn_data",
  "columnValues": {
    "Gender": "Female",
    "Age": 30
  },
  "columnOptions": {
    "Age": {
      "operator": ">",
      "fuzzyMatch": false
    }
  },
  "page": 1,
  "pageSize": 100
}
```
- **Description**: Searches for records matching multiple column criteria simultaneously
- **Parameters**:
  - `tableName` (required): The name of the table
  - `columnValues` (required): Dictionary mapping column names to search values
  - `columnOptions` (optional): Dictionary of column-specific search options
  - `page` (optional): Page number for pagination
  - `pageSize` (optional): Page size for pagination
- **Response**: Matching records with metadata

## Natural Language Processing API

### Natural Language Query
```
POST /nlp
Content-Type: application/json

{
  "query": "Find all customers in New York with age greater than 30"
}
```
- **Description**: Executes a database operation using natural language
- **Parameters**:
  - `query` (required): Natural language query
- **Response**: The operation results

### Supported NLP Operations

#### Search Operations
- "Find all customers in California"
- "Show me users with age greater than 30"
- "Get products where price is less than 100"
- "List customers sorted by name"
- "Count records where status is active"

#### Create Operations
- "Add a new customer with name John Smith and age 35"
- "Create user with email john@example.com and password 123456"
- "Insert product with name 'Laptop' and price 999.99"

#### Update Operations
- "Update customer 123 set status to inactive"
- "Change the email of user 456 to new@example.com"
- "Set the price of product 'Laptop' to 899.99"

#### Delete Operations
- "Delete user with email john@example.com"
- "Remove customer with id 123"
- "Delete all products where price is less than 10"

## Interactive Mode

The MCP server includes an interactive command-line interface that allows you to explore and manipulate your database using both structured commands and natural language queries.

### Starting Interactive Mode
```
node app.js interactive
```

### Interactive Commands

#### Natural Language Queries
Just type your question in plain English, for example:
- "Find all customers in California"
- "Show me users with age greater than 30"
- "Update customer 123 set status to inactive"

#### Data Exploration
- `list tables` - List all tables
- `list columns <table>` - List columns for a table
- `list sample <table> [count]` - Show sample data from a table
- `list stats <table>` - Show statistics for a table

#### Searching
- `search id <table> <id_col> <id>` - Find record by ID
- `search column <table> <col> <val> [operator] [fuzzy=t/f]` - Search on a specific column
- `search multi <table> <col1>=<val1> <col2>=<val2> ...` - Search with multiple criteria

#### Index Management
- `index create <table> <col1> [col2...] [unique=t/f]` - Create an index
- `index list [table]` - List indexes
- `index drop <index_name>` - Drop an index
- `index recommend <table>` - Get index recommendations

## Troubleshooting Common Issues

### Case Sensitivity

PostgreSQL table and column names are case-sensitive when quoted and case-insensitive when not quoted. The MCP API handles this automatically for you, but be aware of the following:

1. When creating tables:
   ```sql
   -- Creates a table with lowercase column names
   CREATE TABLE users (id SERIAL, firstname TEXT, lastname TEXT);
   
   -- Creates a table with exact case column names
   CREATE TABLE "Users" ("Id" SERIAL, "FirstName" TEXT, "LastName" TEXT);
   ```

2. When querying through MCP:
   - For filteredQuery: The API will automatically match column names case-insensitively
   - For direct SQL: You need to match case or use double quotes

### LIMIT Clauses in Paginated Queries

When using the paginatedQuery endpoint, do NOT include LIMIT or OFFSET in your SQL query. The API will add these automatically based on the page and pageSize parameters.

**Incorrect:**
```json
{
  "type": "paginatedQuery",
  "query": "SELECT * FROM customer_churn_data LIMIT 10",
  "page": 1,
  "pageSize": 100
}
```

**Correct:**
```json
{
  "type": "paginatedQuery",
  "query": "SELECT * FROM customer_churn_data",
  "page": 1,
  "pageSize": 100
}
```

### Required Parameters

Make sure to include all required parameters for each endpoint. The most common issues are:

1. Missing `tableName` parameter for table-specific operations
2. Missing `query` parameter for SQL execution
3. Missing `indexName` parameter when dropping an index

### Error Response Format

All errors follow a consistent format to help with debugging:

```json
{
  "type": "endpointType",
  "data": null,
  "error": "Detailed error message"
}
```

Check the error message for specific information about what went wrong.


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

```

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

```

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

```
