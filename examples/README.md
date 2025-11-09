# Navixy Dashboard Examples

This directory contains practical examples of Navixy dashboards demonstrating various use cases and panel types.

## Examples Overview

### Complete Dashboards
- [`dashboard-parameters-example.json`](dashboard-parameters-example.json) - **NEW**: Comprehensive example showcasing Dashboard Parameters with `x-navixy.params` extension. Demonstrates typed parameters, validation, defaults, and SQL parameter binding.
- [`fleet-status-dashboard.json`](fleet-status-dashboard.json) - Complete fleet monitoring dashboard (includes both legacy bindings and new params format)
- [`hello-world-dashboard.json`](hello-world-dashboard.json) - Simple getting started example (includes both legacy bindings and new params format)

## Dashboard Parameters

### New: x-navixy.params Extension

The Dashboard Parameters feature (`x-navixy.params`) provides a standardized way to declare user-facing parameters with type safety, validation, and defaults. This complements the standard dashboard time range settings.

**Key Features:**
- **Type-safe parameters**: Define parameters with types (`time`, `datetime`, `number`, `integer`, `text`, `boolean`, `select`, `multiselect`)
- **Validation**: Set min/max values, patterns, and required flags
- **Defaults**: Provide sensible default values for all parameters
- **SQL binding**: Parameters are automatically bound to SQL queries using `${variable_name}` template variable syntax (matches dashboard conventions)
- **Prepared statements**: All parameter binding uses prepared statements (no string interpolation)

**Example:**
```json
"x-navixy": {
  "params": [
    {
      "name": "client_id",
      "type": "number",
      "label": "Client ID",
      "default": null,
      "required": false,
      "placeholder": "398286",
      "min": 1
    },
    {
      "name": "move_kph",
      "type": "number",
      "label": "Move ≥ kph",
      "default": 50,
      "min": 0,
      "max": 200,
      "step": 5
    },
    {
      "name": "status",
      "type": "select",
      "label": "Status",
      "default": "active",
      "options": [
        { "value": "active", "label": "Active" },
        { "value": "idle", "label": "Idle" }
      ]
    }
  ]
}
```

**SQL Usage:**
```sql
SELECT * FROM tracking_data
WHERE client_id = ${client_id}
  AND speed_kph >= ${move_kph}
  AND status = ${status}
```

**Reserved Variables:**
- `${__from}` and `${__to}` are reserved for dashboard date range and automatically populated from `dashboard.time`

See [`dashboard-parameters-example.json`](dashboard-parameters-example.json) for a complete working example.

### Legacy: parameters.bindings

Older examples use `x-navixy.parameters.bindings` to map dashboard variables to SQL parameters. This approach is still supported for backward compatibility, but the new `x-navixy.params` format is recommended for new dashboards.

**Migration:**
- Existing dashboards continue to work with `parameters.bindings`
- New dashboards should use `x-navixy.params` for better type safety and validation
- Both formats can coexist during migration

## Usage Instructions

### Importing Examples
1. Copy the JSON content from any example file
2. Validate against the Navixy schema
3. Configure your SQL execution endpoint
4. Update parameter definitions (`x-navixy.params`) or bindings (`x-navixy.parameters.bindings`) for your environment
5. Test with your data source

### Customization
- Modify SQL statements for your database schema
- Update parameter types, defaults, and constraints in `x-navixy.params`
- Adjust dataset shapes for your visualization needs
- Configure verification rules for data validation

## Example Categories

### Fleet Management
- Vehicle tracking and status
- Event monitoring and alerts
- Performance metrics
- Geographic data visualization

### Business Intelligence
- Sales and revenue tracking
- User analytics and engagement
- Operational metrics
- Trend analysis

### Infrastructure Monitoring
- System resource utilization
- Service health monitoring
- Performance metrics
- Alert management

## Best Practices Demonstrated

### Security
- Prepared statements with parameter binding (no string interpolation)
- Type-safe parameter validation at runtime
- Tenant isolation with tenant_id requirements
- Read-only query enforcement
- Schema allowlisting

### Performance
- Query optimization techniques
- Appropriate row limits
- Timeout configuration
- Caching strategies

### Data Validation
- Type-safe parameter definitions with `x-navixy.params`
- Parameter validation (min/max, patterns, required flags)
- Dataset shape verification
- Row count constraints
- Column requirement validation

### Parameter Design
- Use descriptive parameter names (snake_case recommended)
- Provide sensible defaults for better UX
- Set appropriate min/max constraints for numeric parameters
- Use `select` type for fixed option sets
- Mark required parameters explicitly
- Order parameters logically using the `order` field

## Testing Examples

### Unit Testing
Each example includes test cases for:
- SQL injection prevention
- Parameter type validation
- Dataset shape verification
- Error handling scenarios

### Integration Testing
Examples demonstrate:
- End-to-end data flow
- Multi-tenant data isolation
- Performance under load
- Error recovery mechanisms

## Parameter Types Reference

### Supported Types

| Type | Description | Example Default | Validation |
|------|-------------|----------------|------------|
| `time` | Time range (relative or absolute) | `"now-7d/d"` | Standard time syntax |
| `datetime` | Single timestamp | `"2025-01-01T00:00:00Z"` | ISO-8601 format |
| `number` | Floating-point number | `50.5` | `min`, `max`, `step` |
| `integer` | Whole number | `100` | `min`, `max` |
| `text` | Free text | `"example"` | `pattern`, `format` |
| `boolean` | True/false | `true` | - |
| `select` | Single selection | `"active"` | Must match `options[*].value` |
| `multiselect` | Multiple selection | `["a", "b"]` | Array of `options[*].value` |

### Common Fields

- `name` (required): Parameter identifier used in SQL (`${name}`). Note: `${__from}` and `${__to}` are reserved for date range.
- `type` (required): Parameter type from the list above
- `label`: Human-readable label for UI
- `description`: Help text for users
- `default`: Initial value (type-sensitive)
- `required`: If `true`, execution fails if missing
- `placeholder`: UI hint for empty fields
- `order`: UI ordering hint (ascending)

### Type-Specific Fields

- **number/integer**: `min`, `max`, `step`
- **text**: `pattern` (ECMA regex), `format` (e.g., `uuid`, `email`)
- **select/multiselect**: `options` (array of `{ value, label }`), `allowCustom` (boolean)

## Contributing Examples

When adding new examples:
1. Follow the existing naming convention
2. Use `x-navixy.params` for parameter definitions (preferred over legacy bindings)
3. Include comprehensive documentation
4. Test with sample data
5. Validate against the JSON schema
6. Include security considerations
7. Document any special requirements
8. Demonstrate proper parameter usage in SQL queries
