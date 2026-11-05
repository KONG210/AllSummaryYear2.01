# Ch 5 · สรุปก่อนสอบ

> ⚠️ ไฟล์นี้ไม่มีสไลด์ Dijkstra / Distance Vector — เช็กกับอาจารย์ว่าออกคำนวณไหม

## 🧠 ต้องเข้าใจ
- Control plane 2 แบบ: **per-router (traditional)** vs **logically centralized (SDN)**
- Internet scale ไม่ได้ถ้า "แบน" → แบ่งเป็น **AS**: **intra-AS** (ใน AS, protocol เดียวกันทั้ง AS) + **inter-AS** (ระหว่าง AS ผ่าน **gateway**)
- Forwarding table = intra-AS (ปลายทางภายใน) + inter-AS & intra-AS (ปลายทางภายนอก)
- ทำไมแยก: **policy** (inter สำคัญ), **scale** (ลดขนาดตาราง), **performance** (intra เน้นได้, inter policy มาก่อน)

## ⭐ OSPF
- link-state, open, **flood LSA ตรงบน IP** ไปทั้ง AS, cost = bandwidth/delay, **Dijkstra**, ทุก message **authenticated**
- Hierarchical 2 ระดับ: **local area + backbone** · area border router **summarize** · boundary router ต่อ AS อื่น
- RIP = DV ทุก 30 วิ (เลิกใช้) · EIGRP = DV เคยเป็นของ Cisco · IS-IS ≈ OSPF

## ⭐ BGP
- de facto inter-domain, "glue of the Internet", **path vector**, peers ใช้ **TCP กึ่งถาวร**
- **eBGP** รับ/ประกาศกับ AS เพื่อนบ้าน · **iBGP** กระจายภายใน · gateway รันทั้งคู่
- Messages: **OPEN** (เปิด+authenticate) · **UPDATE** (ประกาศ/ถอน path) · **KEEPALIVE** (คงไว้ + ACK OPEN) · **NOTIFICATION** (error + ปิด)
- Route = prefix + attributes: **AS-PATH**, **NEXT-HOP** · import policy รับ/ปฏิเสธ
- **Hot potato**: เลือก gateway ที่ intra-domain cost ต่ำสุด ไม่สน inter-domain cost
- เลือก route: **1) local preference 2) shortest AS-PATH 3) closest NEXT-HOP 4) อื่น ๆ**
- Policy: provider ไม่ประกาศ path ที่ไม่ได้รายได้ (ไม่ขน transit) · customer dual-homed ไม่ประกาศ route ให้เป็นทางผ่าน

## ⭐ SDN
- ทำไม: ลด misconfiguration, flow ยืดหยุ่น, คำนวณตารางที่ศูนย์กลางง่ายกว่า, open → นวัตกรรม
- Traditional ทำไม่ได้: บังคับเส้นทางเฉพาะ (ต้องแก้ link weight), load balance แยก 2 ทาง, แยก traffic ตามสี
- 4 ลักษณะ: flow-based forwarding · แยก control/data plane · control อยู่นอก switch · programmable apps
- 3 ชั้น: apps ↔ **northbound API** ↔ **controller (network OS, distributed)** ↔ **southbound API (OpenFlow)** ↔ switches
- Controller: interface layer · network-wide state (distributed DB) · communication (OpenFlow, SNMP)
- OpenFlow ใช้ TCP · controller→switch: **features, configure, modify-state, packet-out** · switch→controller: **packet-in, flow-removed, port status**
- Link failure: port status → อัปเดต state → เรียก Dijkstra app → คำนวณ → flow table ใหม่ → ติดตั้งผ่าน OpenFlow
- ONOS: open-source, **intent** (what not how), distributed core · ORION: Google (Jupiter, B4)
- Mainframe→PC analogy: vertical/closed/slow → horizontal/open/rapid

## ⭐ ICMP
| Type/Code | ความหมาย |
|---|---|
| 0/0 | echo reply |
| 8/0 | echo request (ping) |
| 3/0, 3/1, 3/2, 3/3 | network / host / protocol / **port** unreachable |
| 11/0 | **TTL expired** |
| 4/0 | source quench (ไม่ใช้) |
- อยู่ใน IP datagram · message = type + code + 8 bytes แรกของ datagram ที่ error
- **Traceroute**: UDP TTL=1,2,3... (ชุดละ 3) → router ตอบ 11/0 → ถึงปลายทางตอบ **3/3** → หยุด

## ⭐ Network management
- Components: managing server, managed device + agent, data (config/operational/statistics), protocol
- CLI (ssh ทีละตัว) · SNMP/MIB · NETCONF/YANG (ทั้งเครือข่าย, YANG = modeling, NETCONF = ส่ง)
- SNMP 2 modes: **request/response** และ **trap**
- Get/GetNext/GetBulk, Set (manager→agent) · Response, **Trap** (agent→manager)
- MIB = ข้อมูลอุปกรณ์ (400 modules ใน RFC) · **SMI** = data definition language
