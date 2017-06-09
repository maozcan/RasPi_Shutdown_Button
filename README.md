# Raspberry Pi Shutdown/Restart Button

## Hardware setup
Raspberry Pi GPIO pins are being used to send 'shutdown' command to turn off the Pi and to restart the device from off status.

Physical GPIO pin#7 turns off the system when you press switch#1 for 3+ seconds. The LED that is connected to pin#11 starts to blink and the shutdown routine is triggered. (LED is on when the system is operational.)

While the system is turned off, if the user presses the switch#2 which is connected to pin#5, RasPi gets a reboot signal and the system turns on.

EagleCad schematic and board layouts are available below:

[Schematic](/PiShutdownButton.sch) >>
![Schematic](/RasPi_Shutdown_Schematic.png "Schematic")

[Board](/PiShutdownButton.brd) >>
![Board](/RasPi_Shutdown_Board.png "Board")

The board is then processed using CNC isolation routing; gcode created using php-gcode ULP and then the board is prepared on CNC machine. The pin headers have been soldered parallel to the board to help easier connection to RasPi board.


## Software setup

Python is used to handle the GPIO operations.

Install Python3 if it's not already installed. Also reinstall rpi.gpio package to ensure it is up to date.

```
> sudo apt-get install python3 rpi.gpio
```

Add the python script to the /root folder and paste in the following content.

```
> sudo touch /root/pi_shutdown.py
> sudo nano /root/pi_shutdown.py
```

Add the script to root user's crontab so that it will run at each reboot.

```
> sudo crontab -e
```
Add the following line to the bottom of the file
```
@reboot sudo /usr/bin/python3 /root/pi_shutdown.py  > /root/pi_shutdown.log 2>&1
```


The full code for the **pi_shutdown.py** script.
```python

#!/usr/bin/env python3
# Author: Mehmet Ozcan
# Harware setup ---->
# Microswitch_1 --> physical pins 1 and 7 on RPi (performs the shutdown)
# Microswitch_2 --> physical pins 5 and 9 on RPi (performs reset and also power on)
# LED and Resistor (220 ohm) --> LED+ pin connects to pin 11 on RPi
# .. LED- pin connects to the resistor and the resistor connects to pin 9 on RPİ (GND)
# Instructions ----->
# Install Python3: (and also reinstall rpi.gpio)
# > sudo apt-get install python3 rpi.gpio
# Add this file to /root folder
# > sudo touch /root/pi_shutdown.py
# > sudo nano /root/pi_shutdown.py (and paste all the content)
# > sudo chmod +x /root/pi_shutdown.py
# Add the script to (root user's) crontab so that it runs at each reboot
# > sudo crontab -e
# ... and add the following line to the bottom of the file
# @reboot sudo /usr/bin/python3 /root/pi_shutdown.py  > /root/pi_shutdown.log 2>&1

import os
import time
from time import sleep
import signal
import sys
from subprocess import call
import RPi.GPIO as GPIO

ledPin = 11
buttonPin = 7

def Shutdown(pin):
    pressedTime = time.monotonic()
    while GPIO.input(buttonPin):
        if time.monotonic()-pressedTime >= 3:
            blinkLed(3)
#            GPIO.output(ledPin, GPIO.LOW)
#            print("button pressed")
            call("halt", shell=False)
            break
#    os.system("sudo shutdown -h")
#    sleep(100)

def setup():
    GPIO.setmode(GPIO.BOARD)
    GPIO.setup(buttonPin, GPIO.IN, pull_up_down = GPIO.PUD_DOWN)
    GPIO.add_event_detect(buttonPin, GPIO.RISING, callback=Shutdown, bouncetime=200)
    GPIO.setup(ledPin, GPIO.OUT, initial=1)
    GPIO.setwarnings(False)
    return()

def blinkLed(intervals):
    for interval in range(intervals):
        GPIO.output(ledPin, GPIO.HIGH)
        time.sleep(1)
        GPIO.output(ledPin, GPIO.LOW)
        time.sleep(1)
        GPIO.output(ledPin, GPIO.HIGH)

try:
    setup()
    while True:
        time.sleep(1)

except KeyboardInterrupt: # trap a CTRL+C keyboard interrupt
    GPIO.cleanup() # resets all GPIO ports used by this program


```


Inspired from
https://www.element14.com/community/docs/DOC-78055/l/adding-a-shutdown-button-to-the-raspberry-pi-b
