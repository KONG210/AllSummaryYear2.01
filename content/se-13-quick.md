# L13 · สรุปก่อนสอบ

## ⭐ Test phases
| | Unit | Integration | System |
|---|---|---|---|
| ทดสอบ | unit เล็กสุด | กลุ่ม module ที่รวมกัน | ระบบสมบูรณ์ vs requirement |
| ต้นทุน/ความถี่ | ต่ำสุด / บ่อยสุด | ปานกลาง | สูงสุด / ไม่บ่อย |
| Specification | module interface | interface specs, module breakdown | requirements spec |
| Visible structure | coding details | modular structure (architecture) | none |
| Scaffolding | some | **often extensive** | some |
| หา fault ใน | modules | interactions, compatibility | system functionality |

- Good unit test: **focused (1 function, ไม่พึ่งส่วนอื่น), fast, automatable (CI/CD), repeatable**
- **JUnit**: กลางยุค 90, มาตรฐาน TDD ของ Java, JAR ตอน compile, **@Test**, assertion methods (assertEquals, assertTrue, assertNull ...), test runners, Eclipse/Ant/Maven
- Integration: หาปัญหา interface, data flow, dependency (เช่น function + DB)
- **Unit = รากฐาน** (controllability + visibility สูงสุด) · integration **ชดเชย unit ที่ไม่พอไม่ได้** · เจอ module fault ตอน integration = unit test ไม่พอ · fault ที่ interface ระหว่าง module ที่ถูก = ปัญหา module breakdown/interface spec
- System test: **functional, non-functional (performance, usability, reliability), security**
- เอกสาร: test case template, test report, **traceability matrix** (requirement ↔ test)

## ⭐ Regression
- ตรวจว่าการแก้ใหม่ไม่ทำให้ bug ใหม่/ของเดิมพัง · ใช้หลัง bug fix, feature ใหม่, performance improvement, routine maintenance

## ⭐ Usability
- = ง่ายและมีประสิทธิภาพในการใช้บรรลุเป้าหมาย · 5 components (รูปในสไลด์; Nielsen: learnability, efficiency, memorability, errors, satisfaction)
- Usability testing = ให้ผู้ใช้อนาคตลองใช้ · Methods: **task-based, A/B testing, surveys**
- Performance measurement: เวลาทำ task, จำนวน task ต่อเวลา, success:error ratio, เวลากู้ error, จำนวน error, ฟีเจอร์ที่ไม่ถูกใช้, การใช้ help

## ⭐ Best practices
modular/isolated · CI/CD · test สม่ำเสมอ · mock dependencies · **meaningful coverage > 100%**

## 🧮 โจทย์ส่วนลด (เฉลยสไลด์)
- ECP: ≤0 (−5 invalid) · 1–500 (450 ไม่ลด) · 501–999 (658, 5%) · 1,000–4,999 (2,599, 10%) · 5,000–100,000 (45,100, 20%) · >100,000 (111,111 invalid)
- BVA reduced: −1, 1, 500, 501, 999, 1000, 4999, 5000, 100000, 100001
- ⚠️ สไลด์จัด 500 = ไม่ลด แต่โจทย์บอก 500–999 ลด 5% → ตอนสอบอ่านขอบให้ดี
