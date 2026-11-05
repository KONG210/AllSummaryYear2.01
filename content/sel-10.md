# Lab 10 · Testing (White-box: Path Analysis)

## ภาพรวม
ฝึก **white-box testing** จาก ITDS261 L12–L13: จาก source code → (a) เขียน flow chart (b) คำนวณจำนวน independent path (c) ระบุ path ทั้งหมด (d) ระบุ linearly independent set of paths (e) วิเคราะห์ช่องโหว่ในฐานะ tester

## โจทย์
```python
def calculate_total_price(ages):
    total = 0
    for age in ages:
        if age < 12:
            price = 50
        elif age < 60:
            price = 100
        else:
            price = 70
        total += price
    return total
```

## (a) Flow chart — วิธีคิด
ตั้งชื่อ node:
- **S1:** `total = 0` / เตรียมรอบถัดไป
- **C1:** `for age in ages` — ยังมีสมาชิกเหลือไหม? (decision)
- **C2:** `age < 12`? (decision)
- **C3:** `age < 60`? (decision)
- **S2:** `price = 50` · **S3:** `price = 100` · **S4:** `price = 70`
- **S5:** `total += price` → วนกลับไป C1
- **S6:** `return total`

```text
S1 → C1 ──(หมด)──────────────────────────→ S6
      │(ยังมี)
      ↓
      C2 ──(T)→ S2 ─┐
      │(F)          │
      ↓             │
      C3 ──(T)→ S3 ─┤
      │(F)          │
      ↓             │
      S4 ───────────┤
                    ↓
                    S5 ──→ กลับไป C1
```

## (b) จำนวน Independent Paths
```text
#independent paths = #decision nodes + 1 = 3 (C1, C2, C3) + 1 = 4
```
**คำตอบของคุณ: 4 ✓**

## (c) Path ทั้งหมด (คำตอบของคุณ ✓)
- **Path 1:** S1 – C1 – S6 (list ว่าง ไม่เข้า loop)
- **Path 2:** S1 – C1 – C2 – S2 – S5 – S1 – C1 – S6 (อายุ < 12)
- **Path 3:** S1 – C1 – C2 – C3 – S3 – S5 – S1 – C1 – S6 (12 ≤ อายุ < 60)
- **Path 4:** S1 – C1 – C2 – C3 – S4 – S5 – S1 – C1 – S6 (อายุ ≥ 60)

## (d) Linearly Independent Set
ตาราง path × segment (คำตอบของคุณ)
| Path | Segments | segment ใหม่ที่เพิ่ม |
|---|---|---|
| Path 1 | 1, 2 | 1, 2 |
| Path 2 | 1, 3, 4, 5, 11, 2 | 3, 4, 5, 11 |
| Path 3 | 1, 3, 6, 7, 8, 11, 2 | 6, 7, 8 |
| Path 4 | 1, 3, 6, 9, 10, 11, 2 | 9, 10 |

ทุก path เพิ่ม segment ใหม่อย่างน้อย 1 ตัว และรวมกันครอบคลุม segment 1–11 → **เป็น linearly independent set ✓**

**Test case ที่ใช้รันแต่ละ path**
| Path | Input | Expected |
|---|---|---|
| 1 | `[]` | 0 |
| 2 | `[5]` | 50 |
| 3 | `[30]` | 100 |
| 4 | `[70]` | 70 |

## (e) ช่องโหว่ในฐานะ Tester
**คำตอบของคุณ:** ไม่ตรวจสอบ input — ค่าติดลบ (เช่น −5 ได้ราคา 50 เพราะ −5 < 12), ค่ามากเกินจริง, ชนิดข้อมูลผิด

**ประเด็นเพิ่มเติม**
- ไม่มีการจัดการ `None` หรือ input ที่ไม่ใช่ list
- อายุเป็นทศนิยม/ข้อความ (`"20"`) อาจทำงานผิดหรือ error
- ขอบเขต 12 และ 60 ควรทดสอบด้วย **BVA** (11, 12, 59, 60)

## เชื่อมกับทฤษฎี (L12)
- White-box ใช้ control structure สร้าง test case, รับประกันทุก independent path ≥ 1 ครั้ง, ทุก decision ทั้ง T/F, loop ที่ขอบ (0 รอบ, 1 รอบ)
- Black-box (ECP/BVA) หาช่องโหว่เรื่องค่า input ที่ white-box อาจไม่เห็น → ใช้ร่วมกัน
