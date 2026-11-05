# L11 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- HTTP **stateless** → ใช้ **cookie** (เก็บ client) หรือ **session** (เก็บ server) จำผู้ใช้
- `.env` เก็บค่าตั้งค่านอกโค้ด → portable, ไม่ hardcode
- `mysql2` ต่อ MySQL แล้ว query ด้วย callback `(error, results)`

## 📌 โครงที่ต้องเขียนได้จากความจำ

```js
const express = require("express");
const path = require("path");
const dotenv = require("dotenv");
dotenv.config();
const app = express();
const router = express.Router();
app.use(router);

// cookie
const cp = require("cookie-parser");
router.use(cp());
// session
const session = require("express-session");
router.use(session({ secret: process.env.SESSION_SECRET, resave: false, saveUninitialized: true }));
// db
const mysql = require("mysql2");
const connection = mysql.createConnection({
  host: process.env.MYSQL_HOST, user: process.env.MYSQL_USERNAME,
  password: process.env.MYSQL_PASSWORD, database: process.env.MYSQL_DATABASE
});
connection.connect(err => { if (err) throw err; console.log("Connected DB"); });

router.get("/", (req, res) => res.sendFile(path.join(__dirname, "html/login.html")));
router.get("/login", (req, res) => { res.cookie("tokenID", "abc"); res.send("Login !!"); });
router.get("/restrict", (req, res) => {
  if (req.cookies["tokenID"]) res.send("OK");
  else res.status(401).end("Error 401");
});
router.get("/logout", (req, res) => { res.clearCookie("tokenID"); res.send("Logout !!"); });
router.get("/pageview", (req, res) => {
  if (req.session.numviews) req.session.numviews++; else req.session.numviews = 1;
  res.send("Session created!!!");
});
router.get("/cis-courses", (req, res) => {
  connection.query("SELECT course_code, course_name FROM Course WHERE dept_code='CIS'",
    (error, results) => { if (error) throw error; res.send(results); });
});

app.listen(process.env.PORT, () => console.log(`Server listening on port: ${process.env.PORT}`));
```

## ⭐ ต้องจำ
| อยากทำ | ใช้ |
|---|---|
| สร้างโปรเจกต์ / default | `npm init` / `npm init -y` |
| ติดตั้งทั้งหมดจาก package.json | `npm install` |
| start script | `"start": "nodemon app.js"` แล้ว `npm start` |
| อ่าน env | `process.env.PORT` (หลัง `dotenv.config()`) |
| ตั้ง cookie มีอายุ | `{maxAge: 360000}` หรือ `{expire: 360000 + Date.now()}` |
| จำนวนแถว SELECT | `results.length` |
| แถวที่ถูกแก้ | `results.affectedRows` |
| session options | secret (sign ID), resave:false (save เมื่อแก้), saveUninitialized:true (สร้างให้ทุกคน) |

## ⚠️ มักสับสน
- **res**.cookie = สร้าง · **req**.cookies = อ่าน
- MemoryStore = ใช้ตอนพัฒนาเท่านั้น, restart แล้ว session หาย
- `res.sendFile` ต้องใช้ absolute path
- 401 = ไม่มีสิทธิ์ · 404 = ไม่พบ
- ตัวอย่าง insert ในสไลด์ใช้ GET เพื่อความง่าย แต่ปกติใช้ **POST**
- Template engine (Pug/EJS) แทนตัวแปรใน template แล้วได้ HTML
