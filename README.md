# GPS disciplined NTP server hosted on a RPI3B+

Hardware needed: 

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

GPS module        RPI
+---------------------+
VCC              Pin 4 5V power
GND              Pin 6 GND
TXD              Pin 10 GPIO 15 UART RX
RXD              Pin 8 GPIO 14 UART TX
PPS              Pin 12 GPIO 18 PCM CLOCK

