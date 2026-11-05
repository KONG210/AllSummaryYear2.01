# WebLab 11 · เช็กลิสต์ก่อนสอบ

- [ ] `npm init` → `npm install express nodemon mysql2 dotenv` → `"start": "nodemon app.js"`
- [ ] `.env` มี PORT=3030, DB_HOST, DB_USER, DB_PASS, DB_NAME
- [ ] `dotenv.config()` ก่อนใช้ `process.env`
- [ ] `app.use(express.urlencoded({ extended: true }))` เพื่ออ่านฟอร์ม
- [ ] `GET /` → log `Request at /` → `res.sendFile(path.join(__dirname,"html","searchProf.html"))`
- [ ] ฟอร์ม `action="/form-submit" method="POST"` และ input มี `name`
- [ ] `POST /form-submit` → `req.body.<name>` → `SELECT * FROM Professor WHERE emp_num = ?`
- [ ] `results.length > 0` → log `Found x` + `found.html` · ไม่งั้น `Not found` + `notfound.html`
- [ ] log `${results.length} rows returned`

```js
connection.query("SELECT * FROM Professor WHERE emp_num = ?", [empNum], (error, results) => {
  if (error) throw error;
  if (results.length > 0) res.sendFile(path.join(__dirname, "html", "found.html"));
  else res.sendFile(path.join(__dirname, "html", "notfound.html"));
});
```

⚠️ ลืม urlencoded = `req.body` undefined · ชื่อ input ไม่ตรง = ได้ undefined · ทดสอบ: 297 = พบ, 111 = ไม่พบ
