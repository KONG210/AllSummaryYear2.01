# Quiz 2 · DFD Understanding

> 📌 เรื่องที่ 1 จาก 4 ของ Quiz 2 SE Lab · กลับไป [ภาพรวม Quiz 2](#sel-q2) · เรื่องถัดไป [Error detection](#sel-q2b)

> ⚠️ ใช้กฎมาตรฐานตามตำรา (สัญลักษณ์ DeMarco & Yourdon) เพราะยังไม่มีไฟล์แล็บ DFD ของวิชา

## 1. DFD คืออะไร · สัญลักษณ์ · ระดับ

**DFD (Data Flow Diagram)** = แผนภาพที่แสดงว่า **ข้อมูลไหลจากไหน ผ่านการประมวลผลอะไร และถูกเก็บที่ไหน** · ไม่แสดงลำดับเวลา ไม่แสดง if/loop (ต่างจาก flowchart)

| สัญลักษณ์ | DeMarco & Yourdon | Gane & Sarson | ความหมาย | ตั้งชื่อ |
|---|---|---|---|---|
| **Process** | วงกลม | สี่เหลี่ยมมุมมน มีเลขด้านบน | แปลงข้อมูลเข้าเป็นข้อมูลออก | **คำกริยา + กรรม** เช่น "Calculate Bill" มีเลข 1.0, 2.0 |
| **External Entity** | สี่เหลี่ยม | สี่เหลี่ยม (เงา) | คน/ระบบภายนอกที่ส่งหรือรับข้อมูล | **คำนาม** เช่น Customer |
| **Data Store** | เส้นคู่ขนาน / กล่องเปิดด้านขวา | กล่องเปิดด้านขวา มีรหัส D1 | ที่เก็บข้อมูล (ไฟล์, ตาราง) | **คำนาม** เช่น D1 Orders |
| **Data Flow** | ลูกศร | ลูกศร | ข้อมูลที่เคลื่อน | **คำนาม** บอกว่าข้อมูลอะไร เช่น "order details" |

**ระดับของ DFD**
| ระดับ | หน้าตา |
|---|---|
| **Context diagram** | process **เดียว** (เลข 0) = ทั้งระบบ + external entities + flow เข้า/ออก · **ไม่มี data store** |
| **Level 0** | แตก process 0 เป็น process หลัก 1.0, 2.0, 3.0 … + data stores |
| **Level 1** | แตก process หนึ่ง เช่น 2.0 เป็น 2.1, 2.2, 2.3 … |

> ⭐ **Balancing:** flow เข้า/ออกของ process แม่ ต้อง**ตรงกับ** flow เข้า/ออกรวมของแผนภาพลูก (ไม่มี flow หรือ entity ใหม่โผล่มา หรือหายไป)

## 2. วิธีอ่าน DFD ทีละขั้น

1. **ดูระดับก่อน:** process เดียวเลข 0 และไม่มี data store = context · มี 1.0, 2.0 = level 0 · มี 2.1, 2.2 = level 1 ของ process 2.0
2. **นับของ 4 ชนิด:** external entity (สี่เหลี่ยม), process (วงกลม), data store (กล่องเปิด), data flow (ลูกศร)
3. **ไล่ทีละ process:** เขียนว่า input มีอะไร มาจากไหน → output มีอะไร ไปไหน
4. **ไล่ทีละ data store:** process ไหน **เขียน** (ลูกศรเข้า store) process ไหน **อ่าน** (ลูกศรออกจาก store)
5. **ตามเส้นทางข้อมูล:** เลือก flow หนึ่งจาก entity แล้วตามไปจนออกจากระบบ จะเข้าใจว่าระบบทำงานอย่างไร

> ⭐ ทิศลูกศรสำคัญที่สุด: **ลูกศรเข้า data store = เขียน/บันทึก** · **ลูกศรออกจาก data store = อ่าน/ดึงข้อมูล**

## 3. แบบฝึก A1: อ่าน DFD (Library System, Level 0)

<div class="dfdwrap"><svg class="dfd" viewBox="0 0 760 440" role="img" aria-label="Library DFD level 0" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="currentColor"/></marker></defs><line x1="100.7" y1="217.0" x2="203.3" y2="120.2" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="56.8" y="158.6" width="102.4" height="18" rx="4" class="lbg"/><text x="108.0" y="172.1" text-anchor="middle" class="fl">member details</text><line x1="233.0" y1="134.0" x2="132.6" y2="227.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="187.5" y="170.5" width="82.6" height="18" rx="4" class="lbg"/><text x="228.8" y="184.0" text-anchor="middle" class="fl">member card</text><line x1="295.8" y1="86.0" x2="410.0" y2="76.1" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="314.9" y="71.1" width="76.0" height="18" rx="4" class="lbg"/><text x="352.9" y="84.6" text-anchor="middle" class="fl">new member</text><line x1="470.5" y1="88.0" x2="401.4" y2="219.3" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="432.1" y="143.6" width="95.8" height="18" rx="4" class="lbg"/><text x="480.0" y="157.1" text-anchor="middle" class="fl">member status</text><line x1="150.0" y1="244.1" x2="334.1" y2="256.8" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="190.9" y="240.5" width="102.4" height="18" rx="4" class="lbg"/><text x="242.1" y="254.0" text-anchor="middle" class="fl">borrow request</text><line x1="205.7" y1="382.0" x2="342.3" y2="286.4" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="209.3" y="324.2" width="69.4" height="18" rx="4" class="lbg"/><text x="244.0" y="337.7" text-anchor="middle" class="fl">book info</text><line x1="412.5" y1="292.5" x2="502.0" y2="382.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="456.0" y="327.3" width="82.6" height="18" rx="4" class="lbg"/><text x="497.3" y="340.8" text-anchor="middle" class="fl">loan record</text><line x1="640.4" y1="133.0" x2="620.1" y2="244.2" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="532.3" y="178.6" width="95.8" height="18" rx="4" class="lbg"/><text x="580.2" y="192.1" text-anchor="middle" class="fl">returned book</text><line x1="654.9" y1="245.3" x2="672.0" y2="133.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="662.1" y="179.2" width="82.6" height="18" rx="4" class="lbg"/><text x="703.4" y="192.7" text-anchor="middle" class="fl">fine notice</text><line x1="538.0" y1="382.0" x2="597.5" y2="322.5" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="563.0" y="342.3" width="69.4" height="18" rx="4" class="lbg"/><text x="597.7" y="355.8" text-anchor="middle" class="fl">loan data</text><rect x="30.0" y="217.0" width="120" height="46" class="en"/><text x="90" y="245" text-anchor="middle" class="nl b">Member</text><rect x="605.0" y="87.0" width="110" height="46" class="en"/><text x="660" y="115" text-anchor="middle" class="nl b">Librarian</text><circle cx="250" cy="90" r="46" class="pr"/><text x="250" y="80" text-anchor="middle" class="pn">1.0</text><text x="250" y="97" text-anchor="middle" class="nl">Register</text><text x="250" y="112" text-anchor="middle" class="nl">Member</text><rect x="410.0" y="52.0" width="140" height="36" class="dsbg"/><path d="M550.0,52.0 H410.0 V88.0 H550.0" class="ds"/><line x1="444.0" y1="52.0" x2="444.0" y2="88.0" class="ds"/><text x="427.0" y="75" text-anchor="middle" class="pn">D1</text><text x="497.0" y="75" text-anchor="middle" class="nl">Members</text><circle cx="380" cy="260" r="46" class="pr"/><text x="380" y="250" text-anchor="middle" class="pn">2.0</text><text x="380" y="267" text-anchor="middle" class="nl">Borrow</text><text x="380" y="282" text-anchor="middle" class="nl">Book</text><circle cx="630" cy="290" r="46" class="pr"/><text x="630" y="280" text-anchor="middle" class="pn">3.0</text><text x="630" y="297" text-anchor="middle" class="nl">Return</text><text x="630" y="312" text-anchor="middle" class="nl">Book</text><rect x="110.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M250.0,382.0 H110.0 V418.0 H250.0" class="ds"/><line x1="144.0" y1="382.0" x2="144.0" y2="418.0" class="ds"/><text x="127.0" y="405" text-anchor="middle" class="pn">D2</text><text x="197.0" y="405" text-anchor="middle" class="nl">Books</text><rect x="450.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M590.0,382.0 H450.0 V418.0 H590.0" class="ds"/><line x1="484.0" y1="382.0" x2="484.0" y2="418.0" class="ds"/><text x="467.0" y="405" text-anchor="middle" class="pn">D3</text><text x="537.0" y="405" text-anchor="middle" class="nl">Loans</text></svg></div>

ตอบให้ได้ก่อนเปิดเฉลย:
1. แผนภาพนี้เป็น context diagram หรือ level 0? รู้ได้อย่างไร
2. มี external entity, process และ data store อย่างละกี่ตัว อะไรบ้าง
3. process ใด**เขียน**ข้อมูลลง D1 Members และ process ใด**อ่าน**จาก D1
4. process 2.0 Borrow Book ใช้ข้อมูลจากที่ไหนบ้าง และส่งผลลัพธ์ไปที่ไหน
5. "member card" ไหลจากไหนไปไหน
6. ถ้าบรรณารักษ์ต้องการ**ปิดรายการยืม**เมื่อคืนหนังสือ แผนภาพนี้ขาด flow อะไร

<details><summary>เฉลยแบบฝึก A1</summary>
<ol>
<li><b>Level 0</b> เพราะมีหลาย process ที่มีเลข 1.0, 2.0, 3.0 และมี data store (context diagram มี process เดียวและไม่มี data store)</li>
<li>External entity <b>2</b>: Member, Librarian · Process <b>3</b>: 1.0 Register Member, 2.0 Borrow Book, 3.0 Return Book · Data store <b>3</b>: D1 Members, D2 Books, D3 Loans</li>
<li>เขียน D1: <b>1.0 Register Member</b> (new member) · อ่าน D1: <b>2.0 Borrow Book</b> (member status)</li>
<li>เข้า: <b>borrow request</b> จาก Member, <b>member status</b> จาก D1, <b>book info</b> จาก D2 · ออก: <b>loan record</b> ลง D3 Loans</li>
<li>จาก <b>1.0 Register Member</b> ไปหา <b>Member</b></li>
<li>3.0 Return Book <b>อ่าน</b> D3 อย่างเดียว ไม่มี flow <b>เขียนกลับ</b>ไปที่ D3 (เช่น "returned loan / loan status") จึงอัปเดตรายการยืมไม่ได้ · ถ้าโจทย์บอกว่าต้องปิดรายการ แผนภาพนี้ไม่ครบ</li>
</ol></details>

## 4. แบบฝึก A2: อ่าน Level 1 (แตก 2.0 Grade Exam)

**Process แม่ (ใน Level 0):** `2.0 Grade Exam` รับ **answer sheet** จาก Student และ **answer key** จาก D2 Answer Keys · ส่ง **score** ลง D3 Scores และ **result notice** ให้ Student

<div class="dfdwrap"><svg class="dfd" viewBox="0 0 760 440" role="img" aria-label="Level 1 of 2.0 Grade Exam" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="currentColor"/></marker></defs><line x1="355.0" y1="68.0" x2="223.8" y2="188.8" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="244.8" y="118.4" width="89.2" height="18" rx="4" class="lbg"/><text x="289.4" y="131.9" text-anchor="middle" class="fl">answer sheet</text><line x1="154.0" y1="382.0" x2="180.0" y2="264.9" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="89.0" y="313.5" width="76.0" height="18" rx="4" class="lbg"/><text x="127.0" y="327.0" text-anchor="middle" class="fl">answer key</text><line x1="235.0" y1="229.8" x2="375.0" y2="260.2" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="253.8" y="235.0" width="102.4" height="18" rx="4" class="lbg"/><text x="305.0" y="248.5" text-anchor="middle" class="fl">marked answers</text><line x1="453.7" y1="301.3" x2="540.6" y2="382.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="445.7" y="331.7" width="43.0" height="18" rx="4" class="lbg"/><text x="467.2" y="345.2" text-anchor="middle" class="fl">score</text><line x1="461.9" y1="251.0" x2="598.1" y2="189.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="508.5" y="210.0" width="43.0" height="18" rx="4" class="lbg"/><text x="530.0" y="223.5" text-anchor="middle" class="fl">score</text><line x1="598.5" y1="150.1" x2="427.8" y2="68.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="465.3" y="99.0" width="95.8" height="18" rx="4" class="lbg"/><text x="513.2" y="112.5" text-anchor="middle" class="fl">result notice</text><rect x="320.0" y="22.0" width="120" height="46" class="en"/><text x="380" y="50" text-anchor="middle" class="nl b">Student</text><circle cx="190" cy="220" r="46" class="pr"/><text x="190" y="210" text-anchor="middle" class="pn">2.1</text><text x="190" y="227" text-anchor="middle" class="nl">Check</text><text x="190" y="242" text-anchor="middle" class="nl">Answers</text><circle cx="420" cy="270" r="46" class="pr"/><text x="420" y="260" text-anchor="middle" class="pn">2.2</text><text x="420" y="277" text-anchor="middle" class="nl">Calculate</text><text x="420" y="292" text-anchor="middle" class="nl">Score</text><circle cx="640" cy="170" r="46" class="pr"/><text x="640" y="160" text-anchor="middle" class="pn">2.3</text><text x="640" y="177" text-anchor="middle" class="nl">Notify</text><text x="640" y="192" text-anchor="middle" class="nl">Student</text><rect x="70.0" y="382.0" width="160" height="36" class="dsbg"/><path d="M230.0,382.0 H70.0 V418.0 H230.0" class="ds"/><line x1="104.0" y1="382.0" x2="104.0" y2="418.0" class="ds"/><text x="87.0" y="405" text-anchor="middle" class="pn">D2</text><text x="167.0" y="405" text-anchor="middle" class="nl">Answer Keys</text><rect x="490.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M630.0,382.0 H490.0 V418.0 H630.0" class="ds"/><line x1="524.0" y1="382.0" x2="524.0" y2="418.0" class="ds"/><text x="507.0" y="405" text-anchor="middle" class="pn">D3</text><text x="577.0" y="405" text-anchor="middle" class="nl">Scores</text></svg></div>

1. ทำไม process ในแผนภาพนี้ถึงมีเลข 2.1, 2.2, 2.3
2. process ใดรับ answer sheet และ process ใดเขียนลง D3 Scores
3. flow "marked answers" ไม่มีใน process แม่ ทำให้ไม่ balance หรือไม่ เพราะอะไร
4. แผนภาพนี้ balance กับ process แม่หรือไม่ ให้เทียบทีละ flow
5. ถ้าเพิ่ม flow "rank" จาก 2.2 ไปหา **Teacher** (entity ใหม่) จะเกิดอะไรขึ้น

<details><summary>เฉลยแบบฝึก A2</summary>
<ol>
<li>เพราะเป็น <b>Level 1 ของ process 2.0</b> การแตก process 2.0 ใช้เลข 2.x</li>
<li>รับ answer sheet: <b>2.1 Check Answers</b> · เขียน D3: <b>2.2 Calculate Score</b></li>
<li><b>ไม่ทำให้ผิด</b> เพราะ "marked answers" เป็น flow <b>ภายใน</b>ระหว่าง process ลูก (2.1 → 2.2) ไม่ได้เข้าหรือออกจาก process แม่</li>
<li><b>Balance</b>: เข้า answer sheet ✓ (→2.1), answer key ✓ (D2→2.1) · ออก score ✓ (2.2→D3), result notice ✓ (2.3→Student) ครบ ไม่มีของเกินหรือขาด</li>
<li><b>ไม่ balance</b> เพราะมี entity และ output ใหม่ที่ process แม่ไม่มี ต้องเพิ่ม flow นั้นใน Level 0 ด้วย</li>
</ol></details>

## 5. แบบฝึก A3: แยกองค์ประกอบจากโจทย์

> **โจทย์:** ร้านกาแฟออนไลน์ — **ลูกค้า**สั่งเครื่องดื่มผ่านแอป ระบบ**ตรวจเมนู**จากรายการเมนูที่เก็บไว้ แล้ว**บันทึกคำสั่งซื้อ** ส่ง**รายการที่ต้องทำ**ให้**บาริสต้า** เมื่อทำเสร็จบาริสต้ากดยืนยัน ระบบ**ส่งแจ้งเตือน**ให้ลูกค้ามารับ

แยกให้ได้: external entity, process (ตั้งชื่อแบบกริยา), data store, data flow หลัก แล้วลองวาด Level 0 บนกระดาษ

<details><summary>เฉลยแบบฝึก A3 (ตัวอย่างคำตอบ)</summary>
<ul>
<li><b>External entity:</b> Customer, Barista</li>
<li><b>Process:</b> 1.0 Take Order · 2.0 Send Drink Queue · 3.0 Notify Customer</li>
<li><b>Data store:</b> D1 Menu · D2 Orders</li>
<li><b>Data flow:</b> Customer → 1.0 "drink order" · D1 → 1.0 "menu items" · 1.0 → D2 "order record" · D2 → 2.0 "pending orders" · 2.0 → Barista "drink queue" · Barista → 3.0 "done confirmation" · 3.0 → Customer "pickup notice" (3.0 อาจอ่าน D2 เพื่อรู้ว่าแจ้งใคร)</li>
<li>ตรวจตัวเอง: ทุก process มีทั้งเข้าและออก · ไม่มี entity คุยกันตรงๆ · ทุก flow มีชื่อ</li>
</ul></details>

## จุดเน้นสำหรับสอบ
- 🔥 สัญลักษณ์ 4 ตัว + การตั้งชื่อ (process = กริยา + กรรม, ที่เหลือ = คำนาม)
- 🔥 context (process 0 ตัวเดียว ไม่มี data store) · level 0 (1.0, 2.0) · level 1 (2.1, 2.2)
- 🔥 ลูกศรเข้า store = เขียน · ออกจาก store = อ่าน
- ⭐ flow ภายในระหว่าง process ลูกไม่ต้องอยู่ในแม่ · balancing ดูเฉพาะ flow ที่ข้ามขอบ
- ⭐ DFD ไม่แสดงลำดับเวลา / เงื่อนไข (ต่างจาก flowchart)
