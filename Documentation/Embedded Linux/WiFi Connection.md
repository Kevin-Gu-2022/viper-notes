- Setup a static-IP WiFi hotspot on the Pika Spark: 

```bash
# Create the new connection
sudo nmcli con add type wifi ifname wlan0 con-name MyHotspot autoconnect yes ssid PikaSpark mode ap

# Set to manual IP (no DHCP server)
sudo nmcli con modify MyHotspot ipv4.method manual ipv4.addresses 192.168.50.1/24

# Set Security
sudo nmcli con modify MyHotspot wifi-sec.key-mgmt wpa-psk
sudo nmcli con modify MyHotspot wifi-sec.psk "Pika"

# Bring the connection up
sudo nmcli con up MyHotspot
```
- Tried using DHCP, but ran into some issue
- Because it is static IP, there's a bit more configuration on the client side too:
	- **IP Address:** `192.168.50.50` (Any number between 2 and 254 is fine)
	- **Gateway:** `192.168.50.1` (This is your Portenta's address)
	- **Network Prefix Length:** `24`
	- **DNS 1:** `8.8.8.8`
- Once the client is connected, we can share ROS messages across this interface


If something goes wrong, can always reset the network profile like this:
```bash
# Bring down any existing connection
sudo nmcli con down MyHotspot
# Delete any existing profile
sudo nmcli con delete MyHotspot
```

To see details about interfaces:
```bash
nmcli device
```