# GPS disciplined NTP server hosted on a RPI3B+

Hardware needed: 

Dupont cables

Raspberry Pi

NEO GPS module as seen below:

![GPS module](gps.png?raw=true "GPS module")

OS information: 

```
$ cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

```
$ uname -a 
Linux ntp.home.net 6.1.21-v8+ #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 aarch64 GNU/Linux
```

Add the following lines to the following file: /boot/config.txt

```
#Changes for GPS Clock
dtoverlay=pi3-miniuart-bt # disables bluetooth
dtoverlay=pps-gpio,gpiopin=18 # exposes pps to the kernel and sets it to GPIO pin 18
```
Install updates to the system and add the following packages:

```
sudo apt get update
sudo apt get upgrade 
reboot # if updates were installed and applied
sudo apt install pps-tools gpsd gpsd-clients chrony libcap-dev
```
Power off the RPI and attach the GPS Module to the following Pin outs on the RPI:
```
GPS module        RPI
+---------------------+

VCC              Pin 4 5V power

GND              Pin 6 GND

TXD              Pin 10 GPIO 15 UART RX

RXD              Pin 8 GPIO 14 UART TX

PPS              Pin 12 GPIO 18 PCM CLOCK
```

Power on the Pi and run the following command to ensure a PPS signal is detected by the kernel:
```
$ sudo dmesg | grep pps
[    0.194625] pps_core: LinuxPPS API ver. 1 registered
[    0.194664] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    8.108780] pps pps0: new PPS source pps@12.-1
[    8.111341] pps pps0: Registered IRQ 185 as PPS source
```
Configure GPSd in /etc/default/gpsd:

File should be there upon installation but if not create it using: ```touch gpsd```

You can check ```dmesg``` to see what the gps module serial name is but generally it's going to be at /dev/ttyAMA0

Config file will look like this:

```
# Devices gpsd should collect to at boot time.
# They need to be read/writeable, either by user gpsd or the group dialout.
DEVICES="/dev/ttyAMA0"
START_DAEMON="true"

# Other options you want to pass to gpsd
GPSD_OPTIONS="-n"

# Automatically hot add/remove USB GPS devices via gpsdctl
USBAUTO="true"
```
Enable the service and start it:

``` sudo systemctl enable gpsd && sudo systemctl start gpsd```




