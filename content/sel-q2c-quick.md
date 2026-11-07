# Git Functions · สรุปก่อนสอบ

## Lab 9
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

## อ่านผล git status
- `Untracked files` → add · `Changes not staged` (modified) → add · `Changes to be committed` (staged) → commit · `working tree clean` → push ได้
- `* master` ใน `git branch` = branch ปัจจุบัน · push `[rejected] (fetch first)` → pull ก่อน
