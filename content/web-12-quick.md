# L12 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- **Layer** (data / logic / presentation) แยกเครื่องจริง = **Tier**: 1-tier (เครื่องเดียว, scale ยาก) · 2-tier (client-server) · 3-tier (front-end, middleware, back-end) · N-tier (เพิ่ม app server, distributed)
- **HTTP**: simple, extensible, **stateless แต่ไม่ sessionless**
- **Web Service** (W3C) = machine-to-machine interaction ผ่าน network · ทุก WS เป็น API แต่ไม่กลับกัน · WS ต้องมี network
- **REST เป็น architecture** ไม่ใช่ protocol/standard

## 📌 HTTP Message
| | ส่วนที่ 1 | ส่วนที่ 2 | ส่วนที่ 3 |
|---|---|---|---|
| Request | Request line: `GET /path HTTP/1.1` | Headers: Host, User-Agent, Accept, Content-Type, Authorization | Body (POST/PUT/DELETE) |
| Response | Status line: `HTTP/1.1 200 OK` | Headers: Content-Type, Date, Server, Content-Length | Body |

Status: **200** OK · **404** Not Found · **500** Internal Server Error

## ⭐ REST — ท่องให้ได้
**6 Constraints:** Client/Server · Stateless · Cacheable · Uniform Interface · Layered System · Code-On-Demand (**optional**)

**Uniform Interface 4 ข้อ:** Identifying the resource (URI, ใช้คำนาม) · Resource representation (HTML/XML/JSON, เลือกด้วย Accept) · Self-descriptive messages (metadata) · **HATEOAS** (ใช้ link นำทาง)

| Method | ทำอะไร |
|---|---|
| GET | ดึง |
| POST | สร้าง (server กำหนด URI) — **ไม่ idempotent** |
| PUT | อัปเดต (client กำหนด URI) — **idempotent** |
| PATCH | อัปเดตบางส่วน |
| DELETE | ลบ |

## ⭐ Storage
| | Cookie | LocalStorage | sessionStorage |
|---|---|---|---|
| ขนาด | ~4KB | ~5MB/domain | ~5MB |
| server อ่านได้ | ได้ (header) | ไม่ได้ | ไม่ได้ |
| ส่งทุก request | ใช่ | ไม่ | ไม่ |
| หมดอายุ | ตั้งได้ | จนกว่าจะลบ | ต่อ tab (refresh ไม่หาย) |

## ⭐ JSON
- ต่างจาก object literal: **เป็น String**, **ไม่มี properties/method**, **ชื่อ property มี quote**
- 6 types: Number, String, Boolean, Array, Object, null
- `JSON.stringify(obj)` = serialise · `JSON.parse(str)` = deserialise
- XML: tag ตั้งเอง, schema มี data type, ไม่ได้มีไว้ให้ browser แสดง, เป็นต้นไม้

## 📌 Syntax
```js
fetch(url, { method: "GET" })
  .then(res => res.json())          // res.ok = 200–299
  .then(data => console.log(data))
  .catch(err => console.log(err));
```
```js
let xhr = new XMLHttpRequest();
xhr.open("GET", url);
xhr.responseType = "json";
xhr.onload = () => console.log(xhr.status, xhr.response);
xhr.send();
```
readyState: 0 UNSENT · 1 OPENED · 2 HEADERS_RECEIVED · 3 LOADING · 4 DONE

## ⚠️ มักสับสน
- `Accept` = อยากได้อะไร · `Content-Type` = ส่งอะไรมา
- XHR `load` เกิดแม้ได้ 400/500 · fetch ไม่ reject ตอน 404 → เช็ก `ok`/`status`
- Stateless = ทุก request ต้องมีข้อมูลครบ
- Cacheable ข้อเสีย: ข้อมูลเก่า (stale) → reliability ลด
- Uniform interface ข้อเสีย: ลดประสิทธิภาพ
