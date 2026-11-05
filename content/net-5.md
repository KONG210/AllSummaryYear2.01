# Chapter 5 · Network Layer: Control Plane

> ⚠️ **ข้อสังเกตสำคัญ:** ไฟล์ที่อาจารย์ให้มา **ข้ามส่วน routing algorithms (link state / Dijkstra และ distance vector / Bellman-Ford)** ไป (เลขสไลด์กระโดดจาก 5-17 ไป 5-18 โดยไม่มีหัวข้อ "routing algorithms" ใน roadmap) แต่ OSPF ในบทนี้ใช้ Dijkstra และ RIP/EIGRP ใช้ DV — ถามอาจารย์หรือเพื่อนว่าออกสอบคำนวณ Dijkstra/DV หรือไม่

## ภาพรวมของบท

**เป้าหมาย**
- เข้าใจหลักการของ **network control plane**: traditional routing, **SDN controllers**, network management/configuration
- การใช้งานจริงบน Internet: **OSPF, BGP**, OpenFlow, ODL และ ONOS controllers, **ICMP**, **SNMP**

**Roadmap:** introduction → intra-ISP routing: **OSPF** → routing among ISPs: **BGP** → **SDN** control plane → **ICMP** → network management, configuration: **SNMP**

> 🔗 ภาพใหญ่: บทที่ 4 คือ router "ส่งต่อ" ตามตาราง → บทนี้คือ "ใครสร้างตาราง": ภายใน AS ใช้ **OSPF** (คำนวณเส้นทางสั้นสุด) ระหว่าง AS ใช้ **BGP** (ตามนโยบาย) หรือให้ **SDN controller** คำนวณจากศูนย์กลาง → ใช้ **ICMP** แจ้ง error/ping/traceroute และใช้ **SNMP** ดูแลอุปกรณ์

---

## 1. Introduction

### 🔹 Network-layer functions (ทบทวน)
- **forwarding** (data plane): ย้าย packet จาก input ไป output ของ router
- **routing** (control plane): หาเส้นทางจากต้นทางถึงปลายทาง

**2 วิธีจัดโครงสร้าง control plane**
1. **per-router control (traditional)** — routing algorithm ในทุก router คุยกันเพื่อคำนวณ forwarding table
2. **logically centralized control (SDN)** — **remote controller** คำนวณและติดตั้ง forwarding table ใน router (ผ่าน control agent: CA)

### 🔹 Routing protocols
**เป้าหมาย:** หาเส้นทาง (routes) ที่ "ดี" จาก host ผู้ส่งไป host ผู้รับผ่านเครือข่าย router
- **path** = ลำดับของ router ที่ packet ผ่านจากต้นทางถึงปลายทาง
- **"ดี"** = **cost ต่ำสุด, เร็วที่สุด, แออัดน้อยที่สุด**
- routing คือ "top-10" challenge ของ networking

---

## 2. Making routing scalable

การศึกษาที่ผ่านมาเป็นแบบ **อุดมคติ**: router ทุกตัวเหมือนกัน, เครือข่าย "แบน" → **ไม่จริงในทางปฏิบัติ**

| ปัญหา | รายละเอียด |
|---|---|
| **Scale** | ปลายทางหลายพันล้าน → **เก็บทุกปลายทางใน routing table ไม่ได้**, การแลก routing table จะ **ท่วม link** |
| **Administrative autonomy** | Internet = **network of networks**; admin แต่ละเครือข่ายอยาก **ควบคุม routing ในเครือข่ายตัวเอง** |

### 🔹 Internet approach: Autonomous Systems (AS)
รวม router เป็นกลุ่ม เรียกว่า **"autonomous systems" (AS)** หรือ **"domains"**

| | Intra-AS (intra-domain) | Inter-AS (inter-domain) |
|---|---|---|
| คืออะไร | routing **ระหว่าง router ภายใน AS เดียวกัน** | routing **ระหว่าง AS** |
| กฎ | router ใน AS เดียวกัน **ต้องรัน intra-domain protocol เดียวกัน**; คนละ AS รันต่างกันได้ | **gateway** ทำ inter-domain routing (และ intra-domain ด้วย) |

- **Gateway router:** อยู่ที่ "ขอบ" ของ AS ตัวเอง มี link ไปยัง router ใน AS อื่น

### 🔹 Interconnected ASes
- **forwarding table** ถูกตั้งค่าโดยทั้ง intra-AS และ inter-AS routing
  - **intra-AS** กำหนด entry สำหรับ **ปลายทางภายใน AS**
  - **inter-AS + intra-AS** กำหนด entry สำหรับ **ปลายทางภายนอก**

### 🔹 Inter-AS routing มีบทบาทใน intradomain forwarding
สมมติ router ใน AS1 ได้ datagram ที่จะไปนอก AS1 → ควรส่งไป gateway ใน AS1 แต่ **ตัวไหน?**

**inter-domain routing ของ AS1 ต้อง:**
1. **เรียนรู้** ว่าปลายทางไหนไปถึงได้ผ่าน AS2, ปลายทางไหนผ่าน AS3
2. **กระจาย** ข้อมูลการเข้าถึง (reachability) นี้ให้ **ทุก router ใน AS1**

---

## 3. Intra-AS routing: OSPF

### 🔹 Intra-AS routing protocols ที่พบบ่อย
| Protocol | ประเภท | หมายเหตุ |
|---|---|---|
| **RIP** (Routing Information Protocol, RFC 1723) | classic **DV** | แลก DV **ทุก 30 วินาที**, **ไม่ค่อยใช้แล้ว** |
| **EIGRP** (Enhanced Interior Gateway Routing Protocol) | **DV** based | เคยเป็นของ **Cisco** หลายสิบปี (เปิดในปี 2013, RFC 7868) |
| **OSPF** (Open Shortest Path First, RFC 2328) | **link-state** | **IS-IS** (มาตรฐาน ISO ไม่ใช่ RFC) แทบเหมือน OSPF |

### 🔹 OSPF (Open Shortest Path First)
- **"open"** = เปิดให้ใช้สาธารณะ
- **classic link-state**
  - ทุก router **flood OSPF link-state advertisements** (ส่ง **ตรงบน IP ไม่ใช้ TCP/UDP**) ไปยัง **ทุก router ใน AS**
  - link cost ได้หลาย metric: **bandwidth, delay**
  - ทุก router มี **topology ครบ** แล้วใช้ **Dijkstra's algorithm** คำนวณ forwarding table
- **security:** ทุก OSPF message **ได้รับการ authenticate** (กันการบุกรุก)

### 🔹 Hierarchical OSPF
- **ลำดับชั้น 2 ระดับ: local area, backbone**
  - link-state advertisement flood **เฉพาะใน area หรือใน backbone**
  - แต่ละ node รู้ topology ละเอียด **เฉพาะใน area ตัวเอง**; สำหรับปลายทางอื่นรู้แค่ **ทิศทาง**

| Router | หน้าที่ |
|---|---|
| **Local (internal) routers** | flood LS เฉพาะใน area, คำนวณ routing ใน area, ส่ง packet ออกนอก area ผ่าน area border router |
| **Area border routers** | **"สรุป" (summarize)** ระยะทางไปยังปลายทางใน area ตัวเอง แล้วประกาศใน backbone |
| **Backbone routers** | รัน OSPF จำกัดใน backbone |
| **Boundary router** | เชื่อมต่อกับ **AS อื่น** |

---

## 4. Inter-AS routing: BGP

### 🔹 BGP คืออะไร
> 📌 **BGP (Border Gateway Protocol):** protocol inter-domain routing **มาตรฐานโดยพฤตินัย (de facto)** — "**glue that holds the Internet together**"

- ให้ subnet **ประกาศการมีอยู่และปลายทางที่ไปถึงได้** ให้ทั้ง Internet: "I am here, here is who I can reach, and how"

**BGP ให้แต่ละ AS:**
1. **รับ** ข้อมูลการเข้าถึงปลายทางจาก AS เพื่อนบ้าน (**eBGP**)
2. **กำหนดเส้นทาง** ไปเครือข่ายอื่นตามข้อมูลการเข้าถึงและ **นโยบาย (policy)**
3. **กระจาย** ข้อมูลการเข้าถึงให้ router ภายใน AS (**iBGP**)
4. **ประกาศ** ข้อมูลการเข้าถึงให้เครือข่ายเพื่อนบ้าน

| | eBGP | iBGP |
|---|---|---|
| ระหว่าง | router **ต่าง AS** | router **ใน AS เดียวกัน** (logical) |
| ทำอะไร | รับ/ประกาศเส้นทางกับเพื่อนบ้าน | กระจายเส้นทางภายใน |

**gateway router รันทั้ง eBGP และ iBGP**

### 🔹 BGP basics
- **BGP session:** BGP router 2 ตัว (**"peers"**) แลก BGP message ผ่าน **TCP connection แบบกึ่งถาวร (semi-permanent)**
- ประกาศ **path** ไปยัง prefix ปลายทาง → BGP เป็น **"path vector" protocol**
- เมื่อ gateway 3a ของ AS3 ประกาศ path **AS3,X** ให้ gateway 2c ของ AS2 → **AS3 สัญญากับ AS2 ว่าจะส่ง datagram ไปยัง X ให้**

### 🔹 BGP protocol messages (RFC 4371)
| Message | หน้าที่ |
|---|---|
| **OPEN** | เปิด TCP connection ไปยัง peer และ **authenticate** ผู้ส่ง |
| **UPDATE** | **ประกาศ path ใหม่** (หรือถอน path เก่า) |
| **KEEPALIVE** | คง connection ไว้เมื่อไม่มี UPDATE; และ **ACK คำขอ OPEN** |
| **NOTIFICATION** | **รายงาน error** ของ message ก่อนหน้า; ใช้ **ปิด connection** ด้วย |

### 🔹 Path attributes และ BGP routes
- **BGP advertised route = prefix + attributes**
  - **prefix:** ปลายทางที่ถูกประกาศ
  - attribute สำคัญ 2 ตัว:
    - **AS-PATH:** รายการ AS ที่การประกาศ prefix ผ่านมา
    - **NEXT-HOP:** บอก router ภายใน AS ที่จะไป AS ถัดไป
- **policy-based routing:**
  - gateway ใช้ **import policy** ตัดสินว่าจะ **รับ/ปฏิเสธ path** (เช่น ห้ามผ่าน AS Y)
  - นโยบายของ AS ยังกำหนดว่าจะ **ประกาศ path ต่อ** ให้เพื่อนบ้านอื่นหรือไม่

### 🔹 BGP path advertisement (ตัวอย่าง)
1. router 2c (AS2) ได้ path **AS3,X** (ผ่าน eBGP) จาก 3a (AS3)
2. ตามนโยบาย AS2, 2c **ยอมรับ** แล้วกระจาย (ผ่าน iBGP) ให้ทุก router ใน AS2
3. ตามนโยบาย AS2, router 2a ประกาศ (ผ่าน eBGP) path **AS2,AS3,X** ให้ 1c (AS1)

**หลายเส้นทาง:** 1c เรียนรู้ทั้ง **AS2,AS3,X** (จาก 2a) และ **AS3,X** (จาก 3a) → ตามนโยบายเลือก **AS3,X** แล้วประกาศใน AS1 ผ่าน iBGP

### 🔹 BGP: populating forwarding tables
- 1a, 1b, 1d เรียนรู้ผ่าน iBGP จาก 1c ว่า "**path ไป X ผ่าน 1c**"
- ที่ **1d:** OSPF บอกว่าไป 1c ใช้ **interface 1** → ไป X ใช้ **interface 1**
- ที่ **1a:** OSPF บอกว่าไป 1c ใช้ **interface 2** → ไป X ใช้ **interface 2**

> 🔥 BGP บอก "ออกทาง gateway ไหน" + OSPF บอก "ไป gateway นั้นใช้ interface ไหน" = entry ใน forwarding table

### 🔹 Hot potato routing
- 2d เรียนรู้ (ผ่าน iBGP) ว่าไป X ได้ทาง **2a** หรือ **2c**
- **hot potato routing:** เลือก **gateway ภายในที่ intra-domain cost ต่ำสุด** (เช่น 2d เลือก 2a ที่ cost 201 แทน 2c ที่ 263 แม้จะมี AS hop ไป X มากกว่า) — **ไม่สนใจ inter-domain cost!**
- เหมือน "มันฝรั่งร้อน" รีบโยนออกจากมือ (ออกจาก AS ตัวเอง) ให้เร็วที่สุด

### 🔹 BGP: achieving policy via advertisements
**นโยบายทั่วไปในโลกจริง:** ISP อยากรับส่ง traffic **เฉพาะไป/มาจากลูกค้าของตัวเอง** ไม่อยากขน **transit traffic** ระหว่าง ISP อื่น

**ตัวอย่าง 1** (A, B, C = provider; w, x, y = customer)
- A ประกาศ path **Aw** ให้ B และ C
- **B เลือกไม่ประกาศ BAw ให้ C** เพราะ B ไม่ได้ "รายได้" จากการ route CBAw (C, A, w ไม่ใช่ลูกค้า B)
- C จึงไม่รู้ path CBAw → C ไปหา w ด้วย **CAw** (ไม่ผ่าน B)

**ตัวอย่าง 2**
- **x เป็น dual-homed** (ต่อกับ 2 เครือข่าย: B และ C)
- นโยบาย: x ไม่อยากเป็นทางผ่านจาก B ไป C → **x จะไม่ประกาศ route ไปหา C ให้ B**

### 🔹 BGP route selection (เรียงตามลำดับ)
เมื่อ router รู้หลาย route ไปยัง AS ปลายทาง เลือกจาก:
1. **local preference value** attribute — การตัดสินใจเชิงนโยบาย
2. **shortest AS-PATH**
3. **closest NEXT-HOP router** — hot potato routing
4. เกณฑ์อื่น ๆ

### 🔹 ทำไม Intra-AS กับ Inter-AS routing ต่างกัน?
| เรื่อง | Inter-AS | Intra-AS |
|---|---|---|
| **Policy** | admin อยากควบคุมว่า traffic ไปทางไหน ใครผ่านเครือข่ายตัวเองได้ | admin คนเดียว นโยบายไม่ค่อยเป็นปัญหา |
| **Scale** | hierarchical routing ประหยัดขนาดตาราง ลด update traffic | |
| **Performance** | **policy สำคัญกว่า performance** | **เน้น performance ได้** |

---

## 5. SDN Control Plane

### 🔹 ที่มา
- เดิม network layer ของ Internet ใช้แบบ **distributed, per-router control**
  - **monolithic router** มี switching hardware และรัน protocol มาตรฐาน (IP, RIP, IS-IS, OSPF, BGP) แบบ **proprietary** บน router OS เฉพาะ (เช่น **Cisco IOS**)
  - มี **middlebox** แยกตามหน้าที่: firewall, load balancer, NAT box
- **~2005:** เริ่มสนใจคิดใหม่เรื่อง network control plane

### 🔹 ทำไมต้องเป็น logically centralized control plane?
- **จัดการเครือข่ายง่ายขึ้น:** ลด **router misconfiguration**, ควบคุม traffic flow ได้ยืดหยุ่นขึ้น
- **table-based forwarding** (OpenFlow API) ทำให้ "program" router ได้
  - **centralized programming ง่ายกว่า:** คำนวณตารางที่ศูนย์กลางแล้วแจกจ่าย
  - **distributed programming ยากกว่า:** ตารางเป็นผลของ distributed algorithm ที่ทุก router ต้องรัน
- **open (non-proprietary)** implementation ของ control plane → **ส่งเสริมนวัตกรรม** "let 1000 flowers bloom"

### 🔹 SDN analogy: mainframe → PC revolution
| ยุค mainframe (เหมือน router แบบเดิม) | ยุค PC (เหมือน SDN) |
|---|---|
| **Vertically integrated** — app, OS, hardware เฉพาะทางจากผู้ผลิตเดียว | **Horizontal** — open interfaces (microprocessor / Windows, Linux, Mac OS / apps) |
| Closed, proprietary | Open interfaces |
| Slow innovation | **Rapid innovation** |
| Small industry | Huge industry |

### 🔹 Traffic engineering: ยากกับ traditional routing
(เครือข่าย u, v, w, x, y, z — เส้นทาง least cost ปกติคือ u-x-y-z)
1. **ถ้าอยากให้ traffic u→z ไปทาง uvwz แทน uxyz?** → ต้อง **นิยาม link weight ใหม่** ให้ algorithm คำนวณตามนั้น (หรือต้องมี algorithm ใหม่) — **link weight เป็น "ปุ่ม" ควบคุมเดียว ควบคุมได้น้อย**
2. **ถ้าอยากแบ่ง traffic u→z ไปทั้ง uvwz และ uxyz (load balancing)?** → **ทำไม่ได้** (หรือต้องมี algorithm ใหม่)
3. **ถ้า w อยาก route traffic สีน้ำเงินกับสีแดงต่างกันไป z?** → **ทำไม่ได้** ด้วย destination-based forwarding และ LS, DV routing

→ บทที่ 4: **generalized forwarding และ SDN** ทำ routing แบบไหนก็ได้ที่ต้องการ

### 🔹 4 ลักษณะของ SDN
1. **generalized "flow-based" forwarding** (เช่น OpenFlow)
2. **แยก control plane กับ data plane**
3. **control plane functions อยู่นอก data-plane switches**
4. **programmable** control applications (routing, access control, load balance)

### 🔹 SDN architecture 3 ชั้น

```text
 network-control apps   (routing | access control | load balance ...)
          ↕  northbound API
 SDN Controller (network operating system)
          ↕  southbound API (เช่น OpenFlow)
 SDN-controlled switches (data plane)
```

| ชั้น | หน้าที่ |
|---|---|
| **Data-plane switches** | switch **เร็ว เรียบง่าย ราคาถูก (commodity)** ทำ generalized forwarding ใน hardware; flow table ถูกคำนวณ/ติดตั้งภายใต้การควบคุมของ controller; มี API ควบคุมแบบตาราง (OpenFlow) ที่ **กำหนดว่าอะไรควบคุมได้/ไม่ได้**; มี protocol คุยกับ controller |
| **SDN controller (network OS)** | **เก็บ network state**; คุยกับ app **ด้านบนผ่าน northbound API**; คุยกับ switch **ด้านล่างผ่าน southbound API**; implement เป็น **distributed system** เพื่อ performance, scalability, fault-tolerance, robustness |
| **Network-control apps** | **"สมอง"** ของการควบคุม ใช้บริการระดับล่างผ่าน API ของ controller; **unbundled**: บุคคลที่สามทำได้ (ต่างจากผู้ขาย routing หรือ controller) |

### 🔹 Components of SDN controller
| Layer | ประกอบด้วย |
|---|---|
| **Interface layer** to network control apps | abstraction/API: network graph, RESTful API, intent |
| **Network-wide state management** | สถานะของ link, switch, services: **distributed database** (statistics, flow tables, link-state info, host info, switch info) |
| **Communication layer** | คุยระหว่าง controller กับ switch: **OpenFlow, SNMP** |

### 🔹 OpenFlow protocol
- ทำงานระหว่าง **controller กับ switch**
- ใช้ **TCP** แลก message (เข้ารหัสได้ optional)
- message 3 กลุ่ม: **controller-to-switch**, **asynchronous (switch to controller)**, **symmetric (misc.)**
- **ต่างจาก OpenFlow API** (API ใช้กำหนด generalized forwarding actions)

| Controller-to-switch | ความหมาย |
|---|---|
| **features** | controller ถาม feature ของ switch, switch ตอบ |
| **configure** | ถาม/ตั้ง config ของ switch |
| **modify-state** | **เพิ่ม ลบ แก้ flow entry** ใน OpenFlow table |
| **packet-out** | controller สั่งส่ง packet ออก port ที่ระบุ |

| Switch-to-controller | ความหมาย |
|---|---|
| **packet-in** | ส่ง packet (และการควบคุม) ไปให้ controller (คู่กับ packet-out) |
| **flow-removed** | flow entry ถูกลบที่ switch |
| **port status** | แจ้ง controller ว่า port มีการเปลี่ยนแปลง |

> operator ไม่ได้เขียน OpenFlow message เอง แต่ใช้ **abstraction ระดับสูงที่ controller**

### 🔹 ตัวอย่าง control/data plane interaction (link failure)
1. **S1 เกิด link failure** → ใช้ OpenFlow **port status** message แจ้ง controller
2. controller ได้ message → **อัปเดต link status**
3. **Dijkstra's routing app** ที่ลงทะเบียนไว้ว่าให้เรียกเมื่อ link status เปลี่ยน **ถูกเรียก**
4. Dijkstra ใช้ network graph + link state ใน controller **คำนวณเส้นทางใหม่**
5. link state routing app คุยกับ **flow-table-computation** component คำนวณ flow table ใหม่
6. controller ใช้ **OpenFlow ติดตั้งตารางใหม่** ใน switch ที่ต้องอัปเดต

### 🔹 ONOS SDN controller
- **SDN controller open-source ยุคแรกที่มีอิทธิพล**
- network apps (traffic engineering, firewalling, load balancing) **แยกจาก controller**
- **intent framework:** ระบุบริการระดับสูง "**what rather than how**" (บอกว่าต้องการอะไร ไม่ใช่ทำอย่างไร)
- เน้น **distributed core:** service reliability, replication, performance scaling
- northbound: REST API, intent · southbound: **OpenFlow, Netconf, OVSDB**

### 🔹 Google ORION
- SDN control plane ของ Google (NSDI'21) สำหรับ datacenter (**Jupiter**) และ wide area (**B4**)
- routing (intradomain, iBGP), traffic engineering ทำเป็น app บน ORION core
- edge-edge flow-based control (เช่น CoFlow scheduling) ให้ตรง SLA
- management: pub-sub distributed microservices, ใช้ OpenFlow คุย/monitor switch
- ให้บริการ **intradomain** ภายในเครือข่าย Google
- คุณค่า (Amin Vahdat): ตำแหน่งและบทบาทของทุก element รู้ล่วงหน้า → ตั้งค่าใน controller กลางได้ เปลี่ยนปัญหาจาก "จัดการ config ของแต่ละอุปกรณ์" เป็น "จัดการ config ของเครือข่ายเชิงตรรกะ" → ง่ายขึ้นมากทั้งทางเทคนิคและภาระความคิดของผู้ดูแล

### 🔹 SDN: selected challenges
- **hardening control plane:** ระบบกระจายที่เชื่อถือได้ scale ได้ ปลอดภัย; ทนต่อ failure (ใช้ทฤษฎี reliable distributed system); dependability/security "baked in" ตั้งแต่แรก?
- เครือข่ายที่ตรงความต้องการเฉพาะ: real-time, ultra-reliable, ultra-secure
- **Internet-scaling:** ข้ามมากกว่า 1 AS
- **SDN สำคัญมากใน 5G cellular networks**

### 🔹 SDN กับอนาคตของ protocol แบบเดิม
- SDN-computed vs router-computed forwarding tables = ตัวอย่างหนึ่งของ **logically-centralized-computed vs protocol-computed**
- อาจมี **SDN-computed congestion control**: controller ตั้ง rate ผู้ส่งตามระดับ congestion ที่ router รายงาน
- คำถามเปิด: การ implement ฟังก์ชันเครือข่าย (SDN vs protocols) จะพัฒนาไปทางไหน?

---

## 6. ICMP: Internet Control Message Protocol

- host และ router ใช้ **สื่อสารข้อมูลระดับ network**
  - **error reporting:** unreachable host, network, port, protocol
  - **echo request/reply** (ใช้โดย **ping**)
- อยู่ชั้น network "**เหนือ IP**": **ICMP message ถูกห่อใน IP datagram** (protocol number = 1)
- ICMP message = **type, code + 8 bytes แรกของ IP datagram ที่ทำให้เกิด error**

| Type | Code | ความหมาย |
|---|---|---|
| **0** | 0 | **echo reply (ping)** |
| **3** | 0 | dest. network unreachable |
| 3 | 1 | dest host unreachable |
| 3 | 2 | dest protocol unreachable |
| **3** | **3** | **dest port unreachable** |
| 3 | 6 | dest network unknown |
| 3 | 7 | dest host unknown |
| 4 | 0 | source quench (congestion control — ไม่ใช้แล้ว) |
| **8** | 0 | **echo request (ping)** |
| 9 | 0 | route advertisement |
| 10 | 0 | router discovery |
| **11** | 0 | **TTL expired** |
| 12 | 0 | bad IP header |

### 🔹 Traceroute และ ICMP
1. ต้นทางส่ง **ชุดของ UDP segment** ไปปลายทาง: **ชุดที่ 1 TTL = 1, ชุดที่ 2 TTL = 2, ...** (ชุดละ 3 probes)
2. datagram ในชุดที่ n ไปถึง **router ตัวที่ n**:
   - router **ทิ้ง datagram** แล้วส่ง ICMP **(type 11, code 0 = TTL expired)** กลับมา
   - ICMP อาจมี **ชื่อ router และ IP**
3. เมื่อ ICMP ถึงต้นทาง → **บันทึก RTT**

**เงื่อนไขหยุด**
- UDP segment ไปถึง **host ปลายทาง**
- ปลายทางตอบ ICMP **"port unreachable" (type 3, code 3)** (เพราะส่งไปที่ port ที่ไม่มีใครใช้)
- ต้นทางหยุด

> ⭐ ping = type 8 (request) / type 0 (reply) · traceroute = type 11 (TTL expired) ระหว่างทาง + type 3 code 3 ที่ปลายทาง

---

## 7. Network Management และ SNMP

### 🔹 Network management คืออะไร
- AS (เครือข่าย) มีส่วนประกอบ hardware/software **หลายพันชิ้น** ที่ทำงานร่วมกัน
- ระบบซับซ้อนอื่นที่ต้อง monitor/config/control: เครื่องบินเจ็ท, โรงไฟฟ้านิวเคลียร์

> 📌 "Network management includes the deployment, integration and coordination of the hardware, software, and human elements to **monitor, test, poll, configure, analyze, evaluate, and control** the network and element resources to meet the real-time, operational performance, and Quality of Service requirements at a reasonable cost."

### 🔹 Components of network management
| ส่วน | ความหมาย |
|---|---|
| **Managing server** | application (มักมี **คนดูแลอยู่ใน loop**) ที่จัดการเครือข่าย |
| **Managed device** | อุปกรณ์ที่มี hardware/software จัดการ/ตั้งค่าได้ มี **agent** |
| **Data** | "state" ของอุปกรณ์: **configuration data, operational data, device statistics** |
| **Network management protocol** | server ใช้ **query, configure, manage** อุปกรณ์; อุปกรณ์ใช้ **แจ้งข้อมูล/เหตุการณ์** ให้ server |

### 🔹 Network operator approaches
| วิธี | ลักษณะ |
|---|---|
| **CLI** (Command Line Interface) | operator พิมพ์/script สั่งอุปกรณ์ทีละตัวโดยตรง (เช่น ผ่าน ssh) |
| **SNMP/MIB** | operator query/set ข้อมูลอุปกรณ์ (**MIB**) ด้วย **Simple Network Management Protocol** |
| **NETCONF/YANG** | **นามธรรมกว่า, ทั้งเครือข่าย, holistic**; เน้นจัดการ config **หลายอุปกรณ์**; **YANG** = data modeling language; **NETCONF** = สื่อสาร action/data ที่เข้ากับ YANG ไป/กลับ/ระหว่างอุปกรณ์ |

### 🔹 SNMP protocol: 2 โหมด
- **request/response mode:** managing server ส่ง request → agent ตอบ response
- **trap mode:** agent ส่ง **trap message** แจ้ง server เองเมื่อเกิดเหตุ

| Message type | ทิศทาง | หน้าที่ |
|---|---|---|
| **GetRequest, GetNextRequest, GetBulkRequest** | manager → agent | "get me data" (ค่าเดียว, ค่าถัดไปในลิสต์, ทั้งก้อน) |
| **SetRequest** | manager → agent | ตั้งค่า MIB |
| **Response** | agent → manager | ส่งค่า ตอบ Request |
| **Trap** | agent → manager | **แจ้งเหตุการณ์ผิดปกติ** |

**Message formats**
- Get/set (PDU type 0–3): PDU type, Request ID, Error Status (0–5), Error Index, ตามด้วยคู่ Name–Value ของตัวแปร
- Trap (PDU type 4): PDU type, Enterprise, Agent Addr, Trap Type (0–7), Specific code, Time stamp, ตามด้วย Name–Value

### 🔹 MIB (Management Information Base)
- ข้อมูล operational (และ config บางส่วน) ของอุปกรณ์ รวมเป็น **MIB module**
  - **400 MIB modules** กำหนดใน RFC; มีของผู้ผลิตอีกมาก
- **SMI (Structure of Management Information):** **data definition language**
- ตัวอย่าง MIB สำหรับ UDP:

| Object ID | Name | Type | ความหมาย |
|---|---|---|---|
| 1.3.6.1.2.1.7.1 | UDPInDatagrams | 32-bit counter | จำนวน datagram ที่ส่งถึงทั้งหมด |
| 1.3.6.1.2.1.7.2 | UDPNoPorts | 32-bit counter | ส่งไม่ได้ (ไม่มี app ที่ port) |
| 1.3.6.1.2.1.7.3 | UDInErrors | 32-bit counter | ส่งไม่ได้ (เหตุอื่น) |
| 1.3.6.1.2.1.7.4 | UDPOutDatagrams | 32-bit counter | จำนวนที่ส่งออกทั้งหมด |
| 1.3.6.1.2.1.7.5 | udpTable | SEQUENCE | 1 entry ต่อ port ที่ใช้อยู่ |

---

## จุดที่มักสับสน

| เรื่อง | A | B |
|---|---|---|
| intra-AS vs inter-AS | ใน AS เดียว, เน้น performance (OSPF) | ระหว่าง AS, เน้น policy (BGP) |
| eBGP vs iBGP | คุยต่าง AS | กระจายใน AS |
| OSPF vs RIP | link-state + Dijkstra, flood ทั้ง AS | distance vector, ทุก 30 วิ |
| northbound vs southbound | controller ↔ app ด้านบน | controller ↔ switch ด้านล่าง (OpenFlow) |
| packet-in vs packet-out | switch → controller | controller → switch |
| SNMP request/response vs trap | manager ถาม agent ตอบ | agent แจ้งเอง |
| OpenFlow protocol vs API | คุย controller–switch ผ่าน TCP | ใช้กำหนด forwarding actions |

## จุดเน้นสำหรับสอบ
- 🔥 AS, intra vs inter, gateway router, ทำไมต้องแยก (policy, scale, performance)
- 🔥 OSPF: link-state, flood บน IP, Dijkstra, authenticate, hierarchical (area, backbone, border, boundary)
- 🔥 BGP: eBGP/iBGP, path vector, TCP, 4 messages, AS-PATH/NEXT-HOP, policy, hot potato, route selection 4 ขั้น
- 🔥 SDN: ทำไม centralized, 4 ลักษณะ, 3 ชั้น + northbound/southbound, OpenFlow messages, ตัวอย่าง link failure 6 ขั้น
- ⭐ ICMP type/code (0, 3/0–3, 8, 11) + traceroute
- ⭐ SNMP message types, 2 modes, MIB, SMI, CLI vs SNMP vs NETCONF/YANG
- ⚠️ ตรวจว่า Dijkstra/Bellman-Ford ออกสอบหรือไม่ (ไม่อยู่ในไฟล์)

## ✅ Coverage Check
- [x] Goals, roadmap, functions, per-router vs SDN, routing protocols goal
- [x] Scalable routing, AS, intra/inter, interconnected, inter-AS role
- [x] Intra-AS protocols (RIP, EIGRP, OSPF), OSPF, hierarchical OSPF
- [x] BGP: definition, eBGP/iBGP, basics, messages, attributes, advertisement, multiple paths, forwarding tables, hot potato, policy ×2, selection, why different
- [x] SDN: history, why centralized, analogy, traffic engineering ×3, 4 features, 3 layers, controller components, OpenFlow protocol/messages, example, ONOS, ORION, challenges, future
- [x] ICMP table, traceroute
- [x] Network management def, components, approaches, SNMP modes, messages, formats, MIB
