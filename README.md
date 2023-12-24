# TV05-themostat-radiator-valve
Integrate the smart thermostats TV05 into your heating installation.

*** under construction ***

Issue to solve
the cheap smart thermostats are intended to only heat a room when there is a heat demand. Unfortunately, these thermostats do not have the option to transmit the heat demand to the heating system. As a result, the thermostat in the living room must always have a heat demand in order to heat the rest of the house.
The smart thermostats that are currently popular do not have the option to switch on the heat demand. In this project I converted the heat demand into a simple and cheap bridge for home assistant.
Together with some automation in Home Assistant, each thermostat switches on the heating system when there is a heat demand. Because the heat demand bridge is connected in parallel to the existing thermostat, a fallback in the event of Home Assistant failure is arranged.

Thermostat TV05 Zigbee gateway via Tuya(SmartLife)<BR>
<img src="https://github.com/ltvanderkrogt/TV05-themostat-radiator-valve/blob/c7f54a1b874992f7f603850454003c33be9d03da/img/TV05%20set.png" width=50% height=50%><BR>
<BR>
Reset thermstat  <BR>
<img src="https://github.com/ltvanderkrogt/TV05-themostat-radiator-valve/blob/1f28e88d8415bf406c4c3055466fb719f4e4e0e6/img/Reset.jpeg" width=50% height=50%><BR>
Beware: reset only works if display is on! <BR>
<BR> 

Error codes <BR> 
<img src="https://github.com/ltvanderkrogt/TV05-themostat-radiator-valve/blob/1f28e88d8415bf406c4c3055466fb719f4e4e0e6/img/Error_Codes.jpeg" width=50% height=50%><BR>
<BR> 


Automation architecture <BR>

<img src="https://github.com/ltvanderkrogt/TV05-themostat-radiator-valve/blob/f431a33d6fe4b762c81e91892cad0b628bcf2a17/img/Pelikaan.png" width=90% height=90%><BR>
<BR>
The heat demand bridge <BR>
<img src="https://github.com/ltvanderkrogt/TV05-themostat-radiator-valve/blob/main/img/Relay%20module%20ESP8266.png" width=25% height=25%><BR>

Script ESPHome yaml<br>

```yaml
esphome:
  name: cv-warmtevraag
  friendly_name: cv-warmtevraag

esp8266:
  board: esp01_1m

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "********************************************"

ota:
  password: "*********************************"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Cv-Warmtevraag Fallback Hotspot"
    password: "***********"

captive_portal:
  
output:
  - platform: esp8266_pwm
    pin: GPIO2
    id: onboard_led

light:
  - platform: monochromatic
    name: "Onboard LED"
    output: onboard_led

switch:
  - platform: gpio
    name: "Relay"
    pin: GPIO0
```
<BR>
Automation yaml for each thermostat<BR>

Trigger: This automation is triggered by a state change in the entity climate.hal. It means that the automation will be executed whenever the state of the specified thermostat (climate.hal) changes.

Actions
Execute Script:

Invokes script heat demand

Conditional Actions: for test only

The script includes a conditional block (if) that checks whether the current temperature of the thermostat (climate.hal) is less than the target temperature.
If the condition is true:
Sends a notification to a mobile app stating that the thermostat has a warmth demand. It includes information about the current and target temperatures.
If the condition is false:
Sends a notification to a mobile app stating that the thermostat does not have a warmth demand. It includes information about the current and target temperatures.
Enabled:

The enabled: false statement indicates that this automation is currently disabled. If you want it to be active, you can set enabled: true.<BR>
```yaml
alias: Thermostaat hal temp verandering
description: Thermostaat hal temp verandering
trigger:
  - platform: state
    entity_id:
      - climate.hal
condition: []
action:
  - service: script.1702503879842
    data: {}
  - if:
      - condition: template
        value_template: >-
          {{ state_attr('climate.hal', 'current_temperature') <
          state_attr('climate.hal', 'temperature') }}
    then:
      - service: notify.mobile_app_iphone_rwn1w3gf6k
        data:
          message: >-
            Thermostaat hal warmtevraag aan. Nu 
            {{state_attr('climate.hal','current_temperature') }} ºC ingesteld op
            {{    state_attr('climate.hal','temperature')}} ºC.
    else:
      - service: notify.mobile_app_iphone_rwn1w3gf6k
        data:
          message: >-
            Thermostaat hal warmtevraag uit. Nu 
            {{state_attr('climate.hal','current_temperature') }} ºC ingesteld op
            {{    state_attr('climate.hal','temperature')}} ºC.
    enabled: false
mode: single
```

Script heat demand <BR>

No Warmth Demand (First Branch):

Conditions:
An AND condition is used to check if all the specified thermostats have their current temperature greater than or equal to the target temperature.
Sequence:
For test only: Sends a notification to a mobile app, stating that thermostats have no warmth demand.
Turns OFF the heat demand bridge switch

Warmth Demand Detected (Second Branch):

Conditions:
An OR condition is used to check if any of the specified thermostats have their current temperature less than the target temperature.
Sequence:
For test only: Sends a notification to a mobile app, stating the number of thermostats that have a warmth demand. The count is based on the number of thermostats with the current temperature less than the target temperature.
Turns ON the heat demand bridge switch

```yaml
alias: Warmtevraag
sequence:
  - choose:
      - conditions:
          - condition: and
            conditions:
              - condition: template
                value_template: >-
                  {{ state_attr('climate.hal', 'current_temperature') >=
                  state_attr('climate.hal', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.vincent', 'current_temperature') >=
                  state_attr('climate.vincent', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.thermostaat_voorkamer',
                  'current_temperature') >=
                  state_attr('climate.thermostaat_voorkamer', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.julia', 'current_temperature') >=
                  state_attr('climate.julia', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.werkkamer', 'current_temperature') >=
                  state_attr('climate.werkkamer', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.thermostaat_haardkamer',
                  'current_temperature') >=
                  state_attr('climate.thermostaat_haardkamer', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.logeer', 'current_temperature') >=
                  state_attr('climate.logeer', 'temperature') }}
        sequence:
          - service: notify.mobile_app_iphone_rwn1w3gf6k
            data:
              message: Thermostaten hebben geen warmtevraag.
            enabled: false
          - type: turn_off
            device_id: 35f12613a862ddb2f36de40e6771fdd1
            entity_id: af4924c232cc8f702d298610b3a05315
            domain: switch
      - conditions:
          - condition: or
            conditions:
              - condition: template
                value_template: >-
                  {{ state_attr('climate.hal', 'current_temperature') <
                  state_attr('climate.hal', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.vincent', 'current_temperature') <
                  state_attr('climate.vincent', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.thermostaat_voorkamer',
                  'current_temperature') <
                  state_attr('climate.thermostaat_voorkamer', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.thermostaat_haardkamer',
                  'current_temperature') <
                  state_attr('climate.thermostaat_haardkamer', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.julia', 'current_temperature') <
                  state_attr('climate.julia', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.werkkamer', 'current_temperature') <
                  state_attr('climate.werkkamer', 'temperature') }}
              - condition: template
                value_template: >-
                  {{ state_attr('climate.logeer', 'current_temperature') <
                  state_attr('climate.logeer', 'temperature') }}
        sequence:
          - service: notify.mobile_app_iphone_rwn1w3gf6k
            data:
              message: >
                {%- set active_thermostats = namespace(count=0) -%} {%- for
                entity_id in [
                  'climate.hal', 'climate.vincent', 'climate.thermostaat_voorkamer',
                  'climate.thermostaat_haardkamer', 'climate.julia', 'climate.logeer', 'climate.werkkamer'
                ] -%}
                  {%- set current_temp = state_attr(entity_id, 'current_temperature') -%}
                  {%- set target_temp = state_attr(entity_id, 'temperature') -%}
                  {%- set thermostat_status = 'ON' if current_temp < target_temp else 'OFF' -%}
                  {%- if thermostat_status == 'ON' -%}
                    {%- set active_thermostats.count = active_thermostats.count + 1 -%}
                  {%- endif -%}

                {%- endfor -%} Ten minste {{ active_thermostats.count }}
                thermostaten hebben warmtevraag.
            enabled: false
          - type: turn_on
            device_id: 35f12613a862ddb2f36de40e6771fdd1
            entity_id: af4924c232cc8f702d298610b3a05315
            domain: switch
mode: single
```

Issues<BR> 
Holiday temperature: periode must be set! I set the periode to 10 years. <BR>
Reset at the knob only works if display is on! <BR> 
Don't forget to change the default schedule. <BR> 

