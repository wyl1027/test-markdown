# test-markdown
# 一级标题
## 二级标签

![这是照片](./images/493019.jpg)

**我是文字加粗**

*我是斜体*

[I'm an inline-style link](https://www.google.com)


```shell
#!/bin/bash

#
# Raspberry Pi3 - CentOS 7 - Wi-Fi configuration
#

# Usage: ./wifi-setup.sh [SSID] [PSK]

if [ -z "$1" ]; then
echo "Enter WAP SSID:";
read SSID;
else
SSID=$1;
fi

if [ -z "$2" ]; then
echo "Enter WAP PSK:";
read PSK;
else
PSK=$2;
fi

# install firmware for wifi
echo "Installing Wi-Fi firmware";

curl --location https://github.com/RPi-Distro/firmware-nonfree/raw/54bab3d6a6d43239c71d26464e6e10e5067ffea7/brcm80211/brcm/brcmfmac43430-sdio.bin > /usr/lib/firmware/brcm/brcmfmac43430-sdio.bin

curl --location https://github.com/RPi-Distro/firmware-nonfree/raw/54bab3d6a6d43239c71d26464e6e10e5067ffea7/brcm80211/brcm/brcmfmac43430-sdio.txt > /usr/lib/firmware/brcm/brcmfmac43430-sdio.txt

#restore selinux context for firmware
echo "Restoring selinux context for firmware";
restorecon -Frv /usr/lib/firmware/brcm/

#stop network manager
echo "Stopping and disabling NetworkManager";
systemctl stop NetworkManager
systemctl disable NetworkManager

#enable network
echo "Enabling network";
systemctl enable network

#configure wpa_supplicant
echo "Configuring wpa_supplicant";
echo 'INTERFACES="-i wlan0"
DRIVERS="-D wext"
OTHER_ARGS="-u -f /var/log/wpa_supplicant.log -P /var/run/wpa_supplicant.pid"' > /etc/sysconfig/wpa_supplicant;

#configure wlan0 device
echo "Configuring ifcfg-wlan0";
echo 'BOOTPROTO=dhcp
DEFROUTE=yes
DEVICE=wlan0
KEY_MGMT=WPA-PSK
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
KEY_MGMT=WPA-PSK
NAME=wlan0
ONBOOT=yes
PEERDNS=yes
PEERROUTES=yes
TYPE=Wireless
USERCTL=yes
WPA_ALLOW_WPA=yes
WPA_ALLOW_WPA2=yes
ZONE=trusted
NM_CONTROLLED="no"' > /etc/sysconfig/network-scripts/ifcfg-wlan0

#disable NetworManager control of eth0
echo "Disabling NetworkManager control of eth0 interface";
sed -i -e "s/^NM_CONTROLLED=\"yes\"/NM_CONTROLLED=\"no\"/" /etc/sysconfig/network-scripts/ifcfg-eth0

#add eth0 HWADDR
echo "Adding HWADDR parameter for eth0 interface";
addr=`cat /sys/class/net/eth0/address`
echo "HWADDR=$addr" >> /etc/sysconfig/network-scripts/ifcfg-eth0

#restore selinux context to network-scripts
echo "Restoring selinux context for network-scripts";
restorecon -Frv /etc/sysconfig/network-scripts

#configure your wireless access point
echo "Setting up your wireless network";
wpa_passphrase $SSID $PSK >> /etc/wpa_supplicant/wpa_supplicant.conf

# to prevent an error later on
touch /etc/sysconfig/network

# ensure wpa_supplicant is enabled
echo "Enabling wpa_supplicant";
systemctl enable wpa_supplicant

# enable rc.local for next boot
chmod +x /etc/rc.d/rc.local

# make final setup run on next boot
echo "Setting up final wifi-setup script";
echo "/root/wifi-setup-final.sh" >> /etc/rc.local

echo '#!/bin/bash
#
# Final steps to complete wi-fi setup
#
echo "Completing wi-fi configuration";
#add wlan0 HWADDR
echo "Adding HWADDR parameter for wlan0 interface";
addr=`cat /sys/class/net/wlan0/address`
echo "HWADDR=$addr" >> /etc/sysconfig/network-scripts/ifcfg-wlan0
#reset /etc/rc.local to orignal state
echo "Resetting /etc/rc.local";
echo "touch /var/lock/subsys/local
dhclient wlan0" > /etc/rc.d/rc.local
echo "Final steps complete, rebooting shortly";
echo "Cleaning up wi-fi setup scripts";
rm -f /root/wifi-setup*
sleep 3;
# final reboot
reboot
' > /root/wifi-setup-final.sh

chmod +x /root/wifi-setup-final.sh;

echo "Initial steps complete, rebooting shortly";

sleep 3;

reboot

```
