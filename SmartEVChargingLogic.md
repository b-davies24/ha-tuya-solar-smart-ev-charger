# Logic behind the solar-aware automation

The purpose of this is to ramp-up/ramp-down the charging speed depending on the amount of excess solar being exported to the grid.

## 1: Find/create a sensor that defines the amount of power available for the charger
The first step for this was to find/create something that indicates the amount of power available to be directed to the car.
The easiest way for me was to create a new 'Helper' in Home Assistant called *Power Available for Charging* - this was a 'Template Sensor' with the code `{{ max(0, states('sensor.solaredge_m1_ac_power') |float(0)/1000)}}` 

In this context, `sensor.solaredge_m1_ac_power` is a sensor for grid-related power provided by a solar inverter; if the value is positive, it means that electricity is being exported to the grid. As such, this 'Helper' will return `0` whenever power is being imported from the grid, and a value >0 (in W) when any power is being exported. The `/1000` in the code converts the value in W to kW for later use.

This step may/may not be needed - it all depends on what sensors are available in Home Assistant.

## 2: Create the automation
This will be explained step-by-step, so that it can be reproduced if things change in the future

**When**

Time pattern, with `/10` seconds; this makes it occur every 10s

**And If**

State, with all the charger states except for the state shown when unplugged. This is an example of the YAML for this:
```
condition: state
entity_id: sensor.ev200d_ha_status
state:
  - charging
  - plugged_in
  - waiting
  - fault
  - fault_unplugged
  - paused
enabled: true
```

**Then Do**

Define Variables:
```
variables:
  charger_current: "{{ states('number.ev200d_ha_charging_current') | float(6) }}"
  available_power: "{{ states('sensor.power_available_for_charging') | float(0) }}"
  delta_amps: 2
  max_amps: 32
  min_amps: 6
  new_amps: |
    {% if available_power > 0.6 %}
      {{ [charger_current + delta_amps, max_amps] | min}}
    {% elif available_power < 0.4 %}
      {{ [charger_current - delta_amps, min_amps] | max}}
    {% else %}
      {{ charger_current }}
    {% endif %}

```
For `charger_current`, this should be defined as the control exposed by Tuya for the current (in A) of the EVSE. This automation only works if Home Assistant has the ability to adjust the EVSE's charging current.

`available_power` is the sensor that was set up/chosen in the above section

`delta_amps` is the amount by which the charging current is adjusted each iteration. If you are not on 240V power, or change this to another number, you will need to adjust the numbers in `new_amps`; `0.6` and `0.4` correspond to 0.6kW and 0.4kW - 2A corresponds to ~480W = 0.48kW at 240V, so the 0.4 and 0.6 are the thresholds at which the change in charging current is decreased/increased.
