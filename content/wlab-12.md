# WebLab 12 · เรียก Web Service สาธารณะด้วย fetch

## โจทย์ในหนึ่งประโยค

เขียน HTML + JavaScript เรียกบริการ **Search Recipes (complexSearch)** ของ **spoonacular API** ด้วย **fetch** แล้วแสดงผลการค้นหาอาหาร (ตัวอย่างค้นด้วยคำว่า Chicken) บันทึกเป็น `webservice-lab.html`

> 🔗 ใช้ความรู้จาก L12 (fetch, JSON, REST, query string) และเทียบกับตัวอย่าง axios ใน L13 ที่เรียก API เดียวกันจากฝั่ง Node

## 1. ขั้นตอน
1. สมัครรับ **API key** ที่ spoonacular.com/food-api
2. อ่านเอกสาร **Search Recipes Complex** และ **Authentication** (ส่ง key เป็น query parameter `apiKey`)
3. เขียนหน้าเว็บ: ช่องค้นหา + ปุ่ม → fetch → แสดงผล (รูป + ชื่ออาหาร)

## 2. โครง URL

```text
GET https://api.spoonacular.com/recipes/complexSearch?query=chicken&number=20&apiKey=YOUR_KEY
```
- `query` = คำค้น, `number` = จำนวนผล, `apiKey` = key ของเรา
- ผลลัพธ์อยู่ใน `data.results` (array) แต่ละตัวมี `id`, `title`, `image` (เหมือนตัวอย่าง L13 ที่ใช้ `res.data.results`)

## 3. ตัวอย่างคำตอบ (แนวทาง)

```html
<input id="q" type="text" placeholder="เช่น chicken">
<button id="searchBtn">Search</button>
<div id="output"></div>

<script>
  const API_key = "YOUR_API_KEY";
  document.querySelector("#searchBtn").addEventListener("click", () => {
    const query = document.querySelector("#q").value;
    const url = "https://api.spoonacular.com/recipes/complexSearch?query=" +
                encodeURIComponent(query) + "&number=20&apiKey=" + API_key;
    fetch(url)
      .then((res) => {
        if (!res.ok) throw new Error("HTTP " + res.status);
        return res.json();
      })
      .then((data) => {
        let html = "";
        data.results.forEach((r) => {
          html += `<div><img src="${r.image}" width="200"><p>${r.title}</p></div>`;
        });
        document.querySelector("#output").innerHTML = html;
      })
      .catch((err) => console.log(err));
  });
</script>
```

**จุดสำคัญ**
- fetch ทำงาน **2 ขั้น**: ได้ `Response` → `res.json()` ได้ object
- เช็ก `res.ok` เพราะ fetch **ไม่ reject** เมื่อได้ 401/404
- `encodeURIComponent` ป้องกันคำค้นที่มีช่องว่าง/อักขระพิเศษ
- แสดงผลด้วยการสร้าง HTML แล้วใส่ `innerHTML`

## 4. เวอร์ชัน async/await
```js
async function search(query) {
  const res = await fetch(url);
  const data = await res.json();
  return data.results;
}
```
