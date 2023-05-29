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
Linux <hostname> 6.1.21-v8+ #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 aarch64 GNU/Linux
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

If all goes well you can verify the gps in action by running: ``` cgps or gpsmon ```

In ```gpsmon``` you will see the following ```TOFF:  0.401942753       PPS:  0.000018718``` if there's a PPS value then PPS should be working if not it will be blank. This aides in troubleshooting.

Another test is running the following: ``` sudo ppstest /dev/pps0 ```
Expected output is the following(It will output every second): 
```
$ sudo ppstest /dev/pps0
trying PPS source "/dev/pps0"
found PPS source "/dev/pps0"
ok, found 1 source(s), now start fetching data...
source 0 - assert 1685342059.000001866, sequence: 59719 - clear  0.000000000, sequence: 0
source 0 - assert 1685342060.000028904, sequence: 59721 - clear  0.000000000, sequence: 0
source 0 - assert 1685342060.999998859, sequence: 59722 - clear  0.000000000, sequence: 0
source 0 - assert 1685342062.000001365, sequence: 59723 - clear  0.000000000, sequence: 0
source 0 - assert 1685342063.000002153, sequence: 59724 - clear  0.000000000, sequence: 0
source 0 - assert 1685342064.000002046, sequence: 59725 - clear  0.000000000, sequence: 0
source 0 - assert 1685342065.000001507, sequence: 59726 - clear  0.000000000, sequence: 0
source 0 - assert 1685342066.000001593, sequence: 59727 - clear  0.000000000, sequence: 0
```

Time to configure chrony:

```vi /etc/chrony/chrony.conf```
Add the following:
```
# optional but I prefer to use NIST time servers over the pool servers that come default on the OS
server time-a-b.nist.gov 
server time-e-wwv.nist.gov

#config to use pps as a time source pulled from https://chrony.tuxfamily.org/faq.html
refclock PPS /dev/pps0 lock NMEA refid GPS
refclock SHM 0 offset 0.5 delay 0.1 refid NMEA # This is because the gpsd packaged version in raspian is 3.22 documentation above states this is what you need to put in for versions of gpsd below 3.25
```
```
$ gpsd -V
gpsd: 3.22 (revision 3.22)
```

Save the file and then run the following:

```sudo systemctl enable chronyd && sudo systemctl start chronyd```

then run the following:

```chronyc sources``` or ``` watch -n1 chronyc sources``` to watch it update in "real-time".
After about 30sec chrony should start receiving the PPS signal the GPS signal. As shown below.

```
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
#* GPS                           0   4   377    10   +101ns[ +140ns] +/-  183ns
#x NMEA                          0   4   377    12    +91ms[  +91ms] +/-   51ms
^- time-a-b.nist.gov             1   6    77    14  +1041us[+1041us] +/-   20ms
^- time-e-wwv.nist.gov           1   6    77    14   -292us[ -292us] +/-   21ms
```
Strange occurance is the NMEA column displays as in error. Technically not in use. 








