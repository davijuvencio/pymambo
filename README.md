# pymambo
Python interface for Parrot Mambo

This interface was developed to teach kids (K-12) STEM concepts (programming, math, and more) by having them program a drone to fly autonomously.  Anyone can use it who is interested in autonomous drone programming!  

## Requirements

### Hardware

This code was developed and tested on a Parrot Mambo (regular Mambo, without the FPV camera, we will work on that in the next update) and a Raspberry Pi 3 Model B.  It should work on any linux machine with BLE support.

### Software
You will need python, pybluez, and untangle.  All of these are available for the Raspberry Pi 3.  To install these do the following:

```
sudo apt-get update
sudo apt-get install bluetooth
sudo apt-get install bluez
sudo apt-get install python-bluez
pip install untangle
```

To install the pymambo code, download or clone the repository.

## Using the pymambo library

To use the library, you will first need to find the address of your Mambo.  BLE permissions on linux require that this command run in sudo mode.  To this this, from the directory where you installed the pymambo code, type:

```
sudo python findMambo.py
```

This will identify all BLE devices within hearing of the Pi.  The Mambo will be identified at the end.  Save the address and use it in your connection code (discussed below).  If findMambo does not report "FOUND A MAMBO!", then be sure your Mambo is turned on when you run the findMambo code and that your Pi (or other linux box) has its BLE interface turned on.

## Flying

Once you have gotten the address of your mambo from findMambo, you can use it to connect to the Mambo and fly!  We have provided some example scripts and a list of the available commands for writing your own scripts.

Note that you do not need to run any of the flying code in sudo mode!  That was only for discovery.

I have provided three demo programs. 

```
python demoTricks.py
```
demoTrick.py will take off, demonstrate all 4 types of flips, and then land.

```
python demoDirectFlight.py
```
demoDirectFlight.py will demonstrate directly controlling the roll, pitch, yaw, and vertical control.  Make sure you try this one in a large enough room!

```
python demoClaw.py
```
demoClaw shows you how to control the claw.  The gun can also be controlled through the python interface.  In this demo program, the mambo takes off, opens and closes the claw, and lands again.  Once the FPV camera is integrated, we can use it to actually pick up objects.

## mambo flying commands

Each of the commands available to control the mambo is listed below with its documentation.  The code is also well documented.  All of the functions preceeded with an underscore are intended to be internal functions are not listed below.

* ```Mambo(address)``` create a mambo object with the specific harware address (found using findMambo)
* ```connect(num_retries)``` connect to the Mambo's BLE services and characteristics.  This can take several seconds to ensure the connection is working.  You can specify a maximum number of re-tries.  Returns true if the connection suceeded or False otherwise.
* ```disconnect``` disconnect from the BLE connection
* ```takeoff()``` Sends a single takeoff command to the mambo.  This is not the recommended method.
* ```safe_takeoff()``` This is the recommended method for takeoff.  It sends a command and then checks the sensors (via flying state) to ensure the mambo is actually taking off.  Then it waits until the mambo is flying or hovering to return.
* ```land()``` Sends a single land command to the mambo.  This is not the recommended method.
* ```safe_land()``` This is the recommended method to land the mambo.  Sends commands until the mambo has actually reached the landed state.
* ```hover()``` Puts the mambo into hover mode.  This is the default mode if it is not receiving commands.
* ```flip(direction``` Sends the flip command to the mambo. Valid directions to flip are: front, back, right, left.
* ```turn_degrees()``` Turns the mambo in place the specified number of degrees.  The range is -180 to 180.  This can be accomplished in direct_fly() as well but this one uses the internal mambo sensors (which are not sent out right now) so it is more accurate.
* ```smart_sleep()``` This sleeps the number of seconds but wakes for all BLE notifications.  This comamnd is VERY important.  NEVER use regular time.sleep() as your BLE will disconnect regularly!  Use smart_sleep instead!
* ```turn_on_auto_takeoff()``` This puts the mambo in throw mode.  When it is in throw mode, the eyes will blink.
* ```take_picture()``` The mambo will take a picture with the downward facing camera.  It is stored internally on the mambo and you can download them using a mobile interface.  As soon as I figure out the protocol for downloading the photos, I will add it to the python interface.
* ```ask_for_state_update()``` This sends a request to the mambo to send back ALL states (this includes the claw and gun states).  Only the battery and flying state are currently sent automatically.  This command will return immediately but you should wait a few seconds before using the new state information as it has to be updated by BLE characteristic handlers and it sends each type of state in a separate BLE packet.
* ```fly_direct(roll, pitch, yaw, vertical_movement, duration)``` Fly the mambo directly using the specified roll, pitch, yaw, and vertical movements.  The commands are repeated for duration seconds.  Note there are currently no sensors reported back to the user to ensure that these are working but hopefully that is addressed in a future firmware upgrade.
* ```open_claw()``` Open the claw.  Note that the claw should be attached for this to work.  The id is obtained from a prior ```ask_for_state_update()``` call.
* ```close_claw()``` Close the claw. Note that the claw should be attached for this to work.  The id is obtained from a prior ```ask_for_state_update()``` call.
* ```fire_gun()``` Fires the gun.  Note that the gun should be attached for this to work.  The id is obtained from a prior ```ask_for_state_update()``` call.



## Planned updates/extensions

This is a work in progress.  Planned extensions include:

* FPV camera.  I was developing this beofre the Mambo had a FPV camera.  Once I get a FPV, I will udpate the library.
* Downloading pictures from the downward facing camera.  We can take photos from it (mambo.take_picture()) but I haven't figured out the protocol to download the photos remotely yet.  When I figure that out, I will update the code.
* General BLE stability improvements.  Sometimes the BLE simply disconnects.  Fortunately the mambo stays stable and hovers but it would be nice to detect the disconnect and try to reconnect.
* Sensors.  The mambo currently only sends a limited number of sensors back regularly (flying state and battery).  They have stated they will improve this in a future firmware release.  I will update the code to handle the new sensors (hopefully including altitude!) when the firmware is updated.