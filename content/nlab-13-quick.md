# Lab 13 · เช็กลิสต์ก่อนสอบ

> 📌 PT 14.8.1 · MultiServer 192.168.1.254 · Simulation mode ดู PDU

| App | Transport | Port | สังเกต |
|---|---|---|---|
| HTTP | TCP | 80 | SYN+ACK, PSH+ACK, ก้อนแรกช้าเพราะ handshake |
| FTP | TCP | 21 | SYN → SYN+ACK → ACK · "Welcome to PT Ftp server" |
| DNS | **UDP** | 53 | ไม่มี seq/ack · ANSWER = 192.168.1.254 |
| SMTP | TCP | 25 | ส่งอีเมล |
| POP3 | TCP | 110 | ดึงอีเมล |

- **TCP** = connection-oriented, reliable, handshake, มี seq/ack · **UDP** = connectionless, unreliable, overhead ต่ำ
- TCP flags (ขวา → ซ้าย): FIN, SYN, RST, PSH, ACK, URG
- client ใช้ port ชั่วคราว (~1025) · server ใช้ well-known · ตอบกลับสลับ src/dst
- **Conversation multiplexing** · สีใน simulation = protocol ต่างกัน
