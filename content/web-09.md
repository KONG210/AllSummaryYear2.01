# L9 · Node.js Part I: Intro, Modules, First Server

## ภาพรวมของบท

บทนี้เป็นบทแรกของหน่วย Node.js (มี 3 ตอน: Part I พื้นฐาน, Part II Express, Part III Cookie/Session/Database)

**เป้าหมายของบท (Today Objective)**
- เข้าใจแนวคิดพื้นฐานของ Node.js
- ใช้ module พื้นฐานของ Node ได้
- สร้าง web server ง่าย ๆ ได้เองตั้งแต่ศูนย์

**ลำดับเนื้อหา:** Node.js คืออะไร → Hello World → Global object → ระบบ module (สร้างเอง + built-in: Path, URL, File System, HTTP) → สร้าง server เอง + routing

> 🔗 ภาพใหญ่: เราจะเริ่มจาก "ไฟล์ JS ที่รันนอก browser" → แยกโค้ดเป็น module → ใช้ module สำเร็จรูปจัดการไฟล์/URL → สุดท้ายใช้ `http` module สร้าง server ที่ตอบ request ได้ ซึ่งเป็นพื้นฐานของ Express ใน Part II

---

## 1. RECAP: การพัฒนาเว็บ และ Client-side vs Server-side

### 🔹 Front-end / Back-end / Full-stack

**คืออะไร?**
- **Front-end** = ส่วนที่ผู้ใช้เห็น: HTML, CSS, JavaScript, Framework
- **Back-end** = ส่วนหลังบ้าน: Database, API
- **Full-stack** = ทำได้ทั้งสองฝั่ง

### 🔹 Client-side vs Server-side

**คืออะไร?**
- **Client-side code** เขียนด้วย HTML, CSS, JavaScript และ **รันใน web browser**
- **Server-side code** เขียนได้หลายภาษา เช่น PHP, Python, Ruby, C#, และ **Node.js (JavaScript)** โค้ดฝั่ง server **เข้าถึง OS และ file system ของ server ได้เต็มที่**
- งานหลักของ server-side คือ **เลือกว่าจะส่งเนื้อหาอะไรกลับไปให้ browser** ตาม request ที่เข้ามา

| เรื่อง | Client-side | Server-side |
|---|---|---|
| รันที่ไหน | Browser ของผู้ใช้ | เครื่อง server |
| ภาษา | HTML / CSS / JS | PHP, Python, Ruby, C#, Node.js ฯลฯ |
| เข้าถึงไฟล์ระบบ | ไม่ได้ | ได้ (OS + file system) |
| หน้าที่ | แสดงผล/โต้ตอบ | ตัดสินใจว่าจะตอบอะไรกลับ |

> ⭐ จำง่าย ๆ: client = "หน้าร้าน" ที่ลูกค้าเห็น, server = "ครัว" ที่ทำอาหารตามออเดอร์

---

## 2. Web Page Request / Response (6 ขั้นตอน)

**คืออะไร?** ลำดับการทำงานเมื่อผู้ใช้ขอหน้าเว็บ

**ขั้นตอน**
1. Client ส่ง **HTTP (GET) request** ไปที่ Web Server
2. Web Server รับ request แล้ววิเคราะห์ว่าจะเอา resource จากไหน — ถ้าเป็น **static resource** (ไฟล์ที่มีอยู่แล้ว เช่น HTML, รูป) ข้ามไปขั้นที่ 5 ได้เลย
3. ถ้าเป็น **dynamic resource** → ส่งต่อให้ **Web Application (server-side code)** ซึ่งจะอ่านข้อมูลที่ต้องใช้จาก **database**
4. Server นำข้อมูลที่ได้มา **รวมกับ HTML template**
5. ส่ง **response** ที่มี HTML ที่สร้างเสร็จกลับไป
6. Client ได้รับหน้า HTML

| | Static | Dynamic |
|---|---|---|
| ความหมาย | ไฟล์สำเร็จรูป ส่งได้ทันที | สร้างใหม่ทุกครั้งจากโค้ด + DB |
| ผ่าน Web App/DB ไหม | ไม่ | ผ่าน |
| ตัวอย่าง | `logo.png`, `about.html` | หน้ารายวิชาที่ดึงจากฐานข้อมูล |

### 🔹 Server, IP Address, Port
- **IP Address** = ที่อยู่ของเครื่อง เช่น `10.34.111.111` หรือ `localhost` (เครื่องตัวเอง)
- **Port** = "ช่องประตู" ของโปรแกรมบนเครื่องนั้น เช่น `3000`, `8080`, `8081`
- URL ตัวอย่าง: `http://localhost:8081` = เครื่องตัวเอง, โปรแกรมที่ฟังอยู่ที่ port 8081

> ⭐ IP = ที่อยู่ตึก, Port = เลขห้อง

สไลด์ยังแสดงตัวอย่าง log ของ server Node (ใช้ `nodemon`) ที่ log ทุก request เช่น `Request made at /mypage`, `404: Invalid accessed` และอีกตัวที่ต่อ DB `tinycollege` แล้ว insert ข้อมูล — เป็นภาพตัวอย่างว่าจบหน่วยนี้จะทำอะไรได้

---

## 3. What is Node.js?

### 🔹 นิยาม

> 📌 **Definition:** Node.js is an **asynchronous JavaScript runtime** built on **Chrome's V8 JavaScript engine** — "An open-source server environment"

**อธิบายง่าย ๆ:** ปกติ JS รันได้แค่ใน browser — Node.js เอา "เครื่องยนต์ V8" ของ Chrome มาใส่ให้ JS รันบนเครื่อง/server ได้ และเพิ่ม module สำหรับงานฝั่ง server

**Node = JS Engine (V8) + Modules** เช่น File system, HTTP, Utilities

**สิ่งที่ Node ไม่มี (ต่างจาก JS ใน browser)**
- ❌ ไม่มี **DOM**
- ❌ ไม่มี object อย่าง **`document`**, **`window`**
- ❌ ไม่มี **Cookies** (ในตัว engine)

> ⚠️ มักสับสน: `console.log()` ใน Node **ไม่ใช่** `window.console.log()` เพราะ Node ไม่มี `window`

### 🔹 Node.js Architecture: "Single thread event loop model"

**ทำงานอย่างไร (3 จังหวะตามสไลด์)**
1. เมื่อมี request เข้ามา **main thread** รับ request แล้วส่งงานไปยังหน่วยที่เหมาะสม (File System, Database, Computation)
2. Thread **พร้อมรับ request ถัดไปทันที** โดยไม่ต้องรอหน่วยนั้นทำเสร็จ → เรียกว่า **non-blocking**
3. เมื่อหน่วยนั้นทำงานเสร็จ thread จึงตอบกลับ request นั้น และวนแบบนี้ไปเรื่อย ๆ

**ตัวอย่างเปรียบเทียบ:** พนักงานรับออเดอร์คนเดียว (single thread) รับออเดอร์โต๊ะ 1 ส่งเข้าครัว แล้วไปรับโต๊ะ 2 ต่อเลย ไม่ยืนรอหน้าครัว พออาหารโต๊ะ 1 เสร็จค่อยไปเสิร์ฟ

### 🔹 Node.js Facts
- ✅ เหมาะกับแอป **highly-scalable, data (I/O) intensive, real-time**
- ✅ JavaScript ตลอดทั้งระบบ
- ✅ เหมาะกับแอปที่เป็น **JSON APIs**
- ❌ **ไม่เหมาะกับงาน computation-intensive** (คำนวณหนัก ๆ) เพราะมี thread หลักเดียว ถ้าคำนวณนานจะบล็อกทุกคน

> 🔥 ต้องเข้าใจ: Node เก่งงาน I/O (อ่านไฟล์, DB, network) เพราะ non-blocking แต่ไม่เก่งงาน CPU หนัก

---

## 4. Getting Started + "Hello World"

**ติดตั้ง:** ดาวน์โหลดเวอร์ชัน **LTS** จาก https://nodejs.org แล้วตรวจสอบด้วย

```bash
node --version
# v24.21.0
```

**ขั้นตอนรันไฟล์**
1. เขียนโค้ด JS แล้วบันทึกเป็น `filename.js`
2. เปิด command line แล้ว `cd` ไปยังโฟลเดอร์นั้น
3. รันด้วย `node filename.js`
4. ดูผลใน command line (หรือ browser ถ้าเป็น server)

```js
// helloworld.js
/* Everything is written in JS */
console.log("Hello World!!");
console.log("In the command line");
```
```text
$ node helloworld.js
Hello World!!
In the command line
```

---

## 5. Global Objects

**คืออะไร?** object ที่ **ใช้ได้ในทุก module (ทุกไฟล์) โดยไม่ต้อง import**

| Global | ความหมาย |
|---|---|
| `__dirname` | path ของ **โฟลเดอร์** ที่ไฟล์ปัจจุบันอยู่ |
| `__filename` | path เต็มของ **ไฟล์** ปัจจุบัน (รวมชื่อไฟล์) |
| `exports`, `module` | ใช้ส่งออกของจาก module |
| `require()` | ใช้โหลด module |
| `console.log()` | พิมพ์ข้อความ |
| `process` | ข้อมูลของ process ที่รันอยู่ (เช่น `process.env` ใน Part III) |
| `global` | **global namespace** (คล้าย `window` ใน browser) |

```js
// myglobal_demo.js
let message = "Sam I am";
console.log(__dirname);   // Show current dir
console.log(__filename);  // Show current full path
global.console.log(message);
```

> ⚠️ `__dirname` = โฟลเดอร์, `__filename` = โฟลเดอร์ + ชื่อไฟล์

---

## 6. Node Module System

### 🔹 Modularity in Node.js

**หลักการสำคัญ**
- **ทุกไฟล์ JS ใน Node คือ module หนึ่งตัว**
- module มักประกอบด้วยชุด function ที่เราอยากนำไปใช้
- property และ method ที่ประกาศใน module **มี scope อยู่ในไฟล์นั้น** = เป็น **local/private** ของ module นั้น
- จึงมี function **ชื่อเดียวกันในสองไฟล์ได้โดยไม่ทับกัน**

```js
// file1.js
let message = "Hola";
function sayHello(){ console.log(message); }
sayHello();

// file2.js
function sayHello(){ console.log("Hello"); }   // ชื่อซ้ำได้ ไม่ชนกัน
```

### 🔹 `module.exports`

**คืออะไร?** object พิเศษที่ใช้ **"เปิดเผย" (expose)** ค่าจาก module ให้ไฟล์อื่นใช้ได้ ส่งออกได้หลายแบบ

```js
// 1) Export Literals — msg.js
module.exports = "Hello World!";

// 2) Export Function — hello.js
let message = 'Hello ICT';
function sayHello(){ console.log("Hello"); }
module.exports.sayHi = sayHello;   // ส่งออก sayHello ในชื่อ sayHi

// 3) Export Objects — ictgreet.js
let message = 'Hello ICT';
module.exports.greetICT = message;
```

### 🔹 `require`

**คืออะไร?** ใช้ **โหลด module** จากไฟล์/โฟลเดอร์ โดยใช้ **relative path** (`./`)

```text
(root)
 |_ message/
 |    |_ ictgreet.js
 |    |_ msg.js
 |_ greet.js
 |_ hello.js
```
```js
// greet.js
const eng = require("./hello");
const ictgreet = require('./message/ictgreet');
const msg = require('./message/msg');

eng.sayHi();                       // Hello
console.log(`${msg}`);             // Hello World!
console.log(`${ictgreet.greetICT}`); // Hello ICT
```

> 🔥 ต้องเข้าใจ: ชื่อที่ใช้เรียกคือ **ชื่อที่ export** (`sayHi`) ไม่ใช่ชื่อ function เดิม (`sayHello`)
> ⭐ export literal ตรง ๆ (`module.exports = "..."`) ใช้ตัวแปรที่ require มาได้เลย (`msg`), ส่วน export แบบ property ต้องเรียก `.ชื่อ` (`ictgreet.greetICT`)

### 🔹 Built-in Modules

Node มี module สำเร็จรูปสำหรับสร้าง server ที่สำคัญในบทนี้: **Path, URL, File System, HTTP** (ดูทั้งหมดที่ https://nodejs.org/docs/latest/api/)

---

## 7. Path Module

**คืออะไร?** เครื่องมือจัดการ **path ของไฟล์และโฟลเดอร์**

**ทำไมต้องใช้?** path แต่ละ OS ไม่เหมือนกัน (Windows ใช้ `\`, Mac/Linux ใช้ `/`) → ให้ module จัดการให้อัตโนมัติ

| Method | หน้าที่ |
|---|---|
| `path.resolve()` | หา **absolute path** จาก relative path |
| `path.parse()` | แยก path เป็น object: `root`, `dir`, `base`, `name`, `ext` |
| `path.extname()` | เอานามสกุลไฟล์ |
| `path.join()` | ต่อชิ้นส่วน path เข้าด้วยกัน |

```js
const path = require('path');

let fullPath = path.resolve(__filename);
console.log(fullPath);            // /Users/jidapa.kra/Documents/demo/path.js
console.log(`File extension is ${path.extname(__filename)}`); // .js
console.log(path.parse(fullPath));
/* {
  root: '/',
  dir: '/Users/jidapa.kra/Documents/demo',
  base: 'path.js',
  ext: '.js',
  name: 'path'
} */
console.log(path.join(__dirname, 'test.js')); // /Users/jidapa.kra/Documents/demo/test.js
```

**ความหมายแต่ละส่วนของ `path.parse`**
- `root` = root ของระบบ
- `dir` = path โฟลเดอร์เริ่มจาก root
- `base` = ชื่อไฟล์ + นามสกุล
- `name` = ชื่อไฟล์อย่างเดียว
- `ext` = นามสกุล

> 🔥 `path.join(__dirname, '...')` ใช้บ่อยมาก (เช่น `res.sendFile` ใน Express ต้องใช้ absolute path)

---

## 8. URL Module

**คืออะไร?** เครื่องมือ **แยก (parse) URL** ออกเป็นส่วน ๆ — สร้าง object ด้วย `new URL(...)` แล้วอ่าน property ได้

```js
const url = require('url');
const fakeurl = new URL('https://www.covid19.com:8080/p/a/t/h?country=TH&time=now');

console.log(fakeurl.href);      // URL ทั้งหมด
console.log(fakeurl.protocol);  // https:
console.log(fakeurl.host);      // www.covid19.com:8080
console.log(fakeurl.port);      // 8080
console.log(fakeurl.pathname);  // /p/a/t/h
console.log(fakeurl.search);    // ?country=TH&time=now  (query string)
console.log(fakeurl.searchParams); // URLSearchParams { 'country' => 'TH', 'time' => 'now' }
console.log(fakeurl.searchParams.get('time')); // now
```

| Property | ได้อะไร |
|---|---|
| `protocol` | `https:` |
| `host` | hostname **+ port** |
| `port` | port อย่างเดียว |
| `pathname` | path หลัง host |
| `search` | query string ทั้งก้อน (มี `?`) |
| `searchParams.get(k)` | ค่าของ parameter ตัวเดียว |

> ⚠️ `host` รวม port แต่ `hostname` ไม่รวม / `search` เป็นข้อความ แต่ `searchParams` เป็น object ที่ `.get()` ได้

---

## 9. File System (FS) Module

**คืออะไร?** API สำหรับทำงานกับไฟล์

| งาน | Async | Sync |
|---|---|---|
| อ่าน | `fs.readFile()` | `fs.readFileSync()` |
| สร้าง/ต่อท้าย | `fs.appendFile()` | `fs.appendFileSync()` |
| เขียน | `fs.writeFile()` | `fs.writeFileSync()` |
| ลบ | `fs.unlink()` | `fs.unlinkSync()` |
| เปลี่ยนชื่อ | `fs.rename()` | `fs.renameSync()` |

**หลักการสำคัญ**
- ทุก method มา **เป็นคู่** ทั้ง asynchronous และ synchronous
- **Synchronous (blocking):** JS ที่เหลือต้อง **รอ** จนอ่าน/เขียนไฟล์เสร็จ
- **Asynchronous (non-blocking):** สั่งแล้วไปทำอย่างอื่นต่อ เสร็จแล้วค่อยเรียก callback → **ควรใช้แบบนี้** เพื่อไม่บล็อก

### 🔹 ตัวอย่าง: อ่านไฟล์เดียว (async + callback)

```js
const path = require('path');
const fs = require('fs');
const readme1 = path.join(__dirname, "src/README.txt");

fs.readFile(readme1, "utf8", function(err, contents){   // callback function
  if(err) console.log("Error: ", err);
  else console.log(contents);   // Hello This is README.txt
});
```
- พารามิเตอร์: (path, encoding, callback)
- callback รับ `(err, contents)` — **เช็ก error ก่อนเสมอ**

### 🔹 อ่านหลายไฟล์แบบ async → ลำดับไม่แน่นอน!

ถ้าเรียก `fs.readFile` 3 ครั้งติดกัน ผลที่พิมพ์ออกมา **สลับลำดับได้ทุกครั้งที่รัน** (สไลด์แสดงผล 4 รอบ ลำดับไม่ซ้ำกัน) เพราะไฟล์ไหนอ่านเสร็จก่อนก็พิมพ์ก่อน

### 🔹 Asynchronous Programming: ทำอย่างไรให้ได้ลำดับ

1. ใช้ **เวอร์ชัน synchronous** (ถ้ามี)
2. เขียน **nested callback** (callback ซ้อน callback)
3. หรือหลีกเลี่ยง callback hell ด้วยวิธีที่สะอาดกว่า: **Promise**, **Async/Await**

```js
// Synchronous version — ลำดับแน่นอน 1 → 2 → 3
let content = fs.readFileSync(readme1, "utf8");
console.log(content);
let content2 = fs.readFileSync(readme2, "utf-8");
console.log(content2);
```

```js
// Nested callbacks — ลำดับแน่นอน แต่เกิด "Callback Hell"
fs.readFile(readme1, "utf8", function(err, contents){
  if(err) console.log("Error: ", err);
  else { console.log(contents);
    fs.readFile(readme2, "utf8", function(err, contents){
      if(err) console.log("Error: ", err);
      else { console.log(contents);
        fs.readFile(readme3, "utf8", function(err, contents){
          if(err) console.log("Error: ", err);
          else console.log(contents);
        });
      }
    });
  }
});
```

> 📌 **Callback Hell:** callback ซ้อนกันเป็นขั้นบันได มี `})` เรียงเป็นแถวตอนท้าย → **อ่านยาก ดูแลยาก ไม่ใช่ good practice**

| | Sync | Async (ไม่ซ้อน) | Async ซ้อน callback |
|---|---|---|---|
| บล็อกไหม | บล็อก | ไม่บล็อก | ไม่บล็อก |
| ลำดับผลลัพธ์ | แน่นอน | **ไม่แน่นอน** | แน่นอน |
| ข้อเสีย | ช้า รอทีละอัน | คุมลำดับไม่ได้ | callback hell |

---

## 10. HTTP Module + Build a Simple Server

**คืออะไร?** Node ส่งข้อมูลผ่าน **HTTP** ได้ — `http` module สร้าง **HTTP server** ที่ "ฟัง" (listen) port และตอบกลับเมื่อมี request

**หลักการ**
- Web server รับ **HTTP request** จาก client แล้วส่ง **HTTP response** กลับ
- response เป็นได้ทั้ง **plain text, HTML, JSON**
- ตัวอย่างในสไลด์รันที่ `localhost` port `8081` → URL `http://localhost:8081`

```js
// myFirstNodeServer.js
const http = require("http");          // 1. import HTTP module

const myServer = http.createServer((req, res) => {   // 2. สร้าง server
  res.statusCode = 200;                       // Status 200: OK
  res.setHeader("Content-Type", "text/plain");
  res.write("Hello World!");
  res.end();                                  // จบ response (ต้องมี!)
});

myServer.listen(8081, () => {                // 3. ผูก server กับ port
  console.log("Listening on the port 8081");
});
```

**อธิบายทีละส่วน**
- `http.createServer(callback)` → callback ถูกเรียก **ทุกครั้งที่มี request** รับ `req` (request) และ `res` (response)
- `res.statusCode = 200` → **200 OK** = request สำเร็จ
- `res.setHeader("Content-Type", ...)` → บอกชนิดข้อมูล (`text/plain`, `text/html`)
- `res.write()` → เขียนเนื้อหา, `res.end()` → **ปิด response** (ถ้าไม่เรียก browser จะรอค้าง)
- `listen(8081, cb)` → **bind** server กับ address localhost ที่ port 8081
- หยุด server: **Ctrl+C**

**ทดสอบ:** รัน `node myFirstNodeServer.js` แล้วเปิด `http://localhost:8081` — ใน terminal จะเห็น request 2 อัน: `/` และ **`/favicon.ico`** (browser ขอไอคอนเองอัตโนมัติ)

### 🔹 Request object

```js
const myServer = http.createServer((req, res) => {
  console.log(`url ${req.url}`);       // url /
  console.log(`method ${req.method}`); // method GET
  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain");
  res.write("Request made!");
  res.end();
});
```
- `req.url` = path ที่ขอ เช่น `/`, `/favicon.ico`
- `req.method` = HTTP method เช่น `GET`

---

## 11. Respond with a Web Page + Routing

**แนวคิด:** server มักตอบด้วย **หน้าเว็บ** เช่น `http://localhost:8081/homepage` — เมื่อ request เข้ามาจะถูก **route** ไปยังโค้ดที่จัดการ โดยใช้ **`req.url`** ตัดสินใจ

**ตัวอย่าง:** `req.url` ของ `http://localhost:8081/bbq` คือ `"/bbq"`

**Flow:** เช็ก path → `"/bbq"`? → เปิด `bbq.html` / `"/shabu"`? → เปิด `shabu.html` / ไม่ใช่ทั้งคู่ → **404 Invalid**

```js
// routing.js
const http = require("http");
const fs = require("fs");          // ใช้เปิดไฟล์ HTML

const myServer = http.createServer((req, res) => {
  console.log("Req: " + req.url);
  if (req.url === "/") {
    res.statusCode = 200;
    res.setHeader("Content-Type", "text/plain");
    res.write("Greeting");
    res.end();
  } else if (req.url === "/shabu") {
    fs.readFile("./shabu.html", function(err, data) {
      res.statusCode = 200;
      res.setHeader("Content-Type", "text/html");
      res.write(data);
      res.end();
    });
  } else if (req.url === "/bbq") {
    fs.readFile("./bbq.html", function(err, data) {
      res.statusCode = 200;
      res.setHeader("Content-Type", "text/html");
      res.write(data);
      res.end();
    });
  } else {
    res.statusCode = 404;               // Status 404: Error
    res.setHeader("Content-Type", "text/plain");
    res.write("Invalid requested :(");
    res.end();
  }
});
myServer.listen(8081);
```

> 🔥 เขียน server ด้วย `http` ล้วนต้องเขียน `if/else` เช็ก URL เอง + อ่านไฟล์เอง + ตั้ง header เอง → ยุ่งยาก นี่คือเหตุผลที่ Part II ใช้ **Express** ช่วยจัดการ routing
> ⚠️ HTML ต้องใช้ `Content-Type: text/html` ไม่งั้น browser จะแสดงเป็นโค้ดดิบ

---

## จุดที่มักสับสน

| เรื่อง | A | B | จำง่าย ๆ |
|---|---|---|---|
| `__dirname` vs `__filename` | โฟลเดอร์ | โฟลเดอร์ + ชื่อไฟล์ | name = มีชื่อไฟล์ |
| `readFile` vs `readFileSync` | async, ไม่บล็อก, ใช้ callback | sync, บล็อก, return ค่าเลย | Sync = ต่อคิว |
| `res.write` vs `res.end` | เขียนเนื้อหา (กี่ครั้งก็ได้) | ปิด response (ครั้งเดียว ต้องมี) | end = วางสาย |
| `module.exports = x` vs `module.exports.a = x` | ทั้ง module คือ x | เพิ่ม property ชื่อ a | มีจุด = เรียกด้วยจุด |
| status 200 vs 404 | OK สำเร็จ | Not Found | 4xx = ฝั่ง client ผิด |

## จุดเน้นสำหรับสอบ (สอบเขียนโค้ด)

- 🔥 **เขียน server ด้วย `http.createServer` ได้เองจากหน้าว่าง** พร้อม statusCode, setHeader, write, end, listen
- 🔥 **routing ด้วย `req.url` + `fs.readFile` ส่งไฟล์ HTML + 404**
- 🔥 export/require module (literal, function, object)
- ⭐ path: `resolve`, `parse` (5 ส่วน), `extname`, `join`
- ⭐ URL: `protocol`, `host`, `port`, `pathname`, `search`, `searchParams.get`
- ⚠️ async ไม่รับประกันลำดับ → แก้ด้วย sync / nested callback / Promise / async-await
- 📌 Node.js = asynchronous JS runtime บน V8, single thread event loop, non-blocking

## ✅ Coverage Check
- [x] Node.js Unit / Objective / Outline
- [x] Recap Front/Back-end, Client vs Server-side
- [x] Request/Response 6 ขั้น, IP & Port, ตัวอย่าง Node server
- [x] What is Node.js, Architecture 3 จังหวะ, Facts
- [x] Getting started, Hello World, Global objects
- [x] Modularity, module.exports, require, Built-in modules
- [x] Path, URL, FS (async/sync, read one/multiple, callback hell, sync version)
- [x] HTTP module, create server 1–3, run, test, req/res, routing + ตัวอย่าง bbq/shabu
