---
title: Tractive
description: Instructions on how to integrate Tractive within Home Assistant.
ha_category:
  - Presence Detection
ha_release: 2021.9
ha_iot_class: Cloud Push
ha_config_flow: true
ha_codeowners:
  - '@Danielhiversen'
  - '@zhulik'
  - '@bieniu'
ha_domain: tractive
ha_platforms:
  - binary_sensor
  - device_tracker
  - sensor
  - switch
---

[Tractive](https://tractive.com/en/) is an Austrian company that develops real-time location trackers for pets and other animals using GPS and GSM technology.

The Tractive integration allows you to monitor locations of your pets from within Home Assistant and set up automations based on the information.

## Prerequisites

To use the integration you must be a premium tractive client.

{% include integrations/config_flow.md %}

## Integration Entities

The Tractive integration adds one device tracker and several sensors and switches per registered pet:

![Tractive device tracker](/images/integrations/tractive/device_tracker.png)
