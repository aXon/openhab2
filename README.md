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
- presence detection
	- network pings to phones/computers
- weather/environment
	- using an [enviro+](https://shop.pimoroni.com/products/enviro) to share PM2.5 and PM10 readings with [sensor.community](https://sensor.community) as well as collecting it locally via MQTT (temperature readings uncalibrated and NOx gas detection unused)
- data persistence via [influxDB](https://www.influxdata.com) and then using  [grafana](https://grafana.com) for data exploration/visualisation

Todo list:
- Add [owntracks](https://owntracks.org/booklet/) through a private MQTT server for presence and automatic heating switches
- Enhance heating mode by automatically switching the ebus controller heating mode depending on season and outside temperature
- make presence work with heating modes
