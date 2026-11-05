# WebLab 11 · Express + MySQL: ค้นหาอาจารย์

## โจทย์ในหนึ่งประโยค

สร้าง server ด้วย **Express** ที่ port **3030** ต่อฐานข้อมูล **tinycollege** แสดงฟอร์มค้นหาอาจารย์ด้วย Employee Number แล้วตอบหน้า `found.html` หรือ `notfound.html` ตามผลค้นหา

> 🔗 ใช้ความรู้จาก L11 ทั้งหมด: Express routing, `.env`, `mysql2`, `res.sendFile` + เพิ่มเรื่อง **รับค่าจากฟอร์ม POST**

---

## 1. เตรียมฐานข้อมูล

1. รัน `tinycollege.sql` สร้างฐานข้อมูล
2. สร้าง user ใน MySQL: username **`itds242`**, password **`ict555`**
   - schema privilege เข้าถึง `tinycollege`
   - สิทธิ์ `SELECT`, `INSERT`, `UPDATE`, `DELETE`
3. ทดสอบ `SELECT * FROM Professor;` ต้องได้ **22 rows**

## 2. เตรียมโปรเจกต์

1. ตรวจ `node --version`
2. สร้างโฟลเดอร์ `XX87XXX-expressdb` แล้วเปิดใน VS Code
3. `npm init` (entry point = `app.js`, author = ชื่อ + รหัส)
4. สร้าง `app.js`
5. `npm install express nodemon mysql2 dotenv`
6. แก้ `scripts` เป็น `"start": "nodemon app.js"`
7. วางไฟล์ HTML ไว้ในโฟลเดอร์ `html/`: `found.html`, `notfound.html`, `searchProf.html`

```text
XX87XXX-expressdb
 ├─ html/
 │   ├─ found.html
 │   ├─ notfound.html
 │   └─ searchProf.html
 ├─ node_modules/
 ├─ .env
 ├─ app.js
 ├─ package-lock.json
 └─ package.json
```

## 3. ข้อกำหนดของ server

| Route | ต้องทำ | log ที่ต้องมี |
|---|---|---|
| เริ่ม server | listen port 3030 + ต่อ DB | `Server listening on port: 3030` |
| `GET /` | ส่ง `searchProf.html` | `Request at /` |
| `POST /form-submit` | รับ emp_num → ค้น `Professor.emp_num` | `Request at /form-submit`, `Form submitted with Emp Num: 297`, `1 rows returned`, `Found 297` |
| ไม่พบ | ส่ง `notfound.html` | `0 rows returned`, `Not found` |

- ค่าตั้งค่า (`PORT`, `DB_NAME`, `DB_HOST`, `DB_USER`, ...) ต้องอยู่ใน `.env`
- ฟอร์มใน `searchProf.html` ต้องเป็น `method="POST"` และ `action="/form-submit"`
- ส่งงาน: `app.js` และ `searchProf.html`

> ⚠️ README ข้อ "หากไม่พบ" เขียนว่า response เป็น `found.html` แต่ตัวอย่างผลลัพธ์ที่ 5 แสดง `notfound.html` — ตามตัวอย่างควรส่ง **`notfound.html`**

## 4. ตัวอย่างคำตอบ (แนวทาง)

```text
# .env
PORT=3030
DB_HOST=localhost
DB_USER=itds242
DB_PASS=ict555
DB_NAME=tinycollege
```

```html
<!-- searchProf.html (ส่วนฟอร์ม) -->
<form action="/form-submit" method="POST">
  <input type="text" name="empNum" id="empNum">
  <button type="submit">Submit</button>
</form>
```

```js
// app.js
const express = require("express");
const path = require("path");
const mysql = require("mysql2");
const dotenv = require("dotenv");
dotenv.config();

const app = express();
const router = express.Router();
app.use(express.urlencoded({ extended: true }));   // อ่านข้อมูลจากฟอร์ม POST
app.use(router);

const connection = mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
});
connection.connect((err) => {
  if (err) throw err;
  console.log(`Connected DB: ${process.env.DB_NAME}`);
});

router.get("/", (req, res) => {
  console.log("Request at /");
  res.sendFile(path.join(__dirname, "html", "searchProf.html"));
});

router.post("/form-submit", (req, res) => {
  console.log("Request at /form-submit");
  const empNum = req.body.empNum;            // ตรงกับ name="empNum" ในฟอร์ม
  console.log(`Form submitted with Emp Num: ${empNum}`);
  const sql = "SELECT * FROM Professor WHERE emp_num = ?";
  connection.query(sql, [empNum], (error, results) => {
    if (error) throw error;
    console.log(`${results.length} rows returned`);
    if (results.length > 0) {
      console.log(`Found ${empNum}`);
      res.sendFile(path.join(__dirname, "html", "found.html"));
    } else {
      console.log("Not found");
      res.sendFile(path.join(__dirname, "html", "notfound.html"));
    }
  });
});

app.listen(process.env.PORT, () => {
  console.log(`Server listening on port: ${process.env.PORT}`);
});
```

**อธิบายจุดที่มักพลาด**
- **`express.urlencoded({ extended: true })`**: ฟอร์ม HTML ส่งข้อมูลแบบ urlencoded ถ้าไม่มีบรรทัดนี้ `req.body` จะเป็น `undefined` (สไลด์ L11 ไม่ได้แสดง แต่ต้องใช้เมื่อรับฟอร์ม POST)
- ชื่อใน `req.body.xxx` ต้องตรงกับ **`name`** ของ input
- ใช้ **`?` placeholder** ไม่ต่อ string เอง (กัน SQL injection)
- ตัดสินใจ found/notfound จาก **`results.length`**

## 5. ฝึกต่อยอด (เตรียมสอบ)
- เปลี่ยนเป็นค้นหาจากชื่ออาจารย์ (`LIKE ?` กับ `%name%`)
- ส่งผลเป็น JSON แทนไฟล์ HTML (`res.json(results)`)
- เพิ่ม route insert อาจารย์ด้วย POST
