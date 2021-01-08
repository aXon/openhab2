openHAB2 installation
===============

The easiest installation for openHAB2 is through [openhabian](https://github.com/openhab/openhabian/releases). The image needs to be downloaded and then transferred to an SD card e.g. via [etcher](etcher.io).

Once booted up, the installation status can be seen when logging in through SSH, the user/pw default is openhabian.

It will take a few minutes before the reboot and then the usual

    sudo apt-get update
    sudo apt-get upgrade
followed by a

    sudo openhabian-config
In order to select additional software/settings, go to local/timezone under `System Settings` or installing Log Viewer in `Optional Components`. Additionally an MQTT server should be installed, if sensors with that capability are needed, or e.g. ebusd with MQTT support is interfaced.

Some of the homematic/heating modes is inspired by [Thom Dietrich's github repo](https://github.com/ThomDietrich/openhab-config).

[eBus](http://eBus-wiki.org/doku.php)
-----

Our house has a Vaillant Combi Boiler for hot water and heating installed, and can be integrated with openHAB well, using the [ebusd](https://github.com/john30/ebusd). To make use eBus, I acquired an [eBus board](https://eBus.github.io/adapter/) and installed ebusd (Taken from the ebusd git page):

Add the GPG key to your trusted apt sources (usually root access required) i.e. sudo su and then:

`wget -O - https://raw.githubusercontent.com/john30/ebusd-debian/master/ebusd.gpg.key|apt-key add -`

Copy the right source list for your architecture (check by executing `dpkg --print-architecture`) to /etc/apt/sources.list.d/ (replace ARCH with either amd64, armhf or i386):
`wget -O /etc/apt/sources.list.d/ebusd.list https://raw.githubusercontent.com/john30/ebusd-debian/master/ebusd-ARCH-default.list`

After that, you can simply update the package lists via apt-get update(depending on your distribution) and then install with apt-get install ebusd.

I have the eBus board version 2.2 from October 2018 and am using the CP2102 MICR0 to communicate with it through UART, rather than the Wemos D1 option in order to circumvent problems with WiFi and also to run the ebusd on the same system as openHAB. Another newer version also runs straight off the Rspberry Pi's HAT, however needs a modified driver to work, thus I chose the least cumbersome method to interface with the eBus.

The latest ebusd version, it can automatically download required files to translate the eBus messages to a human-readable format. 
The configuration for ebusd is in /etc/default/ebusd and needs to include the following to work with openhab/MQTT

	EBUSD_OPTS="--logfile=/var/log/ebusd.log --scanconfig --port=8888 --mqtthost=127.0.0.1  --mqttport=1883"
Once set up, ebusd can be enabled as a service and started

	sudo systemctl enable ebusd
	sudo service ebusd start

The ebusd can be interrogated using ebusctl for read/write access as well as other information e.g.

	ebusctl info

reports on loaded CSV files, latency, etc:

	version: ebusd 3.2.v3.2
	update check: revision v3.2-12-g45b9bad available, broadcast.csv: different version available, vaillant/bai.308523.inc: different version available, vaillant/broadcast.csv: different version available, vaillant/errors.inc: different version available, vaillant/hcmode.inc: different version available
	signal: acquired
	symbol rate: 58
	max symbol rate: 113
	min arbitration micros: 2076
	max arbitration micros: 3372
	min symbol latency: 4
	max symbol latency: 6
	reconnects: 0
	masters: 3
	messages: 555
	conditional: 3
	poll: 0
	update: 9
	address 03: master #11
	address 08: slave #11, scanned "MF=Vaillant;ID=BAI00;SW=0603;HW=9102", loaded "vaillant/bai.308523.inc", "vaillant/08.bai.csv"
	address 10: master #2
	address 15: slave #2, scanned "MF=Vaillant;ID=B7V00;SW=0422;HW=5503", loaded "vaillant/15.b7v00.csv"
	address 31: master #8, ebusd
	address 36: slave #8, ebusd

E.g. to read the outside temperature, one can issue the following command - this will also send the MQTT message out

	ebusctl read OutsideTemp

To see all available registers that could be interrogated, one can use

	ebusctl find

to see a list of all the data, that is available (not all are read/write) trough the currently connected eBus.
A handy script to read all or selective data can be found at [the ebusd github page](https://github.com/john30/ebusd/tree/master/contrib/scripts)


MQTT and openHAB binding
------

openHAB does not itself provide a broker for MQTT messages thus another software package can be installed through openhabian-config: mosquitto, which is found within Additional Components.

During the installation, one can add user/pass settings for the service, or leave it unsecured. By default, the service will listen to port 1883 (TCP). The seetings for the service can be found in /etc/mosquitto and amended in the conf.d directory.
Once can test the connection to the broker and see if it is working with following commands to subscribe and post to a specific topic in two terminals.
Terminal 1:

	mosquitto_sub -d -t hello/world

Terminal 2:

  mosquitto_pub -d -t hello/world -m "Hello from Terminal window 2!"

If the broker is working, Terminal 1 should output the message that was sent. Other tools to investigate MQTT e.g. MQTTLens for Chrome can be used to monitor topics.

Zigbee2MQTT
-------

Adding Zigbee (2.4 GHz) devices can be done by using a concentrator stick [from Electrolama](https://electrolama.com/projects/zig-a-zig-ah/#flashing-using-bsl)
and then paired devices can be communicated with through MQTT within openHab.

The install steps/requirements are:
 - having mosquitto setup in generic mode i.e. without restrictions
 - getting zigbee2mqtt installed, [following the instructions on their website](https://www.zigbee2mqtt.io), which can then be found in `/opt/zigbee2mqtt` and started with `npm start` within that folder and the configuration file in `/opt/zigbee2mqtt/data/configuration.yaml`
 - additionally to the initial settings provided by the zigbee2mqtt installation, one has to add the following to the `configuration.yaml`:

  advanced:
      rtscts: false

 - adding a Lidl LED bulb seems to result in a [compatible device being recognised](https://www.zigbee2mqtt.io/devices/TS0505A.html), however network coverage is bad
 - more on the integration with openHab can be found in [this forum thread](https://community.openhab.org/t/openhab-and-zigbee2mqtt-tutorial-for-beginners/83446)

Playing with some settigns via the commandline can be done through mosquitto_pub

  mosquitto_pub -t zigbee2mqtt/bulb/set -m '{"brightness": "100"}'

and mosquitto_sub can be used to listen on the events

  mosquitto_sub -d -t zigbee2mqtt/#

Parsistence with influxdb and grafana
----------

The ideas is to use influxdb (time-driven-series database) and grafana (visualisation) on the FreeNAS/TureNAS server and send the data over from the openhab instance via local network. Loosely based on [this openhab thread](https://community.openhab.org/t/influxdb-grafana-persistence-and-graphing/13761)

Installation steps:
 - Create a new base jail in TrueNAS and then
	
	pkg install influxdb grafana7

 - this will install influxdb 1.8 and grafana 7.3.4 in Dec-2020. The good thing about this is, that the location for the databases can be configured easily within `/usr/local/etc/influxd.conf`. The FreeBSD package also sets up the user influxd
 - Stop the jail and then set up a mount point for /var/db/influxdb to match a safe space on the Radiz2. Also ensure it is either matching with a user on TrueNAS or make it available to all
 - add  `sysrc influxd_enable=yes` and use `service influxd onestart` to see it if works i.e. within `/var/db/influxdb` should be some new folders. Also enable grafana with `sysrc grafana_enable=yes`
 - stop influxdb and ensure the following is enabled/set in `/usr/local/etc/influxd.conf`
	
	[http]

		enabled = true 
		bind-address = ":8086"

 and make sure the service is restarted
 - then create the databases and users

	$ influx
	Connected to http://localhost:8086 version 0.13
	InfluxDB shell version: 0.13
	> CREATE DATABASE openhab_db
	> CREATE USER admin WITH PASSWORD 'SuperSecretPassword123+' WITH ALL PRIVILEGES
	> CREATE USER openhab WITH PASSWORD 'AnotherSuperbPassword456-'
	> CREATE USER grafana WITH PASSWORD 'PleaseLetMeRead789?'
	> GRANT ALL ON openhab_db TO openhab
	> GRANT READ ON openhab_db TO grafana
	> exit

then disable password-less auth in  `/usr/local/etc/influxd.conf`

	[http]

		enabled = true 
		bind-address = ":8086"
		auth-enabled = true 


Now in order to be able to write data to this database, openHAb needs to be configured:
 - first of all, add `influxdb` to the openhab2/addongs.cfg persistence settings
 - Next, add your InfluxDB connection details to the persistence service config file /etc/openhab2/services/influxdb.cfg:

	# The database URL, e.g. http://127.0.0.1:8086 or https://127.0.0.1:8084 .
	# Defaults to: http://127.0.0.1:8086
	url=http://192.168.0.2:8086

	# The name of the database user, e.g. openhab.
	# Defaults to: openhab
	user=openhab

	# The password of the database user.
	password=AnotherSuperbPassword456-

	# The name of the database, e.g. openhab.
	# Defaults to: openhab
	db=openhab_db

	# The retention policy to be used, needs to configured in InfluxDB
	retentionPolicy=autogen


Measuring air quality (luftdaten + MQTT)
----------
Due to the pandemic, I have become interested in measuring air quality personally, as well as liked the idea in contributing to a citizen science project, which [luftdaten.info](https://luftdaten.info) is!
In order to get things up and running quickly, I went with an Enviro+ from Pimoroni, which sends its PM2.5 and PM10 data to luftdaten.info as well, when registered, so I followed this [tutorial](https://learn.pimoroni.com/tutorial/sandyj/enviro-plus-and-luftdaten-air-quality-station) to get the board up and running.

Now the original scripts only allow single use case scenarios i.e. either sending data luftdaten or just via MQTT and running both in parallels does not work due to a clash in access to the various sensors through their respective bus. I wanted to do both and simply inforporated both into a single file. It is not optimised, but does the job as I want to collect the PM data over time through influx and display it with grafana for later analysis.

The raw data looks like this:
	
	{"temperature": 7, "pressure": 100970, "humidity": 40, "oxidised": 9, "reduced": 592, "nh3": 677, "lux": 0, "pm1": 25, "pm25": 40, "pm10": 42, "serial": "00000000xyzxyzxyz"}

As the RPI/enviro+ board  and PM sensors are stuck inside two 90º bent plastic pipes to cover these from rain, the lux is 0. The temperature value seems off, and I could calibrate it in the future. The same goes for the gas values, which are only outputting absolute resistance values, which, without calibration, do not mean much for the time being. I will try and collect them anyways.