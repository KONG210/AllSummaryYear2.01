# Lab 10 · Basic Router Configuration

## ภาพรวมของแล็บ

> 📌 อ้างอิง: **Packet Tracer 10.4.3 – Basic Device Configuration** (CCNA 1 v7) · ตารางที่อยู่ IP และรายการงานมาจากหน้าแล็บ · ส่วน **คำสั่ง IOS** เป็นคำสั่งมาตรฐานของ Cisco ที่ใช้ทำแต่ละขั้น (หน้าเฉลยฉบับเต็มดึงมาไม่ได้)

**เป้าหมายของแล็บ (Objectives)**
1. เติมเอกสารเครือข่าย (addressing table) ให้ครบ
2. ตั้งค่าพื้นฐาน (basic configuration) ให้ **router** และ **switch**
3. ตรวจสอบการเชื่อมต่อ และแก้ปัญหา (troubleshoot)

**สถานการณ์ (Scenario):** เราเป็น LAN technician ต้องตั้งค่า router ที่เชื่อม **2 LAN** เข้าด้วยกัน งานคือ
- ตั้งค่าพื้นฐานของ router และ switch ด้วย Cisco IOS
- ใส่ที่อยู่ **IPv4 และ IPv6** ให้อุปกรณ์เครือข่ายและเครื่อง host
- ทดสอบว่าเชื่อมต่อกันได้ครบ (end-to-end)

> ⚠️ หน้าแล็บระบุว่า **interface VLAN 1 ของ switch Class-A เข้าถึงด้วย IPv6 ไม่ได้** · สังเกตในตาราง switch ทั้งสองตัวไม่มี IPv6 · ถ้าทดสอบแล้ว IPv6 ไปหา Class-A ไม่ได้ ถือว่าปกติ

## 1. Addressing Table

**ภาพเครือข่าย:** router **College** อยู่ตรงกลาง ฝั่ง **G0/0** เป็น LAN วง 128.107.**20**.0 (switch Class-A + Student-1, 2) ฝั่ง **G0/1** เป็น LAN วง 128.107.**30**.0 (switch Class-B + Student-3, 4)

```text
 Student-1 ─┐                                          ┌─ Student-3
            ├─ [Class-A] ── G0/0 [College] G0/1 ── [Class-B] ─┤
 Student-2 ─┘   128.107.20.0/24          128.107.30.0/24       └─ Student-4
                2001:db8:a::/64          2001:db8:b::/64
```

| Device | Interface | IPv4 / Mask | IPv6 / Prefix | Default Gateway |
|---|---|---|---|---|
| **College** (router) | G0/0 | 128.107.20.1 /24 | 2001:db8:a::1/64 · link-local **FE80::1** | N/A |
| **College** (router) | G0/1 | 128.107.30.1 /24 | 2001:db8:b::1/64 · link-local **FE80::1** | N/A |
| **Class-A** (switch) | VLAN 1 | 128.107.20.2 /24 | ไม่มี | 128.107.20.1 |
| **Class-B** (switch) | VLAN 1 | 128.107.30.15 /24 | ไม่มี | 128.107.30.1 |
| Student-1 | NIC | 128.107.20.25 /24 | 2001:db8:a::2/64 | 128.107.20.1 · FE80::1 |
| Student-2 | NIC | 128.107.20.30 /24 | 2001:db8:a::3/64 | 128.107.20.1 · FE80::1 |
| Student-3 | NIC | 128.107.30.25 /24 | 2001:db8:b::2/64 | 128.107.30.1 · FE80::1 |
| Student-4 | NIC | 128.107.30.30 /24 | 2001:db8:b::3/64 | 128.107.30.1 · FE80::1 |

**สังเกตหลักการจากตาราง**
- /24 = subnet mask **255.255.255.0**
- **Gateway ของทุกเครื่องในวง = IP ของ router ขาที่อยู่วงเดียวกัน** (วง .20 ใช้ 128.107.20.1, วง .30 ใช้ 128.107.30.1)
- gateway IPv6 ของ host ใช้ **link-local FE80::1** ของ router (ทั้งสองขาตั้ง FE80::1 เหมือนกันได้ เพราะ link-local ใช้แค่ในลิงก์ของตัวเอง)
- router **ไม่มี default gateway** (N/A) เพราะ router คือ gateway เอง
- switch มี IPv4 ที่ VLAN 1 ไว้ **ให้ admin เข้าไปจัดการ (remote management)** และต้องมี default gateway ถึงจะจัดการจากต่างวงได้

## 2. งานที่ต้องทำ (ตามหน้าแล็บ)

1. เติมข้อมูลที่ขาดใน addressing table
2. ตั้งชื่อ router เป็น **College** และ switch ตัวที่สองเป็น **Class-B**
3. ตั้งรหัสผ่าน user EXEC ที่ **console** และ **VTY lines** และรหัสผ่าน **privileged EXEC แบบเข้ารหัส**
4. **เข้ารหัสรหัสผ่านที่เป็น plaintext** และตั้ง **MOTD banner**
5. ตั้ง IPv4 และ IPv6 ให้ router และ VLAN 1 ของ switch Class-B
6. ตั้ง IPv4 ของ host ให้ครบ รวมทั้ง IPv6 address และ gateway
7. ใส่ **description** ให้ interface (รวมถึง VLAN 1 ของ Class-B)
8. **บันทึก configuration**
9. ทดสอบว่าทุกเครื่อง ping หากันได้ทั้ง IPv4 และ IPv6 แล้วแก้ปัญหาที่ ping ไม่ผ่าน

> ⚠️ รหัสผ่านและข้อความ banner ที่แล็บกำหนด **ไม่ได้อยู่ในส่วนที่ดึงมาได้** ด้านล่างใช้ `<รหัส>` แทน ให้ใช้ค่าตามที่ Instructions ในแล็บกำหนด

## 3. โหมดของ Cisco IOS (ต้องรู้ก่อนพิมพ์คำสั่ง)

| โหมด | Prompt | เข้าด้วย | ใช้ทำอะไร |
|---|---|---|---|
| User EXEC | `College>` | เข้าเครื่องครั้งแรก | ดูข้อมูลได้นิดหน่อย, ping |
| Privileged EXEC | `College#` | `enable` | show ทุกอย่าง, บันทึก config |
| Global config | `College(config)#` | `configure terminal` | ตั้งค่าทั้งเครื่อง (hostname, password) |
| Interface config | `College(config-if)#` | `interface g0/0` | ตั้งค่า interface (IP, description) |
| Line config | `College(config-line)#` | `line console 0` / `line vty 0 4` | รหัสผ่าน console / remote |

- `exit` = ถอยหนึ่งโหมด · `end` (หรือ Ctrl+Z) = กลับไป Privileged EXEC ทันที
- พิมพ์ย่อได้ เช่น `conf t`, `int g0/0`, `sh ip int br`

## 4. ตั้งค่า Router College (คำสั่งมาตรฐาน)

### 🔹 ชื่อเครื่องและความปลอดภัย

```text
Router> enable
Router# configure terminal
Router(config)# hostname College
College(config)# enable secret <รหัส privileged>        ← รหัสเข้า # แบบเข้ารหัส
College(config)# line console 0
College(config-line)# password <รหัส console>
College(config-line)# login                             ← ต้องมี ไม่งั้นไม่ถามรหัส
College(config-line)# exit
College(config)# line vty 0 4
College(config-line)# password <รหัส vty>
College(config-line)# login
College(config-line)# exit
College(config)# service password-encryption           ← เข้ารหัสรหัสผ่าน plaintext
College(config)# banner motd #ข้อความเตือน#
```

| คำสั่ง | ทำอะไร |
|---|---|
| `enable secret` | รหัสเข้า privileged EXEC **เก็บแบบ hash เสมอ** (ปลอดภัยกว่า `enable password`) |
| `line console 0` + `password` + `login` | รหัสตอนต่อสาย console เข้าเครื่อง |
| `line vty 0 4` | รหัสสำหรับ remote (Telnet/SSH) 5 sessions (0–4) |
| `service password-encryption` | เข้ารหัสรหัสผ่านที่ยังเป็นข้อความธรรมดาใน config (console, vty) |
| `banner motd #...#` | ข้อความที่ขึ้นตอนมีคนเข้าเครื่อง (MOTD = Message Of The Day) ใช้ตัวคั่นหัวท้ายเหมือนกัน |

### 🔹 IPv4 / IPv6 และ description ที่ interface

```text
College(config)# ipv6 unicast-routing                  ← เปิดให้ router ส่งต่อ IPv6
College(config)# interface g0/0
College(config-if)# description Connects to Class-A LAN
College(config-if)# ip address 128.107.20.1 255.255.255.0
College(config-if)# ipv6 address 2001:db8:a::1/64
College(config-if)# ipv6 address fe80::1 link-local
College(config-if)# no shutdown                         ← interface router ปิดอยู่ตั้งต้น
College(config-if)# exit
College(config)# interface g0/1
College(config-if)# description Connects to Class-B LAN
College(config-if)# ip address 128.107.30.1 255.255.255.0
College(config-if)# ipv6 address 2001:db8:b::1/64
College(config-if)# ipv6 address fe80::1 link-local
College(config-if)# no shutdown
College(config-if)# end
College# copy running-config startup-config            ← บันทึก
```

> ⭐ จุดที่คนลืมบ่อยที่สุด: **`no shutdown`** (interface ของ router ปิดอยู่เป็นค่าเริ่มต้น) และ **`ipv6 unicast-routing`** (ไม่เปิด = router ไม่ส่งต่อ IPv6 ข้ามวง)

> ⚠️ IPv4 ต้องพิมพ์ mask เต็ม `255.255.255.0` · IPv6 ใช้ `/64` ต่อท้ายได้เลย

## 5. ตั้งค่า Switch Class-B (คำสั่งมาตรฐาน)

```text
Switch> enable
Switch# configure terminal
Switch(config)# hostname Class-B
Class-B(config)# enable secret <รหัส>
Class-B(config)# line console 0
Class-B(config-line)# password <รหัส>
Class-B(config-line)# login
Class-B(config-line)# line vty 0 15                     ← switch มี vty 0–15
Class-B(config-line)# password <รหัส>
Class-B(config-line)# login
Class-B(config-line)# exit
Class-B(config)# service password-encryption
Class-B(config)# banner motd #ข้อความเตือน#
Class-B(config)# interface vlan 1
Class-B(config-if)# description Management interface
Class-B(config-if)# ip address 128.107.30.15 255.255.255.0
Class-B(config-if)# no shutdown
Class-B(config-if)# exit
Class-B(config)# ip default-gateway 128.107.30.1       ← ให้ switch คุยข้ามวงได้
Class-B(config)# end
Class-B# copy running-config startup-config
```

| ต่างจาก router อย่างไร | อธิบาย |
|---|---|
| IP ใส่ที่ **interface vlan 1** (SVI) | switch layer 2 ไม่มี IP ที่พอร์ต ใช้ virtual interface ไว้จัดการเครื่อง |
| ใช้ **`ip default-gateway`** (global config) | router ไม่ใช้คำสั่งนี้ · host ตั้ง gateway ที่การ์ดแลน |
| switch ในแล็บนี้ **ไม่มี IPv6** | ตามตาราง (จึง ping IPv6 ไป switch ไม่ได้) |

## 6. ตั้งค่า Host (PC)

ใน Packet Tracer: คลิก PC → **Desktop → IP Configuration**
- **IPv4:** เลือก Static ใส่ IP Address, Subnet Mask 255.255.255.0, Default Gateway ตามตาราง
- **IPv6:** เลือก Static ใส่ IPv6 Address + prefix 64, IPv6 Gateway = **FE80::1**

ตัวอย่าง Student-3: IPv4 128.107.30.25 / 255.255.255.0 / gateway 128.107.30.1 · IPv6 2001:db8:b::2/64 / gateway FE80::1

## 7. ตรวจสอบและแก้ปัญหา

| คำสั่ง | ที่ไหน | ใช้ดูอะไร |
|---|---|---|
| `show ip interface brief` | router/switch | IPv4 ของแต่ละ interface + สถานะ **up/up** หรือ **administratively down** |
| `show ipv6 interface brief` | router | IPv6 (global + link-local) และสถานะ |
| `show running-config` | router/switch | config ทั้งหมดที่กำลังใช้ |
| `show interfaces` / `show version` | router/switch | รายละเอียด interface / ข้อมูลเครื่อง |
| `ping <ip>` | ทุกเครื่อง | ทดสอบการเชื่อมต่อ (PC: Desktop → Command Prompt) |
| `ipconfig` / `ipv6config` | PC | ดูค่าที่ตั้งไว้ใน PC |

**ลำดับการไล่ปัญหาเมื่อ ping ไม่ผ่าน**
1. ping gateway ของตัวเองก่อน (เช่น Student-1 → 128.107.20.1) ถ้าไม่ผ่าน = ปัญหาในวงตัวเอง
2. ดู `show ip interface brief` ที่ router ว่า interface **up/up** ไหม ถ้า administratively down = ลืม `no shutdown`
3. เช็ก IP, mask, gateway ของ PC ว่าตรงตารางไหม (พิมพ์ผิดหลักเดียวก็ไม่ผ่าน)
4. IPv6 ข้ามวงไม่ผ่าน → ลืม `ipv6 unicast-routing` หรือ PC ตั้ง IPv6 gateway ผิด
5. ping switch จากต่างวงไม่ผ่าน → switch ลืม `ip default-gateway` หรือ VLAN 1 ยัง shutdown

## จุดที่มักสับสน

| เรื่อง | A | B | จำง่ายๆ |
|---|---|---|---|
| `enable secret` vs `enable password` | เก็บแบบ hash (ปลอดภัย) | เก็บ plaintext | ถ้ามีทั้งคู่ **secret ชนะ** |
| `service password-encryption` vs `enable secret` | เข้ารหัสแบบอ่อน ให้รหัส console/vty | hash ที่แข็งแรง สำหรับ privileged | encryption = ปิดไม่ให้คนมองเห็นใน config |
| `ip default-gateway` vs gateway ของ PC | ใช้กับ **switch** (global config) | ตั้งที่การ์ดแลนของ **PC** | router ไม่ต้องมีทั้งสองแบบ |
| `running-config` vs `startup-config` | อยู่ใน RAM ปิดเครื่องหาย | อยู่ใน NVRAM เปิดเครื่องใหม่โหลดอันนี้ | ไม่ copy = ไฟดับแล้วหาย |
| Global unicast vs link-local | 2001:db8:a::1 ใช้ข้ามวงได้ | FE80::1 ใช้แค่ในลิงก์ เป็น gateway ของ host | link-local ซ้ำกันได้คนละขา |

## จุดเน้นสำหรับสอบ
- 🔥 ลำดับโหมด: `>` → `enable` → `#` → `configure terminal` → `(config)#` → `interface` / `line`
- 🔥 คำสั่งชุดความปลอดภัย: `hostname`, `enable secret`, `line console 0` / `line vty 0 4` + `password` + `login`, `service password-encryption`, `banner motd`
- 🔥 interface router: `ip address <ip> <mask>`, `ipv6 address .../64`, `ipv6 address fe80::1 link-local`, `description`, **`no shutdown`**
- 🔥 `ipv6 unicast-routing` เพื่อ route IPv6 · `copy running-config startup-config` เพื่อบันทึก
- ⭐ switch: IP ที่ `interface vlan 1` + `ip default-gateway`
- ⭐ gateway ของ host = IP ของ router ขาวงเดียวกัน · IPv6 gateway = FE80::1
- ⭐ ตรวจด้วย `show ip interface brief`, `show ipv6 interface brief`, `ping`
