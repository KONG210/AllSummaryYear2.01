# L13 · Web Services with Node.js

## ภาพรวมของบท

**เป้าหมาย:** สร้าง Web Service ด้วย Node.js · อธิบาย **CORS** · อธิบาย **JWT** · **ทดสอบ** Web Service

**ลำดับ:** สร้าง WS ด้วย Node.js + MySQL → ทดสอบด้วย Postman → หน้าเว็บเรียก WS → CORS → Token-Based Authentication → JWT → เรียก REST WS จาก Node.js ด้วย Axios → Debug ด้วย VSCode

> 🔗 ภาพใหญ่: L12 สอนทฤษฎี REST → L13 ลงมือสร้าง **Student Web Service** (CRUD ผ่าน GET/POST/PUT/DELETE ตอบเป็น JSON) → หน้า HTML คนละ origin เรียกไม่ได้เพราะ same-origin policy → เปิดด้วย **CORS** → ป้องกันไม่ให้ใครก็เรียกได้ด้วย **JWT** → และเรียก service ของคนอื่นจากฝั่ง Node ด้วย **Axios**

---

## 1. Key REST principles และเทคโนโลยี

**Key REST principles (5 ข้อ)**
1. ให้ทุก "สิ่ง" มี **ID**
2. **เชื่อมโยง** สิ่งต่าง ๆ เข้าด้วยกัน
3. ใช้ **standard methods**
4. resource มีได้ **หลาย representation**
5. สื่อสารแบบ **stateless**

**REST: Technologies** — HTTP เป็นพื้นฐาน, Node.js, XML และ JSON สำหรับแลกเปลี่ยนข้อมูล, JavaScript ฝั่ง client (browser)

---

## 2. สร้าง REST Web Service ด้วย Node.js และ MySQL

### 🔹 ขั้นตอน
1. สร้าง database
2. สร้าง table
3. ติดตั้ง package ที่จำเป็น: **dotenv, express, mysql2, nodemon**
4. สร้างไฟล์ `.env`
5. สร้าง **`studentservices.js`** เข้าถึงฐานข้อมูล
6. สร้าง HTML (**`studentform.html`**) เพื่อโต้ตอบกับ service

**Overview Architecture:** หน้าเว็บ/client ส่ง **GET, POST, PUT, DELETE** ไปที่ Student Web service (studentservices.js) และได้ **JSON** กลับ

### 🔹 .env
```text
# .env
DB_user = "itcs212"
DB_pass = "itcs212"
DB_name = "itcs212"
host = "localhost"
```

### 🔹 studentservices.js — ไฟล์หลักของ service
มี **การเชื่อมต่อฐานข้อมูล** และ **CRUD operations ทั้งหมด**

| Operation | SQL | HTTP | RESTful WS |
|---|---|---|---|
| **Create** | INSERT | PUT / POST | **POST** |
| **Read** (Retrieve) | SELECT | GET | **GET** |
| **Update** (Modify) | UPDATE | PUT / POST / PATCH | **PUT** |
| **Delete** (Destroy) | DELETE | DELETE | **DELETE** |

> 🔥 ต้องจำคอลัมน์สุดท้าย: Create=POST, Read=GET, Update=PUT, Delete=DELETE

### 🔹 Database connection
```js
let dbConn = mysql.createConnection({
  host: process.env.host,
  user: process.env.DB_user,
  password: process.env.DB_pass,
  database: process.env.DB_name,
});
```

> ℹ️ สไลด์ไม่ได้แสดงส่วนหัวไฟล์ทั้งหมด แต่เนื่องจาก route อ่าน `req.body` ที่เป็น JSON server ต้องมี middleware แปลง JSON body (เช่น `express.json()`) — ตรวจกับโค้ดตัวอย่างของอาจารย์อีกครั้ง

### 🔹 Insert Student — `POST /student`

**Request body**
```json
{ "student": { "STU_ID": 99998, "STU_FNAME": "XXXXX", "STU_LNAME": "YYYYY", "STU_AGE": 35 } }
```
```js
route.post('/student', function (req, res) {
  let student = req.body.student;
  console.log(student);
  if (!student) {
    return res.status(400).send({ error: true, message: 'Please provide student information' });
  }
  dbConn.query("INSERT INTO student SET ? ", student, function (error, results) {
    if (error) throw error;
    return res.send({ error: false, data: results.affectedRows,
                      message: 'New student has been created successfully.' });
  });
});
```
**Response:** `{ "error": false, "data": 1, "message": "New student has been created successfully." }`

**จุดสำคัญ**
- `req.body.student` = ข้อมูลจาก JSON body
- ไม่มีข้อมูล → **400 Bad Request**
- **`INSERT INTO student SET ?`** แล้วส่ง object → mysql2 แปลง object เป็น `col = value` ให้ (ชื่อ key ต้องตรงชื่อคอลัมน์)
- `?` = **placeholder** ใส่ค่าอย่างปลอดภัย

### 🔹 Update Student — `PUT /student`

```js
route.put('/student', function (req, res) {
  let student_id = req.body.student.student_id;
  let student = req.body.student;
  if (!student_id || !student) {
    return res.status(400).send({ error: student, message: 'Please provide student information' });
  }
  dbConn.query("UPDATE student SET ? WHERE STU_ID = ?", [student, student_id],
    function (error, results) {
      if (error) throw error;
      return res.send({ error: false, data: results.affectedRows,
                        message: 'Student has been updated successfully.' });
    });
});
```
- **มี `?` สองตัว → ส่ง array `[student, student_id]`** ตามลำดับ
- ⚠️ สไลด์ใช้ `req.body.student.student_id` ในโค้ด แต่ตัวอย่าง body ใช้ key `STU_ID` — ไม่ตรงกัน ต้องให้ชื่อ key ใน body ตรงกับที่โค้ดอ่าน

### 🔹 Delete Student — `DELETE /student`
**Body:** `{ "student_id": 99998 }`
```js
route.delete('/student', function (req, res) {
  let student_id = req.body.student_id;
  if (!student_id) {
    return res.status(400).send({ error: true, message: 'Please provide student_id' });
  }
  dbConn.query('DELETE FROM student WHERE STU_ID = ?', [student_id], function (error, results) {
    if (error) throw error;
    return res.send({ error: false, data: results.affectedRows,
                      message: 'Student has been deleted successfully.' });
  });
});
```

### 🔹 Select a student — `GET /student/4`
```js
route.get('/student/:id', function (req, res) {
  let student_id = req.params.id;       // รับค่าจาก URL parameter
  if (!student_id) {
    return res.status(400).send({ error: true, message: 'Please provide student id.' });
  }
  dbConn.query('SELECT * FROM student where STU_ID=?', student_id, function (error, results) {
    if (error) throw error;
    return res.send({ error: false, data: results[0], message: 'Student retrieved' });
  });
});
```
- **`:id` ใน path** → อ่านด้วย **`req.params.id`**
- `results[0]` = แถวแรก (คนเดียว)

### 🔹 Select all students — `GET /students`
```js
route.get('/students', function (req, res) {
  dbConn.query('SELECT * FROM student', function (error, results) {
    if (error) throw error;
    return res.send({ error: false, data: results, message: 'Student list.' });
  });
});
```
- ส่ง `results` ทั้ง array

**Output:** ทุก output จาก studentservices.js อยู่ในรูป **JSON** รูปแบบเดียวกัน `{ error, data, message }`

| ข้อมูลมาจาก | อ่านด้วย | ตัวอย่าง |
|---|---|---|
| URL path `/student/:id` | `req.params.id` | `GET /student/4` |
| JSON body | `req.body.xxx` | POST/PUT/DELETE |
| query string `?q=x` | `req.query.q` | (จาก Express ทั่วไป) |

---

## 3. Testing ด้วย Postman

- **Postman** = แพลตฟอร์มสำหรับพัฒนา API ช่วยทุกขั้นตอนของการสร้าง API และทำงานร่วมกัน
- ดาวน์โหลดที่ postman.com/downloads
- ทดสอบโดย **สร้าง request**: เลือก method (GET/POST/PUT/DELETE), ใส่ URL เช่น `http://localhost:3000/student`, ใส่ body เป็น **raw JSON** สำหรับ POST/PUT/DELETE แล้วกด Send ดู response

> ⭐ Postman ช่วยทดสอบ PUT/DELETE ที่พิมพ์ใน address bar ของ browser ไม่ได้ (browser ส่งได้แค่ GET)

---

## 4. Student Form: JavaScript เรียก WS

`studentform.html` ใช้โต้ตอบกับ studentservices.js ผ่าน `callStudentService.js`

```js
async function callStudentWS(url, method, sentData = {}) {
  let data;
  if (method == "select") {
    let response = await fetch(url, { method: 'GET' });
    data = await response.json();
  }
  return data;
}

let selectBtnRef = document.querySelector("#select");
selectBtnRef.addEventListener("click", () => {
  STU_ID = STU_IDTxtRef.value;
  callStudentWS("http://localhost:3000/student/" + STU_ID, "select").then((data) => {
    console.log(data);
    if (data) {
      alert(data.message);
      STU_IDTxtRef.value = data.data.STU_ID;
      STU_FNAMETxtRef.value = data.data.STU_FNAME;
      STU_LNAMETxtRef.value = data.data.STU_LNAME;
      STU_AGETxtRef.value = data.data.STU_AGE;
    }
  });
});
```
- `async/await` ทำให้ fetch อ่านง่าย: `await fetch(...)` ได้ Response → `await response.json()` ได้ object
- `data.data.STU_ID` = `data` คือ JSON ทั้งก้อน, `.data` คือ field `data` ข้างใน

---

## 5. CORS (Cross-Origin Resource Sharing)

### 🔹 Cross-domain issues

> 📌 **Same-origin policy:** script ในหน้าเว็บจะเข้าถึงข้อมูลของอีกหน้าได้ **ก็ต่อเมื่อทั้งสองมี origin (domain) เดียวกัน**

> 📌 **CORS:** กลไกที่ **อนุญาต** ให้ resource ที่ถูกจำกัด (เช่น font) บนหน้าเว็บ ถูกขอจาก domain อื่นนอก domain ต้นทางได้

**request แบบ cross-origin จะล้มเหลวถ้าไปที่**
- **domain** ต่างกัน
- **subdomain** ต่างกัน
- **port** ต่างกัน
- **protocol** ต่างกัน

**ตัวอย่าง:** หน้า `http://localhost:3100/studentform.html` เรียก `http://localhost:3000/students` → **port ต่าง** = คนละ origin → ถูกบล็อกถ้า server ไม่เปิด CORS

### 🔹 เปิด CORS ด้วย package `cors`

```bash
npm install --save cors
```
```js
var cors = require('cors');
app.use(cors());          // เปิด CORS ให้ทุก route
```

**เปิดเฉพาะบาง route**
```js
app.get('/student/:id', cors(), function (req, res) {
  res.json({ msg: 'This is CORS-enabled for a Single Route' });
});
```

**กำหนด origin และ method**
```js
let corsOptions = {
  origin: 'http://localhost:3100',
  methods: 'GET,POST,PUT,DELETE'
};
app.use(cors(corsOptions));
```

**หลาย origin ด้วย array (whitelist)**
```js
let whiteList = ['http://localhost:3100', 'http://localhost:3200'];
let corsOptions = { origin: whiteList, methods: 'GET,POST,PUT,DELETE' };
```

**ไม่ใช้ package — ตั้ง header เอง**
```js
res.header("Access-Control-Allow-Origin", "http://localhost:3200");
res.header("Access-Control-Allow-Methods", "GET");
```

> ⭐ CORS ตั้งที่ **server** (ผู้ให้บริการ) ไม่ใช่ที่หน้าเว็บที่เรียก

---

## 6. Token-Based Authentication

**ปัญหา:** ทำอย่างไรให้แอปและ service ปลอดภัย? → **token-based authentication** จัดการสิทธิ์ผู้ใช้อย่างปลอดภัยและมีประสิทธิภาพ

**Flow (8 ขั้นตามสไลด์)**
1. ผู้ใช้ **login** ส่ง `{username, password}`
2. ไปที่ **Authentication Service**
3. **Token Generator** สร้าง token
4. ส่ง **Access Token** กลับ
5. client เก็บ token ไว้ใน **Local Storage / Session Storage / Cookies**
6. ส่ง HTTP request พร้อม header **`Authorization: Bearer <token>`**
7. **Student Web service** ตรวจ token
8. ตอบ **Success หรือ Deny**

---

## 7. JSON Web Tokens (JWT)

> 📌 **JWT:** JSON object ตามมาตรฐานเปิด **RFC 7519** เป็นวิธีที่ปลอดภัยในการส่งข้อมูลระหว่างสองฝ่าย

**ใช้ทำอะไร:** **Authorization**, **Information Exchange**

### 🔹 โครงสร้าง 3 ส่วนคั่นด้วยจุด
```text
xxxxx.yyyyy.zzzzz
Header.Payload.Signature
```

**1. Header** (2 อย่าง)
- ชนิด token (`JWT`)
- อัลกอริทึม sign เช่น **HS256, RS256, HS384**

**2. Payload** = **claims** (ข้อความเกี่ยวกับ entity — มักเป็น user — และข้อมูลเพิ่มเติม) มี 3 แบบ
| Claim | ความหมาย |
|---|---|
| **Registered** | กำหนดไว้แล้ว: **iss** (issuer), **exp** (expiration time), **sub** (subject), **aud** (audience), **iat** (issued at) |
| **Public** | กำหนดเองได้ แต่ควรลงทะเบียนใน **IANA JSON Web Token Registry** เพื่อไม่ชนกัน |
| **Private** | กำหนดเองเพื่อแชร์ระหว่างฝ่ายที่ตกลงกัน |

**3. Signature** = สร้างจาก **encoded header + encoded payload + secret + อัลกอริทึมใน header** แล้ว sign

### 🔹 JWT packages
- **jsonwebtoken** (ใช้ในวิชานี้), express-jwt
- 2 ฟังก์ชันหลัก:

```js
jwt.sign(payload, secretOrPrivateKey, [options, callback])
jwt.verify(token, secretOrPublicKey, [options, callback])
```

**sign — สร้าง token**
- `payload`: object literal, buffer หรือ string ที่เป็น JSON ถูกต้อง
- `secretOrPrivateKey`: secret (สำหรับ HMAC) หรือ PEM key (RSA/ECDSA)
- options: **`algorithm`** (default **HS256**), **`expiresIn`** (วินาที หรือข้อความ เช่น `60`, `"2 days"`, `"10h"`, `"7d"`), `audience`, `issuer`, `header` ฯลฯ

**verify — ตรวจ token**
- `token`: JWT string
- `secretOrPublicKey`
- options: `algorithms` (เช่น `["HS256","HS384"]`), `audience`, `issuer`, **`ignoreExpiration`** (true = ไม่เช็กวันหมดอายุ) ฯลฯ

### 🔹 JWT Implementation (ขั้นตอน)

**1) ตั้ง server**
```bash
npm install --save jsonwebtoken
```
```js
jwt = require("jsonwebtoken");
```
```text
# .env
SECRET = "secret"
```

**2) route login สร้าง token** (เพิ่มปุ่ม Login ในหน้าเว็บ)
```js
router.post("/signin", (req, res) => {
  let user = req.body;
  let jwtToken = jwt.sign({
      email: user.user.email,                 // payload
      userid: user.user.userid.toString()
    }, process.env.SECRET,                    // secret
    { expiresIn: "1h" });                     // option
  res.status(200).json({ token: jwtToken, message: user });
});
```

**3) Middleware ตรวจ token (`middlewares/auth.js`)**
```js
const jwt = require("jsonwebtoken");
module.exports = (req, res, next) => {
  try {
    const token = req.headers.authorization.split(" ")[1];  // "Bearer xxx" → "xxx"
    var decoded = jwt.verify(token, process.env.SECRET);
    console.log(decoded);
    next();                         // ผ่าน → ไป handler ถัดไป
  } catch (error) {
    res.status(401).json({ error: true, message: "Authentication failed!" });
  }
};
```

**4) ใส่ middleware ให้ route ที่ต้องป้องกัน**
```js
authorize = require("./middlewares/auth");
router.get('/students', authorize, function (req, res) {
  dbConn.query('SELECT * FROM student', function (error, results) {
    if (error) throw error;
    return res.send({ error: false, data: results, message: 'Student list.' });
  });
});
```

**5) ฝั่ง client: login ขอ token**
```js
async function callStudentWS(url, method, token = "", sentData = {}) {
  let data;
  if (method == "login") {
    let response = await fetch(url, {
      method: "POST",
      headers: { 'Accept': 'application/json', 'Content-Type': 'application/json' },
      body: JSON.stringify(sentData)
    });
    data = await response.json();
  }
  return data;
}

loginBtnRef.addEventListener("click", () => {
  let user_data = { user: { email: "test@test.com", userid: 1 } };
  callStudentWS("http://localhost:3000/signin", "login", token, user_data)
    .then((data) => { token = data.token; });      // เก็บ token ไว้
});
```

**6) ฝั่ง client: ส่ง token ไปกับ request**
```js
if (method == "select") {
  let response = await fetch(url, {
    method: 'GET',
    headers: { 'Authorization': 'Bearer ' + token }
  });
  data = await response.json();
}
```

**Open question ในสไลด์:** จะเก็บ JWT อย่างปลอดภัยได้อย่างไร? (สไลด์ถามไว้ ไม่ได้เฉลย)

> 🔥 middleware = function `(req, res, next)` ที่คั่นก่อน handler · เรียก `next()` = ไปต่อ · ไม่เรียก/ส่ง 401 = หยุด
> ⚠️ JWT payload **ถูก encode ไม่ได้ถูกเข้ารหัส** — signature มีไว้กันการปลอมแปลง ไม่ได้ซ่อนข้อมูล (ความรู้ทั่วไป ไม่ได้อยู่ในสไลด์)

---

## 8. Calling RESTful WS using Node.js (Axios)

**package ที่ใช้เรียก REST WS จาก Node ได้:** HTTP (standard library), Request, Needle, **Axios**, SuperAgent, Got, Node-fetch — วิชานี้ใช้ **Axios**

```bash
npm install axios
```

**Axios** = library ส่ง HTTP request ใช้ได้ **ทั้ง browser และ Node.js**

**ข้อดี**
- รองรับ browser เก่า
- **ยกเลิก (abort) request ได้**
- ตั้ง **response timeout** ได้
- มี **CSRF protection** ในตัว
- รองรับ upload progress
- **แปลง JSON อัตโนมัติ**
- ใช้ใน Node.js ได้

**Methods**
```js
axios.get(url[, config])
axios.delete(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
axios(config)       // กำหนด method เองใน config
```
- `url` = URL ปลายทาง · `data` = body · `config` = request config เช่น `url`, `method`, **`params`** (URL parameters), `data`

> ⚠️ get/delete ไม่มี `data` parameter · post/put/patch มี `data` ก่อน `config`

### 🔹 ตัวอย่าง: เรียก spoonacular API
```js
const axios = require("axios").default;
const API_key = "Your API Key";
let query = "pizza";
let params = { number: 20, query: query, apiKey: API_key };
let searchURL = "https://api.spoonacular.com/recipes/complexSearch?";

axios.get(searchURL, { params, responseType: "json" })
  .then((res) => {
    console.log(`The number of results: ${res.data.results.length}`);
    console.table(res.data.results);
  })
  .catch((err) => console.log(err));
```
- `params` ถูกต่อเป็น query string ให้อัตโนมัติ
- ข้อมูลอยู่ใน **`res.data`**

### 🔹 ตัวอย่าง: เรียก Student WS
```js
const axios = require('axios').default;
let searchURL = "http://localhost:3000/students";
axios.get(searchURL, { responseType: 'json' })
  .then((res) => { console.log(res.data); });
// ใช้ async/await แทนได้
```

| | fetch | axios |
|---|---|---|
| แปลง JSON | ต้อง `res.json()` เอง | อัตโนมัติ อยู่ใน `res.data` |
| query params | ต่อ string เอง | `{ params }` |
| ใช้ที่ไหน | browser (Node รุ่นใหม่ก็มี) | browser + Node |

---

## 9. Node.js Debugger via VSCode

**ขั้นตอน**
1. เพิ่มโฟลเดอร์โปรเจกต์เข้า **workspace**
2. เลือกเมนู **debug** แถบซ้าย
3. เลือก **create a launch.json file**
4. เลือกโฟลเดอร์งานและเลือก **Node.js**
5. ไฟล์ `launch.json` ถูกสร้าง — แก้ `program` ให้ชี้ไฟล์ที่จะ debug

```json
{
  "type": "node",
  "request": "launch",
  "name": "Launch Program",
  "skipFiles": ["<node_internals>/**"],
  "program": "${workspaceFolder}\\server.js"
}
```
6. ใส่ **breakpoint** ในไฟล์ แล้วเริ่ม debug จากเมนู debug

---

## จุดเน้นสำหรับสอบ (เขียนโค้ด)
- 🔥 เขียน CRUD route 5 ตัว (POST/PUT/DELETE `/student`, GET `/student/:id`, GET `/students`) + `dbConn.query` + `?` placeholder + 400 validation
- 🔥 ตาราง CRUD ↔ SQL ↔ RESTful method
- 🔥 `req.params.id` vs `req.body.x`
- 🔥 `app.use(cors())`, corsOptions (origin, methods, whitelist), header เอง
- 🔥 JWT: `jwt.sign(payload, secret, {expiresIn})`, middleware `verify` + `next()` + 401, header `Authorization: Bearer`
- ⭐ JWT 3 ส่วน, registered claims (iss, exp, sub, aud, iat), 3 ชนิด claim
- ⭐ same-origin: domain / subdomain / port / protocol
- ⭐ axios methods และข้อดี
- 📌 Token-based auth flow 8 ขั้น

## ✅ Coverage Check
- [x] Objectives, REST architecture, contents, key REST principles, technologies
- [x] Creating WS steps, architecture, packages, .env, CRUD table, DB connection
- [x] Insert, Update, Delete, Select one, Select all, Output
- [x] Postman, Student form, callStudentWS, select function
- [x] CORS: same-origin, fails if, cors package, single route, options, whitelist, manual header
- [x] Token-based auth + flow
- [x] JWT: def, use cases, 3 parts, header, payload claims, signature, packages, sign, verify, implementation 1–6, open question
- [x] Axios: packages list, install, advantages, methods, config, spoonacular example, student WS example
- [x] VSCode debugger
