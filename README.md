# Navixy Dashboard Documentation

A comprehensive documentation repository for building and managing dashboards using the Navixy dashboard format.

## Overview

This repository contains documentation, examples, and best practices for creating effective dashboards. Whether you're monitoring infrastructure, applications, or business metrics, this guide will help you build powerful visualizations and alerts.

The repository includes extensive documentation for the **Navixy dashboard format** - a format that enables safe SQL execution with dataset contracts and multi-tenant support.

## Table of Contents

- [Getting Started](#getting-started)
- [Dashboard Design Principles](#dashboard-design-principles)
- [Configuration Examples](#configuration-examples)
- [Navixy Format Documentation](#navixy-format-documentation)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Getting Started

### Prerequisites

- Navixy dashboard runtime or compatible dashboard renderer
- Data sources configured (PostgreSQL, MySQL, etc.)
- Basic understanding of SQL queries

### Quick Setup

1. Clone this repository:
   ```bash
   git clone https://github.com/DanilNezhdanov/navixy-dashboard-schema-v1.git
   ```
2. Review the configuration examples in `/docs/examples/`
3. Import dashboard templates from `/docs/templates/`
4. Customize dashboards for your specific use case

## Dashboard Design Principles

### 1. User-Centric Design
- Design dashboards for specific user roles
- Keep relevant information above the fold
- Use consistent color schemes and layouts

### 2. Performance Optimization
- Limit query complexity
- Use appropriate time ranges
- Implement efficient refresh intervals

### 3. Accessibility
- Use high contrast colors
- Provide meaningful tooltips
- Include descriptive panel titles

## Configuration Examples

Browse the `/docs/examples/` directory for:
- Infrastructure monitoring dashboards
- Application performance monitoring
- Business metrics visualization
- Alert configuration templates

## Best Practices

- **Panel Organization**: Group related metrics logically
- **Time Ranges**: Set appropriate default time ranges
- **Refresh Rates**: Balance real-time needs with performance
- **Annotations**: Use annotations to mark important events
- **Variables**: Implement template variables for flexibility

## Troubleshooting

Common issues and solutions:
- Dashboard loading slowly
- Missing data points
- Alert configuration problems
- Permission and access issues

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Navixy Format Documentation

The Navixy dashboard format provides a comprehensive JSON-based dashboard model with vendor-specific extensions for safe SQL execution.

### Key Features
- **Safe SQL Execution**: Prepared statements with parameter binding
- **Dataset Validation**: Type-safe data contracts for visualizations
- **Multi-tenant Support**: Built-in tenant isolation and security
- **Performance Optimization**: Query deduplication and caching
- **Extensible Visualizations**: Support for custom panel types
- **Standard JSON Model**: Well-defined dashboard structure

### Documentation Structure
- [`navixy/README.md`](navixy/README.md) - Complete format documentation
- [`navixy/schema/`](navixy/schema/) - JSON Schema definitions
- [`navixy/examples/`](navixy/examples/) - Working dashboard examples
- [`navixy/api-reference/`](navixy/api-reference/) - SQL Execution API reference
- [`navixy/security/`](navixy/security/) - Security guidelines and best practices

### Quick Start
1. Review the [Navixy Format Documentation](navixy/README.md)
2. Explore the [Example Dashboards](navixy/examples/)
3. Check the [API Reference](navixy/api-reference/README.md) for implementation details
4. Follow [Security Best Practices](navixy/security/README.md)

## Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [JSON Schema Specification](https://json-schema.org/)
