# rmf-gateway-pi

## Setup
```
# Flash the most recenty Raspberry Pi OS with Desktop: https://www.raspberrypi.org/software/operating-systems/
# Connect over Ethernet to internet
sudo raspi-config    # Activate camera and ssh
sudo reboot

sudo apt update
sudo apt install avahi-daemon wireguard vim -y

# Install RaspAp and configure it 
curl -sL https://install.raspap.com | bash
# Add the following to /etc/dhcpcd.conf
interface wlan0
... # raspap configurations
nogateway

interface eth0
nogateway

# Setup 4g dongle
apt install git ppp libusb-1.0-0-dev wvdial
git clone https://github.com/Trixarian/sakis3g-source.git
cd sakis3g-source
sed -i 's/libusb.h/libusb-1.0\/libusb.h/g' ./dependencies/usb-modeswitch/usb_modeswitch.h
./compile
cp build/sakis3gz /usr/bin/sakis3g
cp files/sakis3g.conf /etc/

# Fill in the following in /etc/wvdial.conf
[Dialer defaults]
Modem = /dev/ttyUSB1
Init3 = AT+CGDCONT=1,”IP”,”internet”
Phone = *99#
Stupid Mode = 1
Username = ” ”
Password = ” “

# run the following
wvdialconf

# Get the hardware ID for D-Link
lsusb | grep D-Link			# Should be 2001:7e35
sudo sakis3g connect    # Go through prompts
> Prompt answers
    > USB device
    > Mobile Connect
    > Interface #1
    > APN: shwap 
    > Username: 0 password: 0

# Once it works, add the following to /etc/sakis3g.conf, changing the values with what you used for successful connection
OTHER="USBMODEM"
USBMODEM="2001:7e35"
USBINTERFACE="1"
APN="shwap"
APN_USER="0"
APN_PASS="0"

# Set Up Wireguard
# Edit /etc/wireguard/wg0.conf
You might need to run apt install raspberrypi-kernel-headers
sudo systemctl start wg-quick@wg0.service # And enable

# If using nginx, change server port to something other than 80 in /etc/lighttpd/lighttpd.conf

# use cron job to startup 4g
# add the following to /home/pi/startup-dlink.bash
#!/bin/bash

ping 10.66.66.1 -c 1 && exit 0
pkill ping
sudo sakis3g disconnect || true
sudo wg-quick down wg0 || true
sudo sakis3g connect
sudo wg-quick up wg0
ping 10.66.66.1 &

# Add this to crontab -e, if your user is pi
* * * * * /bin/bash /home/pi/startup-dlink.bash 
```
