# L12 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- **QA** = ทุกกิจกรรมวัด/ปรับปรุงคุณภาพ **product + process** · **QC** = ยืนยันคุณภาพ **product** และหา fault
- **Verification** = "building the software **right**?" (แต่ละ phase ตรงกับ phase ก่อน) · **Validation** = "building the **right** software?" (ตอนจบ ตรงการใช้งานจริง)
- **Fault** (static defect ในโค้ด) → **Error** (สถานะภายในผิด) → **Failure** (พฤติกรรมภายนอกผิด) · ซอฟต์แวร์ไม่เสื่อม
- Testing = dynamic verification บน test case จำกัด, **พิสูจน์ไม่ได้ว่าถูก 100%**
- Quality = conforms to requirements + fit to use

## ⭐ Error detection 6 แบบ
Code review (inspection, peer review, **walkthrough** = ผู้เขียนอธิบาย, static analysis) · Testing (unit, integration, system, acceptance, **regression**) · **Static analysis** (ไม่รัน: unused var, unreachable code, null pointer) · **Dynamic analysis** (รัน: memory leak, coverage, profiling) · Debugging (identify → reproduce → locate → fix → test again) · Formal methods (model checking, theorem proving)

## 🧮 ECP / BVA
- **ECP**: แบ่ง class (valid/invalid) เลือกตัวแทน 1 ค่า/class · fail 1 ตัว = fail ทั้ง class
- **BVA**: ทดสอบที่ขอบ (−1, ขอบ, +1) · reduced = ขอบล่าง + ขอบบนของแต่ละ class · ใช้กับค่า ordinal เท่านั้น
- 1–1000 → ECP: ≤0 / 1–1000 / ≥1001 · BVA reduced: **0, 1, 1000, 1001**
- ตั๋วอุทยาน (≤11 ฟรี, 12–17 = 100, 18–60 = 200, >60 = 100) → reduced: **−1 (error), 0, 11 (free), 12, 17 (100), 18, 60 (200), 61 (100)**
- รับเฉพาะ 75, 253, 414 → valid: 3 ค่านี้ · invalid: ค่าอื่น

## 🧮 White-box
```text
#independent paths (cyclomatic) = #decision nodes + 1
#logical paths (k binary decisions เรียงกัน) = 2^k
linearly independent = ทุก path เพิ่ม edge ใหม่อย่างน้อย 1
```
- 3 decisions → 8 logical paths, 4 independent paths
- Loop: S1–C1–S3 และ S1–C1–S2–C1–S3
- Coverage มาก→น้อย: **logical paths > independent paths > branch > statement**
- White-box รับประกัน: ทุก independent path ≥ 1 ครั้ง, ทุก decision ทั้ง T/F, loop ที่ขอบ, internal data structure

| | Black-box | White-box |
|---|---|---|
| ชื่อ | behavioral / functional | glass-box / structural |
| ดู | spec, interface | control structure |
| หา | function ผิด/ขาด, interface, DB access, performance, init/termination | path, decision, loop |

## ⭐ อื่น ๆ
- Progression: unit → component → functional → system/regression
- How: intuition, spec-based (black), code-based (white), existing cases (regression)
- หยุด test: รันครบ + แก้ครบ · ไม่เจอเพิ่ม (problem find rate ลดลง) · defect seeding
- **Inspection**: cost-effective, ใช้กับ artifact ระหว่างทาง, เจอเร็ว, กระจายความรู้ · **Testing**: หาถูกแต่แก้แพง, ใช้กับโค้ดเท่านั้น, เจอช้า, จำเป็นวัดคุณภาพ
