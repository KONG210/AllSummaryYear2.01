# Quiz 2 SE Lab · ภาพรวม + แบบฝึกแยกเรื่อง

## สอบอะไร

> 📌 **Quiz 2 (20%)** · **Closed-book** ห้ามนำ lecture note หรือ cheat sheet เข้า · เตรียมเครื่องเขียน · **Section 1: 11 พ.ย. 2569 เวลา 11.15–12.45** · **Section 2: 10 พ.ย. 2569 เวลา 10.45–12.15**

| เรื่อง | บทสรุป + แบบฝึก | มีอะไรบ้าง |
|---|---|---|
| **1. DFD Understanding** | [Quiz 2 · DFD Understanding](#sel-q2a) | สัญลักษณ์, ระดับ, วิธีอ่าน · แบบฝึก 3 ชุด (อ่าน level 0, อ่าน level 1, แยกองค์ประกอบจากโจทย์) |
| **2. Error detection in DFD** | [Quiz 2 · Error Detection](#sel-q2b) | กฎจับผิด + checklist · แบบฝึก 4 ชุด (หาจุดผิด 2 แผนภาพ, balancing 2 ชุด) |
| **3. Git functions** | [Quiz 2 · Git Functions](#sel-q2c) | Lab 9 + Cheat Sheet · แบบฝึก 3 ชุด (สถานการณ์, คำสั่ง cheat sheet, อ่านผล git) |
| **4. Unit test** | ⚠️ รอไฟล์ | ระหว่างนี้ทวน [Lab 10 Testing](#sel-10) และ [L13 Testing](#se-13) |

> ⭐ ทุกบทมีแท็บ **แบบฝึกหัด** (ปรนัย) ของเรื่องนั้นแยกกัน · แท็บแบบฝึกหัดของหน้านี้เป็น **ชุดรวม** ไว้ทดสอบก่อนสอบ

## แผน 3 วันก่อนสอบ (Section 2)

| วัน | ทำอะไร |
|---|---|
| **ส. 8 พ.ย.** | อ่าน DFD Understanding + Error Detection แล้วทำแบบฝึกในบท (ปิดเฉลยก่อน) |
| **อา. 9 พ.ย.** | Git Functions + แบบฝึกหัดปรนัยครบ 3 เรื่อง · ข้อไหนผิดทวนในสมุดข้อที่ผิด |
| **จ. 10 พ.ย. เช้า** | สรุปก่อนสอบของหน้านี้ + ชุดรวม แล้วไปสอบ 10.45 |

## สรุปเร็วทุกเรื่อง

**DFD:** process (วงกลม, กริยา) · entity (สี่เหลี่ยม) · data store (กล่องเปิด) · flow (ลูกศร, มีชื่อ) · context = process 0 ไม่มี store · level 0 = 1.0, 2.0 · level 1 = 2.1, 2.2

**จับผิด:** black hole · miracle · gray hole · entity→entity · entity↔store · store→store · flow ไม่มีชื่อ · process ชื่อคำนาม · unbalanced

**Git:** init/status/add/commit/log/diff · clone/push/fetch/pull · branch/checkout/merge · reset/stash/rm/mv/.gitignore/rebase

## Unit test

> ⚠️ **ยังรอไฟล์ Unit test ของแล็บ** ส่งมาแล้วจะทำบทแยกพร้อมแบบฝึก
