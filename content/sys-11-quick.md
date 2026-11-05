# Lab 11 · สรุปก่อนสอบ

- `fork()` คืน: **<0** ล้มเหลว · **0** = child · **>0** = parent (ค่าคือ PID ของ child) · `getpid()`, `getppid()`, `wait(NULL)`
- fork n ครั้งต่อกัน → **2ⁿ processes**
- Process: memory แยก (child ได้ copy) · Thread: memory ร่วม
- `pthread_create(&t, NULL, func, arg)` · `pthread_join(t, NULL)` · `pthread_exit` · mutex lock/unlock กัน race condition
- compile thread: `gcc x.c -o x -lpthread`
- ลำดับ output ไม่แน่นอน
- (โค้ด fork1–3, thread1–5 อยู่ในสไลด์เลคเชอร์ที่ยังไม่มี)
