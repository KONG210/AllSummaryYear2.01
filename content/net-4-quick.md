# Ch 4 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- **Forwarding** = input → output port ใน router เดียว (data plane, hardware, ns) · **Routing** = วางเส้นทางทั้งหมด (control plane, software, ms)
- Control plane: **traditional** (ในทุก router, กระจายศูนย์) vs **SDN** (remote controller ติดตั้ง table, รวมศูนย์)
- Internet = **best effort**: ไม่รับประกัน bandwidth, loss, order, timing
- Subnet = interface ที่ถึงกันโดย **ไม่ผ่าน router**

## 🧮 Subnetting — สูตร
```text
IP ทั้งหมด = 2^(32 − prefix)       ใช้ได้ = ทั้งหมด − 2
mask octet สุดท้าย = 256 − block
network = floor(octet / block) × block
broadcast = network + block − 1
เลือก prefix: หา 2^k ≥ (เครื่อง + 2) → prefix = 32 − k
```
| /24 | /25 | /26 | /27 | /28 | /29 | /30 |
|---|---|---|---|---|---|---|
| .0 | .128 | .192 | .224 | .240 | .248 | .252 |
| 256/254 | 128/126 | 64/62 | 32/30 | 16/14 | 8/6 | 4/2 |

- **VLSM:** เรียงวงใหญ่ → เล็ก แล้วต่อกัน (Sales /25 .0, IT /26 .128, HR /27 .192, link /30 .224)
- ตัวอย่าง: 10.10.15.110/28 → block 16 → network .96, broadcast .111

## 🧮 Fragmentation — สูตร
```text
max data = floor((MTU − 20) / 8) × 8
offset   = byte เริ่มต้น / 8
total length = data + 20
MF = 1 ทุกชิ้น ยกเว้นชิ้นสุดท้าย (MF = 0) · ID เท่ากันทุกชิ้น
```
- MTU 1500 → data 1480 → offset 0, 185, 370
- MTU 1000, data 3200 → 976 ต่อชิ้น → 4 ชิ้น: offset 0, 122, 244, 366; total 996, 996, 996, 292
- **DF = 1** ห้ามหั่น → drop + ICMP error · ประกอบกลับ **ที่ปลายทางเท่านั้น**

## ⭐ ต้องจำ
- IPv4 header 20 bytes, TCP+IP overhead = 40 bytes, max datagram 64K, ปกติ ≤ 1500
- **TTL** ลด 1 ต่อ router (64 Linux/Mac, 128 Windows) กัน loop · **Protocol field** คงที่: ICMP 1, IGMP 2, TCP 6, UDP 17, GRE 47, OSPF 89
- Private: **10/8, 172.16/12 (172.16–172.31), 192.168/16** (RFC 1918) · Loopback 127.0.0.1
- Class A 1–127 /8 · B 128–191 /16 · C 192–223 /24 · D 224–239 multicast · E 240–255 experimental
- **DHCP**: Discover → Offer → Request → ACK · client **68**, server **67**, broadcast · ให้ IP + gateway + DNS + mask · 2 ขั้นแรกข้ามได้
- **NAT**: (src IP, port) → (NAT IP, new port) + จำในตาราง · ข้อดี: IP เดียว, เปลี่ยน IP ภายใน/ISP ได้, ปลอดภัย · ข้อถกเถียง: แตะ layer 4, ควรใช้ IPv6, ละเมิด end-end, NAT traversal
- **IPv6**: 128 bit, header คงที่ 40 bytes, **ไม่มี checksum / fragmentation / options**, มี flow label, priority, next hdr, hop limit · transition ด้วย **tunneling** (IPv6 ใน IPv4)
- **OpenFlow**: match (link/network/transport fields) + action (forward, drop, modify, send to controller) + stats (counters) · priority แก้ pattern ซ้อน
- Router = longest prefix → forward · Firewall = IP+port → permit/deny · Switch = dst MAC → forward/flood · NAT = IP+port → rewrite
- Middlebox (RFC 3234) · NFV · IP hourglass (thin waist = IP) · RFC 1958: intelligence at edge · end-end argument (Saltzer, Reed, Clark 1981)

## ⚠️ มักสับสน
- Network address หาร block ลงตัว · broadcast = วงถัดไป − 1
- อ่านโจทย์ fragmentation: ให้ "data" หรือ "total length"
- Offset หน่วย 8 bytes · data ต่อชิ้นต้องหาร 8 ลงตัว
- Static IP (server) vs Dynamic IP (DHCP)
- /30 = router-to-router (ใช้ได้ 2)
