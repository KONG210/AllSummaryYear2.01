# Error Detection · สรุปก่อนสอบ

| ข้อผิดพลาด | อาการ |
|---|---|
| **Black hole** | process เข้าอย่างเดียว ไม่มีออก |
| **Miracle** | process ออกอย่างเดียว ไม่มีเข้า |
| **Gray hole** | input ไม่พอสร้าง output |
| Entity → Entity | ลูกศรระหว่าง entity ตรงๆ |
| Entity ↔ Store | entity อ่าน/เขียน store ตรงๆ |
| Store → Store | ย้ายข้อมูลระหว่าง store ตรงๆ |
| Flow ไม่มีชื่อ | ลูกศรไม่มี label |
| ชื่อ process เป็นคำนาม | เช่น "Payment", "Appointment" |
| **Unbalanced** | ลูกมี flow/entity ข้ามขอบที่แม่ไม่มี หรือขาด |

**Checklist:** ทุก process เข้า+ออก+input พอ → ชื่อ process เป็นกริยา → ทุกลูกศรมีชื่อและมี process ที่ปลายหนึ่ง → เทียบกับแผนภาพแม่ → ตอบ **ตำแหน่ง + ชื่อข้อผิด + วิธีแก้**
