---
title: AVM FRITZ!SmartHome
description: Instructions on how to integrate AVM Fritz!DECT components into Home Assistant.
ha_category:
  - Binary Sensor
  - Climate
  - Light
  - Sensor
  - Switch
ha_release: 0.68
ha_iot_class: Local Polling
ha_domain: fritzbox
ha_config_flow: true
ha_ssdp: true
ha_platforms:
  - binary_sensor
  - climate
  - light
  - sensor
  - switch
ha_codeowners:
  - '@mib1185'
  - '@flabbamann'
---

The AVM FRITZ!SmartHome integration for Home Assistant allows you to integrate [FRITZ!DECT](https://en.avm.de/products/fritzdect/) devices like switches, sensors and thermostats.

There is currently support for the following device types within Home Assistant:

- Binary Sensor
- Climate
- Light
- Sensor
- Switch

#### Tested Devices

- [FRITZ!Box 6490 Cable](https://en.avm.de/products/fritzbox/fritzbox-6490-cable/)
- [FRITZ!Box 6591 Cable](https://en.avm.de/products/fritzbox/fritzbox-6591-cable/)
- [FRITZ!Box 7590](https://en.avm.de/products/fritzbox/fritzbox-7590/)
- [FRITZ!Box 7490](https://en.avm.de/service/fritzbox/fritzbox-7490/overview/)
- [FRITZ!Box 7430](https://en.avm.de/service/fritzbox/fritzbox-7430/overview/)
- [FRITZ!DECT 200](https://en.avm.de/products/fritzdect/fritzdect-200/)
- [FRITZ!DECT 301](https://en.avm.de/products/fritzdect/fritzdect-301/)
- [FRITZ!DECT 500](https://en.avm.de/products/fritzdect/fritzdect-500/)
- [Eurotronic Comet DECT](https://eurotronic.org/produkte/elektronische-heizkoerperthermostate/sparmatic-comet/)
- [Magenta SmartHome LED E27](https://www.smarthome.de/geraete/smarthome-led-lampe-e27-farbig-weiss)

{% include integrations/config_flow.md %}

<div class='note'>
The configuration in the UI asks for a username. Starting from FRITZ!OS 7.24 the FRITZ!Box creates a random username for the admin user if you didn't set one yourself. This can be found after logging into the FRITZ!Box and visiting System -> FRITZ!Box Users -> Users. The username starts with "fritz" followed by four random numbers. Under properties on the right it says "created automatically". Prior to FRITZ!OS 7.24 the default username was "admin".
</div>

## Switches & Thermostats

To get AVM FRITZ!DECT switches (e.g. FRITZ!DECT 400/440) or thermostats (e.g. FRITZ!DECT 301) follow the [configuration instructions](#configuration) above.

### Attributes

There are several attributes that can be useful for automations and templates.

| Attribute | Description |
| --------- | ----------- |
| `device_locked` | The state of the key lock at the device.
| `locked` | The state of the lock for configuring the device via the app or the FRITZ!Box web interface.
| `low_battery` | The low battery state indication.
| `battery_level` | The battery level (only available since Fritz!OS 7).
| `holiday_mode` | The state of the holiday mode (only available since Fritz!OS 7).
| `summer_mode` | The state of the summer mode (only available since Fritz!OS 7).
| `window_open` | The state of the window open detection (only available since Fritz!OS 7).

## Sensors

To get AVM FRITZ!DECT sensors (e.g.,  FRITZ!DECT Repeater 100) follow the [configuration instructions](#configuration) above.

### Attributes

There are several attributes that can be useful for automations and templates.

| Attribute | Description |
| --------- | ----------- |
| `device_locked` | The state of the key lock at the device.
| `locked` | The state of the lock for configuring the device via the app or the FRITZ!Box web interface.

## Light

To get AVM FRITZ!DECT lightbulbs (e.g., FRITZ!DECT 500) follow the [configuration instructions](#configuration) above.

<div class='note'>
The FRITZ!DECT 500 lightbulb supports only 36 colors. When a color is picked in home assistant that is not supported by the device, a color that comes close will be activated.
</div>
