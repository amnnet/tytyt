---
title: UptimeRobot
description: Instructions on how to set up Uptime Robot within Home Assistant.
ha_category:
  - System Monitor
ha_release: 0.72
ha_iot_class: Cloud Polling
ha_codeowners:
  - '@ludeeus'
ha_domain: uptimerobot
ha_platforms:
  - binary_sensor
ha_quality_scale: platinum
ha_config_flow: true
---

The `uptimerobot` integration provides binary sensors to get the status for all of your monitors from your account on [UptimeRobot]( https://uptimerobot.com).

{% include integrations/config_flow.md %}

To get your API key, go to [My Settings](https://uptimerobot.com/dashboard#mySettings) on the UptimeRobot website, at the bottom you will find your "Read-Only API Key".

All the data will be fetched from [UptimeRobot](https://uptimerobot.com).
