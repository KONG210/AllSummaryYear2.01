# Chapter 6 · Transport Layer

> ℹ️ สไลด์ใช้เลขหน้า "Transport Layer: 3-x" เพราะในหนังสือคือบทที่ 3 แต่วิชานี้เรียกเป็น Chapter 6 · ใน roadmap มี "Principles of reliable data transfer" แต่ไฟล์ **ไม่มีสไลด์ rdt / Go-Back-N / Selective Repeat** — เช็กกับอาจารย์ว่าออกสอบหรือไม่

## ภาพรวมของบท

**เป้าหมาย**
- เข้าใจหลักการของบริการ transport layer: **multiplexing/demultiplexing, reliable data transfer, flow control, congestion control**
- รู้จัก protocol จริง: **UDP** (connectionless), **TCP** (connection-oriented, reliable), **TCP congestion control**

**Roadmap:** Transport services → Multiplexing/demultiplexing → UDP → TCP (segment, reliable transfer, flow control, connection management) → Principles of congestion control → TCP congestion control → Evolution (QUIC)

> 🔗 ภาพใหญ่: network layer (บท 4–5) ส่ง datagram ระหว่าง **host** → transport layer ส่งต่อให้ถึง **process** ที่ถูกต้อง (demux ด้วย port) → **UDP** ส่งเร็วแบบไม่รับประกัน, **TCP** เพิ่มความน่าเชื่อถือ (seq/ACK, timeout, retransmit) + **flow control** (ไม่ท่วมผู้รับ) + **congestion control** (ไม่ท่วมเครือข่าย)

---

## 1. Transport services and protocols

- ให้ **logical communication ระหว่าง application process** ที่รันบน host ต่างกัน
- ทำงานที่ **end systems** (ไม่ใช่ router)
  - **ผู้ส่ง:** หั่น application message เป็น **segment** แล้วส่งให้ network layer
  - **ผู้รับ:** ประกอบ segment กลับเป็น message แล้วส่งให้ application layer
- Internet มี 2 protocol: **TCP, UDP**

### 🔹 Transport Layer Actions
**Sender:** รับ app message → กำหนดค่า header field → สร้าง segment → ส่งให้ IP
**Receiver:** รับ segment จาก IP → ตรวจ header → ดึง app message ออก → **demultiplex ขึ้นไปยัง application ผ่าน socket**

### 🔹 Two principal Internet transport protocols
| | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---|---|---|
| ความน่าเชื่อถือ | **reliable, in-order** | **unreliable, unordered** |
| congestion control | มี | ไม่มี |
| flow control | มี | ไม่มี |
| connection setup | มี | ไม่มี |
| ลักษณะ | | "**no-frills**" ส่วนขยายของ best-effort IP |

**บริการที่ไม่มีทั้งสองตัว:** **delay guarantees**, **bandwidth guarantees**

---

## 2. Multiplexing / Demultiplexing

| | ความหมาย |
|---|---|
| **Multiplexing (ผู้ส่ง)** | รับข้อมูลจาก **หลาย socket** แล้วใส่ transport header (ใช้ภายหลังตอน demux) |
| **Demultiplexing (ผู้รับ)** | ใช้ข้อมูลใน header ส่ง segment ไปยัง **socket ที่ถูกต้อง** |

**ตัวอย่างคำถาม:** transport layer รู้ได้อย่างไรว่าต้องส่ง message ให้ Firefox ไม่ใช่ Netflix หรือ Skype? → ใช้ **port number** (+ IP)

### 🔹 Demultiplexing ทำงานอย่างไร
- host ได้ IP datagram: แต่ละอันมี **source IP, destination IP** และบรรจุ transport segment 1 อัน
- segment มี **source port #, dest port #** (แต่ละ field 16 bit ในแถวแรกของ header 32 bit)
- host ใช้ **IP + port** ส่ง segment ไปยัง socket ที่เหมาะสม

### 🔹 Connectionless demultiplexing (UDP)
- ตอนสร้าง socket ต้องระบุ **host-local port #** เช่น `DatagramSocket mySocket1 = new DatagramSocket(12534);`
- ตอนส่ง datagram ต้องระบุ **destination IP + destination port #**
- ผู้รับเช็ก **destination port #** แล้วส่งไป socket ที่มี port นั้น
- ⭐ datagram ที่มี **dest port เดียวกัน แต่ source IP/port ต่างกัน → ไป socket เดียวกัน**

**ตัวอย่าง:** host ที่ bind port 6428 ส่งไป 9157 → segment: source 6428, dest 9157; ตอบกลับ: source 9157, dest 6428 (สลับกัน)

### 🔹 Connection-oriented demultiplexing (TCP)
- TCP socket ระบุด้วย **4-tuple:** source IP, source port, dest IP, dest port
- ผู้รับใช้ **ทั้ง 4 ค่า** หา socket
- server รองรับหลาย TCP socket พร้อมกัน แต่ละ socket มี 4-tuple ของตัวเอง ผูกกับ client คนละคน

**ตัวอย่าง:** 3 segment ไปที่ B port 80 ทั้งหมด: (A, 9157), (C, 5775), (C, 9157) → **ไปคนละ socket** เพราะ 4-tuple ต่างกัน

### 🔹 Summary
- mux/demux ใช้ค่าใน header ของ segment/datagram
- **UDP: demux ด้วย destination port อย่างเดียว**
- **TCP: demux ด้วย 4-tuple**
- mux/demux เกิดขึ้น **ทุก layer**

---

## 3. UDP: User Datagram Protocol (RFC 768)

### 🔹 ลักษณะ
- "**no frills, bare bones**" Internet transport protocol
- **best effort:** segment อาจ **หาย** หรือ **มาถึง app ไม่ตามลำดับ**
- **connectionless:** ไม่มี handshaking; แต่ละ segment จัดการ **อิสระ** จากกัน

### 🔹 ทำไมต้องมี UDP?
- **ไม่ต้องตั้ง connection** (ซึ่งเพิ่ม RTT delay)
- **เรียบง่าย:** ไม่มี connection state ที่ผู้ส่ง/ผู้รับ
- **header เล็ก**
- **ไม่มี congestion control** → ส่งเร็วได้เท่าที่ต้องการ, ทำงานได้แม้เครือข่ายแออัด

### 🔹 ใช้กับอะไร
- **streaming multimedia** (ทนการสูญหาย, ไวต่อ rate)
- **DNS**, **SNMP**, **HTTP/3**
- ถ้าต้องการ reliability บน UDP (เช่น HTTP/3): **เพิ่ม reliability และ congestion control ที่ application layer**

### 🔹 UDP actions (ตัวอย่าง SNMP client/server)
- **sender:** รับ SNMP msg → กำหนด UDP header → สร้าง UDP segment → ส่งให้ IP
- **receiver:** รับจาก IP → **ตรวจ UDP checksum** → ดึง message → demux ผ่าน socket

### 🔹 UDP segment header (8 bytes)
```text
|<----------- 32 bits ----------->|
| source port #  | dest port #    |
| length         | checksum       |
|   application data (payload)    |
```
- **length** = ความยาวของ UDP segment เป็น bytes **รวม header**

### 🔹 UDP checksum
**เป้าหมาย:** ตรวจจับ error (bit พลิก) ใน segment
- ตัวอย่างง่าย: ส่ง 5, 6 sum 11 → ได้รับ 4, 6 → ผู้รับคำนวณได้ 10 ≠ 11 → error

### 🔹 Internet checksum
**sender**
- มองเนื้อหา UDP segment (**รวม UDP header และ IP addresses**) เป็นลำดับของ **16-bit integers**
- checksum = **one's complement ของผลรวม (one's complement sum)**
- ใส่ค่าใน checksum field

**receiver**
- คำนวณ checksum ของ segment ที่ได้รับ
- เทียบกับค่าใน checksum field: **ไม่เท่า = เจอ error** · **เท่า = ไม่เจอ error** (แต่อาจยังมี error ก็ได้)

### 🧮 ตัวอย่างคำนวณ (ต้องทำได้)
```text
            1110011001100110
          + 1101010101010101
          ------------------
         1  1011101110111011     ← มี carry ออกจาก bit ซ้ายสุด
wraparound  →         +     1     ← เอา carry มาบวกกลับที่บิตขวาสุด
          ------------------
sum         1011101110111100
checksum    0100010001000011     ← กลับบิตทุกตัว (one's complement)
```

**ขั้นตอน**
1. บวกเลข 16 bit ทีละคู่
2. ถ้ามี **carry ออกจาก bit ซ้ายสุด → บวกกลับ (wraparound)**
3. **กลับบิตทุกตัว** ของผลรวม = checksum
4. (ตรวจ) ผู้รับบวกทุกคำ + checksum ควรได้ 1111111111111111

### 🔹 Internet checksum: weak protection!
ถ้า bit พลิกในตำแหน่งที่หักล้างกัน (เช่น bit หนึ่งของเลขแรก 0→1 และบิตเดียวกันของเลขที่สอง 1→0) **ผลรวมไม่เปลี่ยน → checksum เหมือนเดิม** ตรวจไม่เจอ

### 🔹 Summary: UDP
- "no frills": อาจหาย มาไม่ตามลำดับ — best effort "send and hope for the best"
- ข้อดี: **ไม่ต้อง setup (ไม่เสีย RTT)**, ทำงานได้เมื่อเครือข่ายมีปัญหา, ช่วยเรื่อง reliability ด้วย checksum
- สร้างฟังก์ชันเพิ่มบน UDP ที่ application layer ได้ (เช่น HTTP/3)

---

## 4. TCP: Overview (RFCs 793, 1122, 2018, 5681, 7323)

| คุณสมบัติ | ความหมาย |
|---|---|
| **point-to-point** | ผู้ส่ง 1 ผู้รับ 1 |
| **reliable, in-order byte stream** | **ไม่มี "message boundaries"** |
| **full duplex** | ส่งข้อมูล **สองทิศทาง** ใน connection เดียวกัน; **MSS** = maximum segment size |
| **cumulative ACKs** | ACK บอกว่าได้รับครบถึง byte ไหน |
| **pipelining** | congestion + flow control กำหนด window size |
| **connection-oriented** | handshaking เตรียม state ผู้ส่ง/ผู้รับก่อนส่งข้อมูล |
| **flow controlled** | ผู้ส่งจะไม่ส่งท่วมผู้รับ |

### 🔹 TCP segment structure
```text
|<---------------- 32 bits ---------------->|
| source port #        | dest port #        |
|            sequence number                |
|         acknowledgement number            |
| head | not | C E U A P R S F | receive    |
| len  | used|                 | window     |
| checksum             | Urg data pointer   |
|        options (variable length)          |
|   application data (variable length)      |
```
| Field | ความหมาย |
|---|---|
| **sequence number** | นับเป็น **bytes ของข้อมูล** ใน bytestream (**ไม่ใช่นับ segment!**) |
| **acknowledgement number** | seq # ของ **byte ถัดไปที่คาดว่าจะได้รับ**; ใช้คู่กับ **A bit** (บอกว่าเป็น ACK) |
| **head len** | ความยาว TCP header |
| **C, E** | congestion notification (ECN) |
| **RST, SYN, FIN** | connection management |
| **receive window** | flow control: จำนวน bytes ที่ผู้รับยินดีรับ |
| **checksum** | Internet checksum |
| **options** | TCP options |

### 🔹 Sequence numbers, ACKs
- **Sequence number** = "เลข" ใน byte stream ของ **byte แรก** ในข้อมูลของ segment
- **ACK** = seq # ของ **byte ถัดไปที่คาดหวัง** จากอีกฝั่ง → **cumulative ACK**
- **ผู้รับจัดการ segment ที่มาไม่ตามลำดับอย่างไร?** → **TCP spec ไม่ได้กำหนด ขึ้นอยู่กับผู้ implement**
- sender sequence number space: sent & ACKed | sent not-yet ACKed (**"in-flight"**) | usable not yet sent | not usable (ขนาด window = N)

### 🧮 ตัวอย่าง: simple telnet scenario
| ขั้น | จาก | Seq | ACK | data |
|---|---|---|---|---|
| 1 | A (ผู้ใช้พิมพ์ 'C') | **42** | **79** | 'C' |
| 2 | B (ACK + echo 'C') | **79** | **43** | 'C' |
| 3 | A (ACK echo) | **43** | **80** | — |

**หลักคิด:** ACK = Seq ที่ได้รับ + จำนวน bytes ที่ได้รับ (42 + 1 = 43) · Seq ครั้งถัดไปของตัวเอง = ACK ที่อีกฝ่ายส่งมา

---

## 5. TCP Round Trip Time และ Timeout

**ตั้ง timeout เท่าไหร่?**
- ต้อง **นานกว่า RTT** แต่ RTT เปลี่ยนตลอด
- **สั้นไป:** premature timeout → retransmit โดยไม่จำเป็น
- **ยาวไป:** ตอบสนองต่อ segment ที่หายช้า

**ประมาณ RTT อย่างไร?**
- **SampleRTT** = เวลาจากส่ง segment จนได้ ACK (**ไม่นับ segment ที่ retransmit**)
- SampleRTT แกว่ง → อยากได้ค่าที่ "smoother" → **เฉลี่ยหลายค่าล่าสุด**

### 🧮 สูตร (ต้องจำ)
```text
EstimatedRTT   = (1 − α) × EstimatedRTT + α × SampleRTT        α = 0.125 (ทั่วไป)
DevRTT         = (1 − β) × DevRTT + β × |SampleRTT − EstimatedRTT|   β = 0.25 (ทั่วไป)
TimeoutInterval = EstimatedRTT + 4 × DevRTT
```
- EstimatedRTT = **exponential weighted moving average (EWMA)** — อิทธิพลของค่าเก่าลดลงแบบ exponential
- **4 × DevRTT** = "safety margin" — ถ้า RTT แกว่งมากก็เผื่อมาก

### 🧮 ตัวอย่างคำนวณ
ให้ EstimatedRTT = 100 ms, DevRTT = 10 ms, SampleRTT ใหม่ = 120 ms, α = 0.125, β = 0.25
1. EstimatedRTT = 0.875 × 100 + 0.125 × 120 = 87.5 + 15 = **102.5 ms**
2. DevRTT = 0.75 × 10 + 0.25 × |120 − 102.5| = 7.5 + 4.375 = **11.875 ms**
3. TimeoutInterval = 102.5 + 4 × 11.875 = 102.5 + 47.5 = **150 ms**

> ⚠️ ในตัวอย่างนี้ใช้ EstimatedRTT **ค่าใหม่** ในสูตร DevRTT ตามลำดับในสไลด์ บางตำรา (RFC 6298) อัปเดต DevRTT ก่อนด้วยค่าเก่า — ถ้าอาจารย์ทำแบบไหนให้ทำตามนั้น

---

## 6. TCP reliable data transfer

### 🔹 TCP Sender (simplified) — 3 events
| Event | ทำอะไร |
|---|---|
| **data received from application** | สร้าง segment พร้อม seq # (= byte-stream number ของ byte แรก); **start timer ถ้ายังไม่ทำงาน** (timer = ของ segment ที่ยังไม่ ACK ที่เก่าที่สุด, expiration = TimeOutInterval) |
| **timeout** | **retransmit segment ที่ทำให้ timeout**; restart timer |
| **ACK received** | ถ้า ACK ยืนยัน segment ที่ยังไม่เคย ACK: อัปเดตสิ่งที่ถูก ACK แล้ว; start timer ถ้ายังมี segment ที่ไม่ได้ ACK |

### 🔹 TCP Receiver: ACK generation (RFC 5681)
| Event ที่ผู้รับ | Action |
|---|---|
| segment มาตามลำดับ seq ที่คาด, ข้อมูลก่อนหน้า ACK หมดแล้ว | **delayed ACK:** รอ segment ถัดไปได้ถึง **500 ms** ถ้าไม่มีค่อยส่ง ACK |
| segment มาตามลำดับ และมีอีก segment รอ ACK อยู่ | **ส่ง cumulative ACK ทันที 1 อัน** ACK ทั้งสอง segment |
| segment มา **ไม่ตามลำดับ** (seq สูงกว่าที่คาด) — **เจอ gap** | **ส่ง duplicate ACK ทันที** (ระบุ seq ของ byte ถัดไปที่คาด) |
| segment ที่ **เติม gap** บางส่วนหรือทั้งหมด | ส่ง ACK ทันที ถ้า segment เริ่มที่ขอบล่างของ gap |

### 🔹 Retransmission scenarios
1. **Lost ACK:** A ส่ง Seq=92 (8 bytes), ACK=100 หาย → timeout → ส่ง Seq=92 ใหม่ → B ตอบ ACK=100 → SendBase = 100
2. **Premature timeout:** A ส่ง Seq=92 (8 bytes) และ Seq=100 (20 bytes) → ACK มาช้า timeout ก่อน → ส่ง Seq=92 ใหม่ → B ตอบ **cumulative ACK=120** (เพราะได้ครบถึง 119 แล้ว) → SendBase = 120
3. **Cumulative ACK ครอบ ACK ที่หาย:** ACK=100 หาย แต่ ACK=120 มาถึงก่อน timeout → รู้ว่าได้รับถึง 119 แล้ว ไม่ต้อง retransmit → ส่ง Seq=120 (15 bytes) ต่อ

### 🔹 TCP fast retransmit
- ถ้าผู้ส่งได้ **ACK เพิ่ม 3 อันสำหรับข้อมูลเดียวกัน ("triple duplicate ACKs")** → **ส่ง segment ที่ยังไม่ ACK ที่มี seq น้อยที่สุดใหม่ทันที**
- น่าจะหายแล้ว **ไม่ต้องรอ timeout**
- เหตุผล: dup ACK 3 อัน = ผู้รับได้ 3 segment หลังจาก segment ที่หาย

---

## 7. TCP Flow Control

**คำถาม:** ถ้า network layer ส่งข้อมูลเข้ามาเร็วกว่าที่ application ดึงออกจาก socket buffer จะเกิดอะไร? → buffer ล้น

> 📌 **Flow control:** **ผู้รับควบคุมผู้ส่ง** เพื่อไม่ให้ผู้ส่ง **ท่วม buffer ผู้รับ** ด้วยการส่งเยอะเกินไป เร็วเกินไป

**การทำงาน**
- ผู้รับ **"advertise" พื้นที่ buffer ว่าง** ใน field **rwnd** ของ TCP header
  - **RcvBuffer** ตั้งผ่าน socket options (**default ทั่วไป 4096 bytes**); หลาย OS ปรับอัตโนมัติ
- ผู้ส่ง **จำกัดข้อมูลที่ยังไม่ ACK ("in-flight") ไม่เกิน rwnd** ที่ได้รับ
- **รับประกันว่า receive buffer จะไม่ล้น**

```text
RcvBuffer = buffered data + rwnd (free buffer space)
```

---

## 8. TCP Connection Management

**ก่อนส่งข้อมูล ผู้ส่ง/ผู้รับ "handshake":**
- ตกลงว่าจะสร้าง connection (แต่ละฝ่ายรู้ว่าอีกฝ่ายยินดี)
- ตกลง **parameter** (เช่น **starting seq #**)
- state ของแต่ละฝ่าย: ESTAB, seq # client-to-server / server-to-client, rcvBuffer size

### 🔹 ทำไม 2-way handshake ไม่พอ
2-way: client ส่ง `req_conn(x)` → server ตอบ `acc_conn(x)` → ESTAB

ปัญหาบนเครือข่ายจริง: **delay ไม่แน่นอน, message ถูก retransmit เพราะหาย, message สลับลำดับ, มองไม่เห็นอีกฝั่ง**

| Scenario | เกิดอะไร |
|---|---|
| ปกติ | ไม่มีปัญหา |
| `req_conn(x)` ถูก retransmit มาถึงหลัง client จบไปแล้ว | server เปิด connection ใหม่ → **half open connection (ไม่มี client)** |
| `req_conn(x)` และ `data(x+1)` เก่าถูก retransmit มาถึงภายหลัง | server **รับข้อมูลซ้ำ (dup data accepted)** |

### 🔹 TCP 3-way handshake
| ขั้น | ผู้ส่ง | Segment | state |
|---|---|---|---|
| 1 | client เลือก initial seq **x** | **SYN**: SYNbit=1, Seq=x | client → **SYNSENT** |
| 2 | server เลือก initial seq **y** | **SYNACK**: SYNbit=1, Seq=y, ACKbit=1, **ACKnum=x+1** | server → **SYN RCVD** |
| 3 | client ได้ SYNACK (รู้ว่า server live) | **ACK**: ACKbit=1, **ACKnum=y+1** (segment นี้ **ใส่ข้อมูล client→server ได้**) | client → **ESTAB** |
| — | server ได้ ACK(y) (รู้ว่า client live) | | server → **ESTAB** |

- server: `socket(AF_INET, SOCK_STREAM)` → `bind` → `listen` (state **LISTEN**) → `accept()`
- client: `socket(...)` → `connect((serverName, serverPort))`
- เปรียบเทียบ: การปีนเขา "1. On belay? 2. Belay on. 3. Climbing."

### 🔹 Closing a TCP connection
- client และ server **ต่างฝ่ายต่างปิดฝั่งตัวเอง** โดยส่ง segment ที่ **FIN bit = 1**
- ตอบ FIN ที่ได้รับด้วย **ACK** (รวม ACK กับ FIN ของตัวเองได้)
- รองรับการส่ง FIN พร้อมกันได้

---

## 9. Principles of Congestion Control

> 📌 **Congestion (ไม่เป็นทางการ):** "too many sources sending too much data too fast for network to handle"

**อาการ:** **delay นาน** (คิวใน router buffer) · **packet loss** (buffer ของ router ล้น)

| | Congestion control | Flow control |
|---|---|---|
| ปัญหา | **ผู้ส่งหลายคน** ส่งเร็วเกินไปสำหรับ **เครือข่าย** | **ผู้ส่งคนเดียว** เร็วเกินไปสำหรับ **ผู้รับคนเดียว** |
| (top-10 problem) | ✓ | |

### 🔹 2 แนวทาง
| | End-end congestion control | Network-assisted congestion control |
|---|---|---|
| feedback จากเครือข่าย | **ไม่มี explicit feedback** | router **ให้ feedback โดยตรง** แก่ host ที่มี flow ผ่าน router ที่แออัด |
| รู้ว่าแออัดได้อย่างไร | **อนุมานจาก loss, delay** ที่สังเกตได้ | บอกระดับความแออัด หรือ **ตั้ง sending rate ให้** |
| ตัวอย่าง | **TCP** | **TCP ECN, ATM, DECbit** |

---

## 10. TCP Congestion Control

### 🔹 AIMD (Additive Increase Multiplicative Decrease)
- **แนวคิด:** ผู้ส่งเพิ่ม rate ไปเรื่อย ๆ จนเกิด packet loss แล้วลด rate เมื่อเกิด loss
- **Additive Increase:** เพิ่ม rate **1 MSS ทุก RTT** จนตรวจพบ loss
- **Multiplicative Decrease:** **ลดครึ่ง** ทุกครั้งที่เกิด loss
- ได้กราฟ **"sawtooth" (ฟันเลื่อย)** = probing for bandwidth

**รายละเอียดการลด**
- loss ตรวจพบด้วย **triple duplicate ACK → ลดครึ่ง (TCP Reno)**
- loss ตรวจพบด้วย **timeout → ลดเหลือ 1 MSS (TCP Tahoe)**

**ทำไม AIMD?** เป็น algorithm กระจาย asynchronous ที่พิสูจน์แล้วว่า **optimize congested flow rate ทั้งเครือข่าย** [Kelly] และ **เสถียร**

### 🔹 cwnd (congestion window)
- ส่งประมาณ **cwnd bytes** แล้วรอ RTT รับ ACK แล้วส่งต่อ
- **ผู้ส่งจำกัด:** `LastByteSent − LastByteAcked ≤ cwnd`
- cwnd ปรับตามความแออัดที่สังเกตได้

```text
TCP rate ≈ cwnd / RTT   bytes/sec
```
> ⭐ ข้อมูล in-flight จริงถูกจำกัดโดยทั้ง cwnd (congestion) และ rwnd (flow control)

### 🔹 Slow Start
- เมื่อเริ่ม connection **เพิ่ม rate แบบ exponential** จนเกิด loss ครั้งแรก
  - เริ่ม **cwnd = 1 MSS**
  - **cwnd เพิ่มเป็นสองเท่าทุก RTT** (ทำโดยเพิ่ม cwnd 1 MSS **ต่อทุก ACK** ที่ได้รับ)
- เริ่มช้า แต่เร่งขึ้นเร็วแบบ exponential

### 🔹 จาก Slow Start ไป Congestion Avoidance
- **เปลี่ยนจาก exponential เป็น linear เมื่อไหร่?** → เมื่อ cwnd ถึง **ครึ่งหนึ่งของค่าก่อน timeout**
- ใช้ตัวแปร **ssthresh**: เมื่อเกิด loss **ssthresh = cwnd / 2** (ค่า cwnd ก่อน loss)

### 🔹 Summary: TCP congestion control (state machine)
| State | เมื่อได้ new ACK | ไปสถานะอื่นเมื่อ |
|---|---|---|
| **Slow start** (เริ่ม cwnd = 1 MSS, ssthresh = 64 KB) | cwnd += 1 MSS (×2 ต่อ RTT) | **cwnd > ssthresh → congestion avoidance** · timeout → ssthresh = cwnd/2, cwnd = 1 (อยู่ slow start) · 3 dup ACK → fast recovery |
| **Congestion avoidance** | cwnd += MSS × (MSS/cwnd) (≈ +1 MSS ต่อ RTT) | timeout → ssthresh = cwnd/2, cwnd = 1 MSS, ไป **slow start** · 3 dup ACK → fast recovery |
| **Fast recovery** | new ACK → **cwnd = ssthresh** → ไป **congestion avoidance** | dup ACK → cwnd += 1 MSS · timeout → ssthresh = cwnd/2, cwnd = 1, ไป slow start |

**เมื่อได้ dup ACK ครบ 3 (เข้า fast recovery):** `ssthresh = cwnd/2`, `cwnd = ssthresh + 3`, retransmit segment ที่หาย

### 🧮 ตัวอย่างไล่ cwnd (TCP Reno)
ให้เริ่ม cwnd = 1 MSS, ssthresh = 8 MSS

| รอบ (RTT) | cwnd | สถานะ | เหตุผล |
|---|---|---|---|
| 1 | 1 | slow start | เริ่ม |
| 2 | 2 | slow start | ×2 |
| 3 | 4 | slow start | ×2 |
| 4 | 8 | slow start → CA | ถึง ssthresh |
| 5 | 9 | congestion avoidance | +1 |
| 6 | 10 | CA | +1 |
| 7 | 11 | CA | +1 |
| 8 | 12 | CA | **เกิด 3 dup ACK** ที่ cwnd = 12 |
| — | ssthresh = 6, cwnd = 6 + 3 = 9 | fast recovery | Reno |
| 9 | 6 | CA | ได้ new ACK → cwnd = ssthresh |
| 10 | 7 | CA | +1 |
| ... | ถ้า **timeout** ที่ cwnd = 7 | | ssthresh = 3.5, cwnd = **1** → slow start |

**Tahoe vs Reno:** Tahoe เมื่อเกิด loss ไม่ว่าแบบไหน **cwnd = 1** · Reno ถ้าเป็น 3 dup ACK **ลดครึ่ง** (fast recovery)

### 🔹 TCP CUBIC
- มีวิธีดีกว่า AIMD ในการ "probe" bandwidth ไหม?
- **Insight:** **Wmax** = rate ที่เกิด loss; สภาพ bottleneck **น่าจะไม่เปลี่ยนมาก** → หลังลดครึ่ง ให้ **เร่งขึ้นไปหา Wmax เร็วก่อน แล้วค่อย ๆ เข้าใกล้ Wmax ช้าลง**
- **K** = เวลาที่ window จะถึง Wmax (ปรับได้)
- เพิ่ม W ตาม **กำลังสาม (cube) ของระยะห่างระหว่างเวลาปัจจุบันกับ K**: ห่างจาก K มาก → เพิ่มมาก · ใกล้ K → เพิ่มน้อย (ระวัง)
- **default ใน Linux**, TCP ที่นิยมที่สุดสำหรับ web server (จนถึงราว 2024); throughput สูงกว่า classic TCP

### 🔹 TCP และ "bottleneck link"
- TCP (classic, CUBIC) เพิ่ม rate จนเกิด loss ที่ output ของ router บางตัว = **bottleneck link** (คิวแทบไม่เคยว่าง บางครั้งล้น)
- **insight 1:** เพิ่ม rate ไม่ทำให้ throughput end-end เพิ่ม เพราะ bottleneck ส่งเต็ม rate R อยู่แล้ว
- **insight 2:** เพิ่ม rate จะทำให้ **RTT เพิ่ม** (queueing delay ที่ bottleneck)
- **เป้าหมาย:** "**keep the end-end pipe just full, but not fuller**"
- หลาย flow แชร์ bottleneck: แต่ละ flow ควรได้ **R/N**

### 🔹 Keeping the pipe just full enough
```text
measured throughput = (# bytes sent in RTT interval) / RTT_measured
ideal in-flight data = bottleneck_bandwidth × min_RTT   ("bandwidth-delay product")
```
- in-flight น้อยกว่า BDP: เพิ่มแล้ว RTT ไม่เพิ่ม (คิวยังไม่เกิด)
- in-flight = BDP: arrival rate = link rate (จุดที่ดีที่สุด)
- in-flight มากกว่า BDP: เพิ่มแล้ว **RTT เพิ่ม** (คิวยาว) จนถึง BDP + buffer เต็ม แล้ว loss
- **loss-based TCP** ทำงานที่ขอบ buffer เต็ม · **BBR** ทำงานที่ BDP พอดี (pipe เต็มพอดี)

### 🔹 BBR (Bottleneck Bandwidth and RTT)
- ควบคุม **n_inflight**
- ช่วงเวลายาว: **ลด n_inflight เพื่อระบาย pipe** และวัด **min_RTT** ใหม่
- ช่วงเวลาสั้น:
  - **acceleration:** เพิ่ม rate จน throughput ไม่เพิ่ม (plateau)
  - **cruising:** ส่งที่ rate ที่เครือข่ายส่งได้จริง (ดูจาก ACK)
  - **deceleration:** ตั้งใจลด n_inflight ลดแรงกดคิว หา min_RTT ที่ต่ำกว่า

### 🔹 ECN (Explicit Congestion Notification)
TCP มักใช้ **network-assisted congestion control**:
- **2 bits ใน IP header (ToS field)** ถูก router ทำเครื่องหมายเมื่อแออัด (นโยบายการ mark กำหนดโดย operator)
- ข้อมูลความแออัดถูกส่งถึง **ปลายทาง**
- ปลายทางตั้ง **ECE bit ใน ACK** แจ้งผู้ส่ง
- เกี่ยวทั้ง **IP** (ECN bits) และ **TCP** (C, E bits) — ตัวอย่าง ECN=10 → ECN=11 เมื่อผ่าน router แออัด

### 🔹 TCP Fairness
- **เป้าหมาย:** K session แชร์ bottleneck R → แต่ละ session ได้เฉลี่ย **R/K**
- **TCP fair ไหม?** — **ใช่ ภายใต้สมมติฐานอุดมคติ:** RTT เท่ากัน, จำนวน session คงที่, อยู่ใน congestion avoidance เท่านั้น
  - additive increase เพิ่มทั้งคู่ด้วย slope 1 · multiplicative decrease ลดตามสัดส่วน → เข้าหาเส้น equal share
- **ต้อง fair ทุก app ไหม?**
  - **UDP:** multimedia มักไม่ใช้ TCP เพราะไม่อยากถูกลด rate → ใช้ UDP ส่ง rate คงที่ ทนการสูญหาย; **ไม่มี "Internet police"** คุมการใช้ congestion control
  - **Parallel TCP:** app เปิดหลาย connection ได้ (web browser ทำ) เช่น link R มี 9 connection อยู่ → app ใหม่ขอ 1 TCP ได้ **R/10**; ขอ 11 TCP ได้ **R/2**

---

## 11. Evolution of transport-layer functionality

- TCP, UDP เป็นหลักมา **40 ปี**
- มี TCP หลาย "flavor" สำหรับสถานการณ์เฉพาะ:

| Scenario | Challenge |
|---|---|
| Long, fat pipes (โอนข้อมูลใหญ่) | packet in-flight มาก; loss ทำให้ pipeline หยุด |
| Wireless networks | loss จาก link มี noise, mobility; **TCP เข้าใจผิดว่าเป็น congestion loss** |
| Long-delay links | RTT ยาวมาก |
| Data center networks | ไวต่อ latency |
| Background traffic flows | flow ความสำคัญต่ำ |

- ย้ายฟังก์ชัน transport ไปที่ **application layer บน UDP** → **HTTP/3: QUIC**

### 🔹 QUIC (Quick UDP Internet Connections)
- **application-layer protocol บน UDP** เพื่อเพิ่ม performance ของ HTTP; ใช้บน server/app ของ Google (Chrome, YouTube mobile)
- Stack: **HTTP/2 over TCP** = HTTP/2 + TLS + TCP + IP · **HTTP/3 over UDP** = HTTP/2 (slimmed) + **QUIC** + UDP + IP
- ใช้แนวคิดจากบทนี้: connection establishment, error control, congestion control (อัลกอริทึมคล้าย TCP)
- **connection establishment:** reliability, congestion control, **authentication, encryption** และ state ตั้งใน **1 (หรือ 0) RTT**
- **หลาย application-level "streams" multiplex บน QUIC connection เดียว:** แต่ละ stream มี reliable data transfer และ security แยก แต่ใช้ **congestion control ร่วมกัน**
- **ไม่มี HOL (head-of-line) blocking:** HTTP 1.1/HTTP/2 บน TCP ถ้า segment หนึ่ง error ทุก request ต้องรอ · QUIC error ใน stream หนึ่งไม่บล็อก stream อื่น

### 🔹 TCP throughput
- ค่าเฉลี่ย throughput ตาม window size และ RTT (ไม่นับ slow start, มีข้อมูลส่งตลอด)
- **W** = window size (bytes) ที่เกิด loss → window แกว่งระหว่าง W/2 ถึง W
- **avg window (in-flight) = ¾ W**

```text
avg TCP throughput = (3/4) × W / RTT   bytes/sec
```
🧮 ตัวอย่าง: W = 64,000 bytes, RTT = 100 ms → 0.75 × 64,000 / 0.1 = **480,000 bytes/s** (3.84 Mbps)

---

## จุดเน้นสำหรับสอบ
- 🧮 **Internet checksum** (บวก, wraparound, กลับบิต)
- 🧮 **EstimatedRTT, DevRTT, TimeoutInterval** (α = 0.125, β = 0.25)
- 🧮 **seq/ACK** (ACK = seq + bytes)
- 🧮 **ไล่ cwnd**: slow start ×2, CA +1, 3 dup ACK (Reno: ssthresh = cwnd/2, cwnd = ssthresh + 3 → ssthresh), timeout (cwnd = 1)
- 🧮 avg throughput = 0.75 W / RTT
- 🔥 UDP demux = dest port · TCP = 4-tuple
- 🔥 3-way handshake (SYN, SYNACK, ACK, seq x/y, ACKnum x+1/y+1) + ปัญหา 2-way
- 🔥 flow control (rwnd) vs congestion control (cwnd)
- 🔥 ACK generation 4 กรณี, fast retransmit (3 dup ACK), retransmission scenarios
- ⭐ AIMD, Tahoe vs Reno, CUBIC, BBR, ECN, fairness, QUIC (no HOL blocking)

## ✅ Coverage Check
- [x] Roadmap, overview, services, actions, TCP vs UDP
- [x] Mux/demux, how demux works, connectionless + example, connection-oriented + example, summary
- [x] UDP: why, uses, actions, header, checksum, Internet checksum, example, weak protection, summary
- [x] TCP overview, segment structure, seq/ACK, telnet example
- [x] RTT/timeout, EWMA, DevRTT, TimeoutInterval
- [x] Sender events, receiver ACK generation, 3 retransmission scenarios, fast retransmit
- [x] Flow control
- [x] Connection management, 2-way problems, 3-way handshake, closing
- [x] Congestion principles, 2 approaches
- [x] AIMD, cwnd, slow start, ssthresh, FSM, CUBIC, bottleneck, BDP, BBR, ECN, fairness, UDP/parallel
- [x] Evolution, QUIC, streams/HOL, TCP throughput
- [ ] rdt / GBN / SR — ไม่มีในไฟล์
