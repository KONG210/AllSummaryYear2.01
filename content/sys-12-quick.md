# Lab 12 · สรุปก่อนสอบ

- **LAMP** = Linux + Apache + MySQL + PHP · phpMyAdmin จัดการ MySQL ผ่านเว็บ
- VirtualBox → **Bridged Adapter** · เน็ตมหิดลตั้ง proxy ใน `/etc/apt/apt.conf.d/proxy`
- `sudo apt update` → `sudo apt install apache2` → `ip address` → เปิด `http://IP`
- Document root: **`/var/www/html`**
- `systemctl status | start | stop | restart | reload | enable | disable | is-active | is-enabled apache2`
  - reload = โหลด config ไม่หยุดบริการ · enable/disable = มีผลตอน boot
- Config: **`/etc/apache2/apache2.conf`** (หลัก) · `conf-` / `mods-` / `sites-` **available** (มี) vs **enabled** (เปิดใช้) · sites = virtual hosts
- SSH: `apt install openssh-server` + WinSCP โอนไฟล์
- MySQL: `apt install -y mysql-server mysql-client`, `mysql -V`
- PHP: `apt install -y libapache2-mod-php php-mysql ...`, `php -v`, ทดสอบ `<?php phpinfo(); ?>` ที่ `/info.php`
- phpMyAdmin: install → `cp /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf` → **`a2enconf phpmyadmin`** → `systemctl reload apache2` → `/phpmyadmin/`
- `ps -ef | grep apache2|mysql|php`
- MySQL CLI: `mysql -u user -p` · `show databases; use db; show tables;` · `create user ict identified by 'ict555';` · `grant all privileges on *.* to 'ict';` · `select user from mysql.user;`
