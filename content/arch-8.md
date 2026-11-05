# Ch 8 · Parallel Processors from Client to Cloud

> ℹ️ สไลด์ใช้หัวข้อ "§6.x" ตามหนังสือ Patterson & Hennessy (Chapter 6) แต่ในวิชานี้คือบทที่ VIII

## ภาพรวมของบท

**Chapter overview:** Introduction · ความยากของการเขียนโปรแกรมขนาน · SISD, MIMD, SIMD, SPMD, Vector · Hardware multithreading · Multicore และ shared memory multiprocessors · GPU · Clusters & message passing · Network topologies · (+ benchmarks, performance model)

> 🔗 ภาพใหญ่: ทำให้เร็วขึ้นด้วย **หลายโปรเซสเซอร์** → แต่ยากที่ **ซอฟต์แวร์** → จำแนกเครื่องตาม instruction/data stream (**Flynn**) → ภายใน core ใช้ **multithreading** → หลาย core แชร์หน่วยความจำ (**SMP: UMA/NUMA**) → **GPU** = SIMD + multithread หนัก ๆ → หลายเครื่องแยกหน่วยความจำคุยด้วย **message passing** (cluster, grid) → เชื่อมด้วย **network topology**

---

## 1. Introduction (§6.1)

**เป้าหมาย:** เชื่อมคอมพิวเตอร์หลายเครื่องเพื่อ **performance สูงขึ้น**

| คำศัพท์ | ความหมาย |
|---|---|
| **Multiprocessors** | ระบบหลายโปรเซสเซอร์ — ให้ **scalability, availability, power efficiency** |
| **Task-level (process-level) parallelism** | รัน **งานอิสระหลายงานพร้อมกัน** → **throughput สูง** |
| **Parallel processing program** | **โปรแกรมเดียว** รันบนหลายโปรเซสเซอร์ |
| **Multicore microprocessors** | **ชิปที่มีหลายโปรเซสเซอร์ (cores)** |

### 🔹 Hardware และ Software
| | Serial / Sequential | Parallel / Concurrent |
|---|---|---|
| **Hardware** | Serial: Pentium 4 | Parallel: quad-core Xeon e5345 |
| **Software** | Sequential: matrix multiplication | Concurrent: operating system |

- software แบบ sequential หรือ concurrent **รันได้ทั้งบน serial และ parallel hardware**
- **ความท้าทาย:** ใช้ parallel hardware ให้ **ได้ประสิทธิภาพจริง**

**ที่เรียนมาแล้ว (ความขนานในบทก่อน ๆ):** §2.11 synchronization · §3.6 subword parallelism · §4.10 instruction-level parallelism · §5.10 cache coherence

---

## 2. The Difficulty of Creating Parallel Processing Programs (§6.2)

- **ปัญหาอยู่ที่ parallel software**
- ต้องได้ performance ดีขึ้น **อย่างมีนัยสำคัญ** — ไม่งั้นใช้ uniprocessor ที่เร็วขึ้นดีกว่า **เพราะง่ายกว่า!**

**ความยาก 3 อย่าง**
1. **Partitioning** — แบ่งงาน
2. **Coordination** — ประสานงาน (เช่น synchronization)
3. **Communications overhead** — ค่าใช้จ่ายในการสื่อสารระหว่างโปรเซสเซอร์

---

## 3. SISD, MIMD, SIMD, SPMD, Vector (§6.3)

### 🔹 Flynn's classification (Instruction × Data streams)
| | Single Data | Multiple Data |
|---|---|---|
| **Single Instruction** | **SISD:** Intel Pentium 4 | **SIMD:** SSE instructions ของ x86 |
| **Multiple Instruction** | **MISD:** ไม่มีตัวอย่างในปัจจุบัน | **MIMD:** Intel Xeon e5345 |

> 📌 **SPMD (Single Program Multiple Data):** **โปรแกรมขนานบนเครื่อง MIMD** — โปรแกรมเดียวกันรันทุกโปรเซสเซอร์ แต่มี **conditional code** ให้แต่ละโปรเซสเซอร์ทำต่างกัน

### 🔹 Vector Processors
- functional unit **pipeline ลึกมาก**
- **stream ข้อมูล** จาก/ไป **vector registers** สู่ unit
  - เก็บข้อมูลจาก memory เข้า register
  - เก็บผลจาก register กลับ memory
- **ตัวอย่าง: Vector extension ของ RISC-V**
  - **v0–v31:** 32 registers × 64 elements (element ละ 64 bit)
  - `fld.v`, `fsd.v` = load/store vector
  - `fadd.d.v` = บวก vector ของ double
  - `fadd.d.vs` = บวก scalar เข้ากับทุก element
- **ลด instruction-fetch bandwidth อย่างมาก**

### 🔹 ตัวอย่าง: DAXPY (Y = a × X + Y)
**DAXPY** = **D**ouble-precision **A** × **X** **P**lus **Y** — คำสั่งพีชคณิตเชิงเส้นใน library **BLAS** (vector X คูณ scalar a แล้วบวก vector Y)

```c
void daxpy(size_t n, double a, const double x[], double y[]) {
    for (size_t i = 0; i < n; i++)
        y[i] = a * x[i] + y[i];      // Y = a*X + Y
}
```

**Conventional RISC-V (วน loop ทีละ element)**
```asm
      fld    f0, a(x3)        // load scalar a
      addi   x5, x19, 512     // end of array X
loop: fld    f1, 0(x19)       // load x[i]
      fmul.d f1, f1, f0       // a * x[i]
      fld    f2, 0(x20)       // load y[i]
      fadd.d f2, f2, f1       // a * x[i] + y[i]
      fsd    f2, 0(x20)       // store y[i]
      addi   x19, x19, 8      // increment index to x
      addi   x20, x20, 8      // increment index to y
      bltu   x19, x5, loop    // repeat if not done
```

**Vector RISC-V (ไม่มี loop)**
```asm
      fld      f0, a(x3)      // load scalar a
      fld.v    v0, 0(x19)     // load vector x
      fmul.d.vs v0, v0, f0    // vector-scalar multiply
      fld.v    v1, 0(x20)     // load vector y
      fadd.d.v v1, v1, v0     // vector-vector add
      fsd.v    v1, 0(x20)     // store vector y
```
> ⭐ conventional: 8 คำสั่งใน loop × 64 รอบ ≈ 500+ คำสั่ง · vector: **6 คำสั่ง** → ลด instruction fetch มาก (512 bytes / 8 = 64 elements)

### 🔹 Vector vs Scalar (ข้อดีของ vector)
- ทำให้เขียนโปรแกรม **data-parallel** ง่ายขึ้น
- บอกชัดว่า **ไม่มี loop-carried dependences**
- **ฮาร์ดแวร์ต้องเช็กน้อยลง**
- รูปแบบการเข้าถึงสม่ำเสมอ ได้ประโยชน์จาก **interleaved และ burst memory**
- **หลีกเลี่ยง control hazards** เพราะไม่มี loop
- **ทั่วไปกว่า** media extension เฉพาะกิจ (MMX, SSE)
- เข้ากับ compiler technology ได้ดีกว่า

### 🔹 SIMD
- ทำงาน **ทีละ element บน vector ของข้อมูล** เช่น **MMX, SSE** ใน x86 (หลาย element ใน register กว้าง **128 bit**)
- **ทุกโปรเซสเซอร์รันคำสั่งเดียวกันพร้อมกัน** แต่ละตัวใช้ data address ต่างกัน
- **synchronization ง่ายขึ้น**
- ลดฮาร์ดแวร์ควบคุมคำสั่ง
- **ดีที่สุดกับงาน data-parallel สูง**

### 🔹 Vector vs Multimedia Extensions
| | Vector instructions | Multimedia extensions (MMX, SSE) |
|---|---|---|
| ความกว้าง | **variable** vector width | **fixed** width |
| strided access | **รองรับ** | **ไม่รองรับ** |
| functional units | ผสม pipelined + arrayed | |

---

## 4. Hardware Multithreading (§6.4)

> 📌 **Multithreading:** รัน **หลาย thread พร้อมกัน** — **ทำซ้ำ registers, PC ฯลฯ** และ **สลับ thread ได้เร็ว**

| ชนิด | วิธี | ข้อดี/ข้อเสีย |
|---|---|---|
| **Fine-grain** | **สลับ thread ทุก cycle**; interleave การรันคำสั่ง | ถ้า thread หนึ่ง stall ตัวอื่นรันต่อ (ซ่อน stall สั้นได้) |
| **Coarse-grain** | **สลับเฉพาะ stall นาน** (เช่น **L2-cache miss**) | **ฮาร์ดแวร์ง่ายกว่า** แต่ **ซ่อน stall สั้นไม่ได้** (เช่น data hazards) |
| **Simultaneous (SMT)** | ใน processor แบบ **multiple-issue dynamically scheduled**: schedule คำสั่งจาก **หลาย thread ใน cycle เดียวกัน** เมื่อ functional unit ว่าง; ภายใน thread จัดการ dependency ด้วย scheduling + **register renaming** | ใช้ functional unit คุ้มที่สุด |

**ตัวอย่าง SMT: Intel Pentium-4 HT (Hyper-Threading)** — 2 threads: **registers ทำซ้ำ**, **functional units และ caches ใช้ร่วม**

(สไลด์ Multithreading Example เป็นรูปเปรียบเทียบการใช้ issue slot ของแต่ละแบบ)

### 🔹 Future of Multithreading
- จะอยู่รอดไหม? รูปแบบไหน?
- เรื่อง **พลังงาน** → microarchitecture เรียบง่ายขึ้น → multithreading แบบง่ายขึ้น
- ใช้ **ทนต่อ cache-miss latency** — **thread switch อาจได้ผลที่สุด**
- **core เรียบง่ายหลายตัว** อาจแชร์ทรัพยากรได้ดีกว่า

---

## 5. Multicore and Other Shared Memory Multiprocessors (§6.5)

> 📌 **SMP (Shared Memory Multiprocessor):** ฮาร์ดแวร์ให้ **physical address space เดียว** แก่ทุกโปรเซสเซอร์

- **synchronize ตัวแปรที่แชร์ด้วย locks**
- **Memory access time:**
  - **UMA (Uniform Memory Access)** — เวลาเข้าถึงหน่วยความจำ **เท่ากัน** ทุกโปรเซสเซอร์
  - **NUMA (Nonuniform Memory Access)** — เวลา **ไม่เท่ากัน** (หน่วยความจำใกล้ตัวเร็วกว่า)

### 🧮 ตัวอย่าง: Sum Reduction
**โจทย์:** รวมเลข **64,000 ตัว** บน **UMA 64 โปรเซสเซอร์** (ID: 0 ≤ Pn ≤ 63)

**ขั้น 1: แต่ละโปรเซสเซอร์รวมส่วนของตัวเอง (1000 ตัว)**
```c
sum[Pn] = 0;
for (i = 1000*Pn; i < 1000*(Pn+1); i += 1)
    sum[Pn] += A[i];
```

**ขั้น 2: Reduction (divide and conquer)** — ครึ่งหนึ่งของโปรเซสเซอร์บวกเป็นคู่ แล้วเหลือ 1/4 ... **ต้อง synchronize ระหว่างแต่ละขั้น**
```c
half = 64;
do {
    synch();
    if (half % 2 != 0 && Pn == 0)
        sum[0] += sum[half-1];
        /* Conditional sum needed when half is odd;
           Processor 0 gets missing element */
    half = half / 2;          /* dividing line on who sums */
    if (Pn < half) sum[Pn] += sum[Pn+half];
} while (half > 1);
```
**ไล่ค่า half:** 64 → 32 → 16 → 8 → 4 → 2 → 1 = **6 ขั้น** (log₂64) ผลสุดท้ายอยู่ใน `sum[0]`
- รอบแรก (half = 32): P0–P31 บวก sum[Pn+32]
- ถ้า half เป็นเลขคี่ P0 ต้องเก็บตัวที่เหลือ (`sum[half-1]`)

---

## 6. Introduction to GPUs (§6.6)

### 🔹 History
- **การ์ดจอยุคแรก:** frame buffer memory + address generation สำหรับ video output
- **3D graphics:** เดิมอยู่ในเครื่องระดับสูง (SGI) → **Moore's Law** ทำให้ถูกลง หนาแน่นขึ้น → การ์ด 3D สำหรับ PC และเกมคอนโซล
- **GPU** = โปรเซสเซอร์สำหรับงาน 3D graphics: vertex/pixel processing, shading, texture mapping, rasterization

### 🔹 GPU Architectures
- งาน **data-parallel สูงมาก**
  - GPU **multithread สูงมาก**
  - ใช้ **thread switching ซ่อน memory latency**
  - **พึ่ง multi-level cache น้อย**
  - **graphics memory กว้างและ bandwidth สูง**
- แนวโน้ม **general purpose GPU (GPGPU)**
  - **ระบบ CPU/GPU แบบ heterogeneous: CPU ทำโค้ด sequential, GPU ทำโค้ด parallel**
- ภาษา/API: **DirectX, OpenGL**, Cg, HLSL, **CUDA** (Compute Unified Device Architecture)

### 🔹 Example: NVIDIA Fermi
- ประกอบด้วย **หลาย SIMD processors**
- SIMD processor: **16 SIMD lanes**
- SIMD instruction ทำงานกับ thread กว้าง **32 element** → schedule แบบ dynamic บน processor 16-wide **ใน 2 cycles**
- **32K × 32-bit registers** กระจายตาม lane → **64 registers ต่อ thread context**

### 🔹 Classifying GPUs
- **ไม่เข้ากับ SIMD/MIMD model พอดี**
  - conditional execution ใน thread ให้ **ภาพลวงว่าเป็น MIMD** แต่ **performance ลด**
  - ต้องเขียน general purpose code อย่างระวัง

| | Static (ค้นพบตอน compile) | Dynamic (ค้นพบตอน runtime) |
|---|---|---|
| **Instruction-Level Parallelism** | **VLIW** | **Superscalar** |
| **Data-Level Parallelism** | **SIMD or Vector** | **Tesla Multiprocessor** (GPU) |

### 🔹 Putting GPUs into Perspective
| Feature | Multicore with SIMD | GPU |
|---|---|---|
| SIMD processors | 4 to 8 | 8 to 16 |
| SIMD lanes/processor | 2 to 4 | 8 to 16 |
| Multithreading HW support for SIMD threads | 2 to 4 | **16 to 32** |
| Single:double precision performance ratio | 2:1 | 2:1 |
| Largest cache size | **8 MB** | **0.75 MB** |
| Size of memory address | 64-bit | 64-bit |
| Size of main memory | 8–256 GB | 4–6 GB |
| Memory protection at page level | Yes | Yes |
| **Demand paging** | Yes | **No** |
| **Integrated scalar/SIMD processor** | Yes | **No** |
| **Cache coherent** | Yes | **No** |

(Guide to GPU Terms และ GPU Memory Structures เป็นรูปในสไลด์)

---

## 7. Clusters, WSC, and Other Message-Passing MPs (§6.7)

> 📌 **Message Passing:** แต่ละโปรเซสเซอร์มี **private physical address space** ฮาร์ดแวร์ **ส่ง/รับ message** ระหว่างโปรเซสเซอร์

| | Shared memory (SMP) | Message passing |
|---|---|---|
| address space | **เดียว** ใช้ร่วม | **แยก** ของใครของมัน |
| สื่อสาร | อ่าน/เขียนตัวแปรร่วม + locks | **ส่ง message** |

### 🔹 Loosely Coupled Clusters
- **เครือข่ายของคอมพิวเตอร์อิสระ** — แต่ละเครื่องมี **memory และ OS ของตัวเอง**
- เชื่อมผ่าน **I/O system** (Ethernet/switch, Internet)
- เหมาะกับงานที่ **task อิสระ**: web servers, databases, simulations
- **High availability, scalable, affordable**
- **ปัญหา:** **ค่าดูแลระบบ** (นิยมใช้ virtual machines) · **interconnect bandwidth ต่ำ** (เทียบกับ processor/memory bandwidth ของ SMP)

### 🔹 Grid Computing
- คอมพิวเตอร์แยกกัน เชื่อมด้วย **เครือข่ายระยะไกล** (Internet)
- **แจกงาน (work units) ออกไป แล้วส่งผลกลับ**
- **ใช้เวลาว่างของ PC** ได้ เช่น **SETI@home, World Community Grid**

---

## 8. Introduction to Multiprocessor Network Topologies (§6.8)

**Network topologies** = การจัดวางโปรเซสเซอร์ switch และ link
- **Bus**
- **Ring**
- **2D Mesh**
- **N-cube** (เช่น N = 3 = ลูกบาศก์ 8 node)
- **Fully connected**
- **Multistage networks** (เช่น crossbar, Omega — สไลด์เป็นรูป)

### 🔹 Network Characteristics
- **Performance:** latency per message (เครือข่ายว่าง) · throughput: **link bandwidth**, **total network bandwidth**, **bisection bandwidth** · congestion delays (ขึ้นกับ traffic)
- **Cost**
- **Power**
- **Routability in silicon**

> ⭐ **Bisection bandwidth** = bandwidth ระหว่างสองครึ่งของเครือข่ายเมื่อแบ่งครึ่ง (กรณีแย่สุด) · **Total network bandwidth** = ผลรวม bandwidth ทุก link (คำอธิบายเพิ่มจากหนังสือ)

---

## 9. Multiprocessor Benchmarks and Performance Models (§6.10)

### 🔹 Parallel Benchmarks
| Benchmark | ลักษณะ |
|---|---|
| **Linpack** | matrix linear algebra |
| **SPECrate** | รัน SPEC CPU programs แบบขนาน — **job-level parallelism** |
| **SPLASH** | Stanford Parallel Applications for Shared Memory — ผสม kernel และ app, **strong scaling** |
| **NAS** | NASA Advanced Supercomputing — kernel ด้าน computational fluid dynamics |
| **PARSEC** | Princeton — multithreaded apps ใช้ **Pthreads และ OpenMP** |

### 🔹 Code or Applications?
- benchmark แบบเดิม: **code และ data set ตายตัว**
- parallel programming กำลังพัฒนา → algorithm, ภาษา, tools ควรเป็นส่วนหนึ่งของระบบไหม?
- เปรียบเทียบระบบโดยให้ implement application ที่กำหนด (เช่น Linpack, Berkeley Design Patterns) → **ส่งเสริมนวัตกรรม**

### 🔹 Modeling Performance
- metric = **achievable GFLOPs/sec** วัดด้วย kernel จาก Berkeley Design Patterns
- **Arithmetic intensity** ของ kernel = **FLOPs ต่อ byte ของ memory ที่เข้าถึง**
- สำหรับคอมพิวเตอร์หนึ่ง หา **Peak GFLOPS** (จาก data sheet) และ **Peak memory bytes/sec** (ด้วย **Stream benchmark**)

```text
Arithmetic intensity = FLOPs / bytes of memory accessed
Attainable GFLOPs/sec = min(Peak memory BW × Arithmetic intensity, Peak GFLOPs/sec)   ← Roofline model (จากหนังสือ)
```

---

## 10. Concluding Remarks (§6.14)
- **เป้าหมาย:** performance สูงขึ้นด้วยหลายโปรเซสเซอร์
- **ความยาก:** พัฒนา **parallel software** และออกแบบ **architecture ที่เหมาะสม**
- **SaaS** สำคัญขึ้น และ **clusters เหมาะกับ SaaS**
- **Performance per dollar** และ **performance per Joule** เป็นตัวขับเคลื่อนทั้ง **mobile และ WSC** (warehouse-scale computers)

## จุดเน้นสำหรับสอบ
- 🔥 Flynn: SISD/SIMD/MISD/MIMD + ตัวอย่าง, SPMD
- 🔥 Fine-grain vs coarse-grain vs SMT
- 🔥 UMA vs NUMA, shared memory vs message passing, cluster vs grid
- 🔥 Vector vs scalar, vector vs multimedia extensions, DAXPY
- ⭐ ความยาก 3 อย่าง (partitioning, coordination, communications overhead)
- ⭐ GPU architecture, ตาราง multicore vs GPU, ILP/DLP static/dynamic
- ⭐ sum reduction (6 ขั้น, synch, half คี่)
- ⭐ topologies + network characteristics, benchmarks, arithmetic intensity

## ✅ Coverage Check
- [x] Course outline, chapter overview, introduction, HW/SW, covered sections
- [x] Difficulty of parallel programming
- [x] Flynn, SPMD, vector processors, RISC-V vector, DAXPY, vector vs scalar, SIMD, vector vs multimedia
- [x] Multithreading (fine, coarse, SMT, example, future)
- [x] Shared memory, UMA/NUMA, sum reduction
- [x] GPU history, architecture, Fermi, classification, perspective table
- [x] Message passing, clusters, grid
- [x] Topologies, multistage, characteristics
- [x] Benchmarks, code vs apps, performance modeling, concluding remarks
