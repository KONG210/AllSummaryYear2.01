# Lab 10 · เช็กลิสต์ก่อนสอบ

> 📌 PT 10.4.3 · router **College** เชื่อม 2 LAN: G0/0 = 128.107.20.0/24 (2001:db8:a::/64) · G0/1 = 128.107.30.0/24 (2001:db8:b::/64)

- [ ] `enable` → `configure terminal` → `hostname College` (switch: `hostname Class-B`)
- [ ] `enable secret <รหัส>` (privileged แบบเข้ารหัส)
- [ ] `line console 0` → `password` → `login` · `line vty 0 4` (switch 0 15) → `password` → `login`
- [ ] `service password-encryption` · `banner motd #...#`
- [ ] `ipv6 unicast-routing`
- [ ] `interface g0/0` → `description` → `ip address 128.107.20.1 255.255.255.0` → `ipv6 address 2001:db8:a::1/64` → `ipv6 address fe80::1 link-local` → **`no shutdown`**
- [ ] `interface g0/1` → 128.107.30.1 /24 · 2001:db8:b::1/64 · fe80::1 link-local · **`no shutdown`**
- [ ] Class-B: `interface vlan 1` → `ip address 128.107.30.15 255.255.255.0` → `no shutdown` · `ip default-gateway 128.107.30.1`
- [ ] PC: IPv4 + mask 255.255.255.0 + gateway (.20.1 หรือ .30.1) · IPv6 /64 + gateway **FE80::1**
- [ ] `copy running-config startup-config`
- [ ] ตรวจ: `show ip interface brief` · `show ipv6 interface brief` · `ping`

⚠️ ping ไม่ผ่าน → ลืม `no shutdown` / IPv6 ข้ามวงไม่ได้ → ลืม `ipv6 unicast-routing` / ping switch ข้ามวงไม่ได้ → ลืม `ip default-gateway` · Class-A ไม่มี IPv6 (ping IPv6 ไม่ได้เป็นเรื่องปกติ)
