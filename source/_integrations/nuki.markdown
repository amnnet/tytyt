---
title: Nuki
description: Instructions on how to integrate a Nuki Smart Lock devices.
ha_category:
  - Lock
ha_release: 0.38
ha_iot_class: Local Polling
ha_codeowners:
  - '@pschmitt'
  - '@pvizeli'
  - '@pree'
ha_domain: nuki
ha_platforms:
  - binary_sensor
  - lock
ha_config_flow: true
ha_dhcp: true
---

The Nuki integration allows you to control [Nuki Smart Locks](https://nuki.io/en/smart-lock/) via a [Nuki Bridge](https://nuki.io/en/bridge/).

## Prerequisites

To add a Nuki bridge to your installation, you need to enable developer mode on your bridge and define a port and an access token. This can be achieved using the [Android app](https://play.google.com/store/apps/details?id=io.nuki) or [iPhone app](https://apps.apple.com/app/nuki-smart-lock/id1044998081). Go to manage my devices, and select the bridge. Within the bridge configuration turn on the HTTP API and check the details in the screen. Please note that the API token should be 6-20 characters long, even though the app allows you to set a longer one.

{% include integrations/config_flow.md %}

## Services

### Service `nuki.lock_n_go`

This will first unlock, wait a few seconds (20 by default) then re-lock. The wait period can be customized through the app.
See the [Nuki Website](https://nuki.io/en/support/smart-lock/sl-features/locking-with-the-smart-lock/) for more details about this feature.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of strings that point at `entity_id`s Nuki Locks.
| `unlatch` | yes | Boolean - Whether to unlatch the door when first opening it.

### Service `nuki.set_continuous_mode`

This service allows you to enable or disable the "Continuous Mode" feature of Nuki Openers. This is similar to the Ring-to-Open feature that is mapped to "lock/unlock", except that it doesn't have a time limit - as long as this mode is enabled, the door will open when the buzzer button is pressed, similar to how it works at e.g. a doctor's office or other business during the day. On other Nuki products, this service is a no-op.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of strings that point at `entity_id`s Nuki Locks.
| `enabled` | yes | Boolean - Set to `true` to enable Continuous Mode, or `false` to disable.
