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

[eBus](http://eBus-wiki.org/doku.php)
-----

Our house has a Vaillant Combi Boiler for hot water and heating installed, and can be integrated with openHAB well, using the [ebusd](https://github.com/john30/ebusd). To make use eBus, I acquired an [eBus board](https://eBus.github.io/adapter/) and installed ebusd (Taken from the ebusd git page):

Add the GPG key to your trusted apt sources (usually root access required) i.e. sudo su and then:

`wget -O - https://raw.githubusercontent.com/john30/ebusd-debian/master/ebusd.gpg.key|apt-key add -`

Copy the right source list for your architecture (check by executing `dpkg --print-architecture`) to /etc/apt/sources.list.d/ (replace ARCH with either amd64, armhf or i386):
`wget -O /etc/apt/sources.list.d/ebusd.list https://raw.githubusercontent.com/john30/ebusd-debian/master/ebusd-ARCH-default.list`

After that, you can simply update the package lists via apt-get update(depending on your distribution) and then install with apt-get install ebusd.

I have the eBus board version 2.2 from October 2018 and am using the CP2102 MICR0 to communicate with it through UART, rather than the Wemos D1 option in order to circumvent problems with WiFi and also to run the ebusd on the same system as openHAB. Another newer version also runs straight off the Rspberry Pi's HAT, however needs a modified driver to work, thus I chose the least cumbersome method to interface with the eBus.

The latest ebusd version, it can automatically download required files to translate the eBus messages to a human-readable format. However, I had to download the CSV files, as one of the devices on my eBus was not identifiable through the web downloads, as the device (VRC700f) was identified as 15.b7v00, rather than 15.700:

	[main error] unable to load scan config 15: no file from /etc/ebusd/vaillant with prefix 15. matches ID "b7v00", SW0422, HW5503
Issue the following to retrieve and use the config CSV files offline:

	cd /home/openhabian
	git clone https://github.com/john30/ebusd-configuration.git
	sudo mv /etc/ebusd /etc/ebusd-old
	sudo ln -s /home/openhabian/ebusd-configuration/ebusd-2.1.x/de/ /etc/ebusd

The configuration for ebusd is in /etc/default/ebusd and needs to include the following to detect CSV files locally

	EBUSD_OPTS="--logfile=/var/log/ebusd.log --configpath=/etc/ebusd/ --scanconfig --port=8888"
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

E.g. to read the outside temperature, one can issue the following command

	ebusctl read OutsideTemp

To see all available registers that could be interrogated, one can use

	ebusctl find

to see a list of all the data, that is available (not all are read/write) trough the currently connected eBus.

To enable the MQTT part of ebusd, the defaults start options can be changed to:

	EBUSD_OPTS="--logfile=/var/log/ebusd.log --configpath=/etc/ebusd/ --scanconfig --port=8888 --mqtthost=0.0.0.0 --mqttport=1883 --mqttjson --mqtttopic=ebus/%circuit/%name"

which sets up ebusd to send under the main topic ebus, connects to the local MQTT server on port 1883, and formats it all in JSON.

The level of acces can be further increased, by adding %field in the mqtttopic

	--mqtttopic=ebus/%circuit/%name/%field



eBus openHAB Binding
-----

To use ebusd with openHAB, the latest eBus binding from [csowada's forum thread](https://community.openhab.org/t/eBus-2-0-new-binding-release-candidate-3/33547) is needed. The file can be dropped into the openHAB-share/openHAB-addons folder, which should be shared as a SMB share by default (might need login from openhabian user).


MQTT and openHAB binding
------

openHAB did not itself provide a broker for MQTT messages thus another software package was installed through openhabian-config: mosquitto, which is found within Additional Components.

During the installation, one can add user/pass settings for the service, or leave it unsecured. By default, the service will listen to port 1883 (TCP). The seetings for the service can be found in /etc/mosquitto and amended in the conf.d directory.
Once can test the connection to the broker and see if it is working with following commands to subscribe and post to a specific topic in two terminals.
Terminal 1:

	mosquitto_sub -d -t hello/world

Terminal 2:

	mosquitto_pub -d -t hello/world -m "Hello from Terminal window 2!"

If the broker is working, Terminal 1 should output the message that was sent. Other tools to investigate MQTT e.g. MQTTLens for Chrome can be used to monitor topics.
For simple debugging, one can use the catch-all topic #

	mosquitto_sub -d -v -t \#

which will display all messages on the local mosquitto server, which can help e.g. when trying to find the topic as well as JSON path when setting up new items/things in openhab.

Using MQTT binding to get/set data from ebusd
-----

The usual things files has to be set up as in the included configuration. This means, the MQTT broker setup should have been confirmed, tried and tested as described above and then the individual topics can be listened to. The eBusd works with get/set subtopics and the main topic is then used for updates e.g.:

	ebus/bai/FlowTemp/get
	ebus/bai/FlowTemp/

The openhab2 .things configurartion can only handle a main state topic and a command topic, but not a separate get topic. 

I have therefore used a rule, which is periodically called to send the respective get subtopics (with empty field) and listen to the updates. 
Rule for MQTT action:

	rule "Periodic eBus data update using MQTT get's"
	when
    Time cron "0 0/2 * * * ?"
	then
  		val actions = getActions("mqtt","mqtt:broker:mosquitto")
  		actions.publishMQTT("ebus/bai/FlowTemp/get"," ")    
  		logInfo(filename, "sending MQTT get's to eBus)
	end

MQTT .things file example configuration:

	Bridge mqtt:broker:mosquitto [ host="127.0.0.1", port="1883", secure=false,clientID="openhab2" ]{
    	Thing topic bai "eBusd Boiler BAI" @ "Home" {
    	Channels:
        	Type number : FlowTemp "Boiler Flow Temperature" [ stateTopic="ebus/bai/FlowTemp", transformationPattern="JSONPATH:$.temp.value"]
        }
	}

