---
title: Google Translate Text-to-Speech
description: Instructions on how to setup Google Translate Text-to-Speech with Home Assistant.
ha_category:
  - Text-to-speech
ha_release: 0.35
ha_iot_class: Cloud Push
ha_domain: google_translate
ha_platforms:
  - tts
---

The `google_translate` text-to-speech platform uses the unofficial [Google Translate Text-to-Speech engine](https://translate.google.com/) to read a text with natural sounding voices. Contrary to what the name suggests, the integration only does text-to-speech and does not translate messages sent to it.

## Configuration

To enable text-to-speech with Google, add the following lines to your `configuration.yaml`:

```yaml
# Example configuration.yaml entry
tts:
  - platform: google_translate
```

{% configuration %}
language:
  description: "The default speech language to use."
  required: false
  type: string
  default: "`en`"
{% endconfiguration %}

Check the [complete list of supported languages](https://translate.google.com/intl/en_ALL/about/languages/) (languages where "Talk" feature is enabled in Google Translate) for allowed values.
Use the 2 digit language code which you can find at the end of URL when you click on Language name.

For more information about using text-to-speech with Home Assistant and more details on all the options it provides, see the [TTS documentation](/integrations/tts/).

## Full configuration example

A full configuration sample including optional variables:

```yaml
# Example configuration.yaml entry
tts:
  - platform: google_translate
    language: "de"
```
