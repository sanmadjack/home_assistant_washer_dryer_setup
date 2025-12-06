# Home Assistant Washer & Dryer alerting setup

This configuration does three things:

1. Notifies when the clothes washer is done.
2. Notifies when the clothes dryer is done.
3. Periodically nags if the laundry has not yet been moved to the dryer. In the case of laundry that doesn't need to go in the dryer, or for whatever other reason, the alert includes a button to stop the nags.

## How to setup:

Buy a device that can monitor the power usage of an outlet. I used Sonoff S31s, and these instructions will assume the same.

Set up an MQTT broker that Home Assistant supports, I used Mosquito via HA addons. This doc will not cover Mosquito setup.

Flash the Sonoff S31s with Tasmota, and configure them to connect to the MQTT broker. This doc will not cover flashing or configuring the Tasmotas.

Create two sensor templates using the code in sensor_templates.yaml. This can go in configuration.yaml, or you can put it in a seperate file and then include it in configuration.yaml. In my setup "sensor.washer_plug_energy_power" and "sensor.dryer_plug_energy_power" are the names of the entities that provide the current power information, update them to match whatever your entities are called.

These templates interpret the current power from the S31s and provide a simple "true" or "false" if the power is currently above a certain threshold. For my washer that's 4, and my dryer is 20. You will need to experiment with your equipment to find appropriate values.

Create the helper input_boolean.washer_finished_dryer_not_yet_run. I used icon mdi:washing-machine-alert, you can use whatever you want.

Create these four automations from the associated yaml files:

### Clothes Washer Idle
This waits for the washer power usage to drop below the amount set in the sensor template for 5 minutes. My washer sometimes likes to dip its power use, and this is the buffer I arrived at to prevent it preemptively sending notifications, experiment and find the right values for you.

When the condition is met, it sends a notification to our phones that the washing machine is done, and then sends that same message to our voice assistants throughout the house. Customize these as you need.

The last step is it sets the input_boolean.washer_finished_dryer_not_yet_run to "true". This sets the timer in motion to start nagging about moving the laundry to the dryer. If you don't want the nagging, delete this action.

### Clothes Dryer Idle

This waits for the dryer power usage to drop below the amount set in the sensor template for 5 minutes. My dryer sometimes likes to dip its power use, and this is the buffer I arrived at to prevent it preemptively sending notifications, experiment and find the right values for you.

When the condition is met, it sends a notification to our phones that the dryer is done, and then sends that same message to our voice assistants throughout the house. Customize these as you need.

### Clothes Dryer Active
This script is only necessary for the switching nags.

This waits for the dryer state to flip to true for 10 minutes. The dryer also likes to dip it's power us, and this is the buffer I landed on. Once again, experiment to find the right values for your equipment.

When the condition is met, it sets the input_boolean.washer_finished_dryer_not_yet_run to "false". This ceases the switching nagging alerts

### ACTION - CEASE_WASHER_NAG
This script is only necessary for the switching nags.

This script listens for the action from the nag alert that tells Home Assistant to stop sending the nags. When triggered, it sets the input_boolean.washer_finished_dryer_not_yet_run to "false". This ceases the switching nagging alerts
