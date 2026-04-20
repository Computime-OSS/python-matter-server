# EVSE Charging Use Cases

This document summarizes the two EV charging strategies currently configured in Home Assistant for the Matter EVSE.

## Use Case 1: Price-Based Charging

Goal:
Charge when electricity is relatively cheap, and stop when electricity becomes expensive.

How it works:
- `sensor.price_level` is derived from `sensor.current_electricity_market_price` and `sensor.average_electricity_price`.
- When the price level becomes `VERY_CHEAP`, `CHEAP`, or `NORMAL`, Home Assistant turns on every charger whose entity ID matches `switch.*_enable_charging`.
- When the price level becomes `EXPENSIVE`, `VERY_EXPENSIVE`, or `EXTREMELY_EXPENSIVE`, Home Assistant turns those chargers off.

Main entities:
- `sensor.current_electricity_market_price`
- `sensor.average_electricity_price`
- `sensor.price_level`
- `switch.matter_evse_enable_charging`

Configured automations:
- `price_based_charging_start`
- `price_based_charging_stop`

Behavior summary:
- Simple tariff-based control
- Good for scheduled low-cost charging
- Does not consider solar surplus or house load

## Use Case 2: Solar Surplus Charging

Goal:
Only charge when there is exported solar power available, and increase charging current according to the available surplus.

How it works:
- Home Assistant reads IAMMETER simulator data from `http://127.0.0.1:8080/monitorjson` using Basic Auth.
- The simulator provides three measured channels:
  - `A(Solar)` -> solar generation
  - `B(Grid)` -> grid import/export
  - `C(Load)` -> local load
- Template sensors expose these values as Home Assistant entities.
- `sensor.iammeter_solar_surplus_power` treats negative grid power as exported power and converts it to usable solar surplus.
- `sensor.evse_target_current_from_surplus` converts surplus power into a target charging current using a single-phase 230 V assumption.
- If the target current is greater than zero, Home Assistant:
  - updates `number.matter_evse_user_max_charge_current`
  - turns on `switch.matter_evse_enable_charging`
- If the target current is zero, Home Assistant turns charging off.

Main entities:
- `sensor.iammeter_simulator_api`
- `sensor.iammeter_solar_power`
- `sensor.iammeter_grid_power`
- `sensor.iammeter_load_power`
- `sensor.iammeter_solar_surplus_power`
- `sensor.evse_target_current_from_surplus`
- `sensor.matter_evse_min_charge_current`
- `sensor.matter_evse_max_charge_current`
- `number.matter_evse_user_max_charge_current`
- `switch.matter_evse_enable_charging`

Configured automation:
- `solar_surplus_evse_control`

Behavior summary:
- Dynamic solar-following control
- Attempts to match EV charging current to real-time surplus power
- Stops charging when available surplus is below minimum charging current

## Notes

- The current solar surplus calculation assumes `sensor.iammeter_grid_power < 0` means power is being exported to the grid.
- The current-to-power conversion assumes single-phase 230 V charging.
- If the installation is three-phase, the target current formula should be updated.
- The price-based automations and the solar-surplus automation can conflict because both control `switch.matter_evse_enable_charging`.

Recommended operating mode:
- Enable only one strategy at a time unless a higher-level coordination rule is added.
