# Home Automation - Sigenergy Battery (Amber Electric / GloBird)

Home Assistant automation and dashboard configurations for optimising a Sigenergy battery system. There are two automation variants, one per electricity retailer:

- **Amber Electric**: reacts to real-time wholesale pricing
- **GloBird**: runs on a fixed daily time-of-use schedule

## Overview

This project contains Home Assistant configurations that automatically manage battery charging/discharging. The Amber variant responds to real-time electricity prices from Amber Electric; the GloBird variant switches modes at set times of day. The goal is to maximise savings by:

- **Charging** the battery when prices are low or negative
- **Discharging/Exporting** when feed-in prices are high
- **Self-consumption** during normal price periods

## Requirements

### Hardware
- Sigenergy inverter and battery system
- Shelly EM3 (for hot water system control)

### Home Assistant Integrations
- [Amber Electric Integration](https://www.home-assistant.io/integrations/amberelectric/) (Amber variant only)
- [Sigenergy Integration](https://github.com/TypQxQ/Sigenergy-Local-Modbus)
- [Solcast PV Forecast](https://github.com/BJReplay/ha-solcast-solar)
- Shelly Integration
- [EMHASS](https://github.com/davidusb-geek/emhass) (GloBird dashboard only, for the Energy Plan view, `sensor.mpc_*`)

### Custom Cards (for Dashboard)
- [ApexCharts Card](https://github.com/RomRider/apexcharts-card)
- [Button Card](https://github.com/custom-cards/button-card)
- [Multiple Entity Row](https://github.com/benct/lovelace-multiple-entity-row) (GloBird dashboard only)
- [Power Flow Card Plus](https://github.com/flixlix/power-flow-card-plus) (GloBird dashboard only)
- [Energy Sankey](https://github.com/MindFreeze/ha-sankey-chart) (optional)

## Files

| File | Description |
|------|-------------|
| `amber/battery_automation.yaml` | Price-driven battery control for Amber Electric |
| `globird/battery_automation.yaml` | Time-scheduled battery control for GloBird |
| `amber/dashboard.yaml` | Lovelace dashboard for the Amber setup |
| `globird/dashboard.yaml` | Lovelace dashboard for the GloBird setup |

## Configuration

### Input Helpers Required

Create the following input helpers in Home Assistant:

| Entity | Type | Description |
|--------|------|-------------|
| `input_number.import_threshold` | Number | Price threshold (c/kWh) above which to avoid grid import |
| `input_number.export_threshold` | Number | Feed-in price (c/kWh) above which to export battery |
| `input_number.export_soc_cutoff` | Number | Minimum battery SOC (%) to allow export |
| `input_text.battery_logs` | Text | Logging field for automation activity |
| `timer.ems_timer` | Timer | Prevents automation changes during active timer |

### Switches Required

| Entity | Description |
|--------|-------------|
| `switch.enable_import` | Toggle to enable grid charging |
| `switch.enable_export` | Toggle to enable battery export |

## Automation Logic (Amber)

### Triggers
- Amber Electric 5-minute price updates (non-estimate values only)

### Battery Control Modes

| Condition | Action |
|-----------|--------|
| **Negative prices** | Enable grid charging at 10kW rate |
| **Feed-in price > export threshold** AND **SOC > cutoff** | Enable battery export |
| **Import price > import threshold** | Set to Maximum Self Consumption mode |
| **Battery full (>99.9%)** AND **negative feed-in** | Limit export (optional) |
| **Default** | Maximum Self Consumption at 10kW charge rate |

### Hot Water System Control
- Automatically turns off HWS when import prices exceed threshold
- Prevents unnecessary grid consumption during expensive periods

## Automation Logic (GloBird)

`globird/battery_automation.yaml` is a single automation ("Globird daily schedule") driven by time triggers. It does not depend on the Amber integration or the price threshold helpers.

| Time | Action |
|------|--------|
| **11:00** | Enable grid import (`switch.enable_import`), set max charging limit to 10kW, turn on the hot water system |
| **14:00** | Turn off import and export |
| **18:00** | Export window start (the export actions are currently disabled in the YAML) |
| **20:00** | Turn off import and export |

To use the evening export window, enable the `switch.enable_export` and `number.sigen_plant_grid_export_limitation` actions in the `export_start` branch.

## Dashboard Features

Each retailer folder has its own `dashboard.yaml`.

### Amber (`amber/dashboard.yaml`)

A single **Home** view:

- Import/Export switches, charging and export limits
- Hot water system toggle with live power draw (`switch.shellyem3_hws`)
- Price threshold helpers (import threshold, export threshold, export SOC cutoff)
- EMS control mode logbook
- 24-hour history of grid, PV, and battery power plus battery SOC
- Amber Electric price forecasts: a 1-hour billing interval column chart and a 12-hour buy/feed-in price line chart
- Energy distribution and Sankey cards
- Status badges: EMS mode, battery SOC, PV, consumption, battery and grid power, current Amber buy and feed-in prices, Solcast remaining forecast

### GloBird (`globird/dashboard.yaml`)

Doesn't reference any Amber sensors or price thresholds on its main view. Views:

1. **Home**
   - Import/Export switches, with the export limit shown inline via Multiple Entity Row
   - Hot water system toggle with live power draw (`switch.shellyem3`)
   - EMS control mode logbook
   - 24-hour history of grid, PV, and battery power plus battery SOC (legend shown, series can be toggled)
   - Solcast forecasts and per-circuit power from Shelly meters
   - Energy distribution and Sankey cards
   - Status badges: EMS mode, battery SOC, PV, consumption, battery and grid power, Solcast remaining forecast, available discharge capacity

2. **Energy Plan**
   - 24-hour EMHASS forecast of PV, load, grid, battery power and SOC
   - Forecast buy/sell prices and deferrable load schedule

3. **Sigenergy views** (Home, EMS Control, All Sigenergy provided parameters, Calculated attributes)
   - Power, daily and total energy, EMS settings, ESS limits, alarms and all raw Sigenergy sensor values

4. **test**
   - Work-in-progress section with price threshold helpers and a Power Flow Card Plus

## Installation

1. Copy the automation for your retailer (`amber/battery_automation.yaml` or `globird/battery_automation.yaml`) to your Home Assistant automations
2. Create required input helpers and switches (the GloBird variant only needs the switches)
3. Update device and entity IDs to match your Sigenergy inverter and Shelly device (the two dashboards use different Shelly entity names)
4. Copy the matching `dashboard.yaml` to your Lovelace configuration
5. Install required custom cards via HACS

## Notes

- The Amber automation only triggers on actual prices (not estimates) to avoid erratic behaviour
- A timer (`timer.ems_timer`) can be used to temporarily override the Amber automation
- Trace logging (Amber automation) is enabled with 20 stored traces for debugging

## License

See [LICENSE](LICENSE) file.
