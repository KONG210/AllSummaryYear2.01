# Lab 9 · Git and Code Review

## ภาพรวม
ฝึกใช้ **Git** ตั้งแต่ตั้งค่า → local repo → remote บน GitHub (งานกลุ่ม) → branching → **merge ผ่าน Pull Request พร้อม code review** — คือการลงมือจริงของ ITDS261 **L10 (code review)** และ **L11 (VCS)**

> 🔗 Flow ทั้งหมด: `init/clone` → แก้ไฟล์ → `add` (staging) → `commit` (local) → `push` (remote) → เพื่อน `fetch/pull` → แตก `branch` → push branch → เปิด **Pull Request** → เพื่อน **review + comment** → แก้แล้ว push ใหม่ → **Merge** → ลบ branch

---

## ส่วนที่ 0: คำสั่ง terminal พื้นฐาน
| งาน | Windows (Command Prompt) | Linux/macOS (Terminal) |
|---|---|---|
| แสดงไฟล์ | `dir` | `ls` |
| แสดง directory ปัจจุบัน | `cd` | `pwd` |
| เปลี่ยน directory | `cd <location>` | `cd <location>` |
| สร้าง directory | `mkdir <name>` | `mkdir <name>` |
| แก้ไฟล์ | `notepad se.txt` | `vim se.txt` |

**Vim เบื้องต้น:** กด `i` เข้าโหมดแก้ไข → พิมพ์ → `ESC` → `:w` (บันทึก) → `:q` (ออก) (`:wq` = บันทึกแล้วออก)

## ส่วนที่ 1: ตั้งค่า Git
```bash
git --version
git help
git config --global user.name "Your Name"
git config --global user.email "you@example.com"   # ใช้ตอนสร้าง commit
```

## ส่วนที่ 2: Local repository
```bash
cd <path>/test
git init                          # เริ่มใช้ Git ในโฟลเดอร์
git status                        # ดูสถานะ (se.txt = untracked)
git add se.txt                    # เข้า staging
git status                        # (changes to be committed)
git commit -m "Added se.txt"      # บันทึกเวอร์ชัน
git log                           # ดูประวัติ commit
git log --name-status             # ดูประวัติ + ไฟล์ที่เปลี่ยน
# แก้ se.txt เพิ่มบรรทัด "Introduction to Software Engineering"
git status                        # (modified)
git diff se.txt                   # ดูว่าแก้อะไร
git add se.txt
git commit -m "Added a modification to se.txt"
git diff <commitID_เก่า> <commitID_ใหม่> se.txt   # เทียบระหว่าง 2 commit
```

| สถานะไฟล์ | หมายถึง |
|---|---|
| untracked | Git ยังไม่ติดตาม |
| modified | แก้แล้วยังไม่ add |
| staged | add แล้ว รอ commit |
| committed | บันทึกใน local repo แล้ว |

## ส่วนที่ 3: Remote repository บน GitHub (งานกลุ่ม)
```bash
# ถ้าใช้ MU Wi-Fi ตั้ง proxy ก่อน
git config --global http.proxy http://<user>:<password>@muproxy-basic.mahidol:8080
git clone <repo URL>              # คัดลอก repo กลุ่มมาเครื่อง
cd <repo name>
git status
mkdir lab-exercise-9 && cd lab-exercise-9
# สร้าง <Yourname>.java (Hello <Yourname>)
git add .
git commit -m "Added <Yourname> class file"
git push                          # ส่งขึ้น GitHub
git status
git fetch                         # ดึงข้อมูล remote มาดู (ยังไม่ merge)
git pull                          # ดึง + merge งานเพื่อน
git log
```

| | `git fetch` | `git pull` |
|---|---|---|
| ทำอะไร | ดึงข้อมูลจาก remote มาไว้ **ยังไม่ merge** | **fetch + merge** เข้า branch ปัจจุบัน |

## ส่วนที่ 4: Branching
```bash
git branch                              # ดู branch ปัจจุบัน
git branch dev_<Yourname>               # สร้าง branch
git checkout dev_<Yourname>             # ย้ายไป branch
# สร้าง <Yourname>Feature.java
git add <Yourname>Feature.java
git commit -m "Added a new feature"
ls                                      # เห็นไฟล์ feature
git checkout master
ls                                      # ไม่เห็นไฟล์ feature (อยู่คนละ branch)
git push --set-upstream origin dev_<Yourname>   # push branch ใหม่ครั้งแรก
git fetch
git branch -a                           # ดู branch ทั้งหมดรวม remote
git checkout <branch เพื่อน>
```
> ⭐ ไฟล์ที่ commit ใน branch หนึ่ง **ไม่ปรากฏใน branch อื่น** จนกว่าจะ merge
> ⭐ `--set-upstream` (`-u`) ผูก branch local กับ remote ครั้งแรก ครั้งต่อไป `git push` ได้เลย

## ส่วนที่ 5: Merging และ Code Review ด้วย Pull Request
1. ที่ GitHub → **Branches** → เลือก branch ตัวเอง → **New pull request**
2. ใส่รายละเอียด PR ให้ reviewer เข้าใจ → **assign เพื่อนร่วมทีมทุกคน** → **Create pull request**
3. reviewer คลิก commit → กด **"+"** หน้าบรรทัดเพื่อใส่ **review comment** → **Submit review**
4. **ทุกคนต้อง comment อย่างน้อย 1 comment ในทุก PR ของเพื่อน**
5. เจ้าของ branch แก้ไฟล์ (เพิ่ม `int x = 0; x = x+1;`) → `git add` → `git commit -m "Fixed the feature"` → `git push` → commit ใหม่ขึ้นใน PR อัตโนมัติ → comment แจ้ง reviewer
6. ทุกคน review อีกรอบ จนพอใจ → **Merge pull request** เข้า master
7. **ลบ development branch** หลัง merge
8. ทุกคน sync local:
```bash
git fetch -p          # -p (prune) ลบการอ้างอิง branch remote ที่ถูกลบแล้ว
git branch -vv        # ดู branch + tracking info
git branch -d <branch>   # ลบ branch ที่ merge แล้วใน local
```

## เชื่อมกับทฤษฎี
- **L11:** staging area, add/commit/push/pull, feature branch, master ต้องเสถียร (จึง merge ผ่าน PR ที่ review แล้ว)
- **L10:** modern code review ผ่าน tool (PR), benefits (correctness, knowledge sharing, team ownership, historical record), checklist
