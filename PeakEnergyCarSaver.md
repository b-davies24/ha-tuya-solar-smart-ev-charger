# Garbage name, but hopefully explains what it does
It will automatically stop the car charging from 3pm, and optionally start it up again at 9pm

### When
/3 minutes

### And If
The following all are within an AND condition:
- The time is between 3pm and 9pm
- A Input Boolean Helper (which can be called *Peak Energy Car Saver*) is on (optional, but can make it nicer to switch on/off from a dashboard, especially if you want the car to turn back on at 9pm)
- The current charging mode is *Solar-Priority* or *Fixed Speed*
- The EVSE is currently switched on

### Then Do
1. Turn EVSE off
2. Something to punush you (such as yell at you for being an idot through a speaker) - optional, but fun...

## Example YAML:
```
alias: Peak Energy Car Saver - auto-off
description: ""
triggers:
  - trigger: time_pattern
    minutes: /3
conditions:
  - condition: and
    conditions:
      - condition: time
        after: "15:00:00"
        before: "21:00:00"
      - condition: state
        entity_id: input_boolean.peak_energy_car_saver
        state:
          - "on"
      - condition: state
        entity_id: input_text.current_charging_mode
        state:
          - Solar Priority
          - Fixed Speed
      - condition: state
        entity_id: switch.ev200d_ha
        state:
          - "on"
actions:
  - action: switch.turn_off
    metadata: {}
    target:
      entity_id: switch.ev200d_ha
    data: {}
  - action: media_player.play_media
    metadata: {}
    target:
      entity_id:
        - media_player.office_2
        - media_player.living_kitchen
        - media_player.master_bedroom
    data:
      media:
        media_content_id: >-
          media-source://tts/tts.google_translate_en_com?message=Stop+charging+the+car+you+dickhead%21+Too+much+electricity%21%0AIt+is+peak+tariff%21&language=en-au
        media_content_type: audio/mp3
        metadata:
          title: |-
            Stop charging the car you [censored]! Too much electricity!
            It is peak tariff!
          thumbnail: /api/brands/integration/google_translate/logo.png
          media_class: app
          children_media_class: null
          navigateIds:
            - {}
            - media_content_type: app
              media_content_id: media-source://tts
            - media_content_type: audio/mp3
              media_content_id: >-
                media-source://tts/tts.google_translate_en_com?message=Stop+charging+the+car+you+dickhead%21+Too+much+electricity%21%0AIt+is+peak+tariff%21&language=en-au
          browse_entity_id: media_player.office_2
      announce: true
mode: single

```

## Optional: Auto resume charging at 9pm
You need to make a separate automation for this

### When
Time in 9:01pm

### And If
Both in an AND condition:
- *Peak Energy Car Saver* Helper that you made is on
- **State** of EVSE is not unplugged (fill in same as **State** for the logic):
  ```
  condition: state
  entity_id: sensor.ev200d_ha_status
  state:
    - charging
    - fault
    - fault_unplugged
    - paused
    - plugged_in
    - waiting
    - charged
  ```

### Then Do
Turn on EVSE
