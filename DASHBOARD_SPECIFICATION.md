# Account Usage Dashboard v2 - Reverse Engineered Specification

## Overview

This document provides a reverse-engineered specification of the Databricks "Account Usage Dashboard v2" based on the dashboard JSON file. The dashboard provides comprehensive billing and usage analytics for Databricks accounts.

## Dashboard Structure

- **Total Datasets (Queries)**: 28
- **Total Pages**: 6
- **Queries with Parameters**: 12
- **Queries without Parameters**: 16

## Query Categories

### 1. Selection/Filter Queries (16 queries)
These queries provide dropdown/selection options for dashboard filters:

- **Toggle Options**: Simple array selections for UI toggles
- **Dimension Selections**: Time periods, grouping keys, workspace lists
- **Tag Selections**: Tag keys and values for filtering
- **Product/Feature Selections**: Billing origin products, serverless flags

### 2. Data Queries (12 queries)
These queries retrieve actual usage and billing data:

- **Usage Overview**: Main aggregated usage/cost data
- **Tag Analysis**: Tag-based cost breakdowns
- **Ranking Queries**: Top N analysis by various dimensions
- **Detailed Usage**: Granular usage data

## Query Extraction Format

Each extracted query contains:
- `displayName`: Human-readable query name
- `sqlQuery`: Complete SQL query (formatted for readability)
- `parameters`: Array of parameter definitions with:
  - `displayName`: Parameter display name
  - `keyword`: Parameter keyword used in SQL (`:keyword`)
  - `dataType`: Parameter data type (STRING, DATE, etc.)
  - `defaultValue`: Default value for the parameter
  - `complexType`: Optional (MULTI, RANGE) for complex parameter types

## Parameter Types

### Simple Parameters
- **STRING**: Single string value
- **DATE**: Single date value

### Complex Parameters
- **MULTI**: Multiple selection (array of values)
- **RANGE**: Date range (min/max values)

### Parameter Syntax in SQL
- Simple: `:param_name`
- Range: `:param_name.min` and `:param_name.max`
- Multi: Used with `array_contains(:param_name, value)` or `array_contains(:param_name, 'all')`

## Key Queries

### Main Data Queries

1. **usage_overview** - Primary usage/cost aggregation query
   - Parameters: workspace, time_range, product_category, is_serverless, discounts_by_product, price_table, time_key, group_key, usage_toggle
   - Returns: Time-series usage data grouped by selected dimensions

2. **Tag Analysis Queries** - Multiple queries for tag-based analysis
   - Tag key selection
   - Tag value filtering
   - Tag aggregation modes

3. **Ranking Queries** - Top N analysis queries
   - By workspace, product, SKU
   - By various metadata dimensions

### Selection Queries

1. **select_workspace** - Workspace dropdown options
2. **select_tag_key** - Available tag keys for filtering
3. **select_time_key_overview** - Time aggregation options (Day, Week, Month, Quarter, Year)
4. **select_group_key** - Grouping dimension options (Workspace, Product, SKU)
5. **select_billing_origin_product** - Available billing products
6. **select_serverless_flag** - Serverless/Classic filter options

## Data Sources

The dashboard queries primarily use:
- `system.billing.usage` - Main billing usage table
- `system.billing.list_prices` - List pricing data
- `system.billing.account_prices` - Account-specific pricing
- `system.access.workspaces_latest` - Workspace information

## Common Patterns

### Workspace Filtering
```sql
WHERE (array_contains(:param_workspace, workspace_full_name) OR array_contains(:param_workspace,'all'))
```

### Date Range Filtering
```sql
WHERE usage_date BETWEEN :time_range.min AND :time_range.max
```

### Product Filtering
```sql
WHERE (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all'))
```

### Serverless Filtering
```sql
WHERE (array_contains(:is_serverless, 
  CASE WHEN product_features.is_serverless = True THEN 'Serverless' ELSE 'Classic' END) 
  OR array_contains(:is_serverless,'all'))
```

## Pricing Logic

The dashboard supports:
- **List Prices**: `system.billing.list_prices` with `effective_list.default`
- **Account Prices**: `system.billing.account_prices` with `default`
- **Custom Discounts**: Product-specific discount overrides via `discounts_by_product` parameter
- **Global Discounts**: Wildcard discount (`*=0.1` for 10% off all)

## Time Aggregation

Supports multiple time granularities:
- Day
- Week
- Month
- Quarter
- Year

Uses `date_trunc()` function for aggregation.

## Grouping Dimensions

Supports grouping by:
- Workspace
- Product (billing_origin_product)
- SKU (sku_name)
- Tag Key-Value Pairs
- Matched Status (for tag matching)

## Usage Metrics

Supports two metric types:
- **DBUs**: Usage quantity in Databricks Units
- **Dollars**: Cost in USD (calculated from usage × pricing)

## Tag Analysis Features

- Tag key selection
- Tag value filtering
- Tag aggregation modes:
  - Key-Value Pairs
  - Keys only
  - Values only
- Tag matching status (Matched/Not Matched/All)

## Dashboard Pages

The dashboard contains 6 pages:
1. **Usage Overview** - Main usage and cost trends
2. **Usage Forecast** - Predictive cost analysis
3. **Tag Analysis** - Cost breakdown by custom tags
4. **Top Spending Assets** - Ranking and analysis of highest cost resources
5. **Contract Burndown** - Contract utilization tracking
6. Additional analysis pages

## Query List

### Selection/Filter Queries (18 queries)

1. `select_yes_no_tag_show_mismatch` - Tag matching filter options
2. `select_dbus_dollars_toggle` - Metric type toggle (DBUs vs Dollars)
3. `select_time_key_overview` - Time aggregation options
4. `select_group_key` - Grouping dimension options
5. `select_workspace` - Workspace selection dropdown
6. `select_tag_key` - Available tag keys
7. `select_rank_key` - Ranking dimension options
8. `select_serverless_flag` - Serverless/Classic filter
9. `select_billing_origin_product` - Product category options
10. `select_tag_group` - Tag grouping options
11. `select_tag_agg_toggle` - Tag aggregation mode
12. `select_tag_agg_top_tags` - Top tags aggregation mode
13. `select_tag_search_mode` - Tag search logic (AND/OR)
14. `select_include_remaining` - Include remaining items toggle
15. `select_case_sensitivity` - Case sensitivity toggle
16. `select_feature_flag_toggle` - Feature flag toggle
17. `select_account_prices_toggle` - Pricing table selection
18. `select_enable_links_toggle` - Enable/disable links toggle

### Data Queries (10 queries)

1. **usage_overview** - Main usage/cost aggregation (9 parameters)
2. **usage_forecast** - Usage forecasting (11 parameters)
3. **usage_overview_pop_matrix** - Population matrix view (9 parameters)
4. **tag_analysis_top_tags** - Top tags analysis (11 parameters)
5. **tag_analysis_summary** - Tag analysis summary (14 parameters)
6. **tag_analysis_pop_matrix** - Tag population matrix (14 parameters)
7. **tag_analysis_match_details** - Tag matching details (15 parameters)
8. **top_spending_assets_summary** - Top spending summary (11 parameters)
9. **top_spending_assets_pop_matrix** - Top spending matrix (12 parameters)
10. **contract_burndown** - Contract utilization (6 parameters)

## Common Parameters

### Workspace Parameters
- `param_workspace` (STRING, MULTI) - Workspace selection, default: "all"
- Supports multi-select or single workspace

### Time Parameters
- `time_range` (DATE, RANGE) - Date range filter
- Default formats: "now-30d/d to now/d" or "now-12M/M to now-1M/M"
- Uses `.min` and `.max` in SQL: `:time_range.min` and `:time_range.max`

### Product Parameters
- `product_category` (STRING, MULTI) - Billing origin product filter
- Default: "all" (all products)
- Uses `array_contains()` for filtering

### Serverless Parameters
- `is_serverless` (STRING, MULTI) - Serverless/Classic filter
- Options: "all", "Serverless", "Classic"
- Default: "all"

### Pricing Parameters
- `price_table` (STRING) - Pricing table selection
- Options: "system.billing.list_prices" or "system.billing.account_prices"
- Default: "system.billing.account_prices"

- `discounts_by_product` (STRING) - Custom discount overrides
- Format: "product=discount;product2=discount2"
- Example: "sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0"
- Supports global discount: "*=0.1" (10% off all)

### Display Parameters
- `param_time_key` (STRING) - Time aggregation level
- Options: "Day", "Week", "Month", "Quarter", "Year"
- Default: "Day"

- `param_group_key` (STRING) - Grouping dimension
- Options: "Workspace", "Product", "SKU"
- Default: "Workspace"

- `usage_toggle` (STRING) - Metric display type
- Options: "DBUs", "Dollars"
- Default: "Dollars"

- `param_top_n` (INTEGER) - Number of top items to show
- Default: 10

## Notes

- The dashboard uses Databricks SQL parameter syntax (`:param_name`)
- Complex parameters use special syntax (`.min`, `.max` for ranges)
- Multi-select parameters use `array_contains()` for filtering
- The dashboard supports dynamic pricing table selection
- Custom discount overrides can be applied per product
- Global discounts use wildcard syntax (`*=discount`)
- Date parameters support relative date syntax (e.g., "now-30d/d", "now-12M/M")
- The dashboard uses ANSI mode: `SET ansi_mode = true;`
- Some queries use `IDENTIFIER()` function for dynamic table references

