# Lab 10 · Shell Scripts

> ⚠️ ไฟล์ Lab นี้มีแค่คำสั่งงาน — ตัวอย่าง script **test1.sh ถึง test27.sh อยู่ในสไลด์เลคเชอร์** ที่ยังไม่ได้ส่งมา ส่วน "ความรู้ที่ต้องใช้" ด้านล่างเป็นพื้นฐาน shell script ทั่วไป (ไม่ใช่จากสไลด์) ถ้าได้ไฟล์เลคเชอร์มาจะสรุปให้ตรงกับ test ทั้ง 27 ข้อ

## งานที่ต้องทำ
- **Objective:** เรียนรู้การเขียน shell scripts
- สำหรับทุก script ในสไลด์เลคเชอร์ (**test1.sh – test27.sh**):
  1. สร้าง script
  2. รัน script
  3. ถ่ายภาพโค้ดและผลลัพธ์
  4. **อธิบายว่าแต่ละ script ทำอะไร**
- รวมเป็น PDF ชื่อ `6887xxx_lab10.pdf` ส่งใน MyCourses

## ความรู้ที่ต้องใช้ (พื้นฐาน bash)

### 🔹 สร้างและรัน script
```bash
nano test1.sh          # หรือ vim
chmod +x test1.sh      # ให้สิทธิ์ execute
./test1.sh             # รัน (หรือ bash test1.sh)
```
```bash
#!/bin/bash            # shebang: บอกว่าใช้ bash รัน
echo "Hello World"
```

### 🔹 ตัวแปรและ input
```bash
name="ICT"             # ห้ามมีช่องว่างรอบ =
echo "Hello $name"
read -p "Enter age: " age
echo "Age is $age"
today=$(date)          # command substitution
```
| ตัวแปรพิเศษ | ความหมาย |
|---|---|
| `$0` | ชื่อ script |
| `$1, $2, ...` | argument ตัวที่ 1, 2, ... |
| `$#` | จำนวน argument |
| `$@` | argument ทั้งหมด |
| `$?` | exit status ของคำสั่งล่าสุด (0 = สำเร็จ) |
| `$$` | PID ของ script |

### 🔹 คำนวณ
```bash
a=5; b=3
sum=$((a + b))
echo "Sum = $sum"
expr $a \* $b
```

### 🔹 เงื่อนไข
```bash
if [ $age -ge 18 ]; then
    echo "Adult"
elif [ $age -ge 12 ]; then
    echo "Teen"
else
    echo "Child"
fi
```
| เปรียบเทียบตัวเลข | | เปรียบเทียบ string / ไฟล์ | |
|---|---|---|---|
| `-eq` | = | `=` / `!=` | เท่า / ไม่เท่า |
| `-ne` | ≠ | `-z str` | ว่าง |
| `-gt` / `-ge` | > / ≥ | `-f file` | เป็นไฟล์ |
| `-lt` / `-le` | < / ≤ | `-d dir` | เป็น directory |

```bash
case $choice in
  1) echo "One" ;;
  2) echo "Two" ;;
  *) echo "Other" ;;
esac
```

### 🔹 Loop
```bash
for i in 1 2 3 4 5; do echo $i; done
for f in *.txt; do echo "File: $f"; done
for ((i=1; i<=5; i++)); do echo $i; done
count=1
while [ $count -le 5 ]; do
    echo $count
    count=$((count + 1))
done
```

### 🔹 Function
```bash
greet() {
    echo "Hello $1"
}
greet "Student"
```
