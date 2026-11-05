# Lab 12 · Apache, MySQL, PHP Installation (LAMP)

## ภาพรวม
> 📌 **LAMP stack** = **L**inux, **A**pache, **M**ySQL, **P**HP (หรือ Perl/Python) — นิยมใช้บน Ubuntu server ทำ web server + database server · **phpMyAdmin** = แอป LAMP สำหรับจัดการ MySQL ผ่านเว็บ

> 🔗 เชื่อมกับ ITDS241: Apache คือ web server แบบเดียวกับที่ Node/Express ทำหน้าที่ใน Web, MySQL คือฐานข้อมูลเดียวกับ tinycollege

(ตารางเรียนวิชานี้: สัปดาห์ 10 shell script · 11 process/thread · 12 LAMP · 13 system monitoring · 14 Linux admin · 15 Windows · การวัดผล: lab report 25, quiz 25, midterm 25, final 25, เข้าเรียน 80%)

---

## 1. ติดตั้ง Apache Web Server

**ขั้นตอน**
1. VirtualBox: เปลี่ยน Network เป็น **Bridged Adapter**
2. ถ้าใช้เน็ตมหิดล ตั้ง **proxy**:
```bash
sudo nano /etc/apt/apt.conf.d/proxy
# Acquire::http::proxy "http://<user>:<password>@proxy-sa.mahidol:8080";
export http_proxy="http://usrname:passwrd@host:port"
```
3. ทดสอบออก Internet และติดตั้ง:
```bash
sudo apt update
sudo apt install apache2
ip address                      # ดู IP ของ server
```
4. เปิด browser บน Windows ไปที่ `http://<server IP>` ต้องเห็นหน้า Apache
5. สร้างหน้าเว็บเอง:
```bash
cd /var/www/html                # document root
sudo nano index.html
```

### 🔹 ตรวจสถานะและจัดการ service
```bash
sudo systemctl is-active apache2
sudo systemctl is-enabled apache2
sudo systemctl status apache2

sudo systemctl stop apache2      # หยุด → เข้าเว็บไม่ได้
sudo systemctl start apache2     # เริ่ม
sudo systemctl restart apache2   # หยุดแล้วเริ่มใหม่
sudo systemctl reload apache2    # โหลด config ใหม่โดยไม่หยุด
sudo systemctl disable apache2   # ไม่เริ่มอัตโนมัติตอน boot
sudo systemctl enable apache2    # เริ่มอัตโนมัติตอน boot
```
**แบบฝึก:** stop → ลองเข้าเว็บ (เข้าไม่ได้) → restart → ลองใหม่ (เข้าได้)

| | restart | reload |
|---|---|---|
| ทำอะไร | หยุด process แล้วเริ่มใหม่ (connection หลุด) | อ่าน config ใหม่ **โดยไม่หยุดบริการ** |

| | start/stop | enable/disable |
|---|---|---|
| มีผลเมื่อ | **ตอนนี้** | **ตอน boot ครั้งถัดไป** |

### 🔹 Configuration files (`/etc/apache2`)
| Path | ความหมาย |
|---|---|
| `/etc/apache2/apache2.conf` | **ไฟล์ config หลัก (global)** |
| `conf-available/` | config ที่มีให้ใช้ |
| `conf-enabled/` | config ที่เปิดใช้งาน |
| `mods-available/` | module ที่มี |
| `mods-enabled/` | module ที่เปิดใช้ |
| `sites-available/` | config ของ site (**virtual hosts**) ที่มี |
| `sites-enabled/` | config ของ site ที่เปิดใช้ |

### 🔹 SSH + WinSCP
```bash
sudo apt install openssh-server
```
- บน Windows ติดตั้ง **WinSCP** → login ด้วย username/password ของ Ubuntu → สร้างหน้าเว็บใหม่แล้วโอนไฟล์ไปที่ server → ทดสอบเปิดดู

---

## 2. ติดตั้ง MySQL และ PHP
```bash
sudo apt update
sudo apt install -y mysql-server mysql-client
mysql -V                                         # ดูเวอร์ชัน MySQL
sudo apt install -y libapache2-mod-php php-mysql php-cgi php-common php-mbstring
php -v                                           # ดูเวอร์ชัน PHP
```
**ทดสอบ PHP:**
```bash
cd /var/www/html
sudo nano /var/www/html/info.php
```
```php
<?php phpinfo(); ?>
```
เปิด `http://<server IP>/info.php` (เช่น http://10.34.1.44/info.php) ต้องเห็นหน้าข้อมูล PHP

### 🔹 phpMyAdmin
```bash
sudo apt install -y phpmyadmin php-zip php-gd php-json php-curl
# ระหว่างติดตั้งจะถาม config และตั้งรหัสผ่าน phpmyadmin — ห้ามลืม!
sudo cp /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin                         # enable config
sudo systemctl reload apache2
```
เข้า `http://<server IP>/phpmyadmin/` · user: `phpmyadmin` · password: ที่ตั้งไว้

### 🔹 ตรวจ process
```bash
ps -ef | grep apache2
ps -ef | grep mysql
ps -ef | grep php
```

---

## 3. ใช้ MySQL ที่ command line
```bash
mysql -u phpmyadmin -p          # login ด้วย user phpmyadmin
```
```sql
show databases;
use phpmyadmin;
show tables;
```
**สร้าง user ใหม่ (ใช้ root)**
```bash
sudo mysql -u root -p
```
```sql
create user ict identified by 'ict555';
grant all privileges on *.* to 'ict';
```
```bash
mysql -u ict -p
```
```sql
select user from mysql.user;
show databases;
create database student;        -- (ตามสไลด์: สร้าง student database)
use student;                    -- ต้อง use ก่อนเสมอ
create table ... ;              -- สร้างตาราง (รายละเอียดในสไลด์เป็นรูป)
insert into ... values (...);   -- เพิ่ม record
```

## 4. ใช้ MySQL ผ่าน PHP
- สร้าง `logindb.php` (เชื่อมต่อ DB ด้วยรหัสผ่านของตัวเอง) แล้วรันผ่าน browser
- สร้าง `testdb.php` แล้วรันด้วย `php testdb.php` ดู output
(โค้ด PHP ในสไลด์เป็นรูป — รูปแบบทั่วไป:)
```php
<?php
$conn = new mysqli("localhost", "ict", "ict555", "student");
if ($conn->connect_error) die("Connection failed: " . $conn->connect_error);
echo "Connected successfully";
$result = $conn->query("SELECT * FROM <table>");
while ($row = $result->fetch_assoc()) print_r($row);
$conn->close();
?>
```
