# TWCManager Documentation

## Having Trouble?

If you're having trouble getting TWCManager working, check out our [Troubleshooting Guide](Troubleshooting.md) to see if any of our tips help you out!

## Physical (RS-485) Installation

Please see the [Installation Guide](InstallationGuide.md) for detailed information on the installation of the Tesla Wall Connector interface to TWCManager.

## Software Installation

### Recommended Installation

The recommended installation for this project is on a Raspberry Pi machine using Raspbian. The Raspbian OS can be downloaded from the following location:

   * https://www.raspberrypi.org/downloads/raspbian/

You can flash the Raspbian OS using the tools listed on the following page:

   * https://www.raspberrypi.org/documentation/installation/installing-images/README.md

### Install Required Packages (Debian/Ubuntu/Raspbian)

The following packages are required to fetch and install the TWCManager project. These are the minimal required packages to start the installation process, during which any other required dependencies will be fetched automatically.

```
sudo apt-get update
sudo apt-get install -y git python3 python3-setuptools python3-dev
```

### Default to Python3

TWCManager requires a minimum of python 3.4 to work correctly. To attempt to support Raspberry Pi OS versions going back to 2019, TWCManager is regularly tested against Python 3.5 to ensure that support is retained. As of TWCManager v1.2.2, a number of features are beginning to diverge based on minimum Python versions being higher than those required by TWCManager, so the following features may be unavailable if your Python version is below the minimum:

   * Support for OCPP control module requires a minimum Python 3.6 version

Raspberry Pi OS version 9 (stretch) from 2019 ships with Python 3.5.3. If you are running Raspberry Pi OS version 9 or earlier, you may not have access to a Python interpreter which supports the above features. You may want to consider the use of [pyenv](pyenv.md) to support installation of a newer Python interpreter.

Python versions below 3.6 may show the following error when running ```setup.py```:

```
Traceback (most recent call last):
  File "setup.py", line 3, in <module>
    from setuptools import setup, find_namespace_packages
ImportError: cannot import name find_namespace_packages
```

This indicates an older version of setuptools is installed. To resolve this issue, run the following command:

```
pip3 install --upgrade setuptools
```

### Raspberry Pi OS / Raspbian Buster

You may need to set python3 as your default python interpreter version on Raspberry Pi OS / Debian Buster. The following command will set python 3.7 as your default interpreter.

```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.7 2
```

You can check that this command has been successful by running ```python --version``` and checking that the version is python3.

### Raspbian Stretch

You may need to set python3 as your default python interpreter version on Debian/Ubuntu. The following command will set python 3.5 as your default interpreter. 

```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2
```

You can check that this command has been successful by running ```python --version``` and checking that the version is python3.

### Clone GIT Repository and copy files

During this step, the source code and all related files will be cloned from the GitHub repository and installed into the appropriate location on your system.

We have two versions of the codebase that you may want to check out. The stable version is **v1.2.1**, which will only change for stability or urgent fixes. To check out **v1.2.1**, follow these steps:

```
git clone https://github.com/ngardiner/TWCManager
cd TWCManager
git checkout v1.2.1
sudo make install
```

Alternatively, the **main** branch is the development branch, where all of the new ideas and features are tested prior to becoming the stable branch. This version has more features, but we can't guarantee stability.

```
git clone https://github.com/ngardiner/TWCManager
cd TWCManager
git checkout main
make install
```

### Configure TWCManager
After performing the installation tasks above, edit the /etc/twcmanager/config.json file and customize to suit your environment.

The following documents provide detail on specific areas of configuration:

   * [Policy Customization](PolicyCustomization.md)

### Running the script
Once the above steps are complete, start the TWCManager script with the following command:

```
python -m TWCManager
```

## Monitoring the script operation

After starting TWCManager, the script will run in the foreground and will regularly update with the current status. An example output is as follows:

<pre>
11:57:49: <b>SHA 1234</b>: 00 <b>00.00/00.00A</b> 0000 0000  <b>M</b>: 09 <b>00.00/17.00A</b> 0000 0000
11:57:49: Green energy generates <b>4956W</b>, Consumption <b>726W</b>, Charger Load <b>0W</b>
          Limiting car charging to 20.65A - 3.03A = <b>17.62A</b>.
          Charge when above <b>6A</b> (minAmpsPerTWC).
</pre>

   * SHA 1234 is the reported TWC code for each of the Slave TWCs that the Master is connected to.
   * The 00.00/00.00A next to the Slave TWC code is the current number of amps being utilised and the total number of amps available to that slave. The master divides the total amperage available for use between the connected slaves.
   * M is the same values for the Master device (our script). It shows current utilization and total available amps (in this case, 17A) available for all connected slaves.
   * The second line shows the green energy values detected. In this case, the attached green energy device (solar inverter) is reporting 4956W being generated from solar, 726W being used by other household appliances, and no load being generated by the charger. As we charge a vehicle, that value will increase and may be subtracted from the consumption value if configured to do so.
   * The line below this reports the same values but in amps instead of watts.
   * The final line shows the minAmpsPerTWC value, which is the minimum number of amps that the master must offer each slave before we tell the attached vehicle to charge (via the Tesla API).

## Running TWCManager as a Service

The following commands make TWCManager run automatically (as a service) when the Raspberry Pi boots up.

Enable the service. The --now flag also makes it start immediately. This will persist after rebooting.
```
sudo cp twcmanager.service /etc/systemd/system/twcmanager.service
sudo systemctl enable twcmanager.service --now
```
To check the output of the TWCManager service
```
journalctl -f
```
To disable the TWCManager service permanently use the following command.  This will persist after rebooting.
```
sudo systemctl disable twcmanager
```

## Developing for TWCManager

Your contributions are most welcome! If you've been working on a new EMS module or you want to contribute to the project in any way, please take a look at our [Development Guide](DevelopmentGuide.md) and feel free to get involved!

## Frequently Asked Questions

### How many units can be set up in this fashion?

The TWC Load Balancing protocol allows for up to four units within a group. As we are occupying one of the TWC Load Balancing unit IDs in order to provide the Master control of the charger group, there may be a total of three other units connected. When connecting additional units, they should be chained from the unused (In or Out) RS-485 terminals of the unit that is currently connected to TWCManager, which will allow all (up to) three of the units to be managed by one TWCManager instance.

### What can I do if my TWC is showing a red light blinking on the front of the unit?

This is because it has identified an error. If this occurred after starting the TWCManager.py script, it is highly likely that it has been caused by the TWCManager script.

Check the output of the TWCManager.py script. This will show you the reason for the error if it has been detected by the script. For example, if your rotary switch has not been adjusted to make the TWC a slave unit, you will see the following warning:

```
03:38:12 ERROR: TWC is set to Master mode so it can't be controlled by TWCManager.  Search installation instruction PDF for 'rotary switch' and set switch so its arrow points to F on the dial.
```

Similarly, if you are not running the TWCManager.py script and your TWC is set to Slave Mode, the same error condition will be shown via the TWC blinking red LED. In both cases, the error code is green: solid and red: 4 blinks. If you have any other error condition shown, refer to the table in your TWC user guide for specific details.

### Why is my car only charging at 6A?

There are a few reasons why you might see your car charging only at 6 amps:

  * There is less than 6A worth of generation capacity available
     * In this case, the charger will charge at 6A (2kW), as charging below this rate is of no benefit - it would not be sufficient to power the battery conditioning, and would be very inefficient as most of the power would be lost.
     
  * Your configuration has not been modified from the default
     * There are some safe defaults used in the config.json which ships with TWCManager. In particular:
        * wiringMaxAmpsAllTWCs: The maximum number of amps that all slave TWCs are able to draw simultaneously from the shared power source. Because a load-balanced TWC installation involves each of the TWCs sharing the same power feed, we need to configure a maximum allocation of current for all connected TWCs. This is set to 6A by default.
        * wiringMaxAmpsPerTWC: The maximum number of amps that each individual TWC is capable of drawing. If you think of the previous value (AllTWCs) as the capacity of the trunk power source that all TWCs are drawing from, this value (PerTWC) is the capability of each indiviual TWC, based on the wire gauge between the shared power source and the individual TWC. In a single TWC installation, this will be equal to the value of wiringMaxAmpsAllTWCs.
        
  * Conflict between Charger Consumption and Consumption Sensor
     * There are two ways in which your charger may be wired in an environment where you are able to access a consumption sensor:
        * The charger may be monitored as part of the consumption meter.
        * Your charger may not be monitored as part of the consumption meter.
     * If your charger is not monitored by the consumption meter, you do not need to make any changes to the default configuration. This configuration assumes that when you access consumption data, it does not count the charging load.
     * If your charging load/draw is counted by the consumption meter, this would cause the charger to be consistently forced down to 6A, as all of the generation would be canceled out by the power you draw to charge the vehicle.

### Why 6A?

   * 6A is the default minAmpsPerTWC setting. 
   * Charging at 240V 5A wastes 8.6% more energy than charging at 240V 10A.
      * Because of this, it is important to find a point at which you are comfortable setting a floor for charging current. If this is set too low, it will be inefficient. 

### Why doesn't charging stop when Solar generation drops below minimum

   * There are a few different ways in which we can stop a car from charging on a TWC:

      * V1 Chargers

The first revision of TWC chargers were able to stop charging by setting the maxAmps value to 0 amps. This is not effective for newer TWC chargers, however.

      * Stop communicating with the Slave TWCs. 
      
This is effective, in that stopping communications with a Slave TWC will stop the car from charging. Unfortunately the byproduct of this is that this can sometimes result in the car going into an error state, where charging can only be restarted by unplugging and replugging the TWC.

In an upcoming release, this will be offered as a switchable option to replace the use of Tesla API keys for those who are not comfortable with providing their credentials to the script.
      
      * Use the Tesla API to connect to the car, sending a command to stop/start charging.

This is set up using the web interface. Log in with your Tesla login and password, and the login token will be stored locally within a settings file.

If you have multiple cars, TWCManager will attempt to identify which cars are home using geofencing. The following page of the TMC forums thread explains it better than I could: https://teslamotorsclub.com/tmc/threads/new-wall-connector-load-sharing-protocol.72830/page-16

### Why do I need to log into my Tesla account when using the web interface?

   * TWCManager uses your Tesla login to obtain an API token. This API token is used to talk to your vehicle(s).
   * When the available charger capacity falls below minAmpsPerTWC, the TWCManager script will contact the Tesla API to tell the vehicle to stop charging. If this is not configured, your vehicle will continue to charge at 6A even when the charging policy dictates that we stop charging.

Note: this functionality is still under development in the new interface.  At the moment it requires a little bit of manual work.  
   * Multi-Factor Authentication (MFA) needs to be turned on in your tesla account, done on the tesla website (www.tesla.com)
   * Enter your tesla user name, password and the current MFA code number into the following website (http://registration.teslatasks.com/generateTokens)
   * Copy the two long generated strings into the following settings file, save the file and then reboot the raspberry pi.  
       * Note the names are slightly different between the website that generates the tokens and the settings.json file:
       * "Access Token" is pasted into the field called "carApiBearerToken" 
       * "Refresh Token" is pasted into the field called "carApiRefreshToken"
       * The settings.json file is one long line of text, so you may need to scroll a long way to the end of that line to find the headings.
```
sudo nano /etc/twcmanager/settings.json 
sudo nano reboot
```
### Why does my TWC increase charging momentarily to 21A or 17A around the time that it changes charging rates?

There are a number of reasons for this:

   * In the 2017.44 firmware version Tesla released around October 2017, a bug was introduced which led to vehicle charging rates falling to 6A when charging rates were raised. This is resolved by spiking the amps.

### Why is it so hard to just stop a vehicle from charging?

Good question:

   * Version 1 of the TWC protocol (for TWCs produced prior to and during early 2017) has a command which will cleanly stop charging Tesla vehicles.

   * For Version 2 TWCs produced after this time, there is no single approach which cleanly stops a Tesla vehicle from charging, without the vehicle itself wanting to stop charging. The options that exist are:

      * Ask the vehicle to stop charging by sending a message via the Tesla vehicle API (obviously a Tesla-only approach)

      * Stop communicating with the TWC for 30 seconds or more. This method stops the connected vehicle from charging, and will resume charging once the communications resume, however doing this a number of times will cause the vehicle to give up and the only way to resume charging is to unplug and replug the vehicle.

      * Sending the Stop message to Slave TWCs. This method will cause charging to cease immediately by instructing the TWC to open its relay, causing the charger to de-energise. A lack of CAN bus communication with the vehicle means that whilst the vehicle will stop charging immediately, it will log a number of errors and refuse to re-start charging again until it is unplugged and re-plugged (and will show a red LED ring around the charger port).

         * This method does work however if the DIP 2 switch is set in the **Down** position, which represents the use of legacy charge mode, which does not use CAN bus communication with Tesla vehicles. The usefulness of this configuration is equivalent to the "Stop communicating with slaves" behaviour above - it allows a number of stops and starts before the vehicle marks the charger as bad and refuses to charge until it is unplugged and re-plugged.

So in summary, the lack of a reasonable approach to stopping charging from the TWC side necessitates an API based solution.

### Why do only some people see vehicle VINs in the TWCManager interface?

This feature was only introduced in firmware version 4.5.3, which is found on TWCs manufactured from March 2018 until the end of 2019.

TWCs prior to this version will not respond to queries from TWCManager regarding the VIN of the vehicle connected. Unfortunately there is no sign that newer TWC firmwares are being installed by Tesla vehicles, and the TWCs themselves do not have internet access to upgrade themselves.

### What do we know about the various different Tesla Wall Charger revisions and how they operate?

   * The Tesla HPWC (Gen 1) wall charger was released in 2012 and was sold up until 2016, and can be identified by the LED. The Gen 1 used a bank of 4 DIP switches to configure the supplied amperage, rather than the rotary switch, and does not feature an RS458 bus at all. Gen 1 HPWC chargers are not capable of load sharing and cannot be used with TWCManager.
   * The Tesla TWC (Gen 2) wall charger was released in 2016 and was sold up until 2020, and differs only slighly from the Gen 1 from a visual perspective. The Gen 2 TWC provided better thermal management for the charger cable (an issue on older HPWCs) and better outdoor enclosure sealing, and introduced the charger load sharing protocol that this project uses.
   * The TWC Gen 3 wall charger was released in January 2020 and is still being sold today, and can be identified by the white faceplate and the shorter charging cable. The Gen 3 TWC has a single RS485 header (whereas the Gen 2 has a double RS485 header) and does not currently have sharing capability. It has the capability to connect to WiFi, however the value of this is not yet known.
