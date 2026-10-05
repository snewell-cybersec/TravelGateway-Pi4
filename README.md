# TravelGateway-Pi4 ✈️🛡️

A portable, ad-blocking travel router and secure network gateway built on a **Raspberry Pi 4 (2GB RAM)**. This project establishes an encrypted **WireGuard VPN tunnel** back to a home gateway server, blocks ads and trackers network-wide using **Pi-hole**, and hosts a local **Samba file share** for wireless photo backups.

---

## 🛠️ Hardware Stack
* **SBC:** Raspberry Pi 4 Model B (2GB RAM)
* **OS Storage:** SanDisk Ultra 32GB MicroSD Card
* **Power:** Anker PowerPort C 2 Block + USB-C to USB-C cable (5V/3A 15W min)
* **Uplink:** RJ45 Cat6 Ethernet Cable
* **USB Reader:** USB 3.0 MicroSD Card Reader (For flashing)
* **Wireless Antenna:** BrosTrend AX900 Wi-Fi 6 Adapter (MediaTek MT7921AU Chipset)
* **Cooling:** Easycargo 30mm DC 5V Brushless Fan + Aluminum Heatsinks

---

## 🔌 Hardware Setup & Cooling (Silent 3.3V Profile)
To maximize fan lifespan and keep the unit silent in quiet hotel environments, the cooling fan is under-volted to the **3.3V GPIO pins** rather than the full 5V rail.

* 🔴 **Red Wire (Power):** Connected to **Pin 1** (3.3V Power)
* ⚫ **Black Wire (Ground):** Connected to **Pin 6** (Ground)

### Enclosure Notes
* Aluminum heatsinks are applied directly to the CPU, RAM, and USB controller chips.
* The 30mm fan is mounted to the inside ceiling of the custom 3D-printed case, configured to blow cool outside air downward onto the heatsinks. 
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
2. SSH into the Pi over your local network. (The MediaTek MT7921AU chipset is natively supported by the Linux kernel; no driver installation required).
3. Run the automated RaspAP installer:
   ```bash
   curl -sL https://install.raspap.com | bash
   ```
4. Reboot the Pi and access the web dashboard. Configure the external USB antenna (`wlan1`) as your WAN interface to pull in public Wi-Fi, and the internal Wi-Fi chip (`wlan0`) to broadcast your private hotspot.

### Phase 3: Ad-Blocking Integration (Pi-hole)
1. Install Pi-hole alongside RaspAP by running:
   ```bash
   curl -sSL https://pi-hole.net | bash
   ```
2. Bind the static network profile to your active RaspAP interface.
3. Configure RaspAP's DHCP server settings to force all connected client devices to use the local loopback address (`127.0.0.1`) as their primary upstream DNS server.

### Phase 4: WireGuard VPN Tunnel
1. Access your home network's WireGuard VPN server and generate a new client peer configuration profile.
2. Export the configuration file data as a plain-text `.conf` profile.
3. Navigate to the VPN Client module in the RaspAP dashboard, select the WireGuard protocol engine, and paste the config file data.
4. Enable the connection on startup to ensure all travel traffic tunnels securely through your home network.

### Phase 5: Local Samba Photo Share
1. Connect a high-capacity external USB drive to the remaining blue USB 3.0 port.
2. Install Samba via the terminal:
   ```bash
   sudo apt-get update && sudo apt-get install samba samba-common -y
   ```
3. Configure an encrypted network share to let connected travel devices wirelessly back up photos and videos locally over the private Wi-Fi network.

