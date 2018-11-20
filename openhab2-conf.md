openHAB2 installation
===============

The easiest installation for openHAB2 is through [openhabian](https://github.com/openhab/openhabian/releases). The image needs to be downloaded and then transferred to an SD card e.g. via [etcher](etcher.io).

Once booted up, the installation can be followed through SSH, the user/pw is by default openhabian.

It will take a few minutes before the reboot and then the usual

    sudo apt-get update
    sudo apt-get upgrade
followed by a

    sudo openhabian-config
In order to select additional software/settings, go to local/timezone under `System Settings` or installing Log Viewer in `Optional Components`. Additionally an MQTT server should be installed.
