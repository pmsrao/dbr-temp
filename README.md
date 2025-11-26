# Databricks Account Usage Dashboard - Query Extractor

This repository contains extracted SQL queries and specifications from the Databricks "Account Usage Dashboard v2" dashboard.

## Overview

This project reverse-engineers a Databricks dashboard JSON file to extract:
- All SQL queries with proper formatting
- Query parameters and their default values
- Complete dashboard specification documentation

## Files

- **`extracted_queries.json`** - All 28 queries extracted in JSON format with:
  - `displayName`: Query name
  - `sqlQuery`: Formatted SQL query (with newlines)
  - `parameters`: Array of parameter definitions

- **`extracted_queries_readable.md`** - Human-readable markdown file with all SQL queries formatted for easy viewing

- **`DASHBOARD_SPECIFICATION.md`** - Complete reverse-engineered specification document including:
  - Dashboard structure
  - Query categories
  - Parameter types and syntax
  - Common SQL patterns
  - Data sources
  - Usage patterns

- **`Account Usage Dashboard v2.lvdash.json`** - Original Databricks dashboard file (source)

## Query Statistics

- **Total Queries**: 28
- **Queries with Parameters**: 12
- **Queries without Parameters**: 16
- **Total Parameters**: 118

## Query Categories

### Selection/Filter Queries (18 queries)
- Toggle options
- Dimension selections (time, grouping keys)
- Workspace lists
- Tag selections
- Product/feature selections

### Data Queries (10 queries)
- Usage overview
- Usage forecast
- Tag analysis
- Top spending assets
- Contract burndown

## Usage

### Reading the JSON file

```python
import json

with open('extracted_queries.json', 'r') as f:
    queries = json.load(f)

# Access a query
query = queries[0]
print(query['displayName'])
print(query['sqlQuery'])  # SQL with actual newlines when parsed
print(query['parameters'])
```

### Viewing formatted SQL

The `extracted_queries_readable.md` file contains all queries with properly formatted SQL for easy reading.

## Parameter Types

- **STRING**: Single string value
- **DATE**: Single date value
- **INTEGER**: Integer value
- **DECIMAL**: Decimal value
- **MULTI**: Multiple selection (array)
- **RANGE**: Date range (min/max)

## Data Sources

The dashboard queries use:
- `system.billing.usage` - Main billing usage table
- `system.billing.list_prices` - List pricing data
- `system.billing.account_prices` - Account-specific pricing
- `system.access.workspaces_latest` - Workspace information

## Notes

- SQL queries in JSON use `\n` escape sequences (JSON standard)
- When parsed, `\n` automatically becomes actual newlines
- All queries are formatted with proper indentation and line breaks
- Parameters include default values extracted from the dashboard

## License

This is a reverse-engineered specification for educational and reference purposes.

