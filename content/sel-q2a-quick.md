# DFD Understanding · สรุปก่อนสอบ

| สัญลักษณ์ (Yourdon) | ความหมาย | ชื่อ |
|---|---|---|
| วงกลม | Process | กริยา + กรรม (1.0 Calculate Bill) |
| สี่เหลี่ยม | External entity | คำนาม |
| กล่องเปิด / เส้นคู่ | Data store | คำนาม (D1 Orders) |
| ลูกศร | Data flow | คำนาม (ต้องมีชื่อ) |

- Context = process 0 ตัวเดียว ไม่มี data store · Level 0 = 1.0, 2.0 … · Level 1 ของ 2.0 = 2.1, 2.2 …
- ลูกศร **เข้า** store = เขียน · **ออก** จาก store = อ่าน
- วิธีอ่าน: ดูระดับ → นับ 4 ชนิด → ไล่ input/output ทีละ process → ไล่ใครอ่าน/เขียนแต่ละ store
- Balancing: flow ที่ข้ามขอบของลูก = ของแม่ · flow ภายในระหว่าง process ลูกไม่ต้องมีในแม่
- DFD ไม่แสดงลำดับเวลาหรือเงื่อนไข (ไม่ใช่ flowchart)
