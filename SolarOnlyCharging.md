# Logic behind the solar-only automation

The purpose of this is to turn the EVSE off when there is insufficient solar being exported to the grid.

### When
**Time Pattern** with value of /30

This frequency is lower than that of the smart ev charging logic automation, mainly to try to prevent the EVSE turning off/on/off/on... very frequently.

### And If
**State**, with all the charger states except for the state shown when unplugged. This is so that it doesn't do things when the car is physically unplugged. This is an example of the YAML for this:
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

### Then Do
This is set as two if/then statements to either turn the EVSE on when there is sufficient solar, or off when there is not enough

If: *Power Available for Charging* (this is the sensor set up/determined previously) is greater than 1.5kW

Then: Turn EVSE on

If: original power meter is importing more than 200W from the grid

Then: Turn EVSE off

Note: Again, there is a bit of buffer (so not starting at the theoretical min charging speed of 1.4kW, and the moment you tart importing >0W from grid) to attempt to prevent an on/off/on/off... cycle.

Also note: Don't use the *Power Available for Charging* sensor for both if/thens - the moment the EVSE turns on, with the other smart charging logic automation, it will gradually reduce to 0 (i.e. all the available power is going to the car). Thus, you cannot use *Power Available for Charging* as a trigger to turn off the EVSE - you need a sensor that can tell when you are importing from the grid, and how much you are importing.

## Example YAML for this whole automation
```
alias: Solar-only Charging
description: ""
triggers:
  - trigger: time_pattern
    seconds: /30
conditions:
  - condition: state
    entity_id: sensor.ev200d_ha_status
    state:
      - charging
      - paused
      - plugged_in
      - waiting
      - fault
      - fault_unplugged
      - charged
actions:
  - if:
      - condition: numeric_state
        entity_id: sensor.power_available_for_charging
        above: 1.5
    then:
      - action: switch.turn_on
        metadata: {}
        target:
          entity_id: switch.ev200d_ha
        data: {}
  - if:
      - condition: numeric_state
        entity_id: sensor.solaredge_m1_ac_power
        below: -200
    then:
      - action: switch.turn_off
        metadata: {}
        target:
          entity_id: switch.ev200d_ha
        data: {}
mode: single
```
