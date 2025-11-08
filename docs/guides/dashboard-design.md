# Dashboard Design Best Practices

Creating effective dashboards requires careful consideration of user needs, data presentation, and visual design principles.

> **Note**: This guide references the standard dashboard JSON model. For complete field documentation, refer to the [Dashboard JSON Model documentation](dashboard-json-model.md) and the [Navixy Format Documentation](../../navixy/README.md).

## Design Principles

### 1. User-Centric Design
- **Know Your Audience**: Design for specific user roles (developers, operations, executives)
- **Single Purpose**: Each dashboard should serve one primary use case
- **Progressive Disclosure**: Show overview first, details on demand

### 2. Visual Hierarchy
- **Most Important First**: Place critical metrics at the top
- **Logical Grouping**: Group related metrics together
- **Consistent Layout**: Use consistent spacing and alignment

### 3. Information Density
- **Above the Fold**: Keep critical information visible without scrolling
- **Balanced Density**: Not too sparse, not too cluttered
- **White Space**: Use whitespace to separate sections

## Layout Guidelines

### Panel Organization
```
┌─────────────────────────────────────────┐
│              Header/Title               │
├─────────────────────────────────────────┤
│  Critical Alerts  │  System Status     │
├─────────────────────────────────────────┤
│  Primary Metrics (Row 1)                │
├─────────────────────────────────────────┤
│  Secondary Metrics (Row 2)              │
├─────────────────────────────────────────┤
│  Detailed Metrics (Row 3)               │
└─────────────────────────────────────────┘
```

### Panel Sizing
- **Large Panels**: For primary metrics and trends
- **Medium Panels**: For secondary metrics
- **Small Panels**: For status indicators and counters

## Color and Styling

### Color Palette
- **Consistent Colors**: Use the same color for the same metric across dashboards
- **Semantic Colors**: 
  - Green: Good/Normal states
  - Yellow: Warning states
  - Red: Critical/Error states
  - Blue: Information/Neutral states

### Typography
- **Clear Labels**: Use descriptive panel and axis titles
- **Readable Fonts**: Ensure text is legible at all sizes
- **Consistent Sizing**: Use consistent font sizes throughout

## Data Visualization

### Chart Types
- **Time Series**: For metrics that change over time
- **Gauges**: For current values with thresholds
- **Tables**: For detailed data and logs
- **Stat Panels**: For single values and counters
- **Heatmaps**: For distribution analysis

### Time Ranges

The `time` field sets the default time range:

```json
{
  "time": {
    "from": "now-6h",
    "to": "now"
  }
}
```

- **Default Range**: Set appropriate default time ranges using the `time` field
- **Refresh Interval**: Configure auto-refresh with the `refresh` field (e.g., `"refresh": "30s"`)
- **Time Picker**: Configure time picker options with the `timepicker` field:
  - `refresh_intervals`: Available refresh intervals
  - `time_options`: Quick range options (e.g., "Last 5m", "Last 15m")
  - `nowDelay`: Delay for "now" to account for indexing delays
- **Timezone**: Set timezone with `timezone` field ("browser", "utc", or IANA timezone ID)
- **Custom Ranges**: Allow users to set custom ranges via the time picker UI

## Performance Considerations

### Query Optimization
- **Efficient Queries**: Write optimized queries for your data source
- **Appropriate Intervals**: Use suitable query intervals
- **Data Sampling**: Consider data sampling for large datasets

### Refresh Rates
- **Real-time**: 5-10 seconds for critical metrics
- **Near Real-time**: 30-60 seconds for operational metrics
- **Batch**: 5-15 minutes for business metrics

## Accessibility

### Visual Accessibility
- **High Contrast**: Ensure sufficient color contrast
- **Color Blindness**: Don't rely solely on color to convey information
- **Text Alternatives**: Provide text descriptions for visual elements

### Navigation
- **Keyboard Navigation**: Ensure keyboard accessibility
- **Screen Readers**: Use semantic HTML and ARIA labels
- **Focus Indicators**: Provide clear focus indicators

## Dashboard JSON Model Structure

When creating dashboards, ensure proper structure according to the dashboard JSON model:

### Required Fields
- `title`: Dashboard title (required)
- `time`: Time range object with `from` and `to` (required)
- `panels`: Array of panel objects (required)

### Common Configuration Fields
- `uid`: Unique identifier (8-40 characters)
- `description`: Dashboard description
- `tags`: Array of tag strings for organization
- `style`: Theme ("dark" or "light")
- `timezone`: Time zone setting ("browser", "utc", or IANA timezone)
- `editable`: Boolean for edit permissions
- `graphTooltip`: Tooltip behavior (0, 1, or 2)
- `refresh`: Auto-refresh interval string
- `schemaVersion`: Integer schema version
- `version`: Integer revision number

### Templating (Variables)
Configure dashboard variables in the `templating` field:

```json
{
  "templating": {
    "enable": true,
    "list": [
      {
        "name": "var_server",
        "type": "query",
        "query": "SELECT server FROM servers"
      }
    ]
  }
}
```

### Annotations
Configure annotations in the `annotations` field:

```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "name": "Annotations & Alerts",
        "type": "dashboard",
        "enable": true
      }
    ]
  }
}
```

### Dashboard Links
Add dashboard links in the `links` field:

```json
{
  "links": [
    {
      "title": "Related Dashboard",
      "type": "link",
      "url": "/d/abc123/related-dashboard",
      "targetBlank": true
    }
  ]
}
```

### Panel Structure
Each panel should include:
- `id`: Unique panel ID (numeric)
- `type`: Panel type (e.g., "graph", "timeseries", "table")
- `title`: Panel title
- `gridPos`: Position and size (`x`, `y`, `w`, `h`)

## Common Patterns

### Infrastructure Dashboard
- System overview at the top
- Resource utilization metrics
- Service health indicators
- Recent alerts and events
- Use `tags` field for organization: `["infrastructure", "monitoring"]`

### Application Dashboard
- Application performance metrics
- Error rates and response times
- User activity and engagement
- Business metrics correlation
- Configure `timepicker.refresh_intervals` for appropriate refresh rates

### Business Dashboard
- Key performance indicators
- Trend analysis
- Comparative metrics
- Goal tracking
- Set longer `refresh` intervals: `"refresh": "5m"` or `"refresh": "15m"`

## Anti-Patterns to Avoid

### Information Overload
- Too many panels on one dashboard
- Overly complex visualizations
- Irrelevant metrics for the audience

### Poor Organization
- Random panel placement
- Inconsistent naming conventions
- Missing or unclear labels

### Performance Issues
- Complex queries causing slow loading
- Too frequent refresh rates
- Large datasets without optimization

## Testing and Validation

### User Testing
- Test with actual users
- Gather feedback on usability
- Iterate based on feedback

### Performance Testing
- Monitor dashboard load times
- Test with realistic data volumes
- Optimize based on performance metrics

## Maintenance

### Regular Reviews
- Review dashboard relevance quarterly
- Update queries and visualizations
- Remove unused or outdated panels

### Documentation
- Document dashboard purpose and audience
- Include query explanations
- Maintain change logs

## Tools and Resources

### Design Tools
- Dashboard's built-in panel editor
- External design tools for mockups
- Color palette generators

### Testing Tools
- Browser developer tools
- Performance monitoring
- Accessibility testing tools

## Conclusion

Effective dashboard design is an iterative process that requires understanding user needs, following design principles, and continuous improvement. Start with simple designs and gradually add complexity based on user feedback and requirements.
