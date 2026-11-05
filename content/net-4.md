# Chapter 4 · Network Layer: Data Plane

## ภาพรวมของบท

**เป้าหมาย**
- เข้าใจหลักการของบริการชั้น network โดยเน้น **data plane**: service model, **forwarding vs routing**, **addressing**, **generalized forwarding**, Internet architecture
- ดูการใช้งานจริงบน Internet: **IP protocol**, **NAT**, **middleboxes** (firewall, load balancer)
- หัวใจสำคัญคือการสร้างและระบุที่อยู่ของแพ็กเก็ต (Addressing, Datagram Format) และการส่งต่อข้อมูลบน Internet (IPv4, IPv6)

**Roadmap:** Overview (data plane / control plane) → IP (datagram format, addressing, NAT, IPv6) → Generalized Forwarding (match+action, OpenFlow, middleboxes) → Internet architecture

> 🔗 ภาพใหญ่: network layer ย้าย datagram จากต้นทางไปปลายทาง โดยแต่ละ router ทำ **forwarding** (data plane, บทนี้) ตามตารางที่ได้จาก **routing** (control plane, บทที่ 5) — การจะส่งได้ต้องมี **ที่อยู่ IP** (ต้องคำนวณ subnet เป็น) ซึ่งได้จาก **DHCP**, แชร์ IP เดียวกันด้วย **NAT**, และกำลังเปลี่ยนไป **IPv6**

---

## 1. Network-layer services and protocols

- ขนส่ง **segment** จาก host ต้นทางไป host ปลายทาง
  - **ผู้ส่ง:** ห่อ (encapsulate) segment เป็น **datagram** แล้วส่งให้ link layer
  - **ผู้รับ:** ส่ง segment ขึ้นไปให้ transport layer
- network layer protocol อยู่ใน **ทุกอุปกรณ์ Internet**: host และ router
- **router:**
  - ตรวจ **header field ของทุก IP datagram** ที่ผ่าน
  - ย้าย datagram จาก **input port ไป output port** เพื่อส่งต่อไปตามเส้นทาง end-end

### 🔹 Two key network-layer functions

| Function | ความหมาย | เปรียบเทียบการเดินทาง |
|---|---|---|
| **Forwarding** | ย้าย packet จาก **input link ของ router ไปยัง output link ที่เหมาะสม** | การผ่าน **ทางแยกเดียว** |
| **Routing** | หา **เส้นทางทั้งหมด** ที่ packet จะไปจากต้นทางถึงปลายทาง (routing algorithms) | การ **วางแผนการเดินทาง** ทั้งทริป |

### 🔹 Data plane vs Control plane

| | Data plane | Control plane |
|---|---|---|
| ขอบเขต | **local, per-router** (ภายในแต่ละ router) | **network-wide logic** (ภาพรวมทั้งเครือข่าย) |
| หน้าที่ | ตัดสินว่า datagram ที่เข้า input port จะออก output port ไหน (forwarding) ดูจาก **ค่าใน header** | ตัดสินว่า datagram จะถูก route ผ่าน router ไหนบ้างจาก host ต้นทางถึงปลายทาง |

**Control plane มี 2 แนวทาง**
1. **Traditional routing algorithms** — ทำงาน **ในตัว router แต่ละเครื่อง** (per-router control plane: routing algorithm ของแต่ละ router คุยกันเอง)
   - เหมาะกับงานที่ต้องการ **ความเสถียร ระบบกระจายศูนย์** และการเชื่อมต่อข้ามองค์กรที่ไม่พึ่งระบบกลาง
2. **Software-Defined Networking (SDN)** — ทำงานที่ **remote server** (remote controller คำนวณแล้ว **ติดตั้ง forwarding table** ลงใน router ผ่าน CA)
   - เหมาะกับงานที่ต้องการ **ควบคุมจากศูนย์กลาง** ปรับเส้นทางแบบ dynamic และจัดการอัตโนมัติผ่านซอฟต์แวร์

---

## 2. Network-layer service model

**คำถาม:** network layer รับประกัน Quality of Service (QoS) อะไรบ้าง?

| Architecture | Service Model | Bandwidth | Loss | Order | Timing |
|---|---|---|---|---|---|
| **Internet** | **best effort** | none | no | no | no |
| ATM | Constant Bit Rate | Constant rate | yes | yes | yes |
| ATM | Available Bit Rate | Guaranteed min | no | yes | no |
| Internet | Intserv Guaranteed (RFC 1633) | yes | yes | yes | yes |
| Internet | Diffserv (RFC 2475) | possible | possibly | possibly | no |

**Internet "best effort" ไม่รับประกัน:**
1. ว่าข้อมูลจะ **ส่งถึงปลายทางสำเร็จ**
2. **เวลาหรือลำดับ** การส่ง
3. **bandwidth** ระหว่างต้นทางถึงปลายทาง

### 🔹 Reflections on best-effort (ทำไมยังใช้ได้ดี)
- **กลไกที่เรียบง่าย** ทำให้ Internet ขยายตัวและถูกใช้งานแพร่หลายทั่วโลก
- **จัดสรร bandwidth ให้เพียงพอ** → แอป real-time (เสียง, วิดีโอ) ดี "พอ" เกือบตลอดเวลา
- **บริการกระจายระดับ application** (datacenter, CDN) ที่ **คัดลอกข้อมูล/สำเนา server ไปวางใกล้ผู้ใช้** ทำให้ให้บริการจากหลายที่ได้
- **congestion control** ของบริการ "elastic" ช่วย
- "It's hard to argue with success of best-effort service model"

---

## 3. Router architecture overview

| ส่วน | ทำงานด้วย | ความเร็ว | หน้าที่ |
|---|---|---|---|
| **Routing processor** (control plane) | **software** / CPU | **millisecond** (ถึงวินาที) | คำนวณเส้นทาง อัปเดต routing table, management |
| **Forwarding** (data plane) | **hardware** | **nanosecond** | ส่งต่อให้ทัน line rate ของสายส่ง |

ส่วนประกอบ: **router input ports** → **high-speed switching fabric** → **router output ports** + routing processor

> ⭐ data plane = ฮาร์ดแวร์ เร็วระดับ ns · control plane = ซอฟต์แวร์ ช้ากว่า ระดับ ms

---

## 4. Network Layer ของ Internet

ภายใน network layer ของ host/router มี 3 ส่วน:
1. **Path-selection algorithms** — routing protocols (**OSPF, BGP**) หรือ **SDN controller** → สร้าง **forwarding table**
2. **IP protocol** — datagram format, addressing, packet handling conventions
3. **ICMP protocol** — error reporting, router "signaling"

อยู่ระหว่าง transport layer (TCP, UDP) และ link layer

---

## 5. IP Datagram format (IPv4)

```text
 0                   16                  31   (32 bits ต่อแถว)
+------+-------+---------+---------------------+
| ver  | head. | type of |    total length     |
|      |  len  | service |      (bytes)        |
+------+-------+---------+------+--------------+
|  16-bit identifier     | flgs | fragment     |
|                        |      | offset       |
+--------------+---------+------+--------------+
| time to live | upper   |   header checksum   |
|    (TTL)     | layer   |                     |
+--------------+---------+---------------------+
|          32-bit source IP address            |
+----------------------------------------------+
|        32-bit destination IP address         |
+----------------------------------------------+
|   options (if any) e.g. timestamp, record    |
+----------------------------------------------+
|  payload data (variable length, typically    |
|           a TCP or UDP segment)              |
+----------------------------------------------+
```

| Field | ความหมาย |
|---|---|
| **ver** | หมายเลขเวอร์ชันของ IP |
| **head. len** | ความยาว header (bytes) |
| **type of service** | diffserv (bit 0:5), **ECN** (bit 6:7) |
| **total length** | ความยาว datagram ทั้งหมด (bytes) — **สูงสุด 64K bytes**, ปกติ **≤ 1500 bytes** |
| **identifier, flags, fragment offset** | ใช้กับ **fragmentation/reassembly** |
| **TTL** | จำนวน hop ที่เหลือ — **ลดลง 1 ทุก router** |
| **upper layer** | protocol ชั้นบน (TCP/UDP) ที่จะส่ง payload ให้ |
| **header checksum** | ตรวจความถูกต้องของ header |
| **source / dest IP** | ที่อยู่ 32 bit |
| **options** | เช่น timestamp, record route |

> 🧮 **Overhead:** TCP header 20 bytes + IP header 20 bytes = **40 bytes** + overhead ของ app layer

### 🔹 Fields สำหรับ fragmentation

**1. Identification (IP ID) — 16 bits**
- เหมือน "หมายเลขบัตรคิว" ของแพ็กเก็ต IP ต้นฉบับ
- ผู้ส่งกำหนดเลข ID (เช่น 12345) ถ้า router หั่นเป็น 3 ชิ้น **ทุกชิ้นมี ID เดียวกัน (12345)**
- ปลายทางใช้ ID **จับกลุ่มชิ้นที่เป็นของแพ็กเก็ตเดียวกัน** ก่อนประกอบกลับ

**2. Flags — 3 bits** (ใช้จริง 2 bit)
| Flag | 1 | 0 |
|---|---|---|
| **DF** (Don't Fragment) | **ห้ามหั่น** — ถ้าใหญ่เกิน MTU router **drop แล้วส่ง ICMP error กลับ** | ยอมให้หั่นได้ |
| **MF** (More Fragments) | **ยังมีชิ้นต่อ** (ชิ้นนี้ไม่ใช่ชิ้นสุดท้าย) | **ชิ้นสุดท้ายแล้ว** (หรือไม่เคยถูกหั่น) |

**3. Fragment Offset — 13 bits**
- บอก **ตำแหน่ง** ของข้อมูลชิ้นนั้นในแพ็กเก็ตต้นฉบับ
- มีแค่ 13 bits แต่ IP ใหญ่ได้ถึง 65,535 bytes → มาตรฐานจึงกำหนดให้ **offset นับเป็นหน่วย 8 bytes** (ข้อมูลแต่ละชิ้นต้องหาร 8 ลงตัว)

**การประกอบกลับที่ปลายทาง**
1. ดู **IP ID** รวบรวมชิ้นชุดเดียวกัน
2. ดู **Fragment Offset** เรียงลำดับจากหน้าไปหลัง (กันปัญหาชิ้นมาสลับลำดับ)
3. ดู **MF** — ได้ชิ้นที่ **MF = 0** แสดงว่าได้ชิ้นสุดท้ายแล้ว → ส่งต่อให้ Layer 4 (TCP/UDP)

### 🔹 IP fragmentation/reassembly
- แต่ละ link มี **MTU** (max transfer size) = ขนาด frame ใหญ่สุดของ link นั้น; link ต่างชนิด MTU ต่างกัน
- datagram ใหญ่ถูก **หั่น (fragment)** ภายในเครือข่าย: datagram เดียวกลายเป็นหลายอัน
- **ประกอบกลับ (reassemble) เฉพาะที่ปลายทาง** เท่านั้น
- ใช้ bit ใน IP header ระบุและเรียงลำดับชิ้น

### 🧮 ตัวอย่างที่ 1: 4000-byte datagram, MTU = 1500

**ขั้นตอนคิด**
1. data ต่อชิ้นสูงสุด = MTU − header = 1500 − 20 = **1480** (1480/8 = 185 ลงตัว ✓)
2. หั่นทีละ 1480 → offset ของชิ้น = byte เริ่มต้น / 8

**แบบสไลด์ภาษาไทย (data payload = 4000 bytes)**
| ชิ้น | ช่วง data | data | MF | offset |
|---|---|---|---|---|
| 1 | byte 0–1479 | 1480 | **1** | 0/8 = **0** |
| 2 | byte 1480–2959 | 1480 | **1** | 1480/8 = **185** |
| 3 | byte 2960–3999 | 1040 | **0** | 2960/8 = **370** |

**แบบตารางหนังสือ Kurose (สไลด์ 4-88: datagram length = 4000 รวม header)**
| ชิ้น | length (total) | ID | fragflag (MF) | offset |
|---|---|---|---|---|
| ต้นฉบับ | 4000 | x | 0 | 0 |
| 1 | 1500 | x | **1** | 0 |
| 2 | 1500 | x | **1** | **185** |
| 3 | 1040 | x | **0** | **370** |

> ⚠️ สองแบบนี้ให้ offset และ MF เหมือนกัน ต่างกันแค่ชิ้นสุดท้าย: ถ้า **4000 คือ data** ชิ้นที่ 3 มี data 1040 (total length = 1060) · ถ้า **4000 คือ total length** data จริงมี 3980 ชิ้นที่ 3 มี data 1020 (total length = 1040) — อ่านโจทย์ให้ดีว่าให้ "data" หรือ "datagram length"

### 🧮 ตัวอย่างที่ 2: Data payload 3,200 bytes, MTU = 1,000, header 20, ID = 8888

**ขั้นที่ 1: หา data สูงสุดต่อชิ้น**
- Max Data = MTU − IP header = 1000 − 20 = **980 bytes**
- **ต้องหาร 8 ลงตัว:** 980/8 = 122.5 ✗ → ปัดลง 122 × 8 = **976 bytes**

**ขั้นที่ 2: หั่น**
| ชิ้น | Data (byte range) | Total Length | MF | Offset |
|---|---|---|---|---|
| 1 | 976 (0–975) | 976 + 20 = **996** | 1 | 0/8 = **0** |
| 2 | 976 (976–1,951) | **996** | 1 | 976/8 = **122** |
| 3 | 976 (1,952–2,927) | **996** | 1 | 1,952/8 = **244** |
| 4 | 3,200 − 976×3 = **272** (2,928–3,199) | 272 + 20 = **292** | **0** | 2,928/8 = **366** |

**ตอบ:** 4 fragments ทุกชิ้นมี ID = 8888

**สูตรลัด**
```text
max data ต่อชิ้น = floor((MTU − 20) / 8) × 8
offset ของชิ้นที่ k = (byte เริ่มต้นของชิ้นนั้น) / 8
total length = data + 20
MF = 1 ทุกชิ้น ยกเว้นชิ้นสุดท้าย
```

### 🔹 Protocol Field (กำหนดโดย IANA)

| Protocol | เลข | ใช้จริง |
|---|---|---|
| **ICMP** | **1** | ping, tracert |
| IGMP | 2 | Multicast (IPTV, live streaming) |
| **TCP** | **6** | เว็บ (HTTP/HTTPS), โหลดไฟล์, อีเมล |
| **UDP** | **17** | YouTube, คอลไลน์, เกมออนไลน์, DNS |
| GRE | 47 | อุโมงค์ VPN (tunneling) |
| OSPF | 89 | routing ภายในองค์กร/ISP |

### 🔹 TTL vs Protocol Field

| | Time-to-Live (TTL) | Protocol Field |
|---|---|---|
| หน้าที่หลัก | กำหนด **อายุของแพ็กเก็ต** กันวนไม่รู้จบ (**routing loop**) | ระบุ **protocol Layer 4** ที่รอรับข้อมูล เพื่อส่งต่อให้ถูก (**demultiplexing**) |
| ตัวเลขบอก | จำนวน hop (router) ที่ยังผ่านได้ | ชนิด protocol ใน payload |
| ค่าตัวอย่าง | **64 (Linux/Mac), 128 (Windows)** | 1, 6, 17 |
| การเปลี่ยนแปลง | **ลดลงทีละ 1 ทุก router** | **คงที่** ไม่ถูกแก้โดย router |
| ตำแหน่ง | แถวที่ 3 bit 0–7 | แถวที่ 3 bit 8–15 (ข้าง TTL) |

---

## 6. IP addressing: ประเภทของ IP

### 🔹 ตามการใช้งาน
| | Public IP | Private IP |
|---|---|---|
| ออกโดย | ISP | ใช้ภายในเครือข่ายส่วนตัว (บ้าน ออฟฟิศ LAN) |
| ใช้ทำอะไร | เชื่อมออก Internet | ภายใน |
| ซ้ำได้ไหม | **ห้ามซ้ำทั้งโลก** | **ซ้ำได้** ในคนละเครือข่าย |

**Private range (RFC 1918, กำหนดโดย IETF)**
- **10.0.0.0 – 10.255.255.255**
- **172.16.0.0 – 172.31.255.255**
- **192.168.0.0 – 192.168.255.255**

### 🔹 ตามรูปแบบการจัดสรร (Assignment)
- **Static IP** — กำหนดตายตัว ไม่เปลี่ยน เหมาะกับ server, printer, กล้องวงจรปิดที่ต้องเข้าถึงตลอด
- **Dynamic IP** — **DHCP server** แจกให้อัตโนมัติเมื่อเชื่อมต่อ เหมาะกับมือถือ โน้ตบุ๊ก

### 🔹 Classful addressing

| Class | ช่วง | Default mask | Network : Host | Host/วง | เหมาะกับ |
|---|---|---|---|---|---|
| **A** | 1.0.0.0 – 127.255.255.255 | 255.0.0.0 (/8) | 8 : 24 bits | ~16.7 ล้าน | องค์กรใหญ่มาก, ISP |
| **B** | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | 16 : 16 | 65,534 | บริษัทกลาง-ใหญ่, มหาวิทยาลัย |
| **C** | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | 24 : 8 | 254 | บ้าน, SME |
| **D** | 224.0.0.0 – 239.255.255.255 | — | — | — | **Multicast** (IPTV, routing protocol) |
| **E** | 240.0.0.0 – 255.255.255.255 | — | — | — | **วิจัย/ทดลอง** |

- **127.x.x.x** สงวนไว้: **127.0.0.1 = Loopback (localhost)** ใช้ทดสอบภายในเครื่อง ไม่แจกเป็น IP ทั่วไป

---

## 7. Subnetting (หัวใจของโจทย์คำนวณ)

**Subnetting** = แบ่งเครือข่ายใหญ่เป็นเครือข่ายย่อย แต่ละวงมี 4 ค่า:
1. **Network Address** — IP ตัวแรก = ชื่อวง **ห้ามแจกเครื่อง**
2. **First Usable IP** — ตัวแรกที่แจกได้
3. **Last Usable IP** — ตัวสุดท้ายที่แจกได้
4. **Broadcast Address** — IP ตัวสุดท้าย ส่งหาทุกเครื่องในวง **ห้ามแจกเครื่อง**

### 🧮 สูตร
```text
จำนวน IP ทั้งหมดในวง  = 2^(32 − prefix)
จำนวน IP ที่ใช้งานได้ = 2^(32 − prefix) − 2      (หัก Network และ Broadcast)
Block size              = 2^(32 − prefix)        (สำหรับ prefix ใน octet สุดท้าย)
Subnet mask octet สุดท้าย = 256 − block size
```

### 🔹 กฎ 3 ข้อห้ามลืม
1. **Network Address ต้องหาร block size ลงตัว** (เช่น .0, .64, .128, .192)
2. **Broadcast = Network Address ของวงถัดไป − 1** (ก่อน .64 คือ .63)
3. **Usable IP** = ตัวเลขที่อยู่ **ระหว่าง** Network กับ Broadcast เท่านั้น

### 🔹 ทำไมต้องมี CIDR
เดิมแบ่ง Class A/B/C ตายตัว (**classful**) → เปลือง IP มาก เช่น ต้องการ 300 เครื่อง Class C ได้แค่ 254 ต้องไปขอ Class B ที่ได้ 65,534 → เสีย IP ฟรี ๆ กว่า 65,000 เบอร์
**CIDR (Classless Inter-Domain Routing)** ให้ **ตัดขนาดวงได้อิสระตามความต้องการจริง** ไม่ยึด class

> 📌 **CIDR** (อ่านว่า "cider"): subnet portion ยาวเท่าไหร่ก็ได้ รูปแบบ **a.b.c.d/x** โดย **x = จำนวน bit ของ subnet part** เช่น `200.23.16.0/23` → `11001000 00010111 0001000|0 00000000` (23 bit แรกเป็น subnet)

| Prefix | Subnet Mask | IP ทั้งหมด | ใช้ได้จริง | เหมาะกับ |
|---|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 | LAN บ้าน/แผนก/ออฟฟิศ |
| /25 | 255.255.255.128 | 128 | 126 | แผนกกลาง (ครึ่ง /24) |
| /26 | 255.255.255.192 | 64 | 62 | แผนกเล็ก |
| /27 | 255.255.255.224 | 32 | 30 | แผนก 20–30 เครื่อง |
| /28 | 255.255.255.240 | 16 | 14 | (จากตัวอย่าง /28) |
| /30 | 255.255.255.252 | 4 | **2** | **Router ↔ Router (point-to-point)** |

### 🔹 หลักการ: CIDR ↔ Mask ↔ เลขฐานสอง
- IPv4 ยาว **32 bits**; ตัวเลขหลัง `/` = จำนวน bit จากซ้ายที่ล็อกเป็น **Network ID (เลข 1)** ที่เหลือคือ **Host ID (เลข 0)**
- **/24:** `11111111.11111111.11111111.00000000` = 255.255.255.0, host 8 bits → 2^8 = **256 IPs**
- **/27:** `11111111.11111111.11111111.11100000` = 255.255.255.224, host 5 bits → 2^5 = **32 IPs**

### 🧮 ตัวอย่าง A: 192.168.10.0/26
1. Host bits = 32 − 26 = 6 → 2^6 = **64 IPs** (block size)
2. Mask: 256 − 64 = 192 → **255.255.255.192**
3. วงแรก (0–63): Network **192.168.10.0** · First **.1** · Last **.62** · Broadcast **.63** → แจกได้ **62 เครื่อง**

### 🧮 ตัวอย่าง B: 192.168.1.0/26 แบ่งเป็นวง
- IP ทั้งหมด 64, ใช้ได้ 62, block = 64
- วงที่ 1: Network .0 · ใช้ได้ .1–.62 · Broadcast .63
- วงที่ 2: Network **.64** · ใช้ได้ .65–.126 · Broadcast **.127**
- (วง 3 เริ่ม .128, วง 4 เริ่ม .192)

### 🧮 ตัวอย่าง C: ต้องการ 50 เครื่อง จาก 172.16.0.0/24
1. 50 + 2 = 52 IPs → หา 2^n ≥ 52: 2^5 = 32 ไม่พอ, **2^6 = 64** พอ
2. Prefix = 32 − 6 = **/26**
3. Network **172.16.0.0** · Usable **172.16.0.1 – 172.16.0.62** · Broadcast **172.16.0.63**

### 🧮 ตัวอย่าง D: IP 10.10.15.110/28 อยู่วงไหน
1. Host bits = 4 → block = **16**
2. หาว่า .110 อยู่ช่วงไหน: 16 × 6 = 96, 16 × 7 = 112 → ช่วง **.96 – .111**
3. Network **10.10.15.96** · Broadcast **10.10.15.111** · Usable .97 – .110 (ตรงกับ IP server พอดี)

**ทริค:** `network = floor(host octet / block) × block` → floor(110/16) × 16 = 6 × 16 = 96

### 🧮 ตัวอย่าง E: VLSM (Variable Length Subnet Mask)

**โจทย์:** ได้ 192.168.1.0/24 แบ่งให้ Sales 100, IT 50, HR 20, Router link 2 ให้ประหยัดที่สุด

**วิธี:** เรียงจาก **วงใหญ่ไปเล็ก** แล้วต่อกันไป
1. **Sales 100:** 102 IPs → 2^7 = 128 → **/25** (255.255.255.128) เริ่ม .0
2. **IT 50:** 52 → 2^6 = 64 → **/26** (255.255.255.192) เริ่ม .128
3. **HR 20:** 22 → 2^5 = 32 → **/27** (255.255.255.224) เริ่ม .192
4. **Router link 2:** 4 → 2^2 = 4 → **/30** (255.255.255.252) เริ่ม .224

| แผนก | ต้องการ | Network | Prefix | Mask | Usable | Broadcast |
|---|---|---|---|---|---|---|
| Sales | 100 | 192.168.1.0 | /25 | 255.255.255.128 | .1 – .126 | .127 |
| IT | 50 | 192.168.1.128 | /26 | 255.255.255.192 | .129 – .190 | .191 |
| HR | 20 | 192.168.1.192 | /27 | 255.255.255.224 | .193 – .222 | .223 |
| Router-Link | 2 | 192.168.1.224 | /30 | 255.255.255.252 | .225 – .226 | .227 |

เหลือ **.228 – .255 (28 เบอร์)** ไว้ขยายในอนาคต

---

## 8. IP addressing (นิยามตามหนังสือ) และ Subnets

- **IP address** = ตัวระบุ **32-bit** ที่ผูกกับ **interface** ของ host หรือ router
- **Interface** = จุดเชื่อมระหว่าง host/router กับ physical link
  - **router มักมีหลาย interface**
  - **host มักมี 1–2 interface** (เช่น wired Ethernet, wireless 802.11)
- **Dotted-decimal:** `223.1.1.1 = 11011111 00000001 00000001 00000001`
- interface เชื่อมกันผ่าน Ethernet switch หรือ WiFi base station (ยังไม่ต้องสนใจตอนนี้)

### 🔹 Subnet คืออะไร
> 📌 **Subnet:** กลุ่ม interface ที่ **ถึงกันได้ทางกายภาพโดยไม่ต้องผ่าน router** (คุยกันตรงผ่าน switch หรือ Wi-Fi)

**โครงสร้าง IP:** **subnet part** = high-order bits ที่เหมือนกันในวงเดียวกัน · **host part** = low-order bits ที่เหลือ

**สูตรหา subnet (recipe):** ถอด interface ทุกอันออกจาก host/router ให้เกิด "เกาะ" เครือข่ายที่แยกกัน → **แต่ละเกาะคือ 1 subnet** (เช่น 223.1.1.0/24, 223.1.2.0/24, 223.1.3.0/24)

> ⚠️ ลิงก์ระหว่าง router กับ router ก็นับเป็น subnet ด้วย (ตัวอย่างหนังสือมี 6 subnet: 223.1.1/24, 223.1.2/24, 223.1.3/24, 223.1.7/24, 223.1.8/24, 223.1.9/24)

---

## 9. ได้ IP มาอย่างไร: DHCP

**2 คำถาม:** (1) host ได้ IP (host part) อย่างไร (2) network ได้ IP (network part) อย่างไร

**host ได้ IP:**
- **hard-coded** โดย sysadmin ในไฟล์ config (เช่น `/etc/rc.config` ใน UNIX)
- **DHCP (Dynamic Host Configuration Protocol)** — ขอจาก server แบบไดนามิก "**plug-and-play**"

### 🔹 DHCP
**เป้าหมาย:** host ได้ IP จาก server อัตโนมัติเมื่อ "เข้าร่วม" เครือข่าย
- **ต่ออายุ (renew) lease** ได้
- **นำ address กลับมาใช้ซ้ำ** (ถือไว้เฉพาะตอนเชื่อมต่อ)
- รองรับผู้ใช้มือถือที่เข้า/ออกเครือข่าย

**4 ขั้นตอน (DORA)**
1. **DHCP discover** — host **broadcast** หา server [optional]
2. **DHCP offer** — server เสนอ IP [optional]
3. **DHCP request** — host ขอใช้ IP นั้น
4. **DHCP ack** — server ยืนยัน

| ข้อความ | src | dest | yiaddr | transaction ID |
|---|---|---|---|---|
| Discover | 0.0.0.0, **68** | 255.255.255.255, **67** | 0.0.0.0 | 654 |
| Offer | 223.1.2.5, 67 | 255.255.255.255, 68 | 223.1.2.4 (lifetime 3600 s) | 654 |
| Request | 0.0.0.0, 68 | 255.255.255.255, 67 | 223.1.2.4 | 655 |
| ACK | 223.1.2.5, 67 | 255.255.255.255, 68 | 223.1.2.4 | 655 |

- **client port 68, server port 67**, ส่งแบบ **broadcast 255.255.255.255**
- 2 ขั้นแรก **ข้ามได้** ถ้า client จำและอยากใช้ address เดิม (RFC 2131)
- DHCP server มักอยู่ **ใน router** และให้บริการทุก subnet ที่ router ต่ออยู่

### 🔹 DHCP ให้มากกว่า IP
- address ของ **first-hop router** (default gateway)
- **ชื่อและ IP ของ DNS server**
- **network mask** (บอกส่วน network vs host → รู้ว่าปลายทางอยู่วง LAN เดียวกันหรือนอกวง)

### 🔹 ตัวอย่าง DHCP (encapsulation)
- laptop ใช้ DHCP เพื่อได้ IP, first-hop router, DNS server
- **DHCP REQUEST ห่อใน UDP → ห่อใน IP → ห่อใน Ethernet**
- Ethernet frame **broadcast (dest: FFFFFFFFFFFF)** บน LAN ไปถึง router ที่รัน DHCP server
- แกะ (demux) Ethernet → IP → UDP → DHCP
- server สร้าง **DHCP ACK** (IP client, first-hop router, DNS) ส่งกลับ → client demux ขึ้นไปถึง DHCP
- สไลด์เสริมแสดง Wireshark: Request (Requested IP 192.168.1.101, host name "nomad") และ ACK (Subnet Mask 255.255.255.0, Router 192.168.1.1, DNS servers, domain name)

---

## 10. NAT: Network Address Translation

> 📌 **NAT:** ทุกอุปกรณ์ในเครือข่ายภายใน **ใช้ IPv4 สาธารณะเดียวกัน** ในสายตาโลกภายนอก

- datagram ที่ออกจากเครือข่ายภายใน **มี source NAT IP เดียวกัน (เช่น 138.76.29.7) แต่ source port ต่างกัน**
- ภายในใช้ address **private** (10/8, 172.16/12, 192.168/16) ที่ใช้ได้แค่ในวงภายใน

### 🔹 ข้อดี (ประหยัด + ยืดหยุ่น + ปลอดภัย)
- ขอ IP จาก ISP **แค่ 1 เบอร์** สำหรับทุกเครื่อง
- **เปลี่ยน IP เครื่องภายใน** ได้โดยไม่ต้องแจ้งโลกภายนอก
- **เปลี่ยน ISP** ได้โดยไม่ต้องเปลี่ยน IP เครื่องภายใน
- **ความปลอดภัย:** เครื่องภายใน **ไม่ถูกเข้าถึง/มองเห็นโดยตรง** จากภายนอก

### 🔹 การทำงานของ NAT router (transparent)
1. **ขาออก:** แทน **(source IP, port)** ด้วย **(NAT IP, new port)** — ปลายทางจะตอบกลับมาที่ (NAT IP, new port)
2. **จำ** คู่แปลงทุกคู่ใน **NAT translation table**
3. **ขาเข้า:** แทน (NAT IP, new port) ใน destination ด้วย (source IP, port) เดิมจากตาราง

**ตัวอย่าง (4 ขั้น)**
| ขั้น | เกิดอะไร | S | D |
|---|---|---|---|
| 1 | host 10.0.0.1 ส่งไปเว็บ | 10.0.0.1, 3345 | 128.119.40.186, 80 |
| 2 | NAT เปลี่ยน source + บันทึกตาราง | **138.76.29.7, 5001** | 128.119.40.186, 80 |
| 3 | reply กลับมาที่ NAT | 128.119.40.186, 80 | 138.76.29.7, 5001 |
| 4 | NAT แปลง dest กลับส่งให้ host | 128.119.40.186, 80 | **10.0.0.1, 3345** |

NAT table: `WAN 138.76.29.7, 5001 ↔ LAN 10.0.0.1, 3345`

### 🔹 NAT เป็นที่ถกเถียง (controversial)
- router "ควร" ทำงานแค่ถึง **layer 3** (แต่ NAT ยุ่งกับ port = layer 4)
- ปัญหา IP ไม่พอ "ควร" แก้ด้วย **IPv6**
- **ละเมิด end-to-end argument** (อุปกรณ์ network layer แก้ port)
- **NAT traversal:** ถ้า client ภายนอกอยากต่อ server ที่อยู่หลัง NAT?
- **แต่ NAT อยู่ต่อแน่นอน:** ใช้มากในบ้าน องค์กร และเครือข่าย 4G/5G

---

## 11. IPv6

### 🔹 Motivation
- **หลัก:** address space IPv4 **32-bit จะถูกใช้หมด**
- **เพิ่มเติม:** ประมวลผล/forward เร็วขึ้นด้วย **header ยาวคงที่ 40 bytes**, รองรับการจัดการ "**flows**" ต่างกัน

### 🔹 IPv6 datagram format
| Field | ความหมาย |
|---|---|
| ver | เวอร์ชัน |
| **pri** (priority) | ระบุลำดับความสำคัญของ datagram ใน flow |
| **flow label** | ระบุ datagram ใน "flow" เดียวกัน (แนวคิด flow ยังนิยามไม่ชัด) |
| payload len | ความยาว payload |
| **next hdr** | header/protocol ถัดไป |
| **hop limit** | (คล้าย TTL) |
| source / dest address | **128 bits** |

**สิ่งที่หายไปเมื่อเทียบกับ IPv4 (ออกสอบ)**
- ❌ **ไม่มี checksum** (เพื่อให้ router ประมวลผลเร็ว)
- ❌ **ไม่มี fragmentation/reassembly**
- ❌ **ไม่มี options** (ย้ายไปเป็น upper-layer/next-header protocol)

### 🔹 Transition IPv4 → IPv6: Tunneling
- router ทุกตัว **อัปเกรดพร้อมกันไม่ได้** ("no flag days") — จะทำงานแบบผสมอย่างไร?
- **Tunneling:** ใส่ **IPv6 datagram เป็น payload ใน IPv4 datagram** เมื่อผ่าน router IPv4 ("**packet within a packet**") — ใช้มากในบริบทอื่นด้วย (4G/5G)
- ตัวอย่าง: A(IPv6) → B(IPv6/v4) → C, D (IPv4) → E(IPv6/v4) → F(IPv6)
  - A→B: IPv6 ปกติ (src A, dest F)
  - B→C→D→E: **IPv6 ซ่อนใน IPv4** ที่มี **src B, dest E**
  - E→F: IPv6 ปกติ
- **logical view:** B กับ E ต่อกันด้วย "อุโมงค์"

### 🔹 IPv6 adoption
- Google: ~**40%** ของ client ใช้ IPv6 (2023)
- NIST: **1/3** ของ domain รัฐบาลสหรัฐรองรับ IPv6
- ใช้เวลานานมาก: **25 ปีแล้วยังไม่จบ** (เทียบกับการเปลี่ยนแปลงระดับ app ในช่วงเดียวกัน: WWW, social media, streaming, gaming, telepresence)

| | IPv4 | IPv6 |
|---|---|---|
| ขนาด address | 32 bit | **128 bit** |
| header | ไม่คงที่ (20 + options) | **คงที่ 40 bytes** |
| checksum | มี | **ไม่มี** |
| fragmentation ที่ router | มี | **ไม่มี** |
| อายุแพ็กเก็ต | TTL | hop limit |

---

## 12. Generalized Forwarding: Match + Action

**ทบทวน:** ทุก router มี **forwarding table (aka flow table)**
- **"match plus action"**: จับคู่ bit ใน packet ที่เข้ามา แล้วทำ action
- **destination-based forwarding:** forward ตาม **dest IP** เท่านั้น
- **generalized forwarding:** **หลาย header field** ตัดสิน action ได้, action ได้หลายแบบ: **drop / copy / modify / log**

### 🔹 Flow table abstraction
- **flow** = นิยามจากค่าใน header field (link-, network-, transport-layer)
- กฎแต่ละข้อมี:
  - **match:** pattern ของค่าใน header
  - **actions:** drop, forward, modify หรือ **ส่งให้ controller**
  - **priority:** แก้กรณี pattern ซ้อนกัน
  - **counters:** จำนวน bytes และ packets

| match | action |
|---|---|
| src = \*.\*.\*.\*, dest = 3.4.\*.\* | forward(2) |
| src = 1.2.\*.\*, dest = \*.\*.\*.\* | drop |
| src = 10.1.2.3, dest = \*.\*.\*.\* | send to controller |

(`*` = wildcard)

### 🔹 OpenFlow flow table entries
**Match** (header fields) — **Action** — **Stats** (packet + byte counters)

**Actions:** (1) forward ไป port (2) drop (3) modify header fields (4) encapsulate แล้วส่งให้ controller

**Header fields ที่ match ได้**
- **Link layer:** Ingress Port, Src MAC, Dst MAC, Eth Type, VLAN ID, VLAN Pri
- **Network layer:** IP Src, IP Dst, IP Proto, IP ToS
- **Transport layer:** TCP/UDP Src Port, TCP/UDP Dst Port

### 🔹 OpenFlow examples
| จุดประสงค์ | Match | Action |
|---|---|---|
| Destination-based forwarding | IP Dst = 51.6.0.8 | port6 |
| Firewall: บล็อก SSH | TCP d-port = **22** | drop |
| Firewall: บล็อก host | IP Src = 128.119.1.1 | drop |
| Layer 2 forwarding | MAC dst = 22:A7:23:11:E1:02 | port3 |

### 🔹 OpenFlow abstraction รวมอุปกรณ์หลายแบบ
| อุปกรณ์ | Match | Action |
|---|---|---|
| **Router** | **longest destination IP prefix** | forward out a link |
| **Firewall** | IP addresses และ TCP/UDP port | **permit / deny** |
| **Switch** | **destination MAC** | **forward หรือ flood** |
| **NAT** | IP address และ port | **rewrite** address และ port |

### 🔹 OpenFlow example: orchestrated tables
ตารางที่ประสานกันสร้างพฤติกรรมระดับเครือข่ายได้ เช่น ส่ง datagram จาก h5, h6 ไป h3 หรือ h4 ผ่าน s1 แล้วไป s2
- **s3:** IP Src = 10.3.\*.\*, IP Dst = 10.2.\*.\* → forward(3)
- **s1:** ingress port = 1, IP Src = 10.3.\*.\*, IP Dst = 10.2.\*.\* → forward(4)
- **s2:** ingress port = 2, IP Dst = 10.2.0.3 → forward(3); ingress port = 2, IP Dst = 10.2.0.4 → forward(4)

### 🔹 Summary
- match bit ใน header **ทุก layer** แล้วทำ action: drop, forward, modify, ส่งให้ controller
- "program" พฤติกรรมทั้งเครือข่ายได้ → **network programmability** แบบง่าย (programmable per-packet processing)
- รากเหง้า: **active networking** · ปัจจุบัน: **P4** (p4.org)

---

## 13. Middleboxes

> 📌 **Middlebox (RFC 3234):** "any intermediary box performing functions apart from normal, standard functions of an IP router on the data path between a source host and destination host"

**มีอยู่ทุกที่:** NAT (บ้าน, cellular, องค์กร), Firewalls/IDS (องค์กร, ISP), Load balancers (องค์กร, data center, mobile), Caches (service provider, mobile, CDN), application-specific (service provider, CDN)

**วิวัฒนาการ**
- เดิม: **proprietary (closed) hardware**
- ไปสู่ **"whitebox" hardware** ที่ใช้ open API → เลิกพึ่งฮาร์ดแวร์เฉพาะยี่ห้อ, ทำ action ด้วย match+action, แข่งกันที่ซอฟต์แวร์
- **SDN:** ควบคุมและจัดการ config แบบ (logically) **centralized** มักอยู่บน cloud
- **NFV (Network Functions Virtualization):** บริการที่โปรแกรมได้บน whitebox networking, computation, storage

---

## 14. Internet Architecture

### 🔹 The IP hourglass
- Internet มี **"thin waist"**: protocol ชั้น network **มีตัวเดียวคือ IP** ซึ่ง **ทุกอุปกรณ์ (หลายพันล้าน) ต้อง implement**
- ด้านบนมีหลาย protocol (HTTP, SMTP, RTP, QUIC, DASH, TCP, UDP) ด้านล่างก็หลาย (Ethernet, PPP, WiFi, Bluetooth, PDCP, copper, radio, fiber)
- **วัยกลางคน:** มี "love handles" คือ **middleboxes** (caching, NAT, firewall) ทำงานในเครือข่าย

### 🔹 Architectural Principles (RFC 1958)
> "...the goal is **connectivity**, the tool is the **Internet Protocol**, and the intelligence is **end to end** rather than hidden in the network."

**3 ความเชื่อหลัก:** simple connectivity · IP protocol: narrow waist · **intelligence, complexity at network edge**

### 🔹 The end-end argument
- ฟังก์ชันบางอย่าง (เช่น reliable data transfer, congestion) ทำได้ **ในเครือข่าย (hop-by-hop)** หรือ **ที่ขอบ (end-end)**
- Saltzer, Reed, Clark (1981): ฟังก์ชันจะทำได้ **สมบูรณ์และถูกต้องก็ต่อเมื่อมีความรู้และความช่วยเหลือของ application ที่ปลายทาง** ดังนั้นทำเป็นฟีเจอร์ของระบบสื่อสารเองไม่ได้ (บางครั้งเวอร์ชันไม่สมบูรณ์ในเครือข่ายช่วยเรื่อง performance ได้) — เหตุผลนี้เรียกว่า **end-to-end argument**

### 🔹 Where's the intelligence?
| ยุค | intelligence อยู่ที่ |
|---|---|
| โทรศัพท์ศตวรรษที่ 20 | **network switches** |
| Internet ก่อน 2005 | **edge** (ปลายทาง) |
| Internet หลัง 2005 | **programmable network devices** + infrastructure ระดับ app ขนาดใหญ่ที่ edge |

---

## จุดเน้นสำหรับสอบ
- 🧮 **Subnetting:** หา block size, mask, network, first/last usable, broadcast จาก IP/prefix
- 🧮 **เลือก prefix** จากจำนวนเครื่อง (n + 2 → 2^k) และ **VLSM** เรียงใหญ่ไปเล็ก
- 🧮 **Fragmentation:** max data = floor((MTU−20)/8)×8, offset = byte/8, MF, total length
- 🔥 forwarding vs routing, data plane vs control plane, traditional vs SDN
- 🔥 DHCP 4 ขั้น (discover/offer/request/ack), port 67/68, ให้อะไรบ้าง
- 🔥 NAT: ตาราง translation, ข้อดี 4 ข้อ, ข้อถกเถียง
- ⭐ IPv6 ไม่มี checksum / fragmentation / options; tunneling
- ⭐ TTL vs Protocol field, protocol numbers (ICMP 1, TCP 6, UDP 17)
- ⭐ Private range, Class A–E, loopback 127.0.0.1
- ⭐ OpenFlow: match / action / stats, ตาราง router/firewall/switch/NAT
- 📌 Best-effort ไม่รับประกันอะไร, middlebox (RFC 3234), IP hourglass, end-end argument

## ✅ Coverage Check
- [x] Goals, roadmap, services, forwarding vs routing, data/control plane, per-router vs SDN
- [x] Service model table + best-effort reflections
- [x] Router architecture overview (สไลด์ 15–16 ไม่อยู่ในไฟล์)
- [x] Internet network layer, IPv4 datagram, overhead, ID/flags/offset, ตัวอย่าง fragmentation 2 ข้อ, protocol numbers, TTL vs protocol
- [x] IP types, private range, classes, subnet rules, CIDR, prefix table, binary, ตัวอย่าง 4 ข้อ, VLSM
- [x] IP addressing intro, interfaces, subnets, recipe, CIDR
- [x] DHCP: goals, overview, scenario, more than IP, example, Wireshark
- [x] NAT: concept, advantages, implementation, example, controversy
- [x] IPv6: motivation, format, missing fields, tunneling, adoption
- [x] Generalized forwarding, flow table, OpenFlow entries/examples/abstraction/example, summary
- [x] Middleboxes, IP hourglass, RFC 1958, end-end argument, where's intelligence
