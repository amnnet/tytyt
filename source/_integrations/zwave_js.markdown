---
title: Z-Wave JS
description: Instructions on how to integrate Z-Wave with Home Assistant via Z-Wave JS.
featured: true
ha_category:
  - Binary Sensor
  - Climate
  - Cover
  - Fan
  - Hub
  - Light
  - Lock
  - Number
  - Select
  - Sensor
  - Siren
  - Switch
ha_release: '2021.2'
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@home-assistant/z-wave'
ha_domain: zwave_js
ha_platforms:
  - binary_sensor
  - climate
  - cover
  - fan
  - light
  - lock
  - number
  - sensor
  - select
  - siren
  - switch
---

This integration allows you to control a Z-Wave network via the [Z-Wave JS](https://zwave-js.github.io/node-zwave-js/#/) driver. This is our recommended Z-Wave integration for Home Assistant.

Please review the limitations [below](/integrations/zwave_js/#current-limitations) before you get started as a few devices still may not work or only work partially.

## Quick start (Home Assistant including Supervisor)

To add Z-Wave JS to your installation, plug the Z-Wave stick into the device that runs Home Assistant. Then Go to Configuration >> Integrations in the UI. Click the "Add integration" button in the bottom right and from the list of integrations, select "Z-Wave JS" and follow the instructions shown.

Note: A new network key is automatically generated for you. If this Z-Wave stick has already paired with secure devices, you need to enter the previously used network key. Make sure that you keep a backup of this key in a safe place in case you need to move your Z-Wave stick to another device.

If you do not run Home Assistant OS (the default installation type) or Home Assistant Supervised, please see the [advanced installation instructions](#advanced-installation-instructions).

### Discovery via USB

Some devices can be auto-discovered, which can simplify the Z-Wave JS setup process. The following devices have been tested with discovery, and offer a quick setup experience:

| Device | Identifier | Vendor |
| -------| ---------- | ------ |
| Aeotec Z-Stick Gen5+ | 0658:0200 | https://aeotec.com/z-wave-usb-stick/ |
| Nortek HUSBZB-1 | 10C4:8A2A | https://www.nortekcontrol.com/products/2gig/husbzb-1-gocontrol-quickstick-combo/ |
| Zooz ZST10 | 10C4:EA60 | https://www.getzooz.com/zooz-zst10-s2-stick.html |
| Z-WaveMe UZB | 0658:0200 | https://z-wave.me/products/uzb/ |

Additional devices may be discoverable, however only devices that have been confirmed discoverable are listed above.

## Using Z-Wave

When the Z-Wave integration starts up, it will interview your entire Z-Wave network. Depending on the number of devices paired with the Z-Wave stick, this can take a while. Information about your devices is stored in cache files by Z-Wave JS. Be aware that (re)starting the Z-Wave server will cause your network to be (partially) unresponsive until the interview process is done.

While your Z-Wave mesh is permanently stored on your stick, the additional metadata is not. When you lose the cache files (for example by switching between any of the below-mentioned ways to run the server) all your nodes will have to be re-interviewed again before they can be properly controlled. You can speed up this process by manually waking up your battery-powered devices. Most of the time this is a press on the button on those devices (see their manual). It is not needed to exclude/re-include devices from the mesh. Just be patient and the devices will appear.

<p class='note'>
Advanced users: Make sure that the server started successfully by inspecting the logs. Give the Z-Wave controller some time to start.
</p>

{% include integrations/config_flow.md %}

## Services

### Service `zwave_js.set_config_parameter`

This service will update a configuration parameter. To update multiple partial parameters in a single call, use the `zwave_js.bulk_set_partial_config_parameters` service.

| Service Data Attribute 	| Required  	| Description                                                                                                                               	|
|------------------------	|-----------	|-------------------------------------------------------------------------------------------------------------------------------------------	|
| `entity_id`            	| no        	| Entity (or list of entities) to set the configuration parameter on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                       	|
| `device_id`            	| no        	| Device ID (or list of device IDs) to set the configuration parameter on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `area_id`            	  | no        	| Area ID (or list of area IDs) for devices/entities to set the configuration parameter on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `parameter`            	| yes       	| The parameter number or the name of the property. The name of the property is case sensitive.                                             	|
| `bitmask`              	| no        	| The bitmask for a partial parameter in hex (0xff) or decimal (255) format. If the name of the parameter is provided, this is not needed. 	|
| `value`                	| yes       	| The target value for the parameter as the integer value or the state label. The state label is case sensitive.                       	|

#### Examples of setting a single parameter value

Let's use parameter 31 for [this device](https://devices.zwave-js.io/?jumpTo=0x000c:0x0203:0x0001:0.0) as an example to show examples of different ways that the `LED 1 Blink Status (bottom)` partial parameter can be set. Note that in places where we are using different values for the same key, the different values are interchangeable across the examples. We can, for instance, use `1` or `Blink` interchangeably for the `value` in all of the examples.

Example 1:

```yaml
service: zwave_js.set_config_parameter
target:
  entity_id: switch.fan
data:
  parameter: 31
  bitmask: 0x01
  value: 1
```

Example 2:

```yaml
service: zwave_js.set_config_parameter
target:
  entity_id: switch.fan
data:
  parameter: 31
  bitmask: 1
  value: "Blink"
```

Example 3:

```yaml
service: zwave_js.set_config_parameter
target:
  entity_id: switch.fan
data:
  entity_id: switch.fan
  parameter: "LED 1 Blink Status (bottom)"
  value: "Blink"
```

### Service `zwave_js.bulk_set_partial_config_parameters`

This service will bulk set multiple partial configuration parameters. Be warned that correctly using this service requires advanced knowledge of Z-Wave to use correctly.

| Service Data Attribute 	| Required  	| Description                                                                                                                               	|
|------------------------	|-----------	|-------------------------------------------------------------------------------------------------------------------------------------------	|
| `entity_id`            	| no        	| Entity (or list of entities) to bulk set partial configuration parameters on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                       	|
| `device_id`            	| no        	| Device ID (or list of device IDs) to bulk set partial configuration parameters on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `area_id`            	  | no        	| Area ID (or list of area IDs) for devices/entities to bulk set partial configuration parameters on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `parameter`            	| yes       	| The parameter number of the property. The name of the property is case sensitive.                                             	|
| `value`                	| yes       	| Either the raw integer value that you want to set for the entire parameter, or a dictionary where the keys are either the bitmasks (in integer or hex form) or the partial parameter name and the values are the value you want to set on each partial (either the integer value or a named state when applicable). Note that when using a dictionary, and bitmasks that are not provided will be set to their currently cached values.                       	|

#### Examples of bulk setting partial parameter values

Let's use parameter 21 for [this device](https://devices.zwave-js.io/?jumpTo=0x031e:0x000a:0x0001:0.0) as an example to show how partial parameters can be bulk set. In this case, we want to set `0xff` to `127`, `0x7f00` to `10`, and `0x8000` to `1` (or the raw value of `4735`).

<div class='note'>

When using the dictionary format to map the partial parameter to values, the cached values for the missing partial parameters will be used. So in examples 2, 3, 4, and 5, the service would use the cached value for partial parameters `0xff0000`, `0x3f000000`, and `0x40000000` because new values haven't been specified. If you send the raw integer value, it is assumed that you have calculated the full value, so in example 1, partial parameters `0xff0000`, `0x3f000000`, and `0x40000000` would all be set to `0`.

</div>

Example 1:

```yaml
service: zwave_js.bulk_set_partial_config_parameters
target:
  entity_id: switch.fan
data:
  parameter: 21
  value: 4735
```

Example 2:

```yaml
service: zwave_js.bulk_set_partial_config_parameters
target:
  entity_id: switch.fan
data:
  parameter: 21
  value:
    0xff: 127
    0x7f00: 10
    0x8000: 1
```

Example 3:

```yaml
service: zwave_js.bulk_set_partial_config_parameters
target:
  entity_id: switch.fan
data:
  parameter: 21
  value:
    255: 127
    32512: 10
    32768: 1
```

Example 4:

```yaml
service: zwave_js.bulk_set_partial_config_parameters
target:
  entity_id: switch.fan
data:
  parameter: 21
  value:
    255: 127
    32512: 10
    32768: "Fine"
```

Example 5:

```yaml
service: zwave_js.bulk_set_partial_config_parameters
target:
  entity_id: switch.fan
data:
  parameter: 21
  value:
    "Quick Strip Effect: Hue Color Wheel / Color Temp": 127
    "Quick Strip Effect Intensity": 10
    "Quick Strip Effect Intensity Scale": "Fine"
```

### Service `zwave_js.refresh_value`

This service will refresh the value(s) for an entity. This service will generate extra traffic on your Z-Wave network and should be used sparingly. Updates from devices on battery may take some time to be received.

| Service Data Attribute 	| Required 	| Description                                                                                                                                      	|
|------------------------	|----------	|--------------------------------------------------------------------------------------------------------------------------------------------------	|
| `entity_id`            	| yes      	| Entity or list of entities to refresh values for.                                                                                                	|
| `refresh_all_values`   	| no       	| Whether all values should be refreshed. If  `false`, only the primary value will be refreshed. If  `true`, all watched values will be refreshed. 	|

### Service `zwave_js.set_value`

This service will set a value on a Z-Wave device. It is for advanced use cases where you need to modify the state of a node and can't do it using native Home Assistant entity functionality. Be warned that correctly using this service requires advanced knowledge of Z-Wave. The service provides minimal validation and blindly calls the Z-Wave JS API, so if you are having trouble using it, it is likely because you are providing an incorrect value somewhere. To set a config parameter, you should use the `zwave_js.set_config_parameter` or `zwave_js.bulk_set_partial_config_parameters` services instead of this one.

| Service Data Attribute 	| Required 	| Description                                                                                                                                      	|
|------------------------	|----------	|--------------------------------------------------------------------------------------------------------------------------------------------------	|
| `entity_id`            	| no        	| Entity (or list of entities) to set the value on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                       	|
| `device_id`            	| no        	| Device ID (or list of device IDs) to set the value on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `area_id`            	  | no        	| Area ID (or list of area IDs) for devices/entities to set the value on. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `command_class`       	| yes       	| ID of Command Class that you want to set the value for. 	                                                                                            |
| `property`            	| yes       	| ID of Property that you want to set the value for. 	                                                                                                  |
| `property_key`        	| no        	| ID of Property Key that you want to set the value for. 	                                                                                              |
| `endpoint`   	          | no        	| ID of Endpoint that you want to set the value for. 	                                                                                                  |
| `value`   	            | yes        	| The new value that you want to set. 	                                                                                                                |
| `options`   	          | no        	| Set value options map. Refer to the Z-Wave JS documentation for more information on what options can be set. 	                                                                                                                |
| `wait_for_result`   	  | no        	| Boolean that indicates whether or not to wait for a response from the node. If not included in the payload, the integration will decide whether to wait or not. If set to `true`, note that the service call can take a while if setting a value on an asleep battery device. |

### Service `zwave_js.multicast_set_value`

This service will set a value on multiple Z-Wave devices using multicast. It is for advanced use cases where you need to set the same value on multiple nodes simultaneously. Be warned that correctly using this service requires advanced knowledge of Z-Wave. The service provides minimal validation beyond what is necessary to properly call the Z-Wave JS API, so if you are having trouble using it, it is likely because you are providing an incorrect value somewhere.

| Service Data Attribute 	| Required 	| Description                                                                                                                                      	|
|------------------------	|----------	|--------------------------------------------------------------------------------------------------------------------------------------------------	|
| `entity_id`            	| no        	| Entity (or list of entities) to set the value on via multicast. At least two `entity_id` or `device_id` must be resolved if not broadcasting the command.                       	|
| `device_id`            	| no        	| Device ID (or list of device IDs) to set the value on via multicast. At least two `entity_id` or `device_id` must be resolved if not broadcasting the command.                                                	|
| `area_id`            	  | no        	| Area ID (or list of area IDs) for devices/entities to set the value on via multicast. At least two `entity_id` or `device_id` must be resolved if not broadcasting the command.                                                	|
| `broadcast`             | no          | Boolean that indicates whether you want the message to be broadcast to all nodes on the network. If you have only one Z-Wave JS network configured, you do not need to provide a `device_id` or `entity_id` when this is set to true. When you have multiple Z-Wave JS networks configured, you MUST provide at least one `device_id` or `entity_id` so the service knows which network to target.                                         |
| `command_class`       	| yes       	| ID of Command Class that you want to set the value for. 	                                                                                            |
| `property`            	| yes       	| ID of Property that you want to set the value for. 	                                                                                                  |
| `property_key`        	| no        	| ID of Property Key that you want to set the value for. 	                                                                                              |
| `endpoint`   	          | no        	| ID of Endpoint that you want to set the value for. 	                                                                                                  |
| `value`   	            | yes        	| The new value that you want to set. 	                                                                                                                |
| `options`   	          | no        	| Set value options map. Refer to the Z-Wave JS documentation for more information on what options can be set. 	                                                                                                                |

### Service `zwave_js.ping`

Calling this service forces Z-Wave JS to try to reach a node. This can be used to update the status of the node in Z-Wave JS when you think it doesn't accurately reflect reality, e.g. reviving a failed/dead node or marking the node as asleep.

| Service Data Attribute 	| Required 	| Description                                                                                                                                      	|
|------------------------	|----------	|--------------------------------------------------------------------------------------------------------------------------------------------------	|
| `entity_id`            	| no        	| Entity (or list of entities) to ping. At least one `entity_id`, `device_id`, or `area_id` must be provided.                       	|
| `device_id`            	| no        	| Device ID (or list of device IDs) to ping. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|
| `area_id`            	  | no        	| Area ID (or list of area IDs) for devices/entities to ping. At least one `entity_id`, `device_id`, or `area_id` must be provided.                                                	|

This service can be used in tandem with the node status sensor to track the node's status and fire the command when needed. Here's an example automation that would ping a node when the node status sensor state has changed to dead and remained dead for 30 minutes. Note that this may be useful for some devices but will definitely not be useful for all. In cases where it is not useful, all you will be doing is generating additional traffic on your Z-Wave network which could slow down communication.

```yaml
trigger:
  platform: state
  entity_id:
    - sensor.z_wave_thermostat_node_status
    - sensor.z_wave_lock_node_status
  to: "dead"
  for: "00:30:00"
action:
  - service: zwave_js.ping
    target:
      entity_id: "{{ trigger.entity_id }}"
```

### Service `zwave_js.reset_meter`

This service will reset the meters on a device that supports the Meter Command Class.

| Service Data Attribute | Required | Description                                                                                                        |
| ---------------------- | -------- | ------------------------------------------------------------------------------------------------------------------ |
| `entity_id`            | yes      | Entity (or list of entities) for the meters you want to reset.               |
| `meter_type`           | no       | If supported by the device, indicates the type of meter to reset. Not all devices support this option.             |
| `value`                | no       | If supported by the device, indicates the value to reset the meter to. Not all devices support this option.   |

### Service `zwave_js.set_lock_usercode`

This service will set the usercode of a lock to X at code slot Y.
Valid usercodes are at least 4 digits.

| Service Data Attribute | Required | Description                                          |
| ---------------------- | -------- | ---------------------------------------------------- |
| `entity_id`            | no       | Lock entity or list of entities to set the usercode. |
| `code_slot`            | yes      | The code slot to set the usercode into.              |
| `usercode`             | yes      | The code to set in the slot.                         |

### Service `zwave_js.clear_lock_usercode`

This service will clear the usercode of a lock in code slot X.
Valid code slots are between 1-254.

| Service Data Attribute | Required | Description                                            |
| ---------------------- | -------- | ------------------------------------------------------ |
| `entity_id`            | no       | Lock entity or list of entities to clear the usercode. |
| `code_slot`            | yes      | The code slot to clear the usercode from.              |

## Events

There are two types of events that are fired, notification events and value notification events. You can test what events come in using the event {% my developer_events title="developer tools in Home Assistant" %} and subscribing to the `zwave_js_notification` or `zwave_js_value_notification` events respectively. Once you know what the event data looks like, you can use this to create automations.

### Node events (Notification)

Check the [Z-Wave JS notification event documentation](https://zwave-js.github.io/node-zwave-js/#/api/node?id=quotnotificationquot) for an explanation of the notification event data. These events fire with the `zwave_js_notification` event type.

#### Notification Command Class

These are notification events fired by devices using the Notification command class. The `parameters` attribute in the example below is optional, and when it is included, the keys in the attribute will vary depending on the event.

```json
{
    "domain": "zwave_js",
    "node_id": 1,
    "home_id": "974823419",
    "device_id": "ad8098fe80980974",
    "command_class": 113,
    "command_class_name": "Notification",
    "type": 6,
    "event": 5,
    "label": "Access Control",
    "event_label": "Keypad lock operation",
    "parameters": {"userId": 1}
}
```

#### Entry Control Command Class

These are notification events fired by devices using the Entry Control command class.

```json
{
    "domain": "zwave_js",
    "node_id": 1,
    "home_id": "974823419",
    "device_id": "ad8098fe80980974",
    "command_class": 111,
    "command_class_name": "Entry Control",
    "event_type": 6,
    "data_type": 5,
    "event_data": "555"
}
```

### Scene events (Value Notification)

Value Notifications are used for stateless values, like `Central Scenes` and `Scene Activation`. These events fire with the `zwave_js_value_notification` event type.

Value Notification example:

```json
{
    "domain": "zwave_js",
    "node_id": 1,
    "home_id": "974823419",
    "endpoint": 0,
    "device_id": "ad8098fe80980974",
    "command_class": 91,
    "command_class_name": "Central Scene",
    "label": "Event value",
    "property": "scene",
    "property_name": "scene",
    "property_key": "001",
    "property_key_name": "001",
    "value": "KeyPressed",
    "value_raw": 0
}
```

### Value updated events

Due to some devices not following the Z-Wave spec, there are scenarios where a device will send a value update but a state change won't be detected in Home Assistant. To address the gap, the `zwave_js_value_updated` event can be listened to to capture any value updates that are received by an affected entity. This event is **enabled on a per device and per entity domain basis**, and the entities will have `assumed_state` set to `true`. This change will affect how the UI for these entities look; if you'd like the UI to match other entities of the same type where `assumed_state` is not set to `true`, you can override the setting via [entity customization](/docs/configuration/customizing-devices/#assumed_state).

The following devices currently support this event:

| Make            	| Model                            	| Entity Domain 	|
|-----------------	|----------------------------------	|---------------	|
| Vision Security 	| ZL7432 In Wall Dual Relay Switch 	| `switch`      	|

Value Updated example:

```json
{
    "node_id": 4,
    "home_id": "974823419",
    "device_id": "ad8098fe80980974",
    "entity_id": "switch.in_wall_dual_relay_switch",
    "command_class": 37,
    "command_class_name": "Switch Binary",
    "endpoint": 0,
    "property": "currentValue",
    "property_name": "currentValue",
    "property_key": null,
    "property_key_name": null,
    "value": 0,
    "value_raw": 0
}
```

This event can be used to trigger a refresh of values when the new state needs to be retrieved. Here's an example automation:

```yaml
trigger:
  platform: event
  event_type: zwave_js_value_updated
  event_data:
    entity_id: switch.in_wall_dual_relay_switch
action:
  - service: zwave_js.refresh_value
    target:
      entity_id: switch.in_wall_dual_relay_switch_2, switch.in_wall_dual_relay_switch_3
```

## Automations

### Device automations

Z-Wave JS has support for device triggers and conditions. To use a device automation, use the automation UI to select the "Device" trigger/condition type, and then pick your Z-Wave JS device. Under trigger/condition types, you will see Z-Wave JS specific entries.

### `zwave_js.value_updated` trigger

Z-Wave JS provides the `zwave_js.value_updated` trigger platform which can be used to trigger automations on any Z-Wave JS value update, including Z-Wave values that aren't supported in Home Assistant via entities. While they can't be authored from the automation UI, they can be authored in YAML directly in your `configuration.yaml`.

```yaml
# Example automation trigger that fires whenever the `latchStatus` value changes from `closed` to `opened` on the three devices (devices will be derived from an entity ID).
trigger:
  platform: zwave_js.value_updated
  # At least one `device_id` or `entity_id` must be provided
  device_id: 45d7d3230dbb7441473ec883dab294d4  # Garage Door Lock device ID
  entity_id:
    - lock.front_lock
    - lock.back_door
  # `property` and `command_class` are required
  command_class: 98 # Door Lock CC
  property: "latchStatus"
  # `property_key` and `endpoint` are optional
  property_key: null
  endpoint: 0
  # `from` and `to` will both accept lists of values and the trigger will fire if the value update matches any of the listed values
  from:
    - "closed"
    - "jammed"
  to: "opened"
```

#### Available Trigger Data

In addition to the [standard automation trigger data](/docs/automation/templating/#all), the `zwave_js.value_updated` trigger platform has additional trigger data available for use.

| Template variable            | Data                                                                                       |
|------------------------------|--------------------------------------------------------------------------------------------|
| `trigger.device_id`          | Device ID for the device in the device registry.                                           |
| `trigger.node_id`            | Z-Wave node ID.                                                                            |
| `trigger.command_class`      | Command class ID.                                                                          |
| `trigger.command_class_name` | Command class name.                                                                        |
| `trigger.property`           | Z-Wave Value's property.                                                                   |
| `trigger.property_name`      | Z-Wave Value's property name.                                                              |
| `trigger.property_key`       | Z-Wave Value's property key.                                                               |
| `trigger.property_key_name`  | Z-Wave Value's property key name.                                                          |
| `trigger.endpoint`           | Z-Wave Value's endpoint.                                                                   |
| `trigger.previous_value`     | The previous value for this Z-Wave value (translated to a state name when possible).       |
| `trigger.previous_value_raw` | The raw previous value for this Z-Wave value (the key of the state when a state is named). |
| `trigger.current_value`      | The current value for this Z-Wave value (translated to a state name when possible).        |
| `trigger.current_value_raw`  | The raw current value for this Z-Wave value (the key of the state when a state is named).  |

## Current Limitations

- While support for the most common devices is working, some command classes are not yet (fully) implemented in Z-Wave JS. You can track the status [here](https://github.com/zwave-js/node-zwave-js/issues/6).

You can keep track of the Roadmap for the Z-Wave JS integration [here](https://github.com/home-assistant-libs/zwave-js-server-python/issues/56).

## Migrating from previous Z-Wave implementations

If you're new to Home Assistant, use Z-Wave JS.

The `zwave` and `ozw` integrations have been marked as deprecated and will no longer receive any updates.

It is perfectly doable to switch over from one of the above mentioned previous integrations to the new Z-Wave JS integration. The good news is that your entire Z-Wave network is **stored on your stick** so you will not have to run through your house to recreate your network.

If you are currently running the [`zwave`](/integrations/zwave/) or [`ozw`](/integrations/ozw/) Z-Wave integration we recommend you to migrate to Z-Wave JS. All development focus now goes to Z-Wave JS. The previous implementations are provided as-is. They will not be removed without proper notice but in time there might come technical dependencies that render one or both of those integrations unusable.

### Automatic migration wizard

For the `zwave` integration there is a migration wizard that will help you set up the Z-Wave JS integration, remove the `zwave` integration and migrate the entities and devices that can be mapped from the `zwave` integration to the Z-Wave JS integration. Some entities may not be able to migrate automatically and you will need to rename the corresponding available Z-Wave JS entities manually, after the migration. Before completing the migration you will be shown a list of entities that could not be migrated automatically, and you'll have the option to abort or continue with the migration. The migration wizard is available from the `zwave` integration configuration panel in the GUI.

There is no automatic migration wizard for the `ozw` integration. Please follow the manual migration path below if you want to migrate from `ozw` to Z-Wave JS.

### In a nutshell this is what the migration path looks like

1) Make a **backup** of your Home Assistant configuration. If you're running the supervisor this is very easy to do by creating a backup. You should do this so you'll be able to quickly revert if you may run into unexpected problems.

   <div class='note info'>Write down/copy your Z-Wave network key somewhere, you are going to need it later.</div>

   <div class='note info'>Make a list of what node ID belongs to each device. Your network (Nodes and their config etc) is stored on the stick but the names you gave your devices and entities are not. This step is optional but will save you a lot of time later.</div>

2) Remove the Z-Wave integration from Home Assistant: Configuration --> Integrations --> Z-Wave (or OpenZWave) --> Press the three dots and click Delete.

    <div class='note info'>

    If you have configured Z-Wave manually, make sure to also remove the `zwave:` section from your `configuration.yaml`.

    </div>
 
3) If you were running the OpenZWave beta, make sure to stop (or even remove) the OpenZWave add-on, also make sure it doesn't start automatically at startup.

4) Restart your Home Assistant host. This step is important to make sure that your Z-Wave stick is released by the operating system.

5) Install the Z-Wave JS Server of your choice. If you run the supervisor and you'd like to run the standard add-on, you can skip this step if you want. The add-on is installed automatically for you when you choose so in the integration set-up. Remember to fill in the network key you've saved before.

6) Set up the Z-Wave JS integration and connect it to the server. You should see your nodes being detected by Home Assistant. Carefully watch if the status of the node is "ready". This means it's been fully interviewed (and those details cached) by the Z-Wave JS driver. Battery-powered nodes will only be interviewed when they wake up (at scheduled intervals) which can take from a few hours to a few days. To speed that up, you might want to consider waking the device up once. The manual of your device will tell you how to do a manual wake.

7) Once a node hits the ready state, the entities will be created (so not before). Only at this point, is it safe to rename the device (and so its entities). You will thank yourself at this point for having that list noted down of nodes and their names. This is actually the only real hard part of the migration as you will need to name all your devices again.

8) Enjoy your super fast up-to-date Z-Wave network in Home Assistant with support for all modern devices!

#### Need more help with your migration to Z-Wave JS?

There are a few topics created on the forums that might be of your interest:

- [OpenZwave (beta) -> Z-Wave JS Official add-on](https://community.home-assistant.io/t/switching-from-openzwave-beta-to-zwave-js/276723)

- [OpenZwave (beta) -> ZwaveJS2MQTT](https://community.home-assistant.io/t/switching-from-openzwave-beta-to-zwavejs2mqtt/276724)

- [Z-Wave legacy (1.4) -> Z-Wave JS Official add-on](https://community.home-assistant.io/t/switching-from-zwave-1-4-to-zwave-js/276718/2)

- [Z-Wave legacy (1.4) -> ZwaveJS2MQTT](https://community.home-assistant.io/t/switching-from-zwave-1-4-to-zwavejs2mqtt/276721)

You can also visit the `#zwave` channel on [our discord](/join-chat/).

## Advanced installation instructions

The above instructions won't work if you are using Home Assistant Container, Home Assistant Core, or you don't want to use the built-in Z-Wave JS Server add-on. Below you'll find the more detailed set-up instructions that covers all use cases.

### Requirements

Controlling your Z-Wave network using the Z-Wave JS integration has the following requirements:

1. Run [Z-Wave JS Server](https://github.com/zwave-js/zwave-js-server). This application provides the connection between your Z-Wave USB stick and Home Assistant. This server runs separately from Home Assistant so your Z-Wave mesh will keep running if you restart or stop Home Assistant. The Home Assistant Z-Wave JS integration connects to this server via a websocket connection. You need to run this Z-Wave server before you can use the integration.

2. [Supported Z-Wave dongle](/docs/z-wave/controllers/#supported-z-wave-usb-sticks--hardware-modules). The Z-Wave controller dongle should be connected to the same host as where the Z-Wave JS server is running. In the configuration for the Z-Wave server you need to provide the path to this stick. It's recommended to use the `/dev/serial-by-id/yourdevice` version of the path to your stick, to make sure the path doesn't change over reboots. The most common known path is `/dev/serial/by-id/usb-0658_0200-if00`.

3. A **network key**. This key is used in order to connect securely to compatible devices. The network key consists of 32 hexadecimal characters, for example `2232666D100F795E5BB17F0A1BB7A146` (do not use this one, pick a random one). Without a network key security enabled devices cannot be added securely and will not function correctly. You must provide this network key in the configuration part of the Z-Wave JS Server. For new installations, a unique default key will be auto generated for you. TIP: You could use a site like random.org to create your own random network key. Make sure that you keep a backup of this key in a safe place. You will need to enter the same key to be able to access the securely paired devices.

4. The Z-Wave JS integration in Home Assistant. This integration connects to the Z-Wave JS Server to retrieve the info from your Z-Wave network and turns it into Home Assistant devices and entities.

### Running the Z-Wave JS Server

As stated in the above requirements, you need to run the Z-Wave JS Server somewhere in your network. There are multiple ways to run this server, we'll explain the most common ways below:

#### 1. The official Z-Wave JS add-on, available from the add-on store

_This option is only available for Home Assistant OS (the recommended installation type) and Home Assistant Supervised installations._

This add-on runs the Z-Wave JS server. It can be automatically installed and configured by Home Assistant as part of setting up the integration. You can also set it up manually.

The Z-Wave network can be configured via the built-in Z-Wave JS control panel in Home Assistant.

#### 2. The Z-Wave JS to MQTT add-on installed available from the community add-on store

_This option is only available for Home Assistant OS (the recommended installation type) and Home Assistant Supervised installations._

This add-on includes the Z-Wave JS Server as part of the Z-Wave JS 2 MQTT application. Despite the name, MQTT is not required to run this add-on.

The Z-Wave network can be configured via the built-in Z-Wave JS control panel in Home Assistant and via the Z-Wave JS control panel built into Z-Wave JS 2 MQTT.

Despite what the name suggests, you can actually run this add-on without MQTT enabled. In that case, it will provide you with a full-fledged, attractive and feature-complete UI to manage your Z-Wave nodes and settings.


#### 3. The Z-Wave JS to MQTT Docker container

This is the recommended approach if you're running Home Assistant Container. See the [zwavejs2mqtt documentation](https://zwave-js.github.io/zwavejs2mqtt/#/getting-started/quick-start) for instructions.

After installing the Docker image, make sure you enable the Z-Wave JS Server in the configuration.

#### 4. Run the Z-Wave server yourself

This is considered a very advanced use case. In this case you run the Z-Wave JS Server or zwavejs2mqtt NodeJS application directly. Installation and maintaining this is out of scope for this document. See the [Z-Wave JS server](https://github.com/zwave-js/zwave-js-server) or [zwavejs2mqtt](https://github.com/zwave-js/zwavejs2mqtt) GitHub repository for information.

### Installing and configuring the Z-Wave JS integration in Home Assistant

Once you have the Z-Wave server up and running, it's time to configure the integration in Home Assistant. This integration can be configured using the integrations in the Home Assistant frontend:

1. Click on the `+` sign to add an integration and click on **Z-Wave JS**.

2. If you're running full Home Assistant with supervisor, you will be presented with a dialog that asks if you want to use the Z-Wave JS Supervisor add-on. Check the box if you prefer this option. If you run the server yourself, or prefer the alternative zwavejs2mqtt addon, uncheck this box. After completing the configuration flow, the Z-Wave JS integration will be
available.

3. If you're not running the supervisor or you've unchecked the above mentioned box, you will be asked to enter a websocket URL (defaults to ws://localhost:3000). It is very important that you fill in the correct (docker) IP/hostname here. For example for the Z-Wave JS to MQTT add-on this is `ws://a0d7b954-zwavejs2mqtt:3000`.

## Frequently Asked Questions

### What Z-Wave devices are currently supported?

See the [Z-Wave JS device database](https://devices.zwave-js.io/).

### Can I switch between the Official Z-Wave JS add-on and Z-Wave JS to MQTT?

You can, but you cannot run them at the same time. Only one of them can be active at the same time.

When you switch add-ons, re-add the Z-Wave integration to Home Assistant with the new/updated WebSocket URL. There will be a popup raised that this Z-Wave network is already configured but "under the hood" the WebSocket URL is adjusted.

### I do not see any entities created for my device in Home Assistant

Entities will be created only after the node hits the ready state (interview is completed). Also, note that some devices (like button remotes) do not create any entities but will only provide events when a button is pressed. See the events section on how to handle those events in your automations. If you are certain that your device should have entities and you do not see them (even after a restart of Home Assistant Core), that will be the time to create an issue about your problem on the GitHub issue tracker, see below section of troubleshooting issues.

### Is there a way to easily export a dump of all my current Z-Wave nodes before I migrate?

You can run the script below in the Developer Tools to get a full oversight of your nodes and their entities.

{% raw %}

```yaml
{%- for node, zstates in states | selectattr('attributes.node_id', 'in', range(1000)) | groupby('attributes.node_id') %}

{%- for s in zstates %}
Node {{ node }};{{ s.name }};{{ s.entity_id }}{% endfor %}

{%- endfor %}
```

{% endraw %}

### How can I add (include) a new device to my Z-Wave network?

1. In Home Assistant: open Configuration -> Integrations -> Z-Wave JS -> Configure.
2. Press `Add node`.
3. Press `Start Inclusion`. The Z-Wave controller is now in inclusion mode and will not respond to other commands.
4. Put the device you want to add in inclusion mode. Refer to its manual how this is done.
5. The UI should confirm that the node was added and it will be immediately visible in Home Assistant. After a short while (seconds to minutes) the entities should also be created.
6. If the controller fails to add/find your device, cancel the inclusion process (to unblock your network again). In some cases it might help to first remove a node (exclusion) before you add it, even when the device has not been added to this Z-Wave network yet. Another approach would be to factory reset the device. Info about that is in the manual of your device.

<div class='note info'>
While adding devices, you have the option to use `secure inclusion`, this means that the traffic between the controller and the device will be encrypted. This adds additional overhead to the Z-Wave network so use this option with care. As a general rule of thumb it is advised to only securely include devices that actually NEED this kind of protection. A good example is a Z-Wave door lock.
</div>

### How can I remove (exclude) a device from my Z-Wave network?

1. In Home Assistant: open Configuration -> Integrations -> Z-Wave JS -> Configure.
2. Press `Remove node`.
3. Press `Start Exclusion`. The Z-Wave controller is now in exclusion mode and will not respond to other commands.
4. Put the device you want to remove in exclusion mode. Refer to its manual how this is done.
5. The UI should confirm that the node was removed and the device and entities will be removed from Home Assistant.

### Where do I need to enter the network key?

- You enter the network key during integration setup if you do not have the add-on installed.
- Official Z-Wave JS add-on: In the add-on configuration, directly in the supervisor.
- Z-Wave JS 2 MQTT: In the web UI, go to Settings -> Z-Wave -> Network Key.

### How can I use my OZW network key in zwavejs2mqtt?

You can use your existing network key in zwavejs2mqtt but you need to slightly adjust it.
The OZW looks like this: `0x01, 0x02, 0x03 etc.` while the network key format accepted in zwavejs2mqtt looks like this `0102030405 etc.`. You can simply edit your existing key and remove the `"0x"` part and the `", "` part so it becomes one large string of numbers.

### What's the benefit of using Z-Wave JS to MQTT over the official Add-On?

The official add-on provides the Z-Wave Server in it's bare minimum variant, just enough to serve the Home Assistant integration.
The Z-Wave JS to MQTT project includes the Z-Wave JS Server for convenience but also provides a Z-Wave Control panel and the ability (hence its name) to serve your Z-Wave network to MQTT. You can leave the MQTT Gateway disabled and only use the Control panel but you can even have the MQTT features enabled at the same time. For example to interact with Z-Wave from other devices, while the Home Assistant integration still works (as long as you keep the WS Server enabled in zwavejs2mqtt).

### Z-Wave JS to MQTT seems to provide Home Assistant integration on its own too, now I'm confused

Correct, the Z-Wave (JS) to MQTT project existed before Home Assistant even had plans to move to the Z-Wave JS Driver.
The Home Assistant integration that exists in zwavejs2mqtt is based on MQTT discovery.
The official Z-Wave JS integration is not based on MQTT and is talking directly to the Z-Wave JS Driver (using the WS Server). This gives a better experience.

### Can I run Z-Wave JS to MQTT only for the control panel and nothing else?

Sure, in the settings of zwavejs2mqtt, make sure to enable "WS Server" and disable "Gateway".

### My device does not automatically update its status in HA if I control it manually

Your device might not send automatic status updates to the controller. While the best advice would be to update to recent Z-Wave Plus devices, there is a workaround with active polling (request the status) at some interval. See the section below for more info about this.

### What about polling of devices?

Some legacy devices don't report all their values automatically and require polling to get updated values when controlled manually. Z-Wave JS does not automatically poll devices on a regular basis without user interaction. Polling can quickly lead to network congestion and should be used very sparingly and only where necessary.

- We provide a `zwave_js.refresh_value` service to allow you to manually poll a value, for example from an automation that only polls a device when there is motion in that same room. If you **really** need polling, you can enable this in zwavejs2mqtt but not in the official add-on.

- zwavejs2mqtt allows you to configure scheduled polling on a per-value basis, which you can use to keep certain values updated. It also allows you to poll individual values on-demand from your automations, which should be preferred over blindly polling all the time if possible.

<div class='note warning'>
Polling is considered bad practice and should only be used as a last resort when you use it with care and accept the negative impact on your network. Z-Wave is a very low speed network and poll requests can easily flood your network and slow down your commands.
</div>

### My device is recognized as Unknown Manufacturer and/or some of its functionalities do not work in Z-Wave JS

When your device is not yet fully interviewed, this info will not yet be present. So make sure your device is interviewed at least once. Is the state of your device reported as ready and you still see Unknown Manufacturer, read on.

Z-Wave JS keeps a database of all devices it supports, including any special treatments they need. These are called the device configuration files and they are contributed mainly by the community. Is your device not fully supported, consider [contributing the device configuration file](https://zwave-js.github.io/node-zwave-js/#/config-files/contributing-files).

### I get a lot of sensor entities for my device that I'm probably never going to use

The integration will add as many usable entities for you as possible from the information it retrieves from your Z-Wave devices. Entities that you don't want/like, can be disabled within the Home Assistant interface.

### I renamed my devices in Z-Wave JS 2 MQTT but those names are not visible in Home Assistant

The names are only loaded when the Z-Wave JS integration is started. For Home Assistant
to pick up those new names, either reload the integration or restart Home Assistant.

## Troubleshooting Issues

### Get a dump of the current network state

When trying to determine why something isn't working as you expect, or when reporting an issue with the integration, it is helpful to know what Z-Wave JS sees as the current state of your Z-Wave network. To get a dump of your current network state, follow the menu:

**Configuration** -> **Integrations** -> **Z-Wave JS** -> **Configure** -> **Download a dump of your network to help diagnose issues**

### Interference issues

Many users have reported issues with interference when the USB stick was directly connected to the machine (proximity). If you are having issues try to use a short USB 2.0 A male to female extension cord.
