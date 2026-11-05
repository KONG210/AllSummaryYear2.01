# AIS · 5G Introduction (บรรยายพิเศษ 5 พ.ย. 2026)

## ภาพรวมของบท

> 📌 บรรยายพิเศษโดย **AIS** ร่วมกับ ICT มหิดล · ข้อสอบ Final มี **ข้อจากบริษัทเอกชนบริษัทละ 3 ข้อ** บทนี้น่าจะเป็นหนึ่งในนั้น

**บทนี้เกี่ยวกับอะไร:** เล่าวิวัฒนาการของเครือข่ายมือถือจาก 1G ถึง 6G, เทคโนโลยีใหม่ใน 5G (Ambient IoT, ดาวเทียม/NTN), สถาปัตยกรรม 5G แบบ NSA/SA, ความปลอดภัยใน 5G (NEA/NIA), แล้วต่อไปที่ Steganography/Cryptography และภัยจาก Quantum Computing (QKD, PQC)

**ลำดับหัวข้อ (ตาม Agenda ในสไลด์)**
1. From 1G to 5G and beyond
2. Evolution of technologies features
3. 5G Network Architecture
4. End-to-end security in 5G network
5. Steganography & Cryptography
6. Quantum Computing, QKD and PQC
7. THE MONSTER ACTS ON THE MOONSHINE MODULE (ตัวอย่างการซ่อนข้อความในภาพ)
8. Backup slides (อธิบาย quantum เพิ่มเติม)

**เส้นเรื่องที่เชื่อมกัน:** มือถือพัฒนาทุก ~10 ปี → 5G เพิ่มความสามารถใหม่ (IoT, ดาวเทียม) → ยิ่งเชื่อมต่อมาก ยิ่งต้องมี **security** (เข้ารหัส + ตรวจความถูกต้อง) → การเข้ารหัสวันนี้พึ่ง "คณิตศาสตร์ที่แก้ยาก" → **quantum computer** อาจแก้ได้เร็ว → ต้องมี **QKD / PQC** · ส่วน steganography คืออีกแนวของการปกป้องข้อความ (ซ่อน ไม่ใช่เข้ารหัส)

## 1. From 1G to 5G and beyond

### 🔹 วิวัฒนาการ 1G → 6G

**คืออะไร?** เครือข่ายมือถือเปลี่ยนรุ่นใหญ่ ("G" = Generation) ประมาณ **ทุก 10 ปี** แต่ละรุ่นมีจุดเด่นต่างกัน

| รุ่น | ยุค | จุดเด่น | ลักษณะ | เทคโนโลยีตัวอย่าง |
|---|---|---|---|---|
| **1G** | 1980s | Mobile voice communication | **Analog voice** | AMPS, NMT, TACS |
| **2G** | 1990s | Efficient voice to reach billions | **Digital voice** | D-AMPS, GSM, IS-95 (CDMA) |
| **3G** | 2000s | Focus shifts to **mobile data** | **Wireless Internet** | CDMA2000/EV-DO, WCDMA/HSPA+ |
| **4G** | 2010s | Mobile broadband and emerging expansion | **Mobile broadband** | LTE, LTE Advanced, Gigabit LTE |
| **5G** | 2020s | A **unified connectivity platform** | **Connected intelligent edge** | **5G New Radio (NR)** |
| **6G** | 2030s | The next innovation platform | **Next-gen wireless** | **AI-native**, new spectrum, **RF sensing** |

> ⭐ **"Mobile has made a leap every ~10 years"** · จำคู่ให้ได้: 1G analog → 2G digital voice → 3G data → 4G LTE broadband → 5G NR → 6G AI-native

**ตัวอย่างให้เห็นภาพ:** 1G โทรได้อย่างเดียว เสียงเป็นสัญญาณ analog → 2G เริ่มส่ง SMS ได้เพราะเป็น digital → 3G เปิดเว็บบนมือถือได้ → 4G ดูวิดีโอ/สตรีมได้ลื่น → 5G เชื่อมได้ทั้งคน เครื่องจักร รถ IoT ในแพลตฟอร์มเดียว

### 🔹 Works toward 6G (Release ของ 3GPP)

**คืออะไร?** มาตรฐานมือถือออกโดย **3GPP** เป็นชุดๆ เรียกว่า **Release (Rel)** ออกห่างกันราว **1.5–2 ปี**

| ช่วง | Release | ความหมาย |
|---|---|---|
| **5G** | **Rel-15, 16, 17** (2018–2022) | "A unified platform for innovations" |
| **5G Advanced** | **Rel-18, 19, 20** (2022–2027) | "2nd wave of 5G innovations" และจะพัฒนาต่อเนื่องในยุค 6G |
| **6G** | **Rel-20** (study) → **Rel-21+** (work item, ~2028–2030+) | Next technology leap; ขั้นตอน: foundational research → vision forming → service requirements → study item → work item/trials → IoDTs |

- มี **3GPP 6G workshop** ก่อนเริ่ม Rel-20
- เส้นเวลาด้านล่างมี **WRC-19, WRC-23, WRC-27, WRC-31** (World Radiocommunication Conference = การประชุมจัดสรรคลื่นความถี่ระดับโลก)

> ⭐ 5G = Rel-15 ถึง 17 · **5G Advanced = Rel-18 ขึ้นไป** · 6G เริ่มที่ Rel-20 (ศึกษา) / Rel-21 (มาตรฐาน)

## 2. Evolution of technologies features

### 🔹 ฟีเจอร์ในแต่ละ Release (สไลด์วงกลม Qualcomm)

สไลด์ไฮไลต์กรอบสีเหลืองไว้หลายจุด (น่าจะเป็นจุดที่ผู้บรรยายเน้น):

| Release | ฟีเจอร์ที่ถูกไฮไลต์ | ฟีเจอร์อื่นในสไลด์ |
|---|---|---|
| **Rel-15** (วางรากฐาน 5G NR) | **eMBB** (enhanced Mobile Broadband), **LTE integration** | 5G core + E2E security, Sub-6 GHz + massive MIMO, mmWave, scalable OFDM, advanced channel coding |
| **Rel-16** (ขยายไปอุตสาหกรรม) | **5G NR Cellular V2X** (รถคุยกับสิ่งรอบตัว) | eURLLC (mission-critical), positioning, 5G NR in unlicensed spectrum, IAB, private networks/SDN |
| **Rel-17** (ขยายต่อ) | **Non-terrestrial network (ดาวเทียม)** | NR-Light (RedCap) สำหรับ IoT, spectrum > 52.6 GHz, centimeter-accuracy IIoT, sidelink |
| **Rel-18** (5G Advanced เริ่ม) | **AI/ML data-driven designs** | NTN enhancements, XR, smart repeaters, full-duplex MIMO, NR-Light expansion |
| **Rel-19** ("Our Focus Today") | **Wireless AI**, **Ambient IoT**, **Integrated sensing and communications (ISAC)** | XR evolution, enhanced NTN, duplex evolution, energy savings, higher midband spectrum |

**ศัพท์ที่ควรรู้**
| Term | ความหมาย | อธิบายง่ายๆ |
|---|---|---|
| **eMBB** | enhanced Mobile Broadband | เน็ตมือถือเร็วขึ้นมาก (ดูวิดีโอ 4K) |
| **URLLC** | Ultra-Reliable Low-Latency Communication | เชื่อถือได้สูง หน่วงต่ำมาก (หุ่นยนต์, ผ่าตัดทางไกล) |
| **V2X** | Vehicle-to-Everything | รถคุยกับรถ/ไฟจราจร/คนเดินถนน |
| **NTN** | Non-Terrestrial Network | เครือข่ายที่ไม่ได้อยู่บนพื้น เช่น ดาวเทียม |
| **ISAC** | Integrated Sensing and Communication | ใช้คลื่นมือถือ "รับรู้" สิ่งรอบตัวได้ด้วย (คล้ายเรดาร์) |
| **RedCap / NR-Light** | Reduced Capability | 5G รุ่นเบาสำหรับ IoT ราคาถูก |

### 🔹 Ambient IoT

**คืออะไร?** IoT ที่ **ใช้พลังงานต่ำมากระดับไมโครวัตต์** อาจไม่ต้องมีแบตเตอรี่ ดึงพลังงานจากสิ่งแวดล้อม เป้าหมาย: **"further scale down and expand the reach of 5G IoT"**

**หลักการสำคัญ**
- เป็น **Study Item ใน Release 19** แบบ "Full Stack" ข้าม RAN และ SA working groups
- เทียบกำลังส่ง (Tx power):

| อุปกรณ์ | กำลังส่ง |
|---|---|
| 5G IoT (RedCap, eMTC, NB-IoT) | **200 mW** |
| Low-power Wi-Fi | **50 mW** |
| Bluetooth Low Energy | **1 mW** |
| **Ambient IoT** | **~1 µW** |

- ใช้ peak power **1 ถึงหลักร้อย µW** อาจมี energy storage, ใช้ **backscatter** ได้ (สะท้อนคลื่นกลับแทนการส่งเอง)
- เป้าหมายคลื่น **sub-7 GHz FDD** แบบ in-band 5G NR, guard-band LTE/5G NR หรือ stand-alone
- งานแบ่งตาม RAN working group: RAN1 air interface (frame, sync, waveform, modulation, coding) · RAN2 protocol stack แบบเบา (paging, random access) · RAN3 interface CN-RAN, การหาตำแหน่งอุปกรณ์ · RAN4 การอยู่ร่วมกับ 5G NR/LTE และ RF requirements

**ตัวอย่าง:** ป้าย tag ติดกล่องสินค้าในโกดังที่ไม่มีแบต แต่ส่งตำแหน่งให้เครือข่าย 5G ได้ (คล้าย RFID ที่ต่อกับเครือข่ายมือถือ)

> ⭐ Ambient IoT = **Rel-19**, กำลังส่ง **~1 µW** (ต่ำกว่า BLE 1 mW ถึง 1000 เท่า)

### 🔹 Satellite technologies and NTN

**วงโคจรดาวเทียม (ต้องจำตัวเลข)**
| ชนิด | ชื่อเต็ม | ความสูง |
|---|---|---|
| **GEO** | Geostationary Earth Orbit | **35,785.5 km** (ลอยอยู่กับที่เทียบกับโลก) |
| **MEO** | Medium Earth Orbit | **2,000 – 20,350 km** |
| **LEO** | Low Earth Orbit | **300 – 2,000 km** (เช่น Starlink) |

ในแผนภาพยังมีชั้นที่ต่ำกว่าดาวเทียม: **HAPS** (High Altitude Platform Station, ~15–45 km), **เครื่องบิน** (~10–15 km), **UAV/โดรน** และสถานีภาคพื้น (Core/RAN+Core)

**ลิงก์ 3 แบบในแผนภาพ**
| ลิงก์ | เชื่อมอะไร |
|---|---|
| **Feeder link** | ดาวเทียม/HAPS ↔ สถานีภาคพื้น (gateway) |
| **Service link** | ดาวเทียม/HAPS/เครื่องบิน ↔ ผู้ใช้ |
| **Backhaul** | เชื่อมสถานีฐานกลับเข้าเครือข่ายหลัก |

**NTN (Non-Terrestrial Networks)**
1. คือ **การรวมเครือข่าย 5G เข้ากับการสื่อสารดาวเทียม**
2. พัฒนาตามมาตรฐาน **3GPP**
3. เป้าหมาย: เชื่อม **พื้นที่ห่างไกล**, เพิ่ม broadband ทั่วโลก, ใช้ใน **สถานการณ์ภัยพิบัติ**, รองรับ IoT ในที่ที่ไม่มีเสาภาคพื้น

> ⭐ ใกล้โลกที่สุด → ไกลสุด: **LEO (300–2,000) < MEO (2,000–20,350) < GEO (35,785.5)** · ยิ่งใกล้ ยิ่ง delay ต่ำ แต่ต้องใช้ดาวเทียมหลายดวงเพราะวิ่งผ่านเร็ว (ส่วนนี้เป็นความรู้ทั่วไปประกอบ)

### 🔹 3GPP NTN options (TR 38.811)

| แบบ | ชื่อเรียก | ดาวเทียมทำอะไร | 5G RAN อยู่ที่ |
|---|---|---|---|
| **1. Non-regenerative payload** | **Transparent (bent-pipe)** | แค่ **รับแล้วส่งต่อ** สัญญาณ (เหมือนกระจกสะท้อน) NR via f1 ลงผู้ใช้, NR via f2 ไป gateway | **ภาคพื้น** (gateway → 5G RAN → 5G CN) |
| **2. Regenerative payload** | **Non-transparent (on-board processor)** | **มี 5G RAN อยู่บนดาวเทียม** ประมวลผลเองได้ คุยกับ gateway ด้วย NGc/NGu over f2 | **บนดาวเทียม** (gateway → 5G CN) |

> ⚠️ จำง่าย: **bent-pipe = ท่อโค้ง** แค่ส่งผ่าน (non-regenerative) · **regenerative = สร้างสัญญาณใหม่** เพราะมีเครื่องประมวลผล (RAN) บนดาวเทียม

## 3. 5G Network Architecture

### 🔹 5G NSA vs 5G SA

| | **5G NSA (Non-Standalone)** | **5G SA (Standalone)** |
|---|---|---|
| RAN | **New 5G RAN interworking with 4G** (NR ทำงานคู่กับ LTE) | **Separated 5G RAN and 4G RAN** |
| Core | **Common EPC core network** (ใช้ core ของ 4G) | **New 5G core network (5GC)** |
| ภาพในสไลด์ | มือถือต่อทั้ง NR และ LTE, ทั้งคู่วิ่งไป EPC | มือถือ 5G ต่อ NR → 5GC แยกจากฝั่ง LTE → EPC |

- เส้นทึบ = **User plane** (ข้อมูลผู้ใช้) · เส้นประ = **Control plane** (สัญญาณควบคุม) ใน NSA control plane ยังผ่าน LTE
- **EPC** = Evolved Packet Core (core ของ 4G) · **5GC** = 5G Core

**ทำไมต้องรู้:** NSA ทำให้ผู้ให้บริการเปิด 5G ได้เร็วเพราะใช้ core 4G เดิม แต่จะได้ความสามารถเต็มของ 5G (เช่น network slicing, latency ต่ำ) ต้องเป็น SA ที่มี 5GC (ส่วนเหตุผลนี้เป็นความรู้ทั่วไปประกอบ)

> ⭐ **NSA = ใช้ EPC (core 4G) ร่วม** · **SA = มี 5G core ใหม่ของตัวเอง**

### 🔹 Will Satellite D2D replace Terrestrial Mobile service?

- **D2D (Direct-to-Device)**: ดาวเทียมคุยกับ **มือถือธรรมดาโดยตรง** ตัวอย่าง **Starlink Mobile** "A cellphone tower in space" ใช้ได้กับ **มือถือ LTE ทั่วไปที่ไม่ต้องดัดแปลง** ทุกที่ที่มองเห็นท้องฟ้า ดาวเทียมมี phased array antenna ทำตัวเหมือนเสาสัญญาณ เชื่อมกันด้วยเลเซอร์ และต่อเข้าเครือข่ายผู้ให้บริการเหมือน **roaming partner**
- เส้นทาง: Unmodified cell phone ↔ Starlink constellation ↔ Starlink gateway site ↔ Partner operator network → Text, IoT, Data, Voice

**ประเด็นพิจารณา 3 ข้อ (key considerations):**
1. **Frequency spectrum** (คลื่นความถี่)
2. **Transport network** (เครือข่ายขนส่งข้อมูล)
3. **Indoor coverage** (สัญญาณในอาคาร) ← ดาวเทียมต้อง "มองเห็นท้องฟ้า"

> ⚠️ สไลด์ตั้งเป็นคำถาม ไม่ได้ให้คำตอบตรงๆ · ประเด็นที่ควรตอบได้คือ 3 ข้อพิจารณาข้างบน (โดยเฉพาะ indoor coverage ที่ดาวเทียมทำได้ยาก)

## 4. End-to-end security in 5G network

### 🔹 5G security architecture (3GPP TS 33.501)

**5G security layers 5 ด้าน (ต้องจำ)**
1. **Network access security** – ความปลอดภัยตอนอุปกรณ์เข้าใช้เครือข่าย
2. **Network domain security** – ระหว่างโหนดในเครือข่าย
3. **User domain security** – ระหว่างผู้ใช้กับอุปกรณ์/USIM
4. **Application domain security** – ระหว่าง user application กับ provider application
5. **SBA domain security** – SBA = Service-Based Architecture ของ 5G core

**แผนภาพในสไลด์** แบ่งเป็น 3 ชั้น: **Application stratum** (user application ↔ provider application) · **Home/serving stratum** (USIM, SN = serving network, HE = home environment) · **Transport stratum** (ME = mobile equipment, 3GPP access, non-3GPP access)

**5G system-wide security coverage:** ครอบคลุม 4 ส่วน คือ **Device & GWs · Access & network · Application & cloud infrastructure · ID management for users** แต่ละส่วนมี security management, security functions และ trust anchor (เป็นโมเดลความปลอดภัยแบบแนวนอนเดียวกันทั้งระบบ)

### 🔹 5G security methods

| Threat (ภัย) | Impact (กระทบ) | 5G security | ใช้กับ |
|---|---|---|---|
| **Eavesdropping** (ดักฟัง) | **Data Confidentiality** (ความลับ) | **NEA** (NR Encryption Algorithm) | Signaling plane messages, User plane data |
| **Tampering** (แก้ไขข้อมูล) | **Data Integrity** (ความถูกต้อง) | **NIA** (NR Integrity Algorithm) | Signaling plane messages, User plane data **(optional)** |

**การปรับปรุงความปลอดภัยของ 5G (เทียบรุ่นก่อน)**
| เรื่อง | ปรับอย่างไร | Release |
|---|---|---|
| **Enhanced User Data Protection** | UP (user plane) และ CP (control plane) รองรับทั้ง **encryption และ integrity protection** (NAS: UE↔AMF, RRC/UP: UE↔gNB) | **R15** |
| **Enhanced Privacy Protection** | **Permanent User ID encrypted all time** (เดิม user ID เคยถูกเปิดเผยตอนเข้าใช้ครั้งแรก) | **R15** |
| **Stronger Security Algorithm** | จาก 128-bit keys → **รองรับ 128-bit และ 256-bit keys** | **R16** |

ศัพท์: **UE** = User Equipment (มือถือ) · **gNB** = สถานีฐาน 5G · **AMF** = Access and Mobility Management Function (ใน 5G core) · **NAS** = Non-Access Stratum · **RRC** = Radio Resource Control

### 🔹 NEA Algorithms (Confidentiality)

| ID | Name | Underlying cipher | Key | Origin | 3GPP standard |
|---|---|---|---|---|---|
| **NEA0** | Null encryption | **ไม่เข้ารหัส** | N/A | 3GPP | TS 33.501 |
| **128-NEA1** | SNOW 3G based | SNOW 3G stream cipher | 128 bits | **ETSI SAGE** | TS 35.215 / 35.216 |
| **128-NEA2** | AES based | **AES-128-CTR** | 128 bits | **NIST (US)** | TS 35.226 / 35.227 |
| **128-NEA3** | ZUC based | ZUC 1.6 stream cipher | 128 bits | **ZUC Design Team (China)** | TS 35.231 / 35.232 |

### 🔹 NIA Algorithms (Integrity)

| ID | Name | Underlying cipher | Key | MAC | 3GPP standard |
|---|---|---|---|---|---|
| **NIA0** | Null integrity | None | N/A | N/A | TS 33.501 |
| **128-NIA1** | SNOW 3G based | SNOW 3G + MAC function | 128 bits | **32 bits** | TS 35.215 / 35.217 |
| **128-NIA2** | AES based | **AES-128-CMAC** | 128 bits | **32 bits** | TS 35.226 / 25.228 (ตามที่เขียนในสไลด์) |
| **128-NIA3** | ZUC based | ZUC 1.6 + MAC function | 128 bits | **32 bits** | TS 35.231 / 35.233 |

> ⭐ เลขต่อท้าย **0 = null, 1 = SNOW 3G, 2 = AES, 3 = ZUC** ใช้เหมือนกันทั้ง NEA และ NIA · key 128 bits · MAC 32 bits · AES ฝั่ง NEA ใช้ **CTR** ฝั่ง NIA ใช้ **CMAC**

> ⚠️ **NEA = Encryption = ความลับ (กันดักฟัง)** · **NIA = Integrity = ความถูกต้อง (กันแก้ไข)** · integrity บน user plane เป็น **optional**

## 5. Steganography & Cryptography

| | **Steganography** | **Cryptography** |
|---|---|---|
| นิยาม (สไลด์) | **Concealment** – make the message **invisible** to unintended observers | **Encoding** – transform the message into **cipher text** to prevent unauthorized reading |
| แปลง่ายๆ | **ซ่อน**ให้ไม่รู้ว่ามีข้อความ | **เข้ารหัส**ให้อ่านไม่ออก (แต่รู้ว่ามีข้อความ) |
| ตัวอย่างในสไลด์ | **499 BCE: Histiaeus of Miletus โกนหัวทาส** เขียนข้อความบนหนังศีรษะ (รอผมขึ้นแล้วส่งไป) ถึง Aristagoras ให้ก่อกบฏต่อเปอร์เซีย | **WW2: เครื่อง Enigma** (3-rotor model I, 8-rotor model H) และเครื่องถอดรหัสฝ่ายตรงข้าม **the Bombe** |

**รายละเอียด Enigma**
- จำนวน combination ที่เป็นไปได้ **107,458,687,327,250,619,360,000 ≈ 10^23**
- จุดอ่อนจากคน: ทหารเยอรมันใช้ **ชื่อแฟนเป็นคีย์** เช่น **"CILLY"** นำไปสู่การถอด Enigma ครั้งแรก (ภาพยนตร์ **The Imitation Game**)

> ⭐ จำง่าย: **stegano = ซ่อน (มองไม่เห็น)** · **crypto = แปลง (อ่านไม่ออก)** · บทเรียนจาก Enigma: ต่อให้คีย์มีเป็น 10^23 แบบ ถ้าคนตั้งคีย์เดาง่ายก็โดนถอดได้

## 6. Quantum Computing, QKD and PQC

### 🔹 What is Quantum Computing?

**นิยาม (สไลด์):** Quantum computing ใช้ **quantum mechanics** แก้ปัญหาที่ **classical computer ทำไม่ได้ในเวลาที่เป็นไปได้**

| | Classical | Quantum |
|---|---|---|
| หน่วย | **bit** | **qubit** |
| สถานะ | **0 หรือ 1** (1 state at a time, bits independent) | **0, 1 หรือทุกค่าระหว่างกลางพร้อมกัน** (many states at a time, qubits in combined state) |
| หลักที่ใช้ | – | **superposition, entanglement, interference** |

**ตารางเทียบความเร็ว (ในสไลด์)**
| จำนวน qubit | ข้อมูลที่เก็บได้ | Quantum | Classical supercomputer |
|---|---|---|---|
| 1 qubit | 2 ชิ้น | 1 µs | 0.00000000000000006 s (**classical เร็วกว่า**) |
| 20 qubits | 1 ล้าน (10^6) | 1 µs | 1 µs (เท่ากัน) |
| 50 qubits | 1 พันล้านล้าน (10^15) | 1 µs | **40,000 ปี** |
| 100 qubits | 10^30 | 1 µs | **50,000-million-trillion-trillion ปี** |

> ⭐ ข้อมูลเพิ่มแบบ **2^n** ตามจำนวน qubit (20 qubits ≈ 2^20 ≈ 1 ล้าน) · quantum ได้เปรียบเมื่อ qubit มาก ไม่ใช่ทุกงาน

### 🔹 Today's Quantum Computers

| ค่าย | เครื่อง/ชิป | รายละเอียดในสไลด์ |
|---|---|---|
| **IBM** | IBM Quantum System Two | เชื่อมหลาย QPU ใน data center (Thomas J. Watson Research Center), processor อยู่ใน "chandelier" |
| **Google** | **Sycamore** (Oct 2019) | **54 qubits** |
| **Google** | (25 Dec 2024) | คำนวณใน **5 นาที** สิ่งที่ supercomputer ใช้ **10^25 ปี** (จักรวาลอายุ ~13.8 × 10^9 ปี) |
| **Google** | **Willow** | **105 qubits** |
| **China (USTC)** | **Zuchongzhi 3.0** (Mar 2025) | **105 qubits** superconducting, อ้างว่าเร็วกว่า supercomputer **10^15 เท่า**, คู่แข่งตรงของ Willow · จีนให้ Baidu/Alibaba หยุดวิจัยเอง รวมคนเข้ารัฐวิสาหกิจ |
| **SpinQ** (สตาร์ทอัพเซินเจิ้น) | Triangulum II / Gemini Mini / Gemini Lab | **3-qubit desktop** / **2-qubit portable** สำหรับการศึกษา / lab ทดลองระดับ pulse-gate-algorithm |
| **Fujitsu + RIKEN** | (Apr 2025) | **256 qubits** superconducting, เปิดให้บริษัท/สถาบันใช้ตั้งแต่ Q1 FY2025 |

**IBM roadmap (ย่อ):** Falcon (27 qubits) → Eagle (127) → Heron (133, 2024) → Nighthawk (120 qubits, 5K→15K gates, 2025–2028) → **Starling (2029): fault-tolerant, 100M gates บน 200 logical qubits** → **Blue Jay (2033+): 1B gates, 2,000 logical qubits**

### 🔹 Quantum Key Distribution (QKD)

**คืออะไร?** วิธีแลก **กุญแจเข้ารหัส** โดยใช้คุณสมบัติควอนตัมของ **photon (อนุภาคแสง)** ถ้ามีคนแอบดู จะรู้ทันที

**หลักการ (5 ข้อในสไลด์)**
1. แต่ละ photon เข้ารหัส 1 bit ใน quantum state เช่น **polarization** หรือ **phase**
2. photon เดินทางผ่าน **fiber** หรือ **free-space** channel
3. **การวัด (เช่น ดักฟัง) จะเปลี่ยนคุณสมบัติของ photon** ← หัวใจของ QKD
4. **Quantum channel** ใช้ส่ง raw key material
5. **Classical channel** ใช้ประสานงานและ **authentication** ระหว่างผู้ส่ง-ผู้รับ

**ตัวอย่างในแผนภาพ (Leia → Luke):** Leia ส่งบิตด้วย polarizer แบบ diagonal หรือ horizontal-vertical สุ่มๆ Luke วัดด้วย beamsplitter สุ่มเหมือนกัน · บิตที่ทั้งคู่ใช้ "แบบเดียวกัน" จะถูกเก็บเป็น **sifted key** ส่วนที่ไม่ตรงทิ้ง (ขีด `-`)

### 🔹 Post Quantum Cryptography (PQC)

**คืออะไร?** การเข้ารหัสแบบใหม่ที่ **ทนการโจมตีจาก quantum computer** (แต่ยังรันบนคอมธรรมดาได้)

**เหตุผล (ตามสไลด์)**
1. การเข้ารหัสวันนี้ (**RSA, ECC**) อาศัย **ความยากทางคณิตศาสตร์** เช่น **การแยกตัวประกอบ (factorization)**, discrete log
2. คอมธรรมดา (bit 0/1) ต้องใช้เวลานานมากในการถอด
3. **Quantum computer แก้ได้เร็วกว่ามาก** ด้วย **Shor's algorithm, Grover's algorithm**
4. จึงต้องมีวิธีเข้ารหัสใหม่

| | Traditional | PQC |
|---|---|---|
| ตัวอย่าง | **RSA, Elliptic Curve Cryptography** | **Lattice-based, Code-based** cryptography |
| ต่อคอมธรรมดา | ยาก | ยาก |
| ต่อ quantum | **ง่าย (ถูกถอดได้)** | **ยาก** |

> ⚠️ **QKD vs PQC:** QKD = ใช้ **ฟิสิกส์ (photon)** แลกกุญแจ ต้องมีอุปกรณ์/ช่องสัญญาณพิเศษ · PQC = ใช้ **คณิตศาสตร์แบบใหม่** ที่ quantum ก็แก้ยาก รันบนระบบเดิมได้

### 🔹 Quantum Computing Risk Example: VPN

- **VPN** = สื่อสารส่วนตัวอย่างปลอดภัยบนเครือข่ายสาธารณะ
- **IPsec ที่ใช้ IKE** มักใช้ **Diffie-Hellman (DH), RSA และ/หรือ EC** สำหรับ security association และ authentication
- วิธีเหล่านี้อาศัยความยากของ **discrete log** หรือ **หาตัวประกอบเฉพาะ** → **เสี่ยงต่อ Shor's algorithm**
- กราฟ: **GNFS** (General Number Field Sieve, วิธีที่ดีที่สุดของคอมธรรมดา) โตเร็วกว่า **Shor's algorithm (ln n)^3** มาก · **Shor เร็วกว่า GNFS เมื่อตัวเลขยาวเกิน ~60 หลัก**
- **RSA-2048 มี 617 หลัก (2048 bits)** → Shor ทำให้การแยกตัวประกอบเป็นไปได้มากกว่า GNFS
- ภาพประกอบ: ตัวอย่างรัน Shor บน IBM Quantum (Heron) แยกตัวประกอบ **N = 15 ได้ [3, 5]** · ระบุว่าแค่สาธิตบนฮาร์ดแวร์ NISQ ยัง **ไม่ได้ทำลาย PKI** จริง (RSA-2048 ต้องใช้ qubit ระดับล้านพร้อม error correction)

**Backup: RSA-250** = 250 หลัก (**829 bits**) ถูกแยกตัวประกอบเมื่อ **ก.พ. 2020** ด้วย **Number Field Sieve** (CADO-NFS) ใช้ประมาณ **2,700 CPU core-years** ← แสดงว่าคอมธรรมดาก็ยังใช้เวลามหาศาลแม้กับเลขเล็กกว่า RSA-2048

## 7. THE MONSTER ACTS ON THE MOONSHINE MODULE

**คืออะไร?** ตัวอย่าง **steganography สมัยใหม่โดย AI** (โพสต์ Oct 22, 2026): โมเดล **Fable-5.1** สร้างภาพธรรมดาที่ซ่อนข้อความลับ แล้วให้ Fable อีก session ที่ไม่รู้วิธีซ่อน หาข้อความเจอได้

| ข้อความที่ซ่อน | วิธีซ่อน |
|---|---|
| "THE MONSTER ACTS ON THE MOONSHINE MODULE" | **ความสูงของดาว** 40 ดวง ตาม "ไม้บรรทัดตัวอักษร" ที่มองไม่เห็น ทุก 5 ขั้นคือตัวถัดไป: 30 = space, 35 = A, 40 = B, ... |
| "the last train to Oxford leaves at 11:40 tonight" | **หน้าต่างตึก ติด/ดับ** อ่านแต่ละคอลัมน์เป็น **binary 7 bit** (ติด = 1, ดับ = 0) แล้วแปลง **ASCII** เช่น 1110100 = "t" |
| "the old lighthouse keeper left his logbook in the third drawer of the captain's desk" | **มุมเอียงของก้านทิวลิป**: 1° = A, 2° = B, ..., 26° = Z, 0° = space (84 ดอก: แดง 28, เหลือง 28, ชมพู 28) |

**Fable's 5 ways to hide messages:** **Heights · Rotations · Morse code · Colours · Lit/dark windows**

> ⭐ เชื่อมกับหัวข้อ 5: ทั้งหมดนี้คือ **steganography** (ซ่อนให้มองไม่เห็นว่ามีข้อความ) ไม่ใช่ cryptography · ตัวอย่างคำนวณ: 1110100₂ = 64+32+16+4 = **116** = ASCII **"t"**

## 8. Backup slides: Quantum เพิ่มเติม

### 🔹 Quantum States
- **Schrödinger's cat**: แมวในกล่องกับขวดยาพิษ ถ้า **ไม่สังเกต** = ทั้งตายและเป็นพร้อมกัน (superposition) · ถ้า **สังเกต** = ได้ผลเดียว ตายหรือเป็น
- **Quantum superposition** = อยู่หลายสถานะพร้อมกันจนกว่าจะถูกวัด "**เหรียญที่กำลังหมุนเป็นทั้งหัวและก้อยจนกว่าจะล้ม**"
- **Double slit experiment**: ไม่มีผู้สังเกต → เกิด **interference pattern** · มีผู้สังเกต/วัด → ได้แค่ **2 strips**
- (ภาพ Hydrogen wave function และ Bloch sphere |0⟩, |1⟩ ประกอบ)

### 🔹 Inside a Quantum Computer (Google Sycamore)
- ใช้ถังทรงกระบอกใหญ่รักษาอุณหภูมิต่ำ เพราะ qubit **อ่อนไหวมาก** พลังงานภายนอกรบกวนได้
- ด้านบนมีลิงก์สื่อสารและอุปกรณ์ปั๊ม **liquid helium**
- **216 coaxial cable channels** สายราคา ~**$1,000 ต่อ 60 cm**
- ชิป Sycamore 2 ส่วน: controller interface + processor ที่มี qubit
- **Cryogenic cooling 4 ขั้น:** A ทำความเย็นแบบตู้เย็นปกติ → B evaporative cooling ด้วย **liquid helium** → C **dilution refrigeration** (ผสม Helium-3 และ -4) → D qubit อยู่ที่ **~20 milli-Kelvin (−273.13°C)**
- 0 K = −273.15°C · อวกาศเฉลี่ย 2.7 K (−270.45°C) → **qubit เย็นกว่าอวกาศ**

### 🔹 Why Quantum Computers need extreme cooling? (3 เหตุผล)
1. **Minimizing Thermal Energy** – ร้อน = อนุภาคสั่น = noise ทำ quantum state ผิดพลาด
2. **Superconductivity** – qubit แบบ superconducting (Google, SpinQ) ต้องเย็นจัดจึงนำไฟฟ้าแบบ **ไม่มีความต้านทาน** รักษา state โดยไม่เสียพลังงาน
3. **Reducing Background Radiation and Noise** – ลดการรบกวนจากคลื่นแม่เหล็กไฟฟ้ารอบข้าง

### 🔹 Different Types of Quantum Computing Technologies
แผนภาพวัด **Maturity (TRL)** กับ **Intensity (R&D/investment)**:
- **Superconducting** (electron-based) = **maturity สูงสุด (TRL 3–4) และลงทุนมากสุด**
- กลุ่มกลาง (TRL 2–3): **Photon** (flying), **Electron spin**, **Cold atom**, **Trapped ion** (atom-based)
- กลุ่มต่ำ (TRL 1–2): **Topological**, **NV centers** (nitrogen vacancy)

### 🔹 Four key principles of quantum mechanics
| หลัก | ความหมาย |
|---|---|
| **Superposition** | qubit อยู่ในสถานะผสมของทุกความเป็นไปได้ · **เมื่อวัด จะ collapse เป็น 0 หรือ 1** |
| **Entanglement** | qubit ผูกสถานะกัน **วัดตัวหนึ่ง รู้ข้อมูลอีกตัวทันที** |
| **Interference** | "เครื่องยนต์" ของ quantum computing: amplitude เหมือนคลื่น **เสริมกัน** (เพิ่มโอกาสคำตอบที่ถูก) หรือ **หักล้างกัน** |
| **Decoherence** | ระบบ quantum **หลุด** เป็นสถานะธรรมดา เกิดจากการวัดหรือสิ่งแวดล้อม · ต้อง **ลดให้น้อยที่สุด** |

## จุดที่มักสับสน

| เรื่อง | A | B | จำง่ายๆ |
|---|---|---|---|
| NSA vs SA | **NSA**: 5G RAN + **EPC (core 4G)** | **SA**: 5G RAN + **5GC ใหม่** | Non-Standalone = ยังพึ่ง 4G |
| NEA vs NIA | **NEA** = Encryption → confidentiality → กัน **eavesdropping** | **NIA** = Integrity → integrity → กัน **tampering** | E = Encrypt, I = Integrity |
| Non-regenerative vs Regenerative | **Transparent / bent-pipe**: ส่งผ่านอย่างเดียว RAN อยู่ภาคพื้น | **On-board processor**: RAN อยู่บนดาวเทียม | regenerate = สร้างใหม่ = มีสมองบนฟ้า |
| Steganography vs Cryptography | **ซ่อน** (concealment) มองไม่เห็นว่ามีข้อความ | **เข้ารหัส** (encoding) เห็นแต่อ่านไม่ออก | โกนหัวทาส vs Enigma |
| QKD vs PQC | ใช้ **photon/ฟิสิกส์** แลกกุญแจ, ดักฟังแล้วรู้ตัว | ใช้ **คณิตศาสตร์ใหม่** (lattice, code-based) | Q-K-D = Key แจกด้วยแสง |
| GEO / MEO / LEO | GEO 35,785.5 km | MEO 2,000–20,350 · LEO 300–2,000 | L = Low = ใกล้สุด |

## จุดเน้นสำหรับสอบ
- 🔥 **1G→6G**: analog → digital voice → data → LTE → 5G NR (unified platform) → AI-native · **ทุก ~10 ปี**
- 🔥 **NSA vs SA** (core ใช้ EPC ร่วม vs 5G core ใหม่)
- 🔥 **NEA / NIA**: กันอะไร, 0/1/2/3 = null/SNOW 3G/AES/ZUC, key 128, MAC 32, integrity UP เป็น optional
- 🔥 **Steganography vs Cryptography** + ตัวอย่าง (โกนหัวทาส 499 BCE, Enigma ≈ 10^23, "CILLY")
- 🔥 **QKD** (photon, วัดแล้วเปลี่ยน, quantum channel + classical channel) vs **PQC** (lattice/code-based แทน RSA/ECC)
- ⭐ GEO 35,785.5 km · MEO 2,000–20,350 · LEO 300–2,000 · NTN = 5G + ดาวเทียม (3GPP)
- ⭐ Ambient IoT = Rel-19, ~1 µW · 5G Advanced = Rel-18+ · 6G = Rel-21+
- ⭐ 5 security layers: network access / network domain / user domain / application domain / **SBA domain**
- ⭐ R15: UP+CP มีทั้ง encryption & integrity, permanent user ID เข้ารหัสตลอด · R16: 256-bit keys
- ⭐ Shor's algorithm → เสี่ยงต่อ RSA/DH/ECC (VPN/IPsec/IKE) · Shor ชนะ GNFS เมื่อ > ~60 หลัก · RSA-2048 = 617 หลัก
- ⭐ qubit: Sycamore 54, Willow 105, Zuchongzhi 3.0 105, Fujitsu 256 · 4 หลัก: superposition, entanglement, interference, decoherence
- 📌 D2D 3 ข้อพิจารณา: frequency spectrum, transport network, indoor coverage
- 📌 Fable's 5 ways: heights, rotations, Morse code, colours, lit/dark windows

## ✅ Coverage Check
- [x] Agenda 7 หัวข้อ + backup slides
- [x] 1G–6G timeline (ทุกช่อง) + Release roadmap Rel-15 ถึง Rel-21+, WRC
- [x] Evolution features (เฉพาะจุดที่ไฮไลต์ + ฟีเจอร์อื่นหลัก), Ambient IoT (power, RAN1–4, design)
- [x] Satellite orbits, HAPS/plane/UAV, feeder/service/backhaul, NTN 3 ข้อ, NTN options 2 แบบ
- [x] NSA/SA, Starlink D2D + 3 considerations
- [x] Security layers 5 ด้าน, architecture strata, system-wide coverage, threats table, R15/R16 enhancements, NEA/NIA tables
- [x] Steganography/Cryptography, 499 BCE, Enigma
- [x] Quantum computing, bits vs qubits table, IBM/Google/China/SpinQ/Fujitsu, IBM roadmap, QKD, PQC, VPN risk
- [x] Hidden message examples 3 แบบ + 5 ways
- [x] Backup: quantum states, inside QC, cooling 3 เหตุผล, technology types, 4 principles, RSA-250
- ⚠️ สไลด์ Evolution features ภาพเล็ก อ่านได้ไม่ครบทุกกล่อง สรุปเฉพาะที่อ่านชัดและจุดที่ไฮไลต์
