---
layout: lecture
title: "Code Quality"
description: >
  เรียนรู้เกี่ยวกับ formatting, linting, testing, continuous integration และอื่น ๆ
thumbnail: /static/assets/thumbnails/2026/lec9.png
date: 2026-01-23
ready: true
video:
  aspect: 56.25
  id: XBiLUNx84CQ
---

มีเครื่องมือและเทคนิคหลากหลายที่ช่วยให้นักพัฒนาเขียนโค้ดที่มีคุณภาพสูง ในบทนี้จะครอบคลุมเรื่อง:

- [Formatting](#formatting)
- [Linting](#linting)
- [Testing](#testing)
- [Pre-commit hooks](#pre-commit-hooks)
- [Continuous integration](#continuous-integration)
- [Command runners](#command-runners)

เป็นหัวข้อเสริม เราจะพูดถึง [regular expressions](#regular-expressions) ด้วย ซึ่งเป็นหัวข้อที่มีการใช้งานข้ามศาสตร์ ทั้งในด้าน code quality (เช่น การรัน test เฉพาะบางส่วนที่ตรงกับ pattern) รวมถึงด้านอื่น ๆ อย่าง IDE (เช่น การ search and replace)

เครื่องมือหลายตัวจะเป็นแบบเฉพาะภาษา (เช่น [Ruff](https://docs.astral.sh/ruff/) ซึ่งเป็น linter/formatter สำหรับ Python) แต่ในบางกรณีเครื่องมือจะรองรับหลายภาษา (เช่น [Prettier](https://prettier.io/) ซึ่งเป็น code formatter) อย่างไรก็ตาม แนวคิดเหล่านี้ใกล้เคียงกับสากล --- จะหา code formatter, linter, testing library และอื่น ๆ ได้สำหรับภาษาโปรแกรมใดก็ได้

# Formatting

Code auto-formatter จะจัดรูปแบบ surface syntax ให้สวยงามโดยอัตโนมัติ วิธีนี้ทำให้สามารถโฟกัสกับปัญหาที่ลึกและท้าทายกว่าได้ ขณะที่เครื่องมือ auto-formatting จัดการรายละเอียดปลีกย่อยให้ เช่น ความสม่ำเสมอระหว่างการใช้ `'` กับ `"` สำหรับ string, การมีช่องว่างรอบ binary operator (`x + y` แทน `x+y`), การเรียง `import` statement ตามลำดับ, และการหลีกเลี่ยงบรรทัดที่ยาวเกินไป ประโยชน์หลักอย่างหนึ่งของ code formatter คือการทำให้ code style เป็นมาตรฐานเดียวกันในทุกคนที่ทำงานบน codebase เดียวกัน

เครื่องมือบางตัวอย่าง Prettier มี[ตัวเลือกการตั้งค่าจำนวนมาก](https://prettier.io/docs/configuration) ควร check in ไฟล์ configuration เข้าไปใน [version control](/2026/version-control/) ของโปรเจกต์ด้วย เครื่องมืออื่น ๆ เช่น [Black](https://github.com/psf/black) และ [gofmt](https://pkg.go.dev/cmd/gofmt) มีตัวเลือกการปรับแต่งจำกัดหรือไม่มีเลย เพื่อลด [bikeshedding](https://en.wikipedia.org/wiki/Law_of_triviality) (การถกเถียงเรื่องเล็กน้อยที่ไม่สำคัญ)

สามารถตั้งค่า [IDE integration](/2026/development-environment/#code-intelligence-and-language-servers) กับ code formatter ได้ เพื่อให้โค้ดถูก auto-format ขณะพิมพ์หรือเมื่อบันทึกไฟล์ นอกจากนี้ยังสามารถเพิ่มไฟล์ [EditorConfig](https://editorconfig.org/) เข้าไปในโปรเจกต์ ซึ่งจะบอก IDE ถึงการตั้งค่าระดับโปรเจกต์ เช่น indent size สำหรับแต่ละประเภทไฟล์

# Linting

Linter ทำ static analysis (วิเคราะห์โค้ดโดยไม่ต้องรัน) เพื่อหา antipattern และปัญหาที่อาจเกิดขึ้นในโค้ด เครื่องมือเหล่านี้ทำงานลึกกว่า autoformatter โดยมองไกลกว่าแค่ surface syntax ความลึกของการวิเคราะห์จะแตกต่างกันไปตามเครื่องมือ

Linter มาพร้อมกับรายการ _rule_ ต่าง ๆ โดยมี preset ที่สามารถตั้งค่าได้ในระดับโปรเจกต์ rule ของ linter บางตัวอาจให้ผลลัพธ์ที่เป็น false positive ได้ จึงสามารถปิดการทำงานได้ในระดับไฟล์หรือระดับบรรทัด

Linter ที่ดีจะมี help หรือเอกสารในตัวที่อธิบาย rule แต่ละตัว --- rule นั้นมองหาอะไร ทำไมถึงไม่ดี และอะไรเป็นทางเลือกที่ดีกว่าสำหรับ code pattern นั้น ตัวอย่างเช่น ดูเอกสารของ rule [SIM102](https://docs.astral.sh/ruff/rules/collapsible-if/) ใน [Ruff](https://docs.astral.sh/ruff/) ซึ่งจับ `if` statement ที่ซ้อนกันโดยไม่จำเป็นในโค้ด Python

Linter บางตัวไม่ได้แค่แจ้งปัญหา แต่ยังสามารถ fix ปัญหาบางอย่างให้โดยอัตโนมัติได้อีกด้วย

นอกจาก linter เฉพาะภาษาแล้ว เครื่องมืออีกตัวที่อาจมีประโยชน์คือ [semgrep](https://github.com/semgrep/semgrep) ซึ่งเป็นเครื่องมือ "semantic grep" ที่ทำงานในระดับ AST (แทนที่จะเป็นระดับตัวอักษรอย่าง grep) และรองรับหลายภาษา สามารถใช้ semgrep เขียน custom linter rule สำหรับโปรเจกต์ได้อย่างง่ายดาย ตัวอย่างเช่น ถ้าต้องการป้องกันการใช้ `subprocess.Popen(..., shell=True)` ที่อันตรายใน Python สามารถค้นหา code pattern นั้นได้ด้วย:

```bash
semgrep -l python -e "subprocess.Popen(..., shell=True, ...)"
```

# Testing

Software testing เป็นเทคนิคมาตรฐานในการเพิ่มความมั่นใจว่าโค้ดทำงานถูกต้อง เขียนโค้ดขึ้นมา แล้วก็เขียนโค้ดเพิ่มเพื่อทดสอบโค้ดที่เขียนไว้ และแจ้ง error ถ้าโค้ดไม่ทำงานตามที่คาดไว้

สามารถเขียน test สำหรับโค้ดในระดับความละเอียดที่แตกต่างกันได้: _unit test_ สำหรับ function แต่ละตัว, _integration test_ สำหรับการทำงานร่วมกันระหว่าง module หรือ service, และ _functional test_ สำหรับสถานการณ์ end-to-end สามารถทำ _test-driven development_ โดยเขียน test ก่อนเขียนโค้ดจริง เมื่อพบ bug ในโค้ด สามารถเขียน _regression test_ เพื่อจับได้ถ้า functionality นั้นพังในอนาคต สามารถเขียน _property-based test_ ซึ่งเริ่มต้นจาก [QuickCheck](https://hackage.haskell.org/package/QuickCheck) ใน Haskell และถูก implement ในหลาย library เช่น [Hypothesis](https://hypothesis.readthedocs.io/) สำหรับ Python แนวทาง testing ที่เหมาะสมขึ้นอยู่กับโปรเจกต์ โดยมักจะใช้หลายแนวทางผสมกัน

ถ้าโปรแกรมมี external dependency อย่างฐานข้อมูลหรือ web API การ _mock_ dependency เหล่านั้นใน test อาจเป็นประโยชน์ แทนที่จะให้โค้ดไปเชื่อมต่อกับ third-party dependency ตอนรัน test

## Code coverage

Code coverage เป็น metric ที่ใช้วัดว่า test ดีแค่ไหน code coverage จะดูว่าบรรทัดไหนของโค้ดถูก execute เมื่อรัน test เพื่อให้มั่นใจว่าครอบคลุมทุก code path เครื่องมือ code coverage สามารถแสดง coverage ทีละบรรทัดเพื่อช่วยนำทางในการเขียน test บริการอย่าง [Codecov](https://app.codecov.io) มี web interface สำหรับติดตามและดู code coverage ตลอดประวัติของโปรเจกต์

เช่นเดียวกับ metric อื่น ๆ code coverage ไม่ได้สมบูรณ์แบบ อย่าให้น้ำหนักกับ coverage มากเกินไป ให้เน้นเขียน test ที่มีคุณภาพสูง

# Pre-commit hooks

Git pre-commit [hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) ซึ่งใช้งานง่ายขึ้นด้วย framework [pre-commit](https://pre-commit.com/) จะรันโค้ดที่ผู้ใช้กำหนดโดยอัตโนมัติก่อนทุก Git commit โปรเจกต์ทั่วไปใช้ pre-commit hook เพื่อรัน formatter และ linter และบางครั้ง test โดยอัตโนมัติก่อนทุก commit เพื่อให้มั่นใจว่าโค้ดที่ commit ตรงตาม code style ของโปรเจกต์และปราศจากปัญหาบางประเภท

# Continuous integration

บริการ continuous integration (CI) เช่น [GitHub Actions](https://github.com/features/actions) สามารถรัน script ให้ทุกครั้งที่ push โค้ด (หรือทุก pull request หรือตามตาราง) นักพัฒนาทั่วไปใช้บริการ CI เพื่อรันเครื่องมือ code quality รวมถึง formatter, linter และ test สำหรับ compiled language สามารถตรวจสอบว่าโค้ด compile ได้ สำหรับ statically typed language สามารถตรวจสอบว่า type check ผ่าน การรัน CI ทุกครั้งที่ push commit ใหม่ช่วยจับ error ที่ถูกนำเข้าสู่ version หลักของโค้ด การรันบน pull request ช่วยจับปัญหาจากการ submit ของ contributor การรันตามตารางช่วยจับปัญหาจาก external dependency (เช่น นักพัฒนาปล่อย breaking change โดยไม่ตั้งใจในรูปแบบที่ [เข้ากันได้กับ semver](/2026/shipping-code/#releases--versioning))

เนื่องจาก CI script รันแยกจากเครื่องของนักพัฒนา จึงสามารถรัน job ที่ใช้เวลานานได้อย่างง่ายดาย สามารถนำไปใช้ประโยชน์ได้ เช่น การรัน test แบบ _matrix_ ข้ามระบบปฏิบัติการและเวอร์ชันของภาษาโปรแกรมที่แตกต่างกัน เพื่อให้มั่นใจว่าซอฟต์แวร์ทำงานได้อย่างถูกต้องบนทุก platform

โดยทั่วไปแล้ว script ที่รันใน CI จะไม่ทำการเปลี่ยนแปลงโค้ดโดยตรง แต่จะรันเครื่องมือในโหมด "check-only" แทนโหมด "fix" ตัวอย่างเช่น auto-formatter จะแจ้ง error เมื่อโค้ดไม่เป็นไปตาม format ที่กำหนด

Repository มักจะมี [status badge](https://docs.github.com/en/actions/how-tos/monitor-workflows/add-a-status-badge) ใน README แสดงสถานะ CI และข้อมูลอื่น ๆ เช่น code coverage ตัวอย่างด้านล่างคือสถานะ build ปัจจุบันของ Missing Semester

[![Build Status](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml) [![Links Status](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml/badge.svg)](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml)

> [links checker](https://github.com/missing-semester/missing-semester/blob/master/.github/workflows/links.yml) ของเราซึ่งใช้ GitHub Action [proof-html](https://github.com/anishathalye/proof-html) มักจะ fail อยู่เรื่อย ๆ โดยปกติเกิดจากปัญหาของเว็บไซต์ภายนอก แต่ก็ช่วยให้เราจับและแก้ไข broken link จำนวนมากได้ (บางครั้งเกิดจากพิมพ์ผิด ส่วนใหญ่เกิดจากเว็บไซต์ย้ายเนื้อหาโดยไม่ใส่ redirect หรือเว็บไซต์หายไป)

วิธีที่ดีในการเรียนรู้รายละเอียดของบริการ CI, formatter, linter และ testing library คือการดูตัวอย่าง หาโปรเจกต์ open-source คุณภาพดีบน GitHub --- ยิ่งคล้ายกับโปรเจกต์ของตัวเองในด้านภาษาโปรแกรม, domain, ขนาดและขอบเขต ยิ่งดี --- แล้วศึกษาไฟล์ `pyproject.toml`, `.github/workflows/`, `DEVELOPMENT.md` และไฟล์อื่น ๆ ที่เกี่ยวข้อง

## Continuous deployment

Continuous deployment ใช้ประโยชน์จาก CI infrastructure ในการ _deploy_ การเปลี่ยนแปลงจริง ๆ ตัวอย่างเช่น repository ของ Missing Semester ใช้ continuous deployment ไปยัง GitHub Pages เพื่อให้ทุกครั้งที่ `git push` lecture note ที่อัปเดต เว็บไซต์จะถูก build และ deploy โดยอัตโนมัติ สามารถ build [artifact](/2026/shipping-code/) ประเภทอื่น ๆ ใน CI ได้ด้วย เช่น binary สำหรับ application หรือ Docker image สำหรับ service

# Command runners

Command runner เช่น [just](https://github.com/casey/just) ทำให้การรันคำสั่งในบริบทของโปรเจกต์ง่ายขึ้น เมื่อสร้าง code quality infrastructure สำหรับโปรเจกต์แล้ว ไม่ควรให้นักพัฒนาต้องจำคำสั่งอย่าง `uv run ruff check --fix` ด้วย command runner คำสั่งนี้จะกลายเป็นแค่ `just lint` และสามารถมีการเรียกใช้แบบคล้าย ๆ กันอย่าง `just format`, `just typecheck` เป็นต้น สำหรับเครื่องมือต่าง ๆ ที่นักพัฒนาอาจต้องการรันในโปรเจกต์

Project manager หรือ package manager เฉพาะภาษาบางตัวมี built-in support สำหรับฟังก์ชันนี้ ซึ่งหมายความว่าไม่จำเป็นต้องใช้เครื่องมือที่ไม่ผูกกับภาษาอย่าง `just` ตัวอย่างเช่น section `scripts` ใน `package.json` สำหรับ [npm](https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager) (Node.js) และ section `tool.hatch.envs.*.scripts` ใน `pyproject.toml` สำหรับ [Hatch](https://hatch.pypa.io/) (Python) รองรับฟังก์ชันนี้

# Regular expressions

_Regular expression_ หรือที่เรียกกันย่อ ๆ ว่า "regex" เป็นภาษาที่ใช้แทนเซตของ string regex pattern ใช้กันทั่วไปสำหรับ pattern matching ในบริบทต่าง ๆ เช่น เครื่องมือ command-line และ IDE ตัวอย่างเช่น [ag](https://github.com/ggreer/the_silver_searcher) รองรับ regex pattern สำหรับการค้นหาทั้ง codebase (เช่น `ag "import .* as .*"` จะหา renamed import ทั้งหมดใน Python) และ [go test](https://pkg.go.dev/cmd/go#hdr-Test_packages) รองรับ option `-run [regexp]` สำหรับเลือก test บางส่วน นอกจากนี้ ภาษาโปรแกรมมี built-in support หรือ third-party library สำหรับ regular expression matching ทำให้สามารถใช้ regex สำหรับฟังก์ชันอย่าง pattern matching, validation และ parsing

เพื่อช่วยสร้างความเข้าใจ ด้านล่างคือตัวอย่างบางส่วนของ regex pattern ในบทนี้ใช้ [Python regex syntax](https://docs.python.org/3/library/re.html) regex มีหลายรสชาติ โดยมีความแตกต่างเล็กน้อยระหว่างกัน โดยเฉพาะในฟังก์ชันที่ซับซ้อนกว่า สามารถใช้ online regex tester เช่น [regex101](https://regex101.com/) เพื่อพัฒนาและ debug regular expression

- `abc` --- จับคู่กับ "abc" ตรงตัว
- `missing|semester` --- จับคู่กับ string "missing" หรือ string "semester"
- `\d{4}-\d{2}-\d{2}` --- จับคู่กับวันที่ในรูปแบบ YYYY-MM-DD เช่น "2026-01-14" นอกเหนือจากการตรวจสอบว่า string ประกอบด้วยตัวเลขสี่หลัก, ขีดกลาง, ตัวเลขสองหลัก, ขีดกลาง, และตัวเลขสองหลักแล้ว pattern นี้ไม่ได้ validate วันที่ ดังนั้น "2026-01-99" ก็ตรงกับ regex pattern นี้เช่นกัน
- `.+@.+` --- จับคู่กับ email address ซึ่งเป็น string ที่มีข้อความบางส่วน ตามด้วย "@" แล้วตามด้วยข้อความอีก pattern นี้ทำ validation พื้นฐานที่สุดเท่านั้น และจับคู่กับ string อย่าง "nonsense@@@email" ได้ด้วย regex ที่จับคู่กับ email address โดยไม่มี false positive หรือ false negative [มีอยู่จริง](https://pdw.ex-parrot.com/Mail-RFC822-Address.html) แต่ไม่ practical

## Regex syntax

สามารถหาคู่มือ regex syntax ที่ครบถ้วนได้ใน[เอกสารนี้](https://docs.python.org/3/library/re.html#regular-expression-syntax) (หรือแหล่งข้อมูลอื่น ๆ อีกมากที่มีออนไลน์) ต่อไปนี้เป็น building block พื้นฐาน:

- `abc` จับคู่กับ string ตรงตัว เมื่อตัวอักษรไม่มีความหมายพิเศษ (ในตัวอย่างนี้คือ "abc")
- `.` จับคู่กับตัวอักษรใดก็ได้หนึ่งตัว
- `[abc]` จับคู่กับตัวอักษรหนึ่งตัวที่อยู่ในวงเล็บเหลี่ยม (ในตัวอย่างนี้คือ "a", "b" หรือ "c")
- `[^abc]` จับคู่กับตัวอักษรหนึ่งตัวที่ไม่อยู่ในวงเล็บเหลี่ยม (เช่น "d")
- `[a-f]` จับคู่กับตัวอักษรหนึ่งตัวที่อยู่ในช่วงที่ระบุในวงเล็บเหลี่ยม (เช่น "c" แต่ไม่ใช่ "q")
- `a|b` จับคู่กับ pattern ใดก็ได้ (เช่น "a" หรือ "b")
- `\d` จับคู่กับตัวเลขใดก็ได้ (เช่น "3")
- `\w` จับคู่กับ word character ใดก็ได้ (เช่น "x")
- `\b` จับคู่กับ word _boundary_ ใดก็ได้ (เช่น ใน string "missing semester" จะจับคู่ตรงก่อน "m", หลัง "g", ก่อน "s" และหลัง "r")
- `(...)` จับคู่กับ group ของ pattern
- `...?` จับคู่กับ pattern ศูนย์หรือหนึ่งครั้ง เช่น `words?` จะจับคู่กับ "word" หรือ "words"
- `...*` จับคู่กับ pattern จำนวนเท่าใดก็ได้ เช่น `.*` จะจับคู่กับตัวอักษรใดก็ได้จำนวนเท่าใดก็ได้
- `...+` จับคู่กับ pattern หนึ่งครั้งขึ้นไป เช่น `\d+` จะจับคู่กับตัวเลขที่ไม่เป็นศูนย์จำนวนหลัก
- `...{N}` จับคู่กับ pattern ตรง N ครั้ง เช่น `\d{4}` สำหรับตัวเลข 4 หลัก
- `\.` จับคู่กับ "." ตรงตัว
- `\\` จับคู่กับ "\\" ตรงตัว
- `^` จับคู่กับจุดเริ่มต้นของบรรทัด
- `$` จับคู่กับจุดสิ้นสุดของบรรทัด

## Capture groups and references

ถ้าใช้ regex group `(...)` จะสามารถอ้างอิงส่วนย่อยของสิ่งที่จับคู่ได้ สำหรับการ extract ข้อมูลหรือ search-and-replace ตัวอย่างเช่น หากต้องการ extract เฉพาะเดือนจากวันที่รูปแบบ YYYY-MM-DD สามารถใช้โค้ด Python ต่อไปนี้:

```python
>>> import re
>>> re.match(r"\d{4}-(\d{2})-\d{2}", "2026-01-14").group(1)
'01'
```

ใน text editor สามารถใช้ reference capture group ใน replace pattern ได้ syntax อาจแตกต่างกันระหว่าง IDE ตัวอย่างเช่น ใน VS Code สามารถใช้ตัวแปรอย่าง `$1`, `$2` เป็นต้น และใน Vim สามารถใช้ `\1`, `\2` เป็นต้น เพื่ออ้างอิง group

## Limitations

[Regular language](https://en.wikipedia.org/wiki/Regular_language) มีพลังแต่มีข้อจำกัด มี class ของ string ที่ไม่สามารถแสดงเป็น standard regex ได้ (เช่น [ไม่สามารถ](https://en.wikipedia.org/wiki/Pumping_lemma_for_regular_languages) เขียน regular expression ที่จับคู่กับเซตของ string {a^n b^n \| n &ge; 0} ซึ่งเป็นเซตของ string ที่มี "a" จำนวนหนึ่งตามด้วย "b" จำนวนเท่ากัน ในทาง practical มากขึ้น ภาษาอย่าง HTML ไม่ใช่ regular language) ในทางปฏิบัติ regex engine สมัยใหม่รองรับฟีเจอร์อย่าง lookahead และ backreference ที่ขยายความสามารถเกินกว่า regular language และมีประโยชน์อย่างมากในทางปฏิบัติ แต่สิ่งสำคัญคือต้องรู้ว่ามันยังมีข้อจำกัดใน expressive power สำหรับภาษาที่ซับซ้อนกว่า อาจต้องใช้ parser ที่มีความสามารถมากกว่า (ตัวอย่างหนึ่ง ดู [pyparsing](https://github.com/pyparsing/pyparsing) ซึ่งเป็น [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar) parser)

## Learning regex

แนะนำให้เรียนรู้พื้นฐาน (สิ่งที่ครอบคลุมในบทนี้) แล้วค่อยดู regex reference เมื่อต้องการ แทนที่จะท่องจำภาษาทั้งหมด

เครื่องมือ conversational AI สามารถช่วยสร้าง regex pattern ได้อย่างมีประสิทธิภาพ ตัวอย่างเช่น ลอง prompt LLM ที่ชอบด้วย query ต่อไปนี้:

```
Write a Python-style regex pattern that matches the requested path from log lines from Nginx. Here is an example log line:

169.254.1.1 - - [09/Jan/2026:21:28:51 +0000] "GET /feed.xml HTTP/2.0" 200 2995 "-" "python-requests/2.32.3"
```

# Exercises

1. ตั้งค่า formatter, linter และ pre-commit hook สำหรับโปรเจกต์ที่กำลังทำอยู่ ถ้ามี error เยอะ: autoformatting ควรจัดการ format error ได้ สำหรับ linter error ลองใช้ [AI agent](/2026/agentic-coding/) เพื่อ fix linter error ทั้งหมด ต้องให้แน่ใจว่า AI agent สามารถรัน linter และดูผลลัพธ์ได้ เพื่อให้มันทำงานแบบ iterative loop เพื่อ fix ปัญหาทั้งหมด ตรวจสอบผลลัพธ์อย่างละเอียดเพื่อให้แน่ใจว่า AI ไม่ทำให้โค้ดพัง
2. เรียนรู้ testing library สำหรับภาษาที่รู้จักและเขียน unit test สำหรับโปรเจกต์ที่กำลังทำอยู่ รันเครื่องมือ code coverage สร้าง coverage report ในรูปแบบ HTML และดูผลลัพธ์ หาบรรทัดที่ถูก cover ได้ไหม code coverage น่าจะต่ำมาก ลองเขียน test เพิ่มเองเพื่อปรับปรุง ลองใช้ [AI agent](/2026/agentic-coding/) เพื่อปรับปรุง coverage โดยต้องให้ coding agent สามารถรัน test พร้อม coverage และสร้าง coverage report แบบทีละบรรทัดได้ เพื่อให้มันรู้ว่าควรเน้นตรงไหน test ที่ AI สร้างมาดีจริงไหม
3. ตั้งค่า continuous integration ให้รันทุกครั้งที่ push สำหรับโปรเจกต์ที่กำลังทำอยู่ ให้ CI รัน formatting, linting และ test ทำให้โค้ดพังตั้งใจ (เช่น สร้าง linter violation) แล้วตรวจสอบว่า CI จับได้
4. ลองเขียน [regex pattern](#regular-expressions) และใช้เครื่องมือ command-line `grep` [](/2026/course-shell/) เพื่อหาการใช้ `subprocess.Popen(..., shell=True)` ในโค้ด จากนั้นลอง "break" regex pattern ดู [semgrep](#linting) ยังจับคู่กับโค้ดอันตรายที่ทำให้ grep invocation ล้มเหลวได้อยู่ไหม
5. ฝึกใช้ regex search-and-replace ใน IDE หรือ text editor โดยเปลี่ยน `-` [Markdown bullet marker](https://spec.commonmark.org/0.31.2/#bullet-list-marker) เป็น `*` bullet marker ใน[lecture note เหล่านี้](https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/code-quality.md) สังเกตว่าการแทนที่ตัวอักษร "-" ทั้งหมดในไฟล์จะไม่ถูกต้อง เพราะมีการใช้ตัวอักษรนี้ในบริบทอื่นที่ไม่ใช่ bullet marker
6. เขียน regex เพื่อ capture ชื่อจากโครงสร้าง JSON ในรูปแบบ `{"name": "Alyssa P. Hacker", "college": "MIT"}` (เช่น `Alyssa P. Hacker` ในตัวอย่างนี้) คำใบ้: ในความพยายามแรก อาจจะเขียน regex ที่ extract ได้ `Alyssa P. Hacker", "college": "MIT` ลองอ่านเรื่อง greedy quantifier ใน [Python regex docs](https://docs.python.org/3/library/re.html) เพื่อหาวิธีแก้
    1. ทำให้ regex pattern ทำงานได้แม้ในกรณีที่ชื่อมีตัวอักษร `"` อยู่ด้วย (เครื่องหมาย double quote สามารถ escape ได้ใน JSON ด้วย `\"`)
    2. เรา**ไม่**แนะนำให้ใช้ regular expression สำหรับปัญหา parsing ที่ซับซ้อนในทางปฏิบัติ หาวิธีใช้ JSON parser ของภาษาโปรแกรมที่ใช้อยู่สำหรับงานนี้ เขียนโปรแกรม command-line ที่รับ input จาก stdin เป็นโครงสร้าง JSON ในรูปแบบที่อธิบายไว้ข้างต้น แล้ว output ชื่อไปที่ stdout ควรใช้แค่ไม่กี่บรรทัดของโค้ด ใน Python สามารถทำได้ในบรรทัดเดียวนอกเหนือจาก `import json`
