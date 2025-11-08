# Navixy Dashboard Format Documentation

A comprehensive guide to the Navixy dashboard format that extends a standard JSON dashboard model with SQL execution capabilities and dataset contracts.

> **Note**: This documentation describes the Navixy dashboard format. All standard dashboard fields are documented below, with Navixy-specific extensions prefixed with `x-navixy`.

## Table of Contents

- [Overview](#overview)
- [Dashboard JSON Model](#dashboard-json-model)
  - [Top-Level Dashboard Fields](#top-level-dashboard-fields)
  - [Time Range & Time Picker Settings](#time-range--time-picker-settings)
  - [Templating (Variables)](#templating-variables)
  - [Annotations](#annotations)
  - [Dashboard Links](#dashboard-links)
  - [Panels](#panels)
- [Navixy Extensions](#navixy-extensions)
  - [Dashboard-Level Extensions](#dashboard-level-extensions)
  - [Panel Extensions](#panel-extensions)
- [Dataset Contracts](#dataset-contracts)
- [Visualization Options](#visualization-options)
- [SQL Execution API](#sql-execution-api)
- [Variable Binding](#variable-binding)
- [Verification Rules](#verification-rules)
- [Data Transforms](#data-transforms)
- [Rendering Pipeline](#rendering-pipeline)
- [JSON Schema](#json-schema)
- [Error Handling](#error-handling)
- [Performance & Caching](#performance--caching)
- [Security](#security)
- [Examples](#examples)
- [Developer Workflow](#developer-workflow)

## Overview

The Navixy dashboard format extends a standard dashboard JSON model with vendor-specific extensions that enable:

- **Safe SQL Execution**: Prepared statements with parameter binding
- **Dataset Validation**: Type-safe data contracts for visualizations
- **Multi-tenant Support**: Built-in tenant isolation and security
- **Performance Optimization**: Query deduplication and caching
- **Extensible Visualizations**: Support for custom panel types

This format allows applications to load a single JSON file, execute user-defined SQL safely, and render various visualizations (bar charts, line charts, pie charts, KPI tiles, tables, annotations) from the returned datasets.

## Dashboard JSON Model

The Navixy format is built on top of a standard dashboard JSON model. All standard dashboard fields are supported and documented below. Navixy-specific extensions are prefixed with `x-navixy` and never modify existing dashboard keys.

### Top-Level Dashboard Fields

A dashboard JSON object contains the following top-level fields:

#### `id`
- **Type**: `number | null`
- **Description**: Numeric unique identifier for the dashboard (assigned by the database upon save). It is `null` for new/unsaved dashboards.

#### `uid`
- **Type**: `string`
- **Description**: String unique identifier (8–40 characters) that can be set for the dashboard. This is used to uniquely identify the dashboard in URLs and APIs.

#### `title`
- **Type**: `string`
- **Required**: Yes
- **Description**: The name/title of the dashboard.

#### `description`
- **Type**: `string`
- **Required**: No
- **Description**: A text description of the dashboard. This is stored in the JSON (and accessible via API) but not prominently shown in the UI.

#### `tags`
- **Type**: `string[]`
- **Required**: No
- **Description**: Array of tag strings associated with the dashboard (for organizational purposes).

#### `style`
- **Type**: `"dark" | "light"`
- **Required**: No
- **Description**: Theme of the dashboard, either "dark" or "light" (affects panel background and text colors).

#### `timezone`
- **Type**: `string`
- **Required**: No
- **Description**: Time zone for this dashboard. Usually "browser" (use viewer's local time) or "utc", or an IANA timezone ID (e.g., "America/New_York").

#### `editable`
- **Type**: `boolean`
- **Required**: No
- **Default**: `true`
- **Description**: Boolean indicating if the dashboard can be edited (controls UI lock/unlock).

#### `graphTooltip`
- **Type**: `0 | 1 | 2`
- **Required**: No
- **Description**: Sets the behavior of tooltips and crosshair on graphs:
  - `0` = no shared tooltip
  - `1` = shared crosshair
  - `2` = shared crosshair *and* shared tooltip across panels

#### `time`
- **Type**: `object`
- **Required**: Yes
- **Description**: The time range for the dashboard. It's an object with "from" and "to" properties defining the default time range (e.g. `"from": "now-6h"`, `"to": "now"` for last 6 hours).
  - `from`: Start of the range (e.g. "now-6h" for "now minus 6 hours")
  - `to`: End of the range (e.g. "now" for the current time)
  - These can be relative (like `now-6h`, `now-7d`) or absolute timestamps

#### `refresh`
- **Type**: `string | null`
- **Required**: No
- **Description**: Default auto-refresh interval for the dashboard (e.g. "5s", "1m", etc.). Can be empty or omitted if no auto-refresh is set.

#### `timepicker`
- **Type**: `object`
- **Required**: No
- **Description**: An object describing time picker options (see [Time Picker Settings](#time-range--time-picker-settings) below).

#### `templating`
- **Type**: `object`
- **Required**: No
- **Description**: An object describing dashboard variables (templating). It contains a list of variable definitions (see [Templating (Variables)](#templating-variables) below).

#### `annotations`
- **Type**: `object`
- **Required**: No
- **Description**: An object describing annotation queries configured for this dashboard. It contains a list of annotation definitions (see [Annotations](#annotations) below).

#### `panels`
- **Type**: `array`
- **Required**: Yes
- **Description**: Array of panel objects that make up the dashboard (see [Panels](#panels) section below). Each object in this array is a panel with its own type and settings.

#### `links`
- **Type**: `array`
- **Required**: No
- **Description**: Array of dashboard link objects. These are external or internal links shown on the dashboard (see [Dashboard Links](#dashboard-links) below). If no links are set, this is an empty array.

#### `schemaVersion`
- **Type**: `number`
- **Required**: No
- **Description**: Integer schema version of the dashboard JSON. This increments when the dashboard JSON format changes (e.g. new release with a new field). Common values: 16, 30, 36, 38 for recent versions.

#### `version`
- **Type**: `number`
- **Required**: No
- **Description**: Integer revision of the dashboard, incremented each time the dashboard is saved/updated by a user.

#### `gnetId`
- **Type**: `number | null`
- **Required**: No
- **Description**: ID of the dashboard on a community dashboard site if it was imported from there. It is `null` if the dashboard isn't sourced from a community site.

#### `iteration`
- **Type**: `number | null`
- **Required**: No
- **Description**: An internal revision identifier, often a timestamp value, used internally by the dashboard system. This is distinct from the human-edited version number, and is used to track changes or cache-busting for live dashboards.

#### `fiscalYearStartMonth`
- **Type**: `number`
- **Required**: No
- **Description**: The month (0–11) that the fiscal year starts on, used for custom time ranges (e.g. week numbering). 0 = January, 11 = December.

#### `liveNow`
- **Type**: `boolean`
- **Required**: No
- **Description**: Boolean indicating if the dashboard is currently in "live" mode (real-time streaming data). When `true`, the dashboard UI will continuously poll or stream new data (as if the time range is moving forward, like a live tail).

### Time Range & Time Picker Settings

#### Dashboard Time Range (`time`)

The `time` field specifies the default time range of the dashboard:

```json
{
  "time": {
    "from": "now-6h",
    "to": "now"
  }
}
```

- `from`: Start of the range (e.g. "now-6h" for "now minus 6 hours")
- `to`: End of the range (e.g. "now" for the current time)
- These can be relative (like `now-6h`, `now-7d`) or absolute timestamps

#### Auto-Refresh (`refresh`)

The `refresh` field sets the default refresh interval:

```json
{
  "refresh": "5s"
}
```

- Examples: `"5s"`, `"1m"`, `"5m"`, `"30s"`
- Can be empty or omitted if no auto-refresh is set

#### Timezone (`timezone`)

```json
{
  "timezone": "browser"
}
```

- `"browser"`: Use the viewer's browser time zone
- `"utc"`: Use UTC
- IANA timezone ID: e.g., `"America/New_York"`, `"Europe/London"`

#### Time Picker (`timepicker`)

The `timepicker` field configures the dashboard's time filter UI:

```json
{
  "timepicker": {
    "collapse": false,
    "enable": true,
    "hidden": false,
    "now": true,
    "nowDelay": "1m",
    "refresh_intervals": ["5s", "10s", "30s", "1m", "5m", "15m", "30m", "1h", "2h", "1d"],
    "time_options": ["5m", "15m", "1h", "6h", "12h", "24h", "2d", "7d", "30d"]
  }
}
```

**Fields:**
- `collapse`: Boolean, whether the time picker is in a collapsed (dropdown) state by default
- `enable`: Boolean, whether the time picker is enabled/shown (usually `true`)
- `hidden`: Boolean, if `true`, the time picker UI is hidden
- `now`: Boolean, whether "Now" is a valid option for time range selection (usually `true`)
- `nowDelay`: String, a delay to apply to "now" (e.g. `"1m"`, `"5m"`). This effectively shifts what "now" means, to avoid null data from indexing delays
- `refresh_intervals`: Array of strings for refresh intervals to display in the UI's refresh dropdown
- `time_options`: Array of strings for quick range options (e.g., `"Last 5m"`, `"Last 15m"`, etc.)

### Templating (Variables)

The `templating` field holds the dashboard's template variables (also known as *dashboard variables*). Variables allow dynamic filtering in dashboards (e.g., select server, region, etc.).

```json
{
  "templating": {
    "enable": true,
    "list": [
      {
        "name": "var_tenant",
        "label": "Tenant",
        "type": "constant",
        "query": "8a5e4d6c-1234-5678-9abc-def012345678",
        "current": {
          "value": "8a5e4d6c-1234-5678-9abc-def012345678",
          "text": "8a5e4d6c-1234-5678-9abc-def012345678"
        },
        "options": [
          {
            "text": "8a5e4d6c-1234-5678-9abc-def012345678",
            "value": "8a5e4d6c-1234-5678-9abc-def012345678",
            "selected": true
          }
        ]
      }
    ]
  }
}
```

**Common Variable Fields:**
- `name`: The variable name (unique identifier) used in queries (e.g. `$env`). This is the actual variable key.
- `label`: Optional. Human-friendly label for the variable (shown in the dropdown). If not set, the name is used as the label.
- `type`: Type of variable, such as:
  - `"query"`: Values come from a data source query
  - `"interval"`: Time interval list
  - `"custom"`: Fixed list of values
  - `"constant"`: Constant value
  - `"datasource"`: Data source selector
  - `"textbox"`: Text input
- `query`: For query variables, the query string used to fetch options (syntax depends on data source).
- `datasource`: The data source used for the variable's query. Often `null` means use the default data source, or it can be set to a specific data source name/UID.
- `options`: Array of value options (each option is an object with `text` and `value`, and a `selected` boolean).
- `current`: An object representing the current selected value(s) for the variable. It typically has `text` (the display text) and `value` (the actual value).
- `includeAll`: Boolean indicating if an "All" option is available for this variable (when `true`, an option representing "all values" is added).
- `allValue`: Optional. If "All" is enabled, this can define what value to use when "All" is selected (e.g. a wildcard or regex `.*`).
- `allFormat`: Format used when the "All" option is selected. Common values: `wildcard`, `regex`, `glob`, `pipe`.
- `multi`: Boolean indicating if multiple values can be selected (multi-select).
- `multiFormat`: Format to use when multiple values are selected (e.g., how to join them in queries: `glob`, `wildcard`, `regex`).
- `regex`: Optional. For query variables, a regex pattern to filter or capture part of the results from the query.
- `refresh`: When to update the variable options:
  - `0` or `false` = never refresh automatically
  - `1` = refresh on time range change
  - `2` = refresh on dashboard load
- `description`: Optional. A description for the variable (usually shown as info on hover in the UI).

### Annotations

Annotations provide a way to mark events or highlight regions on graphs (shown as vertical lines or shaded regions with tooltips).

```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "name": "Annotations & Alerts",
        "type": "dashboard",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "datasource": {
          "type": "dashboard",
          "uid": "-- Dashboard --"
        },
        "target": {
          "type": "dashboard",
          "tags": [],
          "limit": 100,
          "matchAny": false
        }
      }
    ]
  }
}
```

**Common Annotation Fields:**
- `builtIn`: If present and non-zero, indicates a built-in annotation source. The default "Annotations & Alerts" query has `builtIn: 1`.
- `name`: Name of the annotation query entry (as it appears in the dashboard UI settings).
- `datasource`: Data source from which to pull annotations. For the built-in dashboard annotation, this is the internal data source (`"type": "dashboard"`, `"uid": "-- Dashboard --"`).
- `enable`: Boolean to enable/disable this annotation query.
- `hide`: Boolean indicating if the annotations from this source should be hidden by default.
- `iconColor`: Color for the annotation marker (vertical line icon color) in RGBA or hex format (e.g., `"rgba(0, 211, 255, 1)"`).
- `type`: Type of annotation query. For the built-in it is `"dashboard"` (meaning it draws from the internal annotation storage for this dashboard, and alerts).
- `target`: Optional, for certain types. An object defining the query for annotations when using tags or an external source.

### Dashboard Links

Dashboard links are used to provide clickable links or drop-down menus on a dashboard, linking to other dashboards or external URLs for context.

```json
{
  "links": [
    {
      "title": "Production Overview",
      "tooltip": "Go to Production cluster overview",
      "type": "link",
      "url": "https://dashboard.mycompany.com/d/abcd1234/production-overview",
      "icon": "external link",
      "targetBlank": true,
      "includeVars": true,
      "keepTime": true
    },
    {
      "title": "Team Dashboards",
      "type": "dashboards",
      "tags": ["team-X"],
      "asDropdown": true
    }
  ]
}
```

**Common Link Fields:**
- `title`: The text to display for the link.
- `type`: The link type:
  - `"dashboards"`: This link will open a list of dashboards (filtered by tags)
  - `"link"`: A direct link to an external URL or absolute/relative dashboard URL
- `url`: Required if `type` is `"link"`. The URL to open when the link is clicked.
- `tags`: For `type: "dashboards"`. An array of tag names to filter which dashboards are shown in the drop-down.
- `asDropdown`: Boolean, used only for `type: "dashboards"`. If `true`, multiple dashboards matching the tags will appear as a dropdown menu under one link title.
- `icon`: Name of an icon to display with the link (e.g. `external link`, `bolt`, etc.).
- `tooltip`: Hover text for the link (displayed when user hovers over the link).
- `targetBlank`: Boolean, if `true` the link opens in a new tab/window (i.e. `target="_blank"`).
- `includeVars`: Boolean, if `true`, the dashboard will include the current dashboard's variable values as URL parameters when navigating via this link.
- `keepTime`: Boolean, if `true`, the current time range is included in the link (as URL params) so that the target page sees the same time window.

### Panels

Panels are the building blocks of a dashboard. Each panel is a visualization (graph, gauge, table, text, etc.) with its own queries and settings.

**Common Panel Fields:**

- `id`: Numeric panel ID, unique within the dashboard. This is used for referencing panels (e.g. in annotations). Panel IDs are assigned when panels are created.
- `type`: Panel type (e.g. `"graph"`, `"timeseries"`, `"gauge"`, `"table"`, `"text"`, etc.). This determines what kind of visualization and options the panel will have.
- `title`: Panel title, displayed at the top of the panel.
- `description`: Optional. Panel description, shown on hover of info icon in the UI.
- `gridPos`: An object specifying the panel's size and position in the dashboard grid:
  - `x`: Column position (0–23, since dashboard width is 24 columns)
  - `y`: Row position (in grid units)
  - `w`: Width in grid columns (1–24)
  - `h`: Height in grid units (each unit ~30px)
- `datasource`: The data source for the panel's queries. In newer versions this is often an object with `{"type": "<type>", "uid": "<datasource UID>"}`, or it can be set to `null` to use the default data source.
- `targets`: An array of query definitions for this panel. Each target defines a query to the datasource (for example, a PromQL or SQL query) and has its own parameters:
  - `refId`: Alphabetical ID of the query (e.g. "A", "B", etc.)
  - Other fields within a target depend on the data source type
- `fieldConfig`: An object defining per-field display settings and overrides. This contains:
  - `defaults`: Default display options for all series/fields, such as color, units, number of decimals, threshold definitions, etc.
  - `overrides`: An array of field override rules to customize specific series/fields beyond the defaults
- `options`: Panel-type-specific settings. This is an object whose contents depend on the panel `type`. For example, a time series chart's options might include settings for legend display, tooltip mode, draw style (line/bar points), etc.
- `pluginVersion`: Version of the panel plugin in use (e.g. "8.5.9" or "10.0.2"). This is set to track which version of the panel was used, for compatibility.
- `transparent`: Optional. Boolean indicating if the panel has a transparent background (so the dashboard background shows through). If `false` or omitted, panel has an opaque background.
- `maxDataPoints`: Optional. Maximum data points the panel should plot. The dashboard system might down-sample data for high-resolution displays based on this. Often seen in certain panels like Graph, Gauge, etc., defaulting to 100 or auto.
- `transformations`: Optional. An array of transformations applied to the panel's data. Each transformation entry has an `id` (type of transformation) and an `options` object defining its settings.

## Navixy Extensions

All Navixy-specific fields are prefixed with `x-navixy` and never modify existing dashboard keys. Tools that don't recognize Navixy keys can safely ignore them.

### Dashboard-Level Extensions

#### `x-navixy`

Dashboard-level Navixy configuration:

```json
{
  "x-navixy": {
    "schemaVersion": "1.0.0",
    "execution": {
      "endpoint": "/api/v1/sql/run",
      "dialect": "postgresql",
      "timeout_ms": 10000,
      "max_rows": 10000,
      "read_only": true,
      "allowed_schemas": ["raw_telematics_data", "raw_business_data"]
    },
    "parameters": {
      "bindings": {
        "tenant_id": "${var_tenant}",
        "from": "${__from}",
        "to": "${__to}"
      }
    }
  }
}
```

**Fields:**
- `schemaVersion`: Semantic version of the Navixy schema (e.g., "1.0.0")
- `execution`: SQL execution configuration:
  - `endpoint`: SQL execution API endpoint
  - `dialect`: SQL dialect (currently supports "postgresql")
  - `timeout_ms`: Maximum query execution time in milliseconds
  - `max_rows`: Maximum rows returned per query
  - `read_only`: Enforce read-only queries
  - `allowed_schemas`: Whitelist of database schemas
- `parameters`: Dashboard-level default parameter bindings:
  - `bindings`: Maps dashboard variables to SQL parameters
  - Can be overridden at the panel level

### Panel Extensions

Each panel extends the standard dashboard structure with Navixy-specific SQL execution and data validation:

```json
{
  "type": "barchart",
  "title": "Events by Type",
  "gridPos": { "x": 0, "y": 0, "w": 12, "h": 8 },
  "x-navixy": {
    "sql": {
      "statement": "SELECT event_type AS category, COUNT(*)::int AS value FROM events WHERE tenant_id = ${tenant_id} AND ts >= ${__from} AND ts < ${__to} GROUP BY 1 ORDER BY 2 DESC LIMIT ${limit}",
      "params": {
        "tenant_id": { "type": "uuid" },
        "from": { "type": "timestamptz" },
        "to": { "type": "timestamptz" },
        "limit": { "type": "int", "default": 20, "min": 1, "max": 1000 }
      },
      "bindings": {
        "tenant_id": "${var_tenant}",
        "from": "${__from}",
        "to": "${__to}"
      },
      "limits": { "timeout_ms": 8000, "max_rows": 5000 },
      "read_only": true
    },
    "dataset": {
      "shape": "category_value",
      "columns": {
        "category": { "type": "string" },
        "value": { "type": "number" }
      }
    },
    "verify": {
      "required_columns": ["category", "value"],
      "types": { "category": "string", "value": "number" },
      "min_rows": 1,
      "max_rows": 1000,
      "unique": ["category"]
    },
    "transform": [
      { "op": "sort", "by": "value", "dir": "desc" },
      { "op": "limit", "n": 20 }
    ],
    "on_empty": "show_placeholder",
    "on_error": "show_message",
    "visualization": {
      "orientation": "vertical",
      "showValues": true,
      "sortOrder": "desc"
    }
  }
}
```

#### `x-navixy.sql`
- `statement`: SQL query with named parameters
- `params`: Parameter type definitions and constraints
- `bindings`: Variable-to-parameter mappings
- `limits`: Panel-specific execution limits
- `read_only`: Enforce read-only queries

#### `x-navixy.dataset`
- `shape`: Expected data structure type
- `columns`: Column type definitions

#### `x-navixy.verify`
- `required_columns`: Mandatory column names
- `types`: Expected column types
- `min_rows`/`max_rows`: Row count constraints
- `unique`: Columns that must contain unique values

#### `x-navixy.transform`
- Array of post-query data transformations
- Operations: sort, limit, rename, calc, pivot/unpivot

#### `x-navixy.visualization`
- Optional visualization options for controlling how data is rendered
- Type-specific options for bar charts, line charts, pie charts, KPI cards, tables, and annotations
- See [Visualization Options](#visualization-options) section for detailed documentation

## Dataset Contracts

Each visualization type expects specific data structures:

| Visual Type | Required Columns | Optional Columns | Notes |
|-------------|------------------|------------------|-------|
| `barchart` | `category:string`, `value:number` | `series:string` | Grouped bars if series present |
| `linechart` | `time:timestamp`, `value:number` | `series:string` | Time must be sortable |
| `piechart` | `category:string`, `value:number` | – | Values ≥ 0, normalized to 100% |
| `kpi` | `value:number` (single row) | `delta:number`, `trend:string` | Uses first row if multiple. `delta` for change display, `trend` for direction (`"up"`, `"down"`, `"stable"`) |
| `table` | Any columns | – | Raw rows with pagination/sorting |
| `annotation` | None | `text:string` | Static content, no SQL |

## Visualization Options

Each panel can include optional visualization settings in `x-navixy.visualization` to control how data is rendered. These options are suggestions for BI renderers and are not mandatory—renderers may implement their own defaults or ignore unsupported options.

### Bar Chart Options

Bar charts display categorical data with rectangular bars. Configure visualization behavior:

```json
{
  "x-navixy": {
    "visualization": {
      "orientation": "vertical",
      "stacking": "none",
      "showValues": false,
      "sortOrder": "desc",
      "barSpacing": 0.2,
      "colorPalette": "classic",
      "showLegend": true,
      "legendPosition": "bottom"
    }
  }
}
```

**Options:**
- `orientation`: `"horizontal"` | `"vertical"` - Bar direction (default: `"vertical"`)
- `stacking`: `"none"` | `"stacked"` | `"percent"` - Stacking mode for grouped bars (default: `"none"`)
- `showValues`: `boolean` - Display value labels on bars (default: `false`)
- `sortOrder`: `"asc"` | `"desc"` | `"none"` - Sort bars by value (default: `"none"`)
- `barSpacing`: `number` - Spacing between bars (0-1, default: `0.2`)
- `colorPalette`: `"classic"` | `"modern"` | `"pastel"` | `"vibrant"` - Color scheme (default: `"classic"`)
- `showLegend`: `boolean` - Show legend when `series` column present (default: `true`)
- `legendPosition`: `"top"` | `"bottom"` | `"left"` | `"right"` - Legend placement (default: `"bottom"`)

### Line Chart Options

Line charts display time-series data with connected points. Configure line appearance and behavior:

```json
{
  "x-navixy": {
    "visualization": {
      "lineStyle": "solid",
      "lineWidth": 2,
      "showPoints": "auto",
      "pointSize": 5,
      "interpolation": "linear",
      "fillArea": "none",
      "showGrid": true,
      "showLegend": true,
      "legendPosition": "bottom",
      "colorPalette": "classic"
    }
  }
}
```

**Options:**
- `lineStyle`: `"solid"` | `"dashed"` | `"dotted"` - Line style (default: `"solid"`)
- `lineWidth`: `number` - Line thickness in pixels (default: `2`)
- `showPoints`: `"always"` | `"auto"` | `"never"` - When to show data points (default: `"auto"`)
- `pointSize`: `number` - Size of data points in pixels (default: `5`)
- `interpolation`: `"linear"` | `"step"` | `"smooth"` - Line interpolation method (default: `"linear"`)
- `fillArea`: `"none"` | `"below"` | `"above"` - Fill area under/over line (default: `"none"`)
- `showGrid`: `boolean` - Show grid lines (default: `true`)
- `showLegend`: `boolean` - Show legend when `series` column present (default: `true`)
- `legendPosition`: `"top"` | `"bottom"` | `"left"` | `"right"` - Legend placement (default: `"bottom"`)
- `colorPalette`: `"classic"` | `"modern"` | `"pastel"` | `"vibrant"` - Color scheme (default: `"classic"`)

### Pie Chart Options

Pie charts display proportional data as circular segments. Configure appearance and labeling:

```json
{
  "x-navixy": {
    "visualization": {
      "donut": false,
      "donutRadius": 0.5,
      "showLabels": "auto",
      "labelFormat": "name_value",
      "labelPosition": "outside",
      "showLegend": true,
      "legendPosition": "bottom",
      "sortOrder": "desc",
      "startAngle": 0,
      "colorPalette": "classic"
    }
  }
}
```

**Options:**
- `donut`: `boolean` - Render as donut chart (default: `false`)
- `donutRadius`: `number` - Inner radius ratio (0-1, default: `0.5`) - only used when `donut: true`
- `showLabels`: `"always"` | `"auto"` | `"never"` - When to show labels (default: `"auto"`)
- `labelFormat`: `"name"` | `"value"` | `"percent"` | `"name_value"` | `"name_percent"` - Label content (default: `"name_value"`)
- `labelPosition`: `"inside"` | `"outside"` - Label placement (default: `"outside"`)
- `showLegend`: `boolean` - Show legend (default: `true`)
- `legendPosition`: `"top"` | `"bottom"` | `"left"` | `"right"` - Legend placement (default: `"bottom"`)
- `sortOrder`: `"asc"` | `"desc"` | `"none"` - Sort segments by value (default: `"desc"`)
- `startAngle`: `number` - Starting angle in degrees (0-360, default: `0`)
- `colorPalette`: `"classic"` | `"modern"` | `"pastel"` | `"vibrant"` - Color scheme (default: `"classic"`)

### KPI Card Options

KPI cards display single metrics with optional context. Configure display mode and formatting:

```json
{
  "x-navixy": {
    "visualization": {
      "displayMode": "value_delta",
      "valueFormat": "short",
      "unit": "",
      "decimals": 0,
      "colorMode": "threshold",
      "showSparkline": false,
      "thresholds": [
        { "value": null, "color": "green" },
        { "value": 50, "color": "yellow" },
        { "value": 80, "color": "red" }
      ],
      "size": "normal"
    }
  }
}
```

**Options:**
- `displayMode`: `"value"` | `"value_delta"` | `"value_trend"` | `"value_delta_trend"` - What to display (default: `"value"`)
  - `"value"`: Show only the main value
  - `"value_delta"`: Show value with delta (requires `delta` column)
  - `"value_trend"`: Show value with trend indicator (requires `trend` column)
  - `"value_delta_trend"`: Show value with both delta and trend
- `valueFormat`: `"short"` | `"long"` | `"currency"` | `"percent"` | `"scientific"` - Number format (default: `"short"`)
- `unit`: `string` - Unit suffix (e.g., `"km"`, `"%"`, `"$"`) (default: `""`)
- `decimals`: `number` - Decimal places (default: `0`)
- `colorMode`: `"value"` | `"threshold"` | `"background"` | `"none"` - Color application (default: `"threshold"`)
- `showSparkline`: `boolean` - Show mini trend line (requires time-series data) (default: `false`)
- `thresholds`: `array` - Color thresholds `[{value: number | null, color: string}]` (default: `[{value: null, color: "green"}]`)
- `size`: `"compact"` | `"normal"` | `"large"` - Card size (default: `"normal"`)

**Dataset Enhancement:** For `value_delta` mode, dataset may include optional `delta:number` column. For `value_trend` mode, dataset may include optional `trend:string` column with values `"up"`, `"down"`, or `"stable"`.

### Table Options

Tables display tabular data with rows and columns. Configure display and interaction:

```json
{
  "x-navixy": {
    "visualization": {
      "showHeader": true,
      "sortable": true,
      "pageSize": 25,
      "showPagination": true,
      "columnWidth": "auto",
      "rowHighlighting": "none",
      "showTotals": false,
      "totalsRow": "bottom"
    }
  }
}
```

**Options:**
- `showHeader`: `boolean` - Show column headers (default: `true`)
- `sortable`: `boolean` - Enable column sorting (default: `true`)
- `pageSize`: `number` - Rows per page (default: `25`)
- `showPagination`: `boolean` - Show pagination controls (default: `true`)
- `columnWidth`: `"auto"` | `"equal"` | `"fit"` - Column width strategy (default: `"auto"`)
- `rowHighlighting`: `"none"` | `"alternating"` | `"hover"` | `"both"` - Row highlighting mode (default: `"none"`)
- `showTotals`: `boolean` - Show totals row (default: `false`)
- `totalsRow`: `"top"` | `"bottom"` - Totals row position (default: `"bottom"`) - only used when `showTotals: true`

### Annotation Options

Annotations mark events on time-series charts. Configure appearance:

```json
{
  "x-navixy": {
    "visualization": {
      "iconColor": "rgba(0, 211, 255, 1)",
      "iconShape": "circle",
      "showText": true,
      "textPosition": "right"
    }
  }
}
```

**Options:**
- `iconColor`: `string` - Marker color (RGBA or hex, default: `"rgba(0, 211, 255, 1)"`)
- `iconShape`: `"circle"` | `"square"` | `"diamond"` | `"triangle"` - Marker shape (default: `"circle"`)
- `showText`: `boolean` - Show annotation text (default: `true`)
- `textPosition`: `"left"` | `"right"` | `"top"` | `"bottom"` - Text position relative to marker (default: `"right"`)

## SQL Execution API

### Request Format

```json
{
  "dialect": "postgresql",
  "statement": "SELECT ... WHERE ts>=${__from} AND ts<${__to} AND tenant_id=${tenant_id}",
  "params": {
    "tenant_id": "e9e4d6...",
    "from": "2025-10-24T10:00:00Z",
    "to": "2025-10-25T10:00:00Z",
    "limit": 20
  },
  "limits": { "timeout_ms": 8000, "max_rows": 5000 },
  "read_only": true
}
```

### Response Format

```json
{
  "columns": [
    { "name": "category", "type": "string" },
    { "name": "value", "type": "number" }
  ],
  "rows": [
    ["Harsh Braking", 312],
    ["Speeding", 271]
  ],
  "stats": { "row_count": 2, "elapsed_ms": 128 }
}
```

### Safety Requirements

- **Prepared Statements**: No string concatenation, only parameter binding
- **Read-Only**: Reject INSERT/UPDATE/DELETE/DDL operations
- **Schema Allowlist**: Enforce SET LOCAL ROLE/SEARCH_PATH per tenant
- **Resource Limits**: Max rows, timeout, and complexity caps
- **Audit Logging**: Log dashboard_uid, panel_id, statement hash, params, timing
- **Tenancy**: Require tenant_id in every statement
- **Rate Limiting**: Per-user/API token limits

See the [API Reference](api-reference/README.md) for detailed documentation.

## Variable Binding

### Binding Syntax
- Maps dashboard variables to SQL parameters via `x-navixy.sql.bindings`
- Supports reserved time range variables: `${__from}`, `${__to}` (automatically populated from `dashboard.time`)
- Type conversion handled by the renderer

### Supported Time Formats
- Epoch milliseconds
- ISO 8601 strings
- Backend converts to `timestamptz`

### Type Coercions
- `:int` - Integer conversion
- `:numeric` - Decimal conversion
- `:uuid` - UUID validation
- `:text[]` - Array conversion for multi-select variables

## Verification Rules

Each panel can declare validation rules in `x-navixy.verify`:

```json
{
  "verify": {
    "required_columns": ["category", "value"],
    "types": { "category": "string", "value": "number" },
    "min_rows": 1,
    "max_rows": 1000,
    "unique": ["category"],
    "assertions": [
      { "op": "gte", "left": "sum(value)", "right": 0 }
    ]
  }
}
```

### Verification Types
- **Column Requirements**: Required columns and types
- **Row Constraints**: Minimum and maximum row counts
- **Uniqueness**: Columns that must contain unique values
- **Assertions**: Advanced validation expressions

## Data Transforms

Post-query transformations applied before rendering:

```json
{
  "transform": [
    { "op": "sort", "by": "value", "dir": "desc" },
    { "op": "limit", "n": 20 },
    { "op": "rename", "map": { "old_name": "new_name" } },
    { "op": "calc", "expr": "value / total" }
  ]
}
```

### Transform Operations
- **`sort`**: Sort by column with direction
- **`limit`**: Limit number of rows
- **`rename`**: Rename columns
- **`calc`**: Calculate new columns
- **`pivot`/`unpivot`**: Reshape data

## Rendering Pipeline

The runtime execution flow:

1. **Load JSON**: Parse dashboard configuration
2. **Validate Schema**: Check against Navixy JSON Schema
3. **Resolve Variables**: Convert dashboard variables to typed bindings
4. **Authorize**: Validate user permissions and tenant access
5. **Execute SQL**: Run prepared statements via API endpoint
6. **Verify Dataset**: Validate against panel contracts
7. **Transform Data**: Apply post-query transformations
8. **Render Panel**: Dispatch to appropriate visualization handler
9. **Handle Fallbacks**: Show placeholders for empty/error states
10. **Log/Trace**: Record execution metrics and correlation IDs

## JSON Schema

The complete JSON Schema definition is available in [`navixy-dashboard.schema.json`](schema/navixy-dashboard.schema.json). Key validation rules:

- Dashboard-level `x-navixy` configuration
- Panel-level SQL and dataset definitions
- Parameter type validation
- Dataset shape verification
- Transform operation validation

## Error Handling

### Error Types and Responses

#### SQL Errors
- **Response**: Structured error message with safe error ID
- **Logging**: Full error details server-side
- **User Display**: User-friendly error message

#### Verification Failures
- **Response**: "Data contract mismatch" with specific rule details
- **Logging**: Validation failure details
- **User Display**: Clear explanation of data requirements

#### Empty Results
- **Response**: Respect `on_empty` setting
- **Options**: Show placeholder or helpful text
- **Logging**: Empty result metrics

#### Timeouts
- **Response**: Suggest narrowing time range or filters
- **Logging**: Timeout exceeded with query details
- **User Display**: Performance optimization suggestions

## Performance & Caching

### Optimization Strategies

#### Query Optimization
- **Tile Prefetch**: Execute KPI tiles first (short queries)
- **Query Deduplication**: Share results for identical statement+params
- **Streaming**: Chunked responses for large tables

#### Caching
- **Cache Key**: `(dashboard_uid, panel_id, params_hash, schemaVersion)`
- **TTL**: Short TTL for live views, longer for historical data
- **ETag**: Conditional requests for unchanged data

#### Resource Management
- **Connection Pooling**: Reuse database connections
- **Query Limits**: Enforce max_rows and timeout constraints
- **Memory Management**: Stream large result sets

## Security

### Security Checklist

#### Static Analysis
- **SQL Linter**: Deny DML/DDL, semicolons, dangerous CTEs
- **Parameter Validation**: Enforce type constraints
- **Tenant Isolation**: Verify tenant_id in every query

#### Runtime Security
- **Prepared Statements**: Prevent SQL injection
- **Schema Isolation**: Enforce allowed_schemas
- **Rate Limiting**: Per-user and per-panel limits
- **Audit Logging**: Complete query execution logs

#### Testing Requirements
- **Unit Tests**: "Evil" variable injection tests
- **Integration Tests**: End-to-end security validation
- **Penetration Testing**: Regular security assessments

See the [Security Guide](security/README.md) for detailed security documentation.

## Examples

### Complete Dashboard Example

See [`fleet-status-dashboard.json`](examples/fleet-status-dashboard.json) for a complete working example.

### Panel Examples

#### KPI Panel
```json
{
  "type": "kpi",
  "title": "Active Vehicles",
  "gridPos": { "x": 0, "y": 0, "w": 4, "h": 4 },
  "x-navixy": {
    "sql": {
      "statement": "SELECT COUNT(DISTINCT device_id)::int AS value FROM raw_telematics_data.tracking_data_core WHERE tenant_id = ${tenant_id} AND device_time >= ${__from}",
      "params": { "tenant_id": { "type": "uuid" }, "from": { "type": "timestamptz" } },
      "bindings": { "tenant_id": "${var_tenant}", "from": "${__from}" }
    },
    "dataset": { "shape": "kpi", "columns": { "value": { "type": "number" } } },
    "verify": { "required_columns": ["value"], "min_rows": 1, "max_rows": 1 }
  }
}
```

#### Bar Chart Panel
```json
{
  "type": "barchart",
  "title": "Events by Type",
  "gridPos": { "x": 4, "y": 0, "w": 12, "h": 8 },
  "x-navixy": {
    "sql": {
      "statement": "SELECT event_type AS category, COUNT(*)::int AS value FROM events WHERE tenant_id = ${tenant_id} AND ts >= ${__from} AND ts < ${__to} GROUP BY 1 ORDER BY 2 DESC LIMIT ${limit}",
      "params": {
        "tenant_id": { "type": "uuid" },
        "from": { "type": "timestamptz" },
        "to": { "type": "timestamptz" },
        "limit": { "type": "int", "default": 20, "min": 1, "max": 1000 }
      },
      "bindings": { "tenant_id": "${var_tenant}", "from": "${__from}", "to": "${__to}" }
    },
    "dataset": { "shape": "category_value", "columns": { "category": { "type": "string" }, "value": { "type": "number" } } },
    "verify": { "required_columns": ["category", "value"], "min_rows": 1, "max_rows": 1000 }
  }
}
```

## Developer Workflow

### 1. Dashboard Authoring
- Start with standard dashboard JSON structure
- Add `x-navixy` extensions for each panel
- Define SQL statements with parameter types
- Specify dataset contracts and verification rules

### 2. Validation
- Validate against JSON Schema
- Run static SQL analysis
- Test with sample data
- Verify security constraints

### 3. Testing
- Unit tests for SQL safety
- Integration tests with real data
- Performance testing with large datasets
- Security penetration testing

### 4. Deployment
- Ship JSON files with application
- Configure SQL execution endpoint
- Set up monitoring and alerting
- Deploy with proper security controls

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [JSON Schema Specification](https://json-schema.org/)
