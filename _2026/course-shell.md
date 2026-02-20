---
layout: lecture
title: "Course Overview + Introduction to the Shell"
description: >
    เรียนรู้แรงจูงใจของคอร์สนี้ และเริ่มต้นใช้งาน shell
thumbnail: /static/assets/thumbnails/2026/lec1.png
date: 2026-01-12
ready: true
video:
    aspect: 56.25
    id: MSgoeuMqUmU
---

# Who are we?

คอร์สนี้สอนโดย [Anish](https://anish.io/),
[Jon](https://thesquareplanet.com/) และ [Jose](http://josejg.com/) ทุกคนเป็นอดีตนักศึกษา MIT ที่เริ่มสอนคลาส MIT IAP นี้ตั้งแต่สมัยยังเป็นนักศึกษา สามารถติดต่อพวกเราได้ที่
[missing-semester@mit.edu](mailto:missing-semester@mit.edu)

พวกเราไม่ได้รับค่าตอบแทนจากการสอนคอร์สนี้ และไม่ได้สร้างรายได้จากคอร์สนี้แต่อย่างใด [เนื้อหาทั้งหมด](https://missing.csail.mit.edu/) และ [วิดีโอบันทึกการสอน](https://www.youtube.com/@MissingSemester) เปิดให้เข้าถึงได้ฟรี ถ้าอยากสนับสนุนผลงานของพวกเรา วิธีที่ดีที่สุดคือช่วยบอกต่อเกี่ยวกับคอร์สนี้ ถ้าเป็นบริษัท มหาวิทยาลัย หรือองค์กรที่นำเนื้อหาไปใช้กับกลุ่มคนจำนวนมาก ส่ง experience reports/testimonials มาทางอีเมลให้พวกเราได้รับรู้ด้วย :)

# Motivation

ในฐานะนักวิทยาการคอมพิวเตอร์ เราย่อมรู้ว่าคอมพิวเตอร์เก่งมากเรื่องงานซ้ำๆ แต่บ่อยครั้งเกินไปที่เราลืมว่าสิ่งนี้ใช้ได้กับ _การใช้งาน_ คอมพิวเตอร์ของเราเองเช่นกัน ไม่ใช่แค่กับ computations ที่เราเขียนโปรแกรมให้ทำ เรามีเครื่องมือมากมายที่ช่วยให้ทำงานได้มีประสิทธิภาพมากขึ้นและแก้ปัญหาที่ซับซ้อนขึ้นได้ แต่พวกเราหลายคนใช้เครื่องมือเหล่านั้นแค่เศษเสี้ยว รู้แค่คำสั่งมหัศจรรย์ไม่กี่อันแบบท่องจำ แล้วก็ copy-paste คำสั่งจากอินเทอร์เน็ตมาใช้เวลาติดปัญหา

คอร์สนี้คือความพยายามที่จะ[แก้ปัญหานี้](/about/)

เราต้องการสอนให้ใช้เครื่องมือที่รู้จักอยู่แล้วได้อย่างเต็มประสิทธิภาพ แนะนำเครื่องมือใหม่ๆ เข้า toolbox และหวังว่าจะจุดประกายความสนใจในการสำรวจ (และอาจสร้าง) เครื่องมือเพิ่มเติมด้วยตัวเอง นี่คือสิ่งที่เราเชื่อว่าเป็น missing semester ที่หายไปจากหลักสูตร Computer Science ส่วนใหญ่

# Class structure

คอร์สนี้ไม่นับหน่วยกิต ประกอบด้วยบรรยาย 9 ครั้ง ครั้งละ 1 ชั่วโมง แต่ละครั้งเน้น[หัวข้อเฉพาะ](/2026/) แต่ละบรรยายค่อนข้างเป็นอิสระจากกัน แม้ว่าเมื่อเรียนไปเรื่อยๆ เราจะสมมติว่าคุณคุ้นเคยกับเนื้อหาจากบทก่อนหน้า เนื้อหาบรรยายมีออนไลน์ แต่อาจมีบางส่วนที่สอนในคลาส (เช่น demo) ที่ไม่ได้อยู่ในเนื้อหาเขียน เช่นเดียวกับปีก่อนๆ เราจะบันทึกวิดีโอบรรยายและโพสต์[ออนไลน์](https://www.youtube.com/@MissingSemester)

เราพยายามครอบคลุมเนื้อหาจำนวนมากภายในบรรยาย 1 ชั่วโมงเพียงไม่กี่ครั้ง ดังนั้นแต่ละบรรยายจะค่อนข้างหนาแน่น เพื่อให้มีเวลาทำความเข้าใจเนื้อหาตามจังหวะของตัวเอง แต่ละบรรยายมีชุดแบบฝึกหัดที่นำทางผ่านจุดสำคัญของบท เราไม่มี office hours แต่แนะนำให้ถามคำถามใน [OSSU Discord](https://ossu.dev/#community) ที่ช่อง `#missing-semester-forum` หรือส่งอีเมลมาที่
[missing-semester@mit.edu](mailto:missing-semester@mit.edu)

เนื่องจากเวลาจำกัด เราไม่สามารถครอบคลุมเครื่องมือทุกตัวในระดับรายละเอียดเท่ากับคอร์สเต็มรูปแบบ เมื่อเป็นไปได้ เราจะชี้แนะแหล่งข้อมูลสำหรับศึกษาเพิ่มเติมเกี่ยวกับเครื่องมือหรือหัวข้อนั้นๆ แต่ถ้าหัวข้อไหนน่าสนใจเป็นพิเศษ อย่าลังเลที่จะติดต่อมาถามได้เลย

สุดท้าย ถ้ามี feedback เกี่ยวกับคอร์ส ส่งมาทางอีเมลที่ [missing-semester@mit.edu](mailto:missing-semester@mit.edu) ได้เลย

# Topic 1: The Shell

{% comment %}
ผู้บรรยาย: Jon
{% endcomment %}

## What is the shell?

คอมพิวเตอร์ทุกวันนี้มี interface หลากหลายสำหรับสั่งงาน ไม่ว่าจะเป็น graphical user interface, voice interface, AR/VR และล่าสุดคือ LLMs สิ่งเหล่านี้ใช้ได้ดีสำหรับ 80% ของกรณีใช้งาน แต่มักมีข้อจำกัดพื้นฐานในสิ่งที่ทำได้ --- กดปุ่มที่ไม่มีอยู่ไม่ได้ หรือสั่งเสียงในสิ่งที่ไม่ได้ถูกโปรแกรมไว้ไม่ได้ เพื่อใช้ประโยชน์จากเครื่องมือในคอมพิวเตอร์ได้อย่างเต็มที่ เราต้องย้อนกลับไปหา textual interface: นั่นคือ Shell

แทบทุก platform ที่หยิบมาใช้จะมี shell ในรูปแบบใดรูปแบบหนึ่ง และหลาย platform มีหลาย shell ให้เลือก แม้จะแตกต่างกันในรายละเอียด แต่แก่นแท้ของมันเหมือนกัน: shell ให้เราสั่งรันโปรแกรม ป้อน input และตรวจสอบ output ในรูปแบบ semi-structured

เพื่อเปิด shell _prompt_ (ที่สามารถพิมพ์คำสั่งได้) ต้องมี _terminal_ ก่อน ซึ่งคือ visual interface ของ shell อุปกรณ์ส่วนใหญ่จะมี terminal มาให้แล้ว หรือติดตั้งเพิ่มได้ง่าย:

- **Linux:**
  กด `Ctrl + Alt + T` (ใช้ได้กับ distribution ส่วนใหญ่) หรือค้นหา
  "Terminal" ในเมนูแอปพลิเคชัน
- **Windows:**
  กด `Win + R` พิมพ์ `cmd` หรือ `powershell` แล้วกด Enter
  หรือค้นหา "Terminal" หรือ "Command Prompt" ใน Start menu
- **macOS:**
  กด `Cmd + Space` เพื่อเปิด Spotlight พิมพ์ "Terminal" แล้วกด Enter
  หรือหาได้ใน Applications → Utilities → Terminal

บน Linux และ macOS ปกติจะเปิด Bourne Again SHell หรือเรียกสั้นๆ ว่า "bash" ซึ่งเป็น shell ที่ใช้กันแพร่หลายที่สุดตัวหนึ่ง และ syntax ของมันคล้ายกับ shell อื่นๆ อีกหลายตัว บน Windows จะเจอ "batch" หรือ "powershell" ขึ้นอยู่กับคำสั่งที่รัน shell เหล่านี้เป็นเฉพาะของ Windows และไม่ใช่สิ่งที่เราจะเน้นในคอร์สนี้ แม้จะมีสิ่งที่เทียบเคียงกันได้กับเนื้อหาส่วนใหญ่ที่สอน สิ่งที่ควรใช้แทนคือ [Windows Subsystem for
Linux](https://docs.microsoft.com/en-us/windows/wsl/) หรือ Linux virtual
machine

Shell อื่นๆ ก็มีอยู่เช่นกัน มักมาพร้อมการปรับปรุงด้าน ergonomics เหนือกว่า bash (fish และ zsh เป็นที่นิยมมากที่สุด) แม้ shell เหล่านี้จะได้รับความนิยมสูง (ผู้สอนทุกคนใช้อยู่) แต่ก็ไม่ได้แพร่หลายเท่า bash และใช้ concepts พื้นฐานเดียวกันหลายอย่าง จึงไม่ได้เน้นในบรรยายนี้

## Why should you care about it?

Shell ไม่ใช่แค่ (โดยปกติ) เร็วกว่าการ "คลิกไปเรื่อย" มาก แต่ยังมาพร้อม expressive power ที่หาได้ยากในโปรแกรมแบบ graphical ตัวใดตัวหนึ่ง อย่างที่จะเห็นกัน shell ให้ความสามารถในการ _รวม_ โปรแกรมต่างๆ เข้าด้วยกันอย่างสร้างสรรค์เพื่อทำให้งานเกือบทุกอย่างเป็นอัตโนมัติ

การรู้จักใช้ shell ยังเป็นประโยชน์อย่างมากสำหรับการท่องโลกของ open-source software (ที่มักมีคำแนะนำการติดตั้งที่ต้องใช้ shell) การสร้าง continuous integration สำหรับโปรเจกต์ซอฟต์แวร์ (อย่างที่อธิบายใน[บรรยาย Code Quality](/2026/code-quality/)) และการ debug errors เมื่อโปรแกรมอื่นล้มเหลว

## Navigating in the shell

เมื่อเปิด terminal จะเห็น _prompt_ ที่มักมีหน้าตาประมาณนี้:

```console
missing:~$ 
```

นี่คือ textual interface หลักของ shell มันบอกว่าอยู่บนเครื่อง `missing` และ "current working directory" หรือตำแหน่งปัจจุบันคือ `~` (ย่อมาจาก "home") เครื่องหมาย `$` บอกว่าไม่ใช่ root user (จะพูดถึงเพิ่มเติมทีหลัง) ที่ prompt นี้สามารถพิมพ์ _command_ ซึ่ง shell จะนำไปตีความ คำสั่งพื้นฐานที่สุดคือการสั่งรันโปรแกรม:

```console
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$ 
```

ตรงนี้เราสั่งรันโปรแกรม `date` ซึ่ง (ไม่น่าแปลกใจ) จะพิมพ์วันที่และเวลาปัจจุบัน จากนั้น shell จะถามคำสั่งถัดไป เราสามารถรันคำสั่งพร้อม _arguments_ ได้ด้วย:

```console
missing:~$ echo hello
hello
```

ในกรณีนี้ เราบอก shell ให้รันโปรแกรม `echo` ด้วย argument `hello` โปรแกรม `echo` แค่พิมพ์ arguments ที่ได้รับออกมา shell จะ parse คำสั่งโดยแยกด้วย whitespace จากนั้นรันโปรแกรมตามคำแรก โดยส่งคำที่ตามมาแต่ละคำเป็น argument ให้โปรแกรม ถ้าต้องการใส่ argument ที่มีช่องว่างหรืออักขระพิเศษ (เช่น directory ชื่อ "My Photos") สามารถครอบด้วย `'` หรือ `"` (`"My Photos"`) หรือ escape เฉพาะอักขระที่เกี่ยวข้องด้วย `\` (`My\ Photos`)

คำสั่งที่สำคัญที่สุดเมื่อเริ่มต้นคือ `man` ย่อมาจาก "manual" โปรแกรม `man` ช่วยให้ดูข้อมูลเพิ่มเติมเกี่ยวกับคำสั่งใดก็ได้ในระบบ ตัวอย่างเช่น ถ้ารัน `man date` จะอธิบายว่า `date` คืออะไร และ arguments ต่างๆ ที่ใส่ได้เพื่อเปลี่ยนพฤติกรรม นอกจากนี้ยังสามารถดู help แบบย่อได้โดยใส่ `--help` เป็น argument ให้กับคำสั่งส่วนใหญ่

> แนะนำให้ติดตั้งและใช้ [`tldr`](https://tldr.sh/) เพิ่มเติมจาก `man`
> เพราะจะแสดงตัวอย่างการใช้งานทั่วไปให้เห็นเลยใน terminal นอกจากนี้ LLMs
> มักอธิบายวิธีทำงานของคำสั่งและวิธีเรียกใช้เพื่อทำสิ่งที่ต้องการได้ดีมาก

หลังจาก `man` แล้ว คำสั่งสำคัญถัดมาคือ `cd` หรือ "change directory" คำสั่งนี้เป็น built-in ของ shell ไม่ใช่โปรแกรมแยกต่างหาก (คือ `which cd` จะบอกว่า "no cd found") ใส่ path เข้าไป แล้ว path นั้นจะกลายเป็น current working directory ซึ่งจะเห็น working directory สะท้อนอยู่ใน shell prompt:

```console
missing:~$ cd /bin
missing:/bin$ cd /
missing:/$ cd ~
missing:~$ 
```

> สังเกตว่า shell มี auto-completion ดังนั้นมักจะพิมพ์ path ได้เร็วขึ้น
> ด้วยการกด `<TAB>`!

คำสั่งหลายตัวจะทำงานกับ current working directory ถ้าไม่ได้ระบุอะไรเพิ่ม ถ้าไม่แน่ใจว่าอยู่ที่ไหน สามารถรัน `pwd` หรือพิมพ์ environment variable `$PWD` (ด้วย `echo $PWD`) ทั้งสองจะแสดง current working directory

Current working directory ยังมีประโยชน์ตรงที่ช่วยให้ใช้ _relative_ paths ได้ path ทั้งหมดที่เห็นมาจนถึงตอนนี้เป็น _absolute_ --- คือเริ่มต้นด้วย `/` และระบุชุด directory ทั้งหมดที่ต้องผ่านเพื่อไปยังตำแหน่งนั้นจาก root ของ file system (`/`) ในทางปฏิบัติ มักใช้ relative paths มากกว่า เรียกว่า relative เพราะมันสัมพัทธ์กับ current working directory ใน relative path (อะไรก็ตามที่ _ไม่_ ขึ้นต้นด้วย `/`) component แรกจะถูกค้นหาใน current working directory และ component ถัดๆ ไปจะ traverse ตามปกติ ตัวอย่าง:

```console
missing:~$ cd /
missing:/$ cd bin
missing:/bin$ 
```

นอกจากนี้ยังมี component "พิเศษ" สองตัวที่มีอยู่ในทุก directory: `.` และ `..` โดย `.` คือ "directory นี้" และ `..` คือ "parent directory" ดังนั้น:

```console
missing:~$ cd /
missing:/$ cd bin/../bin/../bin/././../bin/..
missing:/$ 
```

สามารถใช้ absolute path และ relative path สลับกันได้กับ argument ของคำสั่งใดก็ได้ แค่จำไว้ว่า current working directory คืออะไรเมื่อใช้ relative path!

> แนะนำให้ติดตั้งและใช้
> [`zoxide`](https://github.com/ajeetdsouza/zoxide) เพื่อเร่งความเร็วการ
> `cd` --- `z` จะจำ path ที่เข้าบ่อยและให้เข้าถึงได้โดยพิมพ์น้อยลง

## What is available in the shell?

แล้ว shell รู้ได้อย่างไรว่าจะหาโปรแกรมอย่าง `date` หรือ `echo` ได้ที่ไหน? เมื่อ shell ถูกสั่งให้รันคำสั่ง มันจะดู _environment variable_ ชื่อ `$PATH` ซึ่งเก็บรายการ directory ที่ shell จะค้นหาโปรแกรม:

```console
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

เมื่อรันคำสั่ง `echo` shell จะเห็นว่าต้องรันโปรแกรม `echo` จากนั้นค้นหาในรายการ directory ที่คั่นด้วย `:` ใน `$PATH` เพื่อหาไฟล์ที่มีชื่อตรงกัน เมื่อเจอก็จะรัน (สมมติว่าไฟล์เป็น _executable_ ซึ่งจะพูดถึงเพิ่มเติมทีหลัง) เราสามารถหาว่าโปรแกรมชื่อนั้นรันไฟล์ไหนด้วยโปรแกรม `which` นอกจากนี้ยังข้าม `$PATH` ได้เลยโดยระบุ _path_ ของไฟล์ที่ต้องการรันโดยตรง

วิธีนี้ยังเป็นเบาะแสว่าเราจะหาโปรแกรม _ทั้งหมด_ ที่รันได้ใน shell ได้อย่างไร: ก็คือ list เนื้อหาของทุก directory บน `$PATH` ทำได้โดยส่ง path ของ directory ให้โปรแกรม `ls` ซึ่ง list ไฟล์:

```console
missing:~$ ls /bin
```

> แนะนำให้ติดตั้งและใช้ [`eza`](https://eza.rocks/) สำหรับ `ls`
> ที่เป็นมิตรกับผู้ใช้มากขึ้น

คำสั่งนี้จะพิมพ์โปรแกรม _จำนวนมาก_ บนคอมพิวเตอร์ส่วนใหญ่ แต่เราจะเน้นแค่บางตัวที่สำคัญที่สุด เริ่มจากตัวที่ง่ายก่อน:

- `cat file` พิมพ์เนื้อหาของ `file`
- `sort file` พิมพ์บรรทัดของ `file` ในลำดับที่เรียงแล้ว
- `uniq file` ลบบรรทัดที่ซ้ำกันติดกันออกจาก `file`
- `head file` และ `tail file` พิมพ์บรรทัดแรกๆ และบรรทัดท้ายๆ ของ `file` ตามลำดับ

> แนะนำให้ติดตั้งและใช้ [`bat`](https://github.com/sharkdp/bat)
> แทน `cat` สำหรับ syntax highlighting และ scrolling

นอกจากนี้ยังมี `grep pattern file` ที่ค้นหาบรรทัดที่ตรงกับ `pattern` ใน `file` คำสั่งนี้สมควรได้รับความสนใจมากกว่าเล็กน้อยเพราะมัน _มีประโยชน์มาก_ และมี features หลากหลายกว่าที่คาดคิด `pattern` เป็น _regular expression_ ที่สามารถแสดง patterns ที่ซับซ้อนมากได้ --- เราจะ[พูดถึงเรื่องนี้](/2026/code-quality/#regular-expressions)ในบรรยาย code quality นอกจากนี้ยังระบุ directory แทนไฟล์ได้ (หรือไม่ใส่เลยสำหรับ `.`) แล้วใส่ `-r` เพื่อค้นหา recursively ในทุกไฟล์ภายใน directory

> แนะนำให้ติดตั้งและใช้
> [`ripgrep`](https://github.com/BurntSushi/ripgrep) แทน `grep` สำหรับ
> ทางเลือกที่เร็วกว่าและเป็นมิตรกับผู้ใช้มากกว่า (แต่ portable น้อยกว่า)
> `ripgrep` ยังค้นหา current working directory แบบ recursive
> โดย default อีกด้วย!

นอกจากนี้ยังมีเครื่องมือที่มีประโยชน์มากแต่ interface ซับซ้อนกว่าเล็กน้อย ตัวแรกคือ `sed` ซึ่งเป็น programmatic file editor มันมีภาษาโปรแกรมของตัวเองสำหรับแก้ไขไฟล์แบบอัตโนมัติ แต่การใช้งานที่พบบ่อยที่สุดคือ:

```console
missing:~$ sed -i 's/pattern/replacement/g' file
```

คำสั่งนี้แทนที่ `pattern` ทั้งหมดด้วย `replacement` ใน `file` โดย `-i` บ่งบอกว่าต้องการให้แทนที่แบบ inline (แทนที่จะปล่อย `file` ไม่เปลี่ยนแปลงแล้วพิมพ์เนื้อหาที่แทนที่แล้วออกมา) `s/` เป็นวิธีบอกในภาษา sed ว่าต้องการทำ substitution เครื่องหมาย `/` แยก pattern ออกจาก replacement และ `/g` ที่ท้ายบอกว่าต้องการแทนที่ _ทั้งหมด_ ในแต่ละบรรทัด ไม่ใช่แค่ตัวแรก เช่นเดียวกับ `grep` ตัว `pattern` ตรงนี้เป็น regular expression ซึ่งให้ expressive power อย่างมาก Regular expression substitutions ยังช่วยให้ `replacement` อ้างอิงกลับไปยังส่วนต่างๆ ของ pattern ที่ match ได้ด้วย จะเห็นตัวอย่างในอีกสักครู่

ถัดมาคือ `find` ที่ช่วยค้นหาไฟล์ (แบบ recursive) ที่ตรงตามเงื่อนไขที่กำหนด ตัวอย่าง:

```console
missing:~$ find ~/Downloads -type f -name "*.zip" -mtime +30
```

ค้นหาไฟล์ ZIP ใน download directory ที่เก่ากว่า 30 วัน

```console
missing:~$ find ~ -type f -size +100M -exec ls -lh {} \;
```

ค้นหาไฟล์ที่ใหญ่กว่า 100M ใน home directory และ list ออกมา สังเกตว่า `-exec` รับ _command_ ที่ปิดท้ายด้วย `;` แบบเดี่ยว (ซึ่งต้อง escape เหมือนกับช่องว่าง) โดย `{}` จะถูกแทนที่ด้วยแต่ละ file path ที่ match โดย `find`

```console
missing:~$ find . -name "*.py" -exec grep -l "TODO" {} \;
```

ค้นหาไฟล์ `.py` ที่มี TODO อยู่ข้างใน

Syntax ของ `find` อาจดูน่ากลัวเล็กน้อย แต่หวังว่าจะเห็นภาพว่ามันมีประโยชน์แค่ไหน!

> แนะนำให้ติดตั้งและใช้ [`fd`](https://github.com/sharkdp/fd)
> แทน `find` สำหรับประสบการณ์ที่เป็นมิตรกับผู้ใช้มากกว่า (แต่ portable
> น้อยกว่า!)

ถัดมาคือ `awk` ซึ่งเหมือนกับ `sed` มีภาษาโปรแกรมของตัวเอง ถ้า `sed` ถูกสร้างมาเพื่อแก้ไขไฟล์ `awk` ถูกสร้างมาเพื่อ parse ไฟล์ การใช้งาน `awk` ที่พบบ่อยที่สุดคือกับไฟล์ข้อมูลที่มี syntax เป็นระเบียบ (เช่น CSV files) ที่ต้องการดึงเฉพาะบางส่วนของทุก record (คือทุกบรรทัด):

```console
missing:~$ awk '{print $2}' file
```

พิมพ์ column ที่สองที่แยกด้วย whitespace ของทุกบรรทัดใน `file` ถ้าเพิ่ม `-F,` จะพิมพ์ column ที่สองที่แยกด้วย comma ของทุกบรรทัด `awk` ทำได้มากกว่านี้อีก --- filter rows, compute aggregates และอื่นๆ --- ดูแบบฝึกหัดสำหรับตัวอย่าง

เมื่อนำเครื่องมือเหล่านี้มาใช้ร่วมกัน สามารถทำสิ่งซับซ้อนได้ เช่น:

```console
missing:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

คำสั่งนี้ดึง SSH logs จาก remote server (จะพูดเพิ่มเรื่อง `ssh` ในบรรยายถัดไป) ค้นหา disconnect messages, extract username จากแต่ละ message และพิมพ์ 10 usernames ที่พบบ่อยที่สุดโดยคั่นด้วย comma ทั้งหมดนี้ในคำสั่งเดียว! เราจะปล่อยให้การวิเคราะห์แต่ละขั้นตอนเป็นแบบฝึกหัด

## The shell language (bash)

ตัวอย่างก่อนหน้าแนะนำ concept ใหม่: pipes (`|`) ซึ่งช่วยให้ต่อ output ของโปรแกรมหนึ่งเข้ากับ input ของอีกโปรแกรมหนึ่ง วิธีนี้ทำงานได้เพราะโปรแกรม command-line ส่วนใหญ่จะทำงานกับ "standard input" (ที่ปกติรับ keystrokes) ถ้าไม่ได้ระบุ argument `file` ตัว `|` จะนำ "standard output" (ที่ปกติพิมพ์ออกมาที่ terminal) ของโปรแกรมก่อน `|` มาเป็น standard input ของโปรแกรมหลัง `|` สิ่งนี้ช่วยให้ _compose_ โปรแกรม shell เข้าด้วยกันได้ และเป็นส่วนหนึ่งของสิ่งที่ทำให้ shell เป็นสภาพแวดล้อมที่ productive มาก!

ที่จริงแล้ว shell ส่วนใหญ่ implement ภาษาโปรแกรมเต็มรูปแบบ (อย่าง bash) เหมือนกับ Python หรือ Ruby มันมี variables, conditionals, loops และ functions เมื่อรันคำสั่งใน shell จริงๆ แล้วกำลังเขียนโค้ดเล็กๆ น้อยๆ ที่ shell ตีความ เราจะไม่สอน bash ทั้งหมดวันนี้ แต่มีบางส่วนที่จะพบว่ามีประโยชน์เป็นพิเศษ:

ก่อนอื่นคือ redirects: `>file` ช่วยนำ standard output ของโปรแกรมไปเขียนลง `file` แทนที่จะแสดงที่ terminal ทำให้วิเคราะห์ภายหลังได้ง่ายขึ้น `>>file` จะ append ต่อท้าย `file` แทนที่จะเขียนทับ นอกจากนี้ยังมี `<file` ที่บอก shell ให้อ่านจาก `file` แทนที่จะจาก keyboard เป็น standard input ของโปรแกรม

> ช่วงนี้เป็นจังหวะดีที่จะพูดถึงโปรแกรม `tee` ตัว `tee` จะพิมพ์
> standard input ไปยัง standard output (เหมือน `cat`!) แต่ยัง _เขียน_
> ลงไฟล์ด้วย ดังนั้น `verbose cmd | tee verbose.log | grep CRITICAL`
> จะเก็บ verbose log ฉบับเต็มลงไฟล์ในขณะที่ terminal ยังคงสะอาด!

ถัดมาคือ conditionals: `if command1; then command2; command3; fi` จะรัน `command1` และถ้าไม่เกิด error จะรัน `command2` และ `command3` นอกจากนี้ยังมี `else` branch ได้ด้วย คำสั่งที่ใช้เป็น `command1` บ่อยที่สุดคือ `test` ซึ่งมักเขียนย่อเป็น `[` ช่วยให้ตรวจสอบเงื่อนไขได้ เช่น "ไฟล์มีอยู่หรือไม่" (`test -f file` / `[ -f file ]`) หรือ "string เท่ากันหรือไม่" (`[ "$var" = "string" ]`) ใน bash ยังมี `[[ ]]` ซึ่งเป็น built-in version ของ `test` ที่ "ปลอดภัยกว่า" มีพฤติกรรมแปลกๆ เรื่อง quoting น้อยกว่า

Bash ยังมี loops สองรูปแบบคือ `while` และ `for` โดย `while command1; do command2; command3; done` ทำงานเหมือน `if` ที่เทียบเคียงกัน แต่จะทำซ้ำไปเรื่อยๆ ตราบใดที่ `command1` ไม่เกิด error ส่วน `for varname in a b c d; do command; done` จะรัน `command` สี่ครั้ง โดยแต่ละครั้ง `$varname` จะถูก set เป็นค่า `a`, `b`, `c` และ `d` ตามลำดับ แทนที่จะ list รายการออกมาตรงๆ มักใช้ "command substitution" เช่น:

```bash
for i in $(seq 1 10); do
```

คำสั่งนี้รัน `seq 1 10` (ซึ่งพิมพ์ตัวเลข 1 ถึง 10) จากนั้นแทนที่ `$()` ทั้งหมดด้วย output ของคำสั่งนั้น ได้ for loop ที่วนซ้ำ 10 รอบ ในโค้ดเก่าๆ อาจเห็นการใช้ backticks (เช่น ``for i in `seq 1 10`; do``) แทน `$()` แต่ควรใช้รูปแบบ `$()` เพราะสามารถ nest ได้

แม้ _จะ_ เขียน shell scripts ยาวๆ ใน prompt ได้โดยตรง แต่ปกติจะเขียนลงไฟล์ `.sh` แทน ตัวอย่างเช่น นี่คือ script ที่จะรันโปรแกรมวนซ้ำจนกว่าจะล้มเหลว โดยพิมพ์เฉพาะ output ของรอบที่ล้มเหลว พร้อมทั้ง stress CPU เป็น background (มีประโยชน์สำหรับ reproduce flaky tests เป็นต้น):

```bash
#!/bin/bash
set -euo pipefail

# เริ่ม CPU stress ใน background
stress --cpu 8 & 
STRESS_PID=$!

# ตั้งค่า log file
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# รัน tests จนกว่าจะล้มเหลว
RUN=1
while cargo test my_test > "$LOGFILE" 2>&1; do
    echo "Run $RUN passed"
    ((RUN++))
done

# ทำความสะอาดและรายงานผล
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE"
```

Script นี้มีสิ่งใหม่หลายอย่างที่แนะนำให้ใช้เวลาศึกษา เพราะมีประโยชน์มากในการเขียนคำสั่ง shell เช่น background jobs (`&`) สำหรับรันโปรแกรมพร้อมกัน, [shell
redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html) ที่ซับซ้อนขึ้น และ [arithmetic
expansion](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html)

แต่ควรใช้เวลาสักครู่กับสองบรรทัดแรกของโปรแกรม บรรทัดแรกคือ "shebang" -- จะเห็นสิ่งนี้ที่หัวไฟล์อื่นนอกจาก shell scripts ด้วย เมื่อไฟล์ที่ขึ้นต้นด้วย `#!/path` ถูกรัน shell จะเริ่มโปรแกรมที่ `/path` และส่งเนื้อหาของไฟล์เป็น input ในกรณีของ shell script หมายถึงการส่งเนื้อหาของ shell script ให้ `/bin/bash` แต่ยังเขียน Python scripts ที่มี shebang line เป็น `/usr/bin/python` ได้ด้วย!

บรรทัดที่สองเป็นวิธีทำให้ bash "เข้มงวดมากขึ้น" และลดจำนวน footguns เมื่อเขียน shell scripts `set` รับ arguments ได้มากมาย แต่โดยย่อ: `-e` ทำให้ถ้าคำสั่งใดล้มเหลว script จะออกทันที; `-u` ทำให้การใช้ undefined variables จะทำให้ script crash แทนที่จะใช้ empty string; และ `-o pipefail` ทำให้ถ้าโปรแกรมในลำดับ `|` ล้มเหลว shell script โดยรวมจะออกทันทีเช่นกัน

> Shell programming เป็นหัวข้อที่ลึกซึ้ง เหมือนภาษาโปรแกรมอื่นๆ
> แต่ต้องระวัง: bash มี gotchas จำนวนมากผิดปกติ ถึงขนาดที่มี
> [เว็บไซต์](https://tldp.org/LDP/abs/html/gotchas.html)
> [หลายแห่ง](https://mywiki.wooledge.org/BashPitfalls) ที่อุทิศให้กับการรวบรวม gotchas เหล่านั้น
> แนะนำอย่างยิ่งให้ใช้
> [shellcheck](https://www.shellcheck.net/) เมื่อเขียน shell scripts
> LLMs ก็เก่งในการเขียนและ debug shell scripts รวมถึง
> แปลงเป็นภาษาโปรแกรม "จริงๆ" (อย่าง Python) เมื่อ script ใหญ่เกินไป
> สำหรับ bash (100+ บรรทัด)

# Next steps

ถึงตรงนี้ก็พอรู้จักใช้ shell เพียงพอสำหรับงานพื้นฐานแล้ว ควรสามารถนำทางไปหาไฟล์ที่ต้องการและใช้ functionality พื้นฐานของโปรแกรมส่วนใหญ่ได้ ในบรรยายถัดไปจะพูดเกี่ยวกับการทำงานที่ซับซ้อนขึ้นและทำให้เป็นอัตโนมัติโดยใช้ shell และโปรแกรม command-line ที่มีประโยชน์อีกมากมาย

# Exercises

ทุกบทเรียนในคอร์สนี้มาพร้อมกับชุดแบบฝึกหัด บางข้อให้ทำงานเฉพาะเจาะจง บางข้อเป็นแบบปลายเปิด เช่น "ลองใช้โปรแกรม X และ Y" แนะนำอย่างยิ่งให้ลองทำ

เราไม่ได้เขียนเฉลยสำหรับแบบฝึกหัด ถ้าติดตรงไหน โพสต์ใน `#missing-semester-forum` บน [Discord](https://ossu.dev/#community) หรือส่งอีเมลอธิบายสิ่งที่ลองทำมาแล้ว เราจะพยายามช่วย แบบฝึกหัดเหล่านี้ยังเหมาะเป็น prompt เริ่มต้นสำหรับสนทนากับ LLM เพื่อเจาะลึกหัวข้อแบบ interactive ได้ด้วย คุณค่าที่แท้จริงของแบบฝึกหัดเหล่านี้อยู่ที่การเดินทางค้นหาคำตอบ ไม่ใช่คำตอบเอง แนะนำให้ออกนอกเส้นทางบ้างและถามว่า "ทำไม" เมื่อทำแบบฝึกหัด แทนที่จะหาเส้นทางที่สั้นที่สุดไปสู่คำตอบ

1. สำหรับคอร์สนี้ ต้องใช้ Unix shell เช่น Bash หรือ ZSH ถ้าใช้ Linux หรือ macOS ไม่ต้องทำอะไรเพิ่ม ถ้าใช้ Windows ต้องมั่นใจว่าไม่ได้รัน cmd.exe หรือ PowerShell สามารถใช้ [Windows Subsystem for
   Linux](https://docs.microsoft.com/en-us/windows/wsl/) หรือ Linux virtual
   machine เพื่อใช้เครื่องมือ command-line แบบ Unix เพื่อตรวจสอบว่ากำลังรัน shell ที่ถูกต้อง ลองรันคำสั่ง `echo $SHELL` ถ้าแสดงอะไรประมาณ `/bin/bash` หรือ `/usr/bin/zsh` แสดงว่ากำลังรันโปรแกรมที่ถูกต้อง

2. flag `-l` ของ `ls` ทำอะไร? รัน `ls -l /` แล้วดู output อักขระ 10 ตัวแรกของแต่ละบรรทัดหมายความว่าอะไร? (คำใบ้: `man ls`)

3. ในคำสั่ง `find ~/Downloads -type f -name "*.zip" -mtime +30` ตัว `*.zip` คือ "glob" แล้ว glob คืออะไร? สร้าง test directory พร้อมไฟล์สักหน่อยแล้วทดลองกับ patterns เช่น `ls *.txt`, `ls file?.txt` และ `ls {a,b,c}.txt` ดู [Pattern
   Matching](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html)
   ใน Bash manual

4. `'single quotes'`, `"double quotes"` และ `$'ANSI quotes'` ต่างกันอย่างไร? เขียนคำสั่งที่ echo string ที่มี `$` ตัวอักษรจริง, `!` และ newline character ดู
   [Quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)

5. Shell มี standard streams สามตัว: stdin (0), stdout (1) และ stderr (2) รัน `ls /nonexistent /tmp` แล้ว redirect stdout ไปไฟล์หนึ่งและ stderr ไปอีกไฟล์หนึ่ง จะ redirect ทั้งสองไปไฟล์เดียวกันได้อย่างไร? ดู
   [Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)

6. `$?` เก็บ exit status ของคำสั่งล่าสุด (0 = สำเร็จ) `&&` รันคำสั่งถัดไปเฉพาะเมื่อคำสั่งก่อนหน้าสำเร็จ `||` รันเฉพาะเมื่อคำสั่งก่อนหน้าล้มเหลว เขียน one-liner ที่สร้าง `/tmp/mydir` เฉพาะเมื่อยังไม่มีอยู่ ดู [Exit
   Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)

7. ทำไม `cd` ถึงต้องเป็น built-in ของ shell แทนที่จะเป็นโปรแกรมแยกต่างหาก? (คำใบ้: คิดว่า child process สามารถและไม่สามารถส่งผลกระทบอะไรได้บ้างใน parent)

8. เขียน script ที่รับ filename เป็น argument (`$1`) และตรวจสอบว่าไฟล์มีอยู่หรือไม่ด้วย `test -f` หรือ `[ -f ... ]` ควรพิมพ์ข้อความที่แตกต่างกันขึ้นอยู่กับว่าไฟล์มีอยู่หรือไม่ ดู [Bash
   Conditional
   Expressions](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)

9. บันทึก script จากแบบฝึกหัดก่อนหน้าลงไฟล์ (เช่น `check.sh`)
   ลองรันด้วย `./check.sh somefile` เกิดอะไรขึ้น? จากนั้นรัน
   `chmod +x check.sh` แล้วลองอีกครั้ง ทำไมขั้นตอนนี้จึงจำเป็น? (คำใบ้:
   ดู `ls -l check.sh` ก่อนและหลัง `chmod`)

10. จะเกิดอะไรขึ้นถ้าเพิ่ม `-x` เข้าไปใน `set` flags ใน script? ลองกับ script ง่ายๆ แล้วดู output ดู [The Set
   Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)

11. เขียนคำสั่งที่ copy ไฟล์ไปเป็น backup โดยมีวันที่ของวันนี้ในชื่อไฟล์ (เช่น `notes.txt` → `notes_2026-01-12.txt`) (คำใบ้: `$(date
+%Y-%m-%d)`) ดู [Command
   Substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)

12. แก้ไข flaky test script จากบรรยายให้รับ test command เป็น argument แทนที่จะ hardcode `cargo test my_test` (คำใบ้: `$1`
   หรือ `$@`) ดู [Special
   Parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)

13. ใช้ pipes เพื่อหา 5 นามสกุลไฟล์ที่พบบ่อยที่สุดใน home directory (คำใบ้: รวม `find`, `grep` หรือ `sed` หรือ `awk`, `sort`,
   `uniq -c` และ `head`)

14. `xargs` แปลงบรรทัดจาก stdin เป็น command arguments ใช้ `find` และ `xargs` ร่วมกัน (ไม่ใช่ `find -exec`) เพื่อค้นหาไฟล์ `.sh` ทั้งหมดใน directory แล้วนับจำนวนบรรทัดด้วย `wc -l` โบนัส: ทำให้รองรับชื่อไฟล์ที่มีช่องว่าง (คำใบ้: `-print0` และ `-0`) ดู `man
xargs`

15. ใช้ `curl` เพื่อดึง HTML ของเว็บไซต์คอร์ส
   (`https://missing.csail.mit.edu/`) แล้ว pipe ให้ `grep` เพื่อนับว่ามีบรรยายกี่บท (คำใบ้: หา pattern ที่ปรากฏครั้งเดียวต่อบรรยาย ใช้ `curl -s` เพื่อปิด progress output)

16. [`jq`](https://jqlang.github.io/jq/) เป็นเครื่องมือที่ทรงพลังสำหรับประมวลผลข้อมูล JSON ดึงข้อมูลตัวอย่างที่
   `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json` ด้วย
   `curl` แล้วใช้ `jq` เพื่อดึงเฉพาะชื่อของคนที่มี version มากกว่า 6 (คำใบ้: pipe ให้ `jq .` ก่อนเพื่อดูโครงสร้าง จากนั้นลอง `jq '.[] | select(...) | .name'`)

17. `awk` สามารถ filter บรรทัดตามค่าของ column และจัดการ output ได้
   ตัวอย่างเช่น `awk '$3 ~ /pattern/ {$4=""; print}'` พิมพ์เฉพาะบรรทัดที่ column ที่สาม match กับ `pattern` โดยละ column ที่สี่ออก เขียนคำสั่ง `awk` ที่พิมพ์เฉพาะบรรทัดที่ column ที่สองมากกว่า 100 และสลับ column ที่หนึ่งกับที่สาม ทดสอบด้วย: `printf 'a 50 x\nb 150 y\nc 200 z\n'`

18. วิเคราะห์ SSH log pipeline จากบรรยาย: แต่ละขั้นตอนทำอะไร?
   จากนั้นสร้างสิ่งที่คล้ายกันเพื่อหาคำสั่ง shell ที่ใช้บ่อยที่สุดจาก
   `~/.bash_history` (หรือ `~/.zsh_history`)