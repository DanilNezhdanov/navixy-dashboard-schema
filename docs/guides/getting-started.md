# Getting Started with Dashboards

This guide will help you set up dashboards and create your first dashboard.

## Prerequisites

- Basic understanding of monitoring concepts
- Access to data sources (Prometheus, InfluxDB, MySQL, etc.)
- Dashboard runtime (Navixy-compatible or compatible dashboard renderer)

## Installation Options

### Option 1: Cloud Dashboard Service (Recommended for beginners)
1. Sign up for a dashboard service
2. Create a new dashboard workspace
3. Follow the setup wizard

### Option 2: Self-Hosted Installation

#### Docker
```bash
docker run -d --name=dashboard -p 3000:3000 navixy/dashboard
```

## Initial Setup

1. **Access Dashboard**: Navigate to your dashboard URL (default: `http://localhost:3000`)
2. **Login**: Use your credentials
3. **Add Data Source**: Go to Configuration → Data Sources

## Adding Your First Data Source

### Prometheus Example
1. Click "Add data source"
2. Select "Prometheus"
3. Enter URL: `http://localhost:9090`
4. Click "Save & Test"

### InfluxDB Example
1. Click "Add data source"
2. Select "InfluxDB"
3. Enter URL: `http://localhost:8086`
4. Configure database and credentials
5. Click "Save & Test"

## Creating Your First Dashboard

1. **Create Dashboard**: Click "+" → "Dashboard"
2. **Add Panel**: Click "Add panel"
3. **Configure Query**: Select your data source and write a query
4. **Customize Visualization**: Choose chart type, colors, and formatting
5. **Save Dashboard**: Click "Save" and give it a name

## Basic Query Examples

### Prometheus Queries
```promql
# CPU usage
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)
```

### InfluxDB Queries
```sql
SELECT mean("value") FROM "cpu_usage" WHERE $timeFilter GROUP BY time($__interval) fill(null)
```

## Next Steps

- Explore the [Dashboard Design Guide](dashboard-design.md)
- Check out [Examples](../examples/) for specific use cases
- Learn about [Alerting](alerting.md) for proactive monitoring
- Review [Best Practices](performance.md) for optimization

## Common Issues

### Data Source Connection Failed
- Verify the data source URL is accessible
- Check firewall settings
- Ensure credentials are correct

### No Data Showing
- Verify time range settings
- Check query syntax
- Ensure data source has data for the selected time range

### Dashboard Loading Slowly
- Reduce query complexity
- Increase refresh interval
- Limit number of panels

## Resources

- [Navixy Format Documentation](../../navixy/README.md)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [JSON Schema Specification](https://json-schema.org/)
