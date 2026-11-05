# L11 · สรุปก่อนสอบ

- **CM** = เริ่ม ประเมิน ควบคุมการเปลี่ยนแปลงของซอฟต์แวร์ ระหว่างและหลังการพัฒนา · มาตรฐาน **IEEE 828** (plans), **IEEE 1042** (guide)
- **Configuration item** = อะไรก็ได้ที่เปลี่ยนได้และต้องติดตาม: โค้ด, test drivers, เอกสาร req/analysis/design, คู่มือ, system config (เวอร์ชัน compiler), hardware config (CPU, bus)
- **Version identification:** numbering (V1, V1.1 — ไม่มีความหมาย) vs **attribute-based** (date, creator, language, customer, status)
- **VCS** (ส่วนหนึ่งของ CM): 1) restore เวอร์ชันเก่า 2) ประสานการแก้ไข 3) ระบุเวอร์ชันและ component
- **Git** = free, open-source, **distributed** VCS, เร็ว, footprint เล็ก, local branching ถูก, staging area, หลาย workflow (ดีกว่า SVN, CVS)
- Distributed: เกือบทุกอย่าง local · เร็ว · ทุก clone เป็น backup · ทำ offline ได้ · มี full history

| คำสั่ง | จาก → ไป |
|---|---|
| `git add` | working dir → **staging area (index)** |
| `git commit -m` | staging → **local repo** (เวอร์ชันใหม่ + message) |
| `git push` | local → **remote** (ถูก reject ถ้า remote มีของใหม่ที่เรายังไม่มี) |
| `git pull` | remote → local (**fetch + merge**, conflict จัดการได้) |

- **master**: deployable, เสถียร, ห้าม push สิ่งที่ไม่ test/ทำ build พัง
- แตก branch เมื่อ: feature ใหม่ · release ใหม่ · ทดลอง · แก้ bug · branch ชั่วคราว, merge ส่วนใหญ่อัตโนมัติ, conflict เจ็บปวดถ้าไม่มี tool → ใช้ workflow
- **Branching model:** main = **master** (production-ready) + **develop** (สำหรับ release ถัดไป) · supporting = **feature** (develop→develop), **release** (เตรียม release, develop→master+develop), **hotfix** (bug วิกฤตใน production แตกจาก tag บน master → master+develop)
