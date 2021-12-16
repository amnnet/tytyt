---
title: Verisure
description: Instructions on how to setup Verisure devices within Home Assistant.
ha_category:
  - Hub
  - Alarm
  - Binary Sensor
  - Camera
  - Lock
  - Sensor
  - Switch
ha_release: pre 0.7
ha_iot_class: Cloud Polling
ha_domain: verisure
ha_codeowners:
  - '@frenck'
ha_platforms:
  - alarm_control_panel
  - binary_sensor
  - camera
  - lock
  - sensor
  - switch
ha_config_flow: true
ha_dhcp: true
---

Home Assistant has support to integrate your [Verisure](https://www.verisure.com/) devices.

There is currently support for the following device types within Home Assistant:

- Alarm
- Camera
- Switch (Smartplug)
- Sensor (Thermometers, Hygrometers and Mouse detectors)
- Lock
- Binary Sensor (Door & Window)

{% include integrations/config_flow.md %}

## 2 Factor Authentication Prerequisite

Verisure added 2FA rules to Verisure My Pages that aren't supported through their third-party API integration. If you have 2FA enabled, which is forced by default, you might not be able to use this integration. Here is the suggested way to deactivate 2FA (if it's allowed in your region).

You can deactivate 2FA for your admin account and use that for Home Assistant but this isn't recommended. The steps below sets up a specific Home Assistant user and gives it restricted access.

1. Log in to Verisure My Pages as your admin user and create a new admin user for Home Assistant.
2. Log in as your newly created Home Assistant user, you'll be prompted to set up 2FA, do that and then log out. This will make sure the options below are available.
3. Log in as the Home Assistant user, browse to Account and subscription -> Account -> Login Credentials -> Disable 2FA.<div class='note warning'>This will only be available if the user is admin and has logged in once with 2FA, logged out and in again.</div>
4. Log in as your administrator again and change the Home Assistant user to a restricted user.
5. Change Home Assistant Verisure config to the new user credentials in Home Assistant.
6. Restart Home Assistant.

## Alarm Control Panel

The Verisure alarm control panel platform allows you to control your [Verisure](https://www.verisure.com/) Alarms.

The requirement is that you have setup your Verisure hub first, with the instruction above.

The `changed_by` attribute enables one to be able to take different actions depending on who armed/disarmed the alarm in [automation](/getting-started/automation/).

{% raw %}

```yaml
automation:
  - alias: "Alarm status changed"
    trigger:
      - platform: state
        entity_id: alarm_control_panel.alarm_1
    action:
      - service: notify.notify
        data:
          message: >
            Alarm changed from {{ trigger.from_state.state }}
            to {{ trigger.to_state.state }}
            by {{ trigger.to_state.attributes.changed_by }}
```

{% endraw %}

## Services

| Service | Description |
| ------- | ----------- |
| disable_autolock | Disables autolock function for a specific lock. |
| enable_autolock | Enables autolock function for a specific lock. |
| smartcam_capture | Capture a new image from a specific smartcam. |
