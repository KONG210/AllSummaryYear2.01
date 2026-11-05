# Ch 6 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- Transport = logical communication ระหว่าง **process** (ทำที่ end system) · Network = ระหว่าง **host**
- **UDP**: no-frills, connectionless, unreliable, header 8 bytes, ไม่มี congestion control · ใช้กับ streaming, DNS, SNMP, HTTP/3
- **TCP**: point-to-point, reliable in-order byte stream, full duplex, cumulative ACK, pipelining, connection-oriented, flow controlled
- ไม่มีทั้งคู่: delay guarantee, bandwidth guarantee
- Demux: **UDP = dest port อย่างเดียว** · **TCP = 4-tuple** (src IP, src port, dst IP, dst port)
- Flow control = ผู้รับคนเดียวรับไม่ทัน (**rwnd**) · Congestion control = เครือข่ายรับไม่ไหว (**cwnd**)

## 🧮 สูตรและวิธีคำนวณ
```text
Checksum: บวก 16 บิต → carry ซ้ายสุดบวกกลับ (wraparound) → กลับบิตทั้งหมด
EstimatedRTT = 0.875·EstRTT + 0.125·SampleRTT
DevRTT       = 0.75·DevRTT + 0.25·|SampleRTT − EstRTT|
Timeout      = EstimatedRTT + 4·DevRTT
ACK = Seq ที่ได้รับ + จำนวน bytes
TCP rate ≈ cwnd / RTT          avg throughput = 0.75·W / RTT
LastByteSent − LastByteAcked ≤ cwnd     (และ ≤ rwnd)
BDP = bottleneck bandwidth × min_RTT
```
- ตัวอย่าง checksum: 1110011001100110 + 1101010101010101 → sum 1011101110111100 → **checksum 0100010001000011**
- ตัวอย่าง RTT: Est 100, Dev 10, Sample 120 → Est 102.5, Dev 11.875, Timeout 150
- Telnet: (Seq 42, ACK 79, 'C') → (Seq 79, ACK 43, 'C') → (Seq 43, ACK 80)

## 🧮 cwnd (Reno)
| เหตุการณ์ | ทำอะไร |
|---|---|
| เริ่ม | cwnd = 1 MSS, slow start |
| slow start | cwnd ×2 ต่อ RTT (+1 MSS ต่อ ACK) |
| cwnd ≥ ssthresh | congestion avoidance: +1 MSS ต่อ RTT |
| **3 dup ACK** | ssthresh = cwnd/2, cwnd = ssthresh + 3 → fast recovery → new ACK: cwnd = ssthresh |
| **timeout** | ssthresh = cwnd/2, **cwnd = 1**, slow start |
- **Tahoe**: loss ทุกแบบ → cwnd = 1 · **Reno**: 3 dup ACK → ลดครึ่ง

## ⭐ ต้องจำ
- TCP segment: seq (นับ **byte** ไม่ใช่ segment), ACK (byte ถัดไปที่คาด), flags C E U A P R S F, rwnd, checksum
- **3-way handshake**: SYN(Seq=x) → SYNACK(Seq=y, ACK=x+1) → ACK(ACK=y+1, ใส่ data ได้) · states: LISTEN, SYNSENT, SYN RCVD, ESTAB
- 2-way ไม่พอ → **half-open connection**, **dup data accepted**
- ปิด: แต่ละฝั่งส่ง **FIN** แล้วอีกฝ่าย ACK
- ACK generation: in-order → **delayed ACK 500 ms** · in-order + มีค้าง → cumulative ACK ทันที · out-of-order (gap) → **dup ACK ทันที** · เติม gap → ACK ทันที
- **Fast retransmit**: 3 dup ACK → ส่งใหม่ไม่รอ timeout
- Timeout สั้นไป = retransmit เกิน · ยาวไป = ตอบสนองช้า · SampleRTT ไม่นับ retransmission
- rwnd = พื้นที่ว่างใน RcvBuffer (default 4096 bytes)
- Congestion: delay นาน + loss · end-end (TCP, อนุมานจาก loss/delay) vs network-assisted (ECN, ATM, DECbit)
- **AIMD**: +1 MSS/RTT, ลดครึ่งเมื่อ loss → sawtooth · optimize ทั้งเครือข่าย + เสถียร [Kelly]
- **CUBIC**: ขึ้นเร็วไปหา Wmax แล้วช้าลงใกล้ K (ฟังก์ชันกำลังสาม), default Linux
- **BBR**: คุม in-flight ให้ = BDP (acceleration, cruising, deceleration) · loss-based ทำงานที่ buffer เต็ม
- **ECN**: router mark 2 bits ใน IP ToS → ผู้รับตั้ง **ECE** ใน ACK → ผู้ส่งลด
- **Fairness**: K sessions → R/K (fair ถ้า RTT เท่า, session คงที่, อยู่ CA) · UDP ไม่ fair, parallel TCP ได้ส่วนมากขึ้น (1 TCP → R/10, 11 TCP → R/2)
- **QUIC**: app-layer บน UDP, HTTP/3, 1 หรือ 0 RTT, หลาย stream, **ไม่มี HOL blocking**, congestion control ร่วม
- Internet checksum **weak**: bit พลิกหักล้างกันตรวจไม่เจอ

## ⚠️ มักสับสน
- seq/ACK นับ byte · ACK = byte ถัดไปที่ "อยากได้"
- UDP dest port เดียวกัน (แม้ต่าง source) → socket เดียวกัน · TCP ต่าง 4-tuple → คนละ socket
- ไม่มีสไลด์ rdt / GBN / SR ในไฟล์ — ถามอาจารย์
