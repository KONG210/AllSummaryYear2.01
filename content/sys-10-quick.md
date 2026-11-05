# Lab 10 · Shell cheat sheet

> ไฟล์ test1–27.sh อยู่ในสไลด์เลคเชอร์ที่ยังไม่มี — ด้านล่างเป็นพื้นฐานทั่วไป

```bash
#!/bin/bash                 chmod +x s.sh ; ./s.sh
x=5 (ห้ามเว้นวรรค)           echo "$x"     read -p "?" v
$0 ชื่อ script  $1.. args  $# จำนวน  $@ ทั้งหมด  $? exit status  $$ PID
$((a+b))   $(cmd)   expr $a \* $b
if [ $a -gt $b ]; then ... elif ...; else ...; fi
-eq -ne -gt -ge -lt -le (ตัวเลข)   = != -z (string)   -f -d (ไฟล์)
case $x in 1) ... ;; *) ... ;; esac
for i in 1 2 3; do ...; done    for ((i=0;i<5;i++)); do ...; done
while [ $c -le 5 ]; do c=$((c+1)); done
f() { echo $1; } ; f arg
```
