---
layout: lecture
title: "Packaging and Shipping Code"
description: >
  เรียนรู้เกี่ยวกับการ packaging โปรเจกต์, environments, การทำ versioning, และการ deploy libraries, applications และ services
thumbnail: /static/assets/thumbnails/2026/lec6.png
date: 2026-01-20
ready: true
video:
  aspect: 56.25
  id: KBMiB-8P4Ns
---

การทำให้โค้ดทำงานได้ตามที่ต้องการนั้นยากอยู่แล้ว แต่การทำให้โค้ดเดียวกันรันได้บนเครื่องอื่นที่ไม่ใช่เครื่องของเรามักจะยากกว่าอีก

การ ship code หมายถึงการเอาโค้ดที่เราเขียนมาแปลงให้อยู่ในรูปแบบที่คนอื่นสามารถรันได้โดยไม่ต้องมี setup เหมือนเครื่องของเราทุกประการ
การ ship code มีหลายรูปแบบและขึ้นอยู่กับการเลือกภาษาโปรแกรม, system libraries, ระบบปฏิบัติการ และปัจจัยอื่นๆ อีกมากมาย
นอกจากนี้ยังขึ้นอยู่กับว่าเรากำลังสร้างอะไร: software library, command line tool, และ web service ล้วนมีข้อกำหนดและขั้นตอนการ deploy ที่แตกต่างกัน
อย่างไรก็ตาม มี pattern ร่วมกันระหว่าง scenarios ทั้งหมดเหล่านี้: เราต้องนิยามว่าสิ่งที่ส่งมอบคืออะไร --- หรือที่เรียกว่า _artifact_ --- และมันตั้งสมมติฐานอะไรเกี่ยวกับ environment รอบตัวมัน

ในบทนี้จะครอบคลุมเรื่อง:

- [Dependencies & Environments](#dependencies--environments)
- [Artifacts & Packaging](#artifacts--packaging)
- [Releases & Versioning](#releases--versioning)
- [Reproducibility](#reproducibility)
- [VMs & Containers](#vms--containers)
- [Configuration](#configuration)
- [Services & Orchestration](#services--orchestration)
- [Publishing](#publishing)

เราจะอธิบาย concepts เหล่านี้ผ่านตัวอย่างจาก Python ecosystem เนื่องจากตัวอย่างที่เป็นรูปธรรมช่วยให้เข้าใจได้ง่ายขึ้น แม้ว่าเครื่องมือจะแตกต่างกันสำหรับ ecosystem ของภาษาโปรแกรมอื่นๆ แต่ concepts จะเหมือนกันเป็นส่วนใหญ่

# Dependencies & Environments

ในการพัฒนาซอฟต์แวร์สมัยใหม่ layers of abstraction มีอยู่ทุกหนทุกแห่ง
โปรแกรมต่างๆ จะถ่ายโอน logic ไปให้ libraries หรือ services อื่นๆ โดยธรรมชาติ
อย่างไรก็ตาม สิ่งนี้ทำให้เกิดความสัมพันธ์แบบ _dependency_ ระหว่างโปรแกรมของเรากับ libraries ที่มันต้องการเพื่อทำงาน
ตัวอย่างเช่น ใน Python เมื่อเราต้องการดึงเนื้อหาของเว็บไซต์ เรามักจะทำแบบนี้:

```python
import requests

response = requests.get("https://missing.csail.mit.edu")
```

แต่ library `requests` ไม่ได้มาพร้อมกับ Python runtime ดังนั้นถ้าเรารันโค้ดนี้โดยไม่ได้ install `requests` ไว้ Python จะแจ้ง error:

```console
$ python fetch.py
Traceback (most recent call last):
  File "fetch.py", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

เพื่อทำให้ library นี้พร้อมใช้งาน เราต้องรัน `pip install requests` เพื่อ install มันก่อน
`pip` คือ command line tool ที่ Python มีให้สำหรับการ install packages
การรัน `pip install requests` จะทำขั้นตอนต่อไปนี้:

1. ค้นหา requests ใน Python Package Index ([PyPI](https://pypi.org/))
2. ค้นหา artifact ที่เหมาะกับ platform ที่เรากำลังใช้อยู่
3. Resolve dependencies --- library `requests` เองก็ขึ้นอยู่กับ packages อื่น ดังนั้น installer ต้องหา versions ที่เข้ากันได้ของ transitive dependencies ทั้งหมดและ install มันก่อน
4. Download artifacts จากนั้น unpack และคัดลอกไฟล์ไปยังตำแหน่งที่ถูกต้องใน filesystem ของเรา

```console
$ pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB)
Collecting charset-normalizer<4,>=2
  Downloading charset_normalizer-3.4.0-cp311-cp311-manylinux_x86_64.whl (142 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.10-py3-none-any.whl (70 kB)
Collecting urllib3<3,>=1.21.1
  Downloading urllib3-2.2.3-py3-none-any.whl (126 kB)
Collecting certifi>=2017.4.17
  Downloading certifi-2024.8.30-py3-none-any.whl (167 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2024.8.30 charset-normalizer-3.4.0 idna-3.10 requests-2.32.3 urllib3-2.2.3
```

ตรงนี้จะเห็นว่า `requests` มี dependencies ของตัวเอง เช่น `certifi` หรือ `charset-normalizer` และมันต้องถูก install ก่อนที่จะ install `requests` ได้
เมื่อ install แล้ว Python runtime ก็จะหา library นี้เจอเมื่อ import มัน

```console
$ python -c 'import requests; print(requests.__path__)'
['/usr/local/lib/python3.11/dist-packages/requests']

$ pip list | grep requests
requests        2.32.3
```

ภาษาโปรแกรมแต่ละภาษามีเครื่องมือ, conventions และแนวปฏิบัติที่แตกต่างกันสำหรับการ install และ publish libraries
ในบางภาษาอย่าง Rust, toolchain จะรวมเป็นหนึ่งเดียว --- `cargo` จัดการทั้งการ build, testing, dependency management และ publishing
ในภาษาอื่นอย่าง Python การรวมเป็นหนึ่งเดียวเกิดขึ้นในระดับ specification --- แทนที่จะมี tool เดียว จะมี standardized specifications ที่กำหนดว่า packaging ทำงานอย่างไร ทำให้มี tools หลายตัวแข่งกันสำหรับแต่ละงาน (`pip` vs [`uv`](https://docs.astral.sh/uv/), `setuptools` vs [`hatch`](https://hatch.pypa.io/) vs [`poetry`](https://python-poetry.org/))
และในบาง ecosystem อย่าง LaTeX, distributions อย่าง TeX Live หรือ MacTeX มาพร้อม packages ที่ pre-installed ไว้หลายพันตัว

การเพิ่ม dependencies ยังนำมาซึ่ง dependency conflicts อีกด้วย
Conflicts เกิดขึ้นเมื่อโปรแกรมต้องการ versions ที่เข้ากันไม่ได้ของ dependency ตัวเดียวกัน
ตัวอย่างเช่น ถ้า `tensorflow==2.3.0` ต้องการ `numpy>=1.16.0,<1.19.0` และ `pandas==1.2.0` ต้องการ `numpy>=1.16.5` ก็จะมี version ที่ตรงตามเงื่อนไข `numpy>=1.16.5,<1.19.0` ใช้ได้
แต่ถ้า package อื่นในโปรเจกต์ต้องการ `numpy>=1.19` ก็จะเกิด conflict ขึ้นเพราะไม่มี version ไหนที่ตอบสนองเงื่อนไขทั้งหมดได้

สถานการณ์แบบนี้ --- ที่หลาย packages ต้องการ versions ที่เข้ากันไม่ได้ของ shared dependencies --- มักเรียกกันว่า _dependency hell_
วิธีหนึ่งในการจัดการกับ conflicts คือการแยก dependencies ของแต่ละโปรแกรมออกเป็น _environment_ ของตัวเอง
ใน Python เราสร้าง virtual environment โดยรัน:

```console
$ which python
/usr/bin/python
$ pwd
/home/missingsemester
$ python -m venv venv
$ source venv/bin/activate
$ which python
/home/missingsemester/venv/bin/python
$ which pip
/home/missingsemester/venv/bin/pip
$ python -c 'import requests; print(requests.__path__)'
['/home/missingsemester/venv/lib/python3.11/site-packages/requests']

$ pip list
Package Version
------- -------
pip     24.0
```

ลองนึกภาพว่า environment คือ language runtime เวอร์ชันเต็มที่แยกออกมาต่างหากพร้อม packages ที่ install ไว้เป็นชุดของตัวเอง
Virtual environment หรือ venv นี้จะแยก dependencies ที่ install ไว้ออกจากการ install Python แบบ global
เป็นแนวปฏิบัติที่ดีที่จะมี virtual environment สำหรับแต่ละโปรเจกต์ โดยบรรจุ dependencies ที่ต้องการไว้ข้างใน

> แม้ว่าระบบปฏิบัติการสมัยใหม่หลายตัวจะมาพร้อมกับ programming language runtimes อย่าง Python ที่ install ไว้แล้ว แต่ไม่ควรไปแก้ไข installations เหล่านี้ เพราะ OS อาจพึ่งพามันสำหรับการทำงานของตัวเอง ควรใช้ environments ที่แยกออกมาแทน

ในบางภาษา installation protocol ไม่ได้ถูกกำหนดโดย tool ใด tool หนึ่ง แต่เป็น specification
ใน Python [PEP 517](https://peps.python.org/pep-0517/) กำหนด build system interface และ [PEP 621](https://peps.python.org/pep-0621/) ระบุวิธีเก็บ project metadata ใน `pyproject.toml`
สิ่งนี้ทำให้นักพัฒนาสามารถปรับปรุง `pip` และสร้างเครื่องมือที่ optimize ได้ดีกว่าอย่าง `uv` การ install `uv` แค่รัน `pip install uv`

การใช้ `uv` แทน `pip` มี interface เดียวกันแต่เร็วกว่ามาก:

```console
$ uv pip install requests
Resolved 5 packages in 12ms
Prepared 5 packages in 0.45ms
Installed 5 packages in 8ms
 + certifi==2024.8.30
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

> แนะนำอย่างยิ่งให้ใช้ `uv pip` แทน `pip` ทุกครั้งที่ทำได้ เพราะจะลดเวลาการ install ได้อย่างมาก

นอกจากการแยก dependencies แล้ว environments ยังช่วยให้เราใช้ programming language runtime หลาย versions ได้อีกด้วย

```console
$ uv venv --python 3.12 venv312
Using CPython 3.12.7
Creating virtual environment at: venv312

$ source venv312/bin/activate && python --version
Python 3.12.7

$ uv venv --python 3.11 venv311
Using CPython 3.11.10
Creating virtual environment at: venv311

$ source venv311/bin/activate && python --version
Python 3.11.10
```

สิ่งนี้มีประโยชน์เมื่อต้อง test โค้ดข้าม Python หลาย versions หรือเมื่อโปรเจกต์ต้องการ version เฉพาะ

> ในบางภาษาโปรแกรม แต่ละโปรเจกต์จะได้ environment ของตัวเองสำหรับ dependencies โดยอัตโนมัติแทนที่เราจะต้องสร้างเอง แต่หลักการเดียวกัน ภาษาส่วนใหญ่ในปัจจุบันยังมีกลไกสำหรับจัดการหลาย versions ของภาษาในระบบเดียว แล้วระบุว่าจะใช้ version ไหนสำหรับแต่ละโปรเจกต์

# Artifacts & Packaging

ในการพัฒนาซอฟต์แวร์ เราแยกความแตกต่างระหว่าง source code กับ artifacts นักพัฒนาเขียนและอ่าน source code ในขณะที่ artifacts คือ outputs ที่ถูก package ไว้แล้วพร้อมแจกจ่าย ผลิตขึ้นจาก source code --- พร้อมสำหรับการ install หรือ deploy
Artifact อาจง่ายแค่ไฟล์โค้ดที่เรารัน หรือซับซ้อนเท่ากับ Virtual Machine ทั้งเครื่องที่บรรจุส่วนประกอบทุกอย่างของ application ไว้
ลองดูตัวอย่างนี้ที่เรามีไฟล์ Python `greet.py` อยู่ใน directory ปัจจุบัน:

```console
$ cat greet.py
def greet(name):
    return f"Hello, {name}!"

$ python -c "from greet import greet; print(greet('World'))"
Hello, World!

$ cd /tmp
$ python -c "from greet import greet; print(greet('World'))"
ModuleNotFoundError: No module named 'greet'
```

การ import ล้มเหลวเมื่อเราย้ายไป directory อื่น เพราะ Python จะค้นหา modules เฉพาะในตำแหน่งที่กำหนดไว้เท่านั้น (directory ปัจจุบัน, packages ที่ install ไว้ และ paths ใน `PYTHONPATH`) การทำ packaging แก้ปัญหานี้โดยการ install โค้ดไปยังตำแหน่งที่รู้จัก

ใน Python การ package library เกี่ยวข้องกับการสร้าง artifact ที่ package installers อย่าง `pip` หรือ `uv` สามารถใช้ install ไฟล์ที่เกี่ยวข้องได้
Python artifacts เรียกว่า _wheels_ และบรรจุข้อมูลทั้งหมดที่จำเป็นสำหรับการ install package: ไฟล์โค้ด, metadata เกี่ยวกับ package (ชื่อ, version, dependencies) และคำสั่งว่าจะวางไฟล์ไว้ที่ไหนใน environment
การ build artifact ต้องเขียน project file (หรือที่มักเรียกว่า manifest) ระบุรายละเอียดของโปรเจกต์ dependencies ที่ต้องการ, version ของ package และข้อมูลอื่นๆ ใน Python เราใช้ `pyproject.toml` สำหรับจุดประสงค์นี้

> `pyproject.toml` เป็นวิธีที่ทันสมัยและแนะนำให้ใช้ แม้ว่า packaging methods ก่อนหน้าอย่าง `requirements.txt` หรือ `setup.py` จะยังรองรับอยู่ แต่ควรใช้ `pyproject.toml` ทุกครั้งที่ทำได้

นี่คือ `pyproject.toml` แบบ minimal สำหรับ library ที่มี command-line tool ด้วย:

```toml
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:main"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

Library `typer` เป็น Python package ยอดนิยมสำหรับสร้าง command-line interfaces โดยใช้ boilerplate น้อยที่สุด

และไฟล์ `greeting.py` ที่สอดคล้องกัน:

```python
import typer


def greet(name: str) -> str:
    return f"Hello, {name}!"


def main(name: str):
    print(greet(name))


if __name__ == "__main__":
    typer.run(main)
```

เมื่อมีไฟล์นี้แล้ว เราสามารถ build wheel ได้:

```console
$ uv build
Building source distribution...
Building wheel from source distribution...
Successfully built dist/greeting-0.1.0.tar.gz
Successfully built dist/greeting-0.1.0-py3-none-any.whl

$ ls dist/
greeting-0.1.0-py3-none-any.whl
greeting-0.1.0.tar.gz
```

ไฟล์ `.whl` คือ wheel (zip archive ที่มีโครงสร้างเฉพาะ) และ `.tar.gz` คือ source distribution สำหรับระบบที่ต้อง build จาก source

เราสามารถตรวจสอบเนื้อหาของ wheel เพื่อดูว่ามีอะไรถูก package ไว้บ้าง:

```console
$ unzip -l dist/greeting-0.1.0-py3-none-any.whl
Archive:  dist/greeting-0.1.0-py3-none-any.whl
  Length      Date    Time    Name
---------  ---------- -----   ----
      150  2024-01-15 10:30   greeting.py
      312  2024-01-15 10:30   greeting-0.1.0.dist-info/METADATA
       92  2024-01-15 10:30   greeting-0.1.0.dist-info/WHEEL
        9  2024-01-15 10:30   greeting-0.1.0.dist-info/top_level.txt
      435  2024-01-15 10:30   greeting-0.1.0.dist-info/RECORD
---------                     -------
      998                     5 files
```

ถ้าเราส่ง wheel นี้ให้คนอื่น พวกเขาก็สามารถ install ได้โดยรัน:

```console
$ uv pip install ./greeting-0.1.0-py3-none-any.whl
$ greet Alice
Hello, Alice!
```

คำสั่งนี้จะ install library ที่เรา build ไว้ก่อนหน้านี้เข้าไปใน environment ของพวกเขา รวมถึง `greet` cli tool ด้วย

วิธีนี้มีข้อจำกัด โดยเฉพาะถ้า library ของเราขึ้นอยู่กับ platform-specific libraries เช่น CUDA สำหรับ GPU acceleration แล้ว artifact ของเราจะทำงานได้เฉพาะบนระบบที่ install libraries เฉพาะเหล่านั้นไว้เท่านั้น และเราอาจต้อง build wheels แยกกันสำหรับ platforms (Linux, macOS, Windows) และ architectures (x86, ARM) ที่แตกต่างกัน


เมื่อ install ซอฟต์แวร์ มีความแตกต่างที่สำคัญระหว่างการ install จาก source กับการ install prebuilt binary การ install จาก source หมายถึงการ download โค้ดต้นฉบับแล้ว compile มันบนเครื่องของเรา --- ซึ่งต้องมี compiler และ build tools ติดตั้งไว้ และอาจใช้เวลานานสำหรับโปรเจกต์ขนาดใหญ่

การ install prebuilt binary หมายถึงการ download artifact ที่ถูก compile ไว้แล้วโดยคนอื่น --- เร็วกว่าและง่ายกว่า แต่ binary ต้องตรงกับ platform และ architecture ของเรา
ตัวอย่างเช่น [หน้า releases ของ ripgrep](https://github.com/BurntSushi/ripgrep/releases) แสดง prebuilt binaries สำหรับ Linux (x86_64, ARM), macOS (Intel, Apple Silicon) และ Windows


# Releases & Versioning

โค้ดถูก build อย่างต่อเนื่อง แต่ถูก release เป็นจังหวะๆ
ในการพัฒนาซอฟต์แวร์มีความแตกต่างชัดเจนระหว่าง development environment กับ production environment
โค้ดต้องได้รับการพิสูจน์ว่าทำงานได้ใน dev environment ก่อนที่จะถูก _ship_ ไปยัง prod
กระบวนการ release ประกอบด้วยหลายขั้นตอน ได้แก่ testing, dependency management, versioning, configuration, deployment และ publishing


Software libraries ไม่ได้หยุดนิ่ง แต่วิวัฒนาการไปตามเวลาด้วยการได้รับ fixes และ features ใหม่
เราติดตามวิวัฒนาการนี้ด้วย version identifiers ที่สอดคล้องกับสถานะของ library ณ ช่วงเวลาหนึ่ง
การเปลี่ยนแปลงพฤติกรรมของ library มีตั้งแต่ patches ที่แก้ไข functionality ที่ไม่วิกฤต, features ใหม่ที่ขยาย functionality, ไปจนถึงการเปลี่ยนแปลงที่ทำลาย backwards compatibility
Changelogs บันทึกว่า version หนึ่งมีการเปลี่ยนแปลงอะไรบ้าง --- เป็นเอกสารที่นักพัฒนาซอฟต์แวร์ใช้สื่อสารการเปลี่ยนแปลงที่เกี่ยวข้องกับ release ใหม่

อย่างไรก็ตาม การติดตามการเปลี่ยนแปลงที่เกิดขึ้นอย่างต่อเนื่องในแต่ละ dependency ทั้งหมดนั้นไม่เป็นเรื่องจริง ยิ่งเมื่อคำนึงถึง transitive dependencies --- คือ dependencies ของ dependencies ของเรา

> สามารถดู dependency tree ทั้งหมดของโปรเจกต์ได้ด้วย `uv tree` ซึ่งจะแสดง packages ทั้งหมดและ transitive dependencies ในรูปแบบ tree

เพื่อทำให้ปัญหานี้ง่ายขึ้น จึงมี conventions ว่าจะ version ซอฟต์แวร์อย่างไร และหนึ่งในที่แพร่หลายที่สุดคือ [Semantic Versioning](https://semver.org/) หรือ SemVer
ภายใต้ Semantic Versioning version จะมี identifier ในรูปแบบ MAJOR.MINOR.PATCH โดยแต่ละค่าเป็นจำนวนเต็ม โดยสรุปแล้ว การอัปเกรด:

- PATCH (เช่น 1.2.3 → 1.2.4) ควรมีแค่ bug fixes และเข้ากันได้กับเวอร์ชันก่อนหน้าอย่างสมบูรณ์
- MINOR (เช่น 1.2.3 → 1.3.0) เพิ่ม functionality ใหม่ในแบบที่เข้ากันได้กับเวอร์ชันก่อนหน้า
- MAJOR (เช่น 1.2.3 → 2.0.0) บ่งชี้ว่ามี breaking changes ที่อาจต้องแก้ไขโค้ด

> นี่เป็นการสรุปแบบย่อ แนะนำให้อ่าน SemVer specification ฉบับเต็มเพื่อทำความเข้าใจว่าทำไมการไปจาก 0.1.3 เป็น 0.2.0 อาจทำให้เกิด breaking changes หรือ 1.0.0-rc.1 หมายถึงอะไร
Python packaging รองรับ semantic versioning โดยตรง ดังนั้นเมื่อเราระบุ versions ของ dependencies เราสามารถใช้ specifiers ต่างๆ ได้:

ใน `pyproject.toml` เรามีวิธีต่างๆ ในการจำกัดช่วงของ versions ที่เข้ากันได้ของ dependencies:

```toml
[project]
dependencies = [
    "requests==2.32.3",  # Exact version - only this specific version
    "click>=8.0",        # Minimum version - 8.0 or newer
    "numpy>=1.24,<2.0",  # Range - at least 1.24 but less than 2.0
    "pandas~=2.1.0",     # Compatible release - >=2.1.0 and <2.2.0
]
```

Version specifiers มีอยู่ในหลาย package managers (npm, cargo ฯลฯ) โดยมี semantics ที่แตกต่างกันเล็กน้อย Operator `~=` เป็น "compatible release" operator ของ Python --- `~=2.1.0` หมายถึง "version ใดก็ได้ที่เข้ากันได้กับ 2.1.0" ซึ่งแปลว่า `>=2.1.0` และ `<2.2.0` สิ่งนี้เทียบได้คร่าวๆ กับ caret (`^`) operator ใน npm และ cargo ซึ่งใช้แนวคิด compatibility ของ SemVer

ซอฟต์แวร์ไม่ได้ใช้ semantic versioning ทั้งหมด ทางเลือกที่พบบ่อยคือ Calendar Versioning (CalVer) ที่ versions ตั้งตามวันที่ release แทนที่จะเป็นความหมายเชิง semantic ตัวอย่างเช่น Ubuntu ใช้ versions อย่าง `24.04` (เมษายน 2024) และ `24.10` (ตุลาคม 2024) CalVer ทำให้เห็นได้ง่ายว่า release เก่าแค่ไหน แม้จะไม่ได้สื่ออะไรเกี่ยวกับ compatibility สุดท้ายแล้ว semantic versioning ก็ไม่ได้สมบูรณ์แบบ และบางครั้ง maintainers ก็แนะนำ breaking changes ใน minor หรือ patch releases โดยไม่ได้ตั้งใจ


# Reproducibility

ในการพัฒนาซอฟต์แวร์สมัยใหม่ โค้ดที่เราเขียนอยู่บน layers of abstraction จำนวนมาก
ซึ่งรวมถึงสิ่งต่างๆ เช่น programming language runtime, third party libraries, ระบบปฏิบัติการ หรือแม้แต่ hardware เอง
ความแตกต่างใดๆ ใน layers เหล่านี้อาจเปลี่ยนพฤติกรรมของโค้ดหรือแม้แต่ทำให้มันไม่ทำงานตามที่ต้องการ
ยิ่งกว่านั้น ความแตกต่างใน hardware ที่อยู่เบื้องล่างก็ส่งผลต่อความสามารถในการ ship ซอฟต์แวร์ของเราด้วย

การ pin library หมายถึงการระบุ version ที่แน่นอนแทนที่จะเป็นช่วง เช่น `requests==2.32.3` แทนที่จะเป็น `requests>=2.0`

ส่วนหนึ่งของหน้าที่ package manager คือการพิจารณา constraints ทั้งหมดที่กำหนดโดย dependencies --- และ transitive dependencies --- แล้วสร้างรายการ versions ที่ใช้ได้ซึ่งตอบสนอง constraints ทั้งหมด
รายการ versions เฉพาะนั้นสามารถบันทึกลงไฟล์เพื่อจุดประสงค์ด้าน reproducibility ไฟล์เหล่านี้เรียกว่า _lock files_

```console
$ uv lock
Resolved 12 packages in 45ms

$ cat uv.lock | head -20
version = 1
requires-python = ">=3.11"

[[package]]
name = "certifi"
version = "2024.8.30"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/...", hash = "sha256:..." }
wheels = [
    { url = "https://files.pythonhosted.org/...", hash = "sha256:..." },
]
...
```

ข้อแตกต่างที่สำคัญเมื่อจัดการกับ dependency versioning และ reproducibility คือความแตกต่างระหว่าง libraries กับ applications/services
Library ถูกสร้างมาให้ import และใช้โดยโค้ดอื่นซึ่งอาจมี dependencies ของตัวเอง ดังนั้นการระบุ version constraints ที่เข้มงวดเกินไปอาจทำให้เกิด conflicts กับ dependencies อื่นๆ ของผู้ใช้
ในทางตรงกันข้าม applications หรือ services เป็นผู้บริโภคขั้นสุดท้ายของซอฟต์แวร์ และมักจะเปิดเผย functionality ผ่าน user interface หรือ API ไม่ใช่ผ่าน programming interface
สำหรับ libraries เป็นแนวปฏิบัติที่ดีที่จะระบุ version ranges เพื่อให้เข้ากันได้สูงสุดกับ package ecosystem ที่กว้างขึ้น สำหรับ applications การ pin exact versions ช่วยให้เกิด reproducibility --- ทุกคนที่รัน application ใช้ dependencies เดียวกันทุกประการ


สำหรับโปรเจกต์ที่ต้องการ reproducibility สูงสุด เครื่องมืออย่าง [Nix](https://nixos.org/) และ [Bazel](https://bazel.build/) มี _hermetic_ builds --- ที่ทุก input รวมถึง compilers, system libraries และแม้แต่ build environment เองก็ถูก pin และ content-addressed สิ่งนี้รับประกันว่า outputs จะเหมือนกันทุก bit ไม่ว่าจะ build เมื่อไหร่หรือที่ไหนก็ตาม

> ยังสามารถใช้ NixOS จัดการการ install คอมพิวเตอร์ทั้งเครื่องได้ เพื่อให้สามารถสร้างสำเนาของ setup คอมพิวเตอร์ได้อย่างง่ายดาย และจัดการ configuration ทั้งหมดผ่าน configuration files ที่ถูก version-controlled

ความตึงเครียดที่ไม่มีวันจบในการพัฒนาซอฟต์แวร์คือ software versions ใหม่อาจทำให้เกิดปัญหาทั้งโดยตั้งใจและไม่ตั้งใจ ในขณะเดียวกัน software versions เก่าก็ค่อยๆ มีช่องโหว่ด้านความปลอดภัยเพิ่มขึ้นเรื่อยๆ
เราสามารถจัดการกับสิ่งนี้ได้โดยใช้ continuous integration pipelines (จะเรียนรู้เพิ่มเติมในบท [Code Quality and CI](/2026/code-quality/)) ที่ test application ของเรากับ software versions ใหม่ และมี automation สำหรับตรวจจับเมื่อ dependencies มี versions ใหม่ออกมา เช่น [Dependabot](https://github.com/dependabot)

แม้จะมี CI testing อยู่แล้ว ปัญหาก็ยังเกิดขึ้นได้เมื่ออัปเกรด software versions บ่อยครั้งเนื่องจากความไม่ตรงกันที่หลีกเลี่ยงไม่ได้ระหว่าง dev กับ prod environments
ในสถานการณ์เหล่านั้น แนวทางที่ดีที่สุดคือการมีแผน _rollback_ ที่ version ที่อัปเกรดจะถูก revert และ version ที่ทำงานได้ดีจะถูก deploy กลับแทน

# VMs & Containers

เมื่อเริ่มพึ่งพา dependencies ที่ซับซ้อนมากขึ้น มีแนวโน้มว่า dependencies ของโค้ดจะขยายเกินขอบเขตของสิ่งที่ package manager จัดการได้
เหตุผลที่พบบ่อยคือต้อง interface กับ system libraries หรือ hardware drivers เฉพาะ
ตัวอย่างเช่น ในงาน scientific computing และ AI โปรแกรมมักต้องการ libraries และ drivers เฉพาะทางเพื่อใช้งาน GPU hardware
System-level dependencies หลายตัว (GPU drivers, compiler versions เฉพาะ, shared libraries อย่าง OpenSSL) ยังต้องการการ install แบบ system-wide

แต่เดิมปัญหา dependency ที่กว้างขวางนี้แก้ด้วย Virtual Machines (VMs)
VMs จำลองคอมพิวเตอร์ทั้งเครื่องและให้ environment ที่แยกออกมาอย่างสมบูรณ์พร้อมระบบปฏิบัติการเฉพาะของตัวเอง
แนวทางที่ทันสมัยกว่าคือ containers ซึ่ง package application พร้อมกับ dependencies, libraries และ filesystem แต่ใช้ kernel ของ host ร่วมกันแทนที่จะจำลองคอมพิวเตอร์ทั้งเครื่อง
Containers มีน้ำหนักเบากว่า VMs เพราะใช้ kernel ร่วมกัน ทำให้เริ่มต้นได้เร็วกว่าและรันได้อย่างมีประสิทธิภาพมากกว่า

Container platform ที่ได้รับความนิยมมากที่สุดคือ [Docker](https://www.docker.com/) Docker นำเสนอวิธีมาตรฐานในการ build, distribute และ run containers เบื้องหลังนั้น Docker ใช้ containerd เป็น container runtime --- ซึ่งเป็นมาตรฐานอุตสาหกรรมที่เครื่องมืออื่นอย่าง Kubernetes ก็ใช้เช่นกัน

การรัน container ทำได้ตรงไปตรงมา ตัวอย่างเช่น การรัน Python interpreter ภายใน container เราใช้ `docker run` (flags `-it` ทำให้ container เป็น interactive พร้อม terminal เมื่อ exit ออกมา container ก็จะหยุด)

```console
$ docker run -it python:3.12 python
Python 3.12.7 (main, Nov  5 2024, 02:53:25) [GCC 12.2.0] on linux
>>> print("Hello from inside a container!")
Hello from inside a container!
```

ในทางปฏิบัติ โปรแกรมอาจขึ้นอยู่กับ filesystem ทั้งหมด
เพื่อจัดการกับเรื่องนี้ เราสามารถใช้ container images ที่ ship filesystem ทั้งหมดของ application มาเป็น artifact
Container images ถูกสร้างขึ้นด้วยโปรแกรม ด้วย Docker เราระบุ dependencies, system libraries และ configuration ของ image อย่างแม่นยำโดยใช้ Dockerfile syntax:

```dockerfile
FROM python:3.12
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get install -y libpq-dev
RUN pip install numpy
RUN pip install pandas
COPY . /app
WORKDIR /app
RUN pip install .
```

ข้อแตกต่างที่สำคัญ: Docker **image** คือ artifact ที่ถูก package ไว้ (เหมือน template) ในขณะที่ **container** คือ running instance ของ image นั้น สามารถรันหลาย containers จาก image เดียวกันได้ Images ถูก build เป็น layers โดยแต่ละ instruction (`FROM`, `RUN`, `COPY` ฯลฯ) ใน Dockerfile จะสร้าง layer ใหม่ Docker จะ cache layers เหล่านี้ ดังนั้นถ้าเราเปลี่ยนบรรทัดหนึ่งใน Dockerfile เฉพาะ layer นั้นและ layers ถัดไปเท่านั้นที่ต้อง rebuild

Dockerfile ก่อนหน้านี้มีปัญหาหลายอย่าง: ใช้ Python image แบบเต็มแทนที่จะเป็น slim variant, รัน `RUN` commands แยกกันทำให้เกิด layers ที่ไม่จำเป็น, versions ไม่ได้ถูก pin, และไม่ได้ล้าง package manager caches ทำให้ ship ไฟล์ที่ไม่จำเป็นไปด้วย ข้อผิดพลาดอื่นๆ ที่พบบ่อย ได้แก่ การรัน containers เป็น root อย่างไม่ปลอดภัย และการฝัง secrets ไว้ใน layers โดยไม่ตั้งใจ

นี่คือเวอร์ชันที่ปรับปรุงแล้ว

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*
COPY pyproject.toml uv.lock ./
RUN uv pip install --system -r uv.lock
COPY . /app
```

ในตัวอย่างก่อนหน้านี้จะเห็นว่าแทนที่จะ install `uv` จาก source เรากำลัง copy prebuilt binary จาก image `ghcr.io/astral-sh/uv:latest` สิ่งนี้เรียกว่า _builder_ pattern ด้วย pattern นี้เราไม่ต้อง ship เครื่องมือทั้งหมดที่ต้องใช้ compile โค้ด แค่ final binary ที่ต้องใช้รัน application (`uv` ในกรณีนี้)

Docker มีข้อจำกัดที่สำคัญที่ควรรู้ไว้ ประการแรก container images มักเป็น platform-specific --- image ที่ build สำหรับ `linux/amd64` จะไม่สามารถรันบน `linux/arm64` (Mac ที่ใช้ Apple Silicon) ได้โดยตรง ต้องใช้ emulation ซึ่งช้า ประการที่สอง Docker containers ต้องการ Linux kernel ดังนั้นบน macOS และ Windows Docker จริงๆ แล้วรัน Linux VM แบบ lightweight อยู่เบื้องหลัง ซึ่งเพิ่ม overhead ประการที่สาม การแยกตัวของ Docker อ่อนกว่า VMs --- containers ใช้ host kernel ร่วมกัน ซึ่งเป็นข้อกังวลด้านความปลอดภัยใน multi-tenant environments

> ในปัจจุบัน มีโปรเจกต์มากขึ้นที่ใช้ nix ในการจัดการแม้แต่ libraries และ applications ระดับ "system-wide" ต่อโปรเจกต์ผ่าน [nix flakes](https://serokell.io/blog/practical-nix-flakes)

# Configuration

ซอฟต์แวร์โดยธรรมชาติแล้วสามารถ configure ได้ ในบท [command line environment](/2026/command-line-environment/) เราเห็นโปรแกรมรับ options ผ่าน flags, environment variables หรือแม้แต่ configuration files หรือ dotfiles สิ่งนี้เป็นจริงแม้กับ applications ที่ซับซ้อนกว่า และมี patterns ที่ใช้กันมาแล้วสำหรับการจัดการ configuration ในวงกว้าง
Software configuration ไม่ควรฝังอยู่ในโค้ด แต่ควรถูกให้ตอน runtime
รูปแบบที่พบบ่อยสองอย่างคือ environment variables และ config files

นี่คือตัวอย่างของ application ที่ถูก configure ผ่าน environment variables:

```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]  # Required - will raise if not set
```

Application ยังสามารถถูก configure ผ่าน configuration file ได้ (เช่น โปรแกรม Python ที่ load config ผ่าน `yaml.load`) `config.yaml`:

```yaml
database:
  url: "postgresql://localhost/myapp"
  pool_size: 5
server:
  host: "0.0.0.0"
  port: 8080
  debug: false
```

หลักคิดง่ายๆ สำหรับ configuration คือ codebase เดียวกันควร deploy ไปยัง environments ที่แตกต่างกัน (development, staging, production) ได้โดยเปลี่ยนแค่ configuration ไม่ใช่โค้ด

ในบรรดา configuration options มากมาย มักมี sensitive data เช่น API keys อยู่ด้วย
Secrets ต้องจัดการอย่างระมัดระวังเพื่อหลีกเลี่ยงการเปิดเผยโดยบังเอิญ และต้องไม่รวมอยู่ใน version control


# Services & Orchestration

Applications สมัยใหม่ไม่ค่อยอยู่โดดเดี่ยว Web application ทั่วไปอาจต้องการ database สำหรับ persistent storage, cache สำหรับ performance, message queue สำหรับ background tasks, และ supporting services อื่นๆ อีกหลายตัว แทนที่จะรวมทุกอย่างไว้ใน monolithic application เดียว สถาปัตยกรรมสมัยใหม่มักแยก functionality ออกเป็น services แยกกันที่สามารถพัฒนา, deploy และ scale ได้อย่างอิสระ

ตัวอย่างเช่น ถ้าเราพิจารณาว่า application อาจได้ประโยชน์จากการใช้ cache แทนที่จะสร้างเอง เราสามารถใช้ solutions ที่ผ่านการทดสอบมาแล้วอย่าง [Redis](https://redis.io/) หรือ [Memcached](https://memcached.org/)
เราอาจฝัง Redis ไว้ใน application dependencies โดย build เป็นส่วนหนึ่งของ container แต่นั่นหมายความว่าต้องประสาน dependencies ทั้งหมดระหว่าง Redis กับ application ของเรา ซึ่งอาจยากหรือแม้แต่เป็นไปไม่ได้
แทนที่จะทำแบบนั้น เราสามารถ deploy แต่ละ application แยกกันใน container ของตัวเอง
สิ่งนี้มักเรียกว่า microservice architecture ที่แต่ละ component รันเป็น service อิสระที่สื่อสารกันผ่าน network โดยปกติผ่าน HTTP APIs

[Docker Compose](https://docs.docker.com/compose/) คือเครื่องมือสำหรับกำหนดและรัน multi-container applications แทนที่จะจัดการ containers ทีละตัว เราประกาศ services ทั้งหมดใน YAML file เดียวและ orchestrate มันเข้าด้วยกัน ตอนนี้ application เต็มรูปแบบของเราครอบคลุมมากกว่าหนึ่ง container:

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - REDIS_URL=redis://cache:6379
    depends_on:
      - cache

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

ด้วย `docker compose up` services ทั้งสองจะเริ่มพร้อมกัน และ web application สามารถเชื่อมต่อกับ Redis ได้โดยใช้ hostname `cache` (DNS ภายในของ Docker จะ resolve ชื่อ service โดยอัตโนมัติ)
Docker Compose ให้เราประกาศว่าต้องการ deploy services อย่างไร และจัดการ orchestration ของการเริ่ม services พร้อมกัน, ตั้งค่า networking ระหว่างกัน, และจัดการ shared volumes สำหรับ data persistence

สำหรับ production deployments มักต้องการให้ docker compose services เริ่มอัตโนมัติเมื่อ boot และ restart เมื่อ fail แนวทางที่พบบ่อยคือใช้ systemd จัดการ docker compose deployment:

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target
```

systemd unit file นี้ทำให้ application เริ่มเมื่อระบบ boot (หลังจาก Docker พร้อม) และให้ controls มาตรฐานอย่าง `systemctl start myapp`, `systemctl stop myapp` และ `systemctl status myapp`

เมื่อความต้องการในการ deploy ซับซ้อนขึ้น --- ต้องการ scalability ข้ามหลายเครื่อง, fault tolerance เมื่อ services crash, และ high availability guarantees --- องค์กรก็หันไปใช้ container orchestration platforms ที่ซับซ้อนอย่าง Kubernetes (k8s) ซึ่งสามารถจัดการ containers หลายพันตัวข้ามกลุ่มเครื่อง อย่างไรก็ตาม Kubernetes มี learning curve ที่สูงและ operational overhead ที่มาก ดังนั้นมักจะเกินความจำเป็นสำหรับโปรเจกต์ขนาดเล็ก

การตั้งค่า multi-container นี้เป็นไปได้ส่วนหนึ่งเพราะ services สมัยใหม่สื่อสารกันผ่าน standardized APIs ซึ่งก็คือ HTTP REST APIs ตัวอย่างเช่น เมื่อโปรแกรมโต้ตอบกับ LLM provider อย่าง OpenAI หรือ Anthropic เบื้องหลังมันกำลังส่ง HTTP request ไปยัง servers ของพวกเขาและ parse response ที่ได้กลับมา:

```console
$ curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2023-06-01" \
    -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 256,
         "messages": [{"role": "user", "content": "Explain containers vs VMs in one sentence."}]}'
```

# Publishing

เมื่อโค้ดของเราพิสูจน์แล้วว่าทำงานได้ เราอาจสนใจที่จะ distribute มันให้คนอื่น download และ install
การ distribution มีหลายรูปแบบและผูกติดอยู่กับภาษาโปรแกรมและ environments ที่เราใช้งาน

รูปแบบที่ง่ายที่สุดของการ distribution คือการ upload artifacts ให้คนอื่น download และ install ในเครื่อง
สิ่งนี้ยังเป็นเรื่องปกติอยู่ และสามารถพบได้ในที่อย่าง [Ubuntu's package archive](http://archive.ubuntu.com/ubuntu/pool/main/) ซึ่งเป็น HTTP directory listing ของไฟล์ `.deb`

ในปัจจุบัน GitHub ได้กลายเป็น platform มาตรฐานสำหรับ publish source code และ artifacts
แม้ว่า source code มักจะเปิดให้เข้าถึงได้ GitHub Releases ช่วยให้ maintainers แนบ prebuilt binaries และ artifacts อื่นๆ กับ tagged versions ได้


บาง package managers รองรับการ install โดยตรงจาก GitHub ไม่ว่าจะจาก source หรือจาก pre-built wheel:

```console
# Install from source (will clone and build)
$ pip install git+https://github.com/psf/requests.git

# Install from a specific tag/branch
$ pip install git+https://github.com/psf/requests.git@v2.32.3

# Install a wheel directly from a GitHub release
$ pip install https://github.com/user/repo/releases/download/v1.0/package-1.0-py3-none-any.whl
```

จริงๆ แล้ว บางภาษาอย่าง Go ใช้ decentralized distribution model --- แทนที่จะมี central package repository Go modules ถูก distribute โดยตรงจาก source code repositories
Module paths อย่าง `github.com/gorilla/mux` บ่งบอกว่าโค้ดอยู่ที่ไหน และ `go get` จะ fetch มาจากที่นั่นโดยตรง อย่างไรก็ตาม package managers ส่วนใหญ่อย่าง `pip`, `cargo` หรือ `brew` มี central indexes ของ pre-packaged projects เพื่อให้ง่ายต่อการ distribution และ installation ถ้าเรารัน

```console
$ uv pip install requests --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: requests==2.32.5 [compatible] (requests-2.32.5-py3-none-any.whl)
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl.metadata
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl
```

จะเห็นว่าเรากำลัง fetch wheel `requests` จากที่ไหน สังเกต `py3-none-any` ในชื่อไฟล์ --- หมายความว่า wheel นี้ทำงานได้กับ Python 3 ทุก version, OS ใดก็ได้ และ architecture ใดก็ได้ สำหรับ packages ที่มี compiled code wheel จะเป็น platform-specific:

```console
$ uv pip install numpy --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: numpy==2.2.1 [compatible] (numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl)
```

ตรงนี้ `cp312-cp312-macosx_14_0_arm64` บ่งบอกว่า wheel นี้เฉพาะสำหรับ CPython 3.12 บน macOS 14+ สำหรับ ARM64 (Apple Silicon) ถ้าอยู่บน platform อื่น `pip` จะ download wheel อื่นหรือ build จาก source

ในทางกลับกัน เพื่อให้คนอื่นค้นหา package ที่เราสร้างได้ เราต้อง publish มันไปยัง registries เหล่านี้
ใน Python registry หลักคือ [Python Package Index (PyPI)](https://pypi.org)
เช่นเดียวกับการ install มีหลายวิธีในการ publish packages คำสั่ง `uv publish` มี interface ที่ทันสมัยสำหรับ upload packages ไปยัง PyPI:

```console
$ uv publish --publish-url https://test.pypi.org/legacy/
Publishing greeting-0.1.0.tar.gz
Publishing greeting-0.1.0-py3-none-any.whl
```

ตรงนี้เราใช้ [TestPyPI](https://test.pypi.org) --- package registry แยกต่างหากที่มีไว้สำหรับทดสอบ publishing workflow โดยไม่ทำให้ PyPI จริงรก เมื่อ upload แล้ว สามารถ install จาก TestPyPI ได้:

```console
$ uv pip install --index-url https://test.pypi.org/simple/ greeting
```

ข้อพิจารณาสำคัญเมื่อ publish ซอฟต์แวร์คือเรื่องความน่าเชื่อถือ ผู้ใช้จะตรวจสอบได้อย่างไรว่า package ที่พวกเขา download มาจากเราจริงๆ และไม่ถูก tamper? Package registries ใช้ checksums เพื่อตรวจสอบความถูกต้อง และบาง ecosystems รองรับ package signing เพื่อให้ cryptographic proof ของ authorship

ภาษาต่างๆ มี package registries ของตัวเอง: [crates.io](https://crates.io) สำหรับ Rust, [npm](https://www.npmjs.com) สำหรับ JavaScript, [RubyGems](https://rubygems.org) สำหรับ Ruby, และ [Docker Hub](https://hub.docker.com) สำหรับ container images สำหรับ private หรือ internal packages องค์กรมักจะ deploy package repositories ของตัวเอง (เช่น private PyPI server หรือ private Docker registry) หรือใช้ managed solutions จาก cloud providers

การ deploy web service ไปยัง internet ต้องมี infrastructure เพิ่มเติม: การจดทะเบียน domain name, การ configure DNS ให้ชี้ domain ไปยัง server, และมักจะมี reverse proxy อย่าง nginx เพื่อจัดการ HTTPS และ route traffic สำหรับกรณีที่ง่ายกว่าอย่าง documentation หรือ static sites [GitHub Pages](https://pages.github.com/) ให้ hosting ฟรีโดยตรงจาก repository

<!--
## Documentation

So far we have emphasized the deliverable _artifact_ as the main output of packaging and shipping code.
In addition to the artifact, we need to document for users the code's functionality, installation instructions, and usage examples.

Tools like [Sphinx](https://www.sphinx-doc.org/) (Python) and [MkDocs](https://www.mkdocs.org/) can automatically generate browsable documentation from docstrings and markdown files, often hosted on services like [Read the Docs](https://readthedocs.org/).
For HTTP-based APIs, the [OpenAPI specification](https://www.openapis.org/) (formerly Swagger) provides a standard format for describing API endpoints, which tools can use to generate interactive documentation and client libraries automatically. -->


# Exercises

1. บันทึก environment ด้วย `printenv` ลงไฟล์ สร้าง venv, activate มัน, `printenv` ลงอีกไฟล์หนึ่ง แล้ว `diff before.txt after.txt` มีอะไรเปลี่ยนแปลงใน environment? ทำไม shell ถึงเลือกใช้ venv? (คำใบ้: ดู `$PATH` ก่อนและหลัง activation) ลองรัน `which deactivate` แล้ววิเคราะห์ว่า deactivate bash function ทำอะไร
2. สร้าง Python package ที่มี `pyproject.toml` แล้ว install มันใน virtual environment สร้าง lockfile แล้วตรวจดูเนื้อหา
3. Install Docker แล้วใช้มัน build เว็บไซต์ Missing Semester class ใน local โดยใช้ docker compose
4. เขียน Dockerfile สำหรับ Python application อย่างง่าย จากนั้นเขียน `docker-compose.yml` ที่รัน application พร้อม Redis cache
5. Publish Python package ไปยัง TestPyPI (อย่า publish ไปยัง PyPI จริง เว้นแต่จะเป็นของที่คุ้มค่าจะแบ่งปัน!) จากนั้น build Docker image ที่มี package ดังกล่าว แล้ว push ไปยัง `ghcr.io`
6. สร้างเว็บไซต์โดยใช้ [GitHub Pages](https://docs.github.com/en/pages/quickstart) เพิ่มเติม (ไม่มีเครดิตให้): configure ด้วย custom domain
