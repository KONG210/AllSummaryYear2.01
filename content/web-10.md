# L10 · Node.js Part II: Express Framework

## ภาพรวมของบท

**เป้าหมาย (Today's Objective)**
- สร้าง server ง่าย ๆ ด้วย **Express Framework** ได้
- ทำ **routing** ด้วย Node.js ได้
- ทำ **form handling** (GET และ POST) ได้

**Outline:** RECAP Node.js → Express Framework → `express.Router()` → Form Handling (GET, POST) → (Cookies + Session, Environment Variables, Database ต่อใน Part III)

> 🔗 ภาพใหญ่: L9 เขียน server ด้วย `http` ล้วนต้อง `if (req.url === ...)` เอง → L10 ใช้ **Express** ทำ routing ให้ง่าย (`router.get('/path', ...)`), ส่งไฟล์ด้วย `res.sendFile`, จัดการ 404, เสิร์ฟ **static files** (CSS, รูป), แยก **modular routes** และ **รับข้อมูลจากฟอร์ม** (`req.query` สำหรับ GET, `req.body` สำหรับ POST) → นี่คือพื้นฐานของ WebLab 11 และ Project

---

## 1. RECAP: Node.js (จาก L9)

- **Node.js** = asynchronous JavaScript runtime บน **V8** — ไม่มี DOM, `document`, `window`
- server ด้วย `http` module: `http.createServer((req, res) => {...})`, `res.statusCode`, `res.setHeader`, `res.write`, `res.end`, `listen(8081)`
- เปิด `http://localhost:8081` → terminal เห็น request `/` และ `/favicon.ico`
- routing ด้วย `req.url` + `fs.readFile` ส่ง `bbq.html` / `shabu.html` / 404

```js
// แบบ L9: ต้องเช็ก URL เอง ตั้ง header เอง อ่านไฟล์เอง
if (req.url === "/bbq") {
  fs.readFile("./bbq.html", (err, data) => {
    res.statusCode = 200;
    res.setHeader("Content-Type", "text/html");
    res.write(data); res.end();
  });
} else { res.statusCode = 404; res.end("Invalid requested :("); }
```

---

## 2. Express Framework

> 📌 **Express** = **minimal and flexible** Node.js web application framework ที่มีฟีเจอร์ครบสำหรับเว็บและแอปมือถือ (เป็น framework ของ Node ที่นิยมที่สุดตัวหนึ่ง)

**ช่วยอะไร**
- จัดการ **routing**
- สร้าง **API**
- ใช้กับ **template engine** เช่น **Pug** หรือ **EJS**
  - ตอนรัน template engine **แทนตัวแปรใน template ด้วยค่าจริง**
  - แปลง template เป็น **HTML** ส่งให้ client
  - ทำให้ออกแบบหน้า HTML ง่ายขึ้น

---

## 3. Getting Started with Express

### 🔹 ขั้นตอน
1. สร้างโฟลเดอร์ทำงาน เช่น `MyFirstExpress`
2. **`npm init`** — สร้างโปรเจกต์ (NPM = package manager สำหรับติดตั้ง package ของ Node)
3. **`npm install express`** — ติดตั้ง Express และเพิ่มใน `dependencies`
4. [Optional] **`npm install nodemon`** — เฝ้าดูไฟล์ที่แก้แล้ว **restart โปรแกรมอัตโนมัติ**
5. สร้าง **entry point** และไฟล์อื่น ๆ

### 🔹 `npm init` → `package.json`
ระบบจะถามทีละข้อ (package name, version, description, **entry point**, test command, git repository, keywords, author) — ตั้ง entry point เป็น **`app.js`**

```json
{
  "name": "myfirstexpress",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Jidapa",
  "license": "ISC"
}
```
> 📌 **package.json** = meta-data ของโปรเจกต์ และเก็บรายการ module ที่ต้องใช้

### 🔹 หลัง `npm install express` และ `nodemon`
- ได้ไฟล์ **`package-lock.json`** (ควร commit ไฟล์นี้)
- `package.json` มีส่วน **`dependencies`** เพิ่มขึ้น:
```json
"dependencies": {
  "express": "^5.2.1",
  "nodemon": "^3.1.14"
}
```

### 🔹 สร้าง entry file (`app.js`)
```js
/* 1. Import modules */
const express = require('express');
const port = 3000;

/* 2. Create a new Express application */
const app = express();

/* 3. Create a route at ROOT: / */
app.get('/', (req, res) => {
  res.send('Hello World from Root!');
});

/* 4. Bind our server to the port */
app.listen(port, () => {
  console.log(`Server listening on port: ${port}`);
});
```
- `app.js` = ไฟล์หลักของ server เก็บ **ชุด routing** และโค้ดตั้งค่า server
- รันด้วย `node app.js` → `Server listening on port: 3000`

### 🔹 ตั้ง start script
```json
"scripts": {
  "start": "nodemon app.js"
}
```
- ใส่ `"start": "node [entryfile].js"` หรือ `"start": "nodemon [entryfile].js"`
- รันด้วย **`npm start`** (nodemon เฝ้า `*.*` นามสกุล js, mjs, json แก้ไฟล์แล้ว rerun เอง; พิมพ์ `rs` เพื่อ restart เอง)

### 🔹 โครงสร้างโฟลเดอร์
```text
(root)
 > node_modules
 |_ package-lock.json
 |_ package.json
 |_ app.js
 > html
    |_ bbq.html
    |_ shabu.html
```
เก็บ HTML, CSS และ resource ไว้ในโฟลเดอร์ของโปรเจกต์

---

## 4. Routing ด้วย express.Router

### 🔹 express.Router middleware
- ใช้ **`express.Router()`** ช่วยจัดการ route

> 📌 **Middleware** = function ที่ **เข้าถึง request object (req) และ response object (res)** ได้ (และ `next` เพื่อส่งต่อ)

**ตัวอย่าง route**
| URL | HTTP Method |
|---|---|
| `/` (root) | GET |
| `/shabu` | GET |
| `/bbq` | GET |
| `/product/view` | GET |
| `/product/add` | POST |
| `/user/add` | POST |

### 🔹 Web Server + Routing
```js
const express = require('express');
const path = require('path');
const port = 3000;
const app = express();

/* 1. Create a router object and register the router */
const router = express.Router();
app.use(router);

/* 2. Apply the created router to the route */
router.get('/', (req, res) => {
  res.send('Hello World from Root!');
});

/* 3. Add more routes with the proper HTTP method */
router.get('/shabu', (req, res) => {
  console.log('Shabu shabu...');
  res.sendFile(path.join(`${__dirname}/html/shabu.html`));
});

/* 4. Handle other unspecified paths */
router.use((req, res, next) => {
  console.log("404: Invalid accessed");
  res.status(404).send("Invalid Path");
});

app.listen(port, () => {
  console.log(`Server listening on port: ${port}`);
});
```

**อธิบาย**
- `express.Router()` สร้าง router แล้ว **ลงทะเบียนด้วย `app.use(router)`**
- `router.get('/shabu', ...)` = รับ **HTTP GET** ที่ path `/shabu` แล้วส่งไฟล์ด้วย **`res.sendFile`**
- **`__dirname`** = global variable ได้ **absolute path ของโฟลเดอร์** ที่ไฟล์ที่รันอยู่
- **`path.join`** = รวม path segment เป็น path เดียว
- `res.sendFile` **ต้องใช้ absolute path** จึงใช้ `path.join(__dirname, ...)`
- **`router.use((req, res, next) => ...)`** วางไว้ **ท้ายสุด** = จัดการ path อื่นที่ไม่ได้กำหนด (404) — `router.use` ถูกเรียก **เสมอไม่ว่า HTTP method อะไร**

**ผลลัพธ์**
- `localhost:3000` → "Hello World from Root!"
- `localhost:3000/shabu` → หน้า shabu.html
- `localhost:3000/somewhere` → path ไม่ได้กำหนด → error / "Invalid Path" (404)

> ⚠️ ลำดับสำคัญ: route ที่ประกาศก่อนถูกเช็กก่อน — ถ้าวาง `router.use(404)` ไว้ก่อน route อื่น ทุก request จะได้ 404

### 🔹 Connect to Navigation Bar
```html
<ul class="navbar-nav">
  <li class="nav-item"><a class="nav-link active" href="/">HelloWorld</a></li>
  <li class="nav-item"><a class="nav-link" href="/shabu">Shabu</a></li>
  <li class="nav-item"><a class="nav-link" href="/my-form">Form</a></li>
</ul>
```
> ⭐ path ใน `href` ต้อง **ตรงกับ route** ที่กำหนดใน `app.js`

---

## 5. Serving Static Files

**ปัญหา:** ใน HTML ที่ส่งด้วย `sendFile` การอ้าง **relative path** ไปยังไฟล์ static (รูป, CSS, JavaScript ฝั่ง front-end) **จะไม่ทำงาน**

```html
<!-- demostatic.html (แบบผิด) -->
<link rel="stylesheet" href="../public/mystyle.css">
<img src="../public/images/sampleImg.png">
<!-- No CSS or images work :( -->
```

**วิธีแก้:** ใช้ built-in middleware **`express.static`**

```js
/* 1. Set the static file directory (mount path) */
app.use('/', express.static(path.join(__dirname, 'public')));

router.get('/test', (req, res) => {
  console.log('Test static files...');
  res.sendFile(path.join(`${__dirname}/html/demostatic.html`));
});
```
```html
<!-- 2. Refer to the file using relative paths to the static directory -->
<link rel="stylesheet" href="/mystyle.css">
<img src="/images/sampleImg.png">
```
- ตั้ง static directory เริ่มที่ **`/public`** แล้ว → path ของ resource **ถูก reset** ให้เริ่มจาก `public`
- ไฟล์ `public/mystyle.css` → อ้างด้วย **`/mystyle.css`** · `public/images/sampleImg.png` → **`/images/sampleImg.png`**

```text
(root)
 > html
    |_ demostatic.html
 > public
    |_ mystyle.css
    > images
       |_ sampleImg.png
 |_ app.js
```

---

## 6. Extra: Modular Routes

เมื่อเว็บใหญ่ขึ้น ควรออกแบบ route เป็น **module** — แยก router ตามส่วน เช่น `/product` (add, edit, delete), `/user` (add, edit, view)

```js
const express = require('express');
const app = express();
const router1 = express.Router();
const router2 = express.Router();
app.use("/product", router1);   // Register router1
app.use("/user", router2);      // Register router2

// Main app
app.get('/', (req, res) => res.send("Homepage"));                     // /

// Module: product
router1.get('/', (req, res) => res.send("Module for product - Root"));  // /product
router1.get('/add', (req, res) => res.send("Module for product - Add")); // /product/add

// Module: user
router2.get('/', (req, res) => res.send("Module for user - Root"));      // /user
router2.get('/add', (req, res) => res.send("Module for user - Add"));    // /user/add

app.listen(3030, () => console.log("Server listening at Port 3030"));
```

> 🔥 **URL จริง = mount path ใน `app.use` + path ใน router** เช่น `app.use("/product", router1)` + `router1.get('/add')` = **`/product/add`**

---

## 7. Form Handling

### 🔹 หลักการ
- **form** = ที่ให้ผู้ใช้กรอกข้อมูล
- ควร **validate ที่ browser (client)** เมื่อทำได้ (JS, HTML5 attributes)
- form ถูกส่งไป server ด้วย **GET หรือ POST**
- หลังส่ง server จัดการค่าจาก form เช่น เพิ่ม user ใหม่, อัปเดตสินค้า
- ควร **validate ที่ server เพิ่ม** ถ้าจะจัดการข้อมูลลงฐานข้อมูล

### 🔹 HTML Form
```html
<form id="frmRegister" action="" method="">
  <label for="fname">First name:</label><br>
  <input type="text" name="fname" value="John"><br>
  <label for="lname">Last name:</label><br>
  <input type="text" name="lname" value="Doe"><br>
  <button type="submit">Submit</button>
</form>
```
| Attribute | ความหมาย |
|---|---|
| **`action`** | **ปลายทาง** ที่ form จะส่งไป (path ของ endpoint เช่น `/form-get`, `/form-post`) |
| **`method`** | **HTTP method** ที่ใช้ส่ง (GET หรือ POST) |
| **`name`** (ของ input) | ชื่อที่ใช้ **อ้างถึงค่า** ตอน server รับข้อมูล |

### 🔹 Route แสดงฟอร์ม
```js
router.get('/my-form', (req, res) => {
  console.log('Retrieve the form...');
  res.sendFile(path.join(`${__dirname}/html/myForm.html`));
});
```
เมื่อผู้ใช้ส่ง GET ไป `http://localhost:3000/my-form` จะได้ฟอร์ม

**ถ้าเริ่มจากศูนย์:** 1) `npm init`, `npm install express`, `npm install nodemon` 2) สร้าง `app.js` 3) ตั้ง start ใน `package.json`

---

## 8. Form ด้วย GET

```html
<form id="frmRegister" action="/form-get" method="GET">
  <input type="text" name="fname" value="John">
  <input type="text" name="lname" value="Doe">
  <button type="submit">Submit</button>
</form>
```
```js
/* Handle GET: Form submitted by GET */
router.get("/form-get", function (req, res) {
  console.log(`${req.method}`);   // GET
  console.log(req.query);         // { fname: 'John', lname: 'Doe' }
  const fname = req.query.fname;
  const lname = req.query.lname;
  res.send(`Form submitted by ${fname} ${lname} with ${req.method}`);
});
```
**กฎ**
- path ใน route (`/form-get`) **ต้องตรงกับ `action`** ของ form
- HTTP method ของ route (`router.get`) **ต้องตรงกับ `method`** ของ form
- **`req.query`** = **URL query parameters** (ส่วนหลัง `?` ใน URL)
- `fname`, `lname` มาจาก **`name` attribute** ของ `<input>`

**ผล:** URL กลายเป็น `localhost:3000/form-get?fname=John&lname=Doe` → แสดง "Form submitted by John Doe with GET"

---

## 9. Form ด้วย POST

```html
<form id="frmRegister" action="/form-post" method="POST">
  <input type="text" name="fname" value="John">
  <input type="text" name="lname" value="Doe">
  <button type="submit">Submit</button>
</form>
```

**ต้องมี middleware แยกข้อมูลใน body**
- HTTP POST **เข้ารหัสข้อมูลใน message body** → ต้องมี middleware ดึงข้อมูลออกมา
- ตั้งแต่ **Express 4.16** นำ **body-parser** มาไว้ใน Express แล้ว เขียนได้สั้นลง:

```js
router.use(express.json());                          // parse JSON payload
router.use(express.urlencoded({ extended: true }));  // parse urlencoded payload (form)
```
| Middleware | Parse อะไร |
|---|---|
| **`express.json()`** | request ที่มี **JSON payload** (มาจาก body-parser) |
| **`express.urlencoded()`** | request ที่มี **urlencoded payload** = ข้อมูลจาก HTML form (มาจาก body-parser) |

```js
const path = require("path");
const express = require("express");
const app = express();
const router = express.Router();
app.use("/", router);                                // Register the router
router.use(express.json());
router.use(express.urlencoded({ extended: true }));

router.get("/my-form", (req, res) => { /* sendFile myForm.html */ });

/* Handle POST: Form submitted by POST */
router.post("/form-post", function (req, res) {
  console.log(req.method);        // POST
  console.log(req.body);          // { fname: 'John', lname: 'Doe' }
  const fname = req.body.fname;
  const lname = req.body.lname;
  res.send(`Form submitted by ${fname} ${lname} with ${req.method}`);
});
```
- **`req.body`** = ข้อมูลจาก client (string หรือ JSON object)
- **ต้องมี middleware** parse JSON/urlencoded ไม่งั้น `req.body` = `undefined`
- URL **ไม่มี** `?fname=...` เพราะข้อมูลอยู่ใน body

---

## จุดที่มักสับสน

| เรื่อง | GET | POST |
|---|---|---|
| ข้อมูลอยู่ที่ | **URL (query string)** `?fname=John&lname=Doe` | **message body** |
| อ่านด้วย | **`req.query.fname`** | **`req.body.fname`** |
| route | `router.get(...)` | `router.post(...)` |
| ต้องมี middleware | ไม่ต้อง | **`express.urlencoded({extended:true})`** (+ `express.json()`) |
| เหมาะกับ | ค้นหา, ดึงข้อมูล | ส่งข้อมูลเพิ่ม/แก้, ข้อมูลยาวหรือไม่อยากโชว์ใน URL |

| เรื่อง | A | B |
|---|---|---|
| `app.get` vs `router.get` | route บน app โดยตรง | route บน router (ต้อง `app.use(router)`) |
| `router.get` vs `router.use` | เฉพาะ GET + path ตรง | **ทุก method**, ใช้ทำ middleware/404 |
| `res.send` vs `res.sendFile` | ส่งข้อความ/HTML string | ส่งไฟล์ (absolute path) |
| `req.query` vs `req.params` vs `req.body` | `?a=b` | `/:id` (L13) | body (POST) |

## จุดเน้นสำหรับสอบ (เขียนโค้ด)
- 🔥 เขียน `app.js` จากหน้าว่าง: require, `express()`, `Router()`, `app.use(router)`, `router.get`, `res.sendFile(path.join(__dirname, ...))`, 404 ด้วย `router.use`, `app.listen`
- 🔥 form GET (`req.query`) และ POST (`req.body` + `express.urlencoded`) — action/method/name ต้องตรงกัน
- 🔥 static files: `app.use('/', express.static(path.join(__dirname, 'public')))` แล้วอ้าง `/mystyle.css`
- ⭐ modular routes: `app.use("/product", router1)` → `/product/add`
- ⭐ npm init / install / start script / nodemon / package.json / package-lock.json
- 📌 middleware = function ที่เข้าถึง req, res (และ next)

## ✅ Coverage Check
- [x] Node.js unit, objectives, outline
- [x] Recap: what is Node, simple server, test, routing bbq/shabu
- [x] Express definition, template engine
- [x] Getting started: steps, npm init, package.json, install express/nodemon, entry file, run, start script, folder structure
- [x] Router middleware, route table, server + routing, sendFile, __dirname, path.join, outputs, 404 handler
- [x] Navigation bar, static files (problem + solution), modular routes
- [x] Form handling principles, HTML form, action/method, show form route, GET + req.query, POST + middleware + req.body
