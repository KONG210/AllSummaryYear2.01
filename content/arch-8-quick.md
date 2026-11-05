# Ch 8 · สรุปก่อนสอบ

## 🧠 ต้องเข้าใจ
- เป้าหมาย: หลายโปรเซสเซอร์ → performance สูง (scalability, availability, power efficiency)
- Task-level parallelism (งานอิสระ → throughput) vs parallel processing program (โปรแกรมเดียวหลายโปรเซสเซอร์) · multicore = หลาย core ในชิป
- ปัญหาอยู่ที่ **software**: **partitioning, coordination, communications overhead** (ถ้าเร็วขึ้นไม่มาก ใช้ uniprocessor ดีกว่า)

## ⭐ Flynn
| | Single Data | Multiple Data |
|---|---|---|
| Single Instr | **SISD** (Pentium 4) | **SIMD** (SSE x86) |
| Multiple Instr | **MISD** (ไม่มี) | **MIMD** (Xeon e5345) |
- **SPMD** = โปรแกรมเดียวบน MIMD + conditional code ต่อโปรเซสเซอร์

## ⭐ Vector / SIMD
- Vector: pipeline ลึก, vector registers (RISC-V v0–v31, 32×64 elements ×64 bit), `fld.v fsd.v fadd.d.v fadd.d.vs` → **ลด instruction fetch**
- DAXPY (Y = aX + Y, BLAS): scalar loop ~8 คำสั่ง×64 รอบ → vector **6 คำสั่ง**
- Vector ดีกว่า scalar: data-parallel ง่าย, ไม่มี loop-carried dependence, HW เช็กน้อย, burst memory, ไม่มี control hazard, ทั่วไปกว่า MMX/SSE
- SIMD (MMX, SSE, 128-bit): คำสั่งเดียวพร้อมกัน, sync ง่าย, ดีกับ data-parallel
- Vector = **variable width + strided access** · Multimedia ext = **fixed width, ไม่มี strided**

## ⭐ Multithreading (ทำซ้ำ registers/PC, สลับเร็ว)
- **Fine-grain**: สลับทุก cycle, ซ่อน stall ได้
- **Coarse-grain**: สลับเมื่อ stall นาน (L2 miss), HW ง่าย แต่ซ่อน stall สั้นไม่ได้
- **SMT**: หลาย thread ใน cycle เดียว (multiple-issue, dynamic, register renaming) — Pentium-4 HT (registers แยก, FU/cache ร่วม)

## ⭐ Shared memory vs Message passing
- **SMP**: address space เดียว, locks · **UMA** (เวลาเท่ากัน) vs **NUMA** (ไม่เท่ากัน)
- Sum reduction 64,000 ตัว / 64 โปรเซสเซอร์: แต่ละตัวรวม 1000 → reduction half 64→32→…→1 (6 ขั้น), **synch()** ทุกขั้น, half คี่ P0 เก็บตัวที่เหลือ
- **Message passing**: private address space, send/receive
- **Cluster**: เครื่องอิสระ (memory + OS ของตัวเอง) ผ่าน I/O/Ethernet · high availability, scalable, ถูก · ปัญหา: ค่าดูแล (ใช้ VM), interconnect BW ต่ำ
- **Grid**: เครือข่ายระยะไกล, แจกงานแล้วส่งผลกลับ, ใช้เวลาว่าง PC (SETI@home)

## ⭐ GPU
- data-parallel, **multithread สูงมาก**, thread switching ซ่อน memory latency, cache น้อย, memory กว้าง BW สูง
- Heterogeneous: **CPU = sequential, GPU = parallel** · API: DirectX, OpenGL, Cg, HLSL, **CUDA**
- Fermi: 16 SIMD lanes, thread กว้าง 32 element ใน 2 cycles, 32K×32-bit registers, 64 reg/thread
- ไม่เข้า SIMD/MIMD พอดี (conditional ทำให้ดูเป็น MIMD แต่ช้า)
- ILP: static = **VLIW**, dynamic = **Superscalar** · DLP: static = **SIMD/Vector**, dynamic = **Tesla**
- GPU vs multicore: GPU cache เล็ก (0.75 MB vs 8 MB), memory น้อยกว่า, **ไม่มี demand paging, ไม่มี integrated scalar, ไม่ cache coherent**

## ⭐ Network / Benchmarks
- Topologies: bus, ring, 2D mesh, N-cube, fully connected, multistage
- Characteristics: latency, throughput (link / total / **bisection** bandwidth), congestion, cost, power, routability in silicon
- Benchmarks: **Linpack** (matrix), **SPECrate** (job-level), **SPLASH** (shared memory, strong scaling), **NAS** (CFD), **PARSEC** (Pthreads/OpenMP)
- **Arithmetic intensity = FLOPs / byte** · peak GFLOPS (data sheet) + peak memory BW (Stream)
- Concluding: SaaS + clusters, performance per dollar / per Joule (mobile + WSC)
