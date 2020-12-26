# openhab2
This is my openHAB configuration, fell free to use it to get started with your own :-)

Licensed under the [WTFPL](http://www.wtfpl.net)

Implemented hardware/software
-----
The following has been implemented and partly added to the overall description:
- Homematic devices (wireless) via [RaspberryMatic](https://github.com/jens-maus/RaspberryMatic)
	- radiator valves
        - wall thermostats
- MQTT devices
	- Vaillant boiler/control through [eBusd](https://github.com/john30/ebusd)
	- Zigbee devices via [Zigbee2MQTT](https://www.zigbee2mqtt.io)
- presence detection
	- network pings to phones/computers
- weather (wip)
- data persistence via [influxDB](https://www.influxdata.com) and [grafana](https://grafana.com)

