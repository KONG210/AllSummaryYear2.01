# Lab 13 · Linux System Admin

> ⚠️ ไฟล์ Lab มีแค่รายการงาน — รายละเอียด (เช่น cron job ในสไลด์ #39) อยู่ในสไลด์เลคเชอร์ที่ยังไม่ได้ส่งมา ความรู้ด้านล่างเป็นพื้นฐาน Linux admin ทั่วไป

## งานที่ต้องทำ
**Objectives:** จัดการ Linux server · จัดการ users · system monitoring · cron jobs

1. ดู **PID ของ process "init"**
2. ทดสอบการ **shutdown** หลายรูปแบบ
3. ทดสอบ boot ด้วย **safe/recovery mode**
4. รันเป็น **root** แล้ว shutdown
5. ทำตามขั้นตอน **user management**
6. รัน **cron job** ตามสไลด์ #39
7. แสดงไฟล์ **`.profile` และ `.bashrc`** และบอกว่ามี **environment variables** อะไร
8. แสดงตัวอย่าง log: **wtmp, syslog, auth.log** และอธิบาย 1 บรรทัดของแต่ละ log

PDF ชื่อ `6887xxx_lab13.pdf`

## ความรู้ที่ต้องใช้

### 🔹 init process
```bash
ps -ef | head          # init/systemd คือ PID 1
pidof systemd
ps -p 1 -o pid,comm
```
> init (ปัจจุบันคือ **systemd**) เป็น process แรกที่ kernel สร้าง **PID = 1** และเป็นต้นตระกูลของทุก process

### 🔹 Shutdown / reboot
```bash
sudo shutdown -h now        # ปิดทันที
sudo shutdown -h +10        # ปิดในอีก 10 นาที
sudo shutdown -r now        # รีบูต
sudo shutdown -c            # ยกเลิก shutdown ที่ตั้งไว้
sudo poweroff ; sudo reboot ; sudo halt
sudo su -                   # เป็น root
```
- Recovery mode: ตอน boot กด **Shift/Esc** เข้าเมนู GRUB → *Advanced options* → *recovery mode* (root shell, fsck ฯลฯ)

### 🔹 User management
```bash
sudo adduser alice              # สร้าง user (interactive)
sudo useradd -m -s /bin/bash bob
sudo passwd bob                 # ตั้งรหัสผ่าน
sudo usermod -aG sudo alice     # เพิ่มเข้ากลุ่ม sudo
sudo groupadd devs ; sudo usermod -aG devs bob
id alice ; groups alice
sudo deluser bob ; sudo userdel -r bob   # -r ลบ home ด้วย
cat /etc/passwd ; sudo cat /etc/shadow ; cat /etc/group
```
| ไฟล์ | เก็บอะไร |
|---|---|
| `/etc/passwd` | ข้อมูล user (ชื่อ, UID, GID, home, shell) |
| `/etc/shadow` | **รหัสผ่านที่ hash แล้ว** (อ่านได้เฉพาะ root) |
| `/etc/group` | ข้อมูลกลุ่ม |

### 🔹 System monitoring
```bash
top ; htop          # CPU/memory แบบ real-time
ps aux              # process ทั้งหมด
free -h             # memory
df -h               # disk
du -sh <dir>        # ขนาด directory
uptime ; who ; w ; last
```

### 🔹 Cron jobs
```bash
crontab -e          # แก้ cron ของ user
crontab -l          # ดูรายการ
```
```text
# ┌ minute (0-59)  ┌ hour (0-23)  ┌ day of month (1-31)  ┌ month (1-12)  ┌ day of week (0-7, 0/7 = Sun)
# │                │              │                      │               │
  */5              *              *                      *               *    /home/user/backup.sh
  0                2              *                      *               1    date >> /tmp/mon2am.log
```
- `*/5 * * * *` = ทุก 5 นาที · `0 2 * * 1` = ตี 2 ทุกวันจันทร์

### 🔹 .profile และ .bashrc
```bash
cat ~/.profile ; cat ~/.bashrc
echo $PATH ; env
```
| ไฟล์ | รันเมื่อ | มักมี |
|---|---|---|
| `~/.profile` | **login shell** | `PATH` (เพิ่ม `$HOME/bin`, `$HOME/.local/bin`), เรียก `.bashrc` |
| `~/.bashrc` | **interactive non-login shell** (เปิด terminal) | `HISTSIZE`, `HISTFILESIZE`, `HISTCONTROL`, `PS1` (prompt), alias (`ll`, `ls --color`) |

### 🔹 Logs
| Log | ที่อยู่ | ดูด้วย | เก็บอะไร |
|---|---|---|---|
| **wtmp** | `/var/log/wtmp` (binary) | `last` | ประวัติ **login/logout/reboot** |
| **syslog** | `/var/log/syslog` | `tail`, `less` | log ระบบทั่วไป (service, kernel) |
| **auth.log** | `/var/log/auth.log` | `sudo tail` | **การยืนยันตัวตน**: login, sudo, ssh |

```bash
last | head
sudo tail -n 20 /var/log/syslog
sudo grep sudo /var/log/auth.log | tail
```
ตัวอย่างอธิบาย auth.log: `... sudo: alice : TTY=pts/0 ; PWD=/home/alice ; USER=root ; COMMAND=/usr/bin/apt update` = user alice ใช้ sudo รัน `apt update` ในฐานะ root
