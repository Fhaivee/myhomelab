# myhomelab
this is a effort of me trying to document my current homelab

## 🛠️ Tech Stack
* **Hypervisor:** Proxmox VE (LXC Containers)
* **Networking:** Cloudflare Tunnel, ZeroTier (Self-Hosted Controller/ztncui), Headscale
* **Services:** Vaultwarden, Jellyfin

## 🚧 Key Configurations & Troubleshooting

### 1. Cloudflare Tunnel (Custom Systemd Service)
Sempat mengalami isu di mana `cloudflared` berjalan lancar secara manual tapi gagal saat dijalankan oleh systemd karena masalah permission user.

**Solusi:**
Membuat custom systemd service yang memaksa process berjalan sebagai `root` dan menggunakan path config absolut.

File: `/etc/systemd/system/cloudflared.service`
```ini
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
Type=simple
User=root
Group=root
# Perintah eksekusi manual yang dipindahkan ke service
ExecStart=/usr/local/bin/cloudflared --config /usr/local/etc/cloudflared/config.yml tunnel run proxmox-helper
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

```
### 2. Diagram Topologi

Gambaran:
```mermaid
graph TD
    %% Node Definitions
    Internet(("Internet / ISP"))
    RouterUtama["Router Utama\n(DHCP Server)"]
    RouterKamar["Router Kamar\n(Mode Access Point / Switch)"]
    
    subgraph "Server Proxmox (Node: serverprox)"
        Server["Physical Server\nIP: 192.168.1.36"]
        
        subgraph "LXC Containers"
            CT100("100: Jellyfin")
            CT101("101: Cloudflared\nTunnel Ingress")
            CT102("102: Headscale")
            CT103("103: Samba NAS")
            CT104("104: Vaultwarden")
            CT105("105: ZeroTier\nSubnet Router")
        end
    end

    %% Client Devices
    Laptop["Laptop / PC User"]
    HP["HP User"]

    %% Connections / Flow
    Internet <==>|WAN| RouterUtama
    RouterUtama <==>|"Kabel LAN"| RouterKamar
    RouterKamar <==>|"Kabel LAN"| Server
    
    %% Wireless Connections
    RouterKamar -.->|"WiFi"| Laptop
    RouterKamar -.->|"WiFi"| HP

    %% Internal Logic
    Server --- CT100 & CT101 & CT102 & CT103 & CT104 & CT105
    
    %% Tunnels (Logical)
    CT101 -.->|"Tunnel"| Internet
    CT105 -.->|"Virtual LAN"| Laptop & HP
