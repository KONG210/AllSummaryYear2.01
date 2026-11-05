# เตรียมสอบปลายภาค Network + ข้อเขียน

## รูปแบบข้อสอบ

> 📌 ตามที่อาจารย์แจ้ง (อาจมีการเปลี่ยนแปลง)

| ส่วน | จำนวน | เนื้อหา |
|---|---|---|
| ปรนัย | **50 ข้อ** | เนื้อหาในบท **44 ข้อ** + **บริษัทเอกชนที่มาบรรยาย 6 ข้อ** (บริษัทละ 3 ข้อ) |
| ข้อเขียน | **1 ข้อ** | แนวที่อาจออก: **(1) ออกแบบ white-box (2) ออกแบบ Protocol IPv6 (3) ออกแบบ Routing Protocol** |

> ⚠️ ข้อจากบริษัทเอกชน 6 ข้อ: **บริษัทที่ 1 = AIS (5G Introduction)** อ่านได้ที่บท [AIS · 5G Introduction](#net-ais) · บริษัทที่ 2 ยังไม่มีเนื้อหา ถ้ามีสไลด์ส่งมาเพิ่มได้

ทั้ง 3 แนวของข้อเขียนเป็นโจทย์ "ออกแบบ" ไม่มีคำตอบเดียวที่ถูก คะแนนอยู่ที่ **เหตุผลของการออกแบบ** และการใช้ศัพท์/กลไกจากบทเรียนให้ถูกที่ ด้านล่างมีโครงตอบกลาง แล้วตามด้วยคำตอบตัวอย่างทีละแนว

## 1. โครงตอบข้อเขียนแบบออกแบบ (ใช้ได้ทั้ง 3 แนว)

1. **โจทย์/เป้าหมาย** — เขียน 2–3 บรรทัดว่าระบบนี้ต้องทำอะไร ข้อกำหนดคืออะไร (สมมติขึ้นเองได้ ถ้าโจทย์ไม่ให้มา)
2. **ภาพรวม (diagram)** — วาดกล่อง + ลูกศร แสดงส่วนประกอบหลัก ได้คะแนนง่ายและทำให้อ่านต่อเข้าใจ
3. **ส่วนประกอบและหน้าที่** — ตารางหรือ bullet ทีละส่วน
4. **การทำงานทีละขั้น** — ยกสถานการณ์ 1 อย่างแล้วไล่ลำดับ 1, 2, 3, ...
5. **ตัวอย่างจริง** — ตาราง flow table / header / routing table ที่มีค่าจริง
6. **ข้อดี ข้อเสีย และเหตุผลที่เลือก** — เทียบกับทางเลือกอื่นอย่างน้อย 1 ทาง

> ⭐ เทคนิค: ใช้ศัพท์จากสไลด์ให้ตรง เช่น match+action, data plane / control plane, northbound / southbound API, flow label, next header, link-state, Dijkstra, AS, eBGP/iBGP, hot potato

## 2. แนวที่ 1: ออกแบบ White-box Network

### 🔹 white-box คืออะไร (ต้องเขียนนิยามก่อน)
- **White-box switch/router** = ฮาร์ดแวร์เครือข่ายแบบ **commodity (ทั่วไป ราคาถูก ไม่ผูกยี่ห้อ)** ที่ **แยกฮาร์ดแวร์ออกจากซอฟต์แวร์** แล้วควบคุมผ่าน **open API**
- ตรงข้ามกับแบบเดิม: **monolithic, vertically integrated, proprietary** (ฮาร์ดแวร์ + OS + protocol จากผู้ผลิตเดียว เช่น router ที่รัน Cisco IOS)
- จากสไลด์ Ch4 (Middleboxes): middlebox กำลังเปลี่ยนจาก proprietary hardware ไปเป็น **"whitebox" hardware ที่ใช้ open API**, ทำ action ด้วย **match+action**, แข่งกันที่ซอฟต์แวร์ และควบคุมแบบ **SDN** / **NFV**
- เปรียบเหมือน **mainframe → PC**: จาก vertically integrated, closed, slow innovation ไปเป็น horizontal, open interfaces, rapid innovation

### 🔹 ภาพรวมการออกแบบ

```text
 [ Network-control apps ]  routing (Dijkstra) | firewall | load balancer
            ↕  northbound API (REST / intent)
 [ SDN Controller ]  network OS, เก็บ network-wide state (เช่น ONOS)
            ↕  southbound API: OpenFlow (ผ่าน TCP)
 [ White-box switches ]  s1  s2  s3 ...  ทำ generalized forwarding ด้วย flow table
            ↕
 [ Hosts / Servers ]
```

### 🔹 ส่วนประกอบ
| ชั้น | ออกแบบอย่างไร | เหตุผล |
|---|---|---|
| **Data plane: white-box switch** | ฮาร์ดแวร์ทั่วไป เร็ว เรียบง่าย ทำ **generalized forwarding (match+action)** ใน hardware, มี **flow table** + OpenFlow agent | ถูก เปลี่ยนยี่ห้อได้ ไม่ต้องมี routing protocol ในตัว |
| **Control plane: SDN controller** | logically centralized แต่ implement เป็น **distributed system** (หลายเครื่อง) | เห็นทั้งเครือข่าย, กัน single point of failure, scale ได้ |
| **Apps** | routing, access control, load balancing เขียนเป็น app แยก (unbundled) | บุคคลที่สามเขียนได้, เพิ่มฟีเจอร์ได้โดยไม่เปลี่ยนฮาร์ดแวร์ |
| **Southbound** | **OpenFlow** บน TCP | มาตรฐานเปิด ทุกยี่ห้อคุยได้ |
| **Northbound** | REST API / **intent** ("what rather than how") | admin บอกสิ่งที่ต้องการ ไม่ต้องสั่งทีละ switch |

### 🔹 ออกแบบ flow table (ใส่ค่าจริง)
| Priority | Match | Action | ใช้ทำอะไร |
|---|---|---|---|
| 300 | TCP dst port = **22** | **drop** | firewall บล็อก SSH จากภายนอก |
| 300 | IP src = 128.119.1.1 | **drop** | บล็อก host ที่โจมตี |
| 200 | IP dst = 10.2.0.3 | forward(**port 3**) | destination-based forwarding |
| 200 | IP dst = 10.2.0.4 | forward(**port 4**) | |
| 100 | MAC dst = 22:A7:23:11:E1:02 | forward(port 3) | layer-2 switching |
| 0 | `*` (อื่นๆ ทั้งหมด) | **send to controller** | ไม่รู้จัก flow นี้ → ให้ controller ตัดสิน |

- แต่ละ entry มี **counters** (จำนวน packet/byte) ให้ controller เอาไปดูสถิติ
- **priority** ใช้ตัดสินตอน pattern ซ้อนกัน (กฎ firewall ต้อง priority สูงกว่า)
- ด้วย table แบบเดียวกัน switch ตัวเดียว**ทำหน้าที่ได้หลายแบบ**: router (longest prefix → forward), firewall (IP/port → permit/deny), switch (MAC → forward/flood), NAT (IP+port → rewrite)

### 🔹 การทำงานทีละขั้น
**กรณี A: packet ของ flow ใหม่**
1. packet เข้า switch s1 แล้วไม่ match กฎไหนเลย จึงไปตกที่กฎ `*` → send to controller
2. s1 ส่ง **packet-in** ไปให้ controller
3. routing app คำนวณเส้นทางจาก network graph ใน controller
4. controller ส่ง **modify-state** (เพิ่ม flow entry) ไปทุก switch บนเส้นทาง แล้วส่ง **packet-out** ให้ s1 ปล่อย packet แรกออกไป
5. packet ถัดๆ ไปของ flow นี้ match ใน hardware ทันที ไม่ต้องผ่าน controller อีก

**กรณี B: link เสีย (ตัวอย่างจากสไลด์ Ch5)**
1. s1 พบ link failure แล้วส่ง **port status** ไปแจ้ง controller
2. controller อัปเดต link status
3. Dijkstra routing app ที่ลงทะเบียนไว้ถูกเรียก แล้วคำนวณเส้นทางใหม่
4. flow-table computation สร้าง flow table ใหม่
5. controller ใช้ OpenFlow ติดตั้งตารางใหม่ใน switch ที่ได้รับผลกระทบ

### 🔹 ข้อดี / ข้อเสีย
| ข้อดี | ข้อเสีย / ความท้าทาย |
|---|---|
| ฮาร์ดแวร์ถูก ไม่ติด vendor lock-in | controller เป็นจุดสำคัญ ต้อง **hardening** (reliable, secure, fault-tolerant) |
| ทำ **traffic engineering** ได้ เช่น แบ่ง load ไปหลายเส้นทาง แยก traffic ตามประเภท ซึ่ง destination-based routing ทำไม่ได้ | ทีมต้องมีความรู้ซอฟต์แวร์ ต้องดูแล controller เอง |
| ลด router misconfiguration เพราะจัดการ config จากศูนย์กลาง | ขยายข้ามหลาย AS ยังยาก (Internet-scaling) |
| เพิ่มฟีเจอร์ด้วยซอฟต์แวร์ (NFV) เร็ว | flow แรกมี delay เพิ่ม เพราะต้องส่งไปถาม controller |

> 🔥 ตัวอย่างจริงที่ยกได้: **Google ORION** (SDN control plane ของ datacenter Jupiter และ WAN B4), **ONOS** (open-source controller)

## 3. แนวที่ 2: ออกแบบ Protocol IPv6

### 🔹 ตั้งโจทย์ก่อน: ทำไมต้องออกแบบใหม่
1. **address IPv4 32 bit ใกล้หมด** → ต้องการ address space ใหญ่ขึ้นมาก
2. router ต้อง **ประมวลผล/forward เร็วขึ้น**
3. รองรับ **QoS / flow** (เช่น video, voice ต้องการการดูแลต่างกัน)
4. ต้อง **อยู่ร่วมกับ IPv4** ระหว่างเปลี่ยนผ่าน เพราะเปลี่ยนพร้อมกันทั้งโลกไม่ได้ ("no flag days")

### 🔹 ออกแบบ header (fixed 40 bytes)

```text
 0      4       12                31
 +------+--------+------------------+
 | ver  |  pri   |   flow label     |
 +------+--------+--------+---------+
 |  payload len  |next hdr|hop limit|
 +---------------+--------+---------+
 |   source address (128 bits)      |
 +----------------------------------+
 | destination address (128 bits)   |
 +----------------------------------+
 |  data (payload / extension hdr)  |
```

| Field | หน้าที่ | เหตุผลที่ออกแบบแบบนี้ |
|---|---|---|
| **ver** | = 6 | แยก IPv4/IPv6 |
| **pri** (priority / traffic class) | ลำดับความสำคัญใน flow | รองรับ QoS |
| **flow label** | ระบุ packet ที่อยู่ใน flow เดียวกัน | router จัดการ flow ได้โดยไม่ต้องเปิดดู transport header |
| **payload len** | ความยาวข้อมูลหลัง header | header คงที่ จึงไม่ต้องมี header length |
| **next hdr** | protocol / header ถัดไป (เช่น TCP, UDP หรือ extension header) | **แทนที่ options** ทำให้ header หลักสั้นและคงที่ |
| **hop limit** | ลดทีละ 1 ทุก hop ถึง 0 ทิ้ง | เหมือน TTL กัน loop |
| **src / dst** | **128 bit** | address space ใหญ่มาก |

### 🔹 สิ่งที่ "ตัดออก" เทียบกับ IPv4 และเหตุผล (ออกสอบบ่อย)
| ตัดออก | เหตุผล |
|---|---|
| **checksum** | router ไม่ต้องคำนวณใหม่ทุก hop (เพราะ hop limit เปลี่ยน) จึงเร็วขึ้น และ link layer กับ transport layer ตรวจ error อยู่แล้ว |
| **fragmentation / reassembly ที่ router** | ลดงานของ router ถ้า packet ใหญ่เกินจะทิ้งแล้วส่ง ICMPv6 "**Packet Too Big**" กลับไป ให้ต้นทางลดขนาดเอง |
| **options** | ย้ายไปเป็น **extension header** ผ่าน next hdr ทำให้ header หลักยาวคงที่ 40 bytes |

### 🔹 ออกแบบ address (ส่วนเสริม ไม่ได้อยู่ในสไลด์ แต่ช่วยให้คำตอบสมบูรณ์)
- เขียนเป็น **hex 8 กลุ่ม กลุ่มละ 16 bit** คั่นด้วย `:` เช่น `2001:0db8:0000:0000:0000:ff00:0042:8329`
- ย่อได้: ตัด 0 นำหน้าในแต่ละกลุ่ม และแทนกลุ่ม 0 ที่ติดกันด้วย `::` ได้ **ครั้งเดียว** เช่น `2001:db8::ff00:42:8329`
- ประเภทที่ควรรู้: global unicast `2000::/3`, link-local `fe80::/10`, multicast `ff00::/8`, loopback `::1` (**ไม่มี broadcast** ใช้ multicast แทน)
- รูปแบบทั่วไป: **/64 แรก = network prefix**, **/64 หลัง = interface ID**

### 🔹 ออกแบบการเปลี่ยนผ่าน: Tunneling
- router อัปเกรดพร้อมกันไม่ได้ จึงต้องใช้ **tunneling**: ใส่ **IPv6 datagram ทั้งก้อนเป็น payload ของ IPv4 datagram** ("packet within a packet")
- ตัวอย่าง: A(v6) → B(v4/v6) → C(v4) → D(v4) → E(v4/v6) → F(v6)
  1. A→B: IPv6 ปกติ (src A, dst F)
  2. B→C→D→E: IPv6 ซ่อนอยู่ใน IPv4 ที่มี **src B, dst E**
  3. E แกะ IPv4 ออก แล้ว E→F เป็น IPv6 ปกติ
- มุมมองเชิงตรรกะ: B กับ E ต่อกันด้วย "อุโมงค์"
- ทางเลือกอื่นที่เขียนเสริมได้: **dual-stack** (อุปกรณ์รันทั้ง IPv4 และ IPv6)

### 🔹 สรุปข้อดี / ข้อเสีย
| ข้อดี | ข้อเสีย |
|---|---|
| address แทบไม่จำกัด ไม่ต้องพึ่ง NAT | header ใหญ่ขึ้น (40 bytes เทียบกับ 20 bytes ของ IPv4) |
| router ทำงานเร็วขึ้น (ไม่มี checksum / fragmentation / options) | เปลี่ยนผ่านช้ามาก (25 ปียังไม่จบ ราว 40% ของ client Google ใช้ IPv6 ในปี 2023) |
| รองรับ flow / QoS | ต้องมี tunneling ระหว่างเปลี่ยนผ่าน ซึ่งเพิ่ม overhead |

## 4. แนวที่ 3: ออกแบบ Routing Protocol

### 🔹 ตั้งโจทย์ (ตัวอย่างสถานการณ์)
> บริษัทมี 3 สาขา (กรุงเทพ เชียงใหม่ ภูเก็ต) router รวม ~30 ตัว และต่อ Internet ผ่าน **ISP 2 เจ้า** (ISP-A หลัก, ISP-B สำรอง) ให้ออกแบบ routing

**ข้อกำหนด:** หาเส้นทาง cost ต่ำสุดภายในบริษัท, ปรับตัวเมื่อ link เสีย, ขยายได้, ไม่ยอมเป็นทางผ่าน (transit) ระหว่าง ISP, ปลอดภัย

### 🔹 ภาพรวม: แบ่งเป็น 2 ระดับ

```text
       ISP-A (AS 100)        ISP-B (AS 200)
            \  eBGP            /  eBGP
          [GW1] ---- iBGP ---- [GW2]      ← gateway routers (รันทั้ง BGP + OSPF)
             \                  /
         ===== OSPF Area 0 (backbone) =====
           /          |           \
      Area 1        Area 2        Area 3
     (กรุงเทพ)     (เชียงใหม่)     (ภูเก็ต)
```

| ระดับ | Protocol | เหตุผล |
|---|---|---|
| **Intra-AS** (ภายในบริษัท = AS เดียว) | **OSPF** (link-state, Dijkstra) | admin คนเดียว เน้น **performance**, converge เร็ว, รองรับ authentication |
| **Inter-AS** (ระหว่างบริษัทกับ ISP) | **BGP** (path vector) | ต้องใช้ **policy** และเป็นมาตรฐานของ Internet ("glue") |

### 🔹 ออกแบบ Intra-AS: OSPF แบบลำดับชั้น
1. แบ่งเป็น **Area 0 (backbone)** และ **Area 1–3** (สาขาละ area)
2. router ในแต่ละ area **flood link-state advertisement เฉพาะใน area** ของตัวเอง ทำให้ตารางเล็กและ traffic ของ update น้อยลง
3. **Area border router** สรุป (summarize) เส้นทางของ area แล้วประกาศเข้า backbone
4. **link cost** กำหนดจาก bandwidth (link เร็ว = cost ต่ำ) หรือจาก delay
5. ทุก router มี topology ของ area ครบ แล้วใช้ **Dijkstra** คำนวณ forwarding table เอง
6. OSPF message ทุกอันต้อง **authenticate** และส่งบน IP โดยตรง

### 🔹 ออกแบบ Inter-AS: BGP
1. GW1 เปิด **eBGP session** (TCP) กับ ISP-A ส่วน GW2 เปิดกับ ISP-B
2. GW1 กับ GW2 คุยกันด้วย **iBGP** เพื่อกระจายเส้นทางภายนอกให้ router ทั้งบริษัท
3. **นโยบาย (policy):**
   - **ไม่เป็น transit:** เส้นทางที่ได้จาก ISP-A จะไม่ประกาศต่อให้ ISP-B และกลับกัน (เหมือนตัวอย่าง **dual-homed x** ในสไลด์)
   - ใช้ ISP-A เป็นหลัก: ตั้ง **local preference** ของเส้นทางจาก ISP-A ให้สูงกว่า
   - **import policy:** ปฏิเสธเส้นทางที่น่าสงสัย เช่น prefix ที่ไม่ควรมาจาก ISP นั้น
4. **การเลือกเส้นทาง** ตามลำดับ: local preference → AS-PATH สั้นสุด → NEXT-HOP ใกล้สุด (**hot potato**) → เกณฑ์อื่น
5. **รวมเป็น forwarding table:** BGP บอกว่าจะ "ออกทาง gateway ไหน" ส่วน OSPF บอกว่า "ไป gateway นั้นใช้ interface ไหน"

### 🔹 การทำงานเมื่อ link เสีย
1. router ที่ติดกับ link นั้นตรวจพบ แล้ว flood LSA ใหม่ใน area
2. ทุก router ใน area รัน Dijkstra ใหม่ แล้วได้เส้นทางใหม่
3. ถ้า ISP-A ล่มทั้งหมด BGP session ของ GW1 จะหลุด เส้นทางผ่าน ISP-A ถูกถอนออก (UPDATE withdraw) แล้ว traffic สลับไปออก ISP-B เอง

### 🔹 ทำไมเลือก Link-State มากกว่า Distance Vector (ภายใน AS)
| | Link-State (OSPF) | Distance Vector (RIP) |
|---|---|---|
| ข้อมูลที่รู้ | **topology ทั้งหมด** | แค่ระยะทางจากเพื่อนบ้าน |
| อัลกอริทึม | **Dijkstra** | **Bellman-Ford** |
| ส่งข้อมูลให้ใคร | flood ให้**ทุก router** ใน area | ส่งให้**เพื่อนบ้าน**เท่านั้น |
| ความเร็วในการ converge | เร็ว | ช้า อาจเกิด **count-to-infinity** |
| ภาระ | ใช้ CPU / memory มากกว่า | เบากว่า |

### 🔹 Dijkstra (Link-State) — ตัวอย่างคำนวณ
> 📌 ส่วนนี้ไม่อยู่ในไฟล์สไลด์ที่ได้มา แต่เป็นเนื้อหามาตรฐานของหนังสือ Kurose ที่สไลด์อ้างอิง ถ้าข้อเขียนให้ออกแบบ routing มักต้องแสดงการคำนวณ

กราฟ: u–v 2, u–x 1, u–w 5, v–x 2, v–w 3, x–w 3, x–y 1, w–y 1, w–z 5, y–z 2 (หาเส้นทางจาก **u**)

| ขั้น | N' | D(v),p | D(w),p | D(x),p | D(y),p | D(z),p |
|---|---|---|---|---|---|---|
| 0 | u | 2,u | 5,u | **1,u** | ∞ | ∞ |
| 1 | ux | **2,u** | 4,x | | 2,x | ∞ |
| 2 | uxv | | 4,x | | **2,x** | ∞ |
| 3 | uxvy | | **3,y** | | | 4,y |
| 4 | uxvyw | | | | | **4,y** |
| 5 | uxvywz | | | | | |

- แต่ละขั้น เลือก node ที่ D ต่ำสุดซึ่งยังไม่อยู่ใน N' แล้วอัปเดต `D(n) = min(D(n), D(w) + c(w,n))`
- ขั้น 1: w ลดจาก 5 เหลือ 1+3 = 4 (ผ่าน x), y = 1+1 = 2
- ขั้น 3: w ลดเหลือ 2+1 = 3 (ผ่าน y), z = 2+2 = 4

**Forwarding table ของ u**
| ปลายทาง | ออก link |
|---|---|
| v | (u,v) |
| x, y, w, z | (u,x) |

### 🔹 Bellman-Ford (Distance Vector)
- สูตร: **dx(y) = minᵥ { c(x,v) + dᵥ(y) }**
- ตัวอย่าง (กราฟเดิม): dv(z) = 5, dx(z) = 3, dw(z) = 3
  - du(z) = min{ c(u,v)+dv(z), c(u,x)+dx(z), c(u,w)+dw(z) } = min{ 2+5, 1+3, 5+3 } = **4** (ผ่าน x)
- node ส่ง DV ให้เพื่อนบ้านเมื่อค่าเปลี่ยน แต่ข่าวร้าย (cost เพิ่ม) จะกระจายช้า ทำให้เกิด **count-to-infinity** ซึ่งแก้บางส่วนได้ด้วย **poisoned reverse**

## จุดเน้นสำหรับสอบ
- 🔥 ข้อเขียน: เขียนตามโครง **เป้าหมาย → diagram → ส่วนประกอบ → ขั้นตอน → ตัวอย่างตาราง → ข้อดี/ข้อเสีย**
- 🔥 white-box: commodity hardware + open API + SDN (controller, northbound/southbound, OpenFlow), match+action, packet-in / modify-state / port status
- 🔥 IPv6: header 40 bytes, 128 bit, flow label, next header, hop limit, **ตัด checksum / fragmentation / options พร้อมเหตุผล**, tunneling
- 🔥 Routing: intra (OSPF, areas, Dijkstra) + inter (BGP, eBGP/iBGP, policy, local pref, hot potato), LS vs DV
- ⚠️ อย่าลืม 6 ข้อจากบริษัทเอกชน ทวนโน้ตจากตอนบรรยายด้วย
