# Lab 10 · สรุปก่อนสอบ

1. **อ่านโค้ด หา decision**: `for` (มีสมาชิกเหลือไหม), `if`, `elif` → ทุก `if/elif/while/for` = 1 decision (`else` ไม่นับ)
2. **วาด flow chart**: S = statement, C = decision, loop วนกลับไปที่ C ของ loop
3. **#independent paths = #decisions + 1** (โจทย์นี้ 3 + 1 = **4**)
4. **List path**: ไม่เข้า loop 1 path + เข้า loop แล้วไปแต่ละ branch 1 path
5. **Linearly independent**: ทำตาราง path × segment, แต่ละ path ต้องเพิ่ม segment ใหม่ ≥ 1, รวมแล้วครบทุก segment
6. **Test case ต่อ path**: `[]` → 0 · `[5]` → 50 · `[30]` → 100 · `[70]` → 70
7. **ช่องโหว่**: ไม่ validate input (ติดลบ, เกินจริง, ชนิดผิด, None) · ทดสอบขอบ 11/12, 59/60

```text
S1 → C1 (for) ─หมด→ S6 return
     └─มี→ C2 (<12) ─T→ S2 ─┐
           └F→ C3 (<60) ─T→ S3 ─┤
                 └F→ S4 ────────┴→ S5 total+=price → C1
```
