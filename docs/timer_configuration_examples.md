# Timer Configuration Examples

## Overview

The EDA plugin now supports configurable timer parameters for more flexible record processing. This allows you to tune the system based on your specific requirements.

## Configuration Parameters

### `timer_update_interval`
- **Purpose**: How often the consideration timer is updated
- **Default**: `"1h"` (1 hour)
- **Format**: Time strings like `"1h"`, `"30m"`, `"3600s"`, `"2h30m"`

### `timer_buffer_time`
- **Purpose**: Buffer time to account for slow processing
- **Default**: `"5m"` (5 minutes)
- **Format**: Time strings like `"5m"`, `"300s"`, `"1h5m"`

## Example Configurations

### 1. Default Configuration
```yaml
# Uses default values: 1 hour interval, 5 minute buffer
# No additional configuration needed
```

### 2. High-Frequency Processing
```yaml
# For systems that need more frequent updates
timer_update_interval: "30m"    # Update every 30 minutes
timer_buffer_time: "2m"          # 2 minute buffer
```

### 3. Stable Systems
```yaml
# For systems with stable processing
timer_update_interval: "2h"      # Update every 2 hours
timer_buffer_time: "10m"        # 10 minute buffer
```

### 4. Custom Time Combinations
```yaml
# Complex time intervals
timer_update_interval: "1h30m"   # 1 hour 30 minutes
timer_buffer_time: "7m30s"       # 7 minutes 30 seconds
```

### 5. Development/Testing
```yaml
# For testing with very frequent updates
timer_update_interval: "5m"      # Update every 5 minutes
timer_buffer_time: "1m"         # 1 minute buffer
```

## Time String Format

The time parsing supports flexible formats:

### Basic Units
- `"1h"` = 1 hour
- `"30m"` = 30 minutes  
- `"300s"` = 300 seconds

### Combined Units
- `"2h30m"` = 2 hours 30 minutes
- `"1h5m30s"` = 1 hour 5 minutes 30 seconds

### Numeric Values
- `"3600"` = 3600 seconds (no unit = seconds)

## Logging Output

The system logs the configured timer settings:

```
INFO: Timer settings: update interval=3600s (1.0h), buffer=5 minutes
INFO: Updated consideration timer to 2024-01-15T11:00:00+00:00 (with 5 minute buffer for slow processing)
```

## Monitoring

You can monitor the timer status programmatically:

```python
status = records_source.get_consideration_timer_status()
print(f"Update interval: {status['configured_update_interval_seconds']} seconds")
print(f"Buffer time: {status['configured_buffer_minutes']} minutes")
print(f"Next update in: {status['next_update_in_minutes']:.1f} minutes")
```

## Best Practices

### 1. Choose Appropriate Intervals
- **High-volume systems**: Use shorter intervals (30m-1h)
- **Low-volume systems**: Use longer intervals (2-4h)
- **Development**: Use very short intervals (5-15m)

### 2. Set Buffer Time
- **Fast processing**: 2-5 minutes buffer
- **Slow processing**: 5-15 minutes buffer
- **Network delays**: 10-30 minutes buffer

### 3. Monitor Performance
- Check logs for timer update frequency
- Monitor memory usage patterns
- Adjust based on actual processing times

## Troubleshooting

### Timer Not Updating
- Check if `timer_update_interval` is too long
- Verify system clock accuracy
- Check for processing delays

### Missing Records
- Increase `timer_buffer_time` if processing is slow
- Check for network delays
- Verify ServiceNow timezone settings

### High Memory Usage
- Decrease `timer_update_interval` for more frequent cleanup
- Check for memory leaks in processing logic
- Monitor record volume patterns
