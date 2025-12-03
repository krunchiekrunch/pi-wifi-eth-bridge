# Step 1: Setup Connection

# Requirements
- Internet access on a Wi-Fi interface
- Ethernet interface
- Basic understanding of the command line

### Tested on Raspberry Pi 3B running Raspberry Pi OS Bookworm 64-bit

## CLI Method

This sets IPv6 to ignore, you can change it to `auto` if you want it to be automatic

You will need to replace `yourEthernetInterface` with the Ethernet interface that you are using for forwarding

```
sudo nmcli connection add type ethernet con-name wifibridge ifname yourEthernetInterface ipv4.method shared ipv6.method ignore connection.autoconnect yes connection.autoconnect-priority 0
```

## GUI Method

Start by entering the NetworkManager connection settings page

<img width="622" height="385" alt="image" src="https://github.com/user-attachments/assets/aca1dd7f-b397-4726-a999-1c41a8bc47fa" />

Press "Edit Connections" under "Advance Options"

Create a new connection and choose Ethernet as the type

<img width="542" height="394" alt="image" src="https://github.com/user-attachments/assets/8095cbb1-476d-4bf9-a538-81059770b1f2" />

Set it to auto connect in the General tab

<img width="635" height="76" alt="image" src="https://github.com/user-attachments/assets/7d3df156-8eae-4dfe-a4ce-53dedf5b4014" />

Choose the interface name of the Ethernet interface in Ethernet tab (The interface name for your device many be different)

<img width="421" height="82" alt="image" src="https://github.com/user-attachments/assets/bb6d7e8c-1851-4c39-beaf-b3002c234593" />

In IPv4 settings, set the method to "Share to other computers"

<img width="641" height="97" alt="image" src="https://github.com/user-attachments/assets/3e67b0c7-9429-4301-aa2b-37128fd23d52" />

For IPv6 settings, you can leave the method to "Automatic" or you can change it to "Ignore"

<img width="646" height="83" alt="image" src="https://github.com/user-attachments/assets/b657893b-aa1d-4f87-a313-893989cb366b" />

# Step 2: NAT and forwarding rule

Next, we need to enable NAT and allow forwarding from Ethernet to the Internet

Open your terminal of choice and run

```
sudo iptables -t nat -A POSTROUTING -s 10.42.0.0/24 -o yourWifiInterfaceWithInternet -j MASQUERADE
```

Make sure you replace `yourWifiInterfaceWithInternet` with the name of your Wi-Fi interface (usually wlan0)

And then run this to allow forwarding

```
sudo iptables -A FORWARD -i yourEthernetInterface -o yourWifiInterfaceWithInternet -j ACCEPT
sudo iptables -A FORWARD -i yourWifiInterfaceWithInternet -o yourEthernetInterface-m state --state RELATED,ESTABLISHED -j ACCEPT
```

Again, replace `yourEthernetInterface` and `yourWifiInterfaceWithInternet` with the correct values.

# Step 3: Persist NAT and rules configuration

iptables config will normally reset once the device is rebooted, however, you can make it persist by installing `iptables-persistent`

```
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

# Step 4: Router Configuration (Optional)

If you are connecting the Ethernet to a router, you need to make sure the gateway address and DHCP address range from the router doesn't conflict with the router that your Raspberry Pi's Wi-Fi is connected to.

For example:

<img width="270" height="480" alt="image" src="https://github.com/user-attachments/assets/78cd8bab-0007-4d53-b59f-b981efd6533a" />

# Note

You can change the forwarding input and output by simply changing the name of the network interface, for example, 4G modem to Ethernet, or Ethernet to Wi-Fi

If you are trying to forward Wi-Fi to a hotspot, you will need 2 Wi-Fi interface as one interface cannot do both at the same time
