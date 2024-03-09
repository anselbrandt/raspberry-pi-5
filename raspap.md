# RaspAP Notes

RaspAP Config

sudo nano /etc/dhcpcd.conf

Preconfigured Network

sudo nano /etc/NetworkManager/system-connections/preconfigured.nmconnection

That file needs to be deleted before reboot to prevent NetworkManager from taking over wlan0 and connecting to your wifi
