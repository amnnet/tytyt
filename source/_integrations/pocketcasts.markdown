---
title: Pocket Casts
description: Instructions on how to set up Pocket Casts sensors within Home Assistant.
ha_category:
  - Multimedia
ha_release: 0.39
ha_iot_class: Cloud Polling
ha_domain: pocketcasts
ha_platforms:
  - sensor
---

The `pocketcasts` sensor platform lets you monitor how many unplayed episodes you have of your favorite podcasts at [Pocket Casts](https://play.pocketcasts.com/). This integration requires a [Pocket Casts + Plus](https://www.pocketcasts.com/plus/) subscription to work!

## Configuration

To enable this sensor, add the following lines to your `configuration.yaml`:

```yaml
# Example configuration.yaml entry
sensor:
  - platform: pocketcasts
    username: YOUR_USERNAME
    password: YOUR_PASSWORD
```

{% configuration %}
username:
  description: The username to access the PocketCasts service.
  required: true
  type: string
password:
  description: The password for the given username.
  required: true
  type: string
{% endconfiguration %}
