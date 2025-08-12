# 🚀 Reducing Prometheus Load on Astronomer Software

## Overview
Running Prometheus on-cluster can consume significant resources, especially with the default 15-day retention policy using 100Gi of PVC storage. This guide presents two strategies to optimize resource usage while maintaining observability.

## Strategy 1: Enable Task Usage Metrics
Task usage metrics provide built-in monitoring capabilities that can reduce your dependency on Prometheus metrics.

### Configuration
Add the following to your `values.yaml`:

```yaml
global:
  taskUsageMetricsEnabled: true
```

Once enabled, Astronomer Software collects task usage metrics hourly (at minute 57) and stores them for 90 days by default.

### Accessing Metrics
Navigate through the Software UI based on your scope:

| **Scope** | **Navigation Path** | **Metrics Displayed** |
|-----------|-------------------|---------------------|
| Installation | System Admin → Task Usage | Total runs + per-workspace breakdown |
| Workspace | Select Workspace → Task Usage | Total runs + per-deployment breakdown |
| Deployment | Workspace → Deployment → Task Usage | Total runs + daily breakdown |

## Strategy 2: Optimize Prometheus Configuration

### Reduce Retention Period
The default 15-day retention can be drastically reduced:
- **Minimum recommended**: 1 day for basic deployment health monitoring
- **Resource savings**: Significant reduction from 100Gi PVC usage

### Implement Remote Write
Offload metrics to an external storage system to minimize local resource consumption.

#### Configuration
```yaml
global:
  prometheusEnabled: true

prometheus:
  remote_write:
    - url: https://<endpoint>/api/v1/write
      bearer_token: "<token>"
      queue_config:
        max_shards: 4
        capacity: 1000
```

#### How Remote Write Works
- **Architecture**: Prometheus server handles remote write (not exporters)
- **Flow**: Scrape targets → Write to local TSDB (WAL) → Async batch send to remote endpoint
- **Resilience**: WAL-backed queue retries if remote endpoint is unavailable
- **Local behavior**: Queries and alerts still use local TSDB unless remote_read is configured
- **Data governance**: Local retention policies continue to apply for on-disk data

## Best Practices
- Enable task usage metrics first to assess if it meets your monitoring needs  
- Start with reduced Prometheus retention (1-3 days) and adjust based on requirements  
- Consider remote write for long-term metric storage without cluster load  
- Monitor the impact of changes on your observability workflows

