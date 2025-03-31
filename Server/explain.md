# MCP (Model Context Protocol) - Internal Workings Explanation

## Table of Contents
1. [Server Configuration](#server-configuration)
2. [Dynamic Database Interaction](#dynamic-database-interaction)
3. [Query Processing Flow](#query-processing-flow)
4. [Schema Analysis](#schema-analysis)
5. [Connection Management](#connection-management)
6. [Cursor Support](#cursor-support)

## Server Configuration

### Dynamic Configuration
MCP is not hardcoded to any specific database. Instead, it uses a configuration-based approach:

```javascript
// config/config.js
module.exports = {
    dbType: 'postgres',  // Can be changed to other database types
    dbConfig: {
        user: process.env.DB_USER || 'postgres',
        host: process.env.DB_HOST || 'localhost',
        database: process.env.DB_NAME || 'appointy',
        password: process.env.DB_PASSWORD || '0221',
        port: process.env.DB_PORT || 5432,
    }
};
```

### Plugin-Based Architecture
The system uses a plugin architecture that allows different database types:
```javascript
const plugin = require(`../plugins/${config.dbType}`);
```

## Dynamic Database Interaction

### 1. Schema Discovery
MCP doesn't hardcode table structures. Instead, it:

1. **Discovers Tables Dynamically**:
```javascript
const getAllTables = async () => {
    const query = `
        SELECT 
            table_name,
            pg_class.reltuples::bigint AS estimated_rows
        FROM information_schema.tables
        JOIN pg_class ON pg_class.relname = table_name
        WHERE table_schema = 'public'
    `;
    return await executeQuery(query);
};
```

2. **Analyzes Schema on Demand**:
```javascript
const getTableSchema = async (tableName) => {
    const query = `
        SELECT column_name, data_type, 
               is_nullable, column_default
        FROM information_schema.columns
        WHERE table_schema = 'public' 
        AND table_name = $1
    `;
    return await executeQuery(query, [tableName]);
};
```

### 2. Query Building
Queries are built dynamically based on:
- Table structure
- Column types
- Primary keys
- Relationships

Example of dynamic query building:
```javascript
const buildUpdateQuery = (tableName, columns, primaryKey) => {
    const setClause = Object.keys(columns)
        .map((col, i) => `"${col}" = $${i + 1}`)
        .join(', ');
    return `UPDATE "${tableName}" SET ${setClause} WHERE "${primaryKey}" = $${Object.keys(columns).length + 1}`;
};
```

## Query Processing Flow

MCP follows a systematic flow for processing queries:

1. **Request Reception**
```javascript
@app.route('/api/query', methods=['POST'])
def execute_query():
    data = request.json
    # Dynamic processing based on query type
    response = process_query(data)
    return jsonify(response)
```

2. **Query Analysis**
- Validates table existence
- Checks column types
- Verifies constraints
- Optimizes query plan

3. **Execution Pipeline**
```javascript
const executeQuery = async (query, params = [], options = {}) => {
    // 1. Query validation
    validateQuery(query);

    // 2. Check cache
    const cachedResult = checkCache(query, params);
    if (cachedResult) return cachedResult;

    // 3. Get connection from pool
    const client = await pool.connect();

    try {
        // 4. Execute query
        const result = await client.query(query, params);
        
        // 5. Process results
        return processResults(result);
    } finally {
        // 6. Release connection
        client.release();
    }
};
```

## Schema Analysis

MCP performs real-time schema analysis:

1. **Table Analysis**
```javascript
const analyzeTable = async (tableName) => {
    // Get primary key
    const pkQuery = `
        SELECT a.attname
        FROM pg_index i
        JOIN pg_attribute a ON a.attrelid = i.indrelid
        WHERE i.indrelid = $1::regclass
        AND i.indisprimary
    `;

    // Get foreign keys
    const fkQuery = `
        SELECT
            tc.constraint_name,
            kcu.column_name,
            ccu.table_name AS foreign_table_name,
            ccu.column_name AS foreign_column_name
        FROM information_schema.table_constraints tc
        JOIN information_schema.key_column_usage kcu
        JOIN information_schema.constraint_column_usage ccu
        WHERE tc.constraint_type = 'FOREIGN KEY'
        AND tc.table_name = $1
    `;

    // Execute in parallel
    const [pk, fk] = await Promise.all([
        executeQuery(pkQuery, [tableName]),
        executeQuery(fkQuery, [tableName])
    ]);

    return {
        primaryKey: pk.rows[0]?.attname,
        foreignKeys: fk.rows
    };
};
```

2. **Column Analysis**
```javascript
const analyzeColumns = async (tableName) => {
    const query = `
        SELECT 
            column_name,
            data_type,
            character_maximum_length,
            numeric_precision,
            numeric_scale,
            is_nullable,
            column_default
        FROM information_schema.columns
        WHERE table_name = $1
    `;
    return await executeQuery(query, [tableName]);
};
```

## Connection Management

MCP uses a sophisticated connection pooling system:

```javascript
const pool = new Pool({
    max: process.env.DB_POOL_MAX || 20,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
    allowExitOnIdle: true
});

// Connection event handling
pool.on('connect', client => {
    client.query('SET application_name = $1', ['mcp_client']);
});

pool.on('error', (err, client) => {
    console.error('Unexpected error on idle client', err);
});
```

## Cursor Support

MCP supports cursor-based operations for large datasets:

### 1. Cursor Implementation
```javascript
const fetchWithCursor = async (query, batchSize = 1000) => {
    const client = await pool.connect();
    try {
        // Begin transaction
        await client.query('BEGIN');

        // Declare cursor
        await client.query(`DECLARE mcp_cursor CURSOR FOR ${query}`);

        let results = [];
        let hasMore = true;

        // Fetch in batches
        while (hasMore) {
            const res = await client.query(`FETCH ${batchSize} FROM mcp_cursor`);
            if (res.rows.length > 0) {
                results = results.concat(res.rows);
            }
            hasMore = res.rows.length === batchSize;
        }

        // Close cursor and transaction
        await client.query('CLOSE mcp_cursor');
        await client.query('COMMIT');

        return results;
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
};
```

### 2. Using Cursors for Large Datasets
```javascript
const streamLargeDataset = async (tableName, callback, options = {}) => {
    const batchSize = options.batchSize || 1000;
    const query = `SELECT * FROM ${tableName}`;
    
    const client = await pool.connect();
    try {
        await client.query('BEGIN');
        await client.query(`DECLARE data_cursor CURSOR FOR ${query}`);
        
        let processing = true;
        while (processing) {
            const result = await client.query(`FETCH ${batchSize} FROM data_cursor`);
            if (result.rows.length > 0) {
                await callback(result.rows);
            }
            processing = result.rows.length === batchSize;
        }
        
        await client.query('CLOSE data_cursor');
        await client.query('COMMIT');
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
};
```

### Example Usage of Cursor
```javascript
// Example: Processing large table in batches
app.get('/api/large-table-export', async (req, res) => {
    res.setHeader('Content-Type', 'application/json');
    res.write('[');
    
    let first = true;
    await streamLargeDataset('large_table', async (batch) => {
        for (const row of batch) {
            if (!first) res.write(',');
            res.write(JSON.stringify(row));
            first = false;
        }
    });
    
    res.write(']');
    res.end();
});
```

## Performance Considerations

1. **Connection Pool Optimization**
```javascript
const optimizePool = {
    // Maintain minimum connections
    min: process.env.DB_POOL_MIN || 2,
    
    // Maximum connections based on CPU cores
    max: process.env.DB_POOL_MAX || Math.min(20, os.cpus().length * 4),
    
    // Idle timeout
    idleTimeoutMillis: 30000,
    
    // Connection timeout
    connectionTimeoutMillis: 5000
};
```

2. **Query Optimization**
```javascript
const optimizeQuery = (query, params) => {
    // Add query hints for better execution plans
    if (query.toLowerCase().includes('select')) {
        query = query.replace(/SELECT/i, 'SELECT /*+ INDEX(table_name) */');
    }
    return { query, params };
};
```

This documentation explains how MCP dynamically handles database operations without hardcoding, and how it can be extended to support cursors and other database features. The system is designed to be database-agnostic through its plugin architecture while providing optimized performance through connection pooling and cursor support for large datasets.
