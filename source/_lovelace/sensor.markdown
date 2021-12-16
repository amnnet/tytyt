---
type: card
title: "Sensor Card"
sidebar_label: Sensor
description: "The Sensor card gives you a quick overview of your sensors state with an optional graph to visualize change over time."
---

The Sensor card gives you a quick overview of your sensors state with an optional graph to visualize change over time.

<p class='img'>
  <img src='/images/lovelace/lovelace_sensor.png' alt='Screenshot of the sensor card'>
  Screenshot of the sensor card.
</p>

To add the Sensor card to your user interface, click the Lovelace menu (three dots at the top right of the screen) and then **Edit Dashboard**. Click the "Add Card" button in the bottom right corner and select **Sensor** from the card picker. All options for this card can be configured via the user interface.

{% configuration %}
type:
  required: true
  description: "`sensor`"
  type: string
entity:
  required: true
  description: Entity ID of `sensor` domain.
  type: string
icon:
  required: false
  description: The card icon.
  type: string
name:
  required: false
  description: The card name.
  type: string
graph:
  required: false
  description: Type of graph (`none` or `line`).
  type: string
unit:
  required: false
  description: The unit of measurement.
  type: string
detail:
  required: false
  description: Detail of the graph `1` or `2` (`1` = one point/hour, `2` = six points/hour).
  type: integer
  default: 1
hours_to_show:
  required: false
  description: Hours to show in graph. Minimum is 1 hour. Big values can result in delayed rendering, especially if the selected entities have a lot of state changes.
  type: integer
  default: 24
limits:
  required: false
  description: Limits of the graph Y-axis.
  type: map
  keys:
    min:
      description: Minimum value of the graph Y-axis.
      type: float
      required: false
      default: "`0` if the entity `unit_of_measurement` is `%`, otherwise minimum value of sample"
    max:
      description: Maximum value of the graph Y-axis.
      type: float
      required: false
      default: "`100` if the entity `unit_of_measurement` is `%`, otherwise maximum value of sample"
theme:
  required: false
  description: Override the used theme for this card with any loaded theme. For more information about themes, see the [frontend documentation](/integrations/frontend/).
  type: string
{% endconfiguration %}

## Example

Alternatively, the card can be configured using YAML:

```yaml
type: sensor
entity: sensor.illumination
name: Illumination
```
