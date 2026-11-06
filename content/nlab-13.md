# Lab 13 · TCP and UDP Communications

## ภาพรวมของแล็บ

> 📌 อ้างอิง: **Packet Tracer 14.8.1 – TCP and UDP Communications** (CCNA 1 ITNv7) · แล็บนี้**ไม่มีคะแนน** (not scored)

**Objectives**
- **Part 1:** สร้าง traffic ใน **Simulation mode**
- **Part 2:** ดูการทำงานของ **TCP และ UDP**

**Background:** Simulation mode แสดง **PDU (Protocol Data Unit)** ที่วิ่งในเครือข่ายทีละก้อน แล็บนี้ดูเรื่อง TCP, UDP, **multiplexing** และการใช้ **port number** ระบุ application ทั้งสองฝั่ง

**อุปกรณ์:** MultiServer, HTTP Client, FTP Client, DNS Client, E-Mail Client
- หน้าแล็บ**ไม่มี addressing table** มีแค่ภาพ topology · IP ที่ถูกอ้างถึงคือ
  - **192.168.1.254** = MultiServer (เป้าหมายของ HTTP, FTP, DNS · ชื่อ `multiserver.pt.ptu` แปลงเป็น IP นี้)
  - **192.168.1.255** = broadcast address ของ LAN ฝั่ง client (ใช้ ping เพื่อเติม ARP table)

## 1. Part 1: Generate Traffic in Simulation Mode

1. **เติม ARP table:** ที่ MultiServer ping broadcast address ของ LAN 1 ครั้ง แล้วปิดหน้าต่าง
2. **HTTP:** สลับเป็น Simulation mode · ที่ HTTP Client เปิด browser ไปที่ **192.168.1.254** แล้วย่อหน้าต่าง
3. **FTP:** ที่ FTP Client เปิด command prompt พิมพ์ `ftp 192.168.1.254` แล้วย่อหน้าต่าง
4. **DNS:** ที่ DNS Client พิมพ์ `nslookup multiserver.pt.ptu` แล้วย่อหน้าต่าง
5. **Email:** ที่ E-Mail Client เขียนแล้วส่งอีเมล แล้วย่อหน้าต่าง
6. **ตรวจ:** เช็กว่ามี PDU ของทุก client ขึ้นใน simulation panel
7. **Multiplexing:** กด **Capture/Forward** ดู traffic จากหลายเครื่องวิ่งผ่านเครือข่ายพร้อมกัน

## 2. Part 2: Examine TCP and UDP

1. **HTTP (TCP):** reset simulation · filter เฉพาะ **HTTP + TCP** · สร้าง traffic แล้วเปิดดู TCP header ของแต่ละ PDU
2. **FTP (TCP):** เปิด FTP session · filter **FTP + TCP** · ดู handshake และข้อความต้อนรับจาก server
3. **DNS (UDP):** filter **DNS + UDP** · ดู port ของ UDP และส่วน answer
4. **Email (TCP):** filter **POP3 + SMTP + TCP** · ดู handshake และเทียบ port ที่ใช้

## 3. สิ่งที่สังเกตได้ (ต้องจำ)

| Application | Transport | Port ฝั่ง server (dest) | Port ฝั่ง client (source) | สิ่งที่เห็นในแล็บ |
|---|---|---|---|---|
| **HTTP** | **TCP** | **80** | ประมาณ 1029 (เปลี่ยนได้) | server ตอบ **SYN+ACK** · ข้อมูลจาก client เป็น **PSH+ACK** |
| **FTP** | **TCP** | **21** | ประมาณ 1025 | handshake **SYN → SYN+ACK → ACK** · server ส่งข้อความต้อนรับ |
| **DNS** | **UDP** | **53** | ประมาณ 1025 | **ไม่มี sequence / acknowledgement number** · answer แปลงชื่อเป็น 192.168.1.254 |
| **Email (SMTP)** | **TCP** | **25** | ประมาณ 1025 | handshake แบบเดียวกับ FTP · ข้อมูลเป็น PSH+ACK |
| **Email (POP3)** | **TCP** | **110** | – | ใช้**ดึง**อีเมลลงเครื่อง |

**TCP flag bits** (6 ตำแหน่ง อ่าน**จากขวาไปซ้าย**): **FIN, SYN, RST, PSH, ACK, URG** · ค่า "1" = flag นั้นถูกตั้ง · ตั้งหลาย flag พร้อมกันได้ (เช่น SYN+ACK)

## 4. คำถามในแล็บและคำตอบ

| คำถาม | คำตอบ |
|---|---|
| ทำไม HTTP PDU ก้อนแรกถึงช้า? | TCP ต้อง**สร้าง connection ก่อน** HTTP data ถึงจะวิ่งได้ |
| TCP เชื่อถือได้ (reliable) ไหม? | **ได้** |
| UDP/DNS เชื่อถือได้ไหม? | **ไม่** · UDP ไม่สร้าง connection จึง**ไม่มี sequence และ acknowledgement number** |
| Layer 4 protocol ของ DNS คือ? | **UDP** |
| ส่วนสุดท้ายของ DNS PDU คืออะไร ได้ IP อะไร? | **DNS ANSWER** section ได้ **192.168.1.254** |
| ข้อความจาก FTP server คือ? | "Welcome to PT Ftp server" |
| Email ใช้ transport อะไร? | **TCP** |
| TCP port 25 และ 110 คือ protocol อะไร? | **SMTP (25)** และ **POP3 (110)** |
| สีต่างๆ ใน simulation หมายถึงอะไร? | **protocol ต่างกัน** |
| หลายบทสนทนาใช้สายเดียวกันเรียกว่า? | **Conversation multiplexing** |

## 5. แนวคิดหลัก

| | **TCP** | **UDP** |
|---|---|---|
| แบบ | **Connection-oriented** | **Connectionless** |
| ความน่าเชื่อถือ | **Reliable** | **Unreliable** (ไม่รับประกัน) |
| ก่อนส่งข้อมูล | **Three-way handshake: SYN → SYN+ACK → ACK** | ไม่มี handshake ส่งได้เลย |
| Sequence / ACK number | **มี** | **ไม่มี** |
| Overhead | มากกว่า | **น้อยกว่า** |
| ตัวอย่างในแล็บ | HTTP 80, FTP 21, SMTP 25, POP3 110 | DNS 53 |

- **Source / destination port:** server ใช้ **well-known port** (80, 21, 53, 25, 110) · client ใช้ **port ชั่วคราว** (เช่น ~1025, 1029) · ตอน server ตอบกลับ **source กับ destination port จะสลับกัน**
- **Multiplexing:** หลายบทสนทนาใช้ลิงก์เดียวกันได้ · สายแต่ละเส้นส่ง PDU ได้ทีละก้อนต่อทิศทาง · **port number** ทำให้ข้อมูลไปถึง application ที่ถูกตัว

> ⭐ เชื่อมกับ Network Ch6 (Transport Layer): TCP = reliable, flow/congestion control, handshake · UDP = เร็ว เบา ไม่มี connection · demultiplexing ใช้ port

## จุดที่มักสับสน

| เรื่อง | A | B | จำง่ายๆ |
|---|---|---|---|
| SMTP vs POP3 | **25** ส่งอีเมล | **110** ดึงอีเมล | S = Send |
| HTTP vs FTP | **80** เว็บ | **21** ไฟล์ | ทั้งคู่ TCP |
| DNS | **UDP 53** | – | DNS = ตัวเดียวในแล็บที่เป็น UDP |
| SYN+ACK vs PSH+ACK | ตอบรับตอนเปิด connection | ส่งข้อมูลจริง | PSH = push data |
| source vs dest port | client: port สุ่มสูงๆ | server: well-known port | ตอบกลับ = สลับกัน |

## จุดเน้นสำหรับสอบ
- 🔥 port: **HTTP 80 · FTP 21 · DNS 53 (UDP) · SMTP 25 · POP3 110**
- 🔥 TCP = connection-oriented, reliable, **SYN → SYN+ACK → ACK**, มี seq/ack · UDP = connectionless, unreliable, ไม่มี seq/ack
- 🔥 HTTP ก้อนแรกช้าเพราะต้อง handshake ก่อน
- ⭐ TCP flags 6 ตัว (ขวาไปซ้าย): FIN, SYN, RST, PSH, ACK, URG
- ⭐ ตอบกลับ source/dest port สลับกัน · client ใช้ port ชั่วคราว
- ⭐ **Conversation multiplexing** · สีใน simulation = protocol
- 📌 DNS ANSWER คืน 192.168.1.254 · FTP: "Welcome to PT Ftp server"
