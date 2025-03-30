# MCP (Model-Controller-Plugin) Server

## Overview

MCP is a flexible, extensible database connectivity layer designed to simplify data access and analysis operations. It provides a unified API for connecting to different database systems through a plugin architecture, making it easy to work with various data sources while maintaining a consistent interface.

![MCP Architecture](https://mermaid.ink/img/pako:eNqFkc9qwzAMxl_F-NQW2ifoYYcdBmN76KXsMEJtJTG140CUwkL67nOSNSnt2JyEP33fT5KPMLZWQ4S1c4_i4Ky86_qnDVlHK9wYfUB-qapKPSQ0ghM_EZ4pvJGx0oW1qKn9SuyjI6-ppVPCZhBblXH2kXCo_jzcvXz8_GFi3FvZYXgm9TnXu3VtW5X3bnRWGqG9tzrhi0_2EyO-E2GIpzMdzzp3Q57TQVX1bDb93L5VMzKNNlvH_YaMvG4UYa5SN0WkVsPCpkD9wAcG-1FaOHr8R4gVV5IBmgMTgV7CsGcVFjAUkIeUfOIuxAJYGCL10A8dJvI5Fze5tgK2ZiIu2XKZIFqNfzVmMHBpYQYLmA2hS7y4FMOxgFdCIoQtfQPJZJ21?type=png)

## Key Features

- **Plugin Architecture**: Easily connect to different databases using plugins
- **Unified API**: Consistent interface regardless of the underlying database
- **Advanced Query Capabilities**: Support for complex filtering, pagination, and search
- **RESTful Interface**: HTTP endpoints for all database operations
- **Metadata Inspection**: Explore database structure without prior knowledge
- **Natural Language Processing**: Execute database operations using natural language
- **Structured Command Processing**: Support for client-side LLM integration

## Core Architecture

MCP follows a modular architecture with three main components:

1. **Core**: Central server and routing logic
2. **Plugins**: Database-specific implementations
3. **API Layer**: RESTful endpoints for client applications

### Core Component

The core manages server initialization, routing, and plugin loading:

- `server.js`: Express server setup and initialization
- `router.js`: API route definitions and request handling
- `middleware.js`: Request processing middleware
- `utils.js`: Shared utility functions

### Plugin System

Plugins implement database-specific operations while conforming to a standard interface:

```
plugins/
├── postgres/         # PostgreSQL plugin
│   ├── connection.js # Database connection management
│   ├── index.js      # Plugin entry point and handler mapping
│   ├── inspect.js    # Database metadata inspection
│   ├── nlp.js        # Natural language processing
│   ├── queries.js    # Core query execution
│   └── search.js     # Advanced search capabilities
├── mongodb/          # MongoDB plugin (future)
└── mysql/            # MySQL plugin (future)
```

## Getting Started

### Prerequisites

- Node.js 14+
- NPM or Yarn
- Access to a supported database (PostgreSQL by default)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-repo/mcp-server.git
   cd mcp-server
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure your database connection:
   ```bash
   cp config/config.example.js config/config.js
   # Edit config.js with your database credentials
   ```

4. Start the server:
   ```bash
   npm start
   ```

## Configuration

The server is configured via the `config/config.js` file:

```javascript
module.exports = {
  // Selected database plugin (corresponds to the folder name in plugins/)
  dbType: 'postgres',
  
  // Database connection configuration
  dbConfig: {
    host: 'localhost',
    port: 5432,
    database: 'your_database',
    user: 'username',
    password: 'password'
  },
  
  // Server settings
  server: {
    port: 3000,
    corsOrigin: '*'
  }
};
```

## API Reference

### Database Inspection

- `GET /inspect`: Get overview of all tables and structure
- `GET /api/tables`: List all tables in the database
- `GET /api/table-schema/:tableName`: Get schema for a specific table

### Query Execution

- `POST /query`: Execute custom SQL or database-specific query
- `POST /data/search`: Search for rows matching specific criteria

### Advanced Search

- `POST /search/id`: Find a record by its ID
- `POST /search/column`: Smart search on a specific column
- `POST /search/multi`: Search across multiple columns

### Natural Language Processing

- `POST /nlp`: Execute database operations using natural language
- `POST /execute-structured-command`: Process structured commands from client-side LLMs

### Index Management

- `POST /index/create`: Create a new index
- `GET /index/list`: List existing indexes
- `POST /index/drop`: Remove an index
- `GET /index/recommend`: Get index recommendations

## Extending with New Plugins

To create a new database plugin:

1. Create a new folder in the `plugins` directory:
   ```bash
   mkdir plugins/your-database
   ```

2. Implement the required interface files:
   - `connection.js`: Connection management
   - `index.js`: Plugin entry point and handler mapping
   - `inspect.js`: Database metadata inspection
   - `queries.js`: Core query execution
   - Additional functionality as needed

3. Update your plugin to implement the standard handler interface:
   ```javascript
   module.exports = {
     connect, disconnect,
     executeQuery, executePaginatedQuery,
     // ... other required methods
     handlers: {
       // Standard handlers mapped to your implementation
     }
   };
   ```

4. Change the `dbType` in config to use your new plugin:
   ```javascript
   module.exports = {
     dbType: 'your-database',
     // ...
   };
   ```

## Plugin Implementation Guide

Each plugin must implement the following core components:

### Connection Management

Methods for connecting to and disconnecting from the database:

```javascript
const connect = async (config) => {
  // Implementation specific to database
};

const disconnect = async () => {
  // Close connections properly
};
```

### Query Execution

Standard methods for executing queries:

```javascript
const executeQuery = async (query, params) => {
  // Execute raw query
};

const executePaginatedQuery = async (query, page, pageSize) => {
  // Execute query with pagination
};

const executeFilteredQuery = async (tableName, columns, filters, page, pageSize) => {
  // Build and execute filtered query
};
```

### Metadata Inspection

Methods to explore database structure:

```javascript
const inspectDatabase = async () => {
  // Get overview of all tables
};

const getTableColumns = async (tableName) => {
  // Get column details for a table
};

const getTableSchema = async (tableName) => {
  // Get full schema for a table
};
```

## Under the Hood - Data Flow

When a client makes a request to MCP:

1. The request is received by the Express server in `server.js`
2. It's routed through `router.js` to the appropriate handler
3. The router delegates to the active plugin based on `dbType`
4. The plugin performs the database-specific operations
5. Results are transformed into a standardized response format
6. The response is sent back to the client

This abstraction allows clients to work with any supported database using the same API.

## Integrating with Client Applications

MCP is designed to be a backend service for data-driven applications. To integrate with a client:

1. Connect to MCP's API endpoints using HTTP requests
2. Use the unified API to query data, regardless of the underlying database
3. Leverage advanced features like NLP and structured commands for AI-driven applications

## Common Use Cases

### Data Analysis Applications

MCP excels at providing clean, uniform access to data for analysis:

```javascript
// Client-side code example
async function analyzeCustomerData() {
  // Get data with filtering
  const response = await fetch('http://localhost:3000/data/search', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      tableName: 'customers',
      column: 'signup_date',
      value: '2023-01-01',
      operator: '>='
    })
  });
  
  const data = await response.json();
  // Proceed with analysis
}
```

### AI-Assisted Database Operations

MCP's natural language processing enables AI applications to work with data:

```javascript
// Example of NLP query
const response = await fetch('http://localhost:3000/nlp', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    query: "Show me the top 10 customers by order value in the last month"
  })
});

const result = await response.json();
// Process AI-generated results
```

## Performance Considerations

- Use pagination for large result sets
- Consider index recommendations for frequently queried columns
- Monitor query performance and optimize as needed

## Troubleshooting

Common issues and solutions:

- **Connection failures**: Check database credentials in config.js
- **Slow queries**: Review execution plans and consider indexing
- **Memory issues**: Adjust page sizes for large datasets


