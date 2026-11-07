# Quiz 2 SE Lab · สรุปก่อนสอบ

> 📌 Closed-book · Sec 2: **อ. 10 พ.ย. 10.45–12.15** · Sec 1: พ. 11 พ.ย. 11.15–12.45 · DFD + Git + Unit test

## DFD
| สัญลักษณ์ (Yourdon) | ความหมาย | ชื่อ |
|---|---|---|
| วงกลม | Process | กริยา + กรรม (1.0 Calculate Bill) |
| สี่เหลี่ยม | External entity | คำนาม |
| กล่องเปิด / เส้นคู่ | Data store | คำนาม (D1 Orders) |
| ลูกศร | Data flow | คำนาม (ต้องมีชื่อ) |

- Context = process 0 ตัวเดียว ไม่มี data store · Level 0 = 1.0, 2.0 … · Level 1 = 2.1, 2.2 …
- **Black hole** = เข้าอย่างเดียว · **Miracle** = ออกอย่างเดียว · **Gray hole** = input ไม่พอสร้าง output
- ❌ entity→entity · ❌ entity↔store · ❌ store→store · ❌ flow ไม่มีชื่อ · ❌ process ชื่อเป็นคำนาม
- **Balancing:** flow/entity ของแม่ = ของลูกรวมกัน (ห้ามมีของใหม่โผล่)
- ทุก flow ต้องมี process ที่ปลายใดปลายหนึ่ง

## Git
- `init` · `status` · `add` (→ staging) · `commit -m` (→ local) · `log` · `diff`
- `clone` · `push` · **`fetch` = ดึงมาดู ยังไม่ merge** · **`pull` = fetch + merge**
- `branch <n>` · `checkout <n>` · `push --set-upstream origin <n>` · `branch -a` · `fetch -p` · `branch -vv` · `branch -d`
- untracked → staged → committed → modified
- ไฟล์ใน branch หนึ่งไม่โผล่ใน branch อื่นจนกว่า merge
- PR: push branch → PR → review/comment → แก้ + push → merge → ลบ branch

## Git Cheat Sheet (lecture)
| คำสั่ง | ความหมาย |
|---|---|
| `reset [file]` | unstage (งานยังอยู่) |
| `reset --hard [commit]` | ล้าง staging + working tree กลับไป commit นั้น |
| `diff` / `diff --staged` | ยังไม่ stage / stage แล้วยังไม่ commit |
| `merge [branch]` | รวม branch นั้นเข้า branch ปัจจุบัน |
| `rebase [branch]` | ย้าย commit ของเราไปต่อท้าย branch นั้น |
| `log B..A` / `diff B...A` | อยู่ใน A แต่ไม่อยู่ใน B |
| `log --follow [file]` | ประวัติไฟล์ข้ามการเปลี่ยนชื่อ |
| `show [SHA]` | ดู object แบบอ่านง่าย |
| `rm` / `mv` | ลบ / ย้าย แล้ว stage ให้ |
| `stash` / `list` / `pop` / `drop` | เก็บ / ดู / เอากลับ / ทิ้ง (บนสุดของ stack) |
| `remote add [alias] [url]` | ตั้งชื่อเล่น remote |
| `push [alias] [branch]` | ส่ง branch ขึ้น remote |
| `config --global color.ui auto` | เปิดสี |
| `.gitignore` | กันไฟล์ตาม pattern เช่น `*.log`, `logs/` |

## Unit test
⚠️ รอไฟล์ · ทวน Lab 10 Testing
