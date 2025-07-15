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

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmyroom%2FSmartPoolPumpController%2Fblob%2Fmain%2Fsmart_pool_pump_controller.yaml)

1. Import blueprint into Home Assistant
2. Create automation based on blueprint
3. Configure parameters:
    - **Pump**: Select your pump entity (switch)
    - **Pool Volume**: Pool volume in liters
    - **Pump Flow Rate**: Pump flow rate in l/h
    - **Water Turnover Cycles**: Number of complete water turnovers per day (default 2)
    - **Maximum Daily Run Time**: Maximum daily runtime (hours)
    - **Pump Interval**: Interval between pump cycles (hours, default 1)
    - **Cycle Runtime**: How long pump runs in each cycle (minutes, default 120)
    - **Start Time**: Automation start time - pump can work from this time (default 09:00)
    - **End Time**: Automation end time - pump stops working after this time (default 21:00)
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

    - Current time is within working hours (default 09:00-21:00)
    - Pump is off
    - Set interval has passed since last state change (default 1 hour)
    - Calculated daily limit is not exceeded (Pool Volume ÷ Flow Rate × Turnover Cycles)
    - Maximum daily runtime limit is not exceeded (default 8 hours)

2. **Cycle Runtime**: Pump runs for a **controlled duration** (default 120 minutes)

3. **Automatic Shutdown**: Pump **automatically turns off** after cycle runtime expires

4. **Cooling Period**: Pump stays off during the interval period for cooling

5. **Cycle Repeat**: Process repeats until daily runtime limit is reached

### Dual Limit System

The automation uses **two different daily limits** to optimize pump operation:

1. **Calculated Limit** (Smart): `Pool Volume ÷ Flow Rate × Turnover Cycles`

    - Based on actual pool filtration needs
    - Ensures proper water circulation and filtration
    - Example: 50,000L pool ÷ 10,000L/h × 2 cycles = 10 hours

2. **Maximum Runtime Limit** (Safety): User-defined maximum (default 8 hours)
    - Prevents excessive pump operation
    - Energy and equipment protection
    - Override for unusual conditions

**Active Limit**: The system always uses the **lower** of these two values.

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
-   **Water Turnover Cycles** determine how many times the entire pool volume should be filtered per day
-   **Calculated Daily Limit** = Pool Volume ÷ Pump Flow Rate × Water Turnover Cycles (in hours)
-   The system uses both calculated limit and maximum runtime limit - whichever is lower
-   **Cycle Runtime** should be set based on your pump specifications and cooling requirements
-   **Pump Interval** should be longer than **Cycle Runtime** to ensure cooling periods
-   **Working Hours** allow you to restrict pump operation to specific time periods (e.g., daytime only)
-   In "Auto" mode, pump will never run continuously - it always follows cycle patterns
-   Mode changes ("On"/"Off") take effect immediately, overriding current cycle
-   Pump will be automatically turned off when working hours end, regardless of current cycle
-   It's recommended to set up notifications when daily runtime limit is exceeded

## Example Configuration

**Pool Setup**: 50,000L pool, 10,000L/h pump, 2 turnover cycles

-   **Calculated Limit**: 50,000 ÷ 10,000 × 2 = **10 hours**
-   **Maximum Runtime**: 8 hours (user setting)
-   **Active Limit**: 8 hours (lower value)

## Example Cycle Timeline

With settings (120min cycle, 60min interval, 09:00-21:00 working hours, 8h limit):

-   **09:00** - Working hours start, pump can begin cycles
-   **10:00** - Pump starts (if conditions met)
-   **12:00** - Pump automatically stops (120min runtime completed)
-   **13:00** - Next cycle can start (60min interval from last change)
-   **15:00** - Pump automatically stops again
-   **17:00** - Cycles continue until 8h daily limit reached
-   **21:00** - Working hours end, pump will be turned off if running
-   **21:01-08:59** - Pump stays off regardless of other conditions
