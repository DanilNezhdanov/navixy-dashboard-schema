# **Dashboard JSON Model**

A dashboard is represented as a JSON object containing all its metadata – general properties, panels and their configurations, variable definitions, time settings, etc. This JSON model is used for importing/exporting dashboards and for provisioning them via code. Below is a comprehensive outline of the structure and fields in the dashboard JSON model, along with the meaning of each field.

## **Top-Level Dashboard Fields**

* **id** – Numeric unique identifier for the dashboard (assigned by the database upon save) . It is null for new/un-saved dashboards.

* **uid** – String unique identifier (8–40 characters) that can be set for the dashboard. This is used to uniquely identify the dashboard in URLs and APIs .

* **title** – The name/title of the dashboard .

* **description** – *Optional.* A text description of the dashboard. This is stored in the JSON (and accessible via API) but not prominently shown in the UI .

* **tags** – Array of tag strings associated with the dashboard (for organizational purposes) .

* **style** – Theme of the dashboard, either "dark" or "light" (affects panel background and text colors) .

* **timezone** – Time zone for this dashboard. Usually "browser" (use viewer's local time) or "utc", or an IANA timezone ID .

* **editable** – Boolean indicating if the dashboard can be edited (controls UI lock/unlock) .

* **graphTooltip** – Sets the behavior of tooltips and crosshair on graphs: 0 \= no shared tooltip, 1 \= shared crosshair, 2 \= shared crosshair *and* shared tooltip across panels .

* **time** – The time range for the dashboard. It's an object with "from" and "to" properties defining the default time range (e.g. "from": "now-6h", "to": "now" for last 6 hours) .

* **refresh** – Default auto-refresh interval for the dashboard (e.g. "5s", "1m", etc.). Can be empty or omitted if no auto-refresh is set .

* **timepicker** – An object describing time picker options (see **Time Picker Settings** below) .

* **templating** – An object describing dashboard variables (templating). It contains a list of variable definitions (see **Templating (Variables)** below) .

* **annotations** – An object describing annotation queries configured for this dashboard. It contains a list of annotation definitions (see **Annotations** below) .

* **panels** – Array of panel objects that make up the dashboard (see **Panels** section below). Each object in this array is a panel with its own type and settings .

* **links** – Array of dashboard link objects. These are external or internal links shown on the dashboard (see **Dashboard Links** below). If no links are set, this is an empty array .

* **schemaVersion** – Integer schema version of the dashboard JSON. This increments when the dashboard JSON format changes (e.g. new release with a new field) .

* **version** – Integer revision of the dashboard, incremented each time the dashboard is saved/updated by a user .

* **gnetId** – *Optional.* ID of the dashboard on a community dashboard site if it was imported from there. It is null if the dashboard isn't sourced from a community site .

* **iteration** – *Optional.* An internal revision identifier, often a timestamp value, used internally by the dashboard system. This is distinct from the human-edited version number, and is used to track changes or cache-busting for live dashboards .

* **fiscalYearStartMonth** – *Optional.* The month (0–11) that the fiscal year starts on, used for custom time ranges (e.g. week numbering). 0 \= January, 11 \= December .

* **liveNow** – *Optional.* Boolean indicating if the dashboard is currently in "live" mode (real-time streaming data). When true, the dashboard UI will continuously poll or stream new data (as if the time range is moving forward, like a live tail) .

* **x-navixy** – *Optional vendor extension.* An object containing vendor-specific extensions for dashboard parameters. See **Dashboard Parameters (Dashboard JSON + x-navixy Extension)** section below for details. Unrecognized vendor extensions (fields starting with `x-`) MUST be ignored by consumers for forward compatibility .

## **Panels**

Panels are the building blocks of a dashboard. Each panel is a visualization (graph, gauge, table, text, etc.) with its own queries and settings. In the JSON model, panels are defined in the **"panels"** array, where each entry is a JSON object representing one panel. Most panels share a set of common fields, and some fields are specific to certain panel types.

**Common panel fields include:**

* **id** – Numeric panel ID, unique within the dashboard. This is used for referencing panels (e.g. in annotations). Panel IDs are assigned when panels are created.

* **type** – Panel type (e.g. "graph", "timeseries", "gauge", "table", "text", etc.). This determines what kind of visualization and options the panel will have .

* **title** – Panel title, displayed at the top of the panel .

* **description** – *Optional.* Panel description, shown on hover of info icon in the UI. If provided, stored in JSON as "description": "text..." .

* **gridPos** – An object specifying the panel's size and position in the dashboard grid. It has:

  * x: column position (0–23, since dashboard width is 24 columns) ,

  * y: row position (in grid units) ,

  * w: width in grid columns (1–24) ,

  * h: height in grid units (each unit \~30px) .

     (Panels are laid out on a responsive grid; the dashboard system will automatically adjust positions if there are gaps.)

* **datasource** – The data source for the panel's queries. In newer versions this is often an object with {"type": "\<type\>", "uid": "\<datasource UID\>"}, or it can be set to null to use the default data source. (Older versions used a datasource name string.)

* **targets** – An array of query definitions for this panel. Each target defines a query to the datasource (for example, a PromQL or SQL query) and has its own parameters:

  * refId: Alphabetical ID of the query (e.g. "A", "B", etc.) .

  * Other fields within a target depend on the data source type (e.g., a Prometheus target may have a expr field for the PromQL query; an Elasticsearch target might have query DSL fields). The JSON will include all needed query parameters as nested fields within each target.

  * datasource: Each target can optionally override the panel's data source (similar format as the panel's datasource field) .

* **fieldConfig** – An object defining per-field display settings and overrides. This contains:

  * defaults: Default display options for all series/fields, such as color, units, number of decimals, threshold definitions, etc . For example, under defaults you might see a thresholds object or unit for the unit of measurement.

  * overrides: An array of field override rules to customize specific series/fields beyond the defaults .

* **options** – Panel-type-specific settings. This is an object whose contents depend on the panel **type**. For example, a **time series** chart's options might include settings for legend display, tooltip mode, draw style (line/bar points), etc. A **gauge** panel's options might include orientation or value display settings, and so on. These options correspond to the customization options available for that panel type.

* **pluginVersion** – Version of the panel plugin in use (e.g. "8.5.9" or "10.0.2"). This is set to track which version of the panel was used, for compatibility.

* **transparent** – *Optional.* Boolean indicating if the panel has a transparent background (so the dashboard background shows through). If false or omitted, panel has an opaque background.

* **maxDataPoints** – *Optional.* Maximum data points the panel should plot. The dashboard system might down-sample data for high-resolution displays based on this. Often seen in certain panels like Graph, Gauge, etc., defaulting to 100 or auto.

* **transformations** – *Optional.* An array of transformations applied to the panel's data. Each transformation entry has an id (type of transformation) and an options object defining its settings. (For example, a transform to join series, filter fields, calculate differences, etc., as configured in the UI.)

Each panel type can also include additional fields unique to that type or legacy fields:

* For example, a **Text** panel (type "text") in the JSON above has "mode": "markdown" and a "content" field containing the text/markdown content of the panel .

* An older **Graph** panel (the classic graph, type "graph" in older versions) might have legacy fields like "lines": true, "bars": false, "stack": false, "aliasColors": {} etc., which in newer versions have been superseded by the fieldConfig and options structure. In modern versions (v8+), most visual preferences are under fieldConfig.defaults or options.

* A **Row** panel (type "row") is a special panel that groups other panels. It has a collapsed field and contains child panels in a sub-array when expanded. (Row panels are used for organizing dashboards vertically.)

## **Time Range & Time Picker Settings**

**Dashboard Time Range ("time"):** The time field is an object specifying the default time range of the dashboard. It typically has:

* "from" – the start of the range (e.g. "now-6h" for "now minus 6 hours"), and

* "to" – the end of the range (e.g. "now" for the current time) .

These can be relative (like now-6h, now-7d) or absolute timestamps. The example default is from 6 hours ago to now .

**Auto-Refresh ("refresh"):** The refresh field (at top-level) sets the default refresh interval. For instance, "5s" means the dashboard will refresh every 5 seconds by default . This corresponds to the refresh dropdown in the UI. It can be set to any valid interval string or left blank/null if auto-refresh is off.

**Timezone:** As noted, timezone can be set to "browser" (use the viewer's browser time zone) or "utc" or a specific time zone ID. This affects how the dashboard displays time values.

**Now delay:** In the timepicker settings (below) there is an option to specify a **now delay** (in JSON, nowDelay). This allows shifting the effective "now" to a bit in the past, to accommodate data that arrives late. For example, a nowDelay of 1m would treat "now" as now-1m . (In JSON v1 this appears as timepicker.nowDelay.)

**Hide time picker:** The dashboard system allows the entire time picker UI to be hidden. In JSON, timepicker.hidden: true will hide the time picker in the dashboard UI.

**Time picker ("timepicker"):** The timepicker field is an object that configures the dashboard's time filter UI (the time picker and refresh intervals). Common subfields include:

* **collapse** – Boolean, whether the time picker is in a collapsed (dropdown) state by default .

* **enable** – Boolean, whether the time picker is enabled/shown (usually true; if false, the time picker UI is not shown at all).

* **notice** – (Deprecated/unused in most cases) Boolean for showing a notice. Often left false .

* **now** – Boolean, whether "Now" is a valid option for time range selection (usually true) .

* **hidden** – Boolean, same as *Hide time picker* above; true if the time picker UI should be hidden .

* **nowDelay** – String, a delay to apply to "now" (e.g. "1m", "5m") if set. This effectively shifts what "now" means, to avoid null data from indexing delays .

* **refresh\_intervals** – Array of strings for refresh intervals to display in the UI's refresh dropdown . For example \["5s","10s","30s","1m","5m",...\] defines the options the user can select .

* **time\_options / quick\_ranges** – *Quick ranges* for the time picker's preset list. In JSON, older versions used timepicker.quick\_ranges (an array of objects with display, from, to) to define custom quick range options. These are not commonly customized in JSON exports (defaults are like *Last 5m, 15m, 1h, etc.*) and may not appear unless explicitly set.

* **status** – String field (e.g. "Stable") that may appear in the JSON. This is not typically meaningful for configuration (possibly indicating plugin status).

* **type** – Typically "timepicker" indicating this object's type. (This is mostly an internal marker in JSON.)

**Fiscal Year Start & Week Start:** At top-level, fiscalYearStartMonth can be set to define which month is considered the start of the fiscal year (0-indexed, so 0 = January). This can affect how "Last fiscal year" or similar ranges are calculated. The dashboard system as of schema v1 also assumed weeks start on Monday by default; in schema v2 a weekStart field can exist (with values "monday", "sunday", etc.), but in the default JSON model v1 this is not usually present.

In summary, the *time settings* allow you to control the default time range and refresh rate of the dashboard and how the time picker UI behaves, ensuring the dashboard shows the intended timeframe and provides appropriate quick-select options.

## **Dashboard Parameters (Dashboard JSON + x-navixy Extension)**

This section defines a portable, minimal way to declare dashboard parameters in a dashboard JSON model. It standardizes where defaults and presets live in JSON, how parameters are typed and validated, how parameters are referenced in SQL (via named placeholders), and how runtimes MUST bind values using prepared statements (no string interpolation).

### **Purpose & Scope**

The dashboard parameters specification provides:

* A standardized way to declare dashboard inputs beyond the standard time range
* Type-safe parameter definitions with validation rules
* SQL parameter binding using dashboard template variable syntax (`${variable_name}`)
* Prepared statement execution (mandatory; no string interpolation)
* JSON Schema (2020-12) validation support
* Vendor extension using the `x-...` convention for forward compatibility

The specification is intentionally minimal and production-oriented, favoring existing dashboard fields for time range and adding a single vendor extension (`x-navixy.params`) for non-time parameters.

### **Compatibility & Standards**

The specification follows these standards:

* **Dashboard JSON model** for time range and quick ranges:
  * `time`: default range (`{ "from": "...", "to": "..." }`)
  * `timepicker.quickRanges`: preset list
* **Relative time expressions** use standard semantics (e.g., `now-7d/d`, `now-24h`, `now-1M/M`)
* **Absolute timestamps** are ISO-8601/RFC-3339 (UTC recommended), e.g. `2025-11-06T00:00:00Z`
* **Named SQL parameters** use `${variable_name}` template variable syntax in queries (matches dashboard template variable syntax)
* **Prepared statements** MUST be used at execution (no string concatenation)
* **JSON Schema (2020-12)** shapes are provided for validation
* **Vendor extensions** use the `x-...` convention

### **Where Parameters Live in JSON**

#### **Required (Time Range)**

Use standard dashboard fields for time range:

```json
"time": { "from": "now-7d/d", "to": "now" },
"timepicker": {
  "quickRanges": [
    { "from": "now-24h",  "to": "now",      "display": "Last 24 hours" },
    { "from": "now-7d/d", "to": "now",      "display": "Last 7 days" },
    { "from": "now-30d/d","to": "now",      "display": "Last 30 days" }
  ]
}
```

#### **Extension (Non-Time Parameters)**

Declare additional inputs at the dashboard root using the `x-navixy` vendor extension:

```json
"x-navixy": {
  "params": [
    { "name": "from",       "type": "time",    "label": "From",            "default": "now-7d/d", "required": true },
    { "name": "to",         "type": "time",    "label": "To",              "default": "now",      "required": true },
    { "name": "client_id",  "type": "number",  "label": "Client ID",       "default": null,       "placeholder": "398286" },
    { "name": "move_kph",   "type": "number",  "label": "Move ≥ kph",      "default": 50,         "min": 0 },
    { "name": "idle_kph",   "type": "number",  "label": "Idle ≤ kph",      "default": 1,          "min": 0 },
    { "name": "status",     "type": "select",  "label": "Status",          "options": [
        { "value": "active", "label": "Active" },
        { "value": "idle",   "label": "Idle"   }
      ],
      "default": "active"
    }
  ]
}
```

**Notes:**

* The time range continues to be governed by `time` + `timepicker.quickRanges`. Duplicating `from`/`to` in `x-navixy.params` is allowed for UI uniformity; the runtime MUST reconcile them (see **Execution & Binding Rules** below).
* Unrecognized fields under `x-navixy` MUST be ignored for forward compatibility.

### **Parameter Types & Fields**

#### **Types**

The `type` field must be one of: `"time"`, `"datetime"`, `"number"`, `"integer"`, `"text"`, `"boolean"`, `"select"`, `"multiselect"`.

* **time** – Value is a relative (e.g., `now-7d/d`) or absolute timestamp string; typically used for `from`/`to`.
* **datetime** – A single timestamp (absolute ISO-8601 or resolvable relative).
* **number** / **integer** – Numeric inputs.
* **text** – Free text input.
* **boolean** – `true`/`false` value.
* **select** / **multiselect** – Choose from provided options.

#### **Common Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **name** | string | ✓ | Identifier used in SQL as `${name}`. Lowercase snake_case recommended. Reserved: `__from`, `__to` (for date range). |
| **type** | enum | ✓ | See types above. |
| **label** | string | | Human-readable caption. |
| **description** | string | | Help text. |
| **default** | any | | Initial value (type-sensitive; see below). |
| **required** | boolean | | If `true`, execution MUST fail if missing. |
| **placeholder** | string | | UI hint for empty fields. |
| **order** | number | | UI ordering hint (ascending). |

#### **Type-Specific Fields**

* **number** / **integer**: `min`, `max`, `step`
* **text**: `pattern` (ECMA regex), `format` (e.g., `uuid`, `email`)
* **select** / **multiselect**: `options` (array of `{ value, label }`), `allowCustom` (boolean)

#### **Defaults**

* **time** / **datetime**: Relative (standard syntax) or absolute ISO-8601 string.
* **select**: `default` must be one of `options[*].value`.

### **Execution & Binding Rules**

#### **1. Discovery**

* At runtime, scan each panel's SQL for `${([A-Za-z_][A-Za-z0-9_]*)}` to collect referenced variable names.
* Merge with `x-navixy.params`. Undeclared names become implicit parameters with `type:"text"` and no default.
* **Reserved variables**: `${__from}` and `${__to}` are reserved for dashboard date range and automatically populated from `dashboard.time`.

#### **2. Time Handling**

* The authoritative default time range is `dashboard.time`.
* If `x-navixy.params` includes `from`/`to` with defaults, those SHOULD mirror `dashboard.time`.
* Relative expressions (e.g., `now-7d/d`) MUST be resolved to absolute instants at execution time (rounding semantics: `/d` → start of day, current browser tz unless configured otherwise).

#### **3. Binding**

* Replace each `${variable_name}` with a prepared-statement placeholder appropriate to the dialect (e.g., PostgreSQL `$1..n`, MySQL `?`, ClickHouse `?` or named).
* Resolve `${__from}` and `${__to}` from `dashboard.time` (convert relative expressions like `now-7d/d` to absolute timestamps).
* Provide the values in the correct order.
* Never inline values by string concatenation.

#### **4. Type Mapping (Recommended)**

* `time`/`datetime` → bind as TIMESTAMP (UTC).
* `integer`/`number` → numeric type.
* `boolean` → boolean.
* `text`/`select` → text/varchar.

#### **5. Validation**

* Before execution, validate against the JSON Schema (see below).
* If a required parameter is missing, the runtime MUST surface a clear error.

### **Query Authoring Conventions**

* Use dashboard template variable syntax in SQL:

```sql
WHERE o.client_id = ${client_id}
  AND t.device_time >= ${__from}
  AND t.device_time <  ${__to}
  AND t.speed_kph >= ${move_kph}
```

* **Reserved variables**: `${__from}` and `${__to}` are reserved for dashboard date range and automatically populated from `dashboard.time`.
* Prefer half-open time windows (`>= ${__from} AND < ${__to}`) to avoid double counting at boundaries.
* Keep parameter names consistent with declarations (`client_id`, `move_kph`, etc.).
* This syntax avoids conflicts with PostgreSQL type casts (e.g., `latitude::float`) and matches dashboard template variable conventions.

### **JSON Schema (2020-12) for Validation**

The following JSON Schema can be used to validate dashboard parameter definitions:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.navixy.dev/dashboard-params.schema.json",
  "title": "Dashboard Parameters Extension",
  "type": "object",
  "properties": {
    "time": {
      "type": "object",
      "properties": {
        "from": { "type": "string" },
        "to":   { "type": "string" }
      },
      "required": ["from", "to"]
    },
    "timepicker": {
      "type": "object",
      "properties": {
        "quickRanges": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "from":    { "type": "string" },
              "to":      { "type": "string" },
              "display": { "type": "string" }
            },
            "required": ["from", "to", "display"],
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": true
    },
    "x-navixy": {
      "type": "object",
      "properties": {
        "params": {
          "type": "array",
          "items": { "$ref": "#/$defs/Param" }
        }
      },
      "additionalProperties": true
    }
  },
  "additionalProperties": true,
  "$defs": {
    "Param": {
      "type": "object",
      "properties": {
        "name":  { "type": "string", "pattern": "^[a-z_][a-z0-9_]*$" },
        "type":  { "enum": ["time","datetime","number","integer","text","boolean","select","multiselect"] },
        "label": { "type": "string" },
        "description": { "type": "string" },
        "default": {},
        "required": { "type": "boolean" },
        "placeholder": { "type": "string" },
        "order": { "type": "number" },
        "min": { "type": "number" },
        "max": { "type": "number" },
        "step": { "type": "number" },
        "pattern": { "type": "string" },
        "format": { "type": "string" },
        "options": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "value": {},
              "label": { "type": "string" }
            },
            "required": ["value"],
            "additionalProperties": false
          }
        },
        "allowCustom": { "type": "boolean" }
      },
      "required": ["name","type"],
      "additionalProperties": false,
      "allOf": [
        {
          "if": { "properties": { "type": { "const": "integer" } } },
          "then": { "properties": { "default": { "type": "integer" } } }
        }
      ]
    }
  }
}
```

**Validation Notes:**

* We intentionally allow `time`/`from`/`to` strings to be either relative or absolute. Runtimes MUST resolve relative strings at execution.
* JSON strings must not contain literal newlines (escape as `\n`).

### **Minimal End-to-End Example**

**Dashboard JSON (abridged):**

```json
{
  "title": "Asset Utilization & Idling",
  "time": { "from": "now-7d/d", "to": "now" },
  "timepicker": {
    "quickRanges": [
      { "from": "now-24h",  "to": "now", "display": "Last 24 hours" },
      { "from": "now-7d/d", "to": "now", "display": "Last 7 days" }
    ]
  },
  "x-navixy": {
    "params": [
      { "name": "from", "type": "time", "label": "From", "default": "now-7d/d", "required": true },
      { "name": "to",   "type": "time", "label": "To",   "default": "now",      "required": true },
      { "name": "client_id", "type": "number", "label": "Client ID", "default": null, "placeholder": "398286" },
      { "name": "move_kph",  "type": "number", "label": "Move ≥ kph", "default": 50 },
      { "name": "idle_kph",  "type": "number", "label": "Idle ≤ kph", "default": 1 }
    ]
  }
}
```

**Panel SQL:**

```sql
SELECT o.object_id, SUM(t.odom_delta_km) AS km
FROM raw_telematics_data.tracking_data_core t
JOIN raw_business_data.objects o ON o.device_id = t.device_id
WHERE o.client_id = ${client_id}
  AND t.device_time >= ${__from}
  AND t.device_time <  ${__to}
  AND t.speed_kph >= ${move_kph}
GROUP BY 1
ORDER BY 2 DESC;
```

### **Security, UX, and Interop Guidance**

#### **Security**

* Always use prepared statements for `${variable_name}` parameters.
* Enforce type at runtime (e.g., reject non-numeric for `number`).
* Normalize time to UTC before binding.
* Never use string concatenation or interpolation; always bind via prepared statements.

#### **UX**

* Show a parameter bar below the title.
* Hydrate date presets from `timepicker.quickRanges`.
* Provide "Reset to defaults" (reloads default values and time).

#### **Interop**

* Unknown fields MUST be ignored by consumers.
* Dashboards without `x-navixy` remain valid; only time is honored.

#### **Versioning**

* Optionally include `"x-navixy": { "version": "1.0", "params": [...] }` for future evolution.

### **Migration Rules**

1. **Existing dashboards** require no changes; they already provide `time` and (optionally) `quickRanges`.
2. **To enable non-time inputs**, add `x-navixy.params`.
3. **Queries should use `${__from}`/`${__to}`** for date range (reserved variables automatically populated from `dashboard.time`).

### **Summary**

* Use the native `time` + `timepicker.quickRanges` for ranges and presets.
* Declare extra inputs in `x-navixy.params` with strict types and defaults.
* Reference parameters in SQL as `${variable_name}` (matches dashboard template variable syntax).
* Use `${__from}` and `${__to}` for date range (reserved, automatically populated from `dashboard.time`).
* Bind all parameters via prepared statements (never string interpolation).
* Validate with the JSON Schema above; resolve relative time at execution.

This keeps dashboards portable, safe, and familiar—while adding just enough structure to support robust parameterized BI queries.

## **Templating (Variables)**

The templating field holds the dashboard's template variables (also known as *dashboard variables*). Variables allow dynamic filtering in dashboards (e.g., select server, region, etc.). In the JSON model, templating has the structure:

```
"templating": {
  "enable": true,
  "list": [ ... variable definitions ... ]
}
```

If enable is false, the variables are present but not actively used (this is rarely disabled; usually true by default) . The **list** is an array of variable objects. Each variable object can have the following fields (common ones are listed):

* **name** – The variable name (unique identifier) used in queries (e.g. $env). This is the actual variable key .

* **label** – *Optional.* Human-friendly label for the variable (shown in the dropdown). If not set, the name is used as the label.

* **type** – Type of variable, such as "query" (values come from a data source query), "interval" (time interval list), "custom" (fixed list of values), "constant", etc. .

* **query** – For query variables, the query string used to fetch options (syntax depends on data source). For example, a Prometheus query variable might have a PromQL query here, or for Loki a log query, etc. . In the example above, "query": "tag\_values(cpu.utilization.average, env)" – a query to fetch values for the env tag .

* **datasource** – The data source used for the variable's query. Often null means use the default data source, or it can be set to a specific data source name/UID .

* **options** – Array of value options (each option is an object with text and value, and a selected boolean). The dashboard system populates this list with all available values (or a subset) after running the query. For custom variables, this list is defined explicitly. One of the options can represent the "All" value (if applicable).

* **current** – An object representing the current selected value(s) for the variable. It typically has text (the display text) and value (the actual value) and possibly a tags array for any tag selections. The dashboard system sets this when you save a dashboard with a selected variable value.

* **includeAll** – Boolean indicating if an "All" option is available for this variable (when true, an option representing "all values" is added) .

* **allValue** – *Optional.* If "All" is enabled, this can define what value to use when "All" is selected (e.g. a wildcard or regex .\*). If not set, the dashboard system will compose one (like {"value": "$\_\_all"} meaning match all).

* **allFormat** – Format used when the "All" option is selected. Common values: wildcard, regex, glob, pipe (how the "all" value is interpreted in queries) .

* **multi** – Boolean indicating if multiple values can be selected (multi-select) .

* **multiFormat** – Format to use when multiple values are selected (e.g., how to join them in queries: glob, wildcard, regex) .

* **regex** – *Optional.* For query variables, a regex pattern to filter or capture part of the results from the query. If set, the dashboard system will filter the query results using this regex.

* **refresh** – When to update the variable options: 0 or false \= never refresh automatically, 1 \= refresh on time range change, 2 \= refresh on dashboard load. (In JSON exports, false might appear instead of 0\) .

* **description** – *Optional.* A description for the variable (usually shown as info on hover in the UI). Not present if not set .

In the JSON example above, two query variables are defined (env and app). The env variable includes an "All" option (includeAll: true) with allFormat: "wildcard" and its query fetches distinct env tag values . The app variable does not allow all or multi-select (includeAll false, multi false) and has its own query for app tag values .

Using these variables, panels can reference them with $name syntax, and the dashboard system will substitute the selected value in the query.

## **Annotations**

Annotations provide a way to mark events or highlight regions on graphs (shown as vertical lines or shaded regions with tooltips). In the JSON model, the annotations field contains an object with a **"list"** of annotation definition objects. By default, dashboards have one built-in annotation query (for user-created dashboard annotations and alerts).

Each annotation definition in the list can have the following fields:

* **builtIn** – If present and non-zero, indicates a built-in annotation source. The default **"Annotations & Alerts"** query has "builtIn": 1, meaning it shows any alert or user annotation on the dashboard.

* **name** – Name of the annotation query entry (as it appears in the dashboard UI settings).

* **datasource** – Data source from which to pull annotations. For the built-in dashboard annotation, this is the internal data source ("type": "dashboard", "uid": "-- Dashboard --"). For custom annotations (e.g., from other sources), this would reference the data source by UID.

* **enable** – Boolean to enable/disable this annotation query .

* **hide** – Boolean indicating if the annotations from this source should be hidden by default. The built-in one is often set to true (so that user annotations don't all show unless toggled) .

* **iconColor** – Color for the annotation marker (vertical line icon color) in RGBA or hex format (e.g., "rgba(0, 211, 255, 1)" for the default light blue) .

* **type** – Type of annotation query. For the built-in it is "dashboard" (meaning it draws from the internal annotation storage for this dashboard, and alerts). Another common type is "tags" when pulling annotations by tags from the dashboard database.

* **target** – *Optional, for certain types.* An object defining the query for annotations when using tags or an external source. For example, for tag-based annotations:

  * target.type: "tags" and a tags array to match .

  * limit: max number of annotations to fetch .

  * matchAny: boolean whether to match any tag vs all tags.

     (For built-in type "dashboard", the target might not be needed or is set to the current dashboard by default. In the default JSON above, target is present with "type": "dashboard", "tags": \[\], "limit": 100, "matchAny": false as default .)

* **query** – *Optional.* In some cases (especially when annotations come from data sources like Loki or Elasticsearch), there might be a query field or similar inside target or at top-level to define how to fetch annotation events. (The JSON for those can vary depending on the data source plugin and is beyond the core schema.)

In summary, each annotation entry defines a source of events to show on graphs. The default one is built-in and shows any manually added dashboard annotation or alert state changes (hence the name "Annotations & Alerts") . You can add custom ones, for example to fetch deployment events from an external source – those would appear as additional objects in this list with their own settings (name, data source, target tags or queries) .

## **Dashboard Links**

Dashboard links are used to provide clickable links or drop-down menus on a dashboard, linking to other dashboards or external URLs for context. In the JSON model, the links field is an array of link objects . If no links are configured, it will be an empty array. Each link object can define either a direct link or a special **"Dashboard link"** that references other dashboards by tag.

Key fields for each link object include:

* **title** – The text to display for the link .

* **type** – The link type. Possible values:

  * "dashboards" – This link will open a list of dashboards (filtered by tags) .

  * "link" – A direct link to an external URL or absolute/relative dashboard URL.

* **url** – *Required if* type is "link". The URL to open when the link is clicked . (Not used for type: "dashboards".)

* **tags** – *For type: "dashboards".*\* An array of tag names to filter which dashboards are shown in the drop-down. If this is empty, the dashboard system will list **all** dashboards the user has access to. If tags are specified, only dashboards tagged with any of those will be listed.

* **asDropdown** – Boolean, used only for type: "dashboards". If true, multiple dashboards matching the tags will appear as a dropdown menu under one link title. If false, the dashboard system will create one link per dashboard (when multiple dashboards match) displayed separately. Default is false (but generally for multiple dashboards, you'd set true to avoid clutter).

* **icon** – Name of an icon to display with the link (e.g. external link, bolt, etc.). If set to an empty string or not provided, no icon is shown .

* **tooltip** – Hover text for the link (displayed when user hovers over the link) .

* **targetBlank** – Boolean, if true the link opens in a new tab/window (i.e. target="\_blank"). Default false .

* **includeVars** – Boolean, if true, the dashboard system will include the current dashboard's variable values as URL parameters when navigating via this link. This is useful to carry over context to the target dashboard or link.

* **keepTime** – Boolean, if true, the current time range is included in the link (as URL params) so that the target page sees the same time window. Default false .

* **dashboard** – *For older versions / alternate format.* In some older JSON or when type is "dashboards", you might see a dashboard field (with a dashboard name or UID) if linking a specific dashboard directly. However, in modern use, one would use type:"link" with a url that is a dashboard URL, or use type:"dashboards" with tags.

An example of a link in JSON:

```
{
  "title": "Production Overview",
  "tooltip": "Go to Production cluster overview",
  "type": "link",
      "url": "https://dashboard.mycompany.com/d/abcd1234/production-overview",
  "icon": "external link",
  "targetBlank": true
}
```

This would show a link "Production Overview" with an external link icon, and clicking it opens the given URL in a new tab .

And an example of a dashboards dropdown link:

```
{
  "title": "Team Dashboards",
  "type": "dashboards",
  "tags": ["team-X"],
  "asDropdown": true
}
```

This would create a single link titled "Team Dashboards" that, when clicked, drops down a list of all dashboards tagged "team-X" .

By using dashboard links, you can guide viewers to related dashboards or external systems (like runbooks, external analytics pages, etc.) directly from a dashboard.

---

**References:** This documentation covers the standard fields and structure of the dashboard JSON model. It describes the standard fields and structure that should be preserved for compatibility with the dashboard JSON format.

