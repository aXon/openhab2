# openhab2
This is my openHAB configuration, fell free to use it to get started with your own :-)

Licensed under the [WTFPL](http://www.wtfpl.net)

Implemented hardware/software
-----
The following has been implemented and partly added to the overall description:
- Homematic devices (wireless) via [RaspberryMatic](https://github.com/jens-maus/RaspberryMatic)
	- radiator valves (HM-CC-RT-DN)
	- wall thermostats (HM-TC-IT-WM-W-EU)
- MQTT devices
	- Vaillant boiler/control through [eBusd](https://github.com/john30/ebusd) using an adapter from the [original eBusd developer](https://ebusd.eu)
	- Zigbee devices via [Zigbee2MQTT](https://www.zigbee2mqtt.io) using a [zig-a-zig-ah! (CC2652 Stick)](https://electrolama.com/projects/zig-a-zig-ah/)
	- [Sonoff](https://sonoff.tech) devices running [Tasmota](https://tasmota.github.io/docs/) to control (some) lighting
	- Power monitoring with a Raspberry Pi and a board from [David00](https://github.com/David00/rpi-power-monitor). To integrate it with openhab,  and MQTT capability was added to the scripts that usually output to InfluxDB only.
	- TTN IoT devices can be integrated using [MQTT](https://www.thethingsnetwork.org/docs/applications/mqtt/quick-start/index.html) as well, having to subscribe to their MQTT server with the Application ID and Key for authentication.
	- Controlling Lidl Livarno RGB lights with Aqara cubes
- presence detection
	- network pings to phones/computers
- weather/environment
	- using an [enviro+](https://shop.pimoroni.com/products/enviro) to share PM2.5 and PM10 readings with [sensor.community](https://sensor.community) as well as collecting it locally via MQTT (temperature readings uncalibrated and NOx gas detection unused)
- data persistence via [influxDB](https://www.influxdata.com) and then using  [grafana](https://grafana.com) for data exploration/visualisation

Todo list:
- Add [owntracks](https://owntracks.org/booklet/) through a private MQTT server for presence and automatic heating switches
- Enhance heating mode by automatically switching the ebus controller heating mode depending on season and outside temperature
- make presence work with heating modes

NB: this is aimed at openHAB 2.5.x at the moment.
