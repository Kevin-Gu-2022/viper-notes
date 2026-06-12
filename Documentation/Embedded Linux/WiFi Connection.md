>[!Note]
>As long as both devices are on same WiFi network, they should be able to communicate, though setting up hotspot would reduce hops, i.e. latency

- Setup a static-IP WiFi hotspot on the Pika Spark: 

```bash
# Create the new connection
sudo nmcli con add type wifi ifname wlan0 con-name MyHotspot autoconnect yes ssid PikaSpark mode ap

# Set to manual IP (no DHCP server)
sudo nmcli con modify MyHotspot ipv4.method manual ipv4.addresses 192.168.50.1/24

# Set Security
sudo nmcli con modify MyHotspot wifi-sec.key-mgmt wpa-psk
# Set password
sudo nmcli con modify MyHotspot wifi-sec.psk "Password,1"

# Bring the connection up. Similarly, 'down' to shut hotspot down. Should automatically switch to previous WiFi
sudo nmcli con up MyHotspot
```
- Tried using DHCP, but ran into some issue
- Because it is static IP, there's a bit more configuration on the client side too:
	- **IP Address:** `192.168.50.50` (Any number between 2 and 254 is fine)
	- **Gateway:** `192.168.50.1` (This is your Portenta's address)
	- **Network Prefix Length / Netmask:** `24`
	- **DNS 1:** `8.8.8.8`
- On Ubuntu, the only way to access settings page in GUI seems to require entering password first
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

- If computer and Pika Spark are on same LAN network, then they will be able to share messages
- If Pika Spark's hotspot is turned on, and computer is not connected, they won't be able to share messages
- Obviously connecting directly to Pika Spark will give better latency, as less hops across network
# Troubleshooting
- List out topics on the 2 machines to see of they are communicating
```bash
ros2 topic list
```

- If errors persist, could try explicitly setting the domain ID to be same on both machines:
```bash
export ROS_DOMAIN_ID=0
```

- Time may go out of sync when switch back to WiFi. Check with `date`. Force synchronisation with `timedatectl` command, then `reboot`
- Remember, no WiFi when hotspot is on. This means no package downloads