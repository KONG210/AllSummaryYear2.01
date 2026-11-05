# L12 · Web Services

## ภาพรวมของบท

**เป้าหมาย:** อธิบายแนวคิดของ **JSON** และ **Web Services** ได้ และ **เรียก Web Service ด้วย JavaScript** ได้

**ลำดับ:** Before the Web (สถาปัตยกรรม tier) → HTTP → HTML vs XML → Cookies / LocalStorage / sessionStorage → JSON → Web Services → REST → JavaScript กับ WS (XHR, Fetch)

> 🔗 ภาพใหญ่: ระบบแยกเป็นชั้น (tier) → ชั้นต่าง ๆ คุยกันผ่าน **HTTP** → ข้อมูลที่ส่งต้องอยู่ในรูปที่โปรแกรมอ่านได้ (**XML / JSON**) → เมื่อเปิดบริการให้เครื่องอื่นเรียกใช้ผ่านเน็ตเรียกว่า **Web Service** → แนวทางออกแบบยอดนิยมคือ **REST** → ฝั่ง browser เรียกด้วย **fetch** (และ XHR แบบเก่า) — บทถัดไป (L13) จะสร้าง REST service เองด้วย Node.js

---

## 1. Before the Web: Layers และ Tiers

**Information System มี 3 layer**
1. **Resource Management (database) Layer** — จัดการข้อมูล
2. **Application Logic / Business Layer** — กฎทางธุรกิจ การประมวลผล
3. **Presentation (user interface) Layer** — หน้าจอผู้ใช้

เมื่อแยก layer เหล่านี้ออกจากกัน **ทางกายภาพ** เรียกว่า **Tier**

| สถาปัตยกรรม | ลักษณะ | จุดเด่น/จุดด้อย |
|---|---|---|
| **1-Tier** | 3 layer อยู่บน **เครื่องเดียว / ซอฟต์แวร์เดียว** (dumb terminal + mainframe) | layer ผูกกันแน่น, **scale และย้ายระบบยาก** |
| **2-Tier** | เรียกว่า **Client-Server**: presentation อยู่ที่ client, data อยู่ที่ server; application layer อยู่ฝั่งไหนก็ได้ | |
| **3-Tier** | แต่ละ layer **เกือบอิสระ** และอยู่คนละ tier: **front-end (GUI), middleware, back-end** | |
| **N-Tier** | aka **Distributed application** คล้าย 3-tier แต่ **เพิ่มจำนวน application server** เป็น tier แยก เพื่อกระจาย business logic | |

> ⭐ Layer = การแบ่งหน้าที่ทางตรรกะ · Tier = การแยกเครื่องจริง

### 🔹 WWW basics (ยุคแรก)
1. ผู้ใช้คลิก URL → server เรียก **script** ที่ผูกกับ URL นั้น
2. **CGI script** ทำหน้าที่เป็น client แบบดั้งเดิม
3. script ทำงาน ได้ผลลัพธ์ ส่งกลับให้ server (มักเป็นที่อยู่ของหน้าเว็บ)
4. server ดึงหน้านั้นส่งให้ browser

---

## 2. HTTP

### 🔹 HTTP as a communication protocol
- ออกแบบมาเพื่อ **แลกเปลี่ยนเอกสาร** คล้าย e-mail (ใช้ header แบบ RFC 822 และ **MIME types**)
- เป็น **รากฐานของการแลกเปลี่ยนข้อมูลทั้งหมดบนเว็บ**
- **MIME type** = กลไกบอก client ว่าเอกสารที่ส่งมาเป็นชนิดอะไร

### 🔹 HTTP aspects
- **simple และ human readable**
- **extensible** (ขยายได้)
- **stateless, but not sessionless** — ไม่จำสถานะเอง แต่ทำ session ได้ (ผ่าน cookie)

### 🔹 HTTP Request: 3 ส่วน

**1) Request Line** = `Method` + `Path/Resource` + `HTTP Version`
- Method: GET, POST, PUT, DELETE, ...
- Path: **URI** (Uniform Resource Identifier) หรือ **URL** (Uniform Resource Locator) ของ resource
- Version: HTTP/1.1, HTTP/2.0

**2) Request Headers**
| Header | ความหมาย |
|---|---|
| `Host` | domain/IP ของ server ปลายทาง |
| `User-Agent` | โปรแกรม client ที่ส่ง (browser, OS) |
| `Accept` | ชนิดข้อมูลที่ client รับได้/ต้องการ: `application/json`, `text/html`, `image/png` |
| `Content-Type` | ชนิดข้อมูลของ **body ที่ส่งไป** |
| `Authorization` | credential/token สำหรับเข้า resource ที่ป้องกันไว้ |

**3) Request Body** — ข้อมูลที่ client ส่งไป ใช้หลัก ๆ กับ **POST, PUT, DELETE**; รูปแบบขึ้นกับ `Content-Type`

```http
GET / HTTP/1.1
Host: developer.mozilla.org
Accept-Language: fr
```

### 🔹 HTTP Response: 3 ส่วน

**1) Status Line** = `HTTP Version` + `Status Code` + `Status Text`
| Code | ความหมาย |
|---|---|
| **200 OK** | request สำเร็จ |
| **404 Not Found** | ไม่พบ resource ที่ขอบน server |
| **500 Internal Server Error** | เกิด error ที่ server |

**2) Response Headers**
| Header | ความหมาย |
|---|---|
| `Content-Type` | ชนิดของ body: `text/html`, `application/json`, `image/png` |
| `Date` | วันเวลาที่สร้าง response |
| `Server` | ซอฟต์แวร์ web server + เวอร์ชัน |
| `Content-Length` | ขนาด body เป็น byte |

**3) Response Body** — เนื้อหาจริง เป็น HTML, JSON, XML, text หรือรูปแบบอื่นตาม `Content-Type`

```http
HTTP/1.1 200 OK
Date: Sat, 09 Oct 2010 14:28:02 GMT
Server: Apache
Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
ETag: "51142bc1-7449-479b075b2891b"
Accept-Ranges: bytes
Content-Length: 29769
Content-Type: text/html

<!DOCTYPE html... (29769 bytes ของหน้าเว็บ)
```

**HTTP Components** ที่ต้องรู้: request methods, headers, response status codes

### 🔹 Passing Parameter
การมี **form** ให้ผู้ใช้ส่งข้อมูลไป server ทำให้ต้องปรับ HTML (และ HTTP) แต่ได้ interface ที่ทำได้มากกว่าแค่ดึงไฟล์

> ⚠️ มักสับสน: `Accept` = client บอกว่า **อยากได้** อะไรกลับ · `Content-Type` = บอกว่าข้อมูลที่ **ส่งมา** เป็นอะไร (ใช้ได้ทั้ง request และ response)

---

## 3. Contents and Presentation: HTML vs XML

- **HTML** = markup language บอกว่าเอกสาร **ควรแสดงผลอย่างไร** (รูปแบบการแสดง)
- มี markup language อื่นที่มีมาก่อน HTML เช่น **SGML** (Standard Generalized Markup Language)
- HTML จัดรูปแบบเพื่อมนุษย์อ่าน แต่ **บอกความหมาย (semantics) ของเนื้อหาไม่ได้** → จึงมี **XML**

### 🔹 XML
- **tag ไม่ได้กำหนดไว้ล่วงหน้า** (ต่างจาก HTML) — ตั้งเองได้
- เป้าหมาย: วิธีมาตรฐานในการ **กำหนดโครงสร้างข้อมูลเพื่อแลกเปลี่ยนและจัดเก็บ**
- **XML Schema รองรับ data type**
- **ไม่ได้มีไว้ให้ browser แสดงผล**
- โปรแกรม/เครื่องประมวลผลอัตโนมัติได้
- ใช้เป็นภาษากลางในการ **marshalling/serializing** argument เวลาเรียก service ข้าม Internet
- เอกสาร XML เป็น **โครงสร้างต้นไม้** เริ่มจาก **root** แตกไปถึง **leaves**

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>   <!-- XML Declaration: version + encoding -->
<note>                                        <!-- Root element -->
  <to>Wud</to>                                <!-- Child element -->
  <from>Bell</from>
  <heading>Reminder</heading>
  <body>Please buy fruits today</body>
</note>                                       <!-- End root -->
```

### 🔹 XML Schema กับ XML Document
```xml
<xs:element name="university">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="name" type="xs:string"/>
      <xs:element name="address" type="xs:string"/>
      <xs:element name="city" type="xs:string"/>
      <xs:element name="country" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```
```xml
<university>
  <name>Mahidol</name>
  <address>999 Phuttamonthon 4 Road</address>
  <city>Nakhon Pathom</city>
  <country>Thailand</country>
</university>
```
Schema = "แบบฟอร์ม" กำหนดว่าเอกสารต้องมี element อะไร เรียงอย่างไร ชนิดอะไร · Document = ข้อมูลจริงที่กรอกตามแบบ

| | HTML | XML |
|---|---|---|
| tag | กำหนดไว้แล้ว | ตั้งเอง |
| จุดประสงค์ | แสดงผล | เก็บ/แลกเปลี่ยนข้อมูล |
| สำหรับ | browser/มนุษย์ | โปรแกรม/เครื่อง |
| บอกความหมายข้อมูล | ไม่ได้ | ได้ |

---

## 4. Cookies, LocalStorage, sessionStorage

### 🔹 Cookies
- ชื่ออื่น: HTTP cookie, web cookie
- ข้อมูล **เล็กมาก** ที่ดาวน์โหลดจาก server ตอนเข้าเว็บ **เก็บไว้ใน browser**
- ส่งไปกับ request ไป server เดิม และใช้บอก server ว่า request มาจากไหน เช่น **คง login ไว้**
- เก็บเป็นคู่ **key/value**
- รักษา **session state** ข้ามหลายหน้าได้ เช่น ตะกร้าสินค้า
- จำกัด **ประมาณ 4KB** และ **มีวันหมดอายุ**
- วัตถุประสงค์หลัก: **Session management, Personalization, Tracking**
- ตัวอย่าง: `"username=Wudhichart; expires=Thu, 18 Dec 2016 12:00:00 UTC"`

### 🔹 LocalStorage
- ใช้เก็บข้อมูลคล้าย cookie
- พื้นที่ **อย่างน้อย 5MB ต่อ domain** (ขึ้นกับการตั้งค่า)
- เว็บใน **domain เดียวกันใช้ LocalStorage ร่วมกัน**
- **server แก้ผ่าน HTTP header ไม่ได้**
- เก็บเป็น key/value และ **ทั้ง key และ value ต้องเป็น string**
- การใช้งาน: store, get, set, remove ข้อมูล และจัดการผ่าน GUI (DevTools)

### 🔹 sessionStorage
- เหมือน LocalStorage แต่ **อยู่เฉพาะใน browser tab ปัจจุบัน**
- **refresh หน้า ข้อมูลยังอยู่**

| เรื่อง | Cookies | LocalStorage | sessionStorage |
|---|---|---|---|
| ขนาด | ~**4KB** | ~**5MB**/domain | เหมือน LocalStorage |
| เข้าถึงจาก | **client (JS) และ server (HTTP header)** | **client (JS) เท่านั้น** | client (JS) เท่านั้น |
| หมดอายุ | ตั้งวันหมดอายุได้ | อยู่จนกว่าผู้ใช้/โปรแกรมจะลบ | อยู่เฉพาะ tab นั้น (refresh ไม่หาย) |
| ส่งไป server อัตโนมัติ | **ใช่ ทุก request** | ไม่ | ไม่ |

> ⭐ จำง่าย ๆ: cookie = ตั๋วเล็ก ๆ ที่ยื่นให้ server ทุกครั้ง · LocalStorage = ตู้เก็บของในเครื่อง server มองไม่เห็น · sessionStorage = ตู้ที่ผูกกับ tab

---

## 5. JSON (JavaScript Object Notation)

### 🔹 ปัญหา: จะเก็บ object ลง LocalStorage ได้ไง?
- แอปส่วนใหญ่ออกแบบแบบ **OOP** ข้อมูลจึงเป็น **object**
- แต่ LocalStorage รับแค่ **String** → ต้องแปลง object เป็น string
- เงื่อนไข: การแปลงต้อง **คงทั้งข้อมูลและโครงสร้าง** เพื่อแปลงกลับได้

> 📌 แปลง object → string เรียก **serialisation / stringifying** · แปลงกลับ string → object เรียก **deserialisation / parsing**

### 🔹 JSON คืออะไร
- มาจากแนวคิด **object literal notation** ของ JS
- เป็น **รูปแบบข้อมูลมาตรฐาน** ที่ใช้ส่งข้อมูลมีโครงสร้าง (เช่น object) อย่างแพร่หลาย

**ต่างจาก JS object literal อย่างไร (สำคัญ)**
1. **JSON เป็น String**
2. **JSON ไม่รองรับ properties** แบบ object (เช่น ไม่มี method/function)
3. **ชื่อ property ใน JSON ต้องอยู่ในเครื่องหมายคำพูด** (`"firstName"`)

**ชนิดข้อมูลพื้นฐานของ JSON:** Number, String, Boolean, Array, Object (JSON object), **null**

```js
// JS object literal — ชื่อ property ไม่ต้องมี quote
var wudhichart = {
  firstName: "Wudhichart",
  lastName: "sawangphol",
  age: 30,
  university: "Mahidol",
  feesPaid: true
};
```
```json
{
  "wudhichart": {
    "firstName": "Wudhichart",
    "lastName": "sawangphol",
    "age": 30,
    "university": "Mahidol",
    "feesPaid": true
  }
}
```

**JSON Array**
```json
{
  "people": [
    {"firstName": "Wudhichart", "lastName": "Sawangphol", "age": 30, "university": "Mahidol", "feesPaid": true},
    {"firstName": "Pawitra", "lastName": "Chiravirakul", "age": 30, "university": "Mahidol", "feesPaid": true}
  ]
}
```
เครื่องมือ: JSON Editor (jsoneditoronline.org)

### 🔹 ใช้ JSON เก็บ object ลง LocalStorage
JSON ช่วยได้เพราะ **เป็น String** และ **เก็บโครงสร้าง object ได้ครบ**

```js
// Object -> JSON
let str = JSON.stringify(obj);
localStorage.setItem("user", str);
// JSON -> Object
let back = JSON.parse(localStorage.getItem("user"));
```

> 🔥 `JSON.stringify(object)` = object → JSON string · `JSON.parse(json)` = JSON string → object

---

## 6. Introduction to Web Services (WS)

> 📌 **Definition (W3C):** "A Web service is a software system/application designed to support **interoperable machine-to-machine interaction over a network**"

**หลักการ**
- ให้บริการจากเครื่องระยะไกล (remote machine)
- วิธีมาตรฐานในการ **เชื่อมเว็บแอปเข้าด้วยกัน**
- request/response เข้ารหัสในรูปที่ **โปรแกรมถอดได้ง่าย** — ที่นิยมคือ **XML (SOAP หรือ POX)** และ **JSON**
- มี 2 แบบหลัก: **SOAP** และ **RESTful web services (RESTful Web APIs)**

### 🔹 Open Standards and Architecture
- **SOAP** — Simple Object Access Protocol
- **WSDL** — Web Services Description Language
- **HTTP** — เป็น transport protocol (SOAP/HTTP ส่ง message ข้ามเครือข่าย)
- **REST** — Representational State Transfer

### 🔹 Web Services vs API

| | Web Service | API |
|---|---|---|
| คืออะไร | ใช้ให้ **เครื่องคุยกันผ่านเครือข่าย** | **interface ระหว่าง 2 แอป** |
| ต้องมี network | **ต้องมีเสมอ** | ไม่จำเป็น |
| protocol | ส่วนใหญ่ใช้ HTTP | หลากหลาย |

> ⭐ **Web Service ทุกตัวเป็น API แต่ API ไม่ใช่ Web Service ทุกตัว**

---

## 7. REST Architecture

> 📌 **REST (REpresentational State Transfer)** เป็น **architecture (สถาปัตยกรรม)** — **ไม่ใช่** technology, standard specification หรือ protocol

**REST ใช้มาตรฐานเหล่านี้:** HTTP, URL, XML/HTML/JSON ฯลฯ, MIME types (`text/xml`, `image/gif`, `application/json`, `audio/mpeg`)

### 🔹 REST Constraints/Principles (6 ข้อ — ต้องจำ)

REST constraints = กฎการออกแบบที่ทำให้ได้ลักษณะเฉพาะของ REST

**1. Client/Server**
- client และ server ต้อง **พัฒนาแยกกันได้โดยไม่พึ่งพากัน**
- แยกเรื่อง UI (client) ออกจากเรื่องเก็บข้อมูล (server)
- ให้ส่วนต่าง ๆ พัฒนาอิสระ, UI ย้ายข้ามแพลตฟอร์มได้ดีขึ้น, **scale ดีขึ้น** เพราะ server ง่ายลง

**2. Stateless**
- server **ไม่เก็บอะไรเกี่ยวกับ request ล่าสุดของ client** ถือว่าทุก request ใหม่ — **ไม่มี session, ไม่มี history**
- **ทุก request ต้องมีข้อมูลครบ** ในการเข้าใจ request นั้น
- ใช้ context ที่เก็บบน server ไม่ได้
- เพิ่ม scalability และ reliability

**3. Cacheable**
- resource ที่ cache ได้ **ต้องประกาศตัวเองว่า cacheable**
- ช่วยประสิทธิภาพฝั่ง client และ server รับโหลดน้อยลง (scale ได้)
- ถ้า response cache ได้ client มีสิทธิ์ **ใช้ข้อมูลนั้นซ้ำ** กับ request ที่เหมือนกันภายหลัง
- ข้อเสีย: **reliability อาจลดลง** ถ้าข้อมูลใน cache เก่า (stale) ต่างจากของจริงบน server มาก

**4. Uniform Interface**
- interface ของ API ต้องกำหนดชัดและ **ทำตามอย่างเคร่งครัด**
- resource หนึ่งควรมี **logical URI เดียว** และมีวิธีดึงข้อมูลที่เกี่ยวข้อง
- เข้าถึงทุก resource ด้วย interface กลาง (**HTTP GET, POST, PUT, DELETE**)
- ทำให้สถาปัตยกรรมโดยรวมง่ายขึ้น
- แต่ **ลดประสิทธิภาพ** เพราะส่งข้อมูลในรูปแบบมาตรฐาน ไม่ได้ปรับให้เหมาะกับแต่ละแอป

**5. Layered System**
- deploy API แบบหลายชั้นได้
- **โปร่งใส:** client บอกไม่ได้ว่าต่อตรงกับ server ปลายทางหรือผ่านตัวกลาง
- ช่วย scale ด้วย **load balancing** กระจายบริการหลายเครือข่าย/หลายโปรเซสเซอร์

**6. Code-On-Demand** (**optional** — ข้อเดียวที่ไม่บังคับ)
- ให้ logic ฝั่ง client (เช่น browser) **อัปเดตได้แยกจาก server**
- ขยายความสามารถ client โดย **ดาวน์โหลดโค้ด (applet/script) มารัน**

```json
// GET /api/resource
{
  "data": { "id": 1, "name": "Resource Name" },
  "code": "function printSomething() { console.log('Executing client-side code'); }"
}
```

### 🔹 Guiding principles ของ Uniform Interface (4 ข้อ)

**1. Identifying the resource**
- ทุก resource ต้องมี **URI ที่เฉพาะเจาะจง**
- **Resource** = ข้อมูลอะไรก็ได้ที่ตั้งชื่อได้ (เอกสาร, รูป, บริการ, กลุ่มของ resource, สิ่งของจริงเช่นคน)
- ใช้ **คำนาม (nouns) ไม่ใช่คำกริยา (verbs)** ใน URI
- **Resource identifier (URI)** ระบุ resource หนึ่งตัว, ควรเป็น "nice URI"
- ตัวอย่าง: `GET https://www.googleapis.com/customsearch/v1?parameters` → **Company/Owner** (googleapis) / **API Name** (customsearch) / **Version** (v1)

**REST methods**
| Method | ความหมาย |
|---|---|
| **GET** | ดึงสำเนาของ resource |
| **DELETE** | ลบ resource |
| **POST** | **สร้าง** (บางครั้งใช้ update) |
| **PUT** | **อัปเดต** (บางครั้งใช้สร้าง) |
| **PATCH** | อัปเดต **บางส่วน** ของ resource |

**PUT vs POST (ออกสอบบ่อย)**
- **POST ไม่ idempotent แต่ PUT idempotent** (idempotent = ทำซ้ำกี่ครั้งผลลัพธ์เหมือนทำครั้งเดียว)
- ใช้ **PUT** เมื่อ **client เป็นคนกำหนด URI** ของ resource ใหม่
- ใช้ **POST** เมื่อ **server เป็นคนกำหนด URI** ของ resource ใหม่

> ตัวอย่าง: กด POST สมัครสมาชิก 3 ครั้ง → อาจได้ 3 บัญชี (ไม่ idempotent) · PUT `/student/5` ด้วยข้อมูลเดิม 3 ครั้ง → ยังเป็นนักศึกษาคนที่ 5 ข้อมูลเดิม (idempotent)

**2. Resource representation**
- representation = **เอกสารที่จับสถานะปัจจุบันของ resource**
- resource หนึ่งมี **หลาย representation** ได้ (HTML, XML, JSON, TXT ...)
- client ระบุว่ารับแบบไหนได้ทาง header **`Accept`**
- representation มี **link** ไปยัง resource อื่นได้

**3. Self-descriptive Messages**
- แต่ละ message มี **ข้อมูลพอที่จะรู้ว่าต้องประมวลผลอย่างไร**
- ต้องส่ง **metadata** ใน request/response เช่น HTTP response code, `Host`, `Content-Type`

**4. HATEOAS (Hypermedia as the Engine of Application State)**
- ใช้ **hypertext/link นำทางไปใน API** เช่น response ของ `GET http://api.domain.com/management/departments` จะมี link ไป resource ที่เกี่ยวข้อง

---

## 8. JavaScript and WS

วิธีพื้นฐานเรียก WS ด้วย JS: **XMLHttpRequest (XHR)** และ **Fetch**

### 🔹 XMLHttpRequest
- built-in browser object สำหรับส่ง HTTP request ใน JS
- ปัจจุบันใช้เพื่อ **รองรับ script เก่า** และ **รองรับ browser เก่า**

```js
let request = new XMLHttpRequest();               // 1. สร้าง
request.open(method, URL, [async, user, password]); // 2. ตั้งค่า
request.responseType = "json";                    // (เลือก) ชนิด response
request.send([body]);                             // 3. ส่ง
request.onload = function() {                     // 4. รับผล
  console.log(request.status, request.statusText, request.response);
};
```

**`open()` parameters:** `method` ("GET"/"POST"), `URL`, `async` (ถ้าตั้ง `false` = synchronous), `user`/`password` (basic HTTP auth)

**`send([body])`:** มี parameter เดียว (optional) — GET ไม่มี body, POST ใช้ body ส่งข้อมูล

**Events:** `load` (เสร็จ แม้ status 400/500 และโหลดครบ), `error` (ส่ง request ไม่ได้), `progress` (กำลังดาวน์โหลด รายงานความคืบหน้า) → ใช้ `onload`, `onerror`, `onprogress`

**Response properties:** `status` (เช่น 200, 404, 403), `statusText` (OK, Not Found, Forbidden), `response` (body)

**responseType:** `""` (default, string), `"text"`, `"arraybuffer"`, `"blob"`, `"document"` (XML/HTML), `"json"` (parse ให้อัตโนมัติ)

**readyState**
| ค่า | ชื่อ | ความหมาย |
|---|---|---|
| 0 | UNSENT | สถานะเริ่มต้น |
| 1 | OPENED | เรียก open แล้ว |
| 2 | HEADERS_RECEIVED | ได้ header แล้ว |
| 3 | LOADING | กำลังรับข้อมูล |
| 4 | DONE | เสร็จสมบูรณ์ |

การเปลี่ยน state จัดการด้วย `request.onreadystatechange`

> ⚠️ `load` เกิดขึ้น **แม้ status จะเป็น 400/500** — ต้องเช็ก `status` เอง

### 🔹 Fetch
- ฟังก์ชันสมัยใหม่สำหรับส่ง network request
- **คืนค่าเป็น Promise**

```js
let promise = fetch(url, [options]);   // options: method, headers, ...
```

**การได้ response เป็น 2 ขั้น**
1. promise ของ fetch **resolve เป็น object `Response`** ทันทีที่ server ตอบ header กลับ → เช็ก HTTP status ได้
   - promise **reject ถ้าส่ง HTTP request ไม่ได้**
   - `response.status` = status code, `response.ok` = `true` ถ้า status **200–299**
2. อ่าน **body** ด้วย method ที่คืน promise:
   - `response.text()`, `response.json()`, `response.formData()`, `response.blob()`, `response.arrayBuffer()`
   - `response.body` = ReadableStream อ่านทีละ chunk

```js
function getConferenceResponses() {
  let query = document.querySelector("#q").value;   // ค่าจาก textbox
  let rooturl = "https://www.googleapis.com/customsearch/v1?key=" + APi_key +
                "&cx=" + cx + "&q=" + query;
  fetch(rooturl)
    .then((res) => res.json())          // ขั้นที่ 2: แปลง body เป็น JSON
    .then((data) => {
      console.log(data);
      document.querySelector("#output").innerHTML =
        "<p>" + JSON.stringify(data, null, "\t") + "</p>";
    })
    .catch((err) => console.log(err));
}
```

| | XMLHttpRequest | Fetch |
|---|---|---|
| ยุค | เก่า (รองรับ browser เก่า) | ใหม่ |
| รูปแบบ | event (`onload`) | **Promise** (`.then` / `await`) |
| แปลง JSON | `responseType = "json"` | `response.json()` |

> ⚠️ fetch **ไม่ reject เมื่อได้ 404/500** (reject เฉพาะส่งไม่ได้) → เช็ก `response.ok` เอง

---

## จุดเน้นสำหรับสอบ
- 🔥 โครงสร้าง HTTP request (line/headers/body) และ response (status line/headers/body)
- 🔥 REST **6 constraints** + **4 guiding principles** ของ uniform interface
- 🔥 REST methods + **PUT vs POST (idempotent, ใครกำหนด URI)**
- 🔥 เขียน `fetch(url).then(r => r.json()).then(data => ...).catch(...)` ได้
- ⭐ Cookie vs LocalStorage vs sessionStorage (ขนาด, เข้าถึง, หมดอายุ, ส่งอัตโนมัติ)
- ⭐ JSON ต่างจาก object literal 3 ข้อ, 6 data types, `JSON.stringify` / `JSON.parse`
- ⭐ 1/2/3/N-tier
- 📌 นิยาม Web Service (W3C), WS vs API, REST เป็น architecture ไม่ใช่ protocol
- ⭐ XHR: open/send, events, status, readyState 0–4

## ✅ Coverage Check
- [x] Objectives, Contents
- [x] Before the Web, 1/2/3/N-tier, Web App architecture, WWW basics (CGI)
- [x] HTTP protocol, aspects, request (line/headers/body + example), response (status/headers/body + example), components, passing parameter
- [x] Contents & presentation, HTML vs XML, XML, XML tree, XML Schema
- [x] Cookies, LocalStorage, comparison table, sessionStorage
- [x] JSON: object data, serialisation, JSON vs literal, types, examples, array, editor, stringify/parse
- [x] Web Services, open standards, WS vs API
- [x] REST, constraints 6 ข้อ, uniform interface 4 ข้อ, identifying example, methods, PUT vs POST, representation, self-descriptive, HATEOAS, layered, code on demand
- [x] JS and WS: XHR (open, send, events, properties, responseType, readyState), Fetch (2 stages, example)
