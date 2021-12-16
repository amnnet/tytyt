---
title: Bosch SHC
description: Integrate Bosch SHC.
ha_category:
  - Binary Sensor
ha_release: 2021.6
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@tschamm'
ha_domain: bosch_shc
ha_platforms:
  - binary_sensor
  - sensor
ha_zeroconf: true
---

The Bosch SHC integration allows you to integrate your [Bosch Smart Home Controller](https://www.bosch-smarthome.com) into Home Assistant.

There is currently support for the following device types within Home Assistant:

- [Binary Sensor](#binary-sensor)
- [Sensor](#sensor)

{% include integrations/config_flow.md %}

### Binary Sensor

The binary sensor platform allows you to monitor the states of your shutter contact sensors. Binary sensor devices are added for each of the following devices:

- Shutter Contact

### Sensor

The sensor platform allows you to monitor the states of your temperature, humidity, purity, air quality, power, energy, valve tappet and battery sensors. Sensor devices are added for each of the following devices:

- Thermostat
- Wall Thermostat
- Twinguard
- Smart Plug
- Smart Plug Compact
- Smoke Detector
- Shutter Contact
- Universal Switch

## Client registration

To start the client registration, press and hold the button on the controller until the LED starts flashing. During configuration, a client SSL cert/key pair is generated and registered on the controller. For this step, the system password of your controller is needed, which was created upon initial setup of the controller.
