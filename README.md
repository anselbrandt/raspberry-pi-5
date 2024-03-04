# Gadget Mode on Raspberry Pi 5

https://www.hardill.me.uk/wordpress/2023/12/23/pi5-usb-c-gadget/

### Enable Low Power USB Boot

Add the following to `/boot/config.txt

`usb_max_current_enable=1`

### Update Kernel

Update the Kernel and EEPROM to  enable USB C  connection

`sudo rpi-update`

### Configure kernel

- Add `dtoverlay=dwc2` to the `/boot/config.txt` file
- Add `modules-load=dwc2` to the end of `/boot/cmdline.txt` file
- Add `libcomposite` to the end of the `/etc/modules` file

### libcomposite

Add the following to `/usr/local/sbin/usb-gadget.sh`:

```
#!/bin/bash
 
cd /sys/kernel/config/usb_gadget/
mkdir -p display-pi
cd display-pi
echo 0x1d6b > idVendor # Linux Foundation
echo 0x0104 > idProduct # Multifunction Composite Gadget
echo 0x0103 > bcdDevice # v1.0.3
echo 0x0320 > bcdUSB # USB2
echo 2 > bDeviceClass
mkdir -p strings/0x409
echo "fedcba9876543213" > strings/0x409/serialnumber
echo "Ben Hardill" > strings/0x409/manufacturer
echo "Display-Pi USB Device" > strings/0x409/product
mkdir -p configs/c.1/strings/0x409
echo "CDC" > configs/c.1/strings/0x409/configuration
echo 250 > configs/c.1/MaxPower
echo 0x80 > configs/c.1/bmAttributes
 
#ECM
mkdir -p functions/ecm.usb0
HOST="00:dc:c8:f7:75:15" # "HostPC"
SELF="00:dd:dc:eb:6d:a1" # "BadUSB"
echo $HOST > functions/ecm.usb0/host_addr
echo $SELF > functions/ecm.usb0/dev_addr
ln -s functions/ecm.usb0 configs/c.1/
 
#RNDIS
mkdir -p configs/c.2
echo 0x80 > configs/c.2/bmAttributes
echo 0x250 > configs/c.2/MaxPower
mkdir -p configs/c.2/strings/0x409
echo "RNDIS" > configs/c.2/strings/0x409/configuration
 
echo "1" > os_desc/use
echo "0xcd" > os_desc/b_vendor_code
echo "MSFT100" > os_desc/qw_sign
 
mkdir -p functions/rndis.usb0
HOST_R="00:dc:c8:f7:75:16"
SELF_R="00:dd:dc:eb:6d:a2"
echo $HOST_R > functions/rndis.usb0/dev_addr
echo $SELF_R > functions/rndis.usb0/host_addr
echo "RNDIS" >   functions/rndis.usb0/os_desc/interface.rndis/compatible_id
echo "5162001" > functions/rndis.usb0/os_desc/interface.rndis/sub_compatible_id
 
ln -s functions/rndis.usb0 configs/c.2
ln -s configs/c.2 os_desc
 
udevadm settle -t 5 || :
ls /sys/class/udc > UDC
 
sleep 5
 
nmcli connection up bridge-br0
nmcli connection up bridge-slave-usb0
nmcli connection up bridge-slave-usb1
sleep 5
service dnsmasq restart
```

`sudo chmod +x /usr/local/sbin/usb-gadget.sh`

Add the service to systemd:

`sudo systemctl enable usbgadget.service`

### NetworkManager

```
sudo nmcli con add type bridge ifname br0
sudo nmcli con add type bridge-slave ifname usb0 master br0
sudo nmcli con add type bridge-slave ifname usb1 master br0
sudo nmcli connection modify bridge-br0 ipv4.method manual ipv4.addresses 10.55.0.1/24
```

### DNSMASQ

`sudo apt-get install dnsmasq`

Add the following to `/etc/dnsmasq.d/br0`:

```
dhcp-authoritative
dhcp-rapid-commit
no-ping
interface=br0
dhcp-range=10.55.0.2,10.55.0.6,255.255.255.248,1h
dhcp-option=3
leasefile-ro
```
