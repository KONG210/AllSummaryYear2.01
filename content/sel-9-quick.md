# Lab 9 · Git cheat sheet

```bash
git config --global user.name "..." ; git config --global user.email "..."
git init                     # เริ่ม repo
git clone <url>              # คัดลอก repo จาก GitHub
git status                   # สถานะ: untracked / modified / staged
git add <file> | git add .   # → staging
git commit -m "msg"          # → local repo
git log [--name-status]      # ประวัติ
git diff <file>              # ดูที่แก้ (ยังไม่ add)
git diff <id1> <id2> <file>  # เทียบ 2 commit
git push                     # local → remote
git fetch                    # ดึงข้อมูล remote (ไม่ merge)
git pull                     # fetch + merge
git branch                   # ดู branch
git branch <name>            # สร้าง
git checkout <name>          # ย้าย
git push --set-upstream origin <name>   # push branch ใหม่ครั้งแรก
git branch -a                # ทุก branch รวม remote
git fetch -p                 # prune branch remote ที่ถูกลบ
git branch -vv               # ดู tracking
git branch -d <name>         # ลบ branch local
```

- Terminal: `dir`/`ls`, `cd`/`pwd`, `mkdir` · Vim: `i` → พิมพ์ → `ESC` → `:w` → `:q`
- ไฟล์ใน branch หนึ่งไม่เห็นใน branch อื่นจนกว่าจะ merge
- **PR flow:** New pull request → ใส่รายละเอียด + assign reviewer → reviewer กด "+" comment ที่บรรทัด → submit review → author แก้แล้ว push (commit เข้า PR อัตโนมัติ) → review อีกรอบ → **Merge** → ลบ branch → ทุกคน fetch -p
