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

1. **Simple Interval Logic**: Blueprint uses a set interval between pump activations (default 1 hour)

2. **Manual Operation Tracking**: Tracks total pump runtime during the day via history_stats sensor

3. **Automatic Activation**: In "Auto" mode, pump turns on automatically when:

    - Pump is off
    - Set time has passed since last state change (pump_interval)
    - Daily runtime limit is not exceeded (maximum_run_time)

4. **Manual Modes**:
    - "On" - pump is always on
    - "Off" - pump is always off

## Notes

-   All units of measurement (volume and flow rate) must be in the same system (liters/hour, m³/hour, etc.)
-   Automation checks status every minute
-   Manual pump operation time is automatically tracked
-   It's recommended to set up notifications when daily runtime limit is exceeded
