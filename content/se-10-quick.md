# L10 · สรุปก่อนสอบ

## ⭐ Paradigms
- **Imperative** (how, ทีละขั้น: C, Python, Java, JS) · **Declarative** (what: SQL) · **OOP** (object = attributes + methods) · **Logic** (นิยามปัญหา: Prolog) · multi-paradigm
- **No-code** (ไม่เขียนโค้ด, drag-drop) · **Low-code** (โค้ดน้อย + เขียนเพิ่มได้, ต้องมีความรู้เทคนิคบ้าง) · **Vibe coding** (ภาษาธรรมชาติ + AI, เสี่ยงพึ่ง AI เกิน) — ทั้งหมดผูกกับ platform

## ⭐ Implementation
- = แปลง detailed design เป็นโปรแกรม: writing code, unit testing, debugging, configuration management
- Good: **Readability, Maintainability, Performance, Traceability, Correctness, Completeness**
- Style guide 3 แบบ: **avoid dangers** (exception, static, lambda, threading) · **enforce best practices** (comment ที่ไหน) · **ensure consistency**
- Consistency → scaling (tools), human scaling (ย้ายโปรเจกต์), resilience to time
- Practices: function เล็ก · DRY · เขียนให้คนอ่าน · จัดการ error ชัด · commit เล็กบ่อย · test ไปพร้อมเขียน

## ⭐ 9 Comment Rules
1 ไม่ซ้ำโค้ด · 2 ไม่ใช้แทนโค้ดที่ไม่ชัด · 3 เขียน comment ไม่ได้ = โค้ดมีปัญหา · 4 ลดความสับสน · 5 อธิบายโค้ดที่ไม่ idiomatic · 6 ลิงก์ที่มาของโค้ดที่ copy · 7 ลิงก์ reference ภายนอก · 8 comment ตอนแก้ bug · 9 mark งานที่ยังไม่เสร็จ (TODO)

## ⭐ Debugging / Refactoring
- Debugging 4 phases: **Stabilization (reproduce) → Localization → Correction → Verification**
- Heuristics: routine ที่มี error มักมีอีก, โค้ดใหม่ error มาก · Tools: lint, interactive debugger, comparator, profiler, coverage
- Refactoring (Fowler) = เปลี่ยนโครงสร้างภายใน **ไม่เปลี่ยนพฤติกรรมภายนอก** · ไม่ใช่เขียนใหม่ · เล็กหรือใหญ่ได้ · สนับสนุน emergent design
- Smells: duplicated code, long method, large class, switch, feature envy, intimacy → extract method, substitute algorithm, move method, extract class

## 🔥 Fagan Inspection (IBM, 1976)
| ขั้น | ทำอะไร |
|---|---|
| Planning | entry criteria, หาคน, สถานที่/เวลา |
| Overview | ให้ความรู้กลุ่ม, มอบ **roles** |
| Preparation | ศึกษาเอกสารตามบทบาท |
| Inspection | **หา defect** (ไม่หาวิธีแก้) |
| Rework | **author** แก้ทั้งหมด |
| Follow-up | **moderator**/ทีม ยืนยันว่าแก้แล้ว ไม่มี defect ใหม่ |
- ตรวจได้ทั้ง code, design, requirements, docs, test plans, test cases

## ⭐ Modern Code Review
- ใช้ tool (PR) · Benefits (Google): correctness, เข้าใจได้, consistency, team ownership, knowledge sharing, historical record
- Checklist: implementation, dependencies, security/privacy, logic/bugs, error handling/logging, testing, readability, usability, performance, experts
- PR ทำงานได้แต่ชื่อ `x`, ไม่มี test → ยังไม่ approve; ขั้นต่ำ: ชื่อสื่อความหมาย + unit test; comment สุภาพมีเหตุผล
