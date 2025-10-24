# Timer-Based Record Processing

## Overview

The EDA plugin now uses a timer-based approach instead of fixed record counts for processing records. This provides more robust and predictable behavior, especially for long-running processes.

## How It Works

### 1. **Process Start Time**
- When the plugin starts, it records the process start time
- The initial consideration timer is set to the process start time
- This ensures we only consider records from when the process actually started

### 2. **Hourly Timer Updates**
- Every hour, the consideration timer is automatically updated
- The timer is set to the current time minus a 5-minute buffer
- This buffer accounts for slow processing and ensures we don't miss records

### 3. **Record Filtering**
- Records are only processed if they were updated after the consideration timer
- This prevents processing of very old records that might have been missed
- The system uses the more recent of either the latest processed timestamp or the consideration timer

## Configuration

### Timer Settings (Configurable Parameters)

The timer-based processing system now supports configurable parameters:

```python
# Timer-based processing settings (configurable parameters)
self._process_start_time = datetime.now(timezone.utc)
self._consideration_timer = self._process_start_time

# Parse timer update interval (default: 1 hour = 3600 seconds)
timer_interval_str = args.get("timer_update_interval", "1h")
self._timer_update_interval = self._parse_time_interval(timer_interval_str)

# Parse buffer time (default: 5 minutes)
buffer_time_str = args.get("timer_buffer_time", "5m")
self._timer_buffer_minutes = self._parse_time_interval(buffer_time_str) / 60
```

### Configuration Parameters

#### `timer_update_interval`
- **Default**: `"1h"` (1 hour)
- **Format**: Supports time strings like `"1h"`, `"30m"`, `"3600s"`, `"2h30m"`
- **Purpose**: How often the consideration timer is updated

#### `timer_buffer_time`
- **Default**: `"5m"` (5 minutes)
- **Format**: Supports time strings like `"5m"`, `"300s"`, `"1h5m"`
- **Purpose**: Buffer time to account for slow processing

### Example Configurations

```yaml
# Default configuration (1 hour interval, 5 minute buffer)
timer_update_interval: "1h"
timer_buffer_time: "5m"

# More frequent updates (30 minutes interval, 2 minute buffer)
timer_update_interval: "30m"
timer_buffer_time: "2m"

# Longer intervals for stable systems (2 hours interval, 10 minute buffer)
timer_update_interval: "2h"
timer_buffer_time: "10m"

# Custom combinations
timer_update_interval: "1h30m"
timer_buffer_time: "7m30s"
```

### Buffer Time
- **Default**: 5 minutes
- **Purpose**: Accounts for slow processing and network delays
- **Benefit**: Prevents missing records due to processing delays

## Benefits

### 1. **Predictable Behavior**
- Records are considered based on time, not arbitrary counts
- More consistent processing across different data volumes

### 2. **Memory Efficiency**
- Prevents accumulation of very old records
- Automatic cleanup based on time boundaries

### 3. **Robust Processing**
- Handles slow processing gracefully with buffer time
- Reduces risk of missing records due to processing delays

### 4. **Debugging Support**
- Timer status can be queried for monitoring
- Clear logging of timer updates and record filtering

## Example Usage

### Timer Status Monitoring
```python
# Get current timer status
status = records_source.get_consideration_timer_status()
print(f"Process running for {status['time_since_start_hours']:.2f} hours")
print(f"Next timer update in {status['next_update_in_minutes']:.1f} minutes")
```

### Log Output
```
INFO: Starting with consideration timer at 2024-01-15T10:00:00+00:00 (process started at 2024-01-15T10:00:00+00:00)
INFO: Updated consideration timer to 2024-01-15T11:00:00+00:00 (with 5 minute buffer for slow processing)
DEBUG: Record update timestamp 2024-01-15T10:30:00+00:00 is older than consideration timer 2024-01-15T11:00:00+00:00, skipping
```

## Migration from Fixed Counts

The timer-based system replaces the previous fixed record count approach:

### Before (Fixed Counts)
- Processed fixed number of records per cycle
- Could miss records if processing was slow
- Memory usage could grow unpredictably

### After (Timer-Based)
- Processes records based on time boundaries
- Handles slow processing with buffer time
- Predictable memory usage and cleanup

This approach provides much more robust and predictable behavior for long-running EDA processes.
