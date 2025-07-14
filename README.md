# Smart Pool Pump Controller

Automatic pool pump control based on volume, flow rate, and manual operation tracking throughout the day.

## ☕ Support the Author

Hi! I'm a developer and tech enthusiast who loves building and sharing tools with the community. Your support helps me keep creating and improving. Thank you for your coffee and motivation ☕🙂

[![Buy Me a Coffee](https://img.shields.io/badge/☕%20Buy%20me%20a%20coffee-coffee%20support-yellow)](https://coff.ee/myroom007)

## Description

This blueprint automatically:

-   Calculates required pump runtime for complete water circulation in the pool
-   Accounts for manual pump operation during the day
-   Adjusts automatic schedule to avoid exceeding maximum runtime
-   Provides three operation modes: Auto, On, Off
-   **Prevents pump overheating** with controlled cycles and cooling intervals
-   **Automatic cycle management** - pump runs for set duration then rests

## Required Sensors and Entities

Add the following entities to your `configuration.yaml`:

### 1. Input Select for Pump Operation Mode

```yaml
input_select:
    pool_pump_mode:
        name: Pool Pump Mode
        options:
            - 'Auto'
            - 'On'
            - 'Off'
        initial: 'Auto'
        icon: mdi:water-pump
```

**Modes:**

-   `Auto` - Automatic control by schedule
-   `On` - Forced on (always running)
-   `Off` - Forced off (not running)

### 2. Sensor for Daily Pump Runtime Tracking

```yaml
sensor:
    - platform: history_stats
      name: Daily Pump Runtime Sensor
      entity_id: switch.pool_pump # replace with your pump entity
      state: 'on'
      type: time
      start: '{{ now().replace(hour=0, minute=0, second=0) }}'
      end: '{{ now() }}'
```

## Blueprint Configuration

1. Import blueprint into Home Assistant
2. Create automation based on blueprint
3. Configure parameters:
    - **Pump**: Select your pump entity (switch)
    - **Pool Volume**: Pool volume in liters
    - **Pump Flow Rate**: Pump flow rate in l/h
    - **Maximum Daily Run Time**: Maximum daily runtime (hours)
    - **Pump Interval**: Interval between pump cycles (hours, default 1)
    - **Cycle Runtime**: How long pump runs in each cycle (minutes, default 120)
    - **Pool Pump Mode**: input_select.pool_pump_mode
    - **Daily Pump Runtime Sensor**: sensor.pool_pump_daily_runtime

## Lovelace Card Example

Add the following card to your Lovelace dashboard for convenient pool pump control and monitoring:

```yaml
type: entities
entities:
    - entity: sensor.pool_water_temperature # Optional
    - entity: input_select.pool_pump_mode
    - entity: sensor.daily_pump_runtime_sensor
    - entity: switch.pool_pump
      name: Pool Pump
title: Pool card
```

**Note**: Make sure to use the correct entity names:

-   `sensor.pool_water_temperature` - pool water temperature sensor
-   `switch.pool_pump` - pool pump switch

## How it Works

### Automatic Mode (Auto)

The blueprint now uses **intelligent cycle management** to prevent pump overheating:

1. **Cycle Start**: Pump turns on automatically when:

    - Pump is off
    - Set interval has passed since last state change (default 1 hour)
    - Daily runtime limit is not exceeded (default 8 hours)

2. **Cycle Runtime**: Pump runs for a **controlled duration** (default 120 minutes)

3. **Automatic Shutdown**: Pump **automatically turns off** after cycle runtime expires

4. **Cooling Period**: Pump stays off during the interval period for cooling

5. **Cycle Repeat**: Process repeats until daily runtime limit is reached

### Manual Operation Tracking

-   Tracks total pump runtime during the day via history_stats sensor
-   Accounts for manual pump operation in daily limit calculation

### Manual Modes

-   **"On"** - pump runs continuously (no cycle limits)
-   **"Off"** - pump stays off and automation is disabled

### Overheating Prevention

-   **Controlled cycles** prevent continuous operation
-   **Mandatory cooling periods** between cycles
-   **Configurable runtime** per cycle (5-180 minutes)

## Notes

-   All units of measurement (volume and flow rate) must be in the same system (liters/hour, m³/hour, etc.)
-   Automation checks status every minute and responds immediately to mode changes
-   Manual pump operation time is automatically tracked and counts toward daily limit
-   **Cycle Runtime** should be set based on your pump specifications and cooling requirements
-   **Pump Interval** should be longer than **Cycle Runtime** to ensure cooling periods
-   In "Auto" mode, pump will never run continuously - it always follows cycle patterns
-   Mode changes ("On"/"Off") take effect immediately, overriding current cycle
-   It's recommended to set up notifications when daily runtime limit is exceeded

## Example Cycle Timeline

With default settings (120min cycle, 60min interval):

-   **12:00** - Pump starts (if conditions met)
-   **14:00** - Pump automatically stops (120min runtime completed)
-   **15:00** - Next cycle can start (60min interval from last change)
-   **17:00** - Pump automatically stops again
-   And so on...
