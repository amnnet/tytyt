---
title: MusicCast
description: Instructions on how to integrate Yamaha MusicCast Receivers into Home Assistant.
ha_category:
  - Media Player
ha_release: 0.53
ha_codeowners:
  - '@vigonotion'
  - '@micha91'
ha_iot_class: Local Push
ha_ssdp: true
ha_config_flow: true
ha_domain: yamaha_musiccast
ha_platforms:
  - media_player
---

The Yamaha MusicCast integration allows you to control [Yamaha MusicCast Receivers](https://usa.yamaha.com/products/audio_visual/musiccast/index.html) from Home Assistant.

Supported devices are listed on their [site](https://usa.yamaha.com/products/contents/audio_visual/musiccast/musiccast-compatiblity.html).

{% include integrations/config_flow.md %}

## Grouping functionality

The Yamaha MusicCast integration implements the grouping services. There are some limitations in the MusicCast system for grouping:

- It is not possible to let mediaplayer entities of the same device (e.g., different zones) be in distinct groups.
- If a non-main zone is the master of a group, it is not possible to let other mediaplayers of the same device join this group.

## Play Media functionality

The MusicCast integration supports the Home Assistant media browser for all streaming services, your device supports. For services such as Deezer, you have to log in using the official MusicCast app. In addition, local HTTP URLs can be played back using this service. This includes the Home Assistant text to text services.

It is also possible to recall NetUSB presets using the play media service. To do so "presets:<preset_num>" has to be used as `media_content_id` in the service call.

### Examples:

This is an example service call that plays an audio file from a web server on the local network (like the Home Assistant built-in webserver):

```yaml
service: media_player.play_media
target:
  entity_id: media_player.sonos
data:
  media_content_type: "music"
  media_content_id: "http://192.168.188.18:8123/local/sound_files/doorbell-front.mp3"
```

This example call shows how to call netusb preset 1:

```yaml
service: media_player.play_media
target:
  entity_id: media_player.buro
data:
  media_content_id: "presets:1"
  media_content_type: "music"
```
## Troubleshooting

In this section known problems and their resolution are documented.

### Errors on handling UDP messages

The Yamaha MusicCast integration is working with updates sent from the device to Home Assistant instead of pulling all information every few seconds. There is no error correction mechanism in these messages, so that only 100% correct messages can be processed. Whenever there is a corrupt message the Yamaha MusicCast integration will update all device information and log an error message like these:
* `Received invalid message: <message>`
* `Received non UTF-8 compliant message: b'<binary>'`

If you receive these errors frequently, you should first try to disconnect your MusicCast devices from the power, wait 30 seconds and reconnect them to the power. If this does not help, you can try to use a LAN cable instead of WiFi to connect the device to the network.
