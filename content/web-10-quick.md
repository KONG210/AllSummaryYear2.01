# L10 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- **Express** = minimal, flexible framework ของ Node: routing, API, template engine (Pug/EJS แทนตัวแปรแล้วได้ HTML)
- **Middleware** = function ที่เข้าถึง `req`, `res` (และ `next`)
- URL จริง = mount path (`app.use("/product", r)`) + path ใน router (`r.get("/add")`) = `/product/add`

## 📌 โครงที่ต้องเขียนได้จากความจำ
```js
const express = require('express');
const path = require('path');
const port = 3000;
const app = express();
const router = express.Router();
app.use(router);
router.use(express.json());
router.use(express.urlencoded({ extended: true }));         // ต้องมีสำหรับ POST form
app.use('/', express.static(path.join(__dirname, 'public'))); // static: อ้าง /mystyle.css

router.get('/', (req, res) => res.send('Hello World from Root!'));
router.get('/my-form', (req, res) => res.sendFile(path.join(__dirname, 'html/myForm.html')));

router.get('/form-get', (req, res) => {                   // <form action="/form-get" method="GET">
  res.send(`Form submitted by ${req.query.fname} ${req.query.lname} with ${req.method}`);
});
router.post('/form-post', (req, res) => {                 // <form action="/form-post" method="POST">
  res.send(`Form submitted by ${req.body.fname} ${req.body.lname} with ${req.method}`);
});

router.use((req, res, next) => {                           // วางท้ายสุด = 404
  res.status(404).send('Invalid Path');
});
app.listen(port, () => console.log(`Server listening on port: ${port}`));
```

## ⭐ ต้องจำ
| อยากทำ | ใช้ |
|---|---|
| สร้างโปรเจกต์ | `npm init` (entry point = app.js) → package.json |
| ติดตั้ง | `npm install express` · `npm install nodemon` → dependencies + package-lock.json |
| รันอัตโนมัติ | `"start": "nodemon app.js"` แล้ว `npm start` (`rs` = restart) |
| ส่งไฟล์ | `res.sendFile(path.join(__dirname, 'html/x.html'))` (ต้อง absolute) |
| GET form | `req.query.<name>` (ข้อมูลอยู่ใน URL `?fname=John&lname=Doe`) |
| POST form | `req.body.<name>` + `express.urlencoded({extended:true})` |
| JSON body | `express.json()` |
| static | `express.static(...)` แล้วอ้าง path เริ่มจากโฟลเดอร์ public |
| 404 | `router.use((req,res,next) => res.status(404).send(...))` ท้ายสุด |

## ⚠️ มักสับสน
- `action` ต้องตรง path ของ route · `method` ต้องตรง `router.get/post` · `name` ของ input = key ใน query/body
- ลืม urlencoded → `req.body` undefined
- `router.use` จับทุก method · วาง 404 ก่อน route อื่น = ทุกอย่าง 404
- relative path `../public/x.css` ใช้ไม่ได้กับ sendFile → ใช้ express.static
- `href` ใน navbar ต้องตรงกับ route
- body-parser รวมอยู่ใน Express ตั้งแต่ 4.16
