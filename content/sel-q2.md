# Quiz 2 · DFD + Git (+ Unit test)

## สอบอะไร

> 📌 **Quiz 2 (20%)** · **Closed-book** ห้ามนำ lecture note หรือ cheat sheet เข้า · เตรียมเครื่องเขียน · **Section 1: 11 พ.ย. 2569 เวลา 11.15–12.45** · **Section 2: 10 พ.ย. 2569 เวลา 10.45–12.15**

| หัวข้อ | ในบทนี้ |
|---|---|
| **DFD Understanding** | ข้อ 1–2 (กฎ + อ่านแผนภาพ) |
| **Error detection in DFD** | ข้อ 3–4 (หาจุดผิด + balancing) |
| **Git functions** | ข้อ 5–6 (คำสั่ง + โจทย์สถานการณ์) · ทวนเต็มที่ [Lab 9 Git](#sel-9) |
| **Unit test** | ข้อ 7 · ⚠️ **ยังรอไฟล์** · ระหว่างนี้ทวน [Lab 10 Testing](#sel-10) |

> ⚠️ ไฟล์แล็บ DFD ของวิชานี้ยังไม่มีในเว็บ ส่วน DFD ด้านล่างใช้**กฎมาตรฐานตามตำรา** (สัญลักษณ์แบบ DeMarco & Yourdon) ถ้าแล็บใช้สัญลักษณ์อื่นหรือมีกฎเพิ่ม ส่งไฟล์มาจะปรับให้ตรง

## 1. DFD พื้นฐาน (ต้องรู้ก่อนทำโจทย์)

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

**กฎที่ใช้จับผิด (สำคัญที่สุดสำหรับข้อสอบ)**
| ข้อผิดพลาด | อาการ | ทำไมผิด |
|---|---|---|
| **Black hole** | process มี **input แต่ไม่มี output** | ข้อมูลเข้าไปแล้วหายไปเฉยๆ |
| **Miracle** (spontaneous generation) | process มี **output แต่ไม่มี input** | สร้างข้อมูลจากความว่างเปล่า |
| **Gray hole** | input **ไม่พอ** จะสร้าง output นั้นได้ | เช่นจะคำนวณเกรดแต่ไม่มีคะแนนเข้ามา |
| **Entity → Entity** | flow ระหว่าง external entity สองตัวตรงๆ | อยู่นอกระบบ ระบบไม่เกี่ยว → ไม่วาด |
| **Entity ↔ Data store** | entity อ่าน/เขียน data store ตรงๆ | ต้องผ่าน process เสมอ |
| **Data store → Data store** | ย้ายข้อมูลระหว่าง store ตรงๆ | ต้องมี process ทำการย้าย |
| **Flow ไม่มีชื่อ** | ลูกศรไม่มี label | ไม่รู้ว่าข้อมูลอะไร |
| **ชื่อ process เป็นคำนาม** | เช่น "Payment" | process ต้องเป็นการกระทำ เช่น "Process Payment" |
| **Unbalanced** | ระดับลูกมี flow/entity ที่แม่ไม่มี (หรือกลับกัน) | ผิด balancing |

> ⭐ จำง่าย: **ทุก flow ต้องมี process อยู่ปลายใดปลายหนึ่งเสมอ** · ทุก process ต้องมีทั้ง **เข้าและออก**

## 2. แบบฝึก A: อ่าน DFD (Library System, Level 0)

<div class="dfdwrap"><svg class="dfd" viewBox="0 0 760 440" role="img" aria-label="Library DFD level 0" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="currentColor"/></marker></defs><line x1="100.7" y1="217.0" x2="203.3" y2="120.2" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="56.8" y="158.6" width="102.4" height="18" rx="4" class="lbg"/><text x="108.0" y="172.1" text-anchor="middle" class="fl">member details</text><line x1="233.0" y1="134.0" x2="132.6" y2="227.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="187.5" y="170.5" width="82.6" height="18" rx="4" class="lbg"/><text x="228.8" y="184.0" text-anchor="middle" class="fl">member card</text><line x1="295.8" y1="86.0" x2="410.0" y2="76.1" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="314.9" y="71.1" width="76.0" height="18" rx="4" class="lbg"/><text x="352.9" y="84.6" text-anchor="middle" class="fl">new member</text><line x1="470.5" y1="88.0" x2="401.4" y2="219.3" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="432.1" y="143.6" width="95.8" height="18" rx="4" class="lbg"/><text x="480.0" y="157.1" text-anchor="middle" class="fl">member status</text><line x1="150.0" y1="244.1" x2="334.1" y2="256.8" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="190.9" y="240.5" width="102.4" height="18" rx="4" class="lbg"/><text x="242.1" y="254.0" text-anchor="middle" class="fl">borrow request</text><line x1="205.7" y1="382.0" x2="342.3" y2="286.4" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="209.3" y="324.2" width="69.4" height="18" rx="4" class="lbg"/><text x="244.0" y="337.7" text-anchor="middle" class="fl">book info</text><line x1="412.5" y1="292.5" x2="502.0" y2="382.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="456.0" y="327.3" width="82.6" height="18" rx="4" class="lbg"/><text x="497.3" y="340.8" text-anchor="middle" class="fl">loan record</text><line x1="640.4" y1="133.0" x2="620.1" y2="244.2" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="532.3" y="178.6" width="95.8" height="18" rx="4" class="lbg"/><text x="580.2" y="192.1" text-anchor="middle" class="fl">returned book</text><line x1="654.9" y1="245.3" x2="672.0" y2="133.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="662.1" y="179.2" width="82.6" height="18" rx="4" class="lbg"/><text x="703.4" y="192.7" text-anchor="middle" class="fl">fine notice</text><line x1="538.0" y1="382.0" x2="597.5" y2="322.5" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="563.0" y="342.3" width="69.4" height="18" rx="4" class="lbg"/><text x="597.7" y="355.8" text-anchor="middle" class="fl">loan data</text><rect x="30.0" y="217.0" width="120" height="46" class="en"/><text x="90" y="245" text-anchor="middle" class="nl b">Member</text><rect x="605.0" y="87.0" width="110" height="46" class="en"/><text x="660" y="115" text-anchor="middle" class="nl b">Librarian</text><circle cx="250" cy="90" r="46" class="pr"/><text x="250" y="80" text-anchor="middle" class="pn">1.0</text><text x="250" y="97" text-anchor="middle" class="nl">Register</text><text x="250" y="112" text-anchor="middle" class="nl">Member</text><rect x="410.0" y="52.0" width="140" height="36" class="dsbg"/><path d="M550.0,52.0 H410.0 V88.0 H550.0" class="ds"/><line x1="444.0" y1="52.0" x2="444.0" y2="88.0" class="ds"/><text x="427.0" y="75" text-anchor="middle" class="pn">D1</text><text x="497.0" y="75" text-anchor="middle" class="nl">Members</text><circle cx="380" cy="260" r="46" class="pr"/><text x="380" y="250" text-anchor="middle" class="pn">2.0</text><text x="380" y="267" text-anchor="middle" class="nl">Borrow</text><text x="380" y="282" text-anchor="middle" class="nl">Book</text><circle cx="630" cy="290" r="46" class="pr"/><text x="630" y="280" text-anchor="middle" class="pn">3.0</text><text x="630" y="297" text-anchor="middle" class="nl">Return</text><text x="630" y="312" text-anchor="middle" class="nl">Book</text><rect x="110.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M250.0,382.0 H110.0 V418.0 H250.0" class="ds"/><line x1="144.0" y1="382.0" x2="144.0" y2="418.0" class="ds"/><text x="127.0" y="405" text-anchor="middle" class="pn">D2</text><text x="197.0" y="405" text-anchor="middle" class="nl">Books</text><rect x="450.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M590.0,382.0 H450.0 V418.0 H590.0" class="ds"/><line x1="484.0" y1="382.0" x2="484.0" y2="418.0" class="ds"/><text x="467.0" y="405" text-anchor="middle" class="pn">D3</text><text x="537.0" y="405" text-anchor="middle" class="nl">Loans</text></svg></div>

ตอบให้ได้ก่อนเปิดเฉลย:
1. แผนภาพนี้เป็น context diagram หรือ level 0? รู้ได้อย่างไร
2. มี external entity, process และ data store อย่างละกี่ตัว อะไรบ้าง
3. process ใด**เขียน**ข้อมูลลง D1 Members และ process ใด**อ่าน**จาก D1
4. process 2.0 Borrow Book ใช้ข้อมูลจากที่ไหนบ้าง และส่งผลลัพธ์ไปที่ไหน
5. "member card" ไหลจากไหนไปไหน
6. ถ้าบรรณารักษ์ต้องการ**ปิดรายการยืม**เมื่อคืนหนังสือ แผนภาพนี้ขาด flow อะไร

<details><summary>เฉลยแบบฝึก A</summary>
<ol>
<li><b>Level 0</b> เพราะมีหลาย process ที่มีเลข 1.0, 2.0, 3.0 และมี data store (context diagram มี process เดียวและไม่มี data store)</li>
<li>External entity <b>2</b>: Member, Librarian · Process <b>3</b>: 1.0 Register Member, 2.0 Borrow Book, 3.0 Return Book · Data store <b>3</b>: D1 Members, D2 Books, D3 Loans</li>
<li>เขียน D1: <b>1.0 Register Member</b> (new member) · อ่าน D1: <b>2.0 Borrow Book</b> (member status)</li>
<li>เข้า: <b>borrow request</b> จาก Member, <b>member status</b> จาก D1, <b>book info</b> จาก D2 · ออก: <b>loan record</b> ลง D3 Loans</li>
<li>จาก <b>1.0 Register Member</b> ไปหา <b>Member</b></li>
<li>3.0 Return Book <b>อ่าน</b> D3 อย่างเดียว ไม่มี flow <b>เขียนกลับ</b>ไปที่ D3 (เช่น "returned loan / loan status") จึงอัปเดตรายการยืมไม่ได้ · ถ้าโจทย์บอกว่าต้องปิดรายการ แผนภาพนี้ไม่ครบ</li>
</ol></details>

## 3. แบบฝึก B: หาจุดผิด (Food Ordering, Level 0)

<div class="dfdwrap"><svg class="dfd" viewBox="0 0 760 440" role="img" aria-label="Food ordering DFD with errors" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="currentColor"/></marker></defs><line x1="116.3" y1="103.0" x2="215.4" y2="189.7" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="144.3" y="136.4" width="43.0" height="18" rx="4" class="lbg"/><text x="165.8" y="149.9" text-anchor="middle" class="fl">order</text><line x1="250.0" y1="382.0" x2="250.0" y2="266.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="176.0" y="314.0" width="76.0" height="18" rx="4" class="lbg"/><text x="214.0" y="327.5" text-anchor="middle" class="fl">menu items</text><line x1="287.3" y1="246.9" x2="475.0" y2="382.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)" class="bad"/><line x1="296.0" y1="220.0" x2="424.0" y2="220.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="318.7" y="210.0" width="82.6" height="18" rx="4" class="lbg"/><text x="360.0" y="223.5" text-anchor="middle" class="fl">order total</text><line x1="497.0" y1="382.0" x2="477.6" y2="265.4" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="482.0" y="313.7" width="82.6" height="18" rx="4" class="lbg"/><text x="523.3" y="327.2" text-anchor="middle" class="fl">order lines</text><line x1="150.0" y1="80.0" x2="600.0" y2="80.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="320.5" y="70.0" width="109.0" height="18" rx="4" class="lbg"/><text x="375.0" y="83.5" text-anchor="middle" class="fl">special request</text><line x1="606.0" y1="236.6" x2="150.0" y2="98.2" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="389.9" y="117.4" width="56.2" height="18" rx="4" class="lbg"/><text x="418.0" y="130.9" text-anchor="middle" class="fl">receipt</text><line x1="320.0" y1="400.0" x2="430.0" y2="400.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="350.2" y="390.0" width="49.6" height="18" rx="4" class="lbg"/><text x="375.0" y="403.5" text-anchor="middle" class="fl">prices</text><line x1="91.8" y1="103.0" x2="106.3" y2="284.1" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="24.5" y="183.6" width="89.2" height="18" rx="4" class="lbg"/><text x="69.1" y="197.1" text-anchor="middle" class="fl">payment info</text><rect x="30.0" y="57.0" width="120" height="46" class="en"/><text x="90" y="85" text-anchor="middle" class="nl b">Customer</text><rect x="600.0" y="57.0" width="120" height="46" class="en"/><text x="660" y="85" text-anchor="middle" class="nl b">Restaurant</text><circle cx="250" cy="220" r="46" class="pr"/><text x="250" y="210" text-anchor="middle" class="pn">1.0</text><text x="250" y="227" text-anchor="middle" class="nl">Take</text><text x="250" y="242" text-anchor="middle" class="nl">Order</text><circle cx="470" cy="220" r="46" class="pr"/><text x="470" y="210" text-anchor="middle" class="pn">2.0</text><text x="470" y="227" text-anchor="middle" class="nl">Calculate</text><text x="470" y="242" text-anchor="middle" class="nl">Bill</text><circle cx="650" cy="250" r="46" class="pr"/><text x="650" y="240" text-anchor="middle" class="pn">3.0</text><text x="650" y="257" text-anchor="middle" class="nl">Send</text><text x="650" y="272" text-anchor="middle" class="nl">Receipt</text><circle cx="110" cy="330" r="46" class="pr"/><text x="110" y="320" text-anchor="middle" class="pn">4.0</text><text x="110" y="337" text-anchor="middle" class="nl">Payment</text><rect x="180.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M320.0,382.0 H180.0 V418.0 H320.0" class="ds"/><line x1="214.0" y1="382.0" x2="214.0" y2="418.0" class="ds"/><text x="197.0" y="405" text-anchor="middle" class="pn">D1</text><text x="267.0" y="405" text-anchor="middle" class="nl">Menu</text><rect x="430.0" y="382.0" width="140" height="36" class="dsbg"/><path d="M570.0,382.0 H430.0 V418.0 H570.0" class="ds"/><line x1="464.0" y1="382.0" x2="464.0" y2="418.0" class="ds"/><text x="447.0" y="405" text-anchor="middle" class="pn">D2</text><text x="517.0" y="405" text-anchor="middle" class="nl">Orders</text></svg></div>

แผนภาพนี้มี**จุดผิดอย่างน้อย 6 จุด** ลองหาและบอกชื่อข้อผิดพลาดก่อนเปิดเฉลย

<details><summary>เฉลยแบบฝึก B</summary>
<ol>
<li><b>Black hole: 2.0 Calculate Bill</b> มี input (order total, order lines) แต่ไม่มี output ใดๆ · แก้: เพิ่ม output เช่น "bill" ไปหา Customer</li>
<li><b>Miracle: 3.0 Send Receipt</b> ส่ง receipt ออก แต่ไม่มี input เลย · แก้: ต้องมีข้อมูลเข้า เช่น "bill" จาก 2.0 หรือ "payment confirmation" จาก 4.0</li>
<li><b>Entity → Entity:</b> Customer ส่ง "special request" ไป Restaurant ตรงๆ · แก้: ลบออก หรือให้ผ่าน process (เช่น 1.0 Take Order)</li>
<li><b>Data store → Data store:</b> D1 Menu ส่ง "prices" ไป D2 Orders ตรงๆ · แก้: ให้ process (เช่น 1.0) อ่านราคาแล้วเขียนลง D2</li>
<li><b>Flow ไม่มีชื่อ:</b> ลูกศรจาก 1.0 Take Order ไป D2 Orders · แก้: ตั้งชื่อ เช่น "order details"</li>
<li><b>ชื่อ process เป็นคำนาม:</b> 4.0 "Payment" · แก้: "Process Payment"</li>
<li><b>4.0 ยังเป็น black hole ด้วย:</b> มี payment info เข้า แต่ไม่มี output</li>
</ol></details>

## 4. แบบฝึก C: Balancing (Course Registration)

**Context diagram**

<div class="dfdwrap"><svg class="dfd" viewBox="0 0 760 220" role="img" aria-label="Context diagram" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="currentColor"/></marker></defs><line x1="170.0" y1="92.9" x2="320.1" y2="92.9" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="183.9" y="82.9" width="122.2" height="18" rx="4" class="lbg"/><text x="245.0" y="96.4" text-anchor="middle" class="fl">registration form</text><line x1="320.1" y1="129.5" x2="170.0" y2="129.6" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="210.4" y="119.6" width="69.4" height="18" rx="4" class="lbg"/><text x="245.1" y="133.1" text-anchor="middle" class="fl">timetable</text><line x1="440.0" y1="110.0" x2="580.0" y2="110.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="468.7" y="100.0" width="82.6" height="18" rx="4" class="lbg"/><text x="510.0" y="113.5" text-anchor="middle" class="fl">fee summary</text><rect x="50.0" y="87.0" width="120" height="46" class="en"/><text x="110" y="115" text-anchor="middle" class="nl b">Student</text><circle cx="380" cy="110" r="60" class="pr"/><text x="380" y="100" text-anchor="middle" class="pn">0</text><text x="380" y="117" text-anchor="middle" class="nl">Course</text><text x="380" y="132" text-anchor="middle" class="nl">Registration</text><rect x="580.0" y="87.0" width="140" height="46" class="en"/><text x="650" y="115" text-anchor="middle" class="nl b">Finance Office</text></svg></div>

**Level 0**

<div class="dfdwrap"><svg class="dfd" viewBox="0 0 760 420" role="img" aria-label="Level 0 diagram" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse"><path d="M0,0 L10,5 L0,10 z" fill="currentColor"/></marker></defs><line x1="130.0" y1="130.0" x2="274.0" y2="130.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="140.9" y="120.0" width="122.2" height="18" rx="4" class="lbg"/><text x="202.0" y="133.5" text-anchor="middle" class="fl">registration form</text><line x1="454.0" y1="63.0" x2="361.1" y2="109.4" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="349.8" y="76.2" width="115.6" height="18" rx="4" class="lbg"/><text x="407.6" y="89.7" text-anchor="middle" class="fl">advisor approval</text><line x1="358.3" y1="155.5" x2="431.7" y2="204.5" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="340.5" y="170.0" width="109.0" height="18" rx="4" class="lbg"/><text x="395.0" y="183.5" text-anchor="middle" class="fl">approved course</text><line x1="512.3" y1="211.9" x2="626.3" y2="163.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="528.0" y="177.4" width="82.6" height="18" rx="4" class="lbg"/><text x="569.3" y="190.9" text-anchor="middle" class="fl">fee summary</text><line x1="504.8" y1="260.1" x2="599.2" y2="342.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="544.0" y="291.1" width="76.0" height="18" rx="4" class="lbg"/><text x="582.0" y="304.6" text-anchor="middle" class="fl">enrollment</text><line x1="550.0" y1="355.8" x2="335.9" y2="342.8" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="385.2" y="339.3" width="115.6" height="18" rx="4" class="lbg"/><text x="443.0" y="352.8" text-anchor="middle" class="fl">enrolled courses</text><line x1="257.1" y1="307.9" x2="98.5" y2="153.0" stroke="currentColor" stroke-width="1.6" marker-end="url(#ah)"/><rect x="113.1" y="220.4" width="69.4" height="18" rx="4" class="lbg"/><text x="147.8" y="233.9" text-anchor="middle" class="fl">timetable</text><rect x="20.0" y="107.0" width="110" height="46" class="en"/><text x="75" y="135" text-anchor="middle" class="nl b">Student</text><rect x="440.0" y="17.0" width="120" height="46" class="en"/><text x="500" y="45" text-anchor="middle" class="nl b">Advisor</text><rect x="610.0" y="117.0" width="140" height="46" class="en"/><text x="680" y="145" text-anchor="middle" class="nl b">Finance Office</text><circle cx="320" cy="130" r="46" class="pr"/><text x="320" y="120" text-anchor="middle" class="pn">1.0</text><text x="320" y="137" text-anchor="middle" class="nl">Check</text><text x="320" y="152" text-anchor="middle" class="nl">Prerequisite</text><circle cx="470" cy="230" r="46" class="pr"/><text x="470" y="220" text-anchor="middle" class="pn">2.0</text><text x="470" y="237" text-anchor="middle" class="nl">Enroll</text><text x="470" y="252" text-anchor="middle" class="nl">Course</text><circle cx="290" cy="340" r="46" class="pr"/><text x="290" y="330" text-anchor="middle" class="pn">3.0</text><text x="290" y="347" text-anchor="middle" class="nl">Create</text><text x="290" y="362" text-anchor="middle" class="nl">Timetable</text><rect x="550.0" y="342.0" width="140" height="36" class="dsbg"/><path d="M690.0,342.0 H550.0 V378.0 H690.0" class="ds"/><line x1="584.0" y1="342.0" x2="584.0" y2="378.0" class="ds"/><text x="567.0" y="365" text-anchor="middle" class="pn">D1</text><text x="637.0" y="365" text-anchor="middle" class="nl">Enrollments</text></svg></div>

คำถาม: Level 0 นี้ **balance** กับ context diagram หรือไม่ ถ้าไม่ ผิดตรงไหน และแก้ได้อย่างไร

<details><summary>เฉลยแบบฝึก C</summary>
<ul>
<li>flow ใน context: เข้า <b>registration form</b> (Student) · ออก <b>timetable</b> (Student), <b>fee summary</b> (Finance Office) → ทั้ง 3 ตัวมีอยู่ใน Level 0 ครบ ✓</li>
<li>แต่ Level 0 มี <b>external entity ใหม่ "Advisor"</b> และ flow <b>"advisor approval"</b> เข้ามา ซึ่ง<b>ไม่มีใน context diagram</b> → <b>ไม่ balance</b></li>
<li>แก้: เพิ่ม Advisor และ flow "advisor approval" เข้า process 0 ใน context diagram (หรือถ้า advisor ไม่เกี่ยวกับระบบจริง ให้ลบออกจาก Level 0)</li>
<li>D1 Enrollments ไม่ทำให้ไม่ balance เพราะ data store ภายในไม่ต้องแสดงใน context</li>
</ul></details>

## 5. Git functions: คำสั่งที่ต้องรู้ (จาก Lab 9)

| คำสั่ง | ทำอะไร |
|---|---|
| `git config --global user.name / user.email` | ตั้งชื่อและอีเมลที่จะติดไปกับ commit |
| `git init` | เริ่มใช้ Git ในโฟลเดอร์ (สร้าง local repo) |
| `git status` | ดูสถานะไฟล์ (untracked / modified / staged) |
| `git add <file>` / `git add .` | นำไฟล์เข้า **staging area** |
| `git commit -m "msg"` | บันทึกเวอร์ชันใน **local repo** |
| `git log` / `git log --name-status` | ดูประวัติ commit (+ ไฟล์ที่เปลี่ยน) |
| `git diff <file>` | ดูว่าแก้อะไรที่ยังไม่ stage |
| `git diff <id1> <id2> <file>` | เทียบไฟล์ระหว่าง 2 commit |
| `git clone <url>` | คัดลอก remote repo ลงเครื่อง |
| `git push` | ส่ง commit จาก local ขึ้น remote |
| `git fetch` | ดึงข้อมูลจาก remote มาดู **ยังไม่ merge** |
| `git pull` | **fetch + merge** เข้า branch ปัจจุบัน |
| `git branch` / `git branch <name>` | ดู branch / สร้าง branch |
| `git checkout <name>` | ย้ายไป branch นั้น |
| `git push --set-upstream origin <name>` | push branch ใหม่ครั้งแรก และผูก local กับ remote (`-u`) |
| `git branch -a` | ดู branch ทั้งหมดรวมฝั่ง remote |
| `git fetch -p` | fetch และลบการอ้างอิง branch remote ที่ถูกลบแล้ว (prune) |
| `git branch -vv` | ดู branch พร้อม tracking info |
| `git branch -d <name>` | ลบ branch local ที่ merge แล้ว |

**สถานะไฟล์:** untracked → (`add`) → **staged** → (`commit`) → **committed** → แก้ไฟล์ → **modified** → (`add`) → staged …

**Pull Request flow:** push branch → New pull request → assign reviewer → reviewer comment (กด "+" หน้าบรรทัด) → เจ้าของแก้ + push (commit ใหม่เข้า PR เอง) → review ซ้ำ → **Merge** เข้า master → ลบ branch → ทุกคน `git fetch -p`

## 6. แบบฝึก D: Git จากสถานการณ์

เขียนคำสั่งให้ครบก่อนเปิดเฉลย
1. สร้างโฟลเดอร์ใหม่ให้เป็น Git repo แล้ว commit ไฟล์ `app.java` ครั้งแรก
2. แก้ `app.java` แล้วอยากดูว่าแก้อะไรไปบ้าง**ก่อน** add
3. สร้าง branch ชื่อ `dev_kong` ย้ายไปทำงาน แล้ว push branch นี้ขึ้น GitHub ครั้งแรก
4. อยากรู้ว่าบน remote มีอะไรใหม่ แต่**ยังไม่อยากให้รวม**เข้ากับงานตัวเอง
5. อยากรวมงานเพื่อนจาก remote เข้ามาใน branch ปัจจุบันเลย
6. commit ไฟล์ `Feature.java` ใน `dev_kong` แล้ว `git checkout master` แล้วสั่ง `ls` จะเห็น `Feature.java` ไหม เพราะอะไร
7. PR ถูก merge และ branch บน GitHub ถูกลบแล้ว ต้องทำอะไรที่เครื่องตัวเอง
8. `git status` แสดง `Changes to be committed: modified: se.txt` ไฟล์อยู่ในสถานะอะไร และขั้นต่อไปคืออะไร

<details><summary>เฉลยแบบฝึก D</summary>
<ol>
<li><code>git init</code> → <code>git add app.java</code> → <code>git commit -m "Added app.java"</code></li>
<li><code>git diff app.java</code></li>
<li><code>git branch dev_kong</code> → <code>git checkout dev_kong</code> → (commit งาน) → <code>git push --set-upstream origin dev_kong</code></li>
<li><code>git fetch</code> (ดึงมาดูแต่ยังไม่ merge)</li>
<li><code>git pull</code> (= fetch + merge)</li>
<li><b>ไม่เห็น</b> เพราะ commit อยู่ใน branch dev_kong ไฟล์จะไม่ปรากฏใน branch อื่นจนกว่าจะ merge</li>
<li><code>git checkout master</code> → <code>git pull</code> → <code>git fetch -p</code> → <code>git branch -vv</code> (ดูว่า branch ไหน gone) → <code>git branch -d dev_kong</code></li>
<li><b>staged</b> (add แล้ว) · ขั้นต่อไป <code>git commit -m "..."</code></li>
</ol></details>

## 7. Unit test

> ⚠️ **ยังรอไฟล์ Unit test ของแล็บ** ส่งมาแล้วจะเติมบทนี้พร้อมแบบฝึก
> ระหว่างนี้ทวน [Lab 10 Testing](#sel-10) (flow chart, independent paths, test cases) และ [L13 Unit/Integration/System Testing](#se-13) ของวิชาทฤษฎี

## จุดเน้นสำหรับสอบ
- 🔥 สัญลักษณ์ DFD 4 ตัว + การตั้งชื่อ (process = กริยา, ที่เหลือ = คำนาม)
- 🔥 จับผิด: **black hole, miracle, gray hole**, entity→entity, entity↔store, store→store, flow ไม่มีชื่อ, **unbalanced**
- 🔥 context = process เดียว ไม่มี data store · level 0 แตก process หลัก
- 🔥 Git: init/add/commit/status/log/diff · clone/push/**fetch vs pull** · branch/checkout/`--set-upstream` · PR → merge → `fetch -p` / `branch -d`
- ⭐ ไฟล์ใน branch หนึ่งไม่โผล่ใน branch อื่นจนกว่าจะ merge
- ⭐ สถานะไฟล์: untracked → staged → committed → modified
