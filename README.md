# Konfigurasi Debian Server Sebagai Default Gateway / Router

Dokumentasi konfigurasi Debian GNU/Linux sebagai router menggunakan `iptables` (NAT/Masquerade) dan IP forwarding, dijalankan di VirtualBox.

## Topology

- **Debian Router**
  - `enp0s3` (WAN) → NAT, terhubung ke internet
  - `enp0s8` (LAN) → Internal Network, terhubung ke Client
- **Client**
  - `enp0s3` → Internal Network (sama dengan LAN Debian Router)

```
[ Internet ] -- NAT -- [ enp0s3 : Debian Router : enp0s8 ] -- Internal Network -- [ Client ]
```

## Konfigurasi Server

### 1. Install tools
```bash
apt update
apt install net-tools iptables
```

### 2. Konfigurasi Network Interfaces
```bash
nano /etc/network/interfaces
```

```
auto lo
iface lo inet loopback

# WAN - ke internet (NAT)
auto enp0s3
iface enp0s3 inet dhcp

# LAN - ke client
auto enp0s8
iface enp0s8 inet static
    address 192.168.20.6
    netmask 255.255.255.248
    network 192.168.20.0
```

Restart networking:
```bash
systemctl restart networking
```

### 3. Konfigurasi DNS
```bash
nano /etc/resolv.conf
```
```
nameserver 8.8.8.8
```

### 4. Aktifkan IP Forwarding
```bash
nano /etc/sysctl.conf
```
Uncomment baris berikut:
```
net.ipv4.ip_forward=1
```

### 5. Konfigurasi NAT (Masquerade) via rc.local
```bash
nano /etc/rc.local
```
```
#!/bin/sh -e

iptables -t nat -A POSTROUTING -j MASQUERADE
exit 0
```

Beri izin eksekusi & restart service:
```bash
chmod +x /etc/rc.local
systemctl restart rc-local
reboot
```

## Konfigurasi Client

```bash
nano /etc/network/interfaces
```
```
auto enp0s3
iface enp0s3 inet static
    address 192.168.20.2
    netmask 255.255.255.248
    network 192.168.20.0
    gateway 192.168.20.6
```

```bash
nano /etc/resolv.conf
```
```
nameserver 8.8.8.8
```

```bash
systemctl restart networking
```

## Hasil Pengujian

Ping dari Client ke LAN Debian Router (`192.168.20.6`) → **berhasil, 0% packet loss**

Ping dari Client ke `google.com` → **berhasil**, resolve ke `any-in-2678.1e100.net (216.239.38.120)`

```
64 bytes from any-in-2678.1e100.net (216.239.38.120): icmp_seq=1 ttl=254 time=22.8 ms
```

## Referensi

- https://rafimf.gitlab.io/notes/network/linux/konfigurasi-debian-sebagai-router/
