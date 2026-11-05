# L11 · Node.js Part III: Express Recap, .env, Cookies, Session, Database

## ภาพรวมของบท

**เป้าหมาย:** สร้าง **cookie และ session** ด้วย Node.js ได้ และ **เชื่อม Node.js กับฐานข้อมูลเชิงสัมพันธ์ (MySQL)** ได้

**ลำดับ:** RECAP Express → Environment Variables (.env) → Cookies + Session → Database + Node.js

> 🔗 ภาพใหญ่: Express ช่วยทำ server/routing → `.env` เก็บค่าตั้งค่า (port, รหัส DB, secret) → HTTP จำอะไรไม่ได้ (stateless) จึงต้องใช้ **cookie** (เก็บฝั่ง client) หรือ **session** (เก็บฝั่ง server) → สุดท้ายต่อ **MySQL** ด้วย `mysql2` เพื่อ query ข้อมูลจริง ทั้งหมดนี้คือสิ่งที่ WebLab_11 และ Project ใช้

> ℹ️ สไลด์ L10 (Part II Express) ยังไม่มี — ส่วน RECAP ในบทนี้คือสรุป Express ที่อาจารย์ทวนไว้

---

## 1. RECAP: Express Framework

**คืออะไร?** Express คือ **web application framework ของ Node.js ที่เล็กและยืดหยุ่น** มีชุดฟีเจอร์สำหรับเว็บและแอปมือถือ

**ช่วยอะไร**
- จัดการ **routing**
- สร้าง **API**
- ทำงานกับ **template engine** เช่น **Pug** หรือ **EJS**
  - ตอนรัน template engine จะ **แทนตัวแปรใน template ด้วยค่าจริง**
  - แปลง template เป็น **HTML** แล้วส่งให้ client
  - ทำให้ออกแบบหน้า HTML ง่ายขึ้น

---

## 2. Project Setup & Configuration

### 🔹 คำสั่ง npm (ต้องจำ)

| Command | ทำอะไร | ใช้เมื่อ |
|---|---|---|
| `npm init` | สร้างโปรเจกต์ใหม่ แบบถามทีละข้อ | สร้างโปรเจกต์และอยากกำหนดรายละเอียด |
| `npm init -y` | สร้างด้วยค่า default (ข้ามคำถาม) | ตั้งโปรเจกต์เร็ว ๆ |
| `npm install` | ติดตั้ง dependency ทั้งหมดจาก `package.json` | หลัง clone/pull โปรเจกต์ |
| `npm install <package>` | ติดตั้ง package ตัวเดียว | เพิ่ม library เช่น express |
| `npm uninstall <package>` | ลบ package | ไม่ใช้แล้ว |

### 🔹 ขั้นตอนตั้งโปรเจกต์
1. สร้างโฟลเดอร์งาน
2. `npm init` → ได้ `package.json`
3. กำหนด **entry point** เช่น `app.js`
4. ติดตั้ง dependency: **express** (ทำ server), **nodemon** (restart อัตโนมัติตอนแก้ไฟล์)
5. ตั้ง start script ใน `package.json`

```json
"scripts": {
  "start": "nodemon app.js"
}
```
แล้วรันด้วย `npm start` — แก้ `app.js` เมื่อไหร่ nodemon จะ rerun ให้เอง

### 🔹 RECAP: Web Server + Routing ด้วย Express

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

router.get('/my-form', (req, res) => {
  console.log('Retrieve the form...');
  res.sendFile(path.join(`${__dirname}/html/myForm.html`));
});

app.listen(port, () => {
  console.log(`Server listening on port: ${port}`);
});
```

**อธิบาย**
- `express()` สร้างแอป, `express.Router()` สร้าง router แล้ว **ลงทะเบียนด้วย `app.use(router)`**
- `router.get(path, handler)` = เมื่อมี **GET** มาที่ path นี้ให้ทำ handler
- `res.send()` = ส่งข้อความ/HTML, `res.sendFile()` = ส่งไฟล์ (**ต้องเป็น absolute path** จึงใช้ `path.join(__dirname, ...)`)
- `app.listen(port, cb)` = เริ่ม server

> 🔥 เทียบกับ L9: ไม่ต้อง `if (req.url === ...)` เอง ไม่ต้องตั้ง Content-Type เอง Express ทำให้

---

## 3. Environment Variable

**คืออะไร?** คู่ **key-value** สำหรับเก็บการตั้งค่า เช่น `PORT`, `API_KEY`, `DB_HOST`, `DB_NAME` โดย **ประกาศไว้นอก source code** ทำให้แอปรันได้โดยไม่ต้องแก้โค้ด

**ทำไมต้องใช้?**
- รันแอปบนเครื่องอื่น/cloud ได้
- ไม่ต้อง **hardcode** ค่าที่ขึ้นกับเครื่อง
- โค้ดชุดเดียว deploy ได้ทุกที่
- แอป portable และดูแลง่าย
- จัดเก็บในไฟล์ **`.env`**

### 🔹 ตั้งค่า .env (ขั้นตอน)
1. `npm install dotenv`
2. สร้างไฟล์ `.env` ที่ **root** ของโปรเจกต์
3. ใส่ตัวแปรในไฟล์ `.env`
4. โหลด `.env` ในแอป
5. เข้าถึงค่าด้วย **`process.env`**

```text
# .env
PORT=3000
```
```js
/* Import and use Environmental Variable */
const dotenv = require("dotenv");
dotenv.config();          // โหลดตัวแปรทั้งหมดจาก .env เข้า process.env

app.listen(process.env.PORT, function () {
  console.log("Server listening at Port " + process.env.PORT);
});
```

> 📌 **dotenv** = zero-dependency module ที่โหลด environment variables จาก `.env` เข้า `process.env`
> ⚠️ ต้องเรียก `dotenv.config()` **ก่อน** ใช้ `process.env.X` ไม่งั้นได้ `undefined`

---

## 4. Cookies and Sessions

### 🔹 ทำไมต้องมี

บางครั้งต้องเก็บข้อมูลชั่วคราวไว้ส่งต่อระหว่างหน้า มี 2 วิธี:
- **Cookies** → เก็บที่ **ฝั่ง client**
- **Session** → เก็บที่ **ฝั่ง server**

### 🔹 HTTP เป็น "Stateless"

> 📌 HTTP is **stateless**: ไม่มีความเชื่อมโยงระหว่าง request สองครั้งใด ๆ

**cookie ทำให้เกิด stateful session ได้** คือให้ request แต่ละครั้งใช้ context/state เดียวกัน

**ตัวอย่างเปรียบเทียบ:** พนักงานความจำสั้น จำลูกค้าไม่ได้เลย → ให้ลูกค้าถือ "บัตรคิว" (cookie) มาทุกครั้ง พนักงานจะรู้ว่าเป็นคนเดิม

---

## 5. Cookies

> 📌 **Cookie:** ข้อมูลชิ้นเล็กที่ **server ส่งไปให้ browser** browser อาจเก็บไว้แล้ว **ส่งกลับมาพร้อม request ครั้งถัดไปที่ server เดิม**

**ใช้บอกว่า 2 request มาจาก browser เดียวกัน** ใช้งานหลัก 3 อย่าง: **Session management, Personalization, Tracking**

**กลไก**
- cookie เป็นส่วนขยายของ HTTP protocol
- ใน Node ส่ง header **`Set-Cookie`** ไปกับ response
- browser เก็บ cookie แล้วส่งกลับใน header **`Cookie`** ทุก request ไป server เดิม → เราอ่านได้
- ถ้าใส่ **unique identifier** ใน cookie จะระบุตัว client และผูก request ของเขาเข้าด้วยกันได้

### 🔹 Getting Started
```bash
npm install cookie-parser
```
**cookie-parser** = **middleware** ที่แยก (parse) cookie ที่แนบมากับ request ให้อยู่ใน `req.cookies`

```js
const cp = require("cookie-parser");
router.use(cp());   // ลงทะเบียน middleware
```

### 🔹 Creating a Cookie — `res.cookie(name, value)`

```js
const express = require("express");
const app = express();
const dotenv = require('dotenv');
const router = express.Router();
app.use("/", router);

const cp = require("cookie-parser");
router.use(cp());

router.get('/setCookie', function (req, res) {
  console.log('About to create a cookie');
  res.cookie('cookie#1', 'chocolate-chip-cookie');
  res.cookie('cookie#2', 'peanut-butter-cookie');
  res.send('Cookie has been created and hiding at your browser!!');
});
```
ดู cookie ที่สร้างได้ใน browser: **Developer tool > Application > Cookies**

### 🔹 Retrieve Cookies — `req.cookies`

```js
router.get('/showCookie', (req, res) => {
  console.log(req.cookies);
  res.send(req.cookies);
});
// { 'cookie#1': 'chocolate-chip-cookie', 'cookie#2': 'peanut-butter-cookie' }
```

### 🔹 Retrieve Specific Cookie — `req.cookies["name"]`

```js
router.get('/restrict', (req, res) => {
  const mycookie1 = req.cookies['cookie#1'];
  const mycookie2 = req.cookies['cookie#2'];
  if (mycookie1 && mycookie2)   // ถ้ามีครบทั้งสอง
    res.send('You are allowed to see the content');
  else
    res.status(401).end('Error 401: You need to have our secret cookies');
});
```
ใช้ cookie ทำ **restricted content**: ถ้าเคยเข้า `/setCookie` มาก่อน (มี cookie ครบ) จะเห็นเนื้อหา ถ้าไม่เคยหรือถูกลบ → **401**

### 🔹 More about Cookies

```js
// หมดอายุหลัง 360000 ms นับจากตอนที่ตั้ง (กำหนดเป็นเวลา)
res.cookie(name, "value", {expire: 360000 + Date.now()});
// อายุ 360000 ms (กำหนดเป็นระยะเวลา)
res.cookie(name, "value", {maxAge: 360000});
// ลบ cookie
res.clearCookie("username");
```

### 🔹 ตัวอย่าง Login / Logout ด้วย cookie

```js
router.get('/', (req, res) => {
  res.sendFile(path.join(`${__dirname}/html/login.html`));   // แสดงฟอร์ม login
});

// Form submitted via HTTP GET
router.get("/login", function (req, res) {
  /* Create a cookie as a token
     Note: In practice you need to verify the username/password first */
  res.cookie('tokenID', 'some-random-session-ID');
  res.send('Login !!');
});

router.get('/restrict', (req, res) => {
  const tokenID = req.cookies['tokenID'];
  if (tokenID) res.send('Valid Only the ones with cookie only');
  else res.status(401).end('Error 401: You need to have our secret cookies');
});

router.get("/logout", function (req, res) {
  res.clearCookie("tokenID");          // logout = ลบ cookie token
  res.send('Logout !!');
});
```

**สรุป cookie:** cookie (และ URL parameter) ใช้ขนข้อมูลระหว่าง client กับ server ได้ ถูกส่งไปกับทุก request **แต่อ่านได้และอยู่ฝั่ง client** → ถ้าอยากเก็บข้อมูลที่ client มองไม่เห็นล่ะ? → **Session**

---

## 6. Session

**คืออะไร?** ใช้เก็บข้อมูลที่ **ทุกหน้าในเว็บแอปเดียวกันใช้ได้** เก็บได้ทั้งข้อมูล server และข้อมูล client แต่ **เก็บที่ฝั่ง server (client แก้ไม่ได้)**

### 🔹 Getting Started
```bash
npm install express-session
```
**express-session** = middleware จัดการ session และเก็บใน session storage (ค่าเริ่มต้นคือ **MemoryStore**)

> ⚠️ **MemoryStore** ออกแบบมาเพื่อ debug/พัฒนา **ไม่ใช่สำหรับ production** — production ต้องใช้ session store อื่นที่รองรับ

### 🔹 ตั้งค่า express-session

```js
const session = require('express-session');
router.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: true,
}));
```
```text
# .env
PORT=3000
SESSION_SECRET=mysecretkey
```

| Option | ความหมาย |
|---|---|
| `secret` | ข้อความลับสำหรับ **sign session ID** (ควรเป็นสตริงสุ่ม) |
| `resave: false` | บันทึก **เฉพาะเมื่อ session ถูกแก้ไข** |
| `saveUninitialized: true` | **สร้าง session ให้ทุก user** แม้ยังไม่ได้ใช้ |

### 🔹 Create/Get Session — `req.session.<name>`

```js
router.get("/pageview", function (req, res) {
  if (req.session.numviews) {          // มี numviews แล้ว
    req.session.numviews++;
    console.log(`Visited ${req.session.numviews} times`);
  } else {                             // ครั้งแรก สร้าง numviews
    req.session.numviews = 1;
    console.log("First time visited");
  }
  res.send("Session created!!!");
});
```
ผล: ครั้งแรก "First time visited" แล้ว "Visited 2 times", "3 times", ... — session **เข้าถึงได้จากทุก route** ด้วย `req.session.<name>`

> ⚠️ สไลด์แสดงว่า **nodemon restart → session หาย** กลับไป "First time visited" เพราะ MemoryStore เก็บใน RAM

### เปรียบเทียบ Cookie vs Session

| เรื่อง | Cookie | Session |
|---|---|---|
| เก็บที่ | **Client (browser)** | **Server** |
| package | `cookie-parser` | `express-session` |
| สร้าง | `res.cookie(name, value)` | `req.session.name = value` |
| อ่าน | `req.cookies` / `req.cookies['name']` | `req.session.name` |
| ลบ | `res.clearCookie(name)` | (ลบ property / ทำลาย session) |
| ความปลอดภัย | client อ่าน/แก้ได้ | client มองไม่เห็นข้อมูล |

> ⭐ จำง่าย ๆ: **res**.cookie สร้าง (ส่งออกไปให้ browser) / **req**.cookies อ่าน (browser ส่งกลับมา)

---

## 7. Node.js + Database

### 🔹 Getting Started
- มีหลาย package สำหรับต่อ MySQL; **ในวิชานี้ใช้ `mysql2`**
```bash
npm install mysql2
```
- ใช้ต่อฐานข้อมูลและจัดการข้อมูลด้วย SQL: query แสดงข้อมูล, insert, update, delete

### 🔹 RECAP DB
- **Database** = ชุดข้อมูลที่จัดระเบียบ มีหลายแบบ: Relational, Graph, XML ฯลฯ
- **Relational database** = ชุดของ schemas, tables, queries, reports, views และอื่น ๆ

### 🔹 ตัวอย่างฐานข้อมูล tinycollege (ความสัมพันธ์ 9 ข้อ)
1. อาจารย์ทุกคนทำงานในภาควิชา
2. ทุกภาควิชามีอาจารย์เป็นหัวหน้าภาค
3. นักศึกษาทุกคนเรียนในภาควิชา
4. นักศึกษาทุกคนมีอาจารย์ที่ปรึกษา
5. Class สอนโดยอาจารย์
6. Course เปิดโดยภาควิชา
7. นักศึกษาลงหลาย class
8. Class มีนักศึกษาหลายคนลง
9. Class เป็นของ course หนึ่ง

### 🔹 SQL
> 📌 ภาษามาตรฐานสำหรับเก็บ จัดการ และดึงข้อมูลในฐานข้อมูล และจัดการฐานข้อมูลใน DBMS

คำสั่งพื้นฐาน: **INSERT, UPDATE, DELETE, SELECT, WHERE, GROUP BY, HAVING, ORDER BY**

### 🔹 MySQL Server & Workbench
- ติดตั้ง **MySQL Server** และ **MySQL Workbench** (ใช้จัดการฐานข้อมูลผ่าน GUI; ต้องใช้ root account)

### 🔹 Prepare DB and User (ขั้นตอน)
1. สร้าง DB/table/attribute ผ่าน GUI หรือรัน script (เช่น `tinycollege.sql`)
2. สร้าง **user account** สำหรับเชื่อมต่อ: Username, Password, Host (`localhost`), privilege (SQL commands, DDL commands)
3. ใน Workbench: **Server > Users and Privileges > Add Account**
4. ตั้ง **schema privilege** ว่า user ทำ SQL อะไรได้บ้าง

### 🔹 ตั้งโปรเจกต์ DB + Node
```bash
npm init
npm install express
npm install nodemon
npm install dotenv
npm install mysql2
```
สร้าง `app.js` (routing + server) และ `.env` เช่น `PORT, DB_USER, DB_PWD, DB_HOST`

### 🔹 Connection to DB

```text
# .env
PORT=3000
MYSQL_HOST=localhost
MYSQL_USERNAME=<YOUR USERNAME>
MYSQL_PASSWORD=*************
MYSQL_DATABASE=tinycollege
```
```js
const mysql = require('mysql2');
var connection = mysql.createConnection({
  host     : process.env.MYSQL_HOST,
  user     : process.env.MYSQL_USERNAME,
  password : process.env.MYSQL_PASSWORD,
  database : process.env.MYSQL_DATABASE
});

connection.connect(function(err){
  if (err) throw err;
  console.log(`Connected DB: ${process.env.MYSQL_DATABASE}`);
});
```

### 🔹 SELECT แล้วส่งเป็น JSON

```sql
SELECT course_code, course_name
FROM Course
WHERE dept_code='CIS'      -- 6 row(s) returned
```
```js
router.get("/cis-courses", function (req, res) {
  let sql = `SELECT course_code, course_name
             FROM Course WHERE dept_code='CIS'`;
  connection.query(sql, function (error, results) {
    if (error) throw error;
    console.log(`${results.length} rows returned`);
    return res.send(results);      // results เป็น array → ส่งเป็น JSON
  });
});
```

### 🔹 INSERT

```js
/* Note: ตัวอย่างนี้ใช้ GET เพื่อ insert เพื่อความง่าย ปกติใช้ POST */
router.get("/add241", function (req, res) {
  let sql = `INSERT INTO Course
             VALUES('CIS-223','Introduction to Web Development',3,'CIS')`;
  connection.query(sql, function (error, results) {
    if (error) throw error;
    console.log(`${results.affectedRows} record inserted`);
    return res.redirect("/cis-course");   // กลับไปหน้าแสดงทั้งหมด
  });
});
```

| ผลจาก query | ใช้กับ | ได้อะไร |
|---|---|---|
| `results.length` | SELECT | จำนวนแถวที่ได้ |
| `results.affectedRows` | INSERT/UPDATE/DELETE | จำนวนแถวที่ถูกเปลี่ยน |

### 🔹 [Extra] mysql2
- ศึกษา UPDATE, DELETE เพิ่มที่ w3schools
- `mysql2` รองรับ **Promise API** และใช้กับ **async/await** ได้

---

## จุดที่มักสับสน

| เรื่อง | A | B |
|---|---|---|
| `expire` vs `maxAge` | เวลาหมดอายุ (`360000 + Date.now()`) | ระยะเวลา (`360000`) |
| `res.send(results)` vs `res.redirect(url)` | ส่งข้อมูลกลับ | พาไปอีก route |
| `npm init` vs `npm install` | สร้าง package.json | ติดตั้ง package |
| 401 | Unauthorized — ไม่มีสิทธิ์ (ไม่มี cookie) | ต่างจาก 404 ที่หาไม่เจอ |

## จุดเน้นสำหรับสอบ (เขียนโค้ด)
- 🔥 โครง Express: `express()`, `Router()`, `app.use(router)`, `router.get`, `res.send/sendFile`, `app.listen`
- 🔥 `.env` + `dotenv.config()` + `process.env.X`
- 🔥 cookie: `cookie-parser`, `res.cookie`, `req.cookies['x']`, `res.clearCookie`, ตรวจสิทธิ์แล้ว `res.status(401)`
- 🔥 session: `express-session` + options 3 ตัว + `req.session.x`
- 🔥 `mysql.createConnection({...})`, `connection.connect`, `connection.query(sql, (error, results) => ...)`
- ⭐ ตาราง npm commands, start script `nodemon app.js`
- ⚠️ MemoryStore ไม่ใช่ของ production

## ✅ Coverage Check
- [x] Objective / Outline / Recap Express (template engine)
- [x] npm commands, project setup, start script, routing recap
- [x] Environment variable, .env setup, dotenv
- [x] Cookie & session intro, stateless, cookies def, cookie-parser, create/retrieve/specific, expire/maxAge/clear, login/logout
- [x] Session, express-session, MemoryStore, options, pageview
- [x] DB: mysql2, recap DB, tinycollege, SQL, Workbench, prepare user, project, connection, SELECT, INSERT, extra
