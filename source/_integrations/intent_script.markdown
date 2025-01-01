---
title: Intent Script
description: Instructions on how to setup scripts to run on intents.
ha_category:
  - Intent
ha_release: '0.50'
ha_quality_scale: internal
ha_domain: intent_script
ha_integration_type: integration
---

The `intent_script` integration allows users to configure actions and responses to intents. Intents can be fired by any integration that supports it. Examples are [Alexa](/integrations/alexa/) (Amazon Echo), [Dialogflow](/integrations/dialogflow/) (Google Assistant) and [Snips](/integrations/snips/).

If you are using intent script with LLMs and have parameters, make sure to mention the parameters and their types in the description.

{% raw %}

```yaml
# Example configuration.yaml entry
intent_script:
  GetTemperature:  # Intent type
    description: Return the temperature and notify the user
    speech:
      text: We have {{ states('sensor.temperature') }} degrees
    action:
      action: notify.notify
      data:
        message: Hello from an intent!
```

{% endraw %}

Inside an intent we can define these variables:

{% configuration %}
intent:
  description: Name of the intent. Multiple entries are possible.
  required: true
  type: map
  keys:
    description:
      description: Description of the intent.
      required: false
      type: string
    platforms:
      description: List of domains that the entity supports.
      required: false
      type: list
    action:
      description: Defines an action to run to intents.
      required: false
      type: action
    async_action:
      description: Set to True to have Home Assistant not wait for the script to finish before returning the intent response.
      required: false
      default: false
      type: boolean
    mode:
      description: The [script mode](https://www.home-assistant.io/integrations/script/#script-modes) in which to run the intent script. Use this to define if the intent should be able to run multiple times in parallel.
      required: false
      default: single
      type: string
    card:
      description: Card to display.
      required: false
      type: map
      keys:
        type:
          description: Type of card to display.
          required: false
          default: simple
          type: string
        title:
          description: Title of the card to display.
          required: true
          type: template
        content:
          description: Contents of the card to display.
          required: true
          type: template
    speech:
      description: Text or template to return.
      required: false
      type: map
      keys:
        type:
          description: Type of speech.
          required: false
          default: plain
          type: string
        text:
          description: Text to speech.
          required: true
          type: template
{% endconfiguration %}

## Using the action response

When using a `speech` template, data returned from the executed action are
available in the `action_response` variable.

{% raw %}

```yaml
conversation:
  intents:
    EventCountToday:
      - "How many meetings do I have today?"

intent_script:
  EventCountToday:
    action:
      - action: calendar.get_events
        target:
          entity_id: calendar.my_calendar
        data_template:
          start_date_time: "{{ today_at('00:00') }}"
          duration: { "hours": 24 }
        response_variable: result                     # get action response
      - stop: ""
        response_variable: result                     # and return it
    speech:
      text: "{{ action_response['calendar.my_calendar'].events | length }}"   # use the action's response
```
  DoorsOpen:
    speech:
      text: >
        {% set doors = state_attr('group.all_doors', 'entity_id') %}
        {% set allDoorsClosed = expand(doors) | selectattr('state','in',['tripped']) 
          | list | map(attribute='name') | join(', ') %}

          {% if allDoorsClosed %}
          The following doors are open: {{ allDoorsClosed | default('All Doors are Closed', 1) }}
        {% else %}
          All doors are closed
        {% endif %}
{% endraw %}


  FanPercentOn:
    speech:
      text: "{{ state_attr(fanname, 'friendly_name') }} is set to {{ percent }} percent"       
    action:
      - action: fan.set_percentage
        data:
          percentage: "{{ percent }}"
        target:
          entity_id: "{{ fanname }}"


  HvacStatus:
    speech:
      text: >
         "The HVAC status is {{ states('text.hvac_system_status') }} and the 
         mode is set to {{ states('select.hvac_mode_selection') }}
         {% set allrooms = state_attr('group.all_climate_rooms', 'entity_id') %}
         {% set HVACRooms = expand(allrooms) | selectattr('attributes.hvac_action','in',['heating','cooling']) 
          | list | map(attribute='name') | join(', ') %}

          {% if HVACRooms %}
           The following dampers are open: {{ HVACRooms  | default('None', 1) }}
          {% endif %}          


