# Lab 13 · สรุปก่อนสอบ

> รายละเอียดจากสไลด์เลคเชอร์ยังไม่มี — ด้านล่างเป็นพื้นฐานทั่วไป

- init/systemd = **PID 1** (`ps -p 1`, `pidof systemd`)
- Shutdown: `shutdown -h now | +10`, `shutdown -r now`, `shutdown -c`, `poweroff`, `reboot` · recovery mode ผ่าน GRUB (Shift/Esc)
- Users: `adduser`, `useradd -m`, `passwd`, `usermod -aG group user`, `groupadd`, `deluser`/`userdel -r`, `id`, `groups` · `/etc/passwd` (user info), `/etc/shadow` (hash password, root only), `/etc/group`
- Monitoring: `top`, `htop`, `ps aux`, `free -h`, `df -h`, `du -sh`, `uptime`, `who`, `w`, `last`
- Cron: `crontab -e / -l` · `นาที ชั่วโมง วัน เดือน วันในสัปดาห์ คำสั่ง` · `*/5 * * * *` ทุก 5 นาที
- `.profile` = login shell (PATH) · `.bashrc` = interactive shell (HISTSIZE, PS1, alias)
- Logs: **wtmp** (`last`, login history) · **syslog** (ระบบทั่วไป) · **auth.log** (login, sudo, ssh)
