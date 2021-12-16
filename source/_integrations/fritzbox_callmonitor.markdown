---
title: AVM FRITZ!Box Call Monitor
description: Instructions on how to integrate a phone call monitor for AVM FRITZ!Box routers into Home Assistant.
ha_category:
  - System Monitor
ha_release: 0.27
ha_iot_class: Local Polling
ha_domain: fritzbox_callmonitor
ha_config_flow: true
ha_platforms:
  - sensor
---

The `fritzbox_callmonitor` sensor monitors the call monitor exposed by [AVM FRITZ!Box](https://avm.de/produkte/fritzbox/) routers on TCP port 1012. It will assume the values `idle`, `ringing`, `dialing` or `talking` with the phone numbers involved contained in the state attributes.
It can also access the internal phone book of the router to look up the names corresponding to the phone numbers and store them in the state attributes.

## Prerequisites

To use the FRITZ!Box call monitor in your installation, a user with at least `FRITZ!Box Settings` and `Voice messages, faxes, FRITZ!App Fon and call list` rights has to be created:

1.  Open the web user interface via `fritz.box` or the IP address of your FRITZ!Box (e.g. `192.168.1.1`).
2.  Log in with your admin user credentials. The default admin user credentials can be found at the bottom of your FRITZ!Box.
3.  Navigate to **System** -> **FRITZ!Box User**.
4.  Click the `Add User` button.
5.  Enable the option `User account enabled`.
6.  Enter a user name and password.
7.  Check the rights boxes next to `FRITZ!Box Settings`, which automatically checks the boxes for `Voice messages, faxes, FRITZ!App Fon and call list` and `Smart Home`.
8.  Click the `Apply` button.

## Setup

To activate the call monitor on your FRITZ!Box, dial **#96\*5\*** from any phone connected to it.

{% include integrations/config_flow.md %}

## Examples

### Send notifications on state change

This example shows how to send notifications whenever the sensor's state changes. You will get notified both when you receive a call and also when a call is placed.

{% raw %}

```yaml
# Example configuration.yaml entry.
automation:
  - alias: "Notify about phone state"
    trigger:
      - platform: state
        entity_id: sensor.phone
    action:
      - service: notify.notify
        data:
          title: "Phone"
          message: >-
            {% if is_state("sensor.phone", "idle") %}
              Phone is idle
            {% elif is_state("sensor.phone", "dialing") %}
              Calling {{ state_attr('sensor.phone', 'to_name') }} ({{ state_attr('sensor.phone', 'to') }})
            {% elif is_state("sensor.phone", "ringing") %}
              Incoming call from {{ state_attr('sensor.phone', 'from_name') }} ({{ state_attr('sensor.phone', 'from') }})
            {% else %}
              Talking to {{ state_attr('sensor.phone', 'with_name') }} ({{ state_attr('sensor.phone', 'with') }})
            {% endif %}
```

{% endraw %}
