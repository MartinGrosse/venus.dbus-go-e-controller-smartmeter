# dbus-fronius-smartmeter Service

### Purpose

This service is meant to be run on a raspberry Pi with Venus OS from Victron.

The Python script cyclically reads data from the go-e controller SmartMeter via the go-e controller API and publishes information on the dbus, using the service name com.victronenergy.grid. This makes the Venus OS work as if you had a physical Victron Grid Meter installed.

### Configuration

1. In the Python file, you should put the IP of your go-e controller device that hosts the REST API.
2. Used API keys for each phase are: coltage -> "usv", current(i) and power(p) -> "isv" 
3. In my go-e configuration home and grid is the same and the API key "cec" is used, for cunsumtion values AC power "ccp"

### Installation

1. Copy the files to the /data folder on your venus:

   - /data/dbus-go-e-controller-smartmeter/dbus-go-e-controller-smartmeter.py
   - /data/dbus-fronius-smartmeter/kill_me.sh
   - /data/dbus-go-e-controller-smartmeter/service/run

2. Set permissions for files:

   `chmod 755 /data/dbus-go-e-controller-smartmeter/service/run`

   `chmod 744 /data/dbus-go-e-controller-smartmeter/kill_me.sh`

3. Get two files from the [velib_python](https://github.com/victronenergy/velib_python) and install them on your venus:

   - /data/dbus-go-e-controller-smartmeter/vedbus.py
   - /data/dbus-go-e-controller-smartmeter/ve_utils.py

4. Add a symlink to the file /data/rc.local:

   `ln -s /data/dbus-go-e-controller-smartmeter/service /service/dbus-go-e-controller-smartmeter`

   Or if that file does not exist yet, store the file rc.local from this service on your Raspberry Pi as /data/rc.local .
   You can then create the symlink by just running rc.local:
  
   `rc.local`

   The daemon-tools should automatically start this service within seconds.

### Debugging

You can check the status of the service with svstat:

`svstat /service/dbus-go-e-controller-smartmeter`

It will show something like this:

`/service/dbus-go-e-controller-smartmeter: up (pid 10078) 325 seconds`

If the number of seconds is always 0 or 1 or any other small number, it means that the service crashes and gets restarted all the time.

When you think that the script crashes, start it directly from the command line:

`python /data/dbus-go-e-controller-smartmeter/dbus-go-e-controller-smartmeter.py`

and see if it throws any error messages.

If the script stops with the message

`dbus.exceptions.NameExistsException: Bus name already exists: com.victronenergy.grid"`

it means that the service is still running or another service is using that bus name.

If the script seems running start `dbus-spy`, you should see 
`com.victronenergy.grid.mymeter                                                                      go-e controller Smart Meter`

Use arrow key to go down and hit Enter.

If every thing is fine you see the values and the `UpdateIndex` is rising till 255 and restart by 0 every second.

#### Restart the script

If you want to restart the script, for example after changing it, just run the following command:

`/data/dbus-go-e-controller-smartmeter/kill_me.sh`

The daemon-tools will restart the scriptwithin a few seconds.

### Hardware

In my installation at home, I am using the following Hardware:

- go-e controller as meter (three phases)
- OpenDTU as PV meter (one phases)
- Victron MultiPlus-II - Battery Inverter (single phase)
- Raspberry Pi 3B+ - For running Venus OS
- go-e charger
