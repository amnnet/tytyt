---
title: KNX
description: Instructions on how to integrate KNX components with Home Assistant.
ha_category:
  - Hub
  - Binary Sensor
  - Climate
  - Cover
  - Fan
  - Light
  - Notifications
  - Number
  - Scene
  - Select
  - Sensor
  - Switch
  - Weather
ha_release: 0.24
ha_iot_class: Local Push
ha_codeowners:
  - '@Julius2342'
  - '@farmio'
  - '@marvin-w'
ha_domain: knx
ha_quality_scale: silver
ha_platforms:
  - binary_sensor
  - climate
  - cover
  - fan
  - light
  - notify
  - number
  - scene
  - sensor
  - select
  - switch
  - weather
---

The [KNX](https://www.knx.org) integration for Home Assistant allows you to connect to KNX/IP devices.

The integration requires a local KNX/IP interface or router. Through this, it will establish a connection between Home Assistant and your KNX bus.

<div class='note warning'>

Please note, the KNX platform does not support KNX Secure.

</div>

There is currently support for the following device types within Home Assistant:

- [Binary Sensor](#binary-sensor)
- [Climate](#climate)
- [Cover](#cover)
- [Fan](#fan)
- [Light](#light)
- [Notify](#notify)
- [Number](#number)
- [Scene](#scene)
- [Select](#select)
- [Sensor](#sensor)
- [Switch](#switch)
- [Weather](#weather)

## Basic Configuration

To use your KNX devices from Home Assistant, add the following lines to your `configuration.yaml` file:

```yaml
knx:
```

In order to make use of the various platforms that KNX offers you will need to add the relevant configuration sections to your setup. This could either all be in the Home Assistant main `configuration.yaml` file, or in a separate YAML file that you include in the main file or even be split into multiple dedicated files. See [Splitting up the configuration](/docs/configuration/splitting_configuration/).

```yaml
knx:
  # configure platforms directly in configuration.yaml
  binary_sensor:
    - name: "My first binary sensor"
      state_address: "1/2/3"
    # etc...
  # or outsource platform configuration to separate files
  sensor: !include knx_sensor.yaml
```

Please see the dedicated platform sections below about how to configure them correctly.

{% configuration %}
individual_address:
  description: The KNX individual address (IA) that shall be used for routing or if a tunneling server doesn't assign an IA at connection.
  required: false
  type: string
  default: "15.15.250"
multicast_group:
  description: The multicast group to use for automatic interface discovery and routing communication.
  required: false
  type: string
  default: "224.0.23.12"
multicast_port:
  description: The port for multicast communication.
  required: false
  type: integer
  default: 3671
rate_limit:
  description: Defines the maximum number of telegrams to be sent to the bus per second (range 1-100).
  required: false
  default: 20
  type: integer
state_updater:
  description: The integration will collect the current state of each configured device from the KNX bus to display it correctly within Home Assistant. Set this option to False to prevent this behavior.
  required: false
  default: true
  type: boolean
{% endconfiguration %}

### Group addresses

Group addresses are configured as strings or integers in the format "1/2/3" for 3-level GA-structure, "1/2" for 2-level GA-structure or "1" for free GA-structure.

The HA KNX integration uses configured `state_address` or `*_state_address` to update the state of a function. These addresses are read by GroupValueRead requests on startup and when there was no incoming telegram for one hour (default `sync_state`).

It is possible to configure passive/listening group addresses for all functions of every KNX platform (except `expose` and `notify`). This allows having multiple group addresses to update the state of its function (e.g., the brightness of a light). When group addresses are configured as a list of strings, the first item is the active sending or state-reading address and the rest is registered as passive addresses. This schema behaves like in ETS configuration where the first is the "sending" address and others are just for updating the communication object.

If your KNX device provides active state communication objects it is advised to use `*_state_address` instead of passive addresses as it reduces configuration complexity and avoids wrong states (e.g., when channels are logically locked).

```yaml
knx:
  switch:
    - name: "Switch without passive addresses"
      address: "1/1/1" # this is the address that will be sent to
      state_address: "8/8/8"  # this is the address GroupValueRead requests are sent to
    - name: "Switch with passive addresses"
      address: 
        - "1/1/1" # this is the address that will be sent to
        - "1/1/2" # this and following are passive addresses
        - "1/1/3"
      state_address: 
        - "8/8/8" # this is the address GroupValueRead requests are sent to
        - "8/8/2" # this and following are passive addresses
        - "8/8/3"
```

## Connection

Under normal conditions no connection configuration should be needed. The integration will auto-detect KNX/IP interfaces and connect to one. This requires multicast communication to work in your environment.

### Tunneling

If you want to connect to a specific tunneling server or if the auto detection of the KNX/IP device does not work the IP or/and port of the tunneling device can be configurated.

```yaml
knx:
  tunneling:
    host: "192.168.2.23"
```

{% configuration %}
host:
  description: IP address of the KNX/IP tunneling device.
  type: string
  required: true
port:
  description: Port of the KNX/IP tunneling device.
  type: integer
  required: false
local_ip:
  description: IP address of the local interface.
  type: string
  required: false
route_back:
  description: When True the KNXnet/IP Server shall use the IP address and the port number from the IP package received as the target IP address or port number for the response to the KNXnet/IP Client (for NAT / Docker).
  type: boolean
  default: false
  required: false
{% endconfiguration %}

### Routing

Explicit connection via KNX/IP routing. This requires multicast communication to work in your environment.

```yaml
knx:
  routing:
```

{% configuration %}
local_ip:
  description: The local IP address of the interface that shall be used to send multicast packets. If omitted the default multicast interface is used.
  type: string
  required: false
{% endconfiguration %}

## Events

```yaml
knx:
  event_filter: 
    - "1/0/*"
    - "6/2,3,4-6/*"
```

{% configuration %}
event_filter:
  description: Defines a list of patterns for filtering KNX group addresses. Telegrams with destination addresses matching this pattern are sent to the Home Assistant event bus as `knx_event`.
  required: false
  type: [list, string]
{% endconfiguration %}

Every telegram that matches the filter with its destination field will be announced on the event bus as a `knx_event` event containing data attributes

- `data` contains the raw payload data (e.g., 1 or "[12, 55]").
- `destination` the KNX group address the telegram is sent to as string (e.g., "1/2/3).
- `direction` the direction of the telegram as string ("Incoming" / "Outgoing").
- `source` the KNX individual address of the sender as string (e.g., "1.2.3").
- `telegramtype` the APCI service of the telegram. "GroupValueWrite", "GroupValueRead" or "GroupValueResponse" generate a knx_event.

## Services

In order to directly interact with the KNX bus, you can use the following services:

### Send

```txt
Domain: knx
Service: send
Service Data: {"address": "1/0/15", "payload": 0, "type": "temperature"}
```

{% configuration %}
address:
  description: KNX group address.
  type: string
payload:
  description: Payload to send to the bus. When `type` is not set, raw bytes are sent. Integers are then treated as DPT 1/2/3 payloads. For DPTs > 6 bits send a list. Each value represents 1 octet (0-255). Pad with 0 to DPT byte length.
  type: [integer, list]
type:
  description: If set, the payload will not be sent as raw bytes, but encoded as given DPT. KNX sensor types are valid values - see table in [KNX Sensor](#sensor).
  type: [string, integer, float]
{% endconfiguration %}

### Read

You can use the `homeassistant.update_entity` service call to issue GroupValueRead requests for all `*state_address` of an entity.
To manually send GroupValueRead requests use the `knx.read` service. The response can be used from `knx_event` and will be processed in KNX entities.

```txt
Domain: knx
Service: read
Service Data: {"address": "1/0/15"}
```

{% configuration %}
address:
  description: Group address(es) to send read request to. Lists will read multiple group addresses.
  type: [string, list]
{% endconfiguration %}

```yaml
# Example automation to update a cover position after 10 seconds of movement initiation
automation:
  - trigger:
      - platform: event
        event_type: knx_event
        event_data:
          # Cover move trigger
          destination: "0/4/20"
    action:
      - delay: 0:0:10
      - service: knx.read
        data:
          # Cover position address
          address: "0/4/21"

  - trigger:
      - platform: homeassistant
        event: start
    action:
      # Register the group address to trigger a knx_event
      - service: knx.event_register
        data:
          # Cover move trigger
          address: "0/4/20"
      - service: knx.read
        data:
          # Cover position address
          address: "0/4/21"
```

### Register Event

The `knx.event_register` service can be used to register (or unregister) group addresses to fire `knx_event` Events. Events for group addresses matching the `event_filter` attribute in `configuration.yaml` cannot be unregistered. See [knx_event](#events)

{% configuration %}
address:
  description: Group address that shall be added or removed.
  required: true
  type: [string, list]
remove:
  description: If `True` the group address will be removed.
  required: false
  type: boolean
  default: false
{% endconfiguration %}

### Register Exposure

The `knx.exposure_register` service can be used to register (or unregister) exposures to the KNX bus. Exposures defined in `configuration.yaml` can not be unregistered. Per address only one exposure can be registered. See [expose](#exposing-entity-states-entity-attributes-or-time-to-knx-bus)

{% configuration %}
remove:
  description: In addition to the configuration variables of [expose](#exposing-entity-states-entity-attributes-or-time-to-knx-bus) `remove` set to `True` can be used to remove exposures. Only `address` is required for removal.
  required: false
  type: boolean
  default: false
{% endconfiguration %}

## Exposing entity states, entity attributes or time to KNX bus

KNX integration is able to expose entity states or attributes to KNX bus. The integration will broadcast any change of the exposed value to the KNX bus and answer read requests to the specified group address. It is also possible to expose the current time.

```yaml
# Example configuration.yaml entry
knx:
  expose:
    - type: temperature
      entity_id: sensor.owm_temperature
      address: "0/0/2"
    - type: string
      address: "0/6/4"
      entity_id: sensor.owm_weather
    - type: binary
      entity_id: binary_sensor.kitchen_window
      address: "0/6/5"
    - type: binary
      entity_id: light.office
      address: "0/3/0"
      default: false
    - type: percentU8
      entity_id: light.office
      attribute: brightness
      default: 0
      address: "0/3/1"
    - type: time
      address: "0/0/1"
    - type: datetime
      address: "0/0/23"
```

{% configuration %}
address:
  description: Group address state or attribute updates will be sent to. GroupValueRead requests will be answered.
  type: string
  required: true
type:
  description: Type of the exposed value. Either `binary`, `time`, `date`, `datetime` or any supported type of [KNX Sensor](#sensor) (e.g., "temperature" or "humidity").
  type: [string, integer]
  required: true
entity_id:
  description: Entity ID to be exposed. Not needed for types `time`, `date` and `datetime`.
  type: string
  required: false
attribute:
  description: Attribute of the entity that shall be sent to the KNX bus. If not set (or `None`) the state will be sent.
    For example for a light the state is either "on" or "off". With `attribute` you can expose its "brightness".
  type: string
  required: false
default:
  description: Default value to send to the bus if the state or attribute value is `None`.
    For example a light with state "off" has no brightness attribute so a default value of `0` could be used.
    If not set (or `None`) no value would be sent to the bus and a GroupReadRequest to the address would return the last known value.
  type: [boolean, string, integer, float]
  default: None
  required: false
{% endconfiguration %}

## Binary Sensor

The KNX binary sensor platform allows you to monitor [KNX](https://www.knx.org/) binary sensors.

Binary sensors are read-only. To write to the KNX bus configure an exposure [KNX Integration Expose](/integrations/knx/#exposing-entity-states-entity-attributes-or-time-to-knx-bus).

```yaml
knx:
  binary_sensor:
    - name: "Sensor 1"
      state_address: "6/0/2"
```

{% configuration %}
state_address:
  description: KNX group address of the binary sensor. *DPT 1*
  required: true
  type: [string, list]
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
sync_state:
  description: Actively read the value from the bus. If `false` no GroupValueRead telegrams will be sent to the bus. `sync_state` can be set to `init` to just initialize state on startup, `expire <minutes>` to read the state from the KNX bus when no telegram was received for \<minutes\> or `every <minutes>` to update it regularly every \<minutes\>. Maximum value for \<minutes\> is 1440. If just a number is configured "expire"-behaviour is used. Defaults to `true` which is interpreted as "expire 60".
  required: false
  type: [boolean, string, integer]
  default: true
device_class:
  description: Sets the [class of the device](/integrations/binary_sensor/), changing the device state and icon that is displayed on the frontend.
  required: false
  type: string
reset_after:
  description: Reset back to "off" state after specified seconds.
  required: false
  type: float
invert:
  description: Invert the telegrams payload before processing. This is applied before `context_timeout` or `reset_after` is evaluated.
  required: false
  type: boolean
  default: false
ignore_internal_state:
  description: Specifies if telegrams should ignore the internal state and always trigger a Home Assistant state update.
  required: false
  type: boolean
  default: false
context_timeout:
  description: The time in seconds between multiple identical telegram payloads would count towards the internal counter that is used for automations. Ex. You have automations in place that trigger your lights on button press and another set of lights if you click that button twice. This setting defines the time that a second button press would count toward, so if you set this 3.0 you can take up to 3 seconds in order to trigger the second button press. If set `ignore_internal_state` will be set to `true` internally. Maximum value is 10.0.
  required: false
  type: float
  default: None
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

### Support for automations

You can use a built in event in order to trigger an automation (e.g. to switch on a light when a switch was pressed).

Let's pretend you have a binary sensor with the name `Livingroom.Switch` and you want to switch one light on when the button was pressed once and two other lights when the button was pressed twice. `context_timeout` has to be configured in order for this to work.

```yaml
# Example automation.yaml entry
automation:
  - trigger:
      platform: numeric_state
      entity_id: binary_sensor.livingroom_switch
      attribute: counter
      above: 0
      below: 2
    condition: 
      - condition: state
        entity_id: binary_sensor.cover_abstell
        state: "on"
    action:
      - entity_id: light.hue_color_lamp_1
        service: light.turn_on
  - trigger:
      platform: numeric_state
      entity_id: binary_sensor.livingroom_switch
      attribute: counter
      above: 1
      below: 3
    condition:
      - condition: state
        entity_id: binary_sensor.cover_abstell
        state: "on"
    action:
      - entity_id: light.hue_bloom_1
        service: homeassistant.turn_on
      - entity_id: light.hue_bloom_2
        service: homeassistant.turn_on
```

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
counter:
  description: Set to 2 if you only want the action to be executed if the button was pressed twice. Set to 3 for three times button pressed.
  required: false
  type: integer
  default: 1
hook:
  description: Indicates if the automation should be executed on what state of the binary sensor. Values are "on" or "off".
  required: false
  type: string
  default: "on"
action:
  description: Specify a list of actions analog to the [automation rules](/docs/automation/action/).
  required: false
  type: list
{% endconfiguration %}

## Climate

The KNX climate platform is used as an interface to KNX thermostats and room controllers.

To use your KNX thermostats in your installation, add the following lines to your top level [KNX Integration](/integrations/knx) configuration key in `configuration.yaml`:

```yaml
# Example configuration.yaml entry
knx:
  climate:
    - name: "Kitchen"
      temperature_address: "5/1/1"
      setpoint_shift_address: "5/1/2"
      setpoint_shift_state_address: "5/1/3"
      target_temperature_state_address: "5/1/4"
      operation_mode_address: "5/1/5"
      operation_mode_state_address: "5/1/6"
```

Alternatively, if your device has dedicated binary group addresses for frost/night/comfort mode:

```yaml
# Example configuration.yaml entry
knx:
  climate:
    - name: "Kitchen"
      temperature_address: "5/1/1"
      setpoint_shift_address: "5/1/2"
      setpoint_shift_state_address: "5/1/3"
      target_temperature_state_address: "5/1/4"
      operation_mode_frost_protection_address: "5/1/5"
      operation_mode_night_address: "5/1/6"
      operation_mode_comfort_address: "5/1/7"
      operation_mode_state_address: "5/1/8"
```

If your device doesn't support setpoint_shift calculations (i.e., if you don't provide a `setpoint_shift_address` value) please set the `min_temp` and `max_temp`
attributes of the climate device to avoid issues with exceeding valid temperature values in the frontend. Please do also make sure to add the `target_temperature_address` to the configuration in this case.:

```yaml
# Example configuration.yaml entry
knx:
  climate:
    - name: "Kitchen"
      temperature_address: "5/1/2"
      target_temperature_address: "5/1/4"
      target_temperature_state_address: "5/1/1"
      operation_mode_frost_protection_address: "5/1/5"
      operation_mode_night_address: "5/1/6"
      operation_mode_comfort_address: "5/1/7"
      operation_mode_state_address: "5/1/8"
      operation_mode_standby_address: "5/1/9"
      min_temp: 7.0
      max_temp: 32.0
```

`setpoint_shift_mode` allows the two following DPTs to be used:

- DPT6002 (for 1 byte signed integer)
- DPT9002 (for 2 byte float)

Example:

```yaml
# Example configuration.yaml entry
knx:
  climate:
    - name: "Kitchen"
      temperature_address: "5/1/1"
      setpoint_shift_address: "5/1/2"
      setpoint_shift_state_address: "5/1/3"
      setpoint_shift_mode: "DPT9002"
      target_temperature_state_address: "5/1/4"
      operation_mode_address: "5/1/5"
      operation_mode_state_address: "5/1/6"
```

`operation_mode_frost_protection_address` / `operation_mode_night_address` / `operation_mode_comfort_address` / `operation_mode_standby_address` are not necessary if `operation_mode_address` is specified.
If the actor doesn't support explicit state communication objects the `*_state_address` can be configured with the same group address as the writeable `*_address`. The read flag for the `*_state_address` communication object has to be set in ETS to support initial reading e.g., when starting Home Assistant.

The following values are valid for the `heat_cool_address` and the `heat_cool_state_address`:

- `0` (cooling)
- `1` (heating)

The following values are valid for the Home Assistant [Climate](/integrations/climate/) `hvac_mode` attribute. Supported values for your KNX thermostats can be specified via `controller_modes` configuration variable:

- `Off` (maps internally to `HVAC_MODE_OFF` within Home Assistant)
- `Auto` (maps internally to `HVAC_MODE_AUTO` within Home Assistant)
- `Heat` (maps internally to `HVAC_MODE_HEAT` within Home Assistant)
- `Cool` (maps internally to `HVAC_MODE_COOL` within Home Assistant)
- `Fan only` (maps internally to `HVAC_MODE_FAN_ONLY` within Home Assistant)
- `Dry` (maps internally to `HVAC_MODE_DRY` within Home Assistant)

The following presets are valid for the Home Assistant [Climate](/integrations/climate/) `preset_mode` attribute. Supported values for your KNX thermostats can be specified via `operation_modes` configuration variable:

- `Auto` (maps internally to `PRESET_NONE` within Home Assistant)
- `Comfort` (maps internally to `PRESET_COMFORT` within Home Assistant)
- `Standby` (maps internally to `PRESET_AWAY` within Home Assistant)
- `Night` (maps internally to `PRESET_SLEEP` within Home Assistant)
- `Frost` Protection (maps internally to `PRESET_ECO` within Home Assistant)

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  default: KNX Climate
  type: string
temperature_address:
  description: KNX group address for reading current room temperature from KNX bus. *DPT 9.001*
  required: true
  type: [string, list]
temperature_step:
  description: Defines the step size in Kelvin for each step of setpoint_shift.
  required: false
  type: float
  default: 0.1
target_temperature_address:
  description: KNX group address for setting target temperature. *DPT 9.001*
  required: false
  type: [string, list]
target_temperature_state_address:
  description: KNX group address for reading current target temperature from KNX bus. *DPT 9.001*
  required: true
  type: [string, list]
setpoint_shift_address:
  description: KNX address for setpoint_shift. *DPT 6.010 or DPT 9.002 based on setpoint_shift_mode*
  required: false
  type: [string, list]
setpoint_shift_state_address:
  description: KNX address for reading setpoint_shift. *DPT 6.010 or DPT 9.002 based on setpoint_shift_mode*
  required: false
  type: [string, list]
setpoint_shift_mode:
  description: Defines the internal device DPT used. Either 'DPT6010', 'DPT9002' or None. When `None` or omitted the DPT is auto-assigned from the first incoming telegram.
  required: false
  type: string
  default: None
setpoint_shift_min:
  description: Minimum value of setpoint shift.
  required: false
  default: -6
  type: float
setpoint_shift_max:
  description: Maximum value of setpoint shift.
  required: false
  default: 6
  type: float
active_state_address:
  description: KNX address for reading current activity. `0` is idle, `1` is active. *DPT 1*
  required: false
  type: [string, list]
command_value_state_address:
  description: KNX address for reading current command value in percent. `0` sets the climate entity to idle if `active_state_address` is not set. *DPT 5.001*
  required: false
  type: [string, list]
operation_mode_address:
  description: KNX address for setting operation mode (Frost protection/night/comfort). *DPT 20.102*
  required: false
  type: [string, list]
operation_mode_state_address:
  description: KNX address for reading operation mode. *DPT 20.102*
  required: false
  type: [string, list]
controller_status_address:
  description: KNX address for HVAC controller status (in accordance with KNX AN 097/07 rev 3).
  required: false
  type: [string, list]
controller_status_state_address:
  description: KNX address for reading HVAC controller status.
  required: false
  type: [string, list]
controller_mode_address:
  description: KNX address for setting HVAC controller modes. *DPT 20.105*
  required: false
  type: [string, list]
controller_mode_state_address:
  description: KNX address for reading HVAC control mode. *DPT 20.105*
  required: false
  type: [string, list]
heat_cool_address:
  description: KNX address for switching between heat/cool mode. *DPT 1.100*
  required: false
  type: [string, list]
heat_cool_state_address:
  description: KNX address for reading heat/cool mode. *DPT 1.100*
  required: false
  type: [string, list]
operation_mode_frost_protection_address:
  description: KNX address for switching on/off frost/heat protection mode. *DPT 1*
  required: false
  type: [string, list]
operation_mode_night_address:
  description: KNX address for switching on/off night mode. *DPT 1*
  required: false
  type: [string, list]
operation_mode_comfort_address:
  description: KNX address for switching on/off comfort mode. *DPT 1*
  required: false
  type: [string, list]
operation_mode_standby_address:
  description: KNX address for switching on/off standby mode. *DPT 1*
  required: false
  type: [string, list]
operation_modes:
  description: Overrides the supported operation modes. Provide the supported `preset_mode` values for your device.
  required: false
  type: list
controller_modes:
  description: Overrides the supported controller modes. Provide the supported `hvac_mode` values for your device.
  required: false
  type: list
default_controller_mode:
  description: Overrides the default controller mode. Any Home Assistant `hvac_mode` can be configured. This can, for example, be set to "cool" for cooling-only devices.
  required: false
  default: "heat"
  type: string
on_off_address:
  description: KNX address for switching the climate device on/off. *DPT 1*
  required: false
  type: [string, list]
on_off_invert:
  description: Value for switching the climate device on/off is inverted.
  required: false
  default: false
  type: boolean
on_off_state_address:
  description: KNX address for gathering the current state (on/off) of the climate device. *DPT 1*
  required: false
  type: [string, list]
min_temp:
  description: Override the minimum temperature.
  required: false
  type: float
max_temp:
  description: Override the maximum temperature.
  required: false
  type: float
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Cover

The KNX cover platform is used as an interface to KNX covers.

<div class='note'>

Unlike most KNX devices, Home Assistant defines 0% as closed and 100% as fully open in regards to covers. The corresponding value inversion is done internally by the KNX integration.

</div>

To use your KNX covers in your installation, add the following lines to your top level [KNX Integration](/integrations/knx) configuration key in `configuration.yaml`:

```yaml
# Example configuration.yaml entry
knx:
  cover:
    - name: "Kitchen shutter"
      move_long_address: "3/0/0"
      move_short_address: "3/0/1"
      stop_address: "3/0/4"
      position_address: "3/0/3"
      position_state_address: "3/0/2"
      travelling_time_down: 51
      travelling_time_up: 61
```

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  default: KNX Cover
  type: string
move_long_address:
  description: KNX group address for moving the cover full up or down. *DPT 1*
  required: false
  type: [string, list]
move_short_address:
  description: KNX group address for moving the cover stepwise up or down. Used by some covers also as the means to stop the cover, if no dedicated `stop_address` exists on the actuator. *DPT 1*
  required: false
  type: [string, list]
stop_address:
  description: KNX group address for stopping the current movement of the cover. *DPT 1*
  required: false
  type: [string, list]
position_address:
  description: KNX group address for moving the cover to the dedicated position. *DPT 5.001*
  required: false
  type: [string, list]
position_state_address:
  description: Separate KNX group address for requesting the current position of the cover. *DPT 5.001*
  required: false
  type: [string, list]
angle_address:
  description: KNX group address for tilting the cover to the dedicated angle. *DPT 5.001*
  required: false
  type: [string, list]
angle_state_address:
  description: Separate KNX group address for requesting the current tilt angle of the cover. *DPT 5.001*
  required: false
  type: [string, list]
travelling_time_down:
  description: Time cover needs to travel down in seconds. Needed to calculate the intermediate positions of cover while traveling.
  required: false
  default: 25
  type: integer
travelling_time_up:
  description: Time cover needs to travel up in seconds. Needed to calculate the intermediate positions of cover while traveling.
  required: false
  default: 25
  type: integer
invert_position:
  description: Set this to `true` if your actuator reports fully closed position as 0% in KNX.
  required: false
  default: false
  type: boolean
invert_angle:
  description: Set this to `true` if your actuator reports fully closed tilt as 0% in KNX.
  required: false
  default: false
  type: boolean
device_class:
  description: Sets the [class of the device](/integrations/cover/), changing the device state and icon that is displayed on the frontend.
  required: false
  type: string
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Fan

The KNX fan integration is used to control KNX fans. Following control types are supported:

- Percentage controlled: Fans that set the percentage directly from 0-100%.
- Step controlled: Fans which have a fixed amount of steps to set. The integration will convert percentage to step automatically. The `max_step` attribute is set to the number of steps of the fan, not counting the `off`-step. Example: A fan supports the steps 0 to 3. To use this fan the `max_step` attribute has to be set to `3`. The integration will convert the percentage `66 %` to the step `2` when sending data to KNX.

To use your KNX fan in your installation, add the following lines to your top level [KNX Integration](/integrations/knx) configuration key in `configuration.yaml`:

```yaml
# Example configuration.yaml entry
knx:
  fan:
    - name: "Ceiling fan"
      address: "9/0/1"
      state_address: "9/0/2"
```

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
address:
  description: KNX group address for setting the percentage or step of the fan. *DPT 5.001* or *DPT 5.010*
  required: true
  type: [string, list]
state_address:
  description: KNX group address for retrieving the percentage or step of the fan. *DPT 5.001* or *DPT 5.010*
  required: false
  type: [string, list]
oscillation_address:
  description: KNX group address for switching the fan oscillation on or off. *DPT 1*
  required: false
  type: [string, list]
oscillation_state_address:
  description: KNX group address for retrieving the state of the fan oscillation. *DPT 1*
  required: false
  type: [string, list]
max_step:
  description: The maximum amount of steps for a step-controlled fan. If set, the integration will convert percentages to steps automatically.
  required: false
  type: integer
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Light

The KNX light integration is used as an interface to control KNX actuators for lighting applications such as:

- Switching actuators
- Dimming actuators
- LED controllers
- DALI gateways

To use your KNX light in your installation, add the following lines to your top level [KNX Integration](/integrations/knx) configuration key in `configuration.yaml`:

```yaml
# Example configuration.yaml entry
knx:
  light:
    - name: "Kitchen"
      address: "1/0/9"
```

{% configuration %}
address:
  description: KNX group address for switching the light on and off. *DPT 1.001*
  required: true
  type: [string, list]
state_address:
  description: KNX group address for retrieving the switch state of the light. *DPT 1.001*
  required: false
  type: [string, list]
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
brightness_address:
  description: KNX group address for setting the brightness of the light in percent (absolute dimming). *DPT 5.001*
  required: false
  type: [string, list]
brightness_state_address:
  description: KNX group address for retrieving the brightness of the light in percent. *DPT 5.001*
  required: false
  type: [string, list]
color_address:
  description: KNX group address for setting the RGB color of the light. *DPT 232.600*
  required: false
  type: [string, list]
color_state_address:
  description: KNX group address for retrieving the RGB color of the light. *DPT 232.600*
  required: false
  type: [string, list]
rgbw_address:
  description: KNX group address for setting the RGBW color of the light. *DPT 251.600*
  required: false
  type: [string, list]
rgbw_state_address:
  description: KNX group address for retrieving the RGBW color of the light. *DPT 251.600*
  required: false
  type: [string, list]
hue_address:
  description: KNX group address for setting the hue of the light color in degrees. *DPT 5.003*
  required: false
  type: [string, list]
hue_state_address:
  description: KNX group address for retrieving the hue of the light color in degrees. *DPT 5.003*
  required: false
  type: [string, list]
saturation_address:
  description: KNX group address for setting the saturation of the light color in percent. *DPT 5.001*
  required: false
  type: [string, list]
saturation_state_address:
  description: KNX group address for retrieving the saturation of the light color in percent. *DPT 5.001*
  required: false
  type: [string, list]
xyy_address:
  description: KNX group address for setting the xyY color of the light. *DPT 242.600*
  required: false
  type: [string, list]
xyy_state_address:
  description: KNX group address for retrieving the xyY color of the light. *DPT 242.600*
  required: false
  type: [string, list]
individual_colors:
  description: Used when the actuator only supports individual group addresses for colors. When `individual_colors` is used the root `address` key may be omitted.
  required: false
  type: map
  keys:
    red:
      description: Group addresses for the red component.
      type: map
      required: true
      keys:
        address:
          description: KNX group address to switch the red component. *DPT 1.001*
          type: [string, list]
          required: false
        state_address:
          description: KNX group address for the state of the red component. *DPT 1.001*
          type: [string, list]
          required: false
        brightness_address:
          description: KNX group address to set the brightness of the red component. *DPT 5.001*
          type: [string, list]
          required: true
        brightness_state_address:
          description: KNX group address for the current brightness of the red component. *DPT 5.001*
          type: [string, list]
          required: false
    green:
      description: Group addresses for the green component. Same keys available as for red component above.
      type: map
      required: true
    blue:
      description: Group addresses for the blue component. Same keys available as for red component above.
      type: map
      required: true
    white:
      description: Group addresses for the white component. Same keys available as for red component above.
      type: map
      required: false
color_temperature_address:
  description: KNX group address for setting the color temperature of the light. *DPT 5.001 or 7.600 based on color_temperature_mode*
  required: false
  type: [string, list]
color_temperature_state_address:
  description: KNX group address for retrieving the color temperature of the light. *DPT 5.001 or 7.600 based on color_temperature_mode*
  required: false
  type: [string, list]
color_temperature_mode:
  description: Color temperature group address data type. `absolute` color temperature in Kelvin. *color_temperature_address -> DPT 7.600*. `relative` color temperature in percent cold white (0% warmest; 100% coldest). *color_temperature_address -> DPT 5.001*
  required: false
  type: string
  default: absolute
min_kelvin:
  description: Warmest possible color temperature in Kelvin. Used in combination with `color_temperature_address`.
  required: false
  type: integer
  default: 2700
max_kelvin:
  description: Coldest possible color temperature in Kelvin. Used in combination with `color_temperature_address`.
  required: false
  type: integer
  default: 6000
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

Many KNX devices can change their state internally without a message to the switch address on the KNX bus, e.g., if you configure a scene or a timer on a channel. The optional `state_address` can be used to inform Home Assistant about these state changes. If a KNX message is seen on the bus addressed to the given `state_address` (in most cases from the light actuator), it will overwrite the state of the object.

For switching/light actuators that are only controlled by a single group address and don't have dedicated state communication objects you can set `state_address` to the same value as `address`.

*Note on tunable white:* Home Assistant uses Mireds as the unit for color temperature, whereas KNX typically uses Kelvin. The Kelvin/Mireds relationship is reciprocal, not linear, therefore the color temperature pickers (sliders) in Home Assistant may not align with ones of KNX visualizations. This is the expected behavior.

### Extended configuration examples

```yaml
knx:
  light:
    # dimmable light
    # color mode: brightness
    - name: "Dimmable light"
      address: "1/0/9"
      state_address: "1/1/9"
      brightness_address: "1/2/9"
      brightness_state_address: "1/3/9"
    # XYY light
    # color mode: xy
    - name: "XYY light"
      address: "1/0/9"
      state_address: "1/1/9"
      brightness_address: "1/2/9"  # optional - if not set brightness will be sent over the xyy data point
      brightness_state_address: "1/3/9"
      xyy_address: "1/4/9"
      xyy_state_address: "1/5/9"
    # HS light
    # color mode: hs
    - name: "HS light"
      address: "1/0/9"
      state_address: "1/1/9"
      brightness_address: "1/2/9"  # required for HS
      brightness_state_address: "1/3/9"
      hue_address: "1/4/8"
      hue_state_address: "1/5/8"  # required for HS
      saturation_address: "1/4/9"
      saturation_state_address: "1/5/9"  # required for HS
    # RGB light
    # color mode: rgb
    - name: "RGB light"
      address: "1/0/9"
      state_address: "1/1/9"
      brightness_address: "1/2/9"  # optional for RGB lights
      brightness_state_address: "1/3/9"
      color_address: "1/4/9"
      color_state_address: "1/5/9"
    # RGBW light
    # color mode: rgbw
    - name: "RGBW light"
      address: "0/4/83"
      state_address: "0/4/84"
      brightness_address: "0/4/85"  # optional for RGBW lights
      brightness_state_address: "0/4/86"
      rgbw_address: "0/4/87"
      rgbw_state_address: "0/4/88"
    # RGB(W) individual object light
    # color mode: rgb / rgbw
    - name: "RGBW individual light"
      address: "1/0/9"  # optional for individual color lights
      individual_colors:
        red:
          brightness_address: "0/4/61"
          brightness_state_address: "0/5/61"
        green:
          brightness_address: "0/4/62"
          brightness_state_address: "0/5/62"
        blue:
          brightness_address: "0/4/63"
          brightness_state_address: "0/5/63"
        white:
          brightness_address: "0/4/64"
          brightness_state_address: "0/5/64"
    # tunable white light
    # color mode: color_temp
    - name: "TW light"
      address: "1/0/21"
      state_address: "1/1/21"
      brightness_address: "1/2/21"
      brightness_state_address: "1/3/21"
      color_temperature_address: "1/4/21"
      color_temperature_state_address: "1/5/21"
      color_temperature_mode: absolute
      min_kelvin: 2550
      max_kelvin: 6200
    # actuator without dedicated state communication object
    # color mode: onoff
    - name: "Simple light"
      address: "1/0/5"
      state_address: "1/0/5"
```

## Notify

The KNX notify platform allows you to send notifications to [KNX](https://www.knx.org/) devices as DPT16 strings.

```yaml
knx:
  notify:
    - name: "Alarm"
      address: "5/1/10"
```

{% configuration %}
address:
  description: KNX group address of the notification. *DPT 16.000*
  required: true
  type: [string, list]
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
{% endconfiguration %}

## Number

The KNX number platform allows to send generic numeric values to the KNX bus and update its state from received telegrams. It can optionally respond to read requests from the KNX bus with its current state.

<div class='note'>

Number entities without a `state_address` will restore their last known state after Home Assistant was restarted.

Numbers having a `state_address` configured request their current state from the KNX bus.

</div>

```yaml
# Example configuration.yaml entry
knx:
  number:
    - name: "Duration"
      address: "0/0/1"
      type: time_period_sec
    - name: "Volume"
      address: "0/0/2"
      state_address: "0/0/3"
      type: percent
    - name: "Temperature threshold"
      address: "0/0/4"
      respond_to_read: true
      type: temperature
      min: 20
      max: 24.5
      step: 0.1
      mode: slider
```

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
address:
  description: Group address new values will be sent to.
  required: true
  type: [string, list]
state_address:
  description: Group address for retrieving the state from the KNX bus.
  required: false
  type: [string, list]
type:
  description: Any supported type of [KNX Sensor](#sensor) representing a numeric value (e.g., "percent" or "temperature").
  required: true
  type: [string, integer]
respond_to_read:
  description: Respond to GroupValueRead telegrams received to the configured `address`.
  required: false
  type: boolean
  default: false
min:
  description: Minimum value that can be sent. Defaults to the `type` DPT minimum value.
  required: false
  type: float
max:
  description: Maximum value that can be sent. Defaults to the `type` DPT maximum value.
  required: false
  type: float
step:
  description: Step value. Defaults to the step size defined for the DPT in the KNX specifications.
  required: false
  type: float
mode:
  description: Specifies the mode used in the UI. `auto`, `box` or `slider` are valid.
  required: false
  type: string
  default: auto
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Scene

The KNX scenes platform allows you to trigger [KNX](https://www.knx.org/) scenes. These entities are write-only.

```yaml
# Example configuration.yaml entry
knx:
  scene:
    - name: "Romantic"
      address: 8/8/8
      scene_number: 23
```

{% configuration %}
address:
  description: KNX group address for the scene. *DPT 17.001*
  required: true
  type: [string, list]
scene_number:
  description: KNX scene number to be activated (range 1..64 ).
  required: true
  type: integer
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Select

The KNX select platform allows the user to define a list of values that can be selected via the frontend and can be used within conditions of automation. When a user selects a new item, the assigned generic raw payload is sent to the KNX bus. A received telegram updates the state of the select entity. It can optionally respond to read requests from the KNX bus with its current state.

<div class='note'>

Select entities without a `state_address` will restore their last known state after Home Assistant was restarted.

Selects having a `state_address` configured request their current state from the KNX bus.

</div>

```yaml
# Example configuration.yaml entry
knx:
  select:
    - name: "DPT 2 selector"
      address: "0/0/1"
      payload_length: 0
      options:
        - option: "No control"
          payload: 0
        - option: "Control On"
          payload: 0b10
        - option: "Control Off"
          payload: 0b11
    - name: "DHWMode"
      address: "0/0/2"
      state_address: "0/0/3"
      payload_length: 1
      options:
        - option: "Auto"
          payload: 0
        - option: "LegioProtect"
          payload: 1
        - option: "Normal"
          payload: 2
        - option: "Reduced"
          payload: 3
        - option: "Off/FrostProtect"
          payload: 4
```

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
address:
  description: Group address new values will be sent to.
  required: true
  type: [string, list]
state_address:
  description: Group address for retrieving the state from the KNX bus.
  required: false
  type: [string, list]
payload_length:
  description: The length of the payload expected for the DPT. Use `0` for DPT 1, 2 or 3.
  required: true
  type: integer
options:
  description: List of options to choose from. Each `option` and `payload` have to be unique.
  type: list
  required: true
  keys:
    option:
      description: The name of the option used to trigger the assigned `payload`.
      required: true
      type: string
    payload:
      description: The raw payload assigned to the `option`.
      required: true
      type: integer
respond_to_read:
  description: Respond to GroupValueRead telegrams received to the configured `address`.
  required: false
  type: boolean
  default: false
sync_state:
  description: Actively read the value from the bus. If `false` no GroupValueRead telegrams will be sent to the bus. `sync_state` can be set to `init` to just initialize state on startup, `expire <minutes>` to read the state from the KNX bus when no telegram was received for \<minutes\> or `every <minutes>` to update it regularly every \<minutes\>. Maximum value for \<minutes\> is 1440. If just a number is configured "expire"-behaviour is used. Defaults to `true` which is interpreted as "expire 60".
  required: false
  type: [boolean, string, integer]
  default: true
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Sensor

The KNX sensor platform allows you to monitor [KNX](https://www.knx.org/) sensors.

Sensors are read-only. To write to the KNX bus configure a [Number](#number), an exposure [KNX Integration Expose](/integrations/knx/#exposing-entity-states-entity-attributes-or-time-to-knx-bus) or use the `knx.send` service.

```yaml
# Example configuration.yaml entry
knx:
  sensor:
    - name: "Heating Valve 1"
      state_address: "2/0/0"
      type: percent
```

In order to actively read the sensor data from the bus every 30 minutes you can add the following lines to your `configuration.yaml`:

```yaml
# Example configuration.yaml entry
knx:
  sensor:
    - name: "Heating Valve 1"
      state_address: "2/0/0"
      type: percent
      sync_state: every 30
```

{% configuration %}
state_address:
  description: KNX group address of the sensor.
  required: true
  type: [string, list]
type:
  description: A type from the [value types table](/integrations/knx/#value-types) below must be defined. The DPT of the group address should match the expected KNX DPT to be parsed correctly.
  required: true
  type: [string, integer]
name:
  description: A name for this device used within Home Assistant.
  required: false
  type: string
sync_state:
  description: Actively read the value from the bus. If `false` no GroupValueRead telegrams will be sent to the bus. `sync_state` can be set to `init` to just initialize state on startup, `expire <minutes>` to read the state from the KNX bus when no telegram was received for \<minutes\> or `every <minutes>` to update it regularly every \<minutes\>. Maximum value for \<minutes\> is 1440. If just a number is configured "expire"-behaviour is used. Defaults to `true` which is interpreted as "expire 60".
  required: false
  type: [boolean, string, integer]
  default: true
always_callback:
  description: Defines if telegrams with equal payload as the previously received telegram should trigger a state update within Home Assistant.
  required: false
  type: boolean
  default: false
state_class:
  description: Sets the [state_class](https://developers.home-assistant.io/docs/core/entity/sensor#available-state-classes) of the sensor.
  required: false
  type: string
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

### Value Types

| KNX DPT | type                          | size in byte |           range            | unit           |
| ------: | ----------------------------- | -----------: | :------------------------: | -------------- |
|       5 | 1byte_unsigned                |            1 |         0 ... 255          |                |
|   5.001 | percent                       |            1 |         0 ... 100          | %              |
|   5.003 | angle                         |            1 |         0 ... 360          | °              |
|   5.004 | percentU8                     |            1 |         0 ... 255          | %              |
|   5.005 | decimal_factor                |            1 |         0 ... 255          |                |
|   5.006 | tariff                        |            1 |         0 ... 254          |                |
|   5.010 | pulse                         |            1 |         0 ... 255          | counter pulses |
|       6 | 1byte_signed                  |            1 |        -128 ... 127        |                |
|   6.001 | percentV8                     |            1 |        -128 ... 127        | %              |
|   6.010 | counter_pulses                |            1 |        -128 ... 127        | counter pulses |
|       7 | 2byte_unsigned                |            2 |        0 ... 65535         |                |
|   7.001 | pulse_2byte                   |            2 |        0 ... 65535         | pulses         |
|   7.002 | time_period_msec              |            2 |        0 ... 65535         | ms             |
|   7.003 | time_period_10msec            |            2 |        0 ... 65535         | ms             |
|   7.004 | time_period_100msec           |            2 |        0 ... 65535         | ms             |
|   7.005 | time_period_sec               |            2 |        0 ... 65535         | s              |
|   7.006 | time_period_min               |            2 |        0 ... 65535         | min            |
|   7.007 | time_period_hrs               |            2 |        0 ... 65535         | h              |
|   7.011 | length_mm                     |            2 |        0 ... 65535         | mm             |
|   7.012 | current                       |            2 |        0 ... 65535         | mA             |
|   7.013 | brightness                    |            2 |        0 ... 65535         | lx             |
|   7.600 | color_temperature             |            2 |        0 ... 65535         | K              |
|       8 | 2byte_signed                  |            2 |      -32768 ... 32767      |                |
|   8.001 | pulse_2byte_signed            |            2 |      -32768 ... 32767      | pulses         |
|   8.002 | delta_time_ms                 |            2 |      -32768 ... 32767      | ms             |
|   8.003 | delta_time_10ms               |            2 |      -32768 ... 32767      | ms             |
|   8.004 | delta_time_100ms              |            2 |      -32768 ... 32767      | ms             |
|   8.005 | delta_time_sec                |            2 |      -32768 ... 32767      | s              |
|   8.006 | delta_time_min                |            2 |      -32768 ... 32767      | min            |
|   8.007 | delta_time_hrs                |            2 |      -32768 ... 32767      | h              |
|   8.010 | percentV16                    |            2 |      -32768 ... 32767      | %              |
|   8.011 | rotation_angle                |            2 |      -32768 ... 32767      | °              |
|       9 | 2byte_float                   |            2 |  -671088.64 ... 670760.96  |                |
|   9.001 | temperature                   |            2 |      -273 ... 670760       | °C             |
|   9.002 | temperature_difference_2byte  |            2 |     -670760 ... 670760     | K              |
|   9.003 | temperature_a                 |            2 |     -670760 ... 670760     | K/h            |
|   9.004 | illuminance                   |            2 |        0 ... 670760        | lx             |
|   9.005 | wind_speed_ms                 |            2 |        0 ... 670760        | m/s            |
|   9.006 | pressure_2byte                |            2 |        0 ... 670760        | Pa             |
|   9.007 | humidity                      |            2 |        0 ... 670760        | %              |
|   9.008 | ppm                           |            2 |  -671088.64 ... 670760.96  | ppm            |
|   9.010 | time_1                        |            2 |     -670760 ... 670760     | s              |
|   9.011 | time_2                        |            2 |     -670760 ... 670760     | ms             |
|   9.020 | voltage                       |            2 |  -671088.64 ... 670760.96  | mV             |
|   9.021 | curr                          |            2 |  -671088.64 ... 670760.96  | mA             |
|   9.022 | power_density                 |            2 |  -671088.64 ... 670760.96  | W/m²           |
|   9.023 | kelvin_per_percent            |            2 |  -671088.64 ... 670760.96  | K/%            |
|   9.024 | power_2byte                   |            2 |  -671088.64 ... 670760.96  | kW             |
|   9.025 | volume_flow                   |            2 |  -671088.64 ... 670760.96  | l/h            |
|   9.026 | rain_amount                   |            2 |  -671088.64 ... 670760.96  | l/m²           |
|   9.027 | temperature_f                 |            2 |     -459.6 ... 670760      | °F             |
|   9.028 | wind_speed_kmh                |            2 |        0 ... 670760        | km/h           |
|     9.? | enthalpy                      |            2 |  -671088.64 ... 670760.96  | H              |
|      12 | 4byte_unsigned                |            4 |      0 ... 4294967295      |                |
| 12.1200 | volume_liquid_litre           |            4 |      0 ... 4294967295      | l              |
| 12.1201 | volume_m3                     |            4 |      0 ... 4294967295      | m³             |
|      13 | 4byte_signed                  |            4 | -2147483648 ... 2147483647 |                |
|  13.001 | pulse_4byte                   |            4 | -2147483648 ... 2147483647 | pulses         |
|  13.002 | flow_rate_m3h                 |            4 | -2147483648 ... 2147483647 | m³/h           |
|  13.010 | active_energy                 |            4 | -2147483648 ... 2147483647 | Wh             |
|  13.011 | apparant_energy               |            4 | -2147483648 ... 2147483647 | VAh            |
|  13.012 | reactive_energy               |            4 | -2147483648 ... 2147483647 | VARh           |
|  13.013 | active_energy_kwh             |            4 | -2147483648 ... 2147483647 | kWh            |
|  13.014 | apparant_energy_kvah          |            4 | -2147483648 ... 2147483647 | kVAh           |
|  13.015 | reactive_energy_kvarh         |            4 | -2147483648 ... 2147483647 | kVARh          |
|  13.100 | long_delta_timesec            |            4 | -2147483648 ... 2147483647 | s              |
|      14 | 4byte_float                   |            4 |                            |                |
|  14.000 | acceleration                  |            4 |                            | m/s²           |
|  14.001 | acceleration_angular          |            4 |                            | rad/s²         |
|  14.002 | activation_energy             |            4 |                            | J/mol          |
|  14.003 | activity                      |            4 |                            | s⁻¹            |
|  14.004 | mol                           |            4 |                            | mol            |
|  14.005 | amplitude                     |            4 |                            |                |
|  14.006 | angle_rad                     |            4 |                            | rad            |
|  14.007 | angle_deg                     |            4 |                            | °              |
|  14.008 | angular_momentum              |            4 |                            | J s            |
|  14.009 | angular_velocity              |            4 |                            | rad/s          |
|  14.010 | area                          |            4 |                            | m²             |
|  14.011 | capacitance                   |            4 |                            | F              |
|  14.012 | charge_density_surface        |            4 |                            | C/m²           |
|  14.013 | charge_density_volume         |            4 |                            | C/m³           |
|  14.014 | compressibility               |            4 |                            | m²/N           |
|  14.015 | conductance                   |            4 |                            | S              |
|  14.016 | electrical_conductivity       |            4 |                            | S/m            |
|  14.017 | density                       |            4 |                            | kg/m³          |
|  14.018 | electric_charge               |            4 |                            | C              |
|  14.019 | electric_current              |            4 |                            | A              |
|  14.020 | electric_current_density      |            4 |                            | A/m²           |
|  14.021 | electric_dipole_moment        |            4 |                            | C m            |
|  14.022 | electric_displacement         |            4 |                            | C/m²           |
|  14.023 | electric_field_strength       |            4 |                            | V/m            |
|  14.024 | electric_flux                 |            4 |                            | c              |
|  14.025 | electric_flux_density         |            4 |                            | C/m²           |
|  14.026 | electric_polarization         |            4 |                            | C/m²           |
|  14.027 | electric_potential            |            4 |                            | V              |
|  14.028 | electric_potential_difference |            4 |                            | V              |
|  14.029 | electromagnetic_moment        |            4 |                            | A m²           |
|  14.030 | electromotive_force           |            4 |                            | V              |
|  14.031 | energy                        |            4 |                            | J              |
|  14.032 | force                         |            4 |                            | N              |
|  14.033 | frequency                     |            4 |                            | Hz             |
|  14.034 | angular_frequency             |            4 |                            | rad/s          |
|  14.035 | heatcapacity                  |            4 |                            | J/K            |
|  14.036 | heatflowrate                  |            4 |                            | W              |
|  14.037 | heat_quantity                 |            4 |                            | J              |
|  14.038 | impedance                     |            4 |                            | Ω              |
|  14.039 | length                        |            4 |                            | m              |
|  14.040 | light_quantity                |            4 |                            | lm s           |
|  14.041 | luminance                     |            4 |                            | cd/m²          |
|  14.042 | luminous_flux                 |            4 |                            | lm             |
|  14.043 | luminous_intensity            |            4 |                            | cd             |
|  14.044 | magnetic_field_strength       |            4 |                            | A/m            |
|  14.045 | magnetic_flux                 |            4 |                            | Wb             |
|  14.046 | magnetic_flux_density         |            4 |                            | T              |
|  14.047 | magnetic_moment               |            4 |                            | A m²           |
|  14.048 | magnetic_polarization         |            4 |                            | T              |
|  14.049 | magnetization                 |            4 |                            | A/m            |
|  14.050 | magnetomotive_force           |            4 |                            | A              |
|  14.051 | mass                          |            4 |                            | kg             |
|  14.052 | mass_flux                     |            4 |                            | kg/s           |
|  14.053 | momentum                      |            4 |                            | N/s            |
|  14.054 | phaseanglerad                 |            4 |                            | rad            |
|  14.055 | phaseangledeg                 |            4 |                            | °              |
|  14.056 | power                         |            4 |                            | W              |
|  14.057 | powerfactor                   |            4 |                            | cosΦ           |
|  14.058 | pressure                      |            4 |                            | Pa             |
|  14.059 | reactance                     |            4 |                            | Ω              |
|  14.060 | resistance                    |            4 |                            | Ω              |
|  14.061 | resistivity                   |            4 |                            | Ω m            |
|  14.062 | self_inductance               |            4 |                            | H              |
|  14.063 | solid_angle                   |            4 |                            | sr             |
|  14.064 | sound_intensity               |            4 |                            | W/m²           |
|  14.065 | speed                         |            4 |                            | m/s            |
|  14.066 | stress                        |            4 |                            | Pa             |
|  14.067 | surface_tension               |            4 |                            | N/m            |
|  14.068 | common_temperature            |            4 |                            | °C             |
|  14.069 | absolute_temperature          |            4 |                            | K              |
|  14.070 | temperature_difference        |            4 |                            | K              |
|  14.071 | thermal_capacity              |            4 |                            | J/K            |
|  14.072 | thermal_conductivity          |            4 |                            | W/mK           |
|  14.073 | thermoelectric_power          |            4 |                            | V/K            |
|  14.074 | time_seconds                  |            4 |                            | s              |
|  14.075 | torque                        |            4 |                            | N m            |
|  14.076 | volume                        |            4 |                            | m³             |
|  14.077 | volume_flux                   |            4 |                            | m³/s           |
|  14.078 | weight                        |            4 |                            | N              |
|  14.079 | work                          |            4 |                            | J              |
|  16.000 | string                        |           14 |                            |                |
|  17.001 | scene_number                  |            1 |          1 ... 64          |                |

### More examples

```yaml
# Example configuration.yaml entry
knx:
  sensor:
    - name: "Heating Valve 1"
      state_address: "2/0/0"
      sync_state: init
      type: percent
    - name: "Kitchen Temperature"
      state_address: "6/2/1"
      sync_state: every 60
      type: temperature
      state_class: measurement
```

## Switch

The KNX switch platform is used as an interface to switching actuators.

```yaml
knx:
  switch:
    - name: "Kitchen coffee maker"
      address: "1/1/6"
```

{% configuration %}
address:
  description: KNX group address for switching the switch on/off. *DPT 1*
  required: true
  type: [string, list]
name:
  description: A name for this device used within Home Assistant.
  required: false
  default: KNX Switch
  type: string
state_address:
  description: Separate KNX group address for retrieving the switch state. *DPT 1*
  required: false
  type: [string, list]
invert:
  description: Invert the telegrams payload before processing or sending.
  required: false
  type: boolean
  default: false
respond_to_read:
  description: Respond to GroupValueRead telegrams received to the configured `address`.
  required: false
  type: boolean
  default: false
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

The optional `state_address` can be used to inform Home Assistant about state changes not triggered by a telegram to the `address` e.g., if you configure a timer on a channel. If a KNX message is seen on the bus addressed to the given state address, this will overwrite the state of the switch object.

Switch entities without a `state_address` will restore their last known state after Home Assistant was restarted.
Switches having a `state_address` configured request their current state from the KNX bus.

## Weather

The KNX weather platform is used as an interface to KNX weather stations.

To use your KNX weather station in your installation, add the following lines to your top level [KNX Integration](/integrations/knx) configuration key in `configuration.yaml`:

```yaml
# Example configuration.yaml entry
knx:
  weather:
    - name: "Home"
      address_temperature: "7/0/0"
      address_brightness_south: "7/0/1"
      address_brightness_west: "7/0/2"
      address_brightness_east: "7/0/3"
      address_brightness_north: "7/0/11"
      address_wind_speed: "7/0/4"
      address_rain_alarm: "7/0/5"
      address_frost_alarm: "7/0/6"
      address_wind_alarm: "7/0/7"
      address_day_night: "7/0/8"
      address_air_pressure: "7/0/9"
      address_humidity: "7/0/10"
      sync_state: true
```

{% configuration %}
name:
  description: A name for this device used within Home Assistant.
  required: false
  default: KNX Weather
  type: string
address_temperature:
  description: KNX group address for reading current outside temperature from KNX bus. *DPT 9.001*
  required: true
  type: [string, list]
address_brightness_south:
  description: KNX group address for reading current brightness to south coordinate from KNX bus. *DPT 9.004*
  required: false
  type: [string, list]
address_brightness_west:
  description: KNX group address for reading current brightness to west coordinate from KNX bus. *DPT 9.004*
  required: false
  type: [string, list]
address_brightness_east:
  description: KNX group address for reading current brightness to east coordinate from KNX bus. *DPT 9.004*
  required: false
  type: [string, list]
address_brightness_north:
  description: KNX group address for reading current brightness to north coordinate from KNX bus. *DPT 9.004*
  required: false
  type: [string, list]
address_wind_bearing:
  description: KNX group address for reading current wind bearing from KNX bus. *DPT 5.003*
  required: false
  type: [string, list]
address_wind_speed:
  description: KNX group address for reading current wind speed from KNX bus. *DPT 9.005*
  required: false
  type: [string, list]
address_rain_alarm:
  description: KNX group address for reading if rain alarm is on/off.
  required: false
  type: [string, list]
address_frost_alarm:
  description: KNX group address for reading if frost alarm is on/off.
  required: false
  type: [string, list]
address_wind_alarm:
  description: KNX group address for reading if wind alarm is on/off.
  required: false
  type: [string, list]
address_day_night:
  description: KNX group address for reading if it's day/night.
  required: false
  type: [string, list]
address_air_pressure:
  description: KNX address reading current air pressure. *DPT 9.006*
  required: false
  type: [string, list]
address_humidity:
  description: KNX address for reading current humidity. *DPT 9.007*
  required: false
  type: [string, list]
sync_state:
  description: Actively read the value from the bus. If `false` no GroupValueRead telegrams will be sent to the bus.
  required: false
  type: boolean
  default: true
entity_category:
  description: The [category](https://developers.home-assistant.io/docs/core/entity#generic-properties) of the entity.
  required: false
  type: string
  default: None
{% endconfiguration %}

## Troubleshooting / Common issues

### Logs for the KNX integration

`xknx`, the library used for KNX communication, provides various logging handlers for monitoring and debug purposes.
Add the following lines to your Home Assistant `configuration.yaml` to activate them:

```yaml
logger:
  default: warning
  logs:
    # For most debugging needs `xnx.log` and one of `xknx.knx` or `xknx.telegram` are a good choice.
    xknx: debug  # sets the level of all loggers
    xknx.log: debug  # provides general information (connection, etc.)
    xknx.raw_socket: debug  # logs incoming UDP frames in raw hex format
    xknx.knx: debug  # logs incoming and outgoing KNX/IP frames at socket level
    xknx.telegram: debug  # logs telegrams before they are being processed at device level or sent to an interface
    xknx.state_updater: debug  # provides information about the state updater
```

You can use the service `logger.set_level` to change the log level of a handler on a running instance.
{% my developer_call_service badge service="logger.set_level" %}

### Group address can not be read

Every `*_state_address` is read on startup sequentially if not configured differently. If you see the following errors in your log, a group address could not be read by a GroupValueRead request from Home Assistant in time.

```log
> Could not sync group address '1/2/3' (Entity name - Feature)
> Error: KNX bus did not respond in time (2.0 secs) to GroupValueRead request for: '1/2/3'
```

#### No communication object (CO) assigned to the group address (GA) has the Read-Flag set in ETS

- Enable the read flag for *one* CO assigned to the GA. Use the one most likely to hold the current state (e.g., for a light entity's `brightness_state_address` the according CO of the dimming actuator).

#### Response telegrams are not passing a line coupler, router or other filter in the installation

- Use a dummy device in ETS for Home Assistant. These can be found in the ETS online catalog. Assign it to the line your interface connects Home Assistant to and link its communication objects to the group addresses you are using in Home Assistant. ETS will generate filter tables that are applied to your line couplers after updating their application.

#### Unresponsive system

- The timeout for logging the errors (2 seconds) is started when the GroupValueRead request is scheduled to be sent. On systems experiencing high loads sending can be delayed (e.g., Raspberry Pi running lots of integrations at startup).
Incoming response telegrams are always processed, so no information gets lost.

### Duplicate entities

If you find following error in your log you seem to have a duplicated entity in your configuration.

```log
Platform knx does not generate unique IDs. ID 1/2/3 already exists - ignoring platform.name
```

The `unique_id` for KNX entities is generated based on required configuration values.

- binary_sensor: `state_address`
- climate: `temperature_address` `target_temperature_state_address` `target_temperature_address` `setpoint_shift_address`
- cover: `move_long_address` `position_address`
- fan: `address`
- light: `address` or all combined `brightness_address` if only `individual_colors` is used
- notify: `address`
- scene: `address` and `scene_number`
- sensor: `state_address`
- switch: `address`
- weather: `address_temperature`

There can not be multiple entities on the same platform sharing these exact group addresses, even if they differ in other configuration.

### xknx.yaml configuration

```log
> The 'config_file' option near /homeassistant/configuration.yaml:42 is deprecated, please remove it from your configuration
> Invalid config for [knx]: [config_file] is an invalid option for [knx]. Check: knx->knx->config_file.
```

The feature to specify a xknx configuration schema file in the Home Assistant configuration YAML file (via `config_file:`) is deprecated since Home Assistant 2021.4. You can use the [xknx.yaml config converter](https://xknx.io/config-converter/) to convert it to a Home Assistant compatible `configuration.yaml` schema.
