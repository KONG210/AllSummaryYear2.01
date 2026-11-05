# Lab 11 · Process and Threads

> ⚠️ ไฟล์ Lab มีแค่รายการโปรแกรม (fork1–3.c, thread1–5.c) ซึ่งโค้ดอยู่ในสไลด์เลคเชอร์ที่ยังไม่ได้ส่งมา ส่วนความรู้ด้านล่างเป็นพื้นฐาน fork/pthread ทั่วไป

## งานที่ต้องทำ
- **Objectives:** สร้าง process ด้วย **`fork()`** และสร้าง thread ด้วย **pthread functions**
- สร้าง compile รัน: **fork1.c, fork2.c, fork3.c, thread1.c – thread5.c** แสดง source code และ output
- PDF ชื่อ `6887xxx_lab11.pdf`

## ความรู้ที่ต้องใช้

### 🔹 Process vs Thread
| | Process | Thread |
|---|---|---|
| หน่วยความจำ | **แยกกัน** (copy ของ parent) | **ใช้ร่วมกัน** ภายใน process เดียว |
| สร้างด้วย | `fork()` | `pthread_create()` |
| ค่าใช้จ่าย | หนักกว่า | เบากว่า |
| ตัวแปร global | เปลี่ยนใน child ไม่กระทบ parent | เปลี่ยนแล้วทุก thread เห็น |

### 🔹 fork()
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    if (pid < 0) {
        printf("fork failed\n");
    } else if (pid == 0) {
        printf("Child: PID=%d, parent=%d\n", getpid(), getppid());
    } else {
        wait(NULL);                       // รอ child จบ
        printf("Parent: PID=%d, child=%d\n", getpid(), pid);
    }
    return 0;
}
```
| ค่าที่ `fork()` คืน | หมายถึง |
|---|---|
| `< 0` | สร้างไม่สำเร็จ |
| `0` | กำลังรันใน **child** |
| `> 0` | กำลังรันใน **parent** (ค่า = PID ของ child) |

> 🧮 เรียก `fork()` n ครั้งต่อกัน (ไม่มีเงื่อนไข) → ได้ **2ⁿ processes** (เช่น fork 3 ครั้ง พิมพ์ "hello" 8 ครั้ง)

### 🔹 pthread
```c
#include <stdio.h>
#include <pthread.h>

void *worker(void *arg) {
    int id = *(int *)arg;
    printf("Thread %d running\n", id);
    return NULL;
}

int main() {
    pthread_t t[3];
    int ids[3];
    for (int i = 0; i < 3; i++) {
        ids[i] = i;
        pthread_create(&t[i], NULL, worker, &ids[i]);
    }
    for (int i = 0; i < 3; i++)
        pthread_join(t[i], NULL);         // รอ thread จบ
    return 0;
}
```
```bash
gcc fork1.c -o fork1 && ./fork1
gcc thread1.c -o thread1 -lpthread && ./thread1     # ต้องลิงก์ -lpthread
```

| ฟังก์ชัน | ทำอะไร |
|---|---|
| `pthread_create(&tid, attr, func, arg)` | สร้าง thread รัน `func(arg)` |
| `pthread_join(tid, &ret)` | รอ thread จบ |
| `pthread_exit(ret)` | จบ thread |
| `pthread_mutex_lock/unlock(&m)` | ป้องกัน race condition เมื่อแก้ตัวแปรร่วม |

> ⚠️ ลำดับ output ของหลาย thread/process **ไม่แน่นอน** ขึ้นกับ scheduler · ถ้าหลาย thread แก้ตัวแปรร่วมโดยไม่ใช้ mutex → **race condition**
