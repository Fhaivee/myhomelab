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
