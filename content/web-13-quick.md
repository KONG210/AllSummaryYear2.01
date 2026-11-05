# L13 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- Student WS = Express + mysql2 ตอบ JSON `{ error, data, message }` ทุก route
- **Same-origin policy** บล็อกเรียกข้าม origin → เปิดด้วย **CORS** ที่ server
- **JWT** = token ที่ server sign ให้หลัง login → client ส่ง `Authorization: Bearer <token>` → middleware verify

## ⭐ CRUD ↔ HTTP
| Operation | SQL | RESTful |
|---|---|---|
| Create | INSERT | **POST** |
| Read | SELECT | **GET** |
| Update | UPDATE | **PUT** |
| Delete | DELETE | **DELETE** |

## 📌 โค้ดที่ต้องเขียนได้
```js
route.post('/student', (req, res) => {
  let student = req.body.student;
  if (!student) return res.status(400).send({ error: true, message: 'Please provide student information' });
  dbConn.query("INSERT INTO student SET ?", student, (error, results) => {
    if (error) throw error;
    res.send({ error: false, data: results.affectedRows, message: 'New student has been created successfully.' });
  });
});
route.put('/student', (req, res) => {           // UPDATE student SET ? WHERE STU_ID = ?  , [student, id]
});
route.delete('/student', (req, res) => {        // DELETE FROM student WHERE STU_ID = ?  , [req.body.student_id]
});
route.get('/student/:id', (req, res) => {       // req.params.id → results[0]
});
route.get('/students', (req, res) => {          // SELECT * FROM student → results
});
```

```js
const cors = require('cors');
app.use(cors());                                  // ทุก route
app.use(cors({ origin: ['http://localhost:3100'], methods: 'GET,POST,PUT,DELETE' }));
res.header("Access-Control-Allow-Origin", "http://localhost:3200");   // แบบไม่ใช้ package
```

```js
// login → token
let token = jwt.sign({ email, userid }, process.env.SECRET, { expiresIn: "1h" });
// middleware
module.exports = (req, res, next) => {
  try {
    const token = req.headers.authorization.split(" ")[1];
    jwt.verify(token, process.env.SECRET);
    next();
  } catch (e) { res.status(401).json({ error: true, message: "Authentication failed!" }); }
};
router.get('/students', authorize, handler);
// client
fetch(url, { headers: { 'Authorization': 'Bearer ' + token } });
fetch(url, { method: "POST", headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(data) });
```

```js
axios.get(url, { params: { query: "pizza" }, responseType: "json" }).then(res => res.data);
```

## ⭐ ต้องจำ
- Cross-origin ล้มถ้าต่าง: **domain, subdomain, port, protocol**
- JWT = **Header.Payload.Signature** · Header: type + alg (HS256/RS256/HS384) · default alg = **HS256**
- Registered claims: **iss, exp, sub, aud, iat** · claim 3 แบบ: registered / public (IANA) / private
- Signature = encoded header + encoded payload + secret + algorithm
- JWT standard = **RFC 7519**, ใช้ทำ Authorization, Information Exchange
- Token flow: login → auth service → token generator → access token → เก็บใน Local/Session Storage/Cookie → ส่ง Bearer → service ตรวจ → success/deny
- Axios ข้อดี: browser เก่า, abort, timeout, CSRF protection, upload progress, auto JSON, ใช้ใน Node
- Postman ทดสอบ API (ส่ง PUT/DELETE + JSON body ได้)
- Debug VSCode: launch.json → `"program": "${workspaceFolder}\\server.js"` → breakpoint

## ⚠️ มักสับสน
- `?` สองตัว → ส่ง **array** ตามลำดับ
- `req.params` (จาก `/:id`) vs `req.body` (JSON body) vs `req.query` (`?a=b`)
- validation ผิด → **400**, token ผิด → **401**
- `expiresIn: "1h"` อยู่ใน options (arg ที่ 3) ไม่ใช่ใน payload
- axios ข้อมูลอยู่ที่ `res.data` · fetch ต้อง `await res.json()`
