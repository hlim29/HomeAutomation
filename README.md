# Home Automation - Sigenergy Battery + Amber Electric

Home Assistant automation and dashboard configurations for optimising a Sigenergy battery system with Amber Electric's dynamic pricing.

## Overview

This project contains Home Assistant configurations that automatically manage battery charging/discharging based on real-time electricity prices from Amber Electric. The goal is to maximise savings by:

- **Charging** the battery when prices are low or negative
- **Discharging/Exporting** when feed-in prices are high
- **Self-consumption** during normal price periods

## Requirements

### Hardware
- Sigenergy inverter and battery system
- Shelly EM3 (for hot water system control)

### Home Assistant Integrations
- [Amber Electric Integration](https://www.home-assistant.io/integrations/amberelectric/)
- [Sigenergy Integration](https://github.com/TypQxQ/Sigenergy-Local-Modbus)
- [Solcast PV Forecast](https://github.com/BJReplay/ha-solcast-solar)
- Shelly Integration

### Custom Cards (for Dashboard)
- [ApexCharts Card](https://github.com/RomRider/apexcharts-card)
- [Button Card](https://github.com/custom-cards/button-card)
- Energy Sankey (optional)

## Files

| File | Description |
|------|-------------|
| `battery_automation.yaml` | Main automation logic for battery control |
| `dashboard.yaml` | Lovelace dashboard configuration |

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

## Automation Logic

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

## Dashboard Features

The dashboard provides:

1. **Control Panel**
   - Import/Export switches
   - Charging and export power limits
   - Price thresholds adjustment
   - EMS timer status

2. **Power Flow Charts**
   - 24-hour history of grid, PV, and battery power
   - Battery state of charge

3. **Price Forecasts**
   - Amber Electric buy/sell price forecasts
   - Current and predicted prices over 12 hours

4. **Status Badges**
   - Current EMS mode
   - Battery SOC
   - PV power
   - Grid power
   - Current Amber prices
   - Solcast remaining forecast

## Installation

1. Copy `battery_automation.yaml` to your Home Assistant automations
2. Create required input helpers
3. Update device IDs to match your Sigenergy inverter
4. Copy `dashboard.yaml` to your Lovelace configuration
5. Install required custom cards via HACS

## Notes

- The automation only triggers on actual prices (not estimates) to avoid erratic behaviour
- A timer (`timer.ems_timer`) can be used to temporarily override automation
- Trace logging is enabled with 20 stored traces for debugging

## License

See [LICENSE](LICENSE) file.
