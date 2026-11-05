# L9 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- **Node.js** = asynchronous JS runtime บน **V8** (ของ Chrome) — ไม่มี DOM, `window`, `document`
- **Single thread event loop**: thread เดียวรับ request → ส่งงานไปทำ → รับ request ถัดไปเลย (**non-blocking**) → งานเสร็จค่อยตอบ
- เหมาะ: I/O intensive, real-time, JSON API · ไม่เหมาะ: คำนวณหนัก
- Static resource ส่งตรง · Dynamic resource ผ่าน web app + DB + template
- **ทุกไฟล์ = module**, ตัวแปรในไฟล์เป็น private

## 📌 Syntax ที่ต้องเขียนได้

```js
// export / import
module.exports = "Hello";            // literal
module.exports.sayHi = sayHello;     // function (เรียกด้วย sayHi)
const m = require("./folder/file");  // relative path
```

```js
// server + routing (ต้องเขียนได้จากความจำ)
const http = require("http");
const fs = require("fs");
const server = http.createServer((req, res) => {
  if (req.url === "/bbq") {
    fs.readFile("./bbq.html", (err, data) => {
      res.statusCode = 200;
      res.setHeader("Content-Type", "text/html");
      res.write(data);
      res.end();
    });
  } else {
    res.statusCode = 404;
    res.setHeader("Content-Type", "text/plain");
    res.end("Invalid requested :(");
  }
});
server.listen(8081, () => console.log("Listening on 8081"));
```

```js
fs.readFile(p, "utf8", (err, contents) => { if (err) ...; else ... }); // async
let c = fs.readFileSync(p, "utf8");                                   // sync
```

## ⭐ ต้องจำ
| สิ่งที่ใช้ | ได้อะไร |
|---|---|
| `__dirname` / `__filename` | โฟลเดอร์ / path เต็มรวมชื่อไฟล์ |
| `path.join(__dirname,'x')` | ต่อ path |
| `path.resolve()` | absolute path |
| `path.parse()` | `root, dir, base, name, ext` |
| `path.extname()` | `.js` |
| `new URL(s).host` | host **+ port** |
| `.pathname` / `.search` | `/p/a/t/h` / `?country=TH&time=now` |
| `.searchParams.get('time')` | `now` |
| `req.url` / `req.method` | path / GET |
| FS คู่ async–sync | readFile, appendFile, writeFile, unlink, rename (+Sync) |

## ⚠️ มักสับสน
- async หลายตัวติดกัน → **ลำดับผลไม่แน่นอน**; แก้ด้วย sync, nested callback (= **callback hell**), Promise, async/await
- ลืม `res.end()` = browser ค้าง
- browser ขอ `/favicon.ico` เองทุกครั้ง
- `console.log` ใน Node ไม่ใช่ `window.console.log`
- 200 = OK, 404 = Not Found
