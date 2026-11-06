# Lab 12 · เช็กลิสต์ก่อนสอบ

> 📌 PT 13.2.7 · R1 — R2 — R3 (serial) · ลิงก์ R1–R2 = 10.10.1.4/30 · R2–R3 = 10.10.1.8/30 · PC1 10.10.1.98/27 gw .97 · PC3 10.10.1.18/28 gw .17 · PC2 gw fe80::1 · PC4 gw fe80::3

- **ping** = ถึงไหม (ไม่บอกตรงไหน) · **tracert/traceroute** = hop สุดท้ายที่ตอบ = จุดก่อนขาด
- **IPv4 fault:** trace PC1 หยุด 10.10.1.97, PC3 หยุด 10.10.1.17 → **R2 S0/0/0 = 10.10.1.2 ผิดวง** → แก้เป็น **10.10.1.5 255.255.255.252**
- **IPv6 fault:** PC2 ไปถึง 2001:db8:1:3::2 (R3) แต่ PC4 ไม่ถึงสักที่ → **PC4 gateway ผิด** → แก้เป็น **fe80::3**
- `show ip route`: **C** = connected, **L** = local /32 (R1 S0/0/1: 10.10.1.4/30 + 10.10.1.6/32)
- คำสั่ง: `ipconfig /all`, `ipv6config /all`, `tracert`, `show ip interface brief`, `show ip route`, `show ipv6 interface brief`, Ctrl+C หยุด trace
- mask: /27 = .224 · /28 = .240 · /30 = .252
