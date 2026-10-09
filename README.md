# TravelGateway-Pi4 ✈️🛡️

A portable, ad-blocking travel router and secure network gateway built on a **Raspberry Pi 4 (2GB RAM)**. This project establishes an encrypted **WireGuard VPN tunnel** back to a home gateway server.

---

## 🛠️ Hardware Stack
* **SBC:** Raspberry Pi 4 Model B (2GB RAM)
* **OS Storage:** SanDisk Ultra 32GB MicroSD Card
* **Power:** Anker PowerPort C 2 Block + USB-C to USB-C cable (5V/3A 15W min)
* **Uplink:** RJ45 Cat6 Ethernet Cable
* **USB Reader:** USB 3.0 MicroSD Card Reader (For flashing)
* **Wireless Antenna:** BrosTrend AX900 Wi-Fi 6 Adapter (MediaTek MT7921AU Chipset)
* **Cooling:** 40mm DC 5V Brushless Fan
---

## 🔌 Hardware Setup & Cooling (Silent 3.3V Profile)
To maximize fan lifespan and keep the unit silent in quiet hotel environments, the cooling fan is under-volted to the **3.3V GPIO pins** rather than the full 5V rail.

* 🔴 **Red Wire (Power):** Connected to **Pin 1** (3.3V Power)
* ⚫ **Black Wire (Ground):** Connected to **Pin 6** (Ground)

### Enclosure Notes
* The 40mm fan is mounted to the inside ceiling of the custom 3D-printed case, configured to blow cool outside air downward onto the heatsinks. 
* The case lid is permanently glued shut following hardware verification.

---

## 🚀 Deployment Guide

### Phase 1: OS Installation
1. Insert the 32GB MicroSD card into the USB card reader and plug it into your computer.
2. Open **Raspberry Pi Imager**, select **Raspberry Pi 4**, and choose **Raspberry Pi OS Lite (64-bit)**.
3. Open the Advanced Settings (Gear Icon) and configure:
   * Enable SSH
   * Set secure username and password
   * Set local time zone
4. Flash the image to the card.

### Phase 2: Travel Router Setup (RaspAP)
1. Insert the card into the Pi 4, connect the BrosTrend AX900 antenna to a blue USB 3.0 port, plug in the Ethernet cable to your home router, and power it up.
2. SSH into the Pi over your local network. Requires initializing the external driver using the manufacturer installer script:
```bash
sh -c 'wget linux.brostrend.com/install -O /tmp/install && sh /tmp/install'
```
3. Run the automated RaspAP installer:
   ```bash
   curl -sL https://install.raspap.com | bash
   ```
4. Reboot the Pi and access the web dashboard. Configure the external USB antenna (`wlan1`) as your WAN interface to pull in public Wi-Fi, and the internal Wi-Fi chip (`wlan0`) to broadcast your private hotspot.

### Phase 3: Native Ad-Blocking Integration (Pi-hole v6.0+)
Because RaspAP's administration interface claims port 80 by default, we deploy Pi-hole natively and shift its new embedded FTL web server engine to port 8080 to prevent system service conflicts.

1. Run the official Pi-hole core network installer script:
   ```bash
   curl -sSL https://install.pi-hole.net | bash
   ```
2. Step through the interactive setup menus. When prompted for the network interface, choose `wlan0` (your local hotspot interface) so it can capture incoming client traffic. Select your preferred upstream DNS providers, and note down the administrator dashboard password displayed on the final completion window.
3. Once the installer finishes execution, open the primary configuration file for Pi-hole v6's internal server engine:
   ```bash
   sudo nano /etc/pihole/pihole.toml
   ```
4. Look for the `[webserver]` block, locate the port parameter assignment, and change it to match the configuration below (if the parameter is missing, append it under the webserver heading):
   ```toml
   [webserver]
   port = "8080"
   ```
5. Save and close the file (`Ctrl+O`, `Enter`, `Ctrl+X`), then restart the FTL subsystem engine to release port 80 and apply your changes:
   ```bash
   sudo systemctl restart pihole-FTL
   ```
6. **Link RaspAP to your new ad-shield:** Access the RaspAP admin interface at `http://10.3.141.1`. Navigate to **DHCP Server settings**. Change the primary upstream DNS server pushed out to your travel clients to your hotspot gateway interface IP address: `10.3.141.1`.
7. You can now safely manage your blocklists, view metrics, and access the standalone control interface by visiting: `http://10.3.141.1:8080/admin`.

### Phase 4: WireGuard VPN Tunnel
1. Access your home network's WireGuard VPN server and generate a new client peer configuration profile.
2. Export the configuration file data as a plain-text `.conf` profile.
3. Navigate to the VPN Client module in the RaspAP dashboard, select the WireGuard protocol engine, and paste the config file data.
4. Enable the connection on startup to ensure all travel traffic tunnels securely through your home network.

## 📷 Lab Verification Screenshots

### Active Routing & Security Dashboards
![Pi-hole v6 Ad-Blocking Metrics Dashboard](Images/PiHole%20Dashboard.png)

![RaspAP Active Hotspot Management Panel](Images/RaspAP%20Dashboard.png)

### Assembled Hardware Deployment Node
![TravelGateway-Pi4 Final Hardware Build Configuration](Images/Raspberry%20Pi.jpeg)




