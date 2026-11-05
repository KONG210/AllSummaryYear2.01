# WebLab 12 · เช็กลิสต์ก่อนสอบ

- [ ] URL: `https://api.spoonacular.com/recipes/complexSearch?query=...&number=20&apiKey=...`
- [ ] อ่านค่าจาก input ด้วย `document.querySelector("#q").value`
- [ ] `fetch(url).then(res => res.json()).then(data => ...).catch(...)`
- [ ] วนลูป `data.results` สร้าง `<img src=image>` + `title`
- [ ] ใส่ผลลง `innerHTML`
- [ ] เช็ก `res.ok` (fetch ไม่ reject ตอน 4xx/5xx)

```js
fetch(url)
  .then(res => res.json())
  .then(data => {
    document.querySelector("#output").innerHTML =
      data.results.map(r => `<div><img src="${r.image}"><p>${r.title}</p></div>`).join("");
  })
  .catch(err => console.log(err));
```

เทียบ axios (Node): `axios.get(url, { params: { query, number: 20, apiKey } }).then(res => res.data.results)`
