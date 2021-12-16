---
title: AVM FRITZ!Box Tools
description: Instructions on how to integrate AVM FRITZ!Box based routers into Home Assistant.
ha_category:
  - Presence Detection
  - Binary Sensor
  - Sensor
ha_release: '0.10'
ha_domain: fritz
ha_config_flow: true
ha_codeowners:
  - '@mammuth'
  - '@AaronDavidSchneider'
  - '@chemelli74'
ha_iot_class: Local Polling
ha_platforms:
  - binary_sensor
  - device_tracker
  - sensor
  - switch
ha_ssdp: true
---

The AVM FRITZ!Box Tools integration allows you to control your [AVM FRITZ!Box](https://en.avm.de/products/fritzbox/) based router.

There is support for the following platform types within Home Assistant:

- **Device tracker** - presence detection by looking at connected devices.
- **Binary sensor** - connectivity status.
- **Sensor** - external IP address, uptime and network monitors.
- **Switch** - call deflection, port forward, parental control and Wi-Fi networks.

{% include integrations/config_flow.md %}

<div class='note'>
TR-064 needs to be enabled in the FRITZ!Box network settings for Home Assistant to login and read device info.
</div>

## Username

The configuration in the UI asks for a username. Starting from FRITZ!OS 7.24 the FRITZ!Box creates a random username for the admin user if you didn't set one yourself. This can be found after logging into the FRITZ!Box and visiting System -> FRITZ!Box Users -> Users. The username starts with `fritz` followed by four random numbers. Under properties on the right it says `created automatically`. Prior to FRITZ!OS 7.24 the default username was `admin`.

## Services

Currently supported services are Platform specific:

- `fritz.reconnect`
- `fritz.reboot`
- `fritz.cleanup`

### Platform Services

#### Service `fritz.reboot`

Reboot the router.

| Service data attribute | Optional | Description                                                                                                    |
| ---------------------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `entity_id`            | no       | Only act on a specific  router                                                                                 |

#### Service `fritz.reconnect`

Disconnect and reconnect the router to the Internet.
If you have a dynamic IP address, most likely it will change.

| Service data attribute | Optional | Description                                                                                                    |
| ---------------------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `entity_id`            | no       | Only act on a specific  router                                                                                 |

#### Service `fritz.cleanup`

Remove all stale devices from Home Assistant.
A device is identified as stale when it's still present on Home Assistant but not on the FRITZ!Box.

| Service data attribute | Optional | Description                                                                                                    |
| ---------------------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `entity_id`            | no       | Only act on a specific  router                                                                                 |

## Integration Options

It is possible to change some behaviors through the integration options.
These can be changed at **AVM FRITZ!Box Tools** -> **Configure** on the Integrations page.

- **Consider home**: Number of seconds that must elapse before considering a disconnected device "not at home".

## Additional info

### Parental control

Parental control switches can be used to enable and disable internet access of individual devices. If a device has internet access it will be enabled, otherwise it will be disabled. You can also find the current blocking state of the individual devices in the UI of the FRITZ!Box under `Internet` -> `Filters` -> `Parental Controls` -> `Device Block`

Parental control switches are designed for advanced users, thus they are disabled by default. You need to enable the wanted entities manually.

### Device Tracker

**Note 1**: All devices to be tracked, even the new detected, are disabled by default. You need to enable the wanted entities manually.

**Note 2**: If you don't want to automatically track newly detected devices, disable the integration system option `Enable new added entities`.

### Port Forward

Due to security reasons, AVM implemented the ability to enable/disable a port forward rule only from the host involved in the rule.
As a result, this integration will create entities only for rules that have your Home Assistant host as a destination.

**Note 1**: On your FRITZ!Box, enable the setting `Permit independent port sharing for this device` for the device which runs HA (`Internet` -> `Permit Access` -> `<Name of HA device>`)

**Note 2**: Only works if you have a dedicated IPv4 address (it won't work with DS-Lite)

## Example Automations and Scripts

### Script: Reconnect / get new IP

The following script can be used to easily add a reconnect button to your UI. If you want to reboot your FRITZ!Box, you can use `fritz.reboot` instead.

```yaml
fritz_box_reconnect:
  alias: "Reconnect FRITZ!Box"
  sequence:
    - service: fritz.reconnect
      target:
        entity_id: binary_sensor.fritzbox_7530_connection

```

### Automation: Reconnect / get new IP every night

```yaml
automation:
- alias: "System - Reconnect FRITZ!Box"
  trigger:
    - platform: time
      at: "05:00:00"
  action:
    - service: fritz.reconnect
      target:
        entity_id: binary_sensor.fritzbox_7530_connection

```

### Automation: Phone notification with Wi-fi credentials when guest Wi-fi is created

```yaml
automation:
  - alias: "Guests Wi-fi Turned On -> Send Password To Phone"
    trigger:
      - platform: state
        entity_id: switch.fritzbox_7530_wifi_myssid
        to: "on"
    action:
      - service: notify.notify
        data:
          title: "Guest Wi-Fi is enabled"
          message: "Password: ..."

```
