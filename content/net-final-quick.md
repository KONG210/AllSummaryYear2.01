# Final Network · สรุปก่อนสอบ

## 📌 รูปแบบ
- ปรนัย **50 ข้อ** = เนื้อหา 44 + บริษัทเอกชน 6 (บริษัทละ 3)
- ข้อเขียน **1 ข้อ**: white-box / ออกแบบ IPv6 / ออกแบบ Routing Protocol

## 🧠 โครงตอบข้อเขียน
**เป้าหมาย → diagram → ส่วนประกอบ → ขั้นตอนการทำงาน → ตัวอย่างตาราง → ข้อดี/ข้อเสีย + เหตุผลที่เลือก**

## ⭐ White-box
- commodity hardware แยกจาก software, open API, ไม่ติด vendor (เทียบ mainframe → PC)
- 3 ชั้น: **apps** ↔ northbound (REST/intent) ↔ **SDN controller** ↔ southbound (**OpenFlow/TCP**) ↔ **white-box switches**
- flow table: **match + action + priority + counters** · action: forward / drop / modify / send to controller
- flow ใหม่: packet-in → คำนวณ → **modify-state** + packet-out · link เสีย: **port status** → Dijkstra app → ติดตั้งตารางใหม่
- ข้อดี: ถูก, traffic engineering, ลด misconfig · ข้อเสีย: ต้อง harden controller, flow แรกช้า, ข้าม AS ยาก

## ⭐ IPv6
- address **128 bit**, header **คงที่ 40 bytes**
- fields: ver, pri, **flow label**, payload len, **next hdr**, **hop limit**, src, dst
- ตัด **checksum** (ไม่ต้องคำนวณใหม่ทุก hop), **fragmentation** (ทิ้ง + ICMPv6 Packet Too Big), **options** (→ extension header)
- **tunneling**: IPv6 อยู่ใน payload ของ IPv4 (src/dst = ปลายอุโมงค์) · หรือ dual-stack
- ย่อ address: ตัด 0 นำหน้า, `::` ได้ครั้งเดียว · ไม่มี broadcast

## ⭐ Routing
- **Intra-AS = OSPF** (link-state, Dijkstra, flood ใน area, authenticate) · แบ่ง **Area 0 backbone** + area ย่อย, ABR summarize
- **Inter-AS = BGP** (path vector, TCP) · eBGP กับ ISP, iBGP ภายใน · policy: **ไม่เป็น transit**, local pref
- เลือกเส้นทาง: **local pref → AS-PATH สั้นสุด → hot potato → อื่นๆ**
- forwarding entry = BGP (ออก gateway ไหน) + OSPF (ใช้ interface ไหน)
- Dijkstra: `D(n) = min(D(n), D(w) + c(w,n))` · Bellman-Ford: `dx(y) = minᵥ{c(x,v) + dᵥ(y)}`
- LS: รู้ทั้ง topology, converge เร็ว · DV: รู้แค่เพื่อนบ้าน, count-to-infinity

## ⚠️ อย่าลืม
- เขียนนิยามก่อนเสมอ (white-box คืออะไร, ทำไมต้องมี IPv6)
- ใส่ตัวอย่างที่มีค่าจริง (flow table, header, ตาราง Dijkstra)
- 6 ข้อบริษัทเอกชน: ทวนโน้ตจากตอนบรรยาย
