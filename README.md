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
2. SSH into the Pi over your local network. Requires initializing the external driver using the manufacturer installer script:
```bash
sh -c 'wget linux.brostrend.com/install -O /tmp/install && sh /tmp/install'
```
4. Run the automated RaspAP installer:
   ```bash
   curl -sL https://install.raspap.com | bash
   ```
5. Reboot the Pi and access the web dashboard. Configure the external USB antenna (`wlan1`) as your WAN interface to pull in public Wi-Fi, and the internal Wi-Fi chip (`wlan0`) to broadcast your private hotspot.

### Phase 3: Ad-Blocking Integration (Pi-hole)
1. Install the Docker engine runtime components:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

2. Create a dedicated environment directory for persistent configuration data:
```bash
mkdir ~/pihole && cd ~/pihole
```

3. Create a service layout profile:
```bash
nano docker-compose.yml
```

4. Paste the following composition parameters (this maps DNS natively but routes the web dashboard to port `8080`):
```yaml
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8080:80/tcp"
    environment:
      TZ: 'America/Chicago'
      WEBPASSWORD: 'ChooseSecurePasswordHere'
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

5. Launch the isolated container in detached background execution mode:
```bash
sudo docker compose up -d
```

6. Link RaspAP to the new ad-shield:
   * Access the RaspAP admin dashboard (`http://10.3.141.1.1:8080/admin`).
   * Navigate to **DHCP Server** settings.
   * Change the primary upstream DNS server pushed to travel clients to the loopback address (`127.0.0.1`).
   * Open the standalone Pi-hole control dashboard interface at `http://10.3.141.1:8080/admin` using your configured environment password.


### Phase 4: WireGuard VPN Tunnel
1. Access your home network's WireGuard VPN server and generate a new client peer configuration profile.
2. Export the configuration file data as a plain-text `.conf` profile.
3. Navigate to the VPN Client module in the RaspAP dashboard, select the WireGuard protocol engine, and paste the config file data.
4. Enable the connection on startup to ensure all travel traffic tunnels securely through your home network.
