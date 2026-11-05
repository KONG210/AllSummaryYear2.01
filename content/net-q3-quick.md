# Quiz 3 · สูตรก่อนเข้าห้องสอบ

> 📌 ศุกร์ 6 พ.ย. · คล้าย Quiz 1 · เน้นคำนวณ · ฝึกมือที่ [หน้าฝึกคำนวณ](#calc)

## 🧮 Fragmentation
```text
Max Data = floor((MTU − 20) / 8) × 8        ← ปัดลง ห้ามปัดขึ้น
จำนวนชิ้น = ceil(Data / Max Data)
Total Length = Data + 20
Offset = byte เริ่มต้น / 8                  ← เพิ่มทีละ Max Data / 8
MF = 1 ทุกชิ้น ยกเว้นชิ้นสุดท้าย = 0
```
- ให้ "datagram รวม header" → ลบ 20 ก่อน
- MTU 1000 → 976 · MTU 1500 → 1480 · MTU 576 → 552 · MTU 620 → 600

## 🧮 Subnet
```text
Block = 2^(32 − prefix)
Mask octet ท้าย = 256 − Block
IP ทั้งหมด = Block · แจกได้ = Block − 2
Network = floor(octet / Block) × Block
Broadcast = Network + Block − 1
Usable = Network + 1 .. Broadcast − 1
จำนวนวงใน /24 = 256 / Block
```

| /25 | /26 | /27 | /28 | /29 | /30 |
|---|---|---|---|---|---|
| 128 | 192 | 224 | 240 | 248 | 252 |
| 128 IP | 64 | 32 | 16 | 8 | 4 |

## 🧮 VLSM
1. เรียงมาก → น้อย
2. เครื่อง + 2 → 2^n ที่ ≥ ค่านั้น → prefix = 32 − n
3. วงถัดไปเริ่มที่ broadcast ก่อนหน้า + 1
4. **Gateway = Network + 1** (IP ใช้งานได้ตัวแรก)
5. IP อยู่ VLAN ไหน → ดูว่า octet ท้ายตกช่วง network–broadcast ของวงไหน
6. ช่วงว่าง = (broadcast วงสุดท้าย + 1) ถึง .255

**แนวข้อสอบ:** 10.20.30.0/24 · Nurse 100 → /25 .0–.127 · Doctor 30 → /27 .128–.159 · Pharmacy 6 → /29 .160–.167 · .140 อยู่ VLAN 20 · ว่าง .168–.255

## ⚠️ จุดพลาดบ่อย
- ลืม +2 ใน VLSM / ลืมลบ 2 ตอนนับ usable
- ปัด max data ขึ้น (เกิน MTU)
- ชิ้นสุดท้ายลืมเปลี่ยน MF เป็น 0
- ในโน้ต: fragment 3 จบที่ **2927**, fragment 4 เริ่ม **2928** (ไม่ใช่ 2957/2958)
