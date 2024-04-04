# Network Manager

```
sudo nmcli device status

sudo nmcli radio

sudo nmcli dev wifi list

sudo nmcli device wifi connect <SSID|BSSID> password <password>

# To set up a device as an AP - this assumes that WLAN0 is the wireless interface

sudo nmcli dev wifi hotspot ifname wlan0 <SSID> password "<password>"

sudo nmtui
```
