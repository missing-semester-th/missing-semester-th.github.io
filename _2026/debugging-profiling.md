---
layout: lecture
title: "Debugging and Profiling"
description: >
  เรียนรู้วิธี debug โปรแกรมด้วย logging และ debugger รวมถึงการ profile โค้ดเพื่อวิเคราะห์ประสิทธิภาพ
thumbnail: /static/assets/thumbnails/2026/lec4.png
date: 2026-01-15
ready: true
panopto: "https://mit.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=a72c48e3-5eb2-46fa-aa03-b3b700e1ca8d"
video:
  aspect: 56.25
  id: 8VYT9TcUmKs
---

กฎทองของการเขียนโปรแกรมคือ โค้ดจะไม่ทำในสิ่งที่เราคาดหวัง แต่จะทำในสิ่งที่เราบอกให้มันทำ การปิดช่องว่างตรงนี้บางทีก็เป็นเรื่องที่ยากพอสมควร ในบทนี้เราจะมาพูดถึงเทคนิคที่มีประโยชน์สำหรับจัดการกับโค้ดที่มี bug และโค้ดที่กินทรัพยากรเยอะ นั่นคือ debugging และ profiling

# Debugging

## Printf Debugging และ Logging

> "เครื่องมือ debug ที่มีประสิทธิภาพที่สุดยังคงเป็นการคิดอย่างรอบคอบ ควบคู่กับการวาง print statement อย่างเหมาะสม" — Brian Kernighan, _Unix for Beginners_

วิธีแรกในการ debug โปรแกรมคือการเพิ่ม print statement รอบ ๆ จุดที่พบปัญหา แล้วค่อยทำซ้ำไปเรื่อย ๆ จนกว่าจะดึงข้อมูลออกมาได้มากพอที่จะเข้าใจว่าอะไรเป็นสาเหตุของปัญหา

วิธีที่สองคือการใช้ logging ในโปรแกรม แทนที่จะใช้ print statement แบบ ad hoc Logging นั้นโดยพื้นฐานแล้วคือ "การ print ที่มีความรอบคอบมากขึ้น" ซึ่งมักจะทำผ่าน logging framework ที่มี built-in support สำหรับสิ่งเหล่านี้:

- ความสามารถในการส่ง log (หรือ subset ของ log) ไปยังตำแหน่ง output อื่น
- การตั้ง severity level (เช่น INFO, DEBUG, WARN, ERROR เป็นต้น) และให้สามารถ filter output ตาม level เหล่านั้นได้
- support สำหรับ structured logging ของข้อมูลที่เกี่ยวกับ log entry ซึ่งสามารถดึงออกมาวิเคราะห์ได้ง่ายขึ้นในภายหลัง

Log statement เป็นสิ่งที่ปกติแล้วเราจะใส่ไว้ตั้งแต่ตอนเขียนโปรแกรม เพื่อให้ข้อมูลที่ต้องใช้ debug อาจจะมีอยู่แล้ว และจริง ๆ แล้ว เมื่อพบและแก้ปัญหาโดยใช้ print statement ได้แล้ว มักจะคุ้มค่าที่จะแปลง print เหล่านั้นเป็น log statement ที่เหมาะสมก่อนที่จะลบออก วิธีนี้ทำให้ถ้ามี bug คล้าย ๆ กันเกิดขึ้นในอนาคต เราจะมีข้อมูลสำหรับวินิจฉัยอยู่แล้วโดยไม่ต้องแก้ไขโค้ด

> **Log จาก third-party**: โปรแกรมหลายตัว support flag `-v` หรือ `--verbose` เพื่อแสดงข้อมูลเพิ่มเติมตอนรัน ซึ่งมีประโยชน์ในการค้นหาว่าทำไมคำสั่งถึง fail บางตัวยังอนุญาตให้ใส่ flag ซ้ำเพื่อแสดงรายละเอียดมากขึ้น เมื่อ debug ปัญหาเกี่ยวกับ service (database, web server ฯลฯ) ให้ตรวจสอบ log ของมัน ซึ่งบน Linux มักอยู่ใน `/var/log/` ใช้ `journalctl -u <service>` เพื่อดู log ของ systemd service สำหรับ third-party library ให้ตรวจสอบว่า support debug logging ผ่าน environment variable หรือ configuration ได้หรือไม่

## Debugger

Print debugging ทำงานได้ดีเมื่อรู้ว่าจะ print อะไร และสามารถแก้ไขแล้ว run โค้ดใหม่ได้ง่าย Debugger จะมีค่ามากขึ้นเมื่อไม่แน่ใจว่าต้องการข้อมูลอะไร เมื่อ bug แสดงออกเฉพาะในสภาวะที่ยากต่อการ reproduce หรือเมื่อการแก้ไขและ restart โปรแกรมมีต้นทุนสูง (startup ใช้เวลานาน, state ที่ซับซ้อนที่ต้องสร้างขึ้นมาใหม่ เป็นต้น)

Debugger คือโปรแกรมที่ให้เราโต้ตอบกับการทำงานของโปรแกรมได้ขณะที่มันกำลังรันอยู่ ซึ่งทำให้สามารถ:

- หยุดการทำงานเมื่อถึงบรรทัดที่กำหนด
- ทำงานทีละ instruction
- ตรวจสอบค่าของตัวแปรหลังเกิด crash
- หยุดการทำงานแบบมีเงื่อนไขเมื่อเงื่อนไขที่กำหนดเป็นจริง
- และ feature ขั้นสูงอื่น ๆ อีกมากมาย

ภาษาโปรแกรมส่วนใหญ่ support (หรือมาพร้อมกับ) debugger บางรูปแบบ ที่หลากหลายที่สุดคือ **general-purpose debugger** เช่น [`gdb`](https://www.gnu.org/software/gdb/) (GNU Debugger) และ [`lldb`](https://lldb.llvm.org/) (LLVM Debugger) ซึ่งสามารถ debug native binary ใดก็ได้ หลายภาษายังมี **language-specific debugger** ที่ integrate กับ runtime ได้แน่นแฟ้นยิ่งขึ้น (เช่น pdb ของ Python หรือ jdb ของ Java)

`gdb` เป็น debugger มาตรฐาน de-facto สำหรับ C, C++, Rust และภาษา compiled อื่น ๆ มันช่วยให้ตรวจสอบ process ใดก็ได้และดู machine state ปัจจุบัน: register, stack, program counter และอื่น ๆ

คำสั่ง GDB ที่มีประโยชน์:

- `run` - เริ่มโปรแกรม
- `b {function}` หรือ `b {file}:{line}` - ตั้ง breakpoint
- `c` - ทำงานต่อ
- `step` / `next` / `finish` - step in / step over / step out
- `p {variable}` - แสดงค่าของตัวแปร
- `bt` - แสดง backtrace (call stack)
- `watch {expression}` - หยุดเมื่อค่าเปลี่ยนแปลง

> ลองใช้ TUI mode ของ GDB (`gdb -tui` หรือกด `Ctrl-x a` ภายใน GDB) เพื่อดูหน้าจอแบบแบ่งส่วนที่แสดง source code ควบคู่กับ command prompt

### Record-Replay Debugging

Bug ที่น่าหงุดหงิดที่สุดบางตัวคือ _Heisenbug_: bug ที่ดูเหมือนจะหายไปหรือเปลี่ยนพฤติกรรมเมื่อพยายามสังเกตมัน Race condition, bug ที่ขึ้นอยู่กับ timing และปัญหาที่เกิดขึ้นเฉพาะภายใต้เงื่อนไขของระบบบางอย่าง จัดอยู่ในหมวดนี้ การ debug แบบดั้งเดิมมักจะไร้ประโยชน์ในกรณีนี้ เพราะการรันโปรแกรมอีกครั้งจะให้พฤติกรรมที่ต่างออกไป (เช่น print statement อาจทำให้โค้ดช้าลงจนไม่เกิด race อีกต่อไป)

**Record-replay debugging** แก้ปัญหานี้โดยการบันทึกการทำงานของโปรแกรมและให้สามารถ replay ได้แบบ deterministic กี่ครั้งก็ได้ ยิ่งไปกว่านั้น ยังสามารถ _ย้อนกลับ_ ไปในการทำงานเพื่อหาจุดที่ผิดพลาดได้อีกด้วย

[rr](https://rr-project.org/) เป็นเครื่องมือที่ทรงพลังสำหรับ Linux ที่บันทึกการทำงานของโปรแกรมและอนุญาตให้ replay แบบ deterministic พร้อมความสามารถในการ debug เต็มรูปแบบ มันทำงานร่วมกับ GDB ดังนั้นจึงใช้ interface เดิมที่คุ้นเคยอยู่แล้ว

การใช้งานพื้นฐาน:

```bash
# บันทึกการทำงานของโปรแกรม
rr record ./my_program

# เล่นซ้ำจากการบันทึก (เปิด GDB)
rr replay
```

ความมหัศจรรย์เกิดขึ้นตอน replay เพราะการทำงานเป็นแบบ deterministic จึงสามารถใช้คำสั่ง **reverse debugging** ได้:

- `reverse-continue` (`rc`) - รันย้อนกลับจนถึง breakpoint
- `reverse-step` (`rs`) - ย้อนกลับทีละบรรทัด
- `reverse-next` (`rn`) - ย้อนกลับ โดยข้าม function call
- `reverse-finish` - รันย้อนกลับจนถึงจุดที่เข้าสู่ function ปัจจุบัน

สิ่งนี้ทรงพลังมากสำหรับการ debug สมมติว่าโปรแกรม crash — แทนที่จะเดาว่า bug อยู่ตรงไหนแล้วตั้ง breakpoint สามารถทำได้ดังนี้:

1. รันจนถึงจุด crash
2. ตรวจสอบ state ที่เสียหาย
3. ตั้ง watchpoint บนตัวแปรที่ถูก corrupt
4. `reverse-continue` เพื่อหาจุดที่มันถูก corrupt อย่างแม่นยำ

**เมื่อไหร่ควรใช้ rr:**
- test ที่ fail แบบไม่สม่ำเสมอ
- Race condition และ threading bug
- Crash ที่ reproduce ยาก
- bug ใด ๆ ที่อยากจะ "ย้อนเวลากลับไป"

> หมายเหตุ: rr ทำงานได้เฉพาะบน Linux และต้องใช้ hardware performance counter มันใช้งานไม่ได้ใน VM ที่ไม่เปิด counter เหล่านี้ เช่น AWS EC2 instance ส่วนใหญ่ และไม่ support GPU access สำหรับ macOS ลองดู [Warpspeed](https://warpspeed.dev/)

> **rr กับ concurrency**: เนื่องจาก rr บันทึกการทำงานแบบ deterministic มันจะ serialize thread scheduling ซึ่งหมายความว่า race condition บางอย่างอาจไม่แสดงออกภายใต้ rr หากขึ้นอยู่กับ timing ที่เฉพาะเจาะจง rr ยังคงมีประโยชน์สำหรับการ debug race — เมื่อจับ run ที่ fail ได้แล้ว สามารถ replay ได้อย่างน่าเชื่อถือ — แต่อาจต้องบันทึกหลายครั้งเพื่อจับ bug ที่เกิดแบบไม่สม่ำเสมอ สำหรับ bug ที่ไม่เกี่ยวกับ concurrency rr จะเปล่งประกายที่สุด: สามารถ reproduce การทำงานที่แน่นอนได้เสมอ และใช้ reverse debugging เพื่อตามล่าจุดที่ข้อมูลเสียหาย

## System Call Tracing

บางครั้งจำเป็นต้องเข้าใจว่าโปรแกรมโต้ตอบกับระบบปฏิบัติการอย่างไร โปรแกรมจะทำ [system call](https://en.wikipedia.org/wiki/System_call) เพื่อร้องขอบริการจาก kernel — เปิดไฟล์, จัดสรรหน่วยความจำ, สร้าง process และอื่น ๆ การ trace call เหล่านี้สามารถเปิดเผยได้ว่าทำไมโปรแกรมถึง hang, ไฟล์อะไรที่มันพยายามเข้าถึง หรือมันใช้เวลารอนานแค่ไหนตรงไหน

### strace (Linux) และ dtruss (macOS)

[`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) ช่วยให้สังเกต system call ทุกตัวที่โปรแกรมทำ:

```bash
# Trace system call ทั้งหมด
strace ./my_program

# Trace เฉพาะ call ที่เกี่ยวกับไฟล์
strace -e trace=file ./my_program

# ติดตาม child process (สำคัญสำหรับโปรแกรมที่เปิดโปรแกรมอื่น)
strace -f ./my_program

# Trace process ที่กำลังรันอยู่
strace -p <PID>

# แสดงข้อมูล timing
strace -T ./my_program
```

> บน macOS และ BSD ใช้ [`dtruss`](https://www.manpagez.com/man/1/dtruss/) (ซึ่ง wrap `dtrace`) สำหรับ functionality ที่คล้ายกัน:

> สำหรับเนื้อหาเจาะลึก `strace` ลองดู strace zine ที่ยอดเยี่ยมของ Julia Evans ได้ที่ [strace zine](https://jvns.ca/strace-zine-unfolded.pdf)

### bpftrace และ eBPF

[eBPF](https://ebpf.io/) (extended Berkeley Packet Filter) เป็นเทคโนโลยี Linux ที่ทรงพลัง ซึ่งอนุญาตให้รันโปรแกรมแบบ sandbox ใน kernel [`bpftrace`](https://github.com/iovisor/bpftrace) ให้ syntax ระดับสูงสำหรับเขียนโปรแกรม eBPF โปรแกรมเหล่านี้เป็นโปรแกรมที่รันอยู่ใน kernel จึงมีความสามารถในการแสดงออกสูงมาก (แม้จะมี syntax คล้าย awk ที่ค่อนข้างยุ่งยาก) use case ที่พบบ่อยที่สุดคือการตรวจสอบว่ามี system call อะไรถูกเรียกบ้าง รวมถึง aggregation (เช่น count หรือสถิติ latency) หรือการ introspect (หรือแม้แต่ filter ตาม) argument ของ system call

```bash
# Trace การเปิดไฟล์ทั้งระบบ (แสดงทันที)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# นับ system call ตามชื่อ (แสดงสรุปเมื่อกด Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

นอกจากนี้ยังสามารถเขียนโปรแกรม eBPF โดยตรงด้วยภาษา C โดยใช้ toolchain เช่น [`bcc`](https://github.com/iovisor/bcc) ซึ่งมาพร้อมกับ[เครื่องมือที่มีประโยชน์มากมาย](https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html) เช่น `biosnoop` สำหรับแสดง latency distribution ของ disk operation หรือ `opensnoop` สำหรับแสดงไฟล์ที่เปิดทั้งหมด

ในขณะที่ `strace` มีประโยชน์เพราะเริ่มใช้งานได้ง่าย แต่ `bpftrace` เป็นสิ่งที่ควรเลือกใช้เมื่อต้องการ overhead ต่ำกว่า ต้องการ trace ผ่าน kernel function ต้องการทำ aggregation ใด ๆ เป็นต้น โปรดทราบว่า `bpftrace` ต้องรันด้วยสิทธิ์ `root` และโดยทั่วไปจะ monitor kernel ทั้งหมด ไม่ใช่แค่ process เฉพาะ เพื่อ target โปรแกรมเฉพาะ สามารถ filter ตามชื่อคำสั่งหรือ PID ได้:

```bash
# Filter ตามชื่อคำสั่ง (แสดงสรุปเมื่อกด Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# Trace คำสั่งเฉพาะตั้งแต่เริ่มต้นโดยใช้ -c (cpid = child PID)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la'
```

flag `-c` จะรันคำสั่งที่ระบุและตั้ง `cpid` เป็น PID ของมัน ซึ่งมีประโยชน์สำหรับการ trace โปรแกรมตั้งแต่วินาทีที่มันเริ่มทำงาน เมื่อคำสั่งที่ถูก trace จบลง bpftrace จะแสดงผลลัพธ์ที่ aggregate ไว้

### Network Debugging

สำหรับปัญหาเกี่ยวกับเครือข่าย [`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) และ [Wireshark](https://www.wireshark.org/) ช่วยให้สามารถจับและวิเคราะห์ network packet ได้:

```bash
# จับ packet บน port 80
sudo tcpdump -i any port 80

# จับและบันทึกเป็นไฟล์สำหรับวิเคราะห์ด้วย Wireshark
sudo tcpdump -i any -w capture.pcap
```

สำหรับ HTTPS traffic การเข้ารหัสทำให้ tcpdump มีประโยชน์น้อยลง เครื่องมืออย่าง [mitmproxy](https://mitmproxy.org/) สามารถทำหน้าที่เป็น intercepting proxy เพื่อตรวจสอบ traffic ที่เข้ารหัสได้ Browser developer tools (แท็บ Network) มักเป็นวิธีที่ง่ายที่สุดในการ debug HTTPS request จาก web application — แสดง request/response data ที่ถอดรหัสแล้ว, header และ timing

## Memory Debugging

Memory bug — buffer overflow, use-after-free, memory leak — เป็น bug ที่อันตรายและ debug ยากที่สุดประเภทหนึ่ง มักจะไม่ crash ทันทีแต่จะ corrupt หน่วยความจำในลักษณะที่ก่อให้เกิดปัญหาในภายหลัง

### Sanitizer

วิธีหนึ่งในการหา memory bug คือการใช้ **sanitizer** ซึ่งเป็น feature ของ compiler ที่ instrument โค้ดเพื่อตรวจจับ error ขณะ runtime ตัวอย่างเช่น **AddressSanitizer (ASan)** ที่ใช้กันอย่างแพร่หลาย สามารถตรวจจับ:
- Buffer overflow (stack, heap และ global)
- Use-after-free
- Use-after-return
- Memory leak

```bash
# Compile ด้วย AddressSanitizer
gcc -fsanitize=address -g program.c -o program
./program
```

มี sanitizer ที่มีประโยชน์หลายตัว:

- **ThreadSanitizer (TSan)**: ตรวจจับ data race ในโค้ด multithreaded (`-fsanitize=thread`)
- **MemorySanitizer (MSan)**: ตรวจจับการอ่าน uninitialized memory (`-fsanitize=memory`)
- **UndefinedBehaviorSanitizer (UBSan)**: ตรวจจับ undefined behavior เช่น integer overflow (`-fsanitize=undefined`)

Sanitizer ต้อง recompile แต่เร็วพอที่จะใช้ใน CI pipeline และระหว่างการพัฒนาปกติได้

### Valgrind: เมื่อ Recompile ไม่ได้

[Valgrind](https://valgrind.org/) จะรันโปรแกรมในสิ่งที่คล้ายกับ virtual machine เพื่อตรวจจับ memory error มันช้ากว่า sanitizer แต่ไม่ต้อง recompile:

```bash
valgrind --leak-check=full ./my_program
```

ใช้ Valgrind เมื่อ:
- ไม่มี source code
- ไม่สามารถ recompile ได้ (third-party library)
- ต้องการเครื่องมือเฉพาะทางที่ไม่มีในรูปแบบ sanitizer

Valgrind เป็น controlled execution environment ที่ทรงพลังมาก และเราจะได้เห็นมันอีกในภายหลังเมื่อพูดถึง profiling

## AI สำหรับ Debugging

Large language model กลายเป็นผู้ช่วย debug ที่มีประโยชน์อย่างน่าประหลาดใจ มันเก่งในงาน debugging บางอย่างที่เสริมเครื่องมือแบบดั้งเดิมได้ดี

**จุดที่ LLM เก่ง:**

- **อธิบาย error message ที่เข้าใจยาก**: Compiler error โดยเฉพาะจาก C++ template หรือ borrow checker ของ Rust อาจเข้าใจยากมาก LLM สามารถแปลเป็นภาษาที่เข้าใจง่ายและแนะนำวิธีแก้ไขได้

- **ข้ามขอบเขตของภาษาและ abstraction**: หากกำลัง debug ปัญหาที่ครอบคลุมหลายภาษา (เช่น bug ใน C library ที่แสดงออกผ่าน Python binding) LLM สามารถช่วยนำทางผ่าน layer ต่าง ๆ ได้ มันเก่งเป็นพิเศษในเรื่อง FFI boundary, ปัญหา build system และ cross-language debugging (เช่น โปรแกรมของเรา error แต่เชื่อว่าเกิดจาก bug ใน dependency ตัวหนึ่ง)

- **เชื่อมโยงอาการกับสาเหตุ**: "โปรแกรมของเราทำงานได้ปกติแต่ใช้หน่วยความจำมากกว่าที่คาดไว้ 10 เท่า" เป็นอาการแบบกว้าง ๆ ที่ LLM สามารถช่วยตรวจสอบได้ โดยแนะนำสาเหตุที่เป็นไปได้และสิ่งที่ควรมองหา

- **วิเคราะห์ crash dump และ stack trace**: วาง stack trace แล้วถามว่าอะไรอาจเป็นสาเหตุ

> **หมายเหตุเกี่ยวกับ debug symbol**: สำหรับ stack trace ที่มีความหมายและการ debug ที่มีประสิทธิภาพ ต้องแน่ใจว่า binary (และ library ที่ link) ถูก compile ด้วย debug symbol (flag `-g`) Debug information มักจะเก็บในรูปแบบ DWARF นอกจากนี้ การ compile ด้วย frame pointer (`-fno-omit-frame-pointer`) จะทำให้ stack trace น่าเชื่อถือมากขึ้น โดยเฉพาะสำหรับ profiling tool หากไม่มีสิ่งเหล่านี้ stack trace อาจแสดงเฉพาะ memory address หรือไม่สมบูรณ์ สิ่งนี้สำคัญสำหรับโปรแกรมที่ compile เป็น native (C++, Rust) มากกว่า Python หรือ Java

**ข้อจำกัดที่ควรระวัง:**
- LLM สามารถ hallucinate คำอธิบายที่ฟังดูเข้าท่าแต่ผิดได้
- อาจแนะนำวิธีแก้ที่ปิดบัง bug แทนที่จะแก้มันจริง ๆ
- ควรตรวจสอบคำแนะนำด้วยเครื่องมือ debug จริงเสมอ
- ทำงานได้ดีที่สุดในฐานะส่วนเสริม ไม่ใช่สิ่งทดแทนการเข้าใจโค้ดของตัวเอง

> สิ่งนี้แตกต่างจาก[ความสามารถด้าน AI coding ทั่วไป](/2026/development-environment/#ai-powered-development)ที่กล่าวถึงในบท Development Environment ในที่นี้เราพูดถึงเฉพาะการใช้ LLM เป็นเครื่องมือช่วย debug

# Profiling

แม้ว่าโค้ดจะทำงานได้ตามที่คาดหวัง แต่นั่นอาจยังไม่ดีพอถ้ามันกิน CPU หรือหน่วยความจำทั้งหมดในกระบวนการทำงาน วิชา algorithm มักจะสอน big _O_ notation แต่ไม่ได้สอนวิธีหา hot spot ในโปรแกรม เนื่องจาก[premature optimization เป็นรากเหง้าของปัญหาทั้งปวง](https://wiki.c2.com/?PrematureOptimization) จึงควรเรียนรู้เกี่ยวกับ profiler และ monitoring tool เครื่องมือเหล่านี้จะช่วยให้เข้าใจว่าส่วนไหนของโปรแกรมใช้เวลาและ/หรือทรัพยากรมากที่สุด เพื่อให้โฟกัสการ optimize ไปที่ส่วนนั้น

## Timing

วิธีที่ง่ายที่สุดในการวัดประสิทธิภาพคือการจับเวลา ในหลายสถานการณ์ แค่ print เวลาที่โค้ดใช้ระหว่างสองจุดก็เพียงพอแล้ว

อย่างไรก็ตาม wall clock time อาจทำให้เข้าใจผิดได้ เพราะคอมพิวเตอร์อาจกำลังรัน process อื่นอยู่พร้อมกันหรือรอ event คำสั่ง `time` จะแยกแยะระหว่างเวลา _Real_, _User_ และ _Sys_:

- **Real** - Wall clock time ตั้งแต่เริ่มจนจบ รวมเวลารอด้วย
- **User** - เวลาที่ CPU ใช้รัน user code
- **Sys** - เวลาที่ CPU ใช้รัน kernel code

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s
```

ในที่นี้ request ใช้เวลาเกือบ 300 millisecond (real time) แต่ใช้เวลา CPU เพียง 107ms (user + sys) ส่วนที่เหลือเป็นการรอ network

## Resource Monitoring

บางครั้งขั้นตอนแรกในการวิเคราะห์ประสิทธิภาพของโปรแกรมคือการเข้าใจว่ามันใช้ทรัพยากรจริง ๆ เท่าไหร่ โปรแกรมมักจะรันช้าเมื่อถูกจำกัดทรัพยากร

- **General Monitoring**: [`htop`](https://htop.dev/) เป็น `top` เวอร์ชันปรับปรุงที่แสดงสถิติต่าง ๆ ของ process ที่กำลังรันอยู่ keybind ที่มีประโยชน์: `<F6>` เพื่อเรียงลำดับ process, `t` เพื่อแสดง tree hierarchy, `h` เพื่อ toggle thread นอกจากนี้ยังมี [`btop`](https://github.com/aristocratos/btop) ที่ monitor สิ่งต่าง ๆ ได้มากกว่า _อีกเยอะ_

- **I/O Operations**: [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) แสดงข้อมูลการใช้งาน I/O แบบ live

- **Memory Usage**: [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) แสดง total free และ used memory

- **Open Files**: [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) แสดงรายการข้อมูลไฟล์ที่ process เปิดอยู่ มีประโยชน์สำหรับตรวจสอบว่า process ไหนเปิดไฟล์เฉพาะ

- **Network Connections**: [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) ช่วยให้ monitor network connection ได้ use case ทั่วไปคือการหาว่า process ไหนใช้ port ที่ต้องการ: `ss -tlnp | grep :8080`

- **Network Usage**: [`nethogs`](https://github.com/raboof/nethogs) และ [`iftop`](https://pdw.ex-parrot.com/iftop/) เป็น CLI tool แบบ interactive ที่ดีสำหรับ monitor การใช้งาน network แยกตาม process

## Visualizing Performance Data

มนุษย์จับ pattern จากกราฟได้เร็วกว่าจากตารางตัวเลขมาก เมื่อวิเคราะห์ประสิทธิภาพ การ plot ข้อมูลมักจะเผยให้เห็น trend, spike และ anomaly ที่มองไม่เห็นในตัวเลขดิบ

**ทำให้ข้อมูลพร้อม plot ได้**: เมื่อเพิ่ม print หรือ log statement สำหรับ debug ให้พิจารณา format output ให้สามารถทำกราฟได้ง่ายในภายหลัง timestamp กับค่าในรูปแบบ CSV ง่าย ๆ (`1705012345,42.5`) นั้น plot ได้ง่ายกว่าประโยคข้อความมาก JSON-structured log ก็สามารถ parse และ plot ได้โดยใช้ความพยายามน้อย กล่าวอีกอย่างคือ log ข้อมูล[ในลักษณะที่เป็นระเบียบ](https://vita.had.co.nz/papers/tidy-data.pdf)

**Plot ด่วนด้วย gnuplot**: สำหรับการ plot แบบง่าย ๆ จาก command-line [`gnuplot`](http://www.gnuplot.info/) สามารถสร้างกราฟจาก data file ได้โดยตรง:

```bash
# Plot CSV ง่าย ๆ ที่มี timestamp,value
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines"
```

**สำรวจแบบ iterative ด้วย matplotlib และ ggplot2**: สำหรับการวิเคราะห์เชิงลึก [`matplotlib`](https://matplotlib.org/) ของ Python และ [`ggplot2`](https://ggplot2.tidyverse.org/) ของ R ช่วยให้สำรวจแบบ iterative ได้ ต่างจากการ plot แบบครั้งเดียว เครื่องมือเหล่านี้ให้สามารถ slice และ transform ข้อมูลได้อย่างรวดเร็วเพื่อตรวจสอบสมมติฐาน facet plot ของ ggplot2 นั้นทรงพลังเป็นพิเศษ — สามารถแบ่ง dataset เดียวออกเป็นหลาย subplot ตามหมวดหมู่ (เช่น faceting request latency ตาม endpoint หรือช่วงเวลา) เพื่อดึง pattern ที่มิฉะนั้นจะซ่อนอยู่ออกมา

**ตัวอย่างการใช้งาน:**
- การ plot request latency ตามเวลาจะเผยให้เห็นการช้าลงแบบเป็นคาบ (garbage collection, cron job, traffic pattern) ที่ percentile ดิบบดบัง
- การ visualize เวลาการ insert สำหรับ data structure ที่โตขึ้นเรื่อย ๆ สามารถเผยให้เห็นปัญหา algorithmic complexity — plot ของ vector insertion จะแสดง spike ที่เป็นลักษณะเฉพาะเมื่อ backing array เพิ่มขนาดเป็นเท่าตัว
- การ facet metric ตาม dimension ต่าง ๆ (ประเภท request, กลุ่มผู้ใช้, server) มักจะเผยให้เห็นว่าปัญหา "ทั้งระบบ" จริง ๆ แล้วจำกัดอยู่แค่หมวดหมู่เดียว

## CPU Profiler

เวลาที่คนพูดถึง _profiler_ ส่วนใหญ่จะหมายถึง _CPU profiler_ มีสองประเภทหลัก:

- **Tracing profiler** บันทึกทุก function call ที่โปรแกรมทำ
- **Sampling profiler** สุ่มตรวจโปรแกรมเป็นระยะ ๆ (โดยทั่วไปทุก millisecond) และบันทึก stack ของโปรแกรม

Sampling profiler มี overhead ต่ำกว่าและเป็นที่นิยมมากกว่าสำหรับใช้งานใน production

### perf: sampling profiler

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) เป็น profiler มาตรฐานของ Linux สามารถ profile โปรแกรมใดก็ได้โดยไม่ต้อง recompile:

`perf stat` ให้ภาพรวมคร่าว ๆ ว่าเวลาถูกใช้ที่ไหน:

```bash
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches
```

Output ของ profiler สำหรับโปรแกรมในโลกจริงจะมีข้อมูลจำนวนมาก มนุษย์เป็นสิ่งมีชีวิตที่เน้นการมองเห็นและไม่ถนัดในการอ่านตัวเลขจำนวนมาก [Flame graph](https://www.brendangregg.com/flamegraphs.html) เป็นการ visualize ที่ทำให้ข้อมูลจาก profiling เข้าใจง่ายขึ้นมาก

Flame graph แสดง hierarchy ของ function call ตามแกน Y และเวลาที่ใช้เป็นสัดส่วนตามแกน X มันเป็นแบบ interactive — สามารถคลิกเพื่อ zoom เข้าไปในส่วนเฉพาะของโปรแกรมได้

[![FlameGraph](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

วิธีสร้าง flame graph จากข้อมูล `perf`:

```bash
# บันทึก profile
perf record -g ./my_program

# สร้าง flame graph (ต้องใช้ flamegraph script)
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

> ลองใช้ [Speedscope](https://www.speedscope.app/) สำหรับ flame graph viewer แบบ interactive บนเว็บ หรือ [Perfetto](https://perfetto.dev/) สำหรับการวิเคราะห์ระดับระบบที่ครอบคลุม

### Callgrind ของ Valgrind: tracing profiler

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html) เป็นเครื่องมือ profiling ที่บันทึกประวัติการเรียกและจำนวน instruction ของโปรแกรม ต่างจาก sampling profiler มันให้จำนวนการเรียกที่แม่นยำและสามารถแสดงความสัมพันธ์ระหว่าง caller และ callee ได้:

```bash
# รันด้วย callgrind
valgrind --tool=callgrind ./my_program

# วิเคราะห์ด้วย callgrind_annotate (text) หรือ kcachegrind (GUI)
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>
```

Callgrind ช้ากว่า sampling profiler แต่ให้จำนวนการเรียกที่แม่นยำ และสามารถจำลองพฤติกรรม cache ได้ (ด้วย `--cache-sim=yes`) หากต้องการข้อมูลนั้น

> หากใช้ภาษาเฉพาะ อาจมี profiler ที่เฉพาะทางมากกว่า ตัวอย่างเช่น Python มี [`cProfile`](https://docs.python.org/3/library/profile.html) และ [`py-spy`](https://github.com/benfred/py-spy), Go มี [`go tool pprof`](https://pkg.go.dev/cmd/pprof) และ Rust มี [`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph)

## Memory Profiler

Memory profiler ช่วยให้เข้าใจว่าโปรแกรมใช้หน่วยความจำอย่างไรตามเวลา และช่วยหา memory leak

### Massif ของ Valgrind

[`massif`](https://valgrind.org/docs/manual/ms-manual.html) profile การใช้ heap memory:

```bash
valgrind --tool=massif ./my_program
ms_print massif.out.<pid>
```

สิ่งนี้จะแสดงการใช้ heap ตามเวลา ช่วยระบุ memory leak และการ allocate ที่มากเกินไป

> สำหรับ Python [`memory-profiler`](https://pypi.org/project/memory-profiler/) ให้ข้อมูลการใช้หน่วยความจำแบบแยกรายบรรทัด

## Benchmarking

เมื่อต้องการเปรียบเทียบประสิทธิภาพของ implementation หรือเครื่องมือที่ต่างกัน [`hyperfine`](https://github.com/sharkdp/hyperfine) เป็นเครื่องมือที่ยอดเยี่ยมสำหรับ benchmark โปรแกรม command-line:

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

> สำหรับ web development browser developer tools มี profiler ที่ยอดเยี่ยม ดู [Firefox Profiler](https://profiler.firefox.com/docs/) และ [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools) documentation

# แบบฝึกหัด

## Debugging

1. **Debug sorting algorithm**: pseudocode ต่อไปนี้ implement merge sort แต่มี bug อยู่ ให้ implement ในภาษาที่ต้องการ แล้วใช้ debugger (gdb, lldb, pdb หรือ debugger ของ IDE) เพื่อหาและแก้ bug

   ```
   function merge_sort(arr):
       if length(arr) <= 1:
           return arr
       mid = length(arr) / 2
       left = merge_sort(arr[0..mid])
       right = merge_sort(arr[mid..end])
       return merge(left, right)

   function merge(left, right):
       result = []
       i = 0, j = 0
       while i < length(left) AND j < length(right):
           if left[i] <= right[j]:
               append result, left[i]
               i = i + 1
           else:
               append result, right[i]
               j = j + 1
       append remaining elements from left and right
       return result
   ```

   Test vector: `merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` ควรให้ผลลัพธ์เป็น `[1, 1, 2, 3, 4, 5, 6, 9]` ใช้ breakpoint และ step ผ่าน merge function เพื่อหาจุดที่เลือก element ผิด

2. ติดตั้ง [`rr`](https://rr-project.org/) และใช้ reverse debugging เพื่อหา corruption bug บันทึกโปรแกรมนี้เป็น `corruption.c`:

   ```c
   #include <stdio.h>

   typedef struct {
       int id;
       int scores[3];
   } Student;

   Student students[2];

   void init() {
       students[0].id = 1001;
       students[0].scores[0] = 85;
       students[0].scores[1] = 92;
       students[0].scores[2] = 78;

       students[1].id = 1002;
       students[1].scores[0] = 90;
       students[1].scores[1] = 88;
       students[1].scores[2] = 95;
   }

   void curve_scores(int student_idx, int curve) {
       for (int i = 0; i < 4; i++) {
           students[student_idx].scores[i] += curve;
       }
   }

   int main() {
       init();
       printf("=== Initial state ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       curve_scores(0, 5);

       printf("\n=== After curving ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       if (students[1].id != 1002) {
           printf("\nERROR: Student 1's ID was corrupted! Expected 1002, got %d\n",
                  students[1].id);
           return 1;
       }
       return 0;
   }
   ```

   Compile ด้วย `gcc -g corruption.c -o corruption` แล้วรัน ID ของ Student 1 จะถูก corrupt แต่การ corrupt เกิดขึ้นใน function ที่แตะแค่ student 0 เท่านั้น ใช้ `rr record ./corruption` และ `rr replay` เพื่อหาผู้ต้องสงสัย ตั้ง watchpoint บน `students[1].id` แล้วใช้ `reverse-continue` หลังจากเกิด corruption เพื่อหาว่าบรรทัดไหนของโค้ดเขียนทับค่า

3. Debug memory error ด้วย AddressSanitizer บันทึกโค้ดนี้เป็น `uaf.c`:

   ```c
   #include <stdlib.h>
   #include <string.h>
   #include <stdio.h>

   int main() {
       char *greeting = malloc(32);
       strcpy(greeting, "Hello, world!");
       printf("%s\n", greeting);

       free(greeting);

       greeting[0] = 'J';
       printf("%s\n", greeting);

       return 0;
   }
   ```

   ก่อนอื่น compile และรันโดยไม่ใส่ sanitizer: `gcc uaf.c -o uaf && ./uaf` มันอาจดูเหมือนทำงานได้ปกติ จากนั้น compile ด้วย AddressSanitizer: `gcc -fsanitize=address -g uaf.c -o uaf && ./uaf` อ่าน error report ดู ASan พบ bug อะไร? แก้ไขปัญหาที่มันระบุ

4. ใช้ `strace` (Linux) หรือ `dtruss` (macOS) เพื่อ trace system call ที่คำสั่งอย่าง `ls -l` ทำ มันทำ system call อะไรบ้าง? ลอง trace โปรแกรมที่ซับซ้อนมากขึ้นและดูว่ามันเปิดไฟล์อะไร

5. ใช้ LLM เพื่อช่วย debug error message ที่เข้าใจยาก ลอง copy compiler error (โดยเฉพาะจาก C++ template หรือ Rust) แล้วขอคำอธิบายและวิธีแก้ ลองใส่ output บางส่วนจาก `strace` หรือ address sanitizer ลงไปด้วย

## Profiling

1. ใช้ `perf stat` เพื่อดูสถิติประสิทธิภาพพื้นฐานของโปรแกรมที่เลือก counter ต่าง ๆ หมายถึงอะไร?

2. Profile ด้วย `perf record` บันทึกโค้ดนี้เป็น `slow.c`:

   ```c
   #include <math.h>
   #include <stdio.h>

   double slow_computation(int n) {
       double result = 0;
       for (int i = 0; i < n; i++) {
           for (int j = 0; j < 1000; j++) {
               result += sin(i * j) * cos(i + j);
           }
       }
       return result;
   }

   int main() {
       double r = 0;
       for (int i = 0; i < 100; i++) {
           r += slow_computation(1000);
       }
       printf("Result: %f\n", r);
       return 0;
   }
   ```

   Compile ด้วย debug symbol: `gcc -g -O2 slow.c -o slow -lm` รัน `perf record -g ./slow` จากนั้น `perf report` เพื่อดูว่าเวลาถูกใช้ที่ไหน ลองสร้าง flame graph โดยใช้ flamegraph script

3. ใช้ `hyperfine` เพื่อ benchmark สอง implementation ที่ต่างกันของงานเดียวกัน (เช่น `find` กับ `fd`, `grep` กับ `ripgrep` หรือสองเวอร์ชันของโค้ดตัวเอง)

4. ใช้ `htop` เพื่อ monitor ระบบขณะรันโปรแกรมที่ใช้ทรัพยากรเยอะ ลองใช้ `taskset` เพื่อจำกัดว่า process จะใช้ CPU ตัวไหนได้: `taskset --cpu-list 0,2 stress -c 3` ทำไม `stress` ถึงไม่ใช้สาม CPU?

5. ปัญหาที่พบบ่อยคือ port ที่ต้องการ listen ถูกใช้โดย process อื่นอยู่แล้ว เรียนรู้วิธีหา process นั้น: ก่อนอื่นรัน `python -m http.server 4444` เพื่อเริ่ม web server ขั้นต่ำบน port 4444 บน terminal อื่นรัน `ss -tlnp | grep 4444` เพื่อหา process จากนั้น terminate ด้วย `kill <PID>`
