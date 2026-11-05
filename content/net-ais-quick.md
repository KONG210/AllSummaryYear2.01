# AIS 5G · สรุปก่อนสอบ

> 📌 บรรยายพิเศษจากบริษัทเอกชน · Final มีข้อจากบริษัทละ 3 ข้อ

## 📡 1G → 6G (ทุก ~10 ปี)
| 1G 1980s | 2G 1990s | 3G 2000s | 4G 2010s | 5G 2020s | 6G 2030s |
|---|---|---|---|---|---|
| analog voice | digital voice | wireless internet (data) | mobile broadband (LTE) | unified connectivity platform (5G NR) | AI-native, RF sensing |

- 3GPP Release: **5G = Rel-15–17** · **5G Advanced = Rel-18–20** · **6G = Rel-21+** (Rel-20 = study)
- Rel-19 "focus today": **Wireless AI, Ambient IoT, ISAC** · Rel-17: NTN · Rel-16: V2X · Rel-15: eMBB
- **Ambient IoT ~1 µW** (5G IoT 200 mW > Wi-Fi 50 mW > BLE 1 mW > Ambient ~1 µW)

## 🛰️ ดาวเทียม / NTN
- **GEO 35,785.5 km** · **MEO 2,000–20,350 km** · **LEO 300–2,000 km**
- NTN = 5G + ดาวเทียม (3GPP): พื้นที่ห่างไกล, broadband ทั่วโลก, ภัยพิบัติ, IoT
- **Non-regenerative = transparent/bent-pipe** (RAN ภาคพื้น) · **Regenerative = on-board processor** (RAN บนดาวเทียม)
- Starlink D2D ใช้มือถือ LTE ธรรมดา · ข้อพิจารณา: **spectrum, transport network, indoor coverage**

## 🏗️ Architecture
- **NSA** = 5G RAN + LTE + **EPC (core 4G)** · **SA** = 5G RAN + **5G core ใหม่**

## 🔐 Security
- 5 layers: **network access, network domain, user domain, application domain, SBA domain** (TS 33.501)
- **Eavesdropping → confidentiality → NEA** · **Tampering → integrity → NIA** (UP integrity = optional)
- **0 = Null · 1 = SNOW 3G (ETSI SAGE) · 2 = AES (NIST; NEA CTR / NIA CMAC) · 3 = ZUC (จีน)** · key 128 · MAC 32
- R15: UP+CP มี encryption & integrity, **permanent user ID เข้ารหัสตลอด** · R16: **256-bit keys**

## 🕵️ Stegano vs Crypto
- **Steganography = ซ่อน (concealment)** · 499 BCE โกนหัวทาส (Histiaeus)
- **Cryptography = เข้ารหัส (encoding → cipher text)** · Enigma ≈ **10^23**, Bombe, คีย์ "CILLY"
- Fable 5 ways: heights, rotations, Morse code, colours, lit/dark windows (windows = 7-bit binary → ASCII)

## ⚛️ Quantum
- bit 0/1 vs **qubit** หลายสถานะพร้อมกัน · 4 หลัก: **superposition, entanglement, interference, decoherence**
- Sycamore **54** (2019) · Willow **105** · Zuchongzhi 3.0 **105** · Fujitsu+RIKEN **256** · Google 5 นาที vs 10^25 ปี
- **QKD**: photon (polarization/phase), วัด = เปลี่ยน state, quantum channel (key) + classical channel (auth)
- **PQC**: แทน RSA/ECC ด้วย **lattice-based, code-based** · ภัยจาก **Shor's / Grover's algorithm**
- VPN (IPsec/IKE ใช้ DH, RSA, EC) เสี่ยง Shor · Shor ชนะ GNFS เมื่อ > ~60 หลัก · RSA-2048 = 617 หลัก
- ทำไมต้องเย็น (~20 mK): thermal noise, superconductivity, background radiation
