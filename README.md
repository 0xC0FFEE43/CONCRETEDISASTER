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

