---
title: Sonos
description: Instructions on how to integrate Sonos devices into Home Assistant.
ha_category:
  - Media Player
  - Sensor
featured: true
ha_release: 0.7.3
ha_iot_class: Local Push
ha_config_flow: true
ha_domain: sonos
ha_codeowners:
  - '@cgtobi'
  - '@jjlawren'
ha_ssdp: true
ha_platforms:
  - binary_sensor
  - media_player
  - sensor
  - switch
ha_zeroconf: true
---

The `sonos` integration allows you to control your [Sonos](https://www.sonos.com) wireless speakers from Home Assistant. It also works with IKEA Symfonisk speakers.

{% include integrations/config_flow.md %}

## Feature controls

Speaker-level features are exposed as `switch` entities which allow direct control and indicate if the features are currently enabled.

- **All devices**: Crossfade, Status Light, Touch Controls
- **Home theater devices**: Night Sound & Speech Enhancement

## Battery support

Battery sensors are fully supported for the `Sonos Roam` and `Sonos Move` devices on S2 firmware. `Sonos Move` speakers still on S1 firmware are supported but may update infrequently.

For each speaker with a battery, a `sensor` showing the current battery charge level and a `binary_sensor` showing the power state of the speaker are created. The `binary_sensor` reports if the speaker is currently powered by an external source and its `power_source` attribute shows which specific source is providing the current power. This source attribute can be one of `BATTERY`, `SONOS_CHARGING_RING` if using wireless charging, or `USB_POWER` if charging via USB cable. Note that the Roam will report `SONOS_CHARGING_RING` even when using a generic Qi charger.

<div class='note'>

The battery sensors rely on working change events or updates will be delayed. S1 battery sensors **require** working events to report any data. See more details in [Advanced use](#advanced-use). 

</div>

## Alarm support

The Sonos integration adds one `switch` for each alarm set in the Sonos app. The alarm switches are detected, deleted and assigned automatically and come with several attributes that help to monitor Sonos alarms.

## Playing media

Sonos accepts a variety of `media_content_id` formats in the `media_player.play_media` service, but most commonly as URIs. For example, both Spotify and Tidal share links can be provided as-is. Playback of [music hosted on a Plex server](/integrations/plex#sonos-playback) is possible. Direct HTTP/HTTPS links to local or remote media files can also be used if the Sonos device can reach the URI directly, but specific media encoding support may vary.

Music services which require an account (e.g., Spotify) must first be configured using the Sonos app.

An optional `enqueue` argument can be added to the service call. If `true`, the media will be appended to the end of the playback queue. If not provided or `false` then the queue will be replaced.

### Examples:

This is an example service call that plays an audio file from a web server on the local network (like the Home Assistant built-in webserver):

```yaml
service: media_player.play_media
target:
  entity_id: media_player.sonos
data:
  media_content_type: "music"
  media_content_id: "http://192.168.1.50:8123/local/sound_files/doorbell-front.mp3"
```

Sonos can also play music or playlists from Spotify. Both Spotify URIs and URLs can be used directly. An example service call using a playlist URI:

```yaml
service: media_player.play_media
target:
  entity_id: media_player.sonos
data:
  media_content_type: "playlist"
  media_content_id: "spotify:playlist:abcdefghij0123456789XY"
  enqueue: true
```

An example service call using a Spotify URL:

```yaml
service: media_player.play_media
target:
  entity_id: media_player.sonos
data:
  media_content_type: "music"
  media_content_id: "https://open.spotify.com/album/abcdefghij0123456789YZ"
```

Run a [Plex Media Server](/integrations/plex#sonos-playback) in your home? The Sonos integration can work with that as well. This example plays music directly from your Plex server:

```yaml
service: media_player.play_media
target:
  entity_id: media_player.sonos
data:
  media_content_type: "music"
  media_content_id: 'plex://{ "library_name": "Music", "artist_name": "M83", "album_name": "Hurry Up, We're Dreaming" }'
```

## Services

The Sonos integration makes various custom services available.

### Service `sonos.snapshot`

Take a snapshot of what is currently playing on one or more speakers. This service, and the following one, are useful if you want to play a doorbell or notification sound and resume playback afterwards.

<div class='note'>

The queue is not snapshotted and must be left untouched until the restore. Using `media_player.play_media` is safe and can be used to play a notification sound, including [TTS](/integrations/tts/) announcements.

</div>

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | The speakers to snapshot. To target all Sonos devices, use `all`.
| `with_group` | yes | Should we also snapshot the group layout and the state of other speakers in the group, defaults to true.

### Service `sonos.restore`

Restore a previously taken snapshot of one or more speakers.

<div class='note'>

The playing queue is not snapshotted. Using `sonos.restore` on a speaker that has replaced its queue will restore the playing position, but in the new queue!

</div>

<div class='note'>
A cloud queue cannot be restarted. This includes queues started from within Spotify and queues controlled by Amazon Alexa.
</div>

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of `entity_id`s that should have their snapshot restored. To target all Sonos devices, use `all`.
| `with_group` | yes | Should we also restore the group layout and the state of other speakers in the group, defaults to true.

### Service `sonos.join`

Group players together under a single coordinator. This will make a new group or join to an existing group.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `master` | no | A single `entity_id` that will become/stay the coordinator speaker.
| `entity_id` | yes | String or list of `entity_id`s to join to the master.

### Service `sonos.unjoin`

Remove one or more speakers from their group of speakers.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of `entity_id`s to separate from their coordinator speaker.

### Service `sonos.set_sleep_timer`

Sets a timer that will turn off a speaker by tapering the volume down to 0 after a certain amount of time. Protip: If you set the sleep_time value to 0, then the speaker will immediately start tapering the volume down.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of `entity_id`s that will have their timers set.
| `sleep_time` | no | Integer number of seconds that the speaker should wait until it starts tapering. Cannot exceed 86399 (one day).

### Service `sonos.clear_sleep_timer`

Clear the sleep timer on a speaker, if one is set.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | no | String or list of `entity_id`s that will have their timers cleared. Must be a coordinator speaker.

### Service `sonos.update_alarm`

Update an existing Sonos alarm.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of `entity_id`s that will have their timers cleared. Must be a coordinator speaker.
| `alarm_id` | no | Integer that is used in Sonos to refer to your alarm.
| `time` | yes | Time to set the alarm.
| `volume` | yes | Float for volume level.
| `enabled` | yes | Boolean for whether or not to enable this alarm.
| `include_linked_zones` | yes | Boolean that defines if the alarm also plays on grouped players.

### Service `sonos.play_queue`

Starts playing the Sonos queue.

Force start playing the queue, allows switching from another stream (such as radio) to playing the queue.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of `entity_id`s that will start playing. It must be the coordinator if targeting a group.
| `queue_position` | yes | Position of the song in the queue to start playing from, starts at 0.

### Service `sonos.remove_from_queue`

Removes an item from the queue.
| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of `entity_id`s that will remove an item from the queue. It must be the coordinator if targeting a group.
| `queue_position` | yes | Position in the queue to remove.

{% raw %}

```yaml
# Example automation to remove just played song from queue
alias: "Remove last played song from queue"
id: Remove last played song from queue
trigger:
  - platform: state
    entity_id: media_player.kitchen
  - platform: state
    entity_id: media_player.bathroom
  - platform: state
    entity_id: media_player.move
condition:
  condition: and
  conditions:
    # Coordinator
    - condition: template
      value_template: >
        {{ state_attr( trigger.entity_id , 'sonos_group')[0] ==  trigger.entity_id }}
    # Going from queue to queue
    - condition: template
      value_template: >
        {{ 'queue_position' in trigger.from_state.attributes and 'queue_position' in trigger.to_state.attributes }}
    # Moving forward
    - condition: template
      value_template: >
        {{ trigger.from_state.attributes.queue_position < trigger.to_state.attributes.queue_position }}
action:
  - service: sonos.remove_from_queue
    target:
      entity_id: >
        {{ trigger.entity_id }}
    data:
      queue_position: >
        {{ trigger.from_state.attributes.queue_position }}
```

{% endraw %}

## Advanced use

For advanced uses, there are some manual configuration options available. These are usually only needed if you have a complex network setup where Home Assistant and Sonos are not on the same subnet.

You can disable auto-discovery by specifying the Sonos IP addresses:

```yaml
# Example configuration.yaml entry with manually specified Sonos IP addresses
sonos:
  media_player:
    hosts:
      - 192.0.2.25
      - 192.0.2.26
      - 192.0.2.27
```

If your Home Assistant instance has multiple IP addresses, you can enable the IP address that should be used for Sonos auto-discovery with the [Network](/integrations/network/) integration. This should only be necessary if the Sonos speakers are on a network segment not reachable from the default interface.

The Sonos speakers will attempt to connect back to Home Assistant to deliver change events. By default, Home Assistant will listen on port 1400 but will try the next 100 ports above 1400 if it is in use. You can change the IP address that Home Assistant advertises to Sonos speakers. This can help in NAT scenarios such as when _not_ using the Docker option `--net=host`:

```yaml
# Example configuration.yaml entry modifying the advertised host address
sonos:
  media_player:
    advertise_addr: 192.0.2.1
```
