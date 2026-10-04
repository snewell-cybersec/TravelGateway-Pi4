# TravelGateway-Pi4 ✈️🛡️

A hardened, ad-blocking travel router and secure network gateway built on a **Raspberry Pi 4 (2GB RAM)**. This project establishes an encrypted **WireGuard VPN tunnel** back to a home **GL.iNet Flint 2 router**, sanitizes traffic using a network-wide **Pi-hole ad blocker**, and provides local wireless data storage via a **Samba Photo Vault**.

---

## 🛠️ Hardware Stack & Inventory

### Core Components
*   **Single Board Computer:** Raspberry Pi 4 Model B (2GB RAM Variant)
*   **System Storage:** SanDisk Ultra 32GB MicroSD Card
*   **Power Supply:** Anker PowerPort C 2 Block (Dual USB-C Interface)
*   **Power Conduit:** Pure USB-C to USB-C Cable (Rated 5V/3A 15W Minimum)
*   **Primary Uplink:** Standard RJ45 Cat6 Ethernet Cable

### Amazon Deployment Upgrades
*   **Flashing Interface:** Wansurs USB 3.0 MicroSD Card Reader (5Gbps Adapter)
*   **External Network Array:** BrosTrend AX900 Linux Wi-Fi 6 Adapter (Model AX1L with 6dBi High-Gain Antenna / MediaTek MT7921AU Chipset)
*   **Thermal Management:** Easycargo 30mm DC 5V Brushless Fan + Aluminum Heatsinks

---

## 🔌 Hardware Architecture & Thermal Layout

### Fan Configuration (Silent 3.3V Profile)
To maximize fan lifespan and eliminate high-pitched acoustics in quiet hotel environments, the cooling fan is wired to the **3.3V rails** rather than full 5V power.

*   🔴 **Red Wire (Power):** Connected to **Pin 1** (3.3V Power - Inside Row Corner)
*   ⚫ **Black Wire (Ground):** Connected to **Pin 6** (Ground - Outside Row, Third Pin Down)

### Structural Assembly Notes
*   **Heatsinks:** Chemically bonded via thermal adhesive pads to the CPU (Silver SoC), RAM chip, and USB controller chip.
*   **Chassis Integration:** The 30mm fan is mounted via machine screws to the interior ceiling of the custom 3D-printed enclosure, configured to exhaust/intake air downward over the top heatsinks. The chassis lid is permanently chemically welded (glued) shut following initial hardware validation.

---

## 🚀 Execution & Configuration Roadmap

### Phase 1: Operating System Initialization
1.  Isolate the **32GB MicroSD card** and mount it inside the **Wansurs USB Card Reader**.
2.  Launch the **Raspberry Pi Imager** tool on the Windows host computer.
3.  Target the underlying device profile as **Raspberry Pi 4**.
4.  Select the **Raspberry Pi OS Lite (64-bit)** operating system image.
5.  Open Advanced Customization Settings (Gear Icon) to explicitly enforce:
    *   Target SSH Server Initialization (`Enabled`)
    *   System Administrative Credentials (`Username` & `Password`)
    *   Local Time Zone Mapping
6.  Execute the formatting and image deployment pipeline (`Write`).

### Phase 2: Core Routing Deployment (RaspAP)
1.  Slide the flashed MicroSD card into the Pi 4 slot. Connect the **BrosTrend AX900 Antenna** to a blue USB 3.0 port. 
2.  Attach the **Ethernet cable** directly between the Pi and your home router. Boot the machine using the **Anker power block**.
3.  Because the BrosTrend AX900 relies on a native MediaTek MT7921AU chipset, the core Linux kernel will automatically recognize and activate the adapter without requiring manual driver installations.
4.  Establish an active SSH terminal connection into the Pi over the local area network.
5.  Execute the automated deployment script for the routing dashboard:
    ```bash
    curl -sL https://raspap.com | bash
    ```
6.  Reboot the system and verify local access to the RaspAP web control portal. Configure the **BrosTrend USB Interface (wlan1)** as the primary WAN client (pulling public internet) and the **Internal Wi-Fi chip (wlan0)** as the private LAN hotspot manager (*"MyTravelWiFi"*).

### Phase 3: Traffic Sanitation Deployment (Pi-hole)
1.  Execute the automated network-wide ad-blocking deployment command inside the terminal:
    ```bash
    curl -sSL https://pi-hole.net | bash
    ```
2.  Configure the static network profile to bind safely alongside the active RaspAP interface.
3.  Map RaspAP’s localized DHCP server pool settings to enforce the Pi-hole local loopback IP address (`127.0.0.1`) as the primary upstream DNS server for all connected client devices.

### Phase 4: Encrypted Tunnel Integration (Home Flint 2 VPN)
1.  Authenticate into the administrative control interface of your home **GL.iNet Flint 2 Router**.
2.  Access the **VPN Panel** ➔ **WireGuard Server** workspace and trigger activation.
3.  Generate an independent, isolated client node profile labeled **`TravelPi`**.
4.  Export the resulting cryptographic configuration layout as a plain-text `.conf` file structure.
5.  Return to the **RaspAP Web Portal** on the travel device, navigate to the **VPN Client Profile** module, select the **WireGuard** protocol engine, and upload or paste the target server data.
6.  Enable automatic connectivity tracking blocks to ensure the encrypted tunnel re-initializes natively on device startup.

### Phase 5: Storage Vault Extension (Samba Photo Share)
1.  Connect an auxiliary high-capacity external storage drive to the remaining blue USB 3.0 hardware interface.
2.  Initialize and configure the **Samba (SMB) protocol engine** via the terminal line:
    ```bash
    sudo apt-get install samba samba-common-bin -y
    ```
3.  Expose an encrypted system directory structure allowing travel companions to securely offload photographic assets and media volumes locally to the drive over the private encrypted Wi-Fi link.
