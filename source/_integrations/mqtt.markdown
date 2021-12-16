---
title: MQTT
description: Instructions on how to setup MQTT within Home Assistant.
ha_category:
  - Hub
featured: true
ha_release: pre 0.7
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@emontnemery'
ha_domain: mqtt
ha_platforms:
  - alarm_control_panel
  - binary_sensor
  - camera
  - climate
  - cover
  - fan
  - humidifier
  - lock
  - number
  - scene
  - sensor
  - select
  - switch
---

MQTT (aka MQ Telemetry Transport) is a machine-to-machine or "Internet of Things" connectivity protocol on top of TCP/IP. It allows extremely lightweight publish/subscribe messaging transport.

Your first step to get MQTT and Home Assistant working is to choose a [broker](/docs/mqtt/broker).

{% include integrations/config_flow.md %}

## Manual configuration

Alternatively, if you want to manually configure MQTT, you will need to add the following to your `configuration.yaml` file.

To connect to your [own MQTT broker](/docs/mqtt/broker#run-your-own):

```yaml
# Example configuration.yaml entry
mqtt:
  broker: IP_ADDRESS_BROKER
```

Manual configuration is required when connecting to a broker over TLS.

```yaml
# Example configuration.yaml entry
mqtt:
  certificate: "PATH_TO_CA.crt"
  broker: "IP_ADDRESS_BROKER"
  port: 8883
  username: "MQTT_USERNAME"
  password: !secret MQTT_PASSWORD
```

## Additional features

- [Certificate](/docs/mqtt/certificate/)
- [Discovery](/docs/mqtt/discovery/)
- [Publish & Dump services](/docs/mqtt/service/)
- [Birth and last will messages](/docs/mqtt/birth_will/)
- [Testing your setup](/docs/mqtt/testing/)
- [Logging](/docs/mqtt/logging/)

## Event `event_mqtt_reloaded`

Event `event_mqtt_reloaded` is fired when Manually configured MQTT entities have been reloaded and entities thus might have changed.

This event has no additional data.
