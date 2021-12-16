---
title: Zabbix
description: Instructions on how to integrate Zabbix into Home Assistant.
ha_category:
  - System Monitor
  - Sensor
ha_release: 0.37
ha_iot_class: Local Polling
ha_domain: zabbix
ha_platforms:
  - sensor
---

The `zabbix` integration is the main integration to connect to a [Zabbix](https://www.zabbix.com/) monitoring instance via the Zabbix API.

It is possible to publish Home Assistant state changes to Zabbix. In Zabbix a host has to be created which will contain the Home Assistant states as individual items. These items are automatically created using Zabbix Low-Level Discovery (LLD). In order to make setup in Zabbix easy, you can use this [template](/assets/integrations/zabbix/zbx_template_home_assistant.xml) for the host.

There is currently also support for the following device types within Home Assistant:

- [Sensor](#sensor)

## Configuration

To set the Zabbix integration up, add the following information to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
zabbix:
  host: IP_ADDRESS
```

{% configuration %}
host:
  description: Your Zabbix server.
  required: true
  type: string
path:
  description: Path to your Zabbix install.
  required: false
  type: string
  default: "`/zabbix/`"
ssl:
  description: Set to `true` if your Zabbix installation is using SSL.
  required: false
  type: boolean
  default: false
username:
  description: Your Zabbix username.
  required: false
  type: string
password:
  description: Your Zabbix password.
  required: false
  type: string
publish_states_host:
  description: The host that will receive the state changes from Home Assistant. It needs to be manually created in Zabbix first and have the template associated with it (see above).
  required: false
  type: string
exclude:
  type: list
  description: Configure which integrations should be excluded from being published to Zabbix. ([Configure Filter](#configure-filter))
  required: false
  keys:
    entities:
      type: [string, list]
      description: The list of entity ids to be excluded from being published to Zabbix.
      required: false
    entity_globs:
      type: [string, list]
      description: Exclude all entities matching a listed pattern.
      required: false
    domains:
      type: [string, list]
      description: The list of domains to be excluded from being published to Zabbix.
      required: false
include:
  type: list
  description: Configure which integrations should be included in being published to Zabbix. If set, all other entities will not be published to Zabbix. ([Configure Filter](#configure-filter))
  required: false
  keys:
    entities:
      type: [string, list]
      description: The list of entity ids to be included in being published to Zabbix.
      required: false
    entity_globs:
      type: [string, list]
      description: Include all entities matching a listed pattern.
      required: false
    domains:
      type: [string, list]
      description: The list of domains to be included in being published to Zabbix.
      required: false
{% endconfiguration %}

### Full configuration

```yaml
# Example configuration.yaml entry
zabbix:
  host: ZABBIX_HOST
  path: ZABBIX_PATH
  ssl: false
  username: USERNAME
  password: PASSWORD
  publish_states_host: homeassistant
  exclude:
    domains:
      - device_tracker
    entities:
      - sun.sun
      - sensor.time
```

## Configure Filter

By default, no entity will be excluded. To limit which entities are being published to Zabbix, you can use the `include` and `exclude` parameters.

{% raw %}

```yaml
# Example filter to include specified domains and exclude specified entities
zabbix:
  include:
    domains:
      - alarm_control_panel
      - light
    entity_globs:
      - binary_sensor.*_occupancy
  exclude:
    entities:
      - light.kitchen_light
```

{% endraw %}

Filters are applied as follows:

1. No includes or excludes - pass all entities
2. Includes, no excludes - only include specified entities
3. Excludes, no includes - only exclude specified entities
4. Both includes and excludes:
   - Include domain and/or glob patterns specified
      - If domain is included, and entity not excluded or match exclude glob pattern, pass
      - If entity matches include glob pattern, and entity does not match any exclude criteria (domain, glob pattern or listed), pass
      - If domain is not included, glob pattern does not match, and entity not included, fail
   - Exclude domain and/or glob patterns specified and include does not list domains or glob patterns
      - If domain is excluded and entity not included, fail
      - If entity matches exclude glob pattern and entity not included, fail
      - If entity does not match any exclude criteria (domain, glob pattern or listed), pass
   - Neither include or exclude specifies domains or glob patterns
      - If entity is included, pass (as #2 above)
      - If entity include and exclude, the entity exclude is ignored

## Sensor

The `zabbix` sensor platform let you monitor the current count of active triggers for your [Zabbix](https://www.zabbix.com/) monitoring instance.

<div class='note'>
You must have the <a href="#configuration">Zabbix component</a> configured to use those sensors.
</div>

To set it up, add the following information to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
sensor:
  - platform: zabbix
    triggers:
      name: Important Hosts Trigger Count
      hostids: [10051,10081,10084]
      individual: true
```

{% configuration %}
triggers:
  description: Specifies that this sensor is for Zabbix 'triggers'. In the future there will be other Zabbix sensors.
  required: true
  type: string
name:
  description: Allows you to specify the name for the Sensor, otherwise the host name, as stored in Zabbix, is used. This is useful when you are specifying a list of hostids to monitor as a single count.
  required: false
  type: string
hostids:
  description: This is a list of Zabbix hostids that we want to filter our count on.
  required: false
  type: string
individual:
  description: A 'true'/'false' to specify whether we should show individual sensors when a list of hostids is provided. If false, the sensor state will be the count of all triggers for the specified hosts (or all hosts within the Zabbix instance, if hostids isn't provided).
  required: false
  type: boolean
  default: false
{% endconfiguration %}
