# TV05-themostat-radiator-valve
Integrate the smart thermostats TV05 into your heating installation.

*** under construction ***

Issue to solve
the cheap smart thermostats are intended to only heat a room when there is a heat demand. Unfortunately, these thermostats do not have the option to transmit the heat demand to the heating system. As a result, the thermostat in the living room must always have a heat demand in order to heat the rest of the house.
The smart thermostats that are currently popular do not have the option to switch on the heat demand. In this project I converted the heat demand into a simple and cheap bridge for home assistant.
Together with some automation in Home Assistant, each thermostat switches on the heating system when there is a heat demand. Because the heat demand bridge is connected in parallel to the existing thermostat, a fallback in the event of Home Assistant failure is arranged.

Thermostat TV05 

Automation architecture 

The heat demand bridge 

Automation 

Script 
