# L9 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- **Architectural design** = ส่วนหลัก + หน้าที่ + คุณสมบัติ + interface + ความสัมพันธ์ (เช่น แยก UI, performance ของ client/server, thread-safe ทั้ง library)
- **Detailed design** = ภายใน component: class, algorithm, data structure, schema (เช่น binary search, hash table)
- Design process: SRS → หลาย candidate architecture → ประเมิน → design document

## ⭐ Architectural Styles
| Style | หลักการ | Pro เด่น | Con เด่น |
|---|---|---|---|
| Pipe & Filter | ข้อมูลผ่าน filter ต่อกันด้วย pipe | reuse filter | ไม่เหมาะ interactive |
| Data-centered | central repository ทุก subsystem เข้าถึง | แชร์ข้อมูลมาก, backup กลาง | แก้ data model แพง |
| Layered | แต่ละชั้นพึ่งชั้นล่าง | portable, เปลี่ยนชั้นได้ | ชั้นเยอะช้า |
| Client-Server | client ขอ / server ให้ | distributed, เพิ่ม server ง่าย | server ล่มถ้า request มาก |
| Microservice | service เล็ก process แยก, HTTP API, deploy อิสระ | | |
| Call & Return | main เรียก component ย่อย (ใช้มากสุด) | modular, scale ง่าย | interface ไม่ชัด → tight coupling |

## ⭐ Structured design / Transform mapping
- DFD → structure chart (ใช้ PSPEC, STD) → call and return
- Transform flow (เข้า→แปลง→ออก) vs Transaction flow (transaction center แยก path)
- ขั้นตอน: แยก incoming/outgoing boundary (transaction center) → map จากขอบออกไป → เพิ่ม control module → refine → **first-level factoring** (main → input / processing / output controller) → second-level factoring
- ชั้นบน = decision making · ชั้นล่าง = worker

## ⭐ Principles
- **Basic:** Feasibility (สร้างได้จริง) · Adequacy (ตอบ stakeholder มาก) · Economy (ถูก เร็ว เสี่ยงน้อย) · Changeability (แก้ง่าย)
- **Constructive:** Modularity · Implementability · Aesthetic
- **Modularity:** Small modules · Information hiding · Least privilege · Cohesion · Coupling
- **Implementability:** Simplicity · Design with reuse · Design for reuse
- **Aesthetic:** Beauty (ไม่ซับซ้อนเกิน, ไม่ใช่ UI) · Symmetry (สม่ำเสมอ) · Balance (ไม่มี God class) · Proportion (ขนาดพอดี)
- สนับสนุนกัน: cohesion–coupling–information hiding, simplicity–beauty, simplicity–small modules
- **ขัดกัน:** simplicity vs design for reuse, small modules vs design for reuse, cohesion vs small modules

## ⭐ Cohesion & Coupling
| | Cohesion (ภายใน) | Coupling (ระหว่าง) |
|---|---|---|
| ต้องการ | **สูง** | **ต่ำ** |
| ดีสุด | Functional (1 หน้าที่) | Data (ส่งแค่ที่จำเป็น) · No coupling = อุดมคติ |
| แย่สุด | Coincidental (หลายหน้าที่ไม่เกี่ยวกัน) | Content (เข้าถึงข้อมูลภายใน) |
- ทำหลายหน้าที่ = low cohesion · เรียกตัวแปรภายในโมดูลอื่น = content (high) coupling · แก้หนึ่งต้องแก้อื่น = tight coupling · ผ่าน interface ชัด = loose coupling
- Logical module (business activity) vs Physical module (กระบวนการจริงที่ implement)
