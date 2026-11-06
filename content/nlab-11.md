# Lab 11 · Troubleshoot Default Gateway Issues

## ภาพรวมของแล็บ

> 📌 อ้างอิง: **Packet Tracer 10.3.5 – Troubleshoot Default Gateway Issues** (CCNA 1 v7)

**Objectives**
- **Part 1:** ตรวจเอกสารเครือข่าย (documentation) และหาจุดที่มีปัญหา (isolate problems)
- **Part 2:** ลงมือแก้ ตรวจผล และจดบันทึก (implement, verify, document)

**Background (ตามแล็บ)**
- host จะคุยกับเครือข่ายอื่นได้ต้องมี **IP address, subnet mask และ default gateway**
- **default gateway** ปกติคือ **interface ของ router ที่อยู่ใน LAN เดียวกับ host**
- วิธีแก้ปัญหาที่แล็บใช้มี **5 ขั้น:** ① ตรวจเอกสาร + หาจุดปัญหา → ② หาวิธีแก้ → ③ ลงมือแก้ → ④ ทดสอบ → ⑤ จดบันทึก

## 1. Addressing Table

```text
 PC1 ─┐                                   ┌─ PC3
      ├─ [S1] ── G0/0 [R1] G0/1 ── [S2] ─┤
 PC2 ─┘  192.168.10.0/24   192.168.11.0/24 └─ PC4
```

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | N/A |
| R1 | G0/1 | 192.168.11.1 | 255.255.255.0 | N/A |
| S1 | VLAN 1 | 192.168.10.2 | 255.255.255.0 | **192.168.10.1** |
| S2 | VLAN 1 | 192.168.11.2 | 255.255.255.0 | **192.168.11.1** |
| PC1 | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC2 | NIC | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC3 | NIC | 192.168.11.10 | 255.255.255.0 | 192.168.11.1 |
| PC4 | NIC | 192.168.11.11 | 255.255.255.0 | 192.168.11.1 |

> ⭐ กฎในตาราง: ทุกอุปกรณ์ในวง **192.168.10.x** ใช้ gateway **192.168.10.1** (R1 G0/0) · ทุกอุปกรณ์ในวง **192.168.11.x** ใช้ **192.168.11.1** (R1 G0/1) · switch ก็ต้องมี gateway ด้วย

## 2. ขั้นตอนของแล็บ

### 🔹 Part 1: Verify Documentation and Isolate Problems
1. **Step 1a** เติม addressing table ให้ครบ (รวม default gateway ของ switch และ PC)
2. **Step 1b** ทดสอบ **local connectivity** ก่อน เช่น PC1 → PC2, PC1 → S1, PC1 → R1 แล้วจดผล · **แก้ปัญหาในวงตัวเองให้เสร็จก่อนค่อยทดสอบข้ามวง**
3. **Step 1c** ทดสอบ **remote (end-to-end)** เช่น PC1 → PC4 แล้วจดปัญหา
4. **Step 2a** หาสาเหตุที่น่าจะเป็น: คิดว่าอุปกรณ์ไหนทำให้ล้มเหลว ดูจากสถานะลิงก์และ config
5. **Step 2b** เทียบ IP ของอุปกรณ์กับเอกสาร (ใช้ `ipconfig`)
6. **Step 2c** เสนอและจดวิธีแก้ทีละ 1 อย่าง · **เปลี่ยนทีละอย่าง (change one thing at a time)**

### 🔹 Part 2: Implement, Verify, and Document
1. **Step 1** ลงมือแก้ตามวิธีแรกที่เลือก
2. **Step 2** รันการทดสอบเดิมซ้ำ ถ้าผ่านให้ทำเครื่องหมายว่า verified
3. **Step 3** ถ้ายังมีปัญหา กลับไป Part 2 Step 1 · ถ้าปัญหาในวงแก้หมดแล้ว กลับไป Part 1 Step 1c ทดสอบข้ามวงอีกรอบ

**คำถามนำในแล็บ:** PC1 ping PC2 ได้หรือยัง? · ปัญหาข้ามวงหายด้วยไหม (PC1 ping PC4 ได้ไหม)? · วิธีแก้ตรงกับเอกสารไหม?

## 3. ปัญหาที่ซ่อนไว้ในแล็บ (4 จุด)

| # | อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|---|
| a | **PC1 ping PC2 ไม่ได้** | PC1 มี IP **อยู่นอกวง** ที่ต่ออยู่ | ตั้ง PC1 = **192.168.10.10/24**, gateway **192.168.10.1** |
| b | **ping S2 ไม่ได้** และ S2 ping อะไรไม่ได้ | **S2 ไม่มี IP** | ตั้ง IP ที่ **VLAN 1 SVI** = 192.168.11.2/24 + `no shutdown` + gateway 192.168.11.1 |
| c | เครื่องวงอื่นติดต่อ **PC4** ไม่ได้ | PC4 ตั้ง **default gateway ผิด** | ตั้ง gateway PC4 = **192.168.11.1** |
| d | เครื่องวงอื่นติดต่อ **S1** ไม่ได้ | **S1 ไม่มี default gateway** | `ip default-gateway 192.168.10.1` บน S1 |

> ⭐ สังเกตแพทเทิร์น: **IP ผิดวง → ping ในวงเดียวกันไม่ผ่าน** · **gateway ผิด/ไม่มี → ในวงผ่าน แต่ข้ามวงไม่ผ่าน**

## 4. คำสั่งที่ใช้

**ที่ PC (Command Prompt)**
```text
ipconfig                                        ← ดู IP / mask / gateway
ipconfig 192.168.10.10 255.255.255.0 192.168.10.1   ← ตั้งค่าจาก command prompt
ping 192.168.10.11
ping 192.168.11.11
```

**ที่ switch (S1, S2)**
```text
S2> enable
S2# configure terminal
S2(config)# interface vlan 1
S2(config-if)# ip address 192.168.11.2 255.255.255.0
S2(config-if)# no shutdown
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.11.1
S2(config)# end
S2# copy running-config startup-config
```
(S1 ใช้ `ip default-gateway 192.168.10.1`)

**ตรวจสอบ**
| คำสั่ง | ดูอะไร |
|---|---|
| `show running-config` | มีบรรทัด `ip default-gateway` หรือยัง |
| `show ip interface brief` | VLAN 1 เป็น **up/up** และ IP ถูกต้อง |

## จุดที่มักสับสน

| อาการ | น่าจะเป็น | จำง่ายๆ |
|---|---|---|
| ping เครื่องในวงเดียวกันไม่ได้ | IP / mask ผิด (อยู่นอกวง) | ในวงไม่ใช้ gateway เลย |
| ในวงได้ แต่ข้ามวงไม่ได้ | default gateway ผิดหรือไม่มี | gateway = ประตูออกนอกวง |
| ping switch ไม่ได้เลย | switch ไม่มี IP ที่ VLAN 1 หรือ VLAN 1 shutdown | switch ไม่มี IP = ไม่มีตัวตนให้ ping |
| ping switch ได้จากในวง แต่ข้ามวงไม่ได้ | switch ไม่มี `ip default-gateway` | switch ตอบกลับข้ามวงไม่ได้ |

## จุดเน้นสำหรับสอบ
- 🔥 host ต้องมี **IP + mask + default gateway** · gateway = router interface ใน LAN เดียวกัน
- 🔥 4 ปัญหาในแล็บ: PC1 IP ผิดวง · S2 ไม่มี IP · PC4 gateway ผิด · S1 ไม่มี gateway
- 🔥 วิธีแก้ปัญหา 5 ขั้น · ทดสอบ **local ก่อน remote** · **แก้ทีละอย่าง**
- ⭐ `ipconfig` (PC) · `interface vlan 1` + `ip address` + `no shutdown` + `ip default-gateway` (switch)
- ⭐ ตรวจด้วย `show running-config`, `show ip interface brief`, `ping`
