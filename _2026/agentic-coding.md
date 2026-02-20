---
layout: lecture
title: "Agentic Coding"
description: >
  เรียนรู้วิธีใช้ AI coding agent อย่างมีประสิทธิภาพสำหรับงานพัฒนาซอฟต์แวร์
thumbnail: /static/assets/thumbnails/2026/lec7.png
date: 2026-01-21
ready: true
video:
  aspect: 56.25
  id: sTdz6PZoAnw
---

Coding agent คือ AI model แบบสนทนาที่สามารถเข้าถึง tool ต่าง ๆ เช่น อ่าน/เขียนไฟล์, ค้นหาเว็บ, และรันคำสั่ง shell ได้ มันทำงานได้ทั้งใน IDE หรือเป็นเครื่องมือ command-line หรือ GUI แบบ standalone Coding agent เป็นเครื่องมือที่ทำงานได้อัตโนมัติสูงและทรงพลัง รองรับ use case ที่หลากหลาย

บทนี้ต่อยอดจากเนื้อหา AI-powered development ในบท [Development Environment and Tools](/2026/development-environment/) เพื่อเป็น demo สั้น ๆ ลองมาดูตัวอย่างต่อจากส่วน [AI-powered development](/2026/development-environment/#ai-powered-development) กัน

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

เราลอง prompt coding agent ด้วย task นี้ได้

```
Turn this into a proper command-line program, with argparse for argument parsing. Add type annotations, and make sure the program passes type checking.
```

Agent จะอ่านไฟล์เพื่อทำความเข้าใจ จากนั้นแก้ไขโค้ด แล้วรัน type checker เพื่อให้แน่ใจว่า type annotation ถูกต้อง ถ้ามันทำผิดจนไม่ผ่าน type checking มันก็จะวนแก้ไขเอง แม้ว่า task นี้จะง่ายพอที่จะไม่ค่อยเกิดปัญหา เนื่องจาก coding agent มีสิทธิ์เข้าถึง tool ที่อาจเป็นอันตรายได้ โดยค่าเริ่มต้นแล้ว agent harness จะถามยืนยันผู้ใช้ก่อนเรียก tool

> ถ้า coding agent ทำผิดพลาด เช่น คุณมี binary `mypy` อยู่ใน `$PATH` โดยตรง แต่ agent กลับเรียก `python -m mypy` แทน คุณสามารถให้ feedback เป็นข้อความเพื่อช่วยแก้ไขทิศทางได้

Coding agent รองรับการโต้ตอบแบบ multi-turn ทำให้สามารถพัฒนาต่อยอดผ่านการสนทนาไป-มากับ agent ได้ คุณยังสามารถขัดจังหวะ agent ได้หากมันกำลังไปผิดทาง mental model ที่อาจช่วยได้คือมองตัวเองเป็นหัวหน้าที่ดูแลเด็กฝึกงาน: เด็กฝึกงานจะทำงานละเอียดให้ แต่ต้องการคำแนะนำ และอาจทำผิดบ้างซึ่งต้องคอยแก้ไข

> สำหรับ demo ที่ชัดเจนกว่านี้ ลองขอให้ agent รันสคริปต์ที่ได้ต่อเลย สังเกตผลลัพธ์ แล้วลองขอให้มันแก้ไข (เช่น ขอให้เอาเฉพาะ absolute URL)

# AI model และ agent ทำงานอย่างไร

การอธิบายรายละเอียดภายในของ [large language model (LLM)](https://en.wikipedia.org/wiki/Large_language_model) สมัยใหม่ และโครงสร้างพื้นฐานอย่าง agent harness นั้นอยู่นอกขอบเขตของคอร์สนี้ แต่การเข้าใจแนวคิดหลักในระดับ high-level จะช่วยให้_ใช้งาน_เทคโนโลยีล้ำสมัยนี้ได้อย่างมีประสิทธิภาพ และเข้าใจข้อจำกัดของมัน

LLM สามารถมองได้ว่าเป็นการ model probability distribution ของ completion string (output) จาก prompt string (input) การทำ LLM inference (สิ่งที่เกิดขึ้นเมื่อคุณส่ง query ไปยังแอปแชท) คือการ _sample_ จาก probability distribution นี้ LLM มี _context window_ ขนาดคงที่ ซึ่งเป็นความยาวสูงสุดของ input และ output string

{% comment %}
> ในรูปแบบคณิตศาสตร์ LLM จะ model probability distribution $\pi_\theta$ ของ completion $y$ ที่ขึ้นอยู่กับ prompt $x$ แล้วเรา sample จาก distribution นี้: $\hat{y} \sim \pi_\theta(\cdot \mid x)$
{% endcomment %}

AI tool ต่าง ๆ เช่น conversational chat และ coding agent สร้างอยู่บน primitive นี้ สำหรับการโต้ตอบแบบ multi-turn แอปแชทและ agent ใช้ turn marker และส่ง conversation history ทั้งหมดเป็น prompt string ทุกครั้งที่มี user prompt ใหม่ โดยเรียก LLM inference หนึ่งครั้งต่อ user prompt หนึ่งข้อความ สำหรับ tool-calling agent นั้น harness จะตีความ LLM output บางส่วนเป็นคำร้องขอเรียก tool แล้ว harness จะส่งผลลัพธ์ของ tool call กลับไปให้ model เป็นส่วนหนึ่งของ prompt string (ดังนั้น LLM inference จะรันใหม่ทุกครั้งที่มี tool call/response) แนวคิดหลักของ tool-calling agent สามารถ [implement ได้ใน 200 บรรทัด](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)

## ความเป็นส่วนตัว

AI coding tool ส่วนใหญ่ในการตั้งค่าเริ่มต้นจะส่งข้อมูลของคุณจำนวนมากขึ้น cloud บางครั้ง harness รันแบบ local ในขณะที่ LLM inference รันบน cloud บางครั้งซอฟต์แวร์ส่วนใหญ่ก็รันบน cloud (และเช่น ผู้ให้บริการอาจได้รับสำเนา repository ทั้งหมดรวมถึงการโต้ตอบทุกครั้งที่คุณมีกับ AI tool)

มี AI coding tool แบบ open-source และ LLM แบบ open-source ที่ค่อนข้างดี (แม้จะยังไม่ดีเท่า proprietary model) แต่ในปัจจุบันสำหรับผู้ใช้ส่วนใหญ่ การรัน open LLM ล้ำสมัยแบบ local นั้นยังทำไม่ได้จริงเนื่องจากข้อจำกัดด้านฮาร์ดแวร์

# Use case

Coding agent ช่วยได้ในงานหลากหลายประเภท ตัวอย่างเช่น

- **การ implement feature ใหม่** เหมือนในตัวอย่างข้างต้น คุณสามารถขอให้ coding agent implement feature ได้ การเขียน specification ที่ดีนั้นเป็นศิลปะมากกว่าวิทยาศาสตร์ในตอนนี้ คุณต้องให้ input กับ agent อย่างละเอียดพอที่มันจะทำในสิ่งที่คุณต้องการ (อย่างน้อยก็มุ่งไปถูกทิศทางเพื่อให้คุณ iterate ต่อได้) แต่ไม่ละเอียดเกินจนคุณทำงานเองมากไป Test-driven development จะได้ผลดีเป็นพิเศษ: เขียน test (หรือใช้ coding agent ช่วยเขียน test) ตรวจสอบให้แน่ใจว่า test ครอบคลุมสิ่งที่ต้องการ จากนั้นขอให้ coding agent implement feature Model ดีขึ้นเรื่อย ๆ ดังนั้นคุณต้องคอยอัปเดตความรู้สึกว่า model สามารถทำอะไรได้บ้าง
    > เราใช้ Claude Code เพื่อ [implement](https://github.com/missing-semester/missing-semester/pull/345) sidenote แบบ Tufte เหล่านี้
{%- comment %}
ไม่จำเป็นต้อง demo ส่วนนี้ เพราะตอนเปิดบทก็เป็น demo เล็ก ๆ ของการเพิ่ม feature ใหม่แล้ว
{% endcomment %}
- **การแก้ error** ถ้ามี error จาก compiler, linter, type checker หรือ test คุณสามารถขอให้ agent แก้ไขได้ เช่น prompt ว่า "fix the issues with mypy" Coding model จะทำงานได้ดีเป็นพิเศษเมื่ออยู่ใน feedback loop ดังนั้นพยายามจัดสภาพแวดล้อมให้ model สามารถรัน failing check ได้โดยตรง ซึ่งจะทำให้มัน iterate ได้แบบอัตโนมัติ ถ้าทำแบบนี้ไม่สะดวก คุณก็ให้ feedback กับ model เองได้
    > ใน commit [f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd) ของ missing-semester repo เราใช้ Claude Code กับ prompt "Review the agentic coding lecture for typos and grammatical issues" แล้วตามด้วยให้มันแก้ปัญหาที่พบ ซึ่ง commit ไว้ใน [f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637)
{%- comment %}
Demo การใช้ coding agent แก้ bug ใน https://github.com/anishathalye/dotbot/commit/cef40c902ef0f52f484153413142b5154bbc5e99

เขียน failing test เพื่อ demo bug แล้วขอให้ agent แก้ไข เตรียมไว้ใน branch demo-bugfix

รัน failing test ได้ด้วย:

    hatch test tests/test_cli.py::test_issue_357

Prompt coding agent ได้ด้วย:

    There is a bug I wrote a failing test for, you can repro it with `hatch test tests/test_cli.py::test_issue_357`. Fix the bug.

ให้มัน commit การเปลี่ยนแปลง
{% endcomment %}
- **Refactoring** คุณสามารถใช้ coding agent เพื่อ refactor โค้ดในรูปแบบต่าง ๆ ตั้งแต่งานง่าย ๆ อย่างการเปลี่ยนชื่อ method (การ refactor แบบนี้ยังรองรับโดย [code intelligence](/2026/development-environment/#code-intelligence-and-language-servers) ด้วย) ไปจนถึงงานซับซ้อนอย่างการแยก functionality ออกเป็น module แยกต่างหาก
    > เราใช้ Claude Code เพื่อ [แยก](https://github.com/missing-semester/missing-semester/pull/344) agentic coding ออกเป็นบทเรียนของตัวเอง
{%- comment %}
แสดงการใช้งานใน Missing Semester ชี้ให้เห็นว่า agent ก็ทำผิดพลาดบ้าง
{% endcomment %}
- **Code review** คุณสามารถขอให้ coding agent review โค้ดได้ โดยให้คำแนะนำเบื้องต้น เช่น "review my latest changes that are not yet committed" ถ้าต้องการ review pull request และ coding agent รองรับ web fetch หรือคุณมีเครื่องมือ command-line อย่าง [GitHub CLI](https://cli.github.com/) ติดตั้งอยู่ คุณอาจขอให้ coding agent "Review the pull request {link}" แล้วมันจะจัดการให้เอง
{%- comment %}
ใน Porcupine repo ให้ prompt agent ว่า:

    Review this PR: https://github.com/anishathalye/porcupine/pull/39
{% endcomment %}
- **ทำความเข้าใจโค้ด** คุณสามารถถาม coding agent เกี่ยวกับ codebase ได้ ซึ่งมีประโยชน์มากสำหรับการ onboard เข้าโปรเจกต์ใหม่
{%- comment %}
ตัวอย่าง prompt ใน missing-semester repo:

    How do I run this site locally?

    How are the social preview cards implemented?
{% endcomment %}
- **ใช้แทน shell** คุณสามารถขอให้ coding agent ใช้เครื่องมือเฉพาะทางเพื่อทำงาน ทำให้เรียกคำสั่ง shell ด้วยภาษาธรรมชาติได้ เช่น "use the find command to find all files older than 30 days" หรือ "use mogrify to resize all the jpgs to 50% of their original size"
{%- comment %}
ใน Dotbot repo ให้ prompt agent ว่า:

    Use the ag command to find all Python renaming imports
{% endcomment %}
- **Vibe coding** Agent ทรงพลังพอที่คุณจะ implement แอปพลิเคชันบางตัวได้โดยไม่ต้องเขียนโค้ดเองแม้แต่บรรทัดเดียว
    > [นี่คือตัวอย่าง](https://github.com/cleanlab/office-presence-dashboard) ของโปรเจกต์จริงที่อาจารย์ท่านหนึ่ง vibe-code ขึ้นมา
{%- comment %}
ใน missing-semester repo ให้ prompt agent ว่า:

    Make this site look retro.
{% endcomment %}

# Advanced agent

ในส่วนนี้จะให้ภาพรวมสั้น ๆ ของรูปแบบการใช้งานขั้นสูงและความสามารถเพิ่มเติมของ coding agent

- **Reusable prompt** สร้าง prompt หรือ template ที่นำกลับมาใช้ซ้ำได้ เช่น เขียน prompt ละเอียดสำหรับทำ code review ในแบบเฉพาะเจาะจง แล้วบันทึกเป็น reusable prompt
- **Parallel agent** Coding agent อาจทำงานช้า: คุณ prompt agent แล้วมันอาจทำงานนานหลายสิบนาที คุณสามารถรัน agent หลายตัวพร้อมกันได้ ทั้งทำงานเดียวกัน (LLM เป็น stochastic ดังนั้นการรันซ้ำหลายครั้งแล้วเลือกคำตอบที่ดีที่สุดก็มีประโยชน์) หรือทำคนละงาน (เช่น implement สอง feature ที่ไม่ทับซ้อนกันพร้อมกัน) เพื่อไม่ให้การเปลี่ยนแปลงของ agent แต่ละตัวกระทบกัน สามารถใช้ [git worktree](https://git-scm.com/docs/git-worktree) ซึ่งเราจะพูดถึงในบทเรื่อง [version control](/2026/version-control/)
- **MCP** MCP ย่อมาจาก _Model Context Protocol_ เป็น open protocol ที่ใช้เชื่อมต่อ coding agent กับ tool ต่าง ๆ เช่น [Notion MCP server](https://github.com/makenotion/notion-mcp-server) สามารถให้ agent อ่าน/เขียนเอกสาร Notion ได้ รองรับ use case อย่าง "อ่าน spec ที่ลิงก์ไว้ใน {Notion doc}, ร่างแผน implementation เป็นหน้าใหม่ใน Notion แล้ว implement prototype" สำหรับการค้นหา MCP สามารถใช้ directory อย่าง [Pulse](https://www.pulsemcp.com/servers) และ [Glama](https://glama.ai/mcp/servers)
- **การจัดการ context** ดังที่กล่าวไว้ [ข้างต้น](#how-ai-models-and-agents-work) LLM ที่อยู่เบื้องหลัง coding agent มี _context window_ จำกัด การใช้ coding agent อย่างมีประสิทธิภาพต้องอาศัยการจัดการ context ให้ดี ต้องมั่นใจว่า agent เข้าถึงข้อมูลที่จำเป็น แต่หลีกเลี่ยง context ที่ไม่จำเป็นเพื่อไม่ให้ context window ล้นหรือทำให้ประสิทธิภาพของ model ลดลง (ซึ่งมักเกิดขึ้นเมื่อ context ใหญ่ขึ้น แม้จะไม่ล้น context window ก็ตาม) Agent harness จะจัดการ context ให้โดยอัตโนมัติในระดับหนึ่ง แต่การควบคุมส่วนใหญ่ยังอยู่ที่ผู้ใช้
    - **การล้าง context window** การควบคุมพื้นฐานที่สุด coding agent รองรับการล้าง context window (เริ่มการสนทนาใหม่) ซึ่งควรทำสำหรับ query ที่ไม่เกี่ยวข้องกัน
    - **การย้อนกลับการสนทนา** Coding agent บางตัวรองรับการ undo ขั้นตอนใน conversation history แทนที่จะส่งข้อความ follow-up เพื่อเปลี่ยนทิศทาง agent ในสถานการณ์ที่ "undo" เหมาะสมกว่า วิธีนี้จะจัดการ context ได้ดีกว่า
{%- comment %}
สร้าง demo สั้น ๆ ขึ้นมา
{% endcomment %}
    - **Compaction** เพื่อรองรับการสนทนาที่ยาวไม่จำกัด coding agent รองรับ context _compaction_: ถ้า conversation history ยาวเกินไป มันจะเรียก LLM มาสรุปส่วนต้นของการสนทนาโดยอัตโนมัติ แล้วแทนที่ conversation history ด้วยบทสรุป Agent บางตัวให้ผู้ใช้เรียก compaction เองเมื่อต้องการ
{%- comment %}
แสดง `/compact` ใน Claude Code แสดง summary ทั้งหมด
{% endcomment %}
    - **llms.txt** ไฟล์ `/llms.txt` เป็น [มาตรฐาน](https://llmstxt.org/) ที่เสนอขึ้นมาเพื่อเป็นตำแหน่งของเอกสารที่ LLM จะใช้ตอน inference Product ต่าง ๆ (เช่น [cursor.com/llms.txt](https://cursor.com/llms.txt)), software library (เช่น [ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt)), และ API (เช่น [apify.com/llms.txt](https://apify.com/llms.txt)) อาจมีไฟล์ `llms.txt` ที่เป็นประโยชน์สำหรับการพัฒนา เอกสารเหล่านี้มีความหนาแน่นของข้อมูลต่อ token สูงกว่า จึงมีประสิทธิภาพด้าน context มากกว่าการให้ coding agent fetch และอ่านหน้า HTML เอกสารภายนอกจะมีประโยชน์เมื่อ coding agent ไม่มีความรู้เกี่ยวกับ dependency ที่คุณกำลังใช้ (เช่น เพราะมันถูกเผยแพร่หลัง knowledge cutoff ของ LLM)
{%- comment %}
เปรียบเทียบแบบเคียงข้างกันใน repo ว่าง (บน Desktop หรือที่อื่นที่แยกออกมา โดยรัน `git init` ไว้แล้ว):

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers.

    Write a single-file Python program example in demo.py using semlib to sort "Ilya Sutskever", "Soumith Chintala", and "Donald Knuth" in terms of their fame as AI researchers. See https://semlib.anish.io/llms.txt. Follow links to Markdown versions of any pages linked in llms.txt files.

ไม่แน่ใจว่าทำไม agent ถึงไม่ทำแบบนี้เป็นค่าเริ่มต้น คุณอาจใส่ประโยคสุดท้ายนั้นในไฟล์ CLAUDE.md
{% endcomment %}
    - **AGENTS.md** Coding agent ส่วนใหญ่รองรับ [AGENTS.md](https://agents.md/) หรือรูปแบบที่คล้ายกัน (เช่น Claude Code มองหา `CLAUDE.md`) เป็น README สำหรับ coding agent เมื่อ agent เริ่มทำงาน มันจะโหลดเนื้อหาทั้งหมดของ `AGENTS.md` เข้า context ล่วงหน้า คุณสามารถใช้ไฟล์นี้เพื่อให้คำแนะนำกับ agent ที่ใช้ร่วมกันได้ทุก session (เช่น สั่งให้รัน type checker ทุกครั้งหลังแก้โค้ด, อธิบายวิธีรัน unit test, หรือให้ลิงก์ไปยังเอกสาร third-party ที่ agent สามารถเปิดดูได้) Coding agent บางตัวสามารถสร้างไฟล์นี้โดยอัตโนมัติ (เช่น คำสั่ง `/init` ใน Claude Code) ดู [ที่นี่](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md) สำหรับตัวอย่าง `AGENTS.md` จากโปรเจกต์จริง
{%- comment %}
ตัวอย่าง Dotbot, CLAUDE.md ที่ include @DEVELOPMENT.md และระบุให้รัน type checker และ code formatter ทุกครั้งหลังแก้ไขโค้ด Python

ตัวอย่าง prompt จาก master:

    Remove the "--version" command-line flag.

เป็นงานที่ทำเร็ว เพื่อวัตถุประสงค์ในการ demo
{% endcomment %}
    - **Skill** เนื้อหาใน `AGENTS.md` จะถูกโหลดทั้งหมดเข้า context window ของ agent เสมอ _Skill_ เพิ่มอีกหนึ่งระดับของ indirection เพื่อหลีกเลี่ยง context bloat: คุณสามารถให้รายการ skill พร้อมคำอธิบายกับ agent แล้ว agent จะ "เปิด" skill (โหลดเข้า context window) ตามที่ต้องการ
    - **Subagent** Coding agent บางตัวให้คุณกำหนด subagent ซึ่งเป็น agent สำหรับ workflow เฉพาะงาน Agent ระดับบนสุดสามารถเรียก subagent เพื่อทำงานเฉพาะอย่าง ทำให้ทั้ง agent ระดับบนสุดและ subagent จัดการ context ได้มีประสิทธิภาพมากขึ้น Context ของ agent ระดับบนสุดไม่ถูกเติมด้วยทุกอย่างที่ subagent เห็น และ subagent ก็ได้เฉพาะ context ที่จำเป็นสำหรับงานของมัน ตัวอย่างหนึ่ง coding agent บางตัว implement การค้นหาเว็บเป็น subagent: agent ระดับบนสุดจะส่งคำถามไปยัง subagent ซึ่งจะรัน web search, ดึงหน้าเว็บแต่ละหน้า, วิเคราะห์ แล้วส่งคำตอบกลับมา วิธีนี้ทำให้ context ของ agent ระดับบนสุดไม่ถูกเติมด้วยเนื้อหาทั้งหมดของหน้าเว็บที่ดึงมา และ subagent ก็ไม่ต้องมี conversation history ทั้งหมดของ agent ระดับบนสุดใน context ของมัน

สำหรับ feature ขั้นสูงหลายอย่างที่ต้องเขียน prompt (เช่น skill หรือ subagent) คุณสามารถใช้ LLM ช่วยเริ่มต้นได้ Coding agent บางตัวมี built-in support สำหรับทำเรื่องนี้ เช่น Claude Code สามารถสร้าง subagent จาก prompt สั้น ๆ ได้ (เรียก `/agents` แล้วสร้าง agent ใหม่) ลองสร้าง subagent ด้วย prompt นี้

```
A Python code checking agent that uses `mypy` and `ruff` to type-check, lint, and format *check* any files that have been modified from the last git commit.
```

จากนั้นคุณสามารถใช้ agent ระดับบนสุดเพื่อเรียก subagent อย่างชัดเจนด้วยข้อความเช่น "use the code checker subagent" คุณยังอาจตั้งค่าให้ agent ระดับบนสุดเรียก subagent โดยอัตโนมัติเมื่อเหมาะสม เช่น หลังจากแก้ไขไฟล์ Python ใด ๆ

# สิ่งที่ควรระวัง

AI tool อาจทำผิดพลาดได้ มันสร้างอยู่บน LLM ซึ่งเป็นเพียง model ที่ทำนาย token ถัดไปแบบ probabilistic มันไม่ได้ "ฉลาด" ในแบบเดียวกับมนุษย์ ตรวจสอบผลลัพธ์จาก AI ทั้งในด้านความถูกต้องและ security bug บางครั้งการตรวจสอบโค้ดอาจยากกว่าการเขียนเอง สำหรับโค้ดที่สำคัญ ควรพิจารณาเขียนเอง AI อาจลงไปใน rabbit hole และพยายาม gaslight คุณ ระวังเรื่อง debugging spiral อย่าใช้ AI เป็นไม้ค้ำยัน และระวังการพึ่งพามากเกินไปหรือเข้าใจเพียงผิวเผิน งาน programming อีกจำนวนมากที่ AI ยังทำไม่ได้ Computational thinking ยังคงมีคุณค่า

# ซอฟต์แวร์ที่แนะนำ

IDE และ AI coding extension จำนวนมากมี coding agent ในตัว (ดูคำแนะนำจาก [บท development environment](/2026/development-environment/)) Coding agent ยอดนิยมอื่น ๆ ได้แก่ [Claude Code](https://www.claude.com/product/claude-code) ของ Anthropic, [Codex](https://openai.com/codex/) ของ OpenAI, และ agent แบบ open-source อย่าง [opencode](https://github.com/anomalyco/opencode)

# แบบฝึกหัด

1. เปรียบเทียบประสบการณ์ของการเขียนโค้ดเอง, ใช้ AI autocomplete, inline chat, และ agent โดยทำ programming task เดียวกันสี่ครั้ง ตัวเลือกที่ดีที่สุดคือ feature ขนาดเล็กจากโปรเจกต์ที่คุณกำลังทำอยู่ ถ้าต้องการไอเดียอื่น ลองพิจารณาทำ task แบบ "good first issue" ในโปรเจกต์ open-source บน GitHub หรือโจทย์จาก [Advent of Code](https://adventofcode.com/) หรือ [LeetCode](https://leetcode.com/)
2. ใช้ AI coding agent เพื่อสำรวจ codebase ที่ไม่คุ้นเคย วิธีที่ดีที่สุดคือทำในบริบทที่ต้องการ debug หรือเพิ่ม feature ใหม่ให้กับโปรเจกต์ที่คุณสนใจจริง ๆ ถ้านึกไม่ออก ลองใช้ AI agent เพื่อทำความเข้าใจว่า feature ด้าน security ทำงานอย่างไรใน agent [opencode](https://github.com/anomalyco/opencode)
3. Vibe code แอปเล็ก ๆ จากศูนย์ ห้ามเขียนโค้ดเองแม้แต่บรรทัดเดียว
4. สำหรับ coding agent ที่คุณเลือกใช้ ให้สร้างและทดสอบ `AGENTS.md` (หรือชื่อที่เทียบเท่า เช่น `CLAUDE.md`), reusable prompt (เช่น [custom slash command ใน Claude Code](https://code.claude.com/docs/en/slash-commands) หรือ [custom prompt ใน Codex](https://developers.openai.com/codex/custom-prompts)), skill (เช่น [skill ใน Claude Code](https://code.claude.com/docs/en/skills) หรือ [skill ใน Codex](https://developers.openai.com/codex/skills/)), และ subagent (เช่น [subagent ใน Claude Code](https://code.claude.com/docs/en/sub-agents)) ลองคิดว่าเมื่อไหร่ควรใช้อันไหน โปรดทราบว่า coding agent ที่คุณเลือกอาจไม่รองรับบาง functionality เหล่านี้ คุณสามารถข้ามไปหรือลองใช้ coding agent ตัวอื่นที่รองรับ
5. ใช้ coding agent เพื่อทำงานเดียวกับแบบฝึกหัด Markdown bullet point regex จาก [บท Code Quality](/2026/code-quality/) มันทำงานผ่านการแก้ไขไฟล์โดยตรงหรือไม่? ข้อเสียและข้อจำกัดของการที่ agent แก้ไขไฟล์โดยตรงเพื่อทำงานดังกล่าวคืออะไร? หาวิธี prompt agent ให้ไม่ทำงานผ่านการแก้ไขไฟล์โดยตรง คำใบ้: ขอให้ agent ใช้เครื่องมือ command-line ที่กล่าวถึงใน [บทแรก](/2026/course-shell/)
6. Coding agent ส่วนใหญ่รองรับ "yolo mode" บางรูปแบบ (เช่น ใน Claude Code คือ `--dangerously-skip-permissions`) การใช้โหมดนี้โดยตรงไม่ปลอดภัย แต่อาจยอมรับได้หากรัน coding agent ในสภาพแวดล้อมที่แยกออกมาอย่าง virtual machine หรือ container แล้วจึงเปิดใช้งาน autonomous operation ให้ตั้งค่าระบบนี้บนเครื่องของคุณ เอกสารเช่น [Claude Code devcontainer](https://code.claude.com/docs/en/devcontainer) หรือ [Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/claude-code/) อาจเป็นประโยชน์ มีหลายวิธีในการตั้งค่านี้
