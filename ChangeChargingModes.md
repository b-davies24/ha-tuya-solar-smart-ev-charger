# How do you create different charging 'modes'?
The simplest way at the time was to create different **Scripts** that enabled/disabled different combinations of Automations.

To keep track of what 'mode' you are in, create a Text Helper called *Current Charging Mode*, and make the scripts change the content of the Text Helper. This also helps if you choose to use the Peak Energy Car Saver automation, since it only triggers when you are in *Solar Priority* or *Fixed Speed* modes.

### Solar Only
Enabled both the *Smart EV Chargin Logic* and *Solar-Only Charging* automations. Change *Current Charging Mode* to `Solar Only`

### Solar Priority
Enable *Smart EV Chargin Logic*, and disable *Solar-Only Charging* automations. Change *Current Charging Mode* to `Solar Priority`

### Fixed Speed
Disable both the *Smart EV Chargin Logic* and *Solar-Only Charging* automations. Set the EVSE current to 32A (optional). Change *Current Charging Mode* to `Fixed Speed`

### Stopped
Disable both the *Smart EV Chargin Logic* and *Solar-Only Charging* automations. Turn off EVSE. Change *Current Charging Mode* to `Stopped`

This can be useful to 'reset' things
