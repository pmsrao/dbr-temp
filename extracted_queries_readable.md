# Extracted SQL Queries

Total queries: 28

---

## Query 1: select_yes_no_tag_show_mismatch

### SQL Query

```sql
select
    explode(array(
    'Not Matched Only',
    'Matched Only',
    'All Usage'
  )) as toggle
```

---

## Query 2: select_dbus_dollars_toggle

### SQL Query

```sql
select
    explode(array(
    'DBUs',
    'Dollars'
  )) as toggle
```

---

## Query 3: select_time_key_overview

### SQL Query

```sql
select
    explode(array(
    'Day',
    'Week',
    'Month',
    'Quarter',
    'Year'
  )) as time_key
```

---

## Query 4: select_group_key

### SQL Query

```sql
select
    explode(array(
    'Workspace',
    'Product',
    'SKU' --'Photon',
    --'Serverless'
  )) as group_key
```

---

## Query 5: select_workspace

### SQL Query

```sql
with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest ) select
    DISTINCT workspace_full_name
from
    workspace
```

---

## Query 6: select_tag_key

### Parameters

- **param_workspace** (`param_workspace`): STRING = `<ALL WORKSPACES>`
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]

### SQL Query

```sql
with -- parse workspaces json workspace as ( select
    explode( map_entries(from_json('$$$__WORKSPACES_JSON__$$$', 'map<string,string>')) ) as kvp, kvp['key'] as workspace_id, kvp['value'] as workspace_name ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else concat(workspace_name, ' (id: ', u.workspace_id, ')')
    end as workspace, u.*
from
    system.billing.usage as u
  left join
    workspace
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *
from
    usage_with_ws_filtered_by_date
where
    if(:param_workspace='<ALL WORKSPACES>', true, workspace = :param_workspace) -- all workspaces under account, or single workspace ),
 -- query tag_key_selection as ( select
    distinct(explode(map_keys(custom_tags))) as tag_key
from
    usage_filtered ) select
    *
from
    tag_key_selection
```

---

## Query 7: select_rank_key

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]

### SQL Query

```sql
with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    usage_metadata
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND   (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 -- enumerate all usage_metadata keys usage_metadata_keys as ( select
    distinct(explode( map_keys(from_json( to_json(usage_metadata),
 'map<string,string>' )) )) as rank_key
from
    usage_filtered ) select
    explode(array(
    'workspace',
    'run_as'
  )) as rank_key union all select
    rank_key
from
    usage_metadata_keys
```

---

## Query 8: select_serverless_flag

### SQL Query

```sql
select
    explode(array(
    'all',
    'Serverless',
    'Classic'
  )) as group_key
```

---

## Query 9: select_billing_origin_product

### SQL Query

```sql
select
    DISTINCT billing_origin_product
from
    system.billing.usage
where
    billing_origin_product IS NOT NULL
```

---

## Query 10: select_tag_group

### SQL Query

```sql
select
    explode(array(
    'Tag Key-Value Pairs',
    'Workspace',
    'Product',
    'SKU',
    --'Serverless',
    --'Photon',
    'Matched Status'
  )) as group_key
```

---

## Query 11: select_tag_agg_toggle

### SQL Query

```sql
select
    explode(array(
    'Key-Value Pairs',
    'Keys'
  )) as toggle
```

---

## Query 12: select_tag_agg_top_tags

### SQL Query

```sql
select
    explode(array(
    'Key-Value Pairs',
    'Keys',
    'Values'
  )) as toggle
```

---

## Query 13: select_tag_search_mode

### SQL Query

```sql
select
    explode(array(
    'AND',
    'OR'
  )) as group_key
```

---

## Query 14: select_include_remaining

### SQL Query

```sql
select
    explode(array(
    'Yes',
    'No'
  )) as toggle
```

---

## Query 15: select_case_sensitivity

### SQL Query

```sql
select
    explode(array(
    'Yes',
    'No'
  )) as toggle
```

---

## Query 16: select_feature_flag_toggle

### SQL Query

```sql
select
    explode(array(
    'Enabled',
    'Disabled'
  )) as toggle
```

---

## Query 17: select_account_prices_toggle

### SQL Query

```sql
select
    explode(array(
    'system.billing.list_prices',
    'system.billing.account_prices'
  )) as toggle
```

---

## Query 18: select_enable_links_toggle

### SQL Query

```sql
select
    explode(array(
    'Enabled',
    'Disabled'
  )) as toggle
```

---

## Query 19: usage_overview

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **time_range** (`time_range`): DATE = `now-30d/d to now/d` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **price_table** (`price_table`): STRING = `system.billing.account_prices`
- **param_time_key** (`param_time_key`): STRING = `Month`
- **param_group_key** (`param_group_key`): STRING = `Product`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`

### SQL Query

```sql
SET ansi_mode = true; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND   (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' when :param_time_key = 'Quarter'
      then 'usage_quarter' when :param_time_key = 'Month'
      then 'usage_month' when :param_time_key = 'Week'
      then 'usage_week' when :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date as time_key, *
from
    list_priced_usd ),
 list_priced_usd_with_time_and_group_keys as ( select
    workspace as workspace_norm, case
      when :param_group_key = 'Workspace'
      then workspace_norm WHEN :param_group_key = 'Product'
      then billing_origin_product WHEN :param_group_key = 'SKU'
      then sku_name WHEN :param_group_key = 'Photon'
      then IsPhoton WHEN :param_group_key = 'Serverless'
      then IsServerless
    end AS group_key, *
from
    list_priced_usd_with_time_key u ),
 clean_results AS ( -- query select
    time_key, group_key, case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end AS usage_usd_dynamic, usage_usd, usage_unit, usage_dbus, IsServerless, workspace_norm, 'Actuals' AS usage_type, CONCAT('per ', CAST(:param_time_key AS STRING)) AS time_period, start_time AS start_time_window, end_time AS end_time_window, CONCAT(CAST(start_time AS STRING),
 ' to ', CAST(end_time AS STRING)) AS time_window_string
from
    list_priced_usd_with_time_and_group_keys ) select
    *
from
    clean_results
```

---

## Query 20: usage_forecast

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **time_range** (`time_range`): DATE = `now-14d/d to now/d` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **price_table** (`price_table`): STRING = `system.billing.account_prices`
- **param_time_key** (`param_time_key`): STRING = `Day`
- **param_group_key** (`param_group_key`): STRING = `Product`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **units_to_predict** (`units_to_predict`): DECIMAL = `3`
- **forecast_toggle** (`forecast_toggle`): STRING = `Enabled`

### SQL Query

```sql
SET ansi_mode = true; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 usage_with_ws_filtered_by_date AS ( select
    case
      when workspace_name IS NULL
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end AS workspace, u.*
from
    system.billing.usage AS u
  INNER join
    workspace
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date BETWEEN :time_range.min AND :time_range.max ),
 usage_filtered AS ( select
    *, case
      when product_features.is_serverless = TRUE
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = TRUE
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND (array_contains(:is_serverless, case
      when product_features.is_serverless = TRUE
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) AS usage_year, date_trunc('QUARTER', usage_date) AS usage_quarter, date_trunc('MONTH', usage_date) AS usage_month, date_trunc('WEEK', usage_date) AS usage_week, date_trunc('DAY', usage_date) AS usage_day, date_trunc('HOUR', usage_start_time) AS usage_hour, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered AS u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  LEFT join
    prices AS p
    on u.sku_name=p.sku_name AND u.usage_unit=p.usage_unit AND (u.usage_end_time BETWEEN p.price_start_time AND p.coalesced_price_end_time) ),
 list_priced_usd_with_time_key AS ( select
    identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' WHEN :param_time_key = 'Quarter'
      then 'usage_quarter' WHEN :param_time_key = 'Month'
      then 'usage_month' WHEN :param_time_key = 'Week'
      then 'usage_week' WHEN :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date AS time_key, *
from
    list_priced_usd ),
 list_priced_usd_with_time_and_group_keys AS ( select
    workspace AS workspace_norm, case
      when :param_group_key = 'Workspace'
      then workspace_norm WHEN :param_group_key = 'Product'
      then billing_origin_product WHEN :param_group_key = 'SKU'
      then sku_name WHEN :param_group_key = 'Photon'
      then IsPhoton WHEN :param_group_key = 'Serverless'
      then IsServerless
    end AS group_key, *
from
    list_priced_usd_with_time_key u ),
 clean_results AS ( select
    time_key, MAX(usage_hour) AS max_usage_hour, -- For intraday forecasting SUM(case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd END) AS usage_usd_dynamic, 'Actuals' AS usage_type, MIN(start_time) AS start_time_window, MAX(end_time) AS end_time_window
from
    list_priced_usd_with_time_and_group_keys GROUP BY time_key ),
 clean_results_excluding_current_period AS ( select
    *
from
    clean_results
where
    time_key < DATE_TRUNC(:param_time_key, CURRENT_DATE) ),
 forecast AS ( with
  time_range as (
     select
    case
      when :param_time_key = 'Month'
      then timestampadd(MONTH,  :units_to_predict, now()) WHEN :param_time_key = 'Day'
      then timestampadd(DAY, :units_to_predict, now()) WHEN :param_time_key = 'Week'
      then timestampadd(WEEK, :units_to_predict, now()) WHEN :param_time_key = 'Quarter'
      then timestampadd(QUARTER, :units_to_predict, now()) WHEN :param_time_key = 'Year'
      then timestampadd(YEAR, :units_to_predict, now())
      else timestampadd(MONTH, :units_to_predict, now())
    end AS end_time, :forecast_toggle AS forecast_toggle ) select
    *, MIN(time_key) OVER() AS start_time_window, MAX(time_key) OVER() AS end_time_raw, case
      when UPPER(:param_time_key) = 'DAY'
      then DATE_ADD(end_time_raw, 0) WHEN UPPER(:param_time_key) = 'WEEK'
      then DATE_ADD(end_time_raw, 6) WHEN UPPER(:param_time_key) = 'MONTH'
      then DATE_ADD(DATE_TRUNC('MONTH', ADD_MONTHS(end_time_raw, 1)),
 -1) WHEN UPPER(:param_time_key) = 'QUARTER'
      then DATE_ADD(DATE_TRUNC('QUARTER', ADD_MONTHS(end_time_raw, 3)),
 -1) WHEN UPPER(:param_time_key) = 'YEAR'
      then DATE_ADD(DATE_TRUNC('YEAR', ADD_MONTHS(end_time_raw, 12)),
 -1)
    end AS end_time_window, case
      when :param_time_key = 'Day'
      then 24 -- for intraday forecast attribution WHEN :param_time_key = 'Week'
      then 7 WHEN :param_time_key = 'Month'
      then DAY(LAST_DAY(time_key)) WHEN :param_time_key = 'Quarter'
      then 90  -- Approximation WHEN :param_time_key = 'Year'
      then 365  -- Approximation
    end AS total_period_units, case
      when :param_time_key = 'Day'
      then 24 -- Must do this outside the forecast function WHEN :param_time_key = 'Week'
      then DATEDIFF(CURRENT_DATE, DATE_TRUNC('WEEK', CURRENT_DATE)) + 1 WHEN :param_time_key = 'Month'
      then DAY(CURRENT_DATE) WHEN :param_time_key = 'Quarter'
      then DATEDIFF(CURRENT_DATE, DATE_TRUNC('QUARTER', CURRENT_DATE)) + 1 WHEN :param_time_key = 'Year'
      then DATEDIFF(CURRENT_DATE, DATE_TRUNC('YEAR', CURRENT_DATE)) + 1
    end AS completed_period_units
from
    AI_FORECAST( TABLE (clean_results_excluding_current_period),
 horizon => (select
    MAX(end_time)
from
    time_range),
 time_col => 'time_key', value_col => ARRAY('usage_usd_dynamic'),
 prediction_interval_width => 0.9, parameters => '{"global_floor": 0}' )
where
    usage_usd_dynamic_forecast IS NOT NULL ),
 final_forecast AS ( select
    time_key, :param_time_key AS time_period, usage_usd_dynamic_forecast AS raw_forecast, total_period_units, completed_period_units, case
      when time_key = DATE_TRUNC(:param_time_key, CURRENT_DATE)
      then usage_usd_dynamic_forecast * ((total_period_units - completed_period_units) / total_period_units)
      else usage_usd_dynamic_forecast
    end AS usage_usd_dynamic_forecast, case
      when time_key = DATE_TRUNC(:param_time_key, CURRENT_DATE)
      then usage_usd_dynamic_upper * ((total_period_units - completed_period_units)  / total_period_units)
      else usage_usd_dynamic_upper
    end AS usage_usd_dynamic_upper, case
      when time_key = DATE_TRUNC(:param_time_key, CURRENT_DATE)
      then usage_usd_dynamic_lower * ((total_period_units - completed_period_units)  / total_period_units)
      else usage_usd_dynamic_lower
    end AS usage_usd_dynamic_lower, start_time_window, end_time_window, CONCAT(CAST(start_time_window AS STRING),
 ' to ', CAST(end_time_window AS STRING)) AS time_window_string
from
    forecast ),
 final_results AS ( select
    time_key, usage_usd_dynamic, NULL AS upper_forecast, NULL AS lower_forecast, usage_type, :param_time_key AS time_period, max_usage_hour, start_time_window, end_time_window, CONCAT(CAST(start_time_window AS STRING),
 ' to ', CAST(end_time_window AS STRING)) AS time_window_string
from
    clean_results UNION ALL select
    -- Intraday extrapolation time_key, usage_usd_dynamic + ((24 - HOUR(max_usage_hour) + 1) * (usage_usd_dynamic / (HOUR(max_usage_hour) + 1))) AS usage_usd_dynamic, NULL AS upper_forecast, NULL AS lower_forecast, 'Forecast', :param_time_key AS time_period, max_usage_hour, (select
    MAX(start_time_window)
from
    final_forecast) AS start_time_window, (select
    MAX(start_time_window)
from
    final_forecast) AS end_time_window, CONCAT(CAST(start_time_window AS STRING),
 ' to ', CAST(end_time_window AS STRING)) AS time_window_string
from
    clean_results
where
    :forecast_toggle = 'Enabled' AND :param_time_key = 'Day' AND time_key = end_time_window -- Add back last day to simulate partial linear extrapolation UNION ALL select
    time_key, usage_usd_dynamic_forecast AS usage_usd_dynamic, usage_usd_dynamic_upper AS upper_forecast, usage_usd_dynamic_lower AS lower_forecast, 'Forecast' AS usage_type, time_period AS time_period, NULL AS max_usage_hour, start_time_window, end_time_window, CONCAT(CAST(start_time_window AS STRING),
 ' to ', CAST(end_time_window AS STRING)) AS time_window_string
from
    final_forecast
where
    case
      when :forecast_toggle = 'Enabled'
      then TRUE
      else FALSE
    end ) select
    *, first_value(time_window_string) OVER(ORDER BY time_key DESC) AS forecast_time_window_string
from
    final_results
```

---

## Query 21: usage_overview_pop_matrix

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **param_time_key** (`param_time_key`): STRING = `Month`
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **param_group_key** (`param_group_key`): STRING = `Product`
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`

### SQL Query

```sql
SET ansi_mode = true; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' when :param_time_key = 'Quarter'
      then 'usage_quarter' when :param_time_key = 'Month'
      then 'usage_month' when :param_time_key = 'Week'
      then 'usage_week' when :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date as time_key, *
from
    list_priced_usd
where
    (array_contains(:is_serverless, IsServerless) OR array_contains(:is_serverless,'all')) ),
 list_priced_usd_with_time_and_group_keys as ( select
    workspace as workspace_norm, case
      when :param_group_key = 'Workspace'
      then workspace_norm WHEN :param_group_key = 'Product'
      then billing_origin_product WHEN :param_group_key = 'SKU'
      then sku_name WHEN :param_group_key = 'Photon'
      then IsPhoton WHEN :param_group_key = 'Serverless'
      then IsServerless
    end AS group_key, *
from
    list_priced_usd_with_time_key u ),
 -- calc usage by period grouped_usage_by_period as ( select
    time_key as period_key, replace(replace(group_key, '<', '&lt;'),
 '>', '&gt;') as group_key, -- Make dynamic SUM(case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd END) AS usage_usd
from
    list_priced_usd_with_time_and_group_keys group by all ),
 -- calc periodic change grouped_usage_change as ( select
    period_key, group_key, usage_usd, lag(usage_usd, 1) over (partition by group_key order by period_key) as prev_usage_usd, round(try_divide((usage_usd - prev_usage_usd),
 prev_usage_usd) * 100, 2) as usage_change_percentage
from
    grouped_usage_by_period ),
 total_usage_change as ( select
    period_key, '<b>TOTAL</b>' as group_key, sum(usage_usd) as usage_usd, lag(sum(usage_usd),
 1) over (order by period_key) as prev_usage_usd, round(try_divide((sum(usage_usd) - prev_usage_usd),
 prev_usage_usd) * 100, 2) as usage_change_percentage
from
    grouped_usage_by_period group by period_key ),
 -- periods period_info as ( select
    case
      when :param_time_key = 'Day'
      then :time_range.max::date when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date)
    end as current_period, case
      when :param_time_key = 'Day'
      then :time_range.max::date- interval 1 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 7 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 1 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 3 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 1 year)
    end as last_period, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 2 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 14 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 2 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 6 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 2 year)
    end as 2_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 3 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 21 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 3 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 9 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 3 year)
    end as 3_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 4 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 28 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 4 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 12 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 4 year)
    end as 4_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 5 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 35 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 5 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 15 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 5 year)
    end as 5_periods_ago ),
 -- pivot change usage_change_pivot as ( select
    case
      when period_key = current_period
      then 'Current period' when period_key = last_period
      then 'Last period' when period_key = 2_periods_ago
      then '2 periods ago' when period_key = 3_periods_ago
      then '3 periods ago' when period_key = 4_periods_ago
      then '4 periods ago' when period_key = 5_periods_ago
      then '5 periods ago'
    end as x_period_back, group_key, usage_usd, usage_change_percentage
from
    ( select
    *
from
    grouped_usage_change, period_info union all select
    *
from
    total_usage_change, period_info ) ),
 -- pivot all time all_time_usage_pivot as ( select
    'Start to
    end date' as x_period_back, group_key, sum(usage_usd) as usage_usd, null as usage_change_percentage
from
    grouped_usage_by_period group by group_key ),
 -- pivot total all time all_time_total_usage_pivot as ( select
    'Start to
    end date' as x_period_back, '<b>TOTAL</b>' as group_key, sum(usage_usd) as usage_usd, null as usage_change_percentage
from
    grouped_usage_by_period ),
 union_usage_pivot as ( select
    x_period_back, group_key, case
      when x_period_back = 'Start to
    end date'
      then string(usage_usd)
      else concat('<span style="zoom:1">', usage_usd_str, '</span><span style="zoom:1;color:', change_color, '">&nbsp;', usage_change_str, '</span>')
    end as usage_info
from
    ( select
    case
      when usage_usd >= 1e9
      then concat(format_number(usage_usd / 1e9, 0),
 'B') when usage_usd >= 1e6
      then concat(format_number(usage_usd / 1e6, 0),
 'M') when usage_usd >= 1e3
      then concat(format_number(usage_usd / 1e3, 0),
 'K')
      else format_number(usage_usd, 0)
    end as usage_usd_str, case
      when usage_change_percentage > 10
      then '#00A972' when usage_change_percentage < -10
      then '#FF3621'
      else '#919191'
    end as _change_color, concat('(', if(usage_change_percentage > 0, '+', ''),
 format_number(usage_change_percentage, 0),
 '%)') as _usage_change_str, coalesce(_change_color, '#919191') as change_color, coalesce(_usage_change_str, '') as usage_change_str, *
from
    ( select
    x_period_back, group_key, usage_usd, usage_change_percentage
from
    usage_change_pivot union all select
    *
from
    all_time_usage_pivot union all select
    *
from
    all_time_total_usage_pivot ) ) ),
 pre_format_results AS ( -- query select
    group_key, case
      when :usage_toggle = 'Dollars'
      then concat('$', format_number(float(`Start to
    end date`),
 0)) WHEN :usage_toggle = 'DBUs'
      then concat('', format_number(float(`Start to
    end date`),
 0))
    end as `Start to
    end date`, float(`Start to
    end date`) as _all_time_usage_usd, 2 as _order, coalesce(`5 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `5 periods ago`, coalesce(`4 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `4 periods ago`, coalesce(`3 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `3 periods ago`, coalesce(`2 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `2 periods ago`, coalesce(`Last period`, '<span style="zoom:1;color:#919191">0</span>') as `Last period`, coalesce(`Current period`, '<span style="zoom:1;color:#919191">0</span>') as `Current period`
from
    union_usage_pivot pivot ( first(usage_info) for x_period_back in ( 'Start to
    end date', '5 periods ago', '4 periods ago', '3 periods ago', '2 periods ago', 'Last period', 'Current period' ) ) union all ( select
    '' as group_key, concat('<b><i>', date_format(:time_range.min, 'MMM dd yyyy'),
 ' - ', date_format(:time_range.max, 'MMM dd yyyy'),
 '</i></b>') as `Start to
    end date`, null as _all_time_usage_usd, 1 as _order, concat('<b><i>', date_format(5_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(4_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `5 periods ago`, concat('<b><i>', date_format(4_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(3_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `4 periods ago`, concat('<b><i>', date_format(3_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(2_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `3 periods ago`, concat('<b><i>', date_format(2_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(last_period, -1),
 'MMM dd'),
 '</i></b>') as `2 periods ago`, concat('<b><i>', date_format(last_period, 'MMM dd'),
 ' - ', date_format(date_add(current_period, -1),
 'MMM dd'),
 '</i></b>') as `Last period`, concat('<b><i>', date_format(current_period, 'MMM dd'),
 ' -
    end Time', '</i></b>') as `Current period`
from
    period_info ) ) select
    *, MAX(_all_time_usage_usd) OVER() AS max_usage, MIN(_all_time_usage_usd) OVER() AS min_usage, -- add data bars COALESCE(CONCAT( '<div style="position: relative; height: 20px; border-radius: 4px; overflow: hidden;">', '<div style="position: absolute; width: ', ROUND(try_divide((_all_time_usage_usd - min_usage),
 NULLIF(max_usage - min_usage, 0)) * 100, 2),
 '%; height: 100%; background: #077A9D; border-radius: 4px;"></div>', '<span style="position: absolute; width: 100%; text-align: center; line-height: 20px; font-weight: bold; color: #00A972;">', case
      when :usage_toggle = 'Dollars'
      then '$'
      else '' END, FORMAT_NUMBER(_all_time_usage_usd, 0),
 '</span></div>' ),
 `Start to
    end date`) AS usage_bar_html
from
    pre_format_results order by _order asc, _all_time_usage_usd desc
```

---

## Query 22: tag_analysis_top_tags

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **top_tag_aggregate_by** (`top_tag_aggregate_by`): STRING = `Keys`
- **top_n_tags** (`top_n_tags`): DECIMAL = `10`
- **normalize_case** (`normalize_case`): STRING = `Yes`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **param_time_key** (`param_time_key`): STRING = `Month`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]

### SQL Query

```sql
SET ansi_mode = true; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 exploded_usage_tags_to_compare AS ( select
    record_id, identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' when :param_time_key = 'Quarter'
      then 'usage_quarter' when :param_time_key = 'Month'
      then 'usage_month' when :param_time_key = 'Week'
      then 'usage_week' when :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date as time_key, case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end AS usage_usd, workspace, IsServerless, IsPhoton, billing_origin_product, explode(custom_tags)
from
    list_priced_usd AS u ),
 pre_filter AS ( select
    case
      when :top_tag_aggregate_by = 'Keys'
      then case
      when :normalize_case = 'Yes'
      then trim(lower(key))
      else key
    end WHEN :top_tag_aggregate_by = 'Key-Value Pairs'
      then case
      when :normalize_case = 'Yes'
      then CONCAT(COALESCE(lower(trim(key)),
 ''),
 '=', COALESCE(lower(trim(value)),
 ''))
    end WHEN :top_tag_aggregate_by = 'Values'
      then case
      when :normalize_case = 'Yes'
      then trim(lower(value))
      else value
    end END AS dynamic_tag_rollup, workspace, time_key, IsServerless, IsPhoton, billing_origin_product, SUM(usage_usd) AS usage_usd
from
    exploded_usage_tags_to_compare GROUP BY ALL ),
 tag_rank AS ( select
    dynamic_tag_rollup, SUM(usage_usd) AS usage_usd, ROW_NUMBER() OVER(ORDER BY SUM(usage_usd) DESC) AS tag_rank_number
from
    pre_filter GROUP BY dynamic_tag_rollup ) select
    p.*, tag_rank_number
from
    pre_filter p
  INNER join
    tag_rank tr
    on p.dynamic_tag_rollup = tr.dynamic_tag_rollup
where
    tag_rank_number <= :top_n_tags
```

---

## Query 23: tag_analysis_summary

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`
- **param_time_key** (`param_time_key`): STRING = `Month`
- **param_tag_entries** (`param_tag_entries`): STRING = `Budget;Env`
- **normalize_case** (`normalize_case`): STRING = `Yes`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **param_show_tag_mismatch** (`param_show_tag_mismatch`): STRING = `All Usage`
- **tag_group** (`tag_group`): STRING = `Tag Key-Value Pairs`
- **tag_search_mode** (`tag_search_mode`): STRING = `AND`
- **group_agg_mode** (`group_agg_mode`): STRING = `Key-Value Pairs`

### SQL Query

```sql
SET ansi_mode = true; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' when :param_time_key = 'Quarter'
      then 'usage_quarter' when :param_time_key = 'Month'
      then 'usage_month' when :param_time_key = 'Week'
      then 'usage_week' when :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date as time_key, *
from
    list_priced_usd ),
 tag_entry_parsing AS ( select
    tag_entry, contains(tag_entry, '=') AS is_filter, IF(contains(tag_entry, '='),
 split(tag_entry, '=')[0], tag_entry) AS tag_key, ROW_NUMBER() OVER (ORDER BY tag_entry) AS tag_id
from
    ( select
    explode(split(:param_tag_entries, ';')) AS tag_entry
from
    VALUES(1) AS dummy(x) ) ),
 exploded_usage_tags_to_compare AS ( select
    record_id, explode(custom_tags),
 custom_tags
from
    usage_filtered AS u ),
 matched_records AS (select
    record_id, -- array of maps that match array_distinct(array_agg(case
      when tag_id IS NOT NULL
      then concat(spine.key, '=', spine.value) END)) AS matched_tags_array, array_distinct(array_agg(concat(spine.key, '=', spine.value))) AS usage_tags_array, -- Add keys only agg view array_distinct(array_agg(case
      when tag_id IS NOT NULL
      then spine.key END)) AS matched_keys_array, array_distinct(array_agg(spine.key)) AS usage_keys_array, -- size(matched_tags_array) AS matched_tag_count, (select
    COUNT (0)
from
    tag_entry_parsing
where
    length(tag_entry) > 0) AS search_tag_count, if(search_tag_count = 0 OR (case
      when :tag_search_mode = "AND"
      then search_tag_count <= matched_tag_count WHEN :tag_search_mode = "OR"
      then matched_tag_count > 0 END),
 -- Add key aggregation option case
      when :group_agg_mode = 'Key-Value Pairs'
      then array_join(array_distinct(matched_tags_array),
 ';') WHEN :group_agg_mode = 'Keys'
      then array_join(array_distinct(matched_keys_array),
 ';')
    end , '' )  as _custom_tag_key_value_pairs, if(_custom_tag_key_value_pairs = "", '<NO TAG MATCH>', _custom_tag_key_value_pairs) as custom_tag_key_value_pairs, -- Is match yes/no if (search_tag_count = 0 OR (case
      when :tag_search_mode = "AND"
      then search_tag_count <= matched_tag_count WHEN :tag_search_mode = "OR"
      then matched_tag_count > 0 END),
 '<TAG MATCH>', '<NO TAG MATCH>') AS IsMatched
from
    exploded_usage_tags_to_compare AS spine
  LEFT join
    tag_entry_parsing AS tt
    on (case
      when :normalize_case = 'Yes'
      then (-- Join
    on Key only if is_filter = false (trim(lower(tt.tag_key)) = trim(lower(spine.key)) AND tt.is_filter = false) OR (trim(lower(tt.tag_entry)) = concat(trim(lower(spine.key)),
 '=', trim(lower(spine.value))) AND tt.is_filter = true) )
      else (-- Join
    on Key only if is_filter = true (tt.tag_key = spine.key AND tt.is_filter = false) OR (tt.tag_entry = concat(spine.key, '=', spine.value) AND tt.is_filter = true) )
    end ) GROUP BY record_id ),
 -- match tag entries list_priced_usd_with_matching_tag_kvp as ( select
    mt._custom_tag_key_value_pairs, COALESCE(mt.custom_tag_key_value_pairs, '<NOT TAGGED>') AS custom_tag_key_value_pairs, -- There are matched tag records,
      then records with tags that dont match the ask,
      then totally untagged resources COALESCE(IsMatched, '<NOT TAGGED>') AS IsMatchedClean, array_distinct(mt.matched_tags_array) AS kvp, k.*
from
    list_priced_usd_with_time_key k
  left join
    matched_records as mt
    on k.record_id = mt.record_id ),
 pre_ui_results AS ( -- query select
    time_key, IsServerless, IsPhoton, workspace AS workspace_id, billing_origin_product, sku_name, IsMatchedClean, case
      when custom_tag_key_value_pairs IN ('<NO TAG MATCH>', '<NOT TAGGED>') OR :normalize_case = 'No'
      then custom_tag_key_value_pairs
      else lower(custom_tag_key_value_pairs)
    end AS custom_tag_key_value_pairs, -- Make dynamic case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end AS usage_usd, kvp
from
    list_priced_usd_with_matching_tag_kvp
where
    ( case
      when :param_show_tag_mismatch = 'All Usage'
      then True WHEN :param_show_tag_mismatch = 'Not Matched Only'
      then custom_tag_key_value_pairs IN ('<NO TAG MATCH>', '<NOT TAGGED>') WHEN :param_show_tag_mismatch = 'Matched Only'
      then custom_tag_key_value_pairs NOT IN ('<NO TAG MATCH>', '<NOT TAGGED>')
    end ) ) select
    case
      when :tag_group = 'Workspace'
      then workspace_id WHEN :tag_group = 'Product'
      then billing_origin_product WHEN :tag_group = 'SKU'
      then sku_name WHEN :tag_group = 'Matched Status'
      then IsMatchedClean WHEN :tag_group = 'Tag Key-Value Pairs'
      then custom_tag_key_value_pairs WHEN :tag_group = 'Photon'
      then IsPhoton WHEN :tag_group = 'Serverless'
      then IsServerless
    end AS group_key, case
      when IsMatchedClean = '<TAG MATCH>'
      then usage_usd
      else 0
    end AS Measure_MatchedUsage, *
from
    pre_ui_results
```

---

## Query 24: tag_analysis_pop_matrix

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **param_time_key** (`param_time_key`): STRING = `Month`
- **param_tag_entries** (`param_tag_entries`): STRING = `budgetpolicyname;budgetpolicyid`
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **group_agg_mode** (`group_agg_mode`): STRING = `Key-Value Pairs`
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **price_table** (`price_table`): STRING = `system.billing.list_prices`
- **tag_search_mode** (`tag_search_mode`): STRING = `AND`
- **normalize_case** (`normalize_case`): STRING = `Yes`
- **param_show_tag_mismatch** (`param_show_tag_mismatch`): STRING = `All Usage`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **tag_group** (`tag_group`): STRING = `Tag Key-Value Pairs`

### SQL Query

```sql
SET ansi_mode = True; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' when :param_time_key = 'Quarter'
      then 'usage_quarter' when :param_time_key = 'Month'
      then 'usage_month' when :param_time_key = 'Week'
      then 'usage_week' when :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date as time_key, *
from
    list_priced_usd ),
 tag_entry_parsing AS ( select
    tag_entry, contains(tag_entry, '=') AS is_filter, IF(contains(tag_entry, '='),
 split(tag_entry, '=')[0], tag_entry) AS tag_key, ROW_NUMBER() OVER (ORDER BY tag_entry) AS tag_id
from
    ( select
    explode(split(:param_tag_entries, ';')) AS tag_entry
from
    VALUES(1) AS dummy(x) ) ),
 exploded_usage_tags_to_compare AS ( select
    record_id, explode(custom_tags),
 custom_tags
from
    usage_filtered AS u ),
 matched_records AS (select
    record_id, -- array of maps that match array_distinct(array_agg(case
      when tag_id IS NOT NULL
      then concat(spine.key, '=', spine.value) END)) AS matched_tags_array, array_distinct(array_agg(concat(spine.key, '=', spine.value))) AS usage_tags_array, -- Add keys only agg view array_distinct(array_agg(case
      when tag_id IS NOT NULL
      then spine.key END)) AS matched_keys_array, array_distinct(array_agg(spine.key)) AS usage_keys_array, -- size(matched_tags_array) AS matched_tag_count, (select
    COUNT (0)
from
    tag_entry_parsing
where
    length(tag_entry) > 0) AS search_tag_count, if(search_tag_count = 0 OR (case
      when :tag_search_mode = "AND"
      then search_tag_count <= matched_tag_count WHEN :tag_search_mode = "OR"
      then matched_tag_count > 0 END),
 -- Add key aggregation option case
      when :group_agg_mode = 'Key-Value Pairs'
      then array_join(array_distinct(matched_tags_array),
 ';') WHEN :group_agg_mode = 'Keys'
      then array_join(array_distinct(matched_keys_array),
 ';')
    end , '' )  as _custom_tag_key_value_pairs, if(_custom_tag_key_value_pairs = "", '&lt;NO TAG MATCH&gt;', _custom_tag_key_value_pairs) as custom_tag_key_value_pairs, -- Is match yes/no if (search_tag_count = 0 OR (case
      when :tag_search_mode = "AND"
      then search_tag_count <= matched_tag_count WHEN :tag_search_mode = "OR"
      then matched_tag_count > 0 END),
 '&lt;TAG MATCH&gt;', '&lt;NO TAG MATCH&gt;') AS IsMatched
from
    exploded_usage_tags_to_compare AS spine
  LEFT join
    tag_entry_parsing AS tt
    on (case
      when :normalize_case = 'Yes'
      then (-- Join
    on Key only if is_filter = false (trim(lower(tt.tag_key)) = trim(lower(spine.key)) AND tt.is_filter = false) OR (trim(lower(tt.tag_entry)) = concat(trim(lower(spine.key)),
 '=', trim(lower(spine.value))) AND tt.is_filter = true) )
      else (-- Join
    on Key only if is_filter = true (tt.tag_key = spine.key AND tt.is_filter = false) OR (tt.tag_entry = concat(spine.key, '=', spine.value) AND tt.is_filter = true) )
    end ) GROUP BY record_id ),
 -- match tag entries list_priced_usd_with_matching_tag_kvp as ( select
    mt._custom_tag_key_value_pairs, COALESCE(mt.custom_tag_key_value_pairs, '&lt;NOT TAGGED&gt;') AS custom_tag_key_value_pairs, -- There are matched tag records,
      then records with tags that dont match the ask,
      then totally untagged resources COALESCE(IsMatched, '&lt;NOT TAGGED&gt;') AS IsMatchedClean, mt.matched_tags_array AS kvp, k.*
from
    list_priced_usd_with_time_key k
  left join
    matched_records as mt
    on k.record_id = mt.record_id
where
    (case
      when :param_show_tag_mismatch = 'All Usage'
      then True WHEN :param_show_tag_mismatch = 'Not Matched Only'
      then COALESCE(mt.custom_tag_key_value_pairs, '&lt;NOT TAGGED&gt;') IN ('&lt;NO TAG MATCH&gt;', '&lt;NOT TAGGED&gt;') WHEN :param_show_tag_mismatch = 'Matched Only'
      then COALESCE(mt.custom_tag_key_value_pairs, '&lt;NOT TAGGED&gt;') NOT IN ('&lt;NO TAG MATCH&gt;', '&lt;NOT TAGGED&gt;') END) ),
 total_match_percent AS ( select
    sum(if(custom_tag_key_value_pairs NOT IN ('&lt;NO TAG MATCH&gt;', '&lt;NOT TAGGED&gt;'),
    -- Make dynamic case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd END, 0)) as match_usage_usd, sum(   -- Make dynamic case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd END) as total_usage_usd, round(try_divide(match_usage_usd , total_usage_usd),
 3) as tag_match_percentage
from
    list_priced_usd_with_matching_tag_kvp ),
 -- calc usage by period grouped_usage_by_period as ( select
    time_key as period_key, case
      when :tag_group = 'Workspace'
      then workspace WHEN :tag_group = 'Product'
      then billing_origin_product WHEN :tag_group = 'SKU'
      then sku_name WHEN :tag_group = 'Matched Status'
      then IsMatchedClean WHEN :tag_group = 'Tag Key-Value Pairs'
      then custom_tag_key_value_pairs WHEN :tag_group = 'Photon'
      then IsPhoton WHEN :tag_group = 'Serverless'
      then IsServerless
    end AS group_key, -- Make results aggregation optionally case sensitive sum(   -- Make dynamic case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd END) as usage_usd
from
    list_priced_usd_with_matching_tag_kvp group by all ),
 -- calc periodic change grouped_usage_change as ( select
    period_key, group_key, usage_usd, lag(usage_usd, 1) over (partition by group_key order by period_key) as prev_usage_usd, round(try_divide((usage_usd - prev_usage_usd),
 prev_usage_usd) * 100, 2) as usage_change_percentage
from
    grouped_usage_by_period ),
 total_usage_change as ( select
    period_key, '<b>TOTAL</b>' as group_key, sum(usage_usd) as usage_usd, lag(sum(usage_usd),
 1) over (order by period_key) as prev_usage_usd, round(try_divide((sum(usage_usd) - prev_usage_usd),
 prev_usage_usd) * 100, 2) as usage_change_percentage
from
    grouped_usage_by_period group by period_key ),
 -- periods period_info as ( select
    case
      when :param_time_key = 'Day'
      then :time_range.max::date when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date)
    end as current_period, case
      when :param_time_key = 'Day'
      then :time_range.max::date- interval 1 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 7 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 1 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 3 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 1 year)
    end as last_period, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 2 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 14 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 2 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 6 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 2 year)
    end as 2_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 3 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 21 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 3 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 9 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 3 year)
    end as 3_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 4 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 28 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 4 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 12 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 4 year)
    end as 4_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 5 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 35 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 5 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 15 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 5 year)
    end as 5_periods_ago ),
 -- pivot change usage_change_pivot as ( select
    case
      when period_key = current_period
      then 'Current period' when period_key = last_period
      then 'Last period' when period_key = 2_periods_ago
      then '2 periods ago' when period_key = 3_periods_ago
      then '3 periods ago' when period_key = 4_periods_ago
      then '4 periods ago' when period_key = 5_periods_ago
      then '5 periods ago'
    end as x_period_back, group_key, usage_usd, usage_change_percentage
from
    ( select
    *
from
    grouped_usage_change, period_info union all select
    *
from
    total_usage_change, period_info ) ),
 -- pivot all time all_time_usage_pivot as ( select
    'Start to
    end date' as x_period_back, group_key, sum(usage_usd) as usage_usd, null as usage_change_percentage
from
    grouped_usage_by_period group by group_key ),
 -- pivot total all time all_time_total_usage_pivot as ( select
    'Start to
    end date' as x_period_back, '<b>TOTAL</b>' as group_key, sum(usage_usd) as usage_usd, null as usage_change_percentage
from
    grouped_usage_by_period ),
 union_usage_pivot as ( select
    x_period_back, group_key, case
      when x_period_back = 'Start to
    end date'
      then string(usage_usd)
      else concat('<span style="zoom:1">', usage_usd_str, '</span><span style="zoom:1;color:', change_color, '">&nbsp;', usage_change_str, '</span>')
    end as usage_info
from
    ( select
    case
      when usage_usd >= 1e9
      then concat(format_number(usage_usd / 1e9, 0),
 'B') when usage_usd >= 1e6
      then concat(format_number(usage_usd / 1e6, 0),
 'M') when usage_usd >= 1e3
      then concat(format_number(usage_usd / 1e3, 0),
 'K')
      else format_number(usage_usd, 0)
    end as usage_usd_str, case
      when usage_change_percentage > 10
      then '#00A972' when usage_change_percentage < -10
      then '#FF3621'
      else '#919191'
    end as _change_color, concat('(', if(usage_change_percentage > 0, '+', ''),
 format_number(usage_change_percentage, 0),
 '%)') as _usage_change_str, coalesce(_change_color, '#919191') as change_color, coalesce(_usage_change_str, '') as usage_change_str, *
from
    ( select
    x_period_back, group_key, usage_usd, usage_change_percentage
from
    usage_change_pivot union all select
    *
from
    all_time_usage_pivot union all select
    *
from
    all_time_total_usage_pivot ) ) ),
 -- query results_pre_format AS ( select
    case
      when group_key IN ('&lt;NOT TAGGED&gt;', '&lt;NO TAG MATCH&gt;', '&lt;TAG MATCH&gt;', '<b>TOTAL</b>') OR :normalize_case = 'No'
      then group_key
      else lower(group_key)
    end AS group_key, case
      when :usage_toggle = 'Dollars'
      then concat('$', format_number(float(`Start to
    end date`),
 0)) WHEN :usage_toggle = 'DBUs'
      then concat('', format_number(float(`Start to
    end date`),
 0))
    end as `Start to
    end date`, float(`Start to
    end date`) as _all_time_usage_usd, 2 as _order, coalesce(`5 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `5 periods ago`, coalesce(`4 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `4 periods ago`, coalesce(`3 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `3 periods ago`, coalesce(`2 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `2 periods ago`, coalesce(`Last period`, '<span style="zoom:1;color:#919191">0</span>') as `Last period`, coalesce(`Current period`, '<span style="zoom:1;color:#919191">0</span>') as `Current period`
from
    union_usage_pivot pivot ( first(usage_info) for x_period_back in ( 'Start to
    end date', '5 periods ago', '4 periods ago', '3 periods ago', '2 periods ago', 'Last period', 'Current period' ) ) union all ( select
    CONCAT('  ', round((select
    MAX(tag_match_percentage)
from
    total_match_percent)*100, 2)::string,'% of usage matching tag search') as group_key, -- This is the formatted total matching percent across tags concat('<b><i>', date_format(:time_range.min, 'MMM dd yyyy'),
 ' - ', date_format(:time_range.max, 'MMM dd yyyy'),
 '</i></b>') as `Start to
    end date`, null as _all_time_usage_usd, 1 as _order, concat('<b><i>', date_format(5_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(4_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `5 periods ago`, concat('<b><i>', date_format(4_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(3_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `4 periods ago`, concat('<b><i>', date_format(3_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(2_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `3 periods ago`, concat('<b><i>', date_format(2_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(last_period, -1),
 'MMM dd'),
 '</i></b>') as `2 periods ago`, concat('<b><i>', date_format(last_period, 'MMM dd'),
 ' - ', date_format(date_add(current_period, -1),
 'MMM dd'),
 '</i></b>') as `Last period`, concat('<b><i>', date_format(current_period, 'MMM dd'),
 ' -
    end Time', '</i></b>') as `Current period`
from
    period_info ) ) select
    *, CONCAT( '<span style="color:', case
      when _order = 1
      then '#077A9D' WHEN group_key = '&lt;NOT TAGGED&gt;'
      then '#919191' WHEN group_key = '&lt;NO TAG MATCH&gt;'
      then '#AB4057' WHEN group_key = '&lt;TAG MATCH&gt;'
      then '#00A972'
      else 'inherit' END, case
      when group_key IN ('&lt;NOT TAGGED&gt;', '&lt;NO TAG MATCH&gt;', '&lt;TAG MATCH&gt;')
      then '; font-weight: bold' WHEN _order = 1
      then '; font-weight: bold; font-size: 24px'
      else '' END, ';">', group_key, '</span>' ) AS group_key_html, MAX(_all_time_usage_usd) OVER() AS max_usage, MIN(_all_time_usage_usd) OVER() AS min_usage, COALESCE(CONCAT( '<div style="position: relative; height: 20px; border-radius: 4px; overflow: hidden;">', '<div style="position: absolute; width: ', ROUND(try_divide((_all_time_usage_usd - min_usage),
 NULLIF(max_usage - min_usage, 0))* 100, 2),
 '%; height: 100%; background: #077A9D; border-radius: 4px;"></div>', '<span style="position: absolute; width: 100%; text-align: center; line-height: 20px; font-weight: bold; color: #00A972;">', case
      when :usage_toggle = 'Dollars'
      then '$'
      else '' END, FORMAT_NUMBER(_all_time_usage_usd, 0),
 '</span></div>' ),
 `Start to
    end date`) AS usage_bar_html
from
    results_pre_format order by _order asc, _all_time_usage_usd desc;
```

---

## Query 25: tag_analysis_match_details

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`
- **param_tag_entries** (`param_tag_entries`): STRING = `Budget;Env`
- **group_agg_mode** (`group_agg_mode`): STRING = `Key-Value Pairs`
- **normalize_case** (`normalize_case`): STRING = `Yes`
- **param_time_key** (`param_time_key`): STRING = `Month`
- **param_show_tag_mismatch** (`param_show_tag_mismatch`): STRING = `All Usage`
- **enable_links_toggle** (`enable_links_toggle`): STRING = `Enabled`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **top_n_asset** (`top_n_asset`): DECIMAL = `50`
- **tag_search_mode** (`tag_search_mode`): STRING = `AND`

### SQL Query

```sql
SET ansi_mode = True; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, workspace_name, workspace_url, u.*
from
    system.billing.usage as u
  inner join
    workspace
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) AND -- Serverless Filter (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, u.*, COALESCE( (select
    /*+ BROADCAST(ce) */ MAX(clean_room_name)
from
    system.access.clean_room_events ce
where
    ce.central_clean_room_id = usage_metadata.central_clean_room_id),
 (select
    /*+ BROADCAST(se) */ MAX(endpoint_name)
from
    system.serving.served_entities se
where
    se.workspace_id = workspace_id AND se.endpoint_id = usage_metadata.endpoint_id) ) AS asset_name
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    usage_date as time_key, *
from
    list_priced_usd ),
 tag_entry_parsing AS ( select
    tag_entry, contains(tag_entry, '=') AS is_filter, IF(contains(tag_entry, '='),
 split(tag_entry, '=')[0], tag_entry) AS tag_key, ROW_NUMBER() OVER (ORDER BY tag_entry) AS tag_id
from
    ( select
    explode(split(:param_tag_entries, ';')) AS tag_entry
from
    VALUES(1) AS dummy(x) ) ),
 exploded_usage_tags_to_compare AS ( select
    record_id, explode(custom_tags),
 custom_tags
from
    usage_filtered AS u ),
 matched_records AS (select
    record_id, -- array of maps that match array_distinct(array_agg(case
      when tag_id IS NOT NULL
      then concat(spine.key, '=', spine.value) END)) AS matched_tags_array, array_distinct(array_agg(concat(spine.key, '=', spine.value))) AS usage_tags_array, -- Add keys only agg view array_distinct(array_agg(case
      when tag_id IS NOT NULL
      then spine.key END)) AS matched_keys_array, array_distinct(array_agg(spine.key)) AS usage_keys_array, -- size(matched_tags_array) AS matched_tag_count, (select
    COUNT (0)
from
    tag_entry_parsing
where
    length(tag_entry) > 0) AS search_tag_count, if(search_tag_count = 0 OR (case
      when :tag_search_mode = "AND"
      then search_tag_count <= matched_tag_count WHEN :tag_search_mode = "OR"
      then matched_tag_count > 0 END),
 -- Add key aggregation option case
      when :group_agg_mode = 'Key-Value Pairs'
      then array_join(array_distinct(matched_tags_array),
 ';') WHEN :group_agg_mode = 'Keys'
      then array_join(array_distinct(matched_keys_array),
 ';')
    end , '' )  as _custom_tag_key_value_pairs, if(_custom_tag_key_value_pairs = "", '<NO TAG MATCH>', _custom_tag_key_value_pairs) as custom_tag_key_value_pairs, -- Is match yes/no if (search_tag_count = 0 OR (case
      when :tag_search_mode = "AND"
      then search_tag_count <= matched_tag_count WHEN :tag_search_mode = "OR"
      then matched_tag_count > 0 END),
 '<TAG MATCH>', '<NO TAG MATCH>') AS IsMatched
from
    exploded_usage_tags_to_compare AS spine
  LEFT join
    tag_entry_parsing AS tt
    on (case
      when :normalize_case = 'Yes'
      then (-- Join
    on Key only if is_filter = false (trim(lower(tt.tag_key)) = trim(lower(spine.key)) AND tt.is_filter = false) OR (trim(lower(tt.tag_entry)) = concat(trim(lower(spine.key)),
 '=', trim(lower(spine.value))) AND tt.is_filter = true) )
      else (-- Join
    on Key only if is_filter = true (tt.tag_key = spine.key AND tt.is_filter = false) OR (tt.tag_entry = concat(spine.key, '=', spine.value) AND tt.is_filter = true) )
    end ) GROUP BY record_id ),
 -- match tag entries list_priced_usd_with_matching_tag_kvp as ( select
    mt._custom_tag_key_value_pairs, COALESCE(IsMatched, '<NOT TAGGED>') AS IsMatched, COALESCE(mt.custom_tag_key_value_pairs, '<NOT TAGGED>') AS custom_tag_key_value_pairs, -- There are matched tag records,
      then records with tags that dont match the ask,
      then totally untagged resources mt.matched_tags_array AS kvp, k.*, usage_metadata.job_id AS job_id, usage_metadata.cluster_id AS cluster_id, usage_metadata.dlt_pipeline_id AS dlt_pipeline_id, usage_metadata.warehouse_id AS warehouse_id
from
    list_priced_usd_with_time_key k
  left join
    matched_records as mt
    on k.record_id = mt.record_id ),
 -- Dim tables to get names, deleted, and owner clusters AS ( select
    /*+ REPARTITION(2, cluster_id) */ workspace_id, cluster_id, cluster_name AS object_name, owned_by AS object_owner, case
      when delete_time IS NOT NULL
      then 1
      else 0
    end AS is_deleted
from
    system.compute.clusters QUALIFY ROW_NUMBER() OVER (PARTITION BY workspace_id, cluster_id ORDER BY change_time DESC) = 1 --CLUSTER BY cluster_id ),
 jobs AS ( select
    /*+ REPARTITION(2, job_id) */ workspace_id, job_id, name AS object_name, '' AS object_owner, case
      when delete_time IS NOT NULL
      then 1
      else 0
    end AS is_deleted
from
    system.lakeflow.jobs QUALIFY ROW_NUMBER() OVER (PARTITION BY workspace_id, job_id ORDER BY change_time DESC) = 1 --CLUSTER BY job_id ),
 pipelines AS ( select
    /*+ REPARTITION(2, pipeline_id) */ workspace_id, pipeline_id, name AS object_name, COALESCE(run_as, created_by) AS object_owner, case
      when delete_time IS NOT NULL
      then 1
      else 0
    end AS is_deleted
from
    system.lakeflow.pipelines QUALIFY ROW_NUMBER() OVER (PARTITION BY workspace_id, pipeline_id ORDER BY change_time DESC) = 1 -- CLUSTER BY pipeline_id ),
 warehouses AS ( select
    /*+ REPARTITION(2, warehouse_id) */ workspace_id, warehouse_id, warehouse_name AS object_name, '' AS object_owner, case
      when delete_time IS NOT NULL
      then 1
      else 0
    end AS is_deleted
from
    system.compute.warehouses QUALIFY ROW_NUMBER() OVER (PARTITION BY workspace_id, warehouse_id ORDER BY change_time DESC) = 1 --CLUSTER BY warehouse_id ),
 pre_results AS ( select
    /*+ BROADCAST(j),
 BROADCAST(c),
 BROADCAST(p),
 BROADCAST(w) */ coalesce(identity_metadata.run_as, identity_metadata.owned_by, identity_metadata.created_by, j.object_owner, c.object_owner, p.object_owner) AS asset_owner, case
      when coalesce(j.is_deleted,c.is_deleted, p.is_deleted, w.is_deleted) = 1
      then 'deleted' WHEN coalesce(j.is_deleted,c.is_deleted, p.is_deleted, w.is_deleted) = 0
      then 'active'
      else 'unknown'
    end AS is_deleted, coalesce(tk.asset_name, j.object_name,c.object_name, p.object_name, w.object_name, usage_metadata.endpoint_name, usage_metadata.notebook_path, 'unknown') AS clean_asset_name, -----==== Route to the object itself ====----- case
      when billing_origin_product = 'ALL_PURPOSE'
      then CONCAT(COALESCE(workspace_url, ''),
 '/compute/clusters/', usage_metadata.cluster_id) -- AP clusters WHEN billing_origin_product = 'INTERACTIVE'
      then CONCAT(COALESCE(workspace_url, ''),
 '/editor/notebooks/', usage_metadata.notebook_id) -- Serverless All purpose usage WHEN billing_origin_product = 'JOBS'
      then CONCAT(COALESCE(workspace_url, ''),
 '/jobs/', usage_metadata.job_id) -- works for regular and serverless WHEN billing_origin_product = 'SQL'
      then case
      when usage_metadata.warehouse_id IS NOT NULL
      then CONCAT(COALESCE(workspace_url, ''),
 '/sql/warehouses/', usage_metadata.warehouse_id, '?o=', tk.workspace_id)
      else CONCAT(COALESCE(workspace_url, ''),
 '/pipelines/', usage_metadata.dlt_pipeline_id, '?o=', tk.workspace_id)
    end -- Can be SQL Warehouse or DLT Serverless
from
    ST/MVs WHEN billing_origin_product = 'DLT'
      then CONCAT(COALESCE(workspace_url, ''),
 '/pipelines/', usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'ONLINE_TABLES'
      then CONCAT(COALESCE(workspace_url, ''),
 '/pipelines/', usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'APPS'
      then CONCAT(COALESCE(workspace_url, ''),
 '/apps/', usage_metadata.app_name) WHEN billing_origin_product = 'NOTEBOOKS'
      then CONCAT(COALESCE(workspace_url, ''),
 '/editor/notebooks/', usage_metadata.notebook_id) -- There is a separate SKU for notebooks for some reason WHEN billing_origin_product = 'PREDICTIVE_OPTIMIZATION'
      then NULL -- background service WHEN billing_origin_product = 'CLEAN_ROOM'
      then case
      when asset_name IS NULL
      then NULL
      else CONCAT(COALESCE(workspace_url, ''),
 '/explore/cleanrooms/', COALESCE(asset_name, usage_metadata.central_clean_room_id),
 '?o=', COALESCE(tk.workspace_id, ''))
    end WHEN billing_origin_product = 'LAKEFLOW_CONNECT'
      then CONCAT('/pipelines/', usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'AI_RUNTIME'
      then NULL -- NOT SUPPORTED WHEN billing_origin_product = 'MODEL_SERVING'
      then (case
      when usage_metadata.endpoint_name IS NOT NULL -- Model Serving control panel is by name
      then CONCAT(COALESCE(workspace_url, ''),
 '/ml/endpoints/', usage_metadata.endpoint_name) WHEN usage_metadata.endpoint_id IS NOT NULL
      then NULL --  TO DO - find way to link with id WHEN custom_tags.EndpointId IS NOT NULL
      then NULL --  TO DO - find way to link with id WHEN usage_metadata.cluster_id IS NOT NULL
      then NULL --  TO DO - find way to link with id END) WHEN billing_origin_product = 'VECTOR_SEARCH'
      then CONCAT(COALESCE(workspace_url, ''),
 CONCAT('/compute/vector-search/', usage_metadata.endpoint_name),
 '?o=', COALESCE(tk.workspace_id, '')) WHEN billing_origin_product = 'DATABASE'
      then NULL -- TBD -- WHEN billing_origin_product = 'LAKEHOUSE_MONITORING'
      then custom_tags.LakehouseMonitoringTableId -- TO DO - Needs better link identifier and more than just table Id to link to monitor
    end AS asset_url_router, -----==== Asset Identifier (id or name if no id) ====----- case
      when billing_origin_product = 'ALL_PURPOSE'
      then usage_metadata.cluster_id -- AP clusters WHEN billing_origin_product = 'INTERACTIVE'
      then case
      when usage_metadata.notebook_id IS NOT NULL
      then usage_metadata.notebook_id WHEN product_features.is_serverless = True
      then asset_owner
    end -- Serverless All purpose usage WHEN billing_origin_product = 'NETWORKING'
      then CONCAT(usage_metadata.source_region, ' -> ',usage_metadata.destination_region) WHEN billing_origin_product = 'FINE_GRAINED_ACCESS_CONTROL'
      then case
      when product_features.is_serverless = True
      then asset_owner
    end WHEN billing_origin_product = 'AGENT_EVALUATION'
      then asset_owner -- user doing agent eval WHEN billing_origin_product = 'JOBS'
      then usage_metadata.job_id -- works for regular and serverless WHEN billing_origin_product = 'SQL'
      then case
      when usage_metadata.warehouse_id IS NOT NULL
      then usage_metadata.warehouse_id
      else usage_metadata.dlt_pipeline_id
    end -- Can be SQL Warehouse or DLT Serverless
from
    ST/MVs WHEN billing_origin_product = 'DLT'
      then case
      when usage_metadata.dlt_pipeline_id IS NOT NULL
      then usage_metadata.dlt_pipeline_id WHEN usage_metadata.cluster_id IS NOT NULL
      then usage_metadata.cluster_id
    end -- Can be pipeline id or cluster id for classic DLT WHEN billing_origin_product = 'ONLINE_TABLES'
      then usage_metadata.dlt_pipeline_id WHEN billing_origin_product = 'APPS'
      then usage_metadata.app_name WHEN billing_origin_product = 'CLEAN_ROOM'
      then usage_metadata.central_clean_room_id WHEN billing_origin_product = 'NOTEBOOKS'
      then usage_metadata.notebook_id -- There is a separate SKU for notebooks for some reason WHEN billing_origin_product = 'PREDICTIVE_OPTIMIZATION'
      then 'Background Service' -- background service WHEN billing_origin_product = 'LAKEFLOW_CONNECT'
      then usage_metadata.dlt_pipeline_id WHEN billing_origin_product = 'AI_RUNTIME'
      then usage_metadata.ai_runtime_pool_id WHEN billing_origin_product = 'MODEL_SERVING'
      then (case
      when usage_metadata.endpoint_name IS NOT NULL -- Model Serving control panel is by name
      then usage_metadata.endpoint_name WHEN usage_metadata.endpoint_id IS NOT NULL
      then usage_metadata.endpoint_id WHEN custom_tags.EndpointId IS NOT NULL
      then custom_tags.EndpointId WHEN usage_metadata.cluster_id IS NOT NULL
      then usage_metadata.cluster_id END) WHEN billing_origin_product = 'VECTOR_SEARCH'
      then COALESCE(usage_metadata.endpoint_name, usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'FOUNDATION_MODEL_TRAINING'
      then usage_metadata.run_name -- TO DO - Needs better link identifier and experiemnt Id and user id WHEN billing_origin_product = 'LAKEHOUSE_MONITORING'
      then custom_tags.LakehouseMonitoringTableId -- TO DO - Needs better link identifier and more than just table Id to link to monitor WHEN billing_origin_product = 'DATABASE'
      then usage_metadata.database_instance_id -- TBD
    end AS asset_id, -----==== Type of Asset (id/name description) ====----- case
      when billing_origin_product = 'ALL_PURPOSE'
      then 'All Purpose Cluster Id' -- AP clusters WHEN billing_origin_product = 'INTERACTIVE'
      then case
      when usage_metadata.notebook_id IS NOT NULL
      then 'Serverless Notebook Id' WHEN product_features.is_serverless = True
      then 'Serverless Interactive Usage'
    end WHEN billing_origin_product = 'NETWORKING'
      then 'Network Route' WHEN billing_origin_product = 'FINE_GRAINED_ACCESS_CONTROL'
      then case
      when product_features.is_serverless = True
      then 'Fine Grained Access Filtering'
    end WHEN billing_origin_product = 'AGENT_EVALUATION'
      then 'User Agent Evaluation' -- user doing agent eval WHEN billing_origin_product = 'JOBS'
      then 'Workflow - Job Id' -- works for regular and serverless WHEN billing_origin_product = 'SQL'
      then case
      when usage_metadata.warehouse_id IS NOT NULL
      then 'SQL Warehouse'
      else 'SQL - DLT ST/MVs'
    end -- Can be SQL Warehouse or DLT Serverless
from
    ST/MVs WHEN billing_origin_product = 'DLT'
      then case
      when usage_metadata.dlt_pipeline_id IS NOT NULL
      then 'DLT Pipeline Id' WHEN usage_metadata.cluster_id IS NOT NULL
      then 'DLT Cluster Id'
    end -- Can be pipeline id or cluster id for classic DLT WHEN billing_origin_product = 'ONLINE_TABLES'
      then 'Online Table DLT Pipeline Id' -- 'Not yet recorded' WHEN billing_origin_product = 'APPS'
      then 'Lakehouse App Name' WHEN billing_origin_product = 'CLEAN_ROOM'
      then 'Cleanroom Id / Name' WHEN billing_origin_product = 'LAKEFLOW_CONNECT'
      then 'Lakeflow Pipeline Id' WHEN billing_origin_product = 'AI_RUNTIME'
      then 'AI Runtime Pool Id' WHEN billing_origin_product = 'NOTEBOOKS'
      then 'Serverless Notebook Id'-- There is a separate SKU for notebooks for some reason WHEN billing_origin_product = 'PREDICTIVE_OPTIMIZATION'
      then 'Predictive Optimization Usage' -- background service WHEN billing_origin_product = 'MODEL_SERVING'
      then (case
      when usage_metadata.endpoint_name IS NOT NULL -- Model Serving control panel is by name
      then 'Model Serving Endpoint Name' WHEN usage_metadata.endpoint_id IS NOT NULL
      then 'Model Serving Endpoint Id' WHEN custom_tags.EndpointId IS NOT NULL
      then 'Model Serving Endpoint Id' WHEN usage_metadata.cluster_id IS NOT NULL
      then 'Model Serving Cluster Id' END) WHEN billing_origin_product = 'VECTOR_SEARCH'
      then 'Vector Search Endpoint Name' -- TO DO - Needs better link identifier and owner info - Need Vector search name WHEN billing_origin_product = 'FOUNDATION_MODEL_TRAINING'
      then 'Training Experiment Run'-- TO DO - Needs better link identifier and experiemnt Id and user id WHEN billing_origin_product = 'LAKEHOUSE_MONITORING'
      then 'Table Id Monitor' -- TO DO - Needs better link identifier and more than just table Id to link to monitor WHEN billing_origin_product = 'DATABASE'
      then 'Database Instance Id' -- TO DO - Needs better link identifier via database name
    end AS asset_type, case
      when asset_id IS NOT NULL
      then 'Identified'
      else 'Unknown Asset Id'
    end AS IsClassifiedIntoAsset, case
      when :param_time_key = 'Day'
      then :time_range.max when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max)
    end as current_period, case
      when :param_time_key = 'Day'
      then :time_range.max - interval 1 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max - interval 7 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max - interval 1 month)
    end as last_period, tk.*
from
    (select
    *
from
    list_priced_usd_with_matching_tag_kvp
where
    (case
      when :param_show_tag_mismatch = 'All Usage'
      then True WHEN :param_show_tag_mismatch = 'Not Matched Only'
      then IsMatched IN ('<NO TAG MATCH>', '<NOT TAGGED>') WHEN :param_show_tag_mismatch = 'Matched Only'
      then IsMatched NOT IN ('<NO TAG MATCH>', '<NOT TAGGED>') END)) AS tk
  LEFT join
    jobs j
    on j.job_id = tk.job_id
  LEFT join
    clusters c
    on c.cluster_id = tk.cluster_id
  LEFT join
    pipelines p
    on p.pipeline_id = tk.dlt_pipeline_id
  LEFT join
    warehouses w
    on w.warehouse_id = tk.warehouse_id ),
 agg_final AS ( -- Aggregate select
    workspace AS workspace_id, billing_origin_product, asset_type, asset_id, case
      when :enable_links_toggle = 'Enabled'
      then asset_url_router
      else NULL
    end AS asset_url_router, IsClassifiedIntoAsset, IsServerless, usage_type, COALESCE(MAX(is_deleted),
 'unknown') AS is_object_deleted, FIRST(current_period) AS current_period, FIRST(last_period) AS last_period, SUM(   -- Make dynamic case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end ) FILTER(WHERE usage_date >= current_period) as current_period_usage_usd, SUM(case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end ) FILTER(WHERE usage_date BETWEEN last_period AND current_period) as last_period_usage_usd, MAX_BY(COALESCE(asset_owner),
 usage_end_time) AS usage_owner, -- Most recent asset name (they can change) MAX_BY(clean_asset_name, usage_end_time) AS MostRecentAssetName, -- Most recent is matched MAX_BY(IsMatched, usage_end_time) AS MostRecentIsMatched, -- Most recent tag pairs MAX_BY(custom_tag_key_value_pairs, usage_end_time) AS MostRecentTagKeyValuePairs, -- Most recent usage date MAX_BY(usage_date, usage_end_time) AS MostRecentUsageDate, -- Days since most recent usage date_diff(current_date()::date, MostRecentUsageDate) AS DaysSinceMostRecentUsage, -- First Usage Date In Window MIN_BY(usage_date, usage_end_time) AS EarliestUsageDate, -- Total Usage SUM(case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end ) AS TotalDollarsUsage, SUM(usage_quantity) AS TotalDBUUsageQuantity, -- Most recent custom_tags value MAX_BY(custom_tags, usage_end_time) AS MostRecentTagsOnAsset, -- Most recent usage metadata MAX_BY(usage_metadata, usage_end_time) AS MostRecentUsageMetadata, MAX_BY(identity_metadata, usage_end_time) AS MostRecentIdentityMetadata, MAX_BY(product_features, usage_end_time) AS MostRecentProductFeatures
from
    pre_results GROUP BY workspace, billing_origin_product, asset_type, asset_id, case
      when :enable_links_toggle = 'Enabled'
      then asset_url_router
      else NULL END, IsClassifiedIntoAsset, IsServerless, usage_type ),
 asset_ranks AS ( select
    CONCAT( '<span style="color:', case
      when MostRecentIsMatched = '<NOT TAGGED>'
      then '#919191' WHEN MostRecentIsMatched = '<NO TAG MATCH>'
      then '#AB4057' WHEN MostRecentIsMatched = '<TAG MATCH>'
      then '#00A972'
      else 'inherit' END, '; font-weight: bold', ';">', REPLACE(REPLACE(MostRecentIsMatched, '<', '&lt;'),
 '>', '&gt;'),
 '</span>' ) AS MatchedKey_html, case
      when MostRecentIsMatched = '<NOT TAGGED>'
      then 'NOT TAGGED' WHEN MostRecentIsMatched = '<NO TAG MATCH>'
      then 'NO TAG MATCH' WHEN MostRecentIsMatched = '<TAG MATCH>'
      then 'MATCHED'
    end AS clean_ui_is_matched, af.*, -- PoP usage change case
      when last_period_usage_usd IS NULL
      then 0
      else ROUND((try_divide((current_period_usage_usd - last_period_usage_usd),
 last_period_usage_usd))*100, 2)
    end AS percent_pop_change, -- add data bars
    on USAGE MAX(TotalDollarsUsage) OVER() AS max_usage_dollars, MIN(TotalDollarsUsage) OVER() AS min_usage_dollars, COALESCE(CONCAT( '<div style="position: relative; height: 20px; border-radius: 4px; overflow: hidden;">', '<div style="position: absolute; width: ', ROUND(try_divide((TotalDollarsUsage - min_usage_dollars),
 NULLIF(max_usage_dollars - min_usage_dollars, 0)) * 100, 2),
 '%; height: 100%; background: #077A9D; border-radius: 4px;"></div>', '<span style="position: absolute; width: 100%; text-align: center; line-height: 20px; font-weight: bold; color: #00A972;">', case
      when :usage_toggle = 'Dollars'
      then '$'
      else '' END, FORMAT_NUMBER(TotalDollarsUsage, 0),
 '</span></div>' ),
 CAST(TotalDollarsUsage AS STRING)) AS usage_dollars_html, MAX(DaysSinceMostRecentUsage) OVER() AS max_usage_days, MIN(DaysSinceMostRecentUsage) OVER() AS min_usage_days, CONCAT( '<span style="display: block; width: 100%; padding: 5px; color: white; background-color: rgba(', -- Interpolate RGB values for the gradient
from
    green to red ROUND( 171 * ((DaysSinceMostRecentUsage - min_usage_days) / NULLIF(max_usage_days - min_usage_days, 0)) + 0 * (1 - ((DaysSinceMostRecentUsage - min_usage_days) / NULLIF(max_usage_days - min_usage_days, 0))) ),
 ',', -- Red channel ROUND( 64 * ((DaysSinceMostRecentUsage - min_usage_days) / NULLIF(max_usage_days - min_usage_days, 0)) + 169 * (1 - ((DaysSinceMostRecentUsage - min_usage_days) / NULLIF(max_usage_days - min_usage_days, 0))) ),
 ',', -- Green channel ROUND( 87 * ((DaysSinceMostRecentUsage - min_usage_days) / NULLIF(max_usage_days - min_usage_days, 0)) + 114 * (1 - ((DaysSinceMostRecentUsage - min_usage_days) / NULLIF(max_usage_days - min_usage_days, 0))) ),
 ',', -- Blue channel '0.8); border-radius: 4px;">', ROUND(DaysSinceMostRecentUsage, 1),
 ' days</span>' ) AS heat_color_html, ROW_NUMBER() OVER(ORDER BY TotalDollarsUsage DESC) AS top_n_asset
from
    agg_final af ) select
    *
from
    asset_ranks
where
    top_n_asset <= :top_n_asset ORDER BY TotalDollarsUsage DESC
```

---

## Query 26: top_spending_assets_summary

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **param_time_key** (`param_time_key`): STRING = `Month`
- **param_rank_key** (`param_rank_key`): STRING = `job_id`
- **param_top_n** (`param_top_n`): INTEGER = `10`
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **include_remaining_assets** (`include_remaining_assets`): STRING = `Yes`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`

### SQL Query

```sql
SET ansi_mode = True; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) -- Serverless AND (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    identifier ( case
      when :param_time_key = 'Year'
      then 'usage_year' when :param_time_key = 'Quarter'
      then 'usage_quarter' when :param_time_key = 'Month'
      then 'usage_month' when :param_time_key = 'Week'
      then 'usage_week' when :param_time_key = 'Day'
      then 'usage_date'
      else 'usage_date'
    end )::date as time_key, *
from
    list_priced_usd ),
 -- eval rank_key param list_priced_usd_with_rank_key as ( select
    struct(workspace) as rank_metadata, identifier ( case
      when :param_rank_key = 'workspace'
      then 'rank_metadata' when :param_rank_key = 'run_as'
      then 'identity_metadata'
      else 'usage_metadata'
    end )[:param_rank_key] as rank_key, *
from
    list_priced_usd_with_time_key ),
 -- calc top ranked usage top_ranked_usage as ( select
    rank_key, true as is_top_ranked, sum( case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end ) as usage_usd
from
    list_priced_usd_with_rank_key
where
    rank_key is not null group by rank_key order by usage_usd desc limit :param_top_n ),
 list_priced_usd_with_rank_key_normalized as ( select
    if(is_top_ranked or u.rank_key is null, u.rank_key, '<REMAINING OBJECTS>') as rank_key, u.time_key, u.workspace, case
      when :usage_toggle = 'Dollars'
      then u.usage_usd WHEN :usage_toggle = 'DBUs'
      then u.usage_dbus
      else u.usage_usd
    end AS usage_usd
from
    list_priced_usd_with_rank_key as u
  left join
    top_ranked_usage
    on u.rank_key = top_ranked_usage.rank_key
where
    case
      when :include_remaining_assets = 'Yes'
      then u.rank_key is not null
      else top_ranked_usage.rank_key IS NOT NULL
    end ) -- query select
    COALESCE(rank_key, '<OTHER OBJECT TYPES>') AS rank_key, time_key, workspace, sum(usage_usd) as usage_usd
from
    list_priced_usd_with_rank_key_normalized group by all
```

---

## Query 27: top_spending_assets_pop_matrix

### Parameters

- **param_workspace** (`param_workspace`): STRING = `all` [MULTI]
- **param_time_key** (`param_time_key`): STRING = `Month`
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **time_range** (`time_range`): DATE = `now-12M/M to now-1M/M` [RANGE]
- **product_category** (`product_category`): STRING = `all` [MULTI]
- **param_rank_key** (`param_rank_key`): STRING = `warehouse_id`
- **usage_toggle** (`usage_toggle`): STRING = `Dollars`
- **param_top_n** (`param_top_n`): INTEGER = `10`
- **include_remaining_assets** (`include_remaining_assets`): STRING = `Yes`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`
- **is_serverless** (`is_serverless`): STRING = `all` [MULTI]
- **enable_links_toggle** (`enable_links_toggle`): STRING = `Enabled`

### SQL Query

```sql
SET ansi_mode = True; with
  workspace as (
     select
    account_id, workspace_id, workspace_name, workspace_url, status, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')') AS workspace_full_name
from
    system.access.workspaces_latest
where
    (array_contains(:param_workspace, concat(COALESCE(workspace_name, ''),
 ' (id: ', workspace_id, ')')) OR array_contains(:param_workspace,'all')) ),
 -- apply date filter usage_with_ws_filtered_by_date as ( select
    case
      when workspace_name is null
      then concat('id: ', u.workspace_id)
      else workspace_full_name
    end as workspace, workspace_url, u.*
from
    system.billing.usage as u
  inner join
    workspace -- Must assume this works to filter
    on u.workspace_id = workspace.workspace_id
where
    u.usage_date between :time_range.min and :time_range.max ),
 -- apply workspace filter usage_filtered as ( select
    *, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic'
    end AS IsServerless, case
      when product_features.is_photon = True
      then 'Photon'
      else 'Spark'
    end AS IsPhoton
from
    usage_with_ws_filtered_by_date
where
    -- Product SKU filter (array_contains(:product_category, billing_origin_product) OR array_contains(:product_category,'all')) -- Serverless AND (array_contains(:is_serverless, case
      when product_features.is_serverless = True
      then 'Serverless'
      else 'Classic' END) OR array_contains(:is_serverless,'all')) ),
 parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, sku_name, usage_unit, price_start_time, case
      when :price_table = 'system.billing.list_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.effective_list.default', 'decimal(38,18)') WHEN :price_table = 'system.billing.account_prices'
      then try_variant_get(to_variant_object(pricing),
 '$.default', 'decimal(38,18)')
    end AS unit_px
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 list_priced_usd as ( select
    /*+ BROADCAST(p) */ -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.unit_px * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.unit_px * u.usage_quantity, -- When all
      else fails, use default p.unit_px * u.usage_quantity) as usage_usd, usage_quantity AS usage_dbus, date_trunc('YEAR', usage_date) as usage_year, date_trunc('QUARTER', usage_date) as usage_quarter, date_trunc('MONTH', usage_date) as usage_month, date_trunc('WEEK', usage_date) as usage_week, MIN(usage_date) OVER () AS start_time, MAX(usage_date) OVER () AS end_time, u.*
from
    usage_filtered as u
  LEFT join
    parsed_discounts_table AS discounts
    on (discounts.product = u.sku_name OR discounts.product = u.billing_origin_product)
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 -- Some links require the name, this is expensive so use sparingly asset_names AS ( select
    usage_metadata.central_clean_room_id AS asset_id, MAX_BY(clean_room_name, ce.event_time) AS asset_name
from
    list_priced_usd s
  LEFT join
    system.access.clean_room_events ce
    on ce.central_clean_room_id = s.usage_metadata.central_clean_room_id
where
    billing_origin_product = 'CLEAN_ROOM' GROUP BY usage_metadata.central_clean_room_id ),
 -- eval time_key param list_priced_usd_with_time_key as ( select
    (case
      when :param_time_key = 'Year'
      then usage_year when :param_time_key = 'Quarter'
      then usage_quarter when :param_time_key = 'Month'
      then usage_month when :param_time_key = 'Week'
      then usage_week when :param_time_key = 'Day'
      then usage_date
      else usage_date
    end )::date as time_key, *
from
    list_priced_usd ),
 -- eval rank_key param list_priced_usd_with_rank_key as ( select
    struct(workspace) as rank_metadata, identifier ( case
      when :param_rank_key = 'workspace'
      then 'rank_metadata' when :param_rank_key = 'run_as'
      then 'identity_metadata'
      else 'usage_metadata'
    end )[:param_rank_key] as rank_key, *
from
    list_priced_usd_with_time_key ),
 -- calc top ranked usage top_ranked_usage as ( select
    rank_key, true as is_top_ranked, sum( case
      when :usage_toggle = 'Dollars'
      then usage_usd WHEN :usage_toggle = 'DBUs'
      then usage_dbus
      else usage_usd
    end ) as usage_usd
from
    list_priced_usd_with_rank_key
where
    rank_key is not null group by rank_key order by usage_usd desc limit :param_top_n ),
 list_priced_usd_with_rank_key_normalized as ( select
    /*+ BROADCAST(aa),
 BROADCAST(top_ranked_usage) */ if(is_top_ranked or u.rank_key is null, u.rank_key, 'REMAINING OBJECTS') as rank_key, u.time_key, case
      when :usage_toggle = 'Dollars'
      then u.usage_usd WHEN :usage_toggle = 'DBUs'
      then u.usage_dbus
      else u.usage_usd
    end AS usage_usd, case
      when u.rank_key != 'REMAINING OBJECTS'
      then coalesce(identity_metadata.run_as, identity_metadata.owned_by, identity_metadata.created_by)
    end AS usage_owner, -----==== Route to the asset itself ====----- case
      when u.rank_key != 'REMAINING OBJECTS'
      then case
      when :param_rank_key = 'workspace'
      then workspace_url WHEN :param_rank_key = 'run_as'
      then NULL WHEN billing_origin_product = 'ALL_PURPOSE'
      then CONCAT(COALESCE(workspace_url, ''),
 '/compute/clusters/', usage_metadata.cluster_id) -- AP clusters WHEN billing_origin_product = 'INTERACTIVE'
      then CONCAT(COALESCE(workspace_url, ''),
 '/editor/notebooks/', usage_metadata.notebook_id) -- Serverless All purpose usage WHEN billing_origin_product = 'JOBS'
      then CONCAT(COALESCE(workspace_url, ''),
 '/jobs/', usage_metadata.job_id) -- works for regular and serverless WHEN billing_origin_product = 'SQL'
      then case
      when usage_metadata.warehouse_id IS NOT NULL
      then CONCAT(COALESCE(workspace_url, ''),
 '/sql/warehouses/', usage_metadata.warehouse_id, '?o=', workspace_id)
      else CONCAT(COALESCE(workspace_url, ''),
 '/pipelines/', usage_metadata.dlt_pipeline_id, '?o=', workspace_id)
    end -- Can be SQL Warehouse or DLT Serverless
from
    ST/MVs WHEN billing_origin_product = 'DLT'
      then CONCAT(COALESCE(workspace_url, ''),
 '/pipelines/', usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'ONLINE_TABLES'
      then CONCAT(COALESCE(workspace_url, ''),
 '/pipelines/', usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'APPS'
      then CONCAT(COALESCE(workspace_url, ''),
 '/apps/', usage_metadata.app_name) WHEN billing_origin_product = 'NOTEBOOKS'
      then CONCAT(COALESCE(workspace_url, ''),
 '/editor/notebooks/', usage_metadata.notebook_id) -- There is a separate SKU for notebooks for some reason WHEN billing_origin_product = 'PREDICTIVE_OPTIMIZATION'
      then NULL -- background service WHEN billing_origin_product = 'CLEAN_ROOM'
      then case
      when aa.asset_name IS NULL
      then NULL
      else CONCAT(COALESCE(workspace_url, ''),
 '/explore/cleanrooms/', COALESCE(aa.asset_name, usage_metadata.central_clean_room_id),
 '?o=', COALESCE(workspace_id, ''))
    end WHEN billing_origin_product = 'LAKEFLOW_CONNECT'
      then CONCAT('/pipelines/', usage_metadata.dlt_pipeline_id) WHEN billing_origin_product = 'AI_RUNTIME'
      then NULL -- NOT SUPPORTED WHEN billing_origin_product = 'MODEL_SERVING'
      then (case
      when usage_metadata.endpoint_name IS NOT NULL -- Model Serving control panel is by name
      then CONCAT(COALESCE(workspace_url, ''),
 '/ml/endpoints/', usage_metadata.endpoint_name) WHEN usage_metadata.endpoint_id IS NOT NULL
      then NULL --  TO DO - find way to link with id WHEN custom_tags.EndpointId IS NOT NULL
      then NULL --  TO DO - find way to link with id WHEN usage_metadata.cluster_id IS NOT NULL
      then NULL --  TO DO - find way to link with id END) WHEN billing_origin_product = 'VECTOR_SEARCH'
      then CONCAT(COALESCE(workspace_url, ''),
 CONCAT('/compute/vector-search/', usage_metadata.endpoint_name),
 '?o=', COALESCE(workspace_id, '')) -- WHEN billing_origin_product = 'LAKEHOUSE_MONITORING'
      then custom_tags.LakehouseMonitoringTableId -- TO DO - Needs better link identifier and more than just table Id to link to monitor WHEN billing_origin_product = 'DATABASE'
      then NULL -- TO DO
    end END AS asset_url_router
from
    list_priced_usd_with_rank_key as u
  left join
    top_ranked_usage
    on u.rank_key = top_ranked_usage.rank_key
  left join
    asset_names aa
    on u.usage_metadata.central_clean_room_id = aa.asset_id
where
    case
      when :include_remaining_assets = 'Yes'
      then u.rank_key is not null
      else top_ranked_usage.rank_key IS NOT NULL
    end ),
 group_key_routes AS ( select
    rank_key, MAX_BY(asset_url_router, time_key) AS latest_asset_route, MAX_BY(usage_owner, time_key) AS latest_asset_owner
from
    list_priced_usd_with_rank_key_normalized
where
    rank_key != 'REMAINING OBJECTS' GROUP BY rank_key ),
 -- calc usage by period grouped_usage_by_period as ( select
    time_key as period_key, rank_key as group_key, sum(usage_usd) as usage_usd
from
    list_priced_usd_with_rank_key_normalized group by time_key, rank_key ),
 -- calc periodic change grouped_usage_change as ( select
    period_key, group_key, usage_usd, lag(usage_usd, 1) over (partition by group_key order by period_key) as prev_usage_usd, round(try_divide((usage_usd - prev_usage_usd),
 prev_usage_usd) * 100, 2) as usage_change_percentage
from
    grouped_usage_by_period ),
 total_usage_change as ( select
    period_key, 'TOTAL' as group_key, sum(usage_usd) as usage_usd, lag(sum(usage_usd),
 1) over (order by period_key) as prev_usage_usd, round(try_divide((sum(usage_usd) - prev_usage_usd),
 prev_usage_usd) * 100, 2) as usage_change_percentage
from
    grouped_usage_by_period group by period_key ),
 -- periods period_info as ( select
    case
      when :param_time_key = 'Day'
      then :time_range.max::date when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date)
    end as current_period, case
      when :param_time_key = 'Day'
      then :time_range.max::date- interval 1 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 7 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 1 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 3 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 1 year)
    end as last_period, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 2 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 14 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 2 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 6 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 2 year)
    end as 2_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 3 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 21 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 3 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 9 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 3 year)
    end as 3_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 4 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 28 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 4 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 12 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 4 year)
    end as 4_periods_ago, case
      when :param_time_key = 'Day'
      then :time_range.max::date - interval 5 day when :param_time_key = 'Week'
      then date_trunc('WEEK', :time_range.max::date - interval 35 day) when :param_time_key = 'Month'
      then date_trunc('MONTH', :time_range.max::date - interval 5 month) when :param_time_key = 'Quarter'
      then date_trunc('QUARTER', :time_range.max::date - interval 15 month) when :param_time_key = 'Year'
      then date_trunc('YEAR', :time_range.max::date - interval 5 year)
    end as 5_periods_ago ),
 -- pivot change usage_change_pivot as ( select
    case
      when period_key = current_period
      then 'Current period' when period_key = last_period
      then 'Last period' when period_key = 2_periods_ago
      then '2 periods ago' when period_key = 3_periods_ago
      then '3 periods ago' when period_key = 4_periods_ago
      then '4 periods ago' when period_key = 5_periods_ago
      then '5 periods ago'
    end as x_period_back, group_key, usage_usd, usage_change_percentage
from
    ( select
    *
from
    grouped_usage_change, period_info union all select
    *
from
    total_usage_change, period_info ) ),
 -- pivot all time all_time_usage_pivot as ( select
    'Start to
    end date' as x_period_back, group_key, sum(usage_usd) as usage_usd, null as usage_change_percentage
from
    grouped_usage_by_period group by group_key ),
 -- pivot total all time all_time_total_usage_pivot as ( select
    'Start to
    end date' as x_period_back, 'TOTAL' as group_key, sum(usage_usd) as usage_usd, null as usage_change_percentage
from
    grouped_usage_by_period ),
 union_usage_pivot as ( select
    x_period_back, group_key, case
      when x_period_back = 'Start to
    end date'
      then string(usage_usd)
      else concat('<span style="zoom:1">', usage_usd_str, '</span><span style="zoom:1;color:', change_color, '">&nbsp;', usage_change_str, '</span>')
    end as usage_info
from
    ( select
    case
      when usage_usd >= 1e9
      then concat(format_number(usage_usd / 1e9, 0),
 'B') when usage_usd >= 1e6
      then concat(format_number(usage_usd / 1e6, 0),
 'M') when usage_usd >= 1e3
      then concat(format_number(usage_usd / 1e3, 0),
 'K')
      else format_number(usage_usd, 0)
    end as usage_usd_str, case
      when usage_change_percentage > 10
      then '#00A972' when usage_change_percentage < -10
      then '#FF3621'
      else '#919191'
    end as _change_color, concat('(', if(usage_change_percentage > 0, '+', ''),
 format_number(usage_change_percentage, 0),
 '%)') as _usage_change_str, coalesce(_change_color, '#919191') as change_color, coalesce(_usage_change_str, '') as usage_change_str, *
from
    ( select
    x_period_back, group_key, usage_usd, usage_change_percentage
from
    usage_change_pivot union all select
    *
from
    all_time_usage_pivot union all select
    *
from
    all_time_total_usage_pivot ) ) ),
 pre_format_results AS ( -- query select
    group_key, case
      when :usage_toggle = 'Dollars'
      then concat('$', format_number(float(`Start to
    end date`),
 0)) WHEN :usage_toggle = 'DBUs'
      then concat('', format_number(float(`Start to
    end date`),
 0))
      else concat('$', format_number(float(`Start to
    end date`),
 0))
    end as `Start to
    end date`, float(`Start to
    end date`) as _all_time_usage_usd, 2 as _order, coalesce(`5 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `5 periods ago`, coalesce(`4 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `4 periods ago`, coalesce(`3 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `3 periods ago`, coalesce(`2 periods ago`, '<span style="zoom:1;color:#919191">0</span>') as `2 periods ago`, coalesce(`Last period`, '<span style="zoom:1;color:#919191">0</span>') as `Last period`, coalesce(`Current period`, '<span style="zoom:1;color:#919191">0</span>') as `Current period`
from
    union_usage_pivot pivot ( first(usage_info) for x_period_back in ( 'Start to
    end date', '5 periods ago', '4 periods ago', '3 periods ago', '2 periods ago', 'Last period', 'Current period' ) ) union all ( select
    '' as group_key, concat('<b><i>', date_format(:time_range.min, 'MMM dd yyyy'),
 ' - ', date_format(:time_range.max, 'MMM dd yyyy'),
 '</i></b>') as `Start to
    end date`, null as _all_time_usage_usd, 1 as _order, concat('<b><i>', date_format(5_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(4_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `5 periods ago`, concat('<b><i>', date_format(4_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(3_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `4 periods ago`, concat('<b><i>', date_format(3_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(2_periods_ago, -1),
 'MMM dd'),
 '</i></b>') as `3 periods ago`, concat('<b><i>', date_format(2_periods_ago, 'MMM dd'),
 ' - ', date_format(date_add(last_period, -1),
 'MMM dd'),
 '</i></b>') as `2 periods ago`, concat('<b><i>', date_format(last_period, 'MMM dd'),
 ' - ', date_format(date_add(current_period, -1),
 'MMM dd'),
 '</i></b>') as `Last period`, concat('<b><i>', date_format(current_period, 'MMM dd'),
 ' -
    end Time', '</i></b>') as `Current period`
from
    period_info ) ) select
    /*+ BROADCAST(gkr) */ r.*, case
      when :enable_links_toggle = 'Enabled'
      then gkr.latest_asset_route
      else NULL
    end AS latest_asset_route, gkr.latest_asset_owner, MAX(_all_time_usage_usd) OVER() AS max_usage, MIN(_all_time_usage_usd) OVER() AS min_usage, -- add data bars COALESCE(CONCAT( '<div style="position: relative; height: 20px; border-radius: 4px; overflow: hidden;">', '<div style="position: absolute; width: ', ROUND(try_divide((_all_time_usage_usd - min_usage),
 NULLIF(max_usage - min_usage, 0)) * 100, 2),
 '%; height: 100%; background: #077A9D; border-radius: 4px;"></div>', '<span style="position: absolute; width: 100%; text-align: center; line-height: 20px; font-weight: bold; color: #00A972;">', case
      when :usage_toggle = 'Dollars'
      then '$'
      else '' END, FORMAT_NUMBER(_all_time_usage_usd, 0),
 '</span></div>' ),
 `Start to
    end date`) AS usage_bar_html
from
    pre_format_results AS r
  LEFT join
    group_key_routes AS gkr
    on r.group_key = gkr.rank_key order by _order asc, _all_time_usage_usd desc
```

---

## Query 28: contract_burndown

### Parameters

- **contract_start_date** (`contract_start_date`): DATE = `2024-01-01T00:00:00.000`
- **contract_end_date** (`contract_end_date`): DATE = `now/d`
- **usage_amount** (`usage_amount`): DECIMAL = `3000000`
- **support_amount** (`support_amount`): DECIMAL = `500000`
- **discounts_by_product** (`discounts_by_product`): STRING = `sql=0.0;VECTOR_SEARCH=0.0;ALL_PURPOSE=0.0;MODEL_SERVING=0.0;JOBS=0.0;INTERACTIVE=0.0;LAKEHOUSE_MONITORING=0.0;DLT=0.0`
- **price_table** (`price_table`): STRING = `system.billing.list_prices`

### SQL Query

```sql
/* This section is a sample queries that shows how to build a contract burndown analysis given the following input params: 1. Contract Start Date 2. Contract
    end Date 3. Total Usage Amount of Contract 4. Total Support / Overhead amount of Contract 5. Discounts by product/sku list separataed by ; */ SET ansi_mode = True; WITH -- calc list priced usage in USD prices as ( select
    coalesce(price_end_time, date_add(current_date, 1)) as coalesced_price_end_time, * , 'list' AS price_type
from
    IDENTIFIER(:price_table)
where
    currency_code = 'USD' ),
 contract_dates AS ( select
    explode(days) AS usage_date
from
    ( select
    sequence(:contract_start_date::date, :contract_end_date::date) AS days ) ),
 contract_burndown AS ( select
    *, SUM(daily_amount) OVER (ORDER BY usage_date) AS cumulative_estimated_contract_usage, try_divide(cumulative_estimated_contract_usage, total_usage_amount) AS estimated_contract_used
from
    ( select
    contract_dates.usage_date, :usage_amount::float AS total_usage_amount, :support_amount::float AS total_support_amount, (:usage_amount::float / (select
    COUNT(0)
from
    contract_dates)) AS daily_amount
from
    contract_dates ) ),
 -- Now tag combos can be matched in 2 separate ways: key only, or the key=value pair if optionall provided parsed_discounts_table AS ( with
  split_data as (
     select
    explode(split(regexp_replace(upper(:discounts_by_product),
 '\\s+', ''),
 ';')) AS kv_pair
from
    VALUES(1) AS dummy(x) ),
 clean_keys AS ( select
    ROW_NUMBER() OVER (ORDER BY kv_pair) AS order_id, split(kv_pair, '=')[0] AS product, try_cast(split(kv_pair, '=')[1] AS decimal(10,3)) AS discount, kv_pair AS combination, case
      when contains(kv_pair, '=')
      then 1
      else 0
    end AS ContainsValuePair
from
    split_data ) select
    *
from
    clean_keys
where
    ContainsValuePair = 1 ),
 list_priced_usd as ( select
    -- Dynamic Prices Logic COALESCE( -- When there is a * global discounts,
      then use that for all (1-try_cast(regexp_extract(:discounts_by_product, '\\s*\\*\\s*=\\s*([0-9]*\\.?[0-9]+)', 1) AS decimal(10, 3)))* p.pricing.effective_list.default * u.usage_quantity, -- When no account prices enabled,
      then use overrides first
      then default pricing (1-COALESCE(discounts.discount, 0))* p.pricing.effective_list.default * u.usage_quantity, -- When all
      else fails, use default p.pricing.effective_list.default * u.usage_quantity) as usage_usd, u.*
from
    system.billing.usage as u
  LEFT join
    parsed_discounts_table AS discounts
    on discounts.product = u.billing_origin_product
  left join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) ),
 daily_usage AS ( select
    u.usage_date, SUM(coalesce(((1-COALESCE(discounts.discount, 0))* p.pricing.default)*usage_quantity, u.usage_quantity * p.pricing.default, 0)) AS DollarDBUs, SUM(u.usage_quantity * p.pricing.default) AS ListDollarDBUs, SUM(usage_quantity) AS DBUs
from
    system.billing.usage AS u
  LEFT join
    parsed_discounts_table AS discounts
    on discounts.product = u.billing_origin_product
  LEFT join
    prices as p
    on u.sku_name=p.sku_name and u.usage_unit=p.usage_unit and (u.usage_end_time between p.price_start_time and p.coalesced_price_end_time) GROUP BY u.usage_date ),
 clean_burn AS ( select
    c.*, u.usage_date AS u_usage_date, u.DollarDBUs, u.DBUs, (u.DollarDBUs / c.total_usage_amount) * total_support_amount AS SupportDollars, -- Support is burned down in proportion to the usage SUM(DollarDBUs) OVER (ORDER BY c.usage_date) AS cumulative_actual_spend, try_divide(cumulative_actual_spend, total_usage_amount) AS actual_contract_used, -- % completion Flags case
      when c.usage_date < current_date()
      then 'Past Usage' WHEN c.usage_date = current_date()
      then 'Today' WHEN c.usage_date > current_date()
      then 'Future'
    end AS TimePeriodFlag
from
    contract_burndown c
  LEFT join
    daily_usage u
    on c.usage_date = u.usage_date ),
 with_flags AS ( select
    *, SUM(SupportDollars) OVER (ORDER BY usage_date) AS cumulative_actual_spend_support, case
      when usage_date >= current_date()
      then current_date()::date
      else usage_date::date
    end AS contract_active_date, case
      when cumulative_actual_spend > cumulative_estimated_contract_usage
      then 'Burning Above Run Rate'
      else 'Within Run Rate'
    end AS RunRateFlag, case
      when cumulative_actual_spend <= total_usage_amount
      then 'Within Contract'
      else 'Burst Usage'
    end AS BurstFlag, case
      when cumulative_actual_spend > total_usage_amount
      then 1
      else 0
    end AS DidBurst, MAX_BY(estimated_contract_used, usage_date)  FILTER (WHERE TimePeriodFlag != 'Future') AS MostRecentEstimatedBurnProportion, MAX_BY(actual_contract_used, usage_date)  FILTER (WHERE TimePeriodFlag != 'Future') AS MostRecentActualBurnProportion
from
    clean_burn GROUP BY ALL ),
 remaining_tcv AS ( select
    *, COALESCE((select
    MIN(usage_date) AS burst_date
from
    with_flags
where
    DidBurst = 1)::date::string, 'Not Bursting') AS BurstDate, -- Contract remaining total_usage_amount - cumulative_actual_spend AS contract_usage_remaining, total_usage_amount + total_support_amount AS total_contract_value, (total_usage_amount + total_support_amount) - (cumulative_actual_spend + cumulative_actual_spend_support) AS contract_total_remaining
from
    with_flags ) select
    *
from
    remaining_tcv --WHERE u_usage_date IS NOT NULL -- Will not look into future past current usage
```

---

