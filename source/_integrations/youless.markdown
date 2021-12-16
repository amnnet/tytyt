---
title: YouLess
description: Instructions on how to integrate your YouLess device into Home Assistant.
ha_category:
  - Sensor
  - Energy
ha_iot_class: Local Polling
ha_config_flow: true
ha_release: 2021.8
ha_domain: youless
ha_codeowners:
  - '@gjong'
ha_platforms:
  - sensor
---

The YouLess integration for Home Assistant allows you to read the meter values from sensors created by [YouLess](https://www.youless.nl/home.html).

The integration is tested and verified for the following devices from YouLess:

* LS110
* LS120 running PVOutput firmware
* LS120 running Enologic firmware

{% include integrations/config_flow.md %}

The integration will create sensors for you to display:

- The current power usage
- The gas meter
- The current meter value (including high and low)
- The delivery meter value, for Solar power
