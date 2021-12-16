---
title: AirNow
description: Instructions on how to integrate AirNow within Home Assistant.
ha_category:
  - Health
ha_release: 2021.2
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_codeowners:
  - '@asymworks'
ha_domain: airnow
ha_platforms:
  - sensor
---

The `airnow` integration uses the [AirNow](https://www.airnow.gov/) web service
as a source for air quality data for your location.

## Setup

To generate an AirNow API key, go to the [AirNow Developer Tools Page](https://docs.airnowapi.org/account/request/) page.

{% include integrations/config_flow.md %}

<div class="note">

The AirNow API allows 500 data updates per hour, but since observations are only updated hourly, the default update rate is set to 2 per hour and should not trigger rate limiting. If you use this API key for other purposes, ensure the total request rate does not exceed 500 per hour.

</div>
