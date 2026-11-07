# Quiz 2 · Git Functions

> 📌 เรื่องที่ 3 จาก 4 ของ Quiz 2 SE Lab · กลับไป [ภาพรวม Quiz 2](#sel-q2) · ทวนขั้นตอนเต็มใน [Lab 9 Git](#sel-9)

## 1. คำสั่งที่ต้องรู้ (จาก Lab 9)

| คำสั่ง | ทำอะไร |
|---|---|
| `git config --global user.name / user.email` | ตั้งชื่อและอีเมลที่จะติดไปกับ commit |
| `git init` | เริ่มใช้ Git ในโฟลเดอร์ (สร้าง local repo) |
| `git status` | ดูสถานะไฟล์ (untracked / modified / staged) |
| `git add <file>` / `git add .` | นำไฟล์เข้า **staging area** |
| `git commit -m "msg"` | บันทึกเวอร์ชันใน **local repo** |
| `git log` / `git log --name-status` | ดูประวัติ commit (+ ไฟล์ที่เปลี่ยน) |
| `git diff <file>` | ดูว่าแก้อะไรที่ยังไม่ stage |
| `git diff <id1> <id2> <file>` | เทียบไฟล์ระหว่าง 2 commit |
| `git clone <url>` | คัดลอก remote repo ลงเครื่อง |
| `git push` | ส่ง commit จาก local ขึ้น remote |
| `git fetch` | ดึงข้อมูลจาก remote มาดู **ยังไม่ merge** |
| `git pull` | **fetch + merge** เข้า branch ปัจจุบัน |
| `git branch` / `git branch <name>` | ดู branch / สร้าง branch |
| `git checkout <name>` | ย้ายไป branch นั้น |
| `git push --set-upstream origin <name>` | push branch ใหม่ครั้งแรก และผูก local กับ remote (`-u`) |
| `git branch -a` | ดู branch ทั้งหมดรวมฝั่ง remote |
| `git fetch -p` | fetch และลบการอ้างอิง branch remote ที่ถูกลบแล้ว (prune) |
| `git branch -vv` | ดู branch พร้อม tracking info |
| `git branch -d <name>` | ลบ branch local ที่ merge แล้ว |

**สถานะไฟล์:** untracked → (`add`) → **staged** → (`commit`) → **committed** → แก้ไฟล์ → **modified** → (`add`) → staged …

**Pull Request flow:** push branch → New pull request → assign reviewer → reviewer comment (กด "+" หน้าบรรทัด) → เจ้าของแก้ + push (commit ใหม่เข้า PR เอง) → review ซ้ำ → **Merge** เข้า master → ลบ branch → ทุกคน `git fetch -p`

## 2. Git Cheat Sheet (จาก Lecture)

> 📌 จาก **GitHub Education – Git Cheat Sheet** ที่ใช้ในวิชาทฤษฎี · รวมคำสั่งที่ Lab 9 ไม่ได้ใช้ ควรรู้ความหมายทุกตัว

**Setup**
| คำสั่ง | ความหมาย |
|---|---|
| `git config --global user.name "[firstname lastname]"` | ตั้งชื่อ ใช้ระบุว่าใครทำอะไรในประวัติ |
| `git config --global user.email "[valid-email]"` | ตั้งอีเมลที่ติดไปกับทุก commit |
| `git config --global color.ui auto` | เปิดสีใน command line ให้อ่านง่าย |

**Setup & Init**
| คำสั่ง | ความหมาย |
|---|---|
| `git init` | ทำให้โฟลเดอร์ที่มีอยู่เป็น Git repository |
| `git clone [url]` | ดึง repository ทั้งหมดจาก URL ที่ host ไว้ |

**Stage & Snapshot**
| คำสั่ง | ความหมาย |
|---|---|
| `git status` | แสดงไฟล์ที่ถูกแก้ใน working directory และไฟล์ที่ stage ไว้รอ commit |
| `git add [file]` | stage ไฟล์ตามที่เป็นตอนนี้ เพื่อ commit ครั้งถัดไป |
| `git reset [file]` | **ยกเลิกการ stage** ไฟล์ แต่ **ยังเก็บการแก้ไว้** ใน working directory |
| `git diff` | ดูสิ่งที่แก้แต่ **ยังไม่ stage** |
| `git diff --staged` | ดูสิ่งที่ **stage แล้วแต่ยังไม่ commit** |
| `git commit -m "[descriptive message]"` | บันทึกสิ่งที่ stage เป็น snapshot ใหม่ |

**Branch & Merge**
| คำสั่ง | ความหมาย |
|---|---|
| `git branch` | แสดง branch ทั้งหมด · `*` อยู่หน้า branch ที่ใช้อยู่ |
| `git branch [branch-name]` | สร้าง branch ใหม่ที่ commit ปัจจุบัน |
| `git checkout` | ย้ายไป branch อื่น และ checkout มาที่ working directory |
| `git merge [branch]` | รวมประวัติของ branch ที่ระบุ **เข้ามาใน branch ปัจจุบัน** |
| `git log` | แสดง commit ทั้งหมดในประวัติของ branch ปัจจุบัน |

**Inspect & Compare**
| คำสั่ง | ความหมาย |
|---|---|
| `git log` | ประวัติ commit ของ branch ที่ใช้อยู่ |
| `git log branchB..branchA` | commit ที่อยู่ใน **branchA แต่ไม่อยู่ใน branchB** |
| `git log --follow [file]` | commit ที่เปลี่ยนไฟล์นี้ **แม้ไฟล์ถูกเปลี่ยนชื่อ** |
| `git diff branchB...branchA` | diff ของสิ่งที่อยู่ใน **branchA แต่ไม่อยู่ใน branchB** |
| `git show [SHA]` | แสดง object ใดๆ ใน Git แบบอ่านง่าย |

**Tracking Path Changes**
| คำสั่ง | ความหมาย |
|---|---|
| `git rm [file]` | ลบไฟล์ออกจากโปรเจกต์ และ **stage การลบ** ไว้รอ commit |
| `git mv [existing-path] [new-path]` | ย้าย/เปลี่ยนชื่อไฟล์ และ **stage การย้าย** |
| `git log --stat -M` | แสดง log พร้อมบอกว่า path ไหนถูกย้าย |

**Ignoring Patterns**
- สร้างไฟล์ **`.gitignore`** ใส่ pattern แบบตรงตัวหรือ wildcard เช่น `logs/`, `*.notes`, `pattern*/` เพื่อ **กันไม่ให้ stage/commit ไฟล์ที่ไม่ต้องการ**
- `git config --global core.excludesfile [file]` = ignore pattern ระดับระบบ ใช้กับทุก local repo

**Share & Update**
| คำสั่ง | ความหมาย |
|---|---|
| `git remote add [alias] [url]` | เพิ่ม URL ของ git เป็นชื่อเล่น (alias) เช่น `origin` |
| `git fetch [alias]` | ดึงทุก branch จาก remote นั้นลงมา |
| `git merge [alias]/[branch]` | merge branch ฝั่ง remote เข้า branch ปัจจุบัน ให้ทันสมัย |
| `git push [alias] [branch]` | ส่ง commit ของ branch local ขึ้น branch ฝั่ง remote |
| `git pull` | **fetch และ merge** commit จาก remote branch ที่ติดตามอยู่ |

**Rewrite History**
| คำสั่ง | ความหมาย |
|---|---|
| `git rebase [branch]` | นำ commit ของ branch ปัจจุบันไป **ต่อท้าย** branch ที่ระบุ |
| `git reset --hard [commit]` | ล้าง staging area และ **เขียน working tree ทับ** ด้วย commit ที่ระบุ (งานที่ยังไม่ commit หาย) |

**Temporary Commits (Stash)** – เก็บไฟล์ที่แก้ไว้ชั่วคราวเพื่อสลับ branch
| คำสั่ง | ความหมาย |
|---|---|
| `git stash` | เก็บการแก้ทั้งที่ modified และ staged ไว้ก่อน |
| `git stash list` | ดูรายการ stash เรียงแบบ stack |
| `git stash pop` | เอางานจาก **บนสุดของ stack** กลับมา |
| `git stash drop` | **ทิ้ง** งานบนสุดของ stack |

**คู่ที่ข้อสอบชอบถาม**
| คู่ | ต่างกันอย่างไร |
|---|---|
| `git diff` vs `git diff --staged` | ยังไม่ stage vs stage แล้วแต่ยังไม่ commit |
| `git reset [file]` vs `git reset --hard [commit]` | แค่เอาออกจาก staging (งานยังอยู่) vs ล้างทั้ง staging และ working tree |
| `git fetch` vs `git pull` | ดึงมาอย่างเดียว vs ดึง + merge |
| `git merge` vs `git rebase` | รวมประวัติของอีก branch เข้ามา vs ย้าย commit ของเราไปต่อท้ายอีก branch |
| `git stash pop` vs `git stash drop` | เอากลับมาใช้ vs ทิ้ง |
| `git rm` vs ลบไฟล์เอง | `git rm` ลบ + stage การลบให้ด้วย |
| `git log branchB..branchA` | อ่านว่า "อยู่ใน A แต่ไม่อยู่ใน B" (ตัวหลังลบตัวหน้า) |

## 3. แบบฝึก C1: Git จากสถานการณ์ (Lab 9)

เขียนคำสั่งให้ครบก่อนเปิดเฉลย
1. สร้างโฟลเดอร์ใหม่ให้เป็น Git repo แล้ว commit ไฟล์ `app.java` ครั้งแรก
2. แก้ `app.java` แล้วอยากดูว่าแก้อะไรไปบ้าง**ก่อน** add
3. สร้าง branch ชื่อ `dev_kong` ย้ายไปทำงาน แล้ว push branch นี้ขึ้น GitHub ครั้งแรก
4. อยากรู้ว่าบน remote มีอะไรใหม่ แต่**ยังไม่อยากให้รวม**เข้ากับงานตัวเอง
5. อยากรวมงานเพื่อนจาก remote เข้ามาใน branch ปัจจุบันเลย
6. commit ไฟล์ `Feature.java` ใน `dev_kong` แล้ว `git checkout master` แล้วสั่ง `ls` จะเห็น `Feature.java` ไหม เพราะอะไร
7. PR ถูก merge และ branch บน GitHub ถูกลบแล้ว ต้องทำอะไรที่เครื่องตัวเอง
8. `git status` แสดง `Changes to be committed: modified: se.txt` ไฟล์อยู่ในสถานะอะไร และขั้นต่อไปคืออะไร

<details><summary>เฉลยแบบฝึก C1</summary>
<ol>
<li><code>git init</code> → <code>git add app.java</code> → <code>git commit -m "Added app.java"</code></li>
<li><code>git diff app.java</code></li>
<li><code>git branch dev_kong</code> → <code>git checkout dev_kong</code> → (commit งาน) → <code>git push --set-upstream origin dev_kong</code></li>
<li><code>git fetch</code> (ดึงมาดูแต่ยังไม่ merge)</li>
<li><code>git pull</code> (= fetch + merge)</li>
<li><b>ไม่เห็น</b> เพราะ commit อยู่ใน branch dev_kong ไฟล์จะไม่ปรากฏใน branch อื่นจนกว่าจะ merge</li>
<li><code>git checkout master</code> → <code>git pull</code> → <code>git fetch -p</code> → <code>git branch -vv</code> (ดูว่า branch ไหน gone) → <code>git branch -d dev_kong</code></li>
<li><b>staged</b> (add แล้ว) · ขั้นต่อไป <code>git commit -m "..."</code></li>
</ol></details>

## 4. แบบฝึก C2: คำสั่งจาก Cheat Sheet

1. `git add` ไฟล์ `login.java` ไปแล้ว แต่อยากเอาออกจาก staging โดย**ไม่ให้งานที่แก้หาย**
2. อยากดูว่า**สิ่งที่ stage ไว้แล้ว**ต่างจาก commit ล่าสุดอย่างไร
3. กำลังทำงานค้างใน branch `feature` (ยังไม่อยาก commit) แต่ต้องรีบสลับไปแก้บั๊กที่ `master` แล้วค่อยกลับมาทำต่อ
4. อยากรวมงานจาก branch `feature` เข้า `master`
5. อยากรู้ว่ามี commit อะไรบ้างที่อยู่ใน `feature` แต่ยังไม่อยู่ใน `master`
6. เปลี่ยนชื่อไฟล์ `util.java` เป็น `helper.java` ให้ Git รู้และ stage ไว้ให้เลย
7. ไม่อยากให้ไฟล์ทุกตัวที่ลงท้าย `.log` และโฟลเดอร์ `build/` ถูก commit
8. เพิ่ม remote ชื่อ `origin` ที่ URL `https://github.com/team/app.git` แล้ว push branch `master` ขึ้นไป
9. ทดลองแก้ไปมั่วจนพัง อยากย้อน working directory กลับไปเหมือน commit `a1b2c3` ทั้งหมด (ยอมให้งานที่ยังไม่ commit หาย)

<details><summary>เฉลยแบบฝึก C2</summary>
<ol>
<li><code>git reset login.java</code> (unstage แต่การแก้ยังอยู่)</li>
<li><code>git diff --staged</code></li>
<li><code>git stash</code> → <code>git checkout master</code> → แก้บั๊ก + commit → <code>git checkout feature</code> → <code>git stash pop</code></li>
<li><code>git checkout master</code> → <code>git merge feature</code> (merge เข้า branch ปัจจุบัน)</li>
<li><code>git log master..feature</code> (อยู่ใน feature แต่ไม่อยู่ใน master)</li>
<li><code>git mv util.java helper.java</code></li>
<li>สร้างไฟล์ <code>.gitignore</code> ใส่บรรทัด <code>*.log</code> และ <code>build/</code></li>
<li><code>git remote add origin https://github.com/team/app.git</code> → <code>git push origin master</code></li>
<li><code>git reset --hard a1b2c3</code></li>
</ol></details>

## 5. แบบฝึก C3: อ่านผลลัพธ์ของ Git

อ่านผลแต่ละอันแล้วตอบ: ไฟล์/repo อยู่สถานะอะไร และควรทำอะไรต่อ

**(1)**
```text
$ git status
Untracked files:
        Hello.java
```
**(2)**
```text
$ git status
Changes not staged for commit:
        modified:   se.txt
```
**(3)**
```text
$ git status
Changes to be committed:
        new file:   Feature.java
```
**(4)**
```text
$ git status
nothing to commit, working tree clean
```
**(5)**
```text
$ git branch
  dev_kong
* master
```
**(6)**
```text
$ git push
 ! [rejected]        master -> master (fetch first)
```

<details><summary>เฉลยแบบฝึก C3</summary>
<ol>
<li><b>Untracked</b> Git ยังไม่ติดตาม → <code>git add Hello.java</code></li>
<li><b>Modified</b> แก้แล้วยังไม่ stage → <code>git add se.txt</code> (ดูก่อนได้ด้วย <code>git diff se.txt</code>)</li>
<li><b>Staged</b> (ไฟล์ใหม่ add แล้ว) → <code>git commit -m "..."</code> (ถ้าจะเอาออกจาก staging ใช้ <code>git reset Feature.java</code>)</li>
<li>ทุกอย่าง <b>commit แล้ว</b> ไม่มีอะไรค้าง → ถ้าจะส่งขึ้น remote ใช้ <code>git push</code></li>
<li>อยู่ที่ branch <b>master</b> (เครื่องหมาย <code>*</code>) · มีอีก branch คือ dev_kong → ย้ายด้วย <code>git checkout dev_kong</code></li>
<li>push <b>ถูก reject</b> เพราะ remote มีการเปลี่ยนแปลงที่ local ยังไม่มี → <code>git pull</code> ก่อน แล้วค่อย <code>git push</code> (ตรงกับ L11: push อาจถูก reject ถ้า remote มีของใหม่)</li>
</ol></details>

## จุดเน้นสำหรับสอบ
- 🔥 สถานะไฟล์: untracked → staged → committed → modified และคำสั่งที่พาไปแต่ละสถานะ
- 🔥 **fetch vs pull** · `push --set-upstream` ครั้งแรก · ไฟล์ใน branch หนึ่งไม่โผล่ใน branch อื่นจนกว่า merge
- 🔥 Cheat sheet: `reset [file]` vs `reset --hard` · `diff` vs `diff --staged` · `merge` vs `rebase` · `stash` / `pop` / `drop` · `rm` / `mv` · `.gitignore` · `log B..A`
- ⭐ PR flow: push branch → PR → review → แก้ + push → merge → ลบ branch → `fetch -p`
- ⭐ push ถูก reject → pull ก่อน
