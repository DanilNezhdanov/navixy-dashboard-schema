# Infrastructure Monitoring Dashboard

A comprehensive dashboard for monitoring server infrastructure including CPU, memory, disk, and network metrics.

## Overview

This dashboard provides a complete view of your infrastructure health with:
- System resource utilization
- Network performance metrics
- Disk usage and I/O statistics
- Service status indicators
- Alert notifications

## Metrics Included

### CPU Metrics
- CPU usage percentage per core
- Load average
- CPU temperature (if available)
- Process count

### Memory Metrics
- Total memory usage
- Available memory
- Swap usage
- Memory pressure indicators

### Disk Metrics
- Disk usage percentage
- Disk I/O rates
- Disk latency
- Available space

### Network Metrics
- Network interface utilization
- Packet rates
- Error rates
- Connection counts

## Configuration

### Data Source Requirements
- Prometheus with node_exporter
- Or InfluxDB with Telegraf

### Required Labels/Tags
- `instance`: Server identifier
- `job`: Exporter job name
- `device`: Disk device name
- `interface`: Network interface name

## Customization

### Variables
- `$instance`: Server selection
- `$job`: Exporter job selection
- `$interval`: Query interval

### Thresholds
- CPU: Warning at 70%, Critical at 90%
- Memory: Warning at 80%, Critical at 95%
- Disk: Warning at 80%, Critical at 95%
- Network: Warning at 80% utilization

## Alert Rules

### Critical Alerts
- CPU usage > 90% for 5 minutes
- Memory usage > 95% for 5 minutes
- Disk usage > 95% for 5 minutes
- Service down for 1 minute

### Warning Alerts
- CPU usage > 70% for 10 minutes
- Memory usage > 80% for 10 minutes
- Disk usage > 80% for 10 minutes
- High network error rate

## Installation

1. Import the dashboard JSON file
2. Configure your data source
3. Update queries for your environment
4. Set up alert rules
5. Customize thresholds as needed

## Maintenance

- Review metrics monthly
- Update thresholds based on usage patterns
- Add new servers to monitoring
- Review and update alert rules
