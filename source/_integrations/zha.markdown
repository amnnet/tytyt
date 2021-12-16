---
title: Zigbee Home Automation
description: Instructions on how to integrate your Zigbee Home Automation (ZHA) devices within Home Assistant.
ha_category:
  - Hub
  - Alarm
  - Binary Sensor
  - Climate
  - Fan
  - Light
  - Lock
  - Sensor
  - Switch
  - Cover
  - Number
ha_release: 0.44
ha_iot_class: Local Polling
featured: true
ha_config_flow: true
ha_codeowners:
  - '@dmulcahey'
  - '@adminiuga'
ha_domain: zha
ha_platforms:
  - alarm_control_panel
  - binary_sensor
  - climate
  - cover
  - device_tracker
  - fan
  - light
  - lock
  - number
  - sensor
  - switch
ha_zeroconf: true
---

The ZHA (Zigbee Home Automation) integration allows you to connect many off-the-shelf [Zigbee based devices](https://zigbeealliance.org) directly to Home Assistant, using one of the many available Zigbee coordinators.

ZHA uses an open-source Python library implementing a hardware-independent Zigbee stack called [zigpy](https://github.com/zigpy/zigpy). All coordinators compatible with zigpy can be used with ZHA.

There is currently support for the following device types within Home Assistant:

- Alarm Control Panel
- Binary Sensor
- Climate (beta)
- Cover
- Fan
- Light
- Lock
- Number (i.e. analog output)
- Sensor
- Switch

There is also support for grouping of lights, switches, and fans (i.e. support for commanding device groups as entities). At least two entities must be added to a group before the group entity is created. As well as support for binding/unbinding (i.e. bind a remote to a lightbulb or group).

## Compatible hardware

ZHA integration uses a hardware independent Zigbee stack implementation with modular design, which means that it can support any one of the many Zigbee coordinator radio modules/adapters available from different manufacturers, as long as that module/adapter is compatible with [zigpy](https://github.com/zigpy/zigpy).

Note! Zigbee 3.0 support or not in zigpy, depends primarily on your Zigbee coordinator hardware and its firmware. Some Zigbee coordinator hardware supports Zigbee 3.0 but might be shipped with an older firmware which does not. In such a case you may want to upgrade the firmware manually yourself.

Some other Zigbee coordinator hardware may not support a firmware that is capable of Zigbee 3.0 at all but can still be fully functional and feature-complete for your needs. This is very common as many, if not most, Zigbee devices do not yet Zigbee 3.0. As a general rule, newer Zigbee coordinator hardware generally supports Zigbee 3.0 firmware and it is up to its manufacturer to make such firmware available for them.

### Known working Zigbee radio modules

- dresden elektronik deCONZ based Zigbee radios (via the [zigpy-deconz](https://github.com/zigpy/zigpy-deconz) library for zigpy)
  - [ConBee II (a.k.a. ConBee 2) USB adapter from dresden elektronik](https://phoscon.de/conbee2)
  - [ConBee USB adapter from dresden elektronik](https://phoscon.de/conbee)
  - [RaspBee II (a.k.a. RaspBee 2) Raspberry Pi Shield from dresden elektronik](https://www.dresden-elektronik.com/product/raspbee-II.html)
  - [RaspBee Raspberry Pi Shield from dresden elektronik](https://phoscon.de/raspbee)
- Silicon Labs EmberZNet based radios using the EZSP protocol (via the [bellows](https://github.com/zigpy/bellows) library for zigpy)
  - [ITead Zigbee 3.0 USB Dongle (EFR32MG21) Model 9888010100045](https://www.itead.cc/zigbee-3-0-usb-dongle.html)
  - [ITead Sonoff ZBBridge](https://www.itead.cc/smart-home/sonoff-zbbridge.html) (Note! [WiFi-based bridges are not recommended for ZHA with EZSP radios](https://github.com/home-assistant/home-assistant.io/issues/17170). Also, this first have to be flashed with [Tasmota firmware and Silabs EmberZNet NCP EZSP UART Host firmware to use as Serial-to-IP adapter](https://www.digiblur.com/2020/07/how-to-use-sonoff-zigbee-bridge-with.html))
  - [Nortek GoControl QuickStick Combo Model HUSBZB-1 (Z-Wave & Zigbee Ember 3581 USB Adapter)](https://www.nortekcontrol.com/products/2gig/husbzb-1-gocontrol-quickstick-combo/) (Note! Not a must but recommend [upgrade the EmberZNet NCP application firmware](https://github.com/walthowd/husbzb-firmware))
  - [Elelabs Zigbee USB Adapter](https://elelabs.com/products/elelabs_usb_adapter.html)/[POPP ZB-Stick](https://shop.zwave.eu/detail/index/sArticle/2496) (Note! Not a must but recommend [upgrade the EmberZNet NCP application firmware](https://github.com/Elelabs/elelabs-zigbee-ezsp-utility))
  - [Elelabs Zigbee Raspberry Pi Shield](https://elelabs.com/products/elelabs_zigbee_shield.html) (Note! Not a must but recommend [upgrade the EmberZNet NCP application firmware](https://github.com/Elelabs/elelabs-zigbee-ezsp-utility))
  - [Bitron Video/Smabit BV AV2010/10 USB-Stick](https://bv.smabit.eu/index.php/smart-home-produkte/zb-funkstick/) with Silicon Labs Ember 3587
  - Telegesis ETRX357USB/ETRX357USB-LR/ETRX357USB-LRS+8M (Note! These first have to be [flashed with other EmberZNet firmware](https://github.com/walthowd/husbzb-firmware))
- Texas Instruments based radios (via the [zigpy-znp](https://github.com/zigpy/zigpy-znp) library for zigpy)
  - [CC2652P/CC2652R/CC2652RB USB stick, module, or dev board hardware flashed with Z-Stack coordinator firmware](https://www.zigbee2mqtt.io/information/supported_adapters)
  - [CC1352P/CC1352R USB stick, module, or dev board hardware flashed with Z-Stack coordinator firmware](https://www.zigbee2mqtt.io/information/supported_adapters)
  - [CC2538 USB stick, module, or dev board hardware flashed with Z-Stack coordinator firmware](https://www.zigbee2mqtt.io/information/supported_adapters)
  - [CC2530/CC2531 USB stick, module, or dev board hardware flashed with Z-Stack coordinator firmware](https://www.zigbee2mqtt.io/information/supported_adapters) (not recommended for Zigbee networks with more than 20 devices)
- Digi XBee Zigbee based radios (via the [zigpy-xbee](https://github.com/zigpy/zigpy-xbee) library for zigpy)
  - [Digi XBee Series 3 (xbee3-24)](https://www.digi.com/products/embedded-systems/digi-xbee/rf-modules/2-4-ghz-rf-modules/xbee3-zigbee-3) and [Digi XBee Series S2C](https://www.digi.com/products/embedded-systems/digi-xbee/rf-modules/2-4-ghz-rf-modules/xbee-zigbee) modules
    - Note! While not a must, [it is recommend to upgrade XBee Series 3 and S2C to newest firmware using XCTU](https://www.digi.com/resources/documentation/Digidocs/90002002/Default.htm#Tasks/t_load_zb_firmware.htm)
  - [Digi XBee Series 2 (S2)](https://www.digi.com/support/productdetail?pid=3430) modules (Note! This first have to be [flashed with Zigbee Coordinator API firmware](https://www.digi.com/support/productdetail?pid=3430))
- ZiGate based radios (via the [zigpy-zigate](https://github.com/zigpy/zigpy-zigate) library for zigpy and require firmware 3.1d or later)
  - [ZiGate USB](https://zigate.fr/produit/zigate-usb/)
  - [ZiGate USB-DIN](https://zigate.fr/produit/zigatev2-usb-din/)
  - [PiZiGate (ZiGate Raspberry Pi module)](https://zigate.fr/produit/pizigatev2/)
  - [ZiGate-Ethernet (Ethernet gateway board for PiZiGate)](https://zigate.fr/produit/zigate-ethernet/)
  - [ZiGate + WiFi Pack](https://zigate.fr/produit/zigatev2-pack-wifi/)

#### Warning about Wi-Fi-based Zigbee-to-Serial bridges/gateways

<div class="note warning">

The **EZSP** protocol requires a stable connection to the serial port. With _ITEAD Sonoff ZBBridge_ connecting over the WiFi network
it is expected to see `NCP entered failed state. Requesting APP controller restart` in the logs. This is a normal part of the operation and indicates there was a drop in communication between ZHA and SonOff bridge.

</div>

## Configuration - GUI

Connect your radio module and restart Home Assistant.

From the Home Assistant front page go to **Configuration** and then select **Integrations** from the list.

Use the plus button in the bottom right to add a new integration called **ZHA**.

In the popup:

- Serial Device Path - List of detected serial ports on the system. You need to pick one to which your
radio is connected
- Submit

Press `Submit` and the integration will try to detect radio type automatically. If unsuccessful, you will get
a new pop-up asking for a radio type. In the pop-up:

- Radio Type

| Radio Type | Zigbee Radio Hardware |
| ------------- | ------------- |
| `ezsp`  | Silicon Labs EmberZNet protocol (e.g., Elelabs, HUSBZB-1, Telegesis) |
| `deconz` | dresden elektronik deCONZ protocol (e.g., ConBee I/II, RaspBee I/II) |
| `znp` | Texas Instruments (e.g., CC253x, CC26x2, CC13x2) |
| `zigate` | ZiGate Serial protocol (e.g., ZiGate USB-TTL, PiZiGate, ZiGate WiFi) |
| `xbee` | Digi XBee ZB Coordinator Firmware protocol (e.g., Digi XBee Series 2, 2C, 3) |

- Submit

Press `Submit` to save radio type and you will get a new form asking for port settings specific for this
radio type. In the pop-up:
- Serial device path
- port speed (not applicable for all radios)
- data flow control (not applicable for all radios)

Most devices need at the very least the serial device path, like `/dev/ttyUSB0`, but it is recommended to use
device path from `/dev/serial/by-id` folder,
e.g., `/dev/serial/by-id/usb-Silicon_Labs_HubZ_Smart_Home_Controller_C0F003D3-if01-port0`  
A list of available device paths can be found in **Supervisor** > **System** > **Host System** > **dot menu** > **Hardware**.

Press `Submit`. The success dialog will appear or an error will be displayed in the popup. An error is likely if Home Assistant can't access the USB device or your device is not up to date. Refer to [Troubleshooting](#troubleshooting) below for more information.

### ZiGate or Sonoff ZBBridge Devices

If you are use ZiGate or Sonoff ZBBridge you have to use some special usb_path configuration:

- ZiGate USB TTL or DIN: `/dev/ttyUSB0` or `auto` to auto discover the zigate
- PiZigate : `pizigate:/dev/ttyS0`
- Wifi Zigate : `socket://[IP]:[PORT]` for example `socket://192.168.1.10:9999`
- Sonoff ZBBridge : `socket://[IP]:[PORT]` for example `socket://192.168.1.11:8888`

### Discovery via USB or Zeroconf

Some devices can be auto-discovered, which can simplify the ZHA setup process. The following devices have been tested with discovery and offer a quick setup experience:

| Device | Discovery Method | Identifier |
| -------| ---------------- | ---------- |
| [ConBee II](https://phoscon.de/en/conbee2) | USB | 1CF1:0030 |
| [Nortek HUSBZB-1](https://www.nortekcontrol.com/products/2gig/husbzb-1-gocontrol-quickstick-combo/) | USB | 10C4:8A2A |
| [slae.sh CC2652RB development stick](https://slae.sh/projects/cc2652/) | USB | 10C4:EA60 |
| [Tube’s EFR32 Pro Ethernet/Serial Coordinator](https://www.tubeszb.com/) | USB| 10C4:EA60 |
| [Tube's CC2652P2 USB-powered Zigbee to Ethernet Serial Coordinator)](https://www.tubeszb.com/) | Zeroconf | tube_zb_gw_cc2652p2.local. |
| [Tube's CC2652P2 PoE-powered Zigbee to Ethernet Serial Coordinator)](https://www.tubeszb.com/) | Zeroconf | tube_zb_gw_cc2652p2_poe.local. |
| [Tube's EFR32 Based Zigbee to Ethernet Serial Coordinator)](https://www.tubeszb.com/) | Zeroconf | tube_zb_gw_efr32.local. |

Additional devices in the [Known working Zigbee radio modules](#known-working-zigbee-radio-modules) list may be discoverable, however, only devices that have been confirmed discoverable are listed above.

## Configuration - YAML

For more advanced configuration, you can modify `configuration.yaml` and restart Home Assistant

{% configuration %}
database_path:
  description: _Full_ path to the database which will keep persistent network data.
  required: true
  type: string
enable_quirks:
  description: Enable quirks mode for devices where manufacturers didn't follow specs.
  required: false
  type: boolean
  default: true
custom_quirks_path:
  description: Full path to a directory containing custom quirk modules that will take precedence over any built-in quirks matching a device.
  required: false
  type: string
{% endconfiguration %}

### OTA firmware updates

ZHA component has the ability to automatically download and perform OTA (Over-The-Air) firmware updates of Zigbee devices if the OTA firmware provider source URL for updates is available. OTA firmware updating is set to disabled (`false`) in the configuration by default.

Currently, OTA providers for firmware updates are only available for IKEA and LEDVANCE devices. OTA updates for device of other manufactures could also be supported by ZHA dependencies in the future, if these manufacturers publish their firmware publicly.

To enable OTA firmware updates for the ZHA integration you need to add the following configuration to your `configuration.yaml` and restart Home Assistant:

```yaml
zha:
  zigpy_config:
    ota:
      ikea_provider: true                        # Auto update Trådfri devices
      ledvance_provider: true                    # Auto update LEDVANCE devices
      #otau_directory: /path/to/your/ota/folder  # Utilize .ota files to update everything else
```

You can choose if the IKEA or LEDVANCE provider should be set to enabled (`true`) or disabled (`false`) individually. After the OTA firmware upgrades are finished, you can set these to `false` again if you do not want ZHA to automatically download and perform OTA firmware upgrades in the future.

Note that the `otau_directory` setting is optional and can be used for any firmware files you have downloaded yourself, for any device type and manufacturer. For example, Philips Hue firmwares manually downloaded from [here](https://github.com/dresden-elektronik/deconz-rest-plugin/wiki/OTA-Image-Types---Firmware-versions) and/or [here](https://github.com/Koenkk/zigbee-OTA/blob/a02a4cb33f7c46b4d2916805bfcad582124ec975/index.json) added to the `otau_directory` can be flashed, although a manual `zha.issue_zigbee_cluster_command` command currently (as of 2021.3.3) must be issued against the IEEE of the Philips Hue device under Developer Tools->Services, e.g.:

```yaml
service: zha.issue_zigbee_cluster_command
data:
  ieee: "xx:xx:xx:xx:xx:xx:xx:xx"
  endpoint_id: 11
  cluster_id: 25
  cluster_type: out
  command: 0
  command_type: client
  args:
    - 0
    - 100
```

Note: `cluster_id: 25` may also be `cluster_id: 0x0019`. The two are synonymous. 

### Defining Zigbee channel to use

ZHA prefers to use Zigbee channel 15 by default. You can change this using YAML configuration, but this only works
if there's no existing network. To change the channel for an existing network, radio has to be factory reset and a new network to be formed. This requires re-pairing of all the devices.

```yaml
zha:
  zigpy_config:
    network:
      channel: 15             # What channel the radio should try to use.
      channels: [15, 20, 25]  # Channel mask
```

This is a good reference for channel selection [Zigbee and Wifi Coexistance](https://support.metageek.com/hc/en-us/articles/203845040-ZigBee-and-WiFi-Coexistence)

### Modifying the device type

As not all device manufacturers follow the Zigbee standard, at times a device can be incorrectly classified. For example, a switch could be classified as a light.

To correct the device type, also called domain, add the following to your `configuration.yaml` and restart Home Assistant:

```yaml
zha:
  device_config:
    84:71:27:ff:fe:93:17:24-1:    # format: {ieee}-{endpoint_id}
      type: "switch"              # corrected device type
```

`{ieee}` is the device hardware address which can be read from the Home Assistant UI when looking at *Device info*. From device info, you can find the `{endpoint_id}` by viewing the *Zigbee device signature*.

## Services

### Service `zha.permit`

To add new devices to the network, call the `permit` service on the `zha` domain. Do this by clicking the Service icon in Developer tools and typing `zha.permit` in the **Service** dropdown box. Next, follow the device instructions for adding, scanning or factory reset.

This service opens network for joining new devices.

|  Data | Optional | Description |
| ---- | ---- | ----------- |
| `duration` |  yes | For how long to allow new devices to join, default 60s
| `ieee` | yes | allow new devices to join via an existing device

To join a new device using an install code (ZB3 devices) use the following data attributes (must use parameters only
from the same group:

|  Data | Parameter Group | Description |
| ---- | ---- | ----------- |
| `src_ieee` | install_code | The IEEE address of the joining ZB3 device. Use with `install_code`
| `install_code` | install_code | Install Code of the joining device. Use with `src_ieee`
| `qr_code` | qr_code | QR code containing IEEE and Install Code of the joining ZB3 device

<div class='note'>

  Currently `qr_code` supports QR Install Codes from:
  
  - Aqara
  - Consciot
  - Embrighten

</div>

### Service `zha.remove`

This service removes an existing device from the network. You can find the IEEE address of the device on the device card of Zigbee devices. An example of an IEEE address data parameter format is `00:0d::6f:00:05:7d:2d:34`.

|  Data | Optional | Description |
| ---- | ---- | ----------- |
| `ieee` | no | IEEE address of the device to remove

### Service `zha.set_lock_user_code`

This service sets a lock code on a Zigbee lock.

|  Data | Optional | Description |
| --------- | ---- | ----------- |
| `code_slot` | no | Which lock code slot to store the code. Ex. 1-32 will work for Kwikset 954
| `user_code` | no | Code to set on the lock. Ex. Kwikset accepts numbers 4-8 digits in length

### Service `zha.clear_lock_user_code`

This service clears a lock code from a Zigbee lock.

|  Data | Optional | Description |
| --------- | ---- | ----------- |
| `code_slot` | no | Which lock code slot to clear

### Service `zha.enable_lock_user_code`

This service enables a lock code on a Zigbee lock.

|  Data | Optional | Description |
| --------- | ---- | ----------- |
| `code_slot` | no | Which lock code slot to enable

### Service `zha.disable_lock_user_code`

This service disables a lock code on a Zigbee lock.

|  Data | Optional | Description |
| --------- | ---- | ----------- |
| `code_slot` | no | Which lock code slot to disable

## Adding devices

To add a new device:

1. Go to the **Integrations** page, find the **Zigbee Home Automation** integration that was added by the configuration steps above, and select **Configure**.
1. Click on the plus button at the bottom right corner to start a scan for new devices.
1. Reset your Zigbee devices according to the device instructions provided by the manufacturer (e.g., turn on/off lights up to 10 times, switches usually have a reset button/pin). It might take a few seconds for the devices to appear. You can click on **Show logs** for more verbose output.
1. Once the device is found, it will appear on that page and will be automatically added to your devices. You can optionally change its name and add it to an area (you can change this later). You can search again to add another device, or you can go back to the list of added devices.

### Best practices for avoiding pairing difficulties

- If possible try to pair your Zigbee devices in their intended final location, (and not pair it next to the Zigbee coordinator and then need to move it after). 
  - Pairing a Zigbee device next to the Zigbee coordinator and then moving it later can result in dropped/lost connections or other issues. 
    - If the device you want to add is not brand new and as such never paired before then you always have to make sure to first manually reset the device to its factory default settings before you will be able to add/pair it. 
- Some battery-operated Zigbee devices are known to have problems with pairing if they have Low battery voltage. 
    - Some people have reported replacing the battery on their newly received Xiaomi/Aqara devices solved pairing issues. 
- Check that you have enough Zigbee router devices (also known as Zigbee signal repeaters or range extenders) and if you do not have any, invest and add some mains-powered devices that will work as Zigbee routers. 
    - Aim to start out with mains-powered devices before adding battery-operated devices as a "weak" Zigbee network mesh (e.g., the device is too far from the Zigbee coordinator or a Zigbee router) may prevent some devices from being paired. Zigbee router devices are also needed to increase the maximum of devices that can be connected to your Zigbee mesh network. 
    - Note that some Zigbee devices are not fully compatible with all brands of Zigbee router devices. Xiaomi/Aqara devices are for example known not to work with Zigbee router devices from Centralite, General Electrics, Iris, Ledvance/OSRAM/ LIGHTIFY/Sylvania, Orvibo, PEQ, Securifi, and SmartThings/Samsung. Better results can usually be achieved by using mains-powered devices IKEA and Nue/3A Home or dedicated DIY routing devices based on Texas Instruments CC253x/CC26x2 and XBee Series 2/3 Zigbee radios. 
- Be patient as the pairing of some Zigbee devices may require multiple attempts and you may sometimes need to try again and again. 
    - Some devices, like example those from Xiaomi/Aqara, are known to not be 100% compliant with the standard Zigbee specifications and may therefore require many paring attempts over 10-20 minutes or longer. 

### Using router devices

You use routers to increase the number of Zigbee devices that can be used in a network. The total number of Zigbee devices that you have on a Zigbee network depends on a few things, but you should know that Zigbee coordinator hardware and firmware only plays a larger role in Zigbee networks with a lot of devices. More important is how many directly connected devices ("direct children") versus how many routers are connected to your Zigbee coordinator. Zigpy library which ZHA uses has an upper limit. This is 32 direct children, but if your Zigbee coordinator hardware is powerful enough then you can still have hundreds of Zigbee devices connected through routers.  

Even the least powerful Zigbee coordinator hardware supported by Zigpy is CC2530/2531 and its default firmware, only supports 20 devices connected directly to the coordinator. However, by having routers in your Zigbee network, the mesh network size can be extended. You can assume that most, if not all mains/AC-powered devices, e.g., wall-plugs and always powered-on lightbulbs in your Zigbee network can serve as a router. You can even use CC2530/CC2531 with router firmware, as additional routers (which in their turn have a limit of 21 devices).

An example using the default CC2531 coordinator firmware + two CC2531 routers; Your device limit will be:

- Coordinator: 15 devices - 2 routers = 13
- Router one: + 21 devices
- Router two: + 21 devices
- Total device limit = **55 devices**

### Binding and unbinding

ZHA support for binding and unbinding. Binding is an action in Zigbee which defines relations between two Zigbee devices, specific endpoints, and cluster id. It provides a mechanism for attaching an endpoint on one Zigbee node to one or more endpoints on another Zigbee node or Zigbee group (a group of Zigbee devices). 

Binding is a "target destination" in form of a device address or group ID, endpoint, and cluster. For example, binding a Zigbee device like a remote to a Zigbee lightbulb, switch or group of lightbulbs allows direct control of the "target" device (light, switch, shade) from the "remote" Zigbee device, bypassing ZHA.  This means that the remote can control the lightbulb/group of lightbulbs even when the Zigbee coordinator is not available.
Binding is only supported between the same cluster, for example, "output cluster id 6" (on/off cluster) of a remote, can be only bound to an "input cluster id 6" on the target device -- light, switch.  

Note that not all devices support binding as it depends on the Zigbee implementation of the device itself. Also, by default ZHA bind remotes to the coordinator, so the coordinator could receive ZCL commands from the remotes and originate zha_events. However, some remotes, for example, the Philips RWL021 can only be bound to a single destination and it is not possible to make this switch to bind to other destinations like a device or groups unless you first unbind the remote from the coordinator. After you unbind the remote from the ZHA coordinator you can then bind it directly to any other Zigbee device or a group.

Binding a remote directly to a bulb or group has the benefit of faster response time and smoother control. This greatly improves user feedback experience functions like dimming as the remote then directly dims the lightbulb and thus does not have to make the software roundtrip via the ZHA coordinator.

## Troubleshooting

To help resolve any kinks or compatibility problems, report bugs as issues with debug logs. Please follow the instructions in this troubleshooting section.

### Knowing which devices are supported

There is no official compatibility list of supported devices for the simple reason that practically all devices Zigbee Home Automation that are fully compliant with the standards and specifications as set by the [Zigbee Alliance](https://zigbeealliance.org) should technically be compatible with this ZHA integration. The fact remains, however, that some hardware manufacturers do not always fully comply with each set specification, which can cause a few devices to only partially work or not work at all with ZHA, but developers can create workarounds for such issues via a solution for 'ZHA exception and deviation handling' that this implementation features. See that section for more information.

Tip to new users is that, while there is no official list of supported devices, some ZHA users take comfort that blakadder maintains an unofficial Zigbee Device Compatibility Repository which anyone can submit compatibility reports to, it can be found at [zigbee.blakadder.com](https://zigbee.blakadder.com) and currently contains independent compatibility lists and device pairing tips for several home automation gateway/bridge/hub software, including but not limited to open source Zigbee implementations such as; ZHA, Tasmota, Zigbee2MQTT, and ZiGate.

### ZHA exception and deviation handling

The ZHA implementation in Home Assistant relies on a library called "[ZHA Device Handlers](https://github.com/zigpy/zha-device-handlers)" to resolve issues with Zigbee devices that do not fully conform with the Zigbee standards. The few devices that deviate from the Zigbee specifications set by the [Zigbee Alliance](https://zigbeealliance.org) may therefore require proper bug reports with debug logs from users to assistant the developers in writing custom [ZHA Device Handlers](https://github.com/zigpy/zha-device-handlers/blob/dev/README.md) for all of a device functions to work properly with the ZHA integration.

Such a custom "ZHA Device Handler" are Python scripts that internally are also referred to as a "quirk" because they fix "quirks", like deviations from the standard specifications. ZHA Device Handles do this by transparently, acting as a translator, translating and converting non-compliant device messages and instead present them to the application as coming from a virtual compliant device. These ZHA Device Handlers for Home Assistant can thus be used to parse custom messages to and from Zigbee devices. The ZHA Device Handlers that are made can then be reused by all users in future versions of Home Assistant.

The custom quirks implementations for zigpy implemented as ZHA Device Handlers for Home Assistant are a similar concept to that of [Hub-connected Device Handlers for the SmartThings Classics platform](https://docs.smartthings.com/en/latest/device-type-developers-guide/) as well as that of [Zigbee-Herdsman Converters (formerly Zigbee-Shepherd Converters) as used by Zigbee2mqtt](https://www.zigbee2mqtt.io/how_tos/how_to_support_new_devices.html), meaning they are each virtual representations of a physical device that expose additional functionality that is not provided out-of-the-box by the existing integration between these platforms.

### Reporting issues

When reporting issues, please provide the following information in addition to information requested by issue template:

1. Debug logs for the issue, see [debug logging](#debug-logging)
2. Model of Zigbee radio being used
3. If issue is related to a specific Zigbee device, provide device Zigbee signature. Signature is available at
**Configuration** -> **Integrations** -> **Zigbee Home Automation** (click **Configure**) -> **Devices** (pick your device) -> **Zigbee Device Signature**

### Debug logging

To enable debug logging for ZHA component and radio libraries, add the following [logger](/integrations/logger/) configuration to `configuration.yaml`:

```yaml
logger:
  default: info
  logs:
    homeassistant.core: debug
    homeassistant.components.zha: debug
    bellows.zigbee.application: debug
    bellows.ezsp: debug
    zigpy: debug
    zigpy_deconz.zigbee.application: debug
    zigpy_deconz.api: debug
    zigpy_xbee.zigbee.application: debug
    zigpy_xbee.api: debug
    zigpy_zigate: debug
    zigpy_znp: debug
    zhaquirks: debug
```

### Add Philips Hue bulbs that have previously been added to another bridge

Philips Hue bulbs/lights that have previously been paired/added to another bridge/gateway will not show up during search in ZHA to pair/add a Zigbee device. That is because you have to first manually restore your bulbs/lights back to their factory default settings first, and just removing them from your old bridge/gateway is not enough to do so. Instead to achieve a proper device factory reset you can use one of these methods below.

Using a Philips Hue Dimmer Switch or Lutron Connected Bulb Remote is probably the easiest way to factory-reset your bulbs. For this to work, the remote does not have to be paired with your previous bridge. Also, make sure there are no other Hue bulbs nearby that have just been turned on when using this method as you otherwise risk resetting them too.

#### Philips Hue Dimmer Switch

1. Turn on your Hue bulb/light you want to reset. (It is important that the bulb has just been turned).
2. Hold the Philips Hue Dimmer Switch near your bulb (closer than 10 centimeters / 4 inches).
3. Press and hold the (I)/(ON) and (O)/(OFF) buttons on the Philips Hue Dimmer Switch for about 10 seconds continuously until your bulb starts to blink. On the newer switches hold the I/O and Scene Switch buttons.
4. Your bulb should stop blinking and eventually turn on again. At the same time, a green light on the top left of your remote indicates that your bulb has been successfully reset to factory default settings.

Note: If you are not unable to reset the bulb, remove it from the Hue Bridge and retry the procedure.

#### Lutron Connected Bulb Remote

1. Turn on your Hue bulb/light you want to reset. (It is important that the bulb has just been turned).
2. Hold the Dimmer Switch near your bulb (closer than 10 centimeters / 4 inches)
3. Press and hold the 2nd (up arrow) and 4th (light off) buttons on the Lutron Connected Bulb Remote simultaneously for about 10 seconds continuously until your bulb starts to blink and the green LED on the remote should also start blink slowly.
4. Continue to hold both buttons on the remote until the green LED on it stops blinking. Your bulb should also have stopped blinking and eventually turn on again indicating that your bulb has been successfully reset to factory default settings.

#### hue-thief

Follow the instructions on [https://github.com/vanviegen/hue-thief/](https://github.com/vanviegen/hue-thief/) (EZSP-based Zigbee USB stick required)

### ZHA Start up issue with Home Assistant or Home Assistant Container

On Linux hosts ZHA can fail to start during HA startup or restarts because the Zigbee USB device is being claimed by the host's modemmanager service. To fix this disable the modemmanager on the host system.

To remove modemmanager from a Debian/Ubuntu host run this command:

```bash
sudo apt-get purge modemmanager
```

### Can't connect to USB device and using Docker

If you are using Docker and can't connect, you most likely need to forward your device from the host machine to the Docker instance. This can be achieved by adding the device mapping to the end of the startup string or ideally using Docker compose.

#### Docker Compose

Install Docker-Compose for your platform (Linux - `sudo apt-get install docker-compose`).

Create a `docker-compose.yml` with the following data:

```yaml
version: '2'
services:
  homeassistant:
    # customizable name
    container_name: home-assistant

    # must be image for your platform, this is the rpi3 variant
    image: homeassistant/raspberrypi3-homeassistant
    volumes:
      - <DIRECTORY HOLDING HOME ASSISTANT CONFIG FILES>:/config
      - /etc/localtime:/etc/localtime:ro
    devices:
      # your usb device forwarding to the docker image
      - /dev/ttyUSB0:/dev/ttyUSB0
    restart: always
    network_mode: host
```
