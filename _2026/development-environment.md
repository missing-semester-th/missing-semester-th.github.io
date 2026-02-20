---
layout: lecture
title: "Development Environment and Tools"
description: >
  เรียนรู้เกี่ยวกับ IDE, Vim, language server และเครื่องมือพัฒนาซอฟต์แวร์ที่ขับเคลื่อนด้วย AI
thumbnail: /static/assets/thumbnails/2026/lec3.png
date: 2026-01-14
ready: true
video:
  aspect: 56.25
  id: QnM1nVzrkx8
---

_Development environment_ คือชุดเครื่องมือสำหรับพัฒนาซอฟต์แวร์ หัวใจของ development environment คือความสามารถในการแก้ไขข้อความ พร้อมด้วย feature ต่าง ๆ อย่างเช่น syntax highlighting, type checking, code formatting และ autocomplete _Integrated development environment_ (IDE) อย่าง [VS Code][vs-code] รวมความสามารถทั้งหมดนี้ไว้ในแอปพลิเคชันเดียว ส่วน workflow แบบ terminal-based จะรวมเครื่องมือต่าง ๆ เข้าด้วยกัน เช่น [tmux](https://github.com/tmux/tmux) (terminal multiplexer), [Vim](https://www.vim.org/) (text editor), [Zsh](https://www.zsh.org/) (shell) และ command-line tool เฉพาะภาษา เช่น [Ruff](https://docs.astral.sh/ruff/) (Python linter และ code formatter) และ [Mypy](https://mypy-lang.org/) (Python type checker)

IDE กับ workflow แบบ terminal-based ต่างก็มีจุดเด่นและจุดด้อยของตัวเอง ตัวอย่างเช่น IDE แบบ graphical อาจเรียนรู้ได้ง่ายกว่า และ IDE ในปัจจุบันโดยทั่วไปจะมี AI integration ที่พร้อมใช้งานได้ดีกว่า เช่น AI autocomplete ในทางกลับกัน workflow แบบ terminal-based นั้นเบาและอาจเป็นทางเลือกเดียวในสภาพแวดล้อมที่ไม่มี GUI หรือไม่สามารถติดตั้งซอฟต์แวร์ได้ แนะนำให้ทำความคุ้นเคยกับทั้งสองแบบในระดับพื้นฐาน และพัฒนาความเชี่ยวชาญอย่างน้อยหนึ่งแบบ หากยังไม่มี IDE ที่ถนัด แนะนำให้เริ่มจาก [VS Code][vs-code]

ในบทนี้จะครอบคลุม:

- [การแก้ไขข้อความและ Vim](#text-editing-and-vim)
- [Code intelligence และ language server](#code-intelligence-and-language-servers)
- [การพัฒนาซอฟต์แวร์ด้วย AI](#ai-powered-development)
- [Extension และ feature อื่น ๆ ของ IDE](#extensions-and-other-ide-functionality)

[vs-code]: https://code.visualstudio.com/

# Text editing and Vim

เวลาเขียนโปรแกรม เราใช้เวลาส่วนใหญ่ไปกับการนำทางไปตามส่วนต่าง ๆ ของโค้ด อ่านโค้ดเป็นท่อน ๆ และแก้ไขโค้ด มากกว่าจะนั่งพิมพ์ข้อความยาว ๆ หรืออ่านไฟล์ตั้งแต่ต้นจนจบ [Vim] เป็น text editor ที่ออกแบบมาให้เหมาะกับรูปแบบการทำงานแบบนี้โดยเฉพาะ

**ปรัชญาของ Vim** Vim มีแนวคิดที่สวยงามเป็นรากฐาน: interface ของมันนั้นเป็นภาษาโปรแกรมในตัวเอง ออกแบบมาเพื่อการนำทางและแก้ไขข้อความ keystroke (ที่ตั้งชื่อให้จำง่าย) คือ command และ command เหล่านี้สามารถ compose เข้าด้วยกันได้ Vim หลีกเลี่ยงการใช้เมาส์เพราะมันช้าเกินไป Vim แม้แต่ยังหลีกเลี่ยงการใช้ปุ่มลูกศรเพราะต้องเคลื่อนมือมากเกินไป ผลลัพธ์คือ editor ที่รู้สึกเหมือน brain-computer interface และทำงานได้เร็วทันความคิด

**การรองรับ Vim ในซอฟต์แวร์อื่น** ไม่จำเป็นต้องใช้ [Vim] ตัวจริงเพื่อจะได้ประโยชน์จากแนวคิดหลักของมัน โปรแกรมจำนวนมากที่เกี่ยวข้องกับการแก้ไขข้อความรองรับ "Vim mode" ไม่ว่าจะเป็น feature ที่มีมาในตัวหรือเป็น plugin ตัวอย่างเช่น VS Code มี plugin [VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim), Zsh มี[การรองรับ Vim emulation ในตัว](https://zsh.sourceforge.io/Guide/zshguide04.html) และแม้แต่ Claude Code ก็มี[การรองรับ Vim editor mode ในตัว](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode) เครื่องมือไหนก็ตามที่ใช้อยู่ที่เกี่ยวข้องกับการแก้ไขข้อความ มีโอกาสสูงที่จะรองรับ Vim mode ในรูปแบบใดรูปแบบหนึ่ง

## Modal editing

Vim เป็น _modal editor_: มันมี mode การทำงานที่แตกต่างกันสำหรับงานประเภทต่าง ๆ

- **Normal**: สำหรับเลื่อนไปมาในไฟล์และทำการแก้ไข
- **Insert**: สำหรับแทรกข้อความ
- **Replace**: สำหรับแทนที่ข้อความ
- **Visual** (plain, line หรือ block): สำหรับเลือกบล็อกข้อความ
- **Command-line**: สำหรับรัน command

keystroke มีความหมายต่างกันใน mode ที่ต่างกัน ตัวอย่างเช่น ตัวอักษร `x` ใน Insert mode จะแทรกตัวอักษร "x" ธรรมดา แต่ใน Normal mode จะลบตัวอักษรที่ cursor อยู่ และใน Visual mode จะลบส่วนที่เลือกไว้

ในการตั้งค่าเริ่มต้น Vim จะแสดง mode ปัจจุบันที่มุมล่างซ้าย mode เริ่มต้นคือ Normal mode โดยทั่วไปจะใช้เวลาส่วนใหญ่สลับไปมาระหว่าง Normal mode กับ Insert mode

การเปลี่ยน mode ทำได้โดยกด `<ESC>` (ปุ่ม escape) เพื่อกลับไป Normal mode จากทุก mode จาก Normal mode เข้า Insert mode ด้วย `i`, Replace mode ด้วย `R`, Visual mode ด้วย `v`, Visual Line mode ด้วย `V`, Visual Block mode ด้วย `<C-v>` (Ctrl-V ซึ่งบางทีเขียนว่า `^V`) และ Command-line mode ด้วย `:`

เมื่อใช้ Vim จะต้องกดปุ่ม `<ESC>` บ่อยมาก: ลอง remap Caps Lock ให้เป็น Escape ([วิธีทำบน macOS](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)) หรือสร้าง [mapping ทดแทน](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings) สำหรับ `<ESC>` ด้วย key sequence ง่าย ๆ

## พื้นฐาน: การแทรกข้อความ

จาก Normal mode กด `i` เพื่อเข้า Insert mode ตอนนี้ Vim จะทำงานเหมือน text editor ทั่วไป จนกว่าจะกด `<ESC>` เพื่อกลับไป Normal mode ความรู้นี้รวมกับพื้นฐานที่อธิบายไปข้างต้นก็เพียงพอสำหรับการเริ่มแก้ไขไฟล์ด้วย Vim แล้ว (ถึงแม้จะยังไม่มีประสิทธิภาพมากนัก หากใช้เวลาทั้งหมดแก้ไขจาก Insert mode)

## Interface ของ Vim คือภาษาโปรแกรม

Interface ของ Vim คือภาษาโปรแกรม keystroke (ที่ตั้งชื่อให้จำง่าย) คือ command และ command เหล่านี้สามารถ _compose_ เข้าด้วยกันได้ สิ่งนี้ทำให้การเคลื่อนที่และแก้ไขมีประสิทธิภาพ โดยเฉพาะเมื่อ command กลายเป็น muscle memory เหมือนกับที่การพิมพ์จะมีประสิทธิภาพสูงมากเมื่อคุ้นเคยกับ keyboard layout แล้ว

### Movement

ควรอยู่ใน Normal mode เป็นส่วนใหญ่ โดยใช้ movement command เพื่อนำทางในไฟล์ movement ใน Vim ยังเรียกว่า "noun" เพราะมันอ้างถึงชิ้นส่วนของข้อความ

- การเคลื่อนที่พื้นฐาน: `hjkl` (ซ้าย, ลง, ขึ้น, ขวา)
- คำ: `w` (คำถัดไป), `b` (ต้นคำ), `e` (ท้ายคำ)
- บรรทัด: `0` (ต้นบรรทัด), `^` (ตัวอักษรแรกที่ไม่ใช่ช่องว่าง), `$` (ท้ายบรรทัด)
- หน้าจอ: `H` (บนสุดของหน้าจอ), `M` (กลางหน้าจอ), `L` (ล่างสุดของหน้าจอ)
- เลื่อนหน้าจอ: `Ctrl-u` (ขึ้น), `Ctrl-d` (ลง)
- ไฟล์: `gg` (ต้นไฟล์), `G` (ท้ายไฟล์)
- เลขบรรทัด: `:{number}<CR>` หรือ `{number}G` (บรรทัดที่ {number})
    - `<CR>` หมายถึงปุ่ม carriage return / enter
- อื่น ๆ: `%` (ตัวที่จับคู่กัน เช่น วงเล็บหรือปีกกา)
- ค้นหาตัวอักษร: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - ค้นหา/ไปยังตัวอักษร {character} ไปข้างหน้า/ข้างหลังบนบรรทัดปัจจุบัน
    - `,` / `;` สำหรับเลื่อนไปยังผลลัพธ์ที่พบ
- ค้นหา: `/{regex}`, `n` / `N` สำหรับเลื่อนไปยังผลลัพธ์ที่พบ

### Selection

Visual mode ต่าง ๆ:

- Visual: `v`
- Visual Line: `V`
- Visual Block: `Ctrl-v`

ใช้ movement key เพื่อสร้างการเลือกได้

### Edits

ทุกอย่างที่เคยทำด้วยเมาส์ ตอนนี้ทำด้วย keyboard โดยใช้ editing command ที่ compose กับ movement command ได้ ตรงนี้แหละที่ interface ของ Vim เริ่มดูเหมือนภาษาโปรแกรม editing command ของ Vim ยังเรียกว่า "verb" เพราะ verb กระทำต่อ noun

- `i` เข้า Insert mode
    - แต่สำหรับการจัดการ/ลบข้อความ ควรใช้อะไรที่ดีกว่า backspace
- `o` / `O` แทรกบรรทัดใหม่ข้างล่าง / ข้างบน
- `d{motion}` ลบตาม {motion}
    - เช่น `dw` คือลบคำ, `d$` คือลบถึงท้ายบรรทัด, `d0` คือลบถึงต้นบรรทัด
- `c{motion}` เปลี่ยนตาม {motion}
    - เช่น `cw` คือเปลี่ยนคำ
    - เหมือน `d{motion}` ตามด้วย `i`
- `x` ลบตัวอักษร (เทียบเท่ากับ `dl`)
- `s` แทนที่ตัวอักษร (เทียบเท่ากับ `cl`)
- Visual mode + การจัดการ
    - เลือกข้อความ แล้วกด `d` เพื่อลบ หรือ `c` เพื่อเปลี่ยน
- `u` เพื่อ undo, `<C-r>` เพื่อ redo
- `y` เพื่อ copy / "yank" (command อื่นบางตัว เช่น `d` ก็ copy ด้วย)
- `p` เพื่อ paste
- ยังมีอีกมากที่ต้องเรียนรู้: ตัวอย่างเช่น `~` สลับตัวพิมพ์เล็ก-ใหญ่ของตัวอักษร และ `J` เชื่อมบรรทัดเข้าด้วยกัน

### Counts

สามารถรวม noun กับ verb พร้อมกับ count ได้ ซึ่งจะทำ action นั้นซ้ำตามจำนวนที่กำหนด

- `3w` เลื่อนไปข้างหน้า 3 คำ
- `5j` เลื่อนลง 5 บรรทัด
- `7dw` ลบ 7 คำ

### Modifiers

สามารถใช้ modifier เพื่อเปลี่ยนความหมายของ noun ได้ modifier บางตัว ได้แก่ `i` ซึ่งหมายถึง "inner" หรือ "inside" และ `a` ซึ่งหมายถึง "around"

- `ci(` เปลี่ยนเนื้อหาภายในวงเล็บคู่ปัจจุบัน
- `ci[` เปลี่ยนเนื้อหาภายในวงเล็บเหลี่ยมคู่ปัจจุบัน
- `da'` ลบ string ที่อยู่ใน single quote รวมถึง single quote ที่ล้อมรอบด้วย

## รวมทุกอย่างเข้าด้วยกัน

นี่คือ implementation ของ [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) ที่มีบั๊ก:

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20)
```

เราใช้ลำดับ command ต่อไปนี้เพื่อแก้ปัญหา โดยเริ่มจาก Normal mode:

- Main ไม่เคยถูกเรียก
    - `G` เพื่อกระโดดไปท้ายไฟล์
    - `o` เพื่อ **o**pen บรรทัดใหม่ข้างล่าง
    - พิมพ์ `if __name__ == "__main__": main()`
        - หาก editor มีการรองรับภาษา Python อาจจะ auto-indent ให้ใน Insert mode
    - `<ESC>` เพื่อกลับไป Normal mode
- เริ่มที่ 0 แทนที่จะเป็น 1
    - `/` ตามด้วย `range` แล้วกด `<CR>` เพื่อค้นหา "range"
    - `ww` เพื่อเลื่อนไปข้างหน้าสองคำ (**w**ord) (จะใช้ `2w` ก็ได้ แต่ในทางปฏิบัติ สำหรับจำนวนน้อย ๆ มักจะกดปุ่มซ้ำแทนการใช้ count)
    - `i` เพื่อเข้า **i**nsert mode แล้วเพิ่ม `1,`
    - `<ESC>` เพื่อกลับไป Normal mode
    - `e` เพื่อกระโดดไปท้ายคำถัดไป (**e**nd)
    - `a` เพื่อเริ่ม **a**ppend ข้อความ แล้วเพิ่ม `+ 1`
    - `<ESC>` เพื่อกลับไป Normal mode
- แสดง "fizz" สำหรับตัวเลขที่หารด้วย 5 ลงตัว
    - `:6<CR>` เพื่อไปที่บรรทัดที่ 6
    - `ci"` เพื่อ **c**hange **i**nside เครื่องหมาย '**"**' แล้วเปลี่ยนเป็น `"buzz"`
    - `<ESC>` เพื่อกลับไป Normal mode

## การเรียนรู้ Vim

วิธีที่ดีที่สุดในการเรียน Vim คือเรียนพื้นฐาน (ที่ครอบคลุมไปแล้วข้างต้น) จากนั้นก็เปิดใช้ Vim mode ในซอฟต์แวร์ทุกตัวแล้วเริ่มใช้งานจริง หลีกเลี่ยงการใช้เมาส์หรือปุ่มลูกศร ใน editor บางตัวสามารถ unbind ปุ่มลูกศรเพื่อบังคับตัวเองให้สร้างนิสัยที่ดีได้

### แหล่งเรียนรู้เพิ่มเติม

- [บทเรียน Vim](/2020/editors/) จาก class รุ่นก่อนหน้า --- เราได้ครอบคลุม Vim ไว้อย่างละเอียดกว่าในนั้น
- `vimtutor` เป็น tutorial ที่ติดตั้งมาพร้อมกับ Vim --- หากติดตั้ง Vim ไว้แล้ว สามารถรัน `vimtutor` จาก shell ได้เลย
- [Vim Adventures](https://vim-adventures.com/) เป็นเกมสำหรับเรียน Vim
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2019/) มี Vim tip ต่าง ๆ
- [VimGolf](https://www.vimgolf.com/) คือ [code golf](https://en.wikipedia.org/wiki/Code_golf) แต่ภาษาโปรแกรมที่ใช้คือ UI ของ Vim
- [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
- [Vim Screencasts](http://vimcasts.org/)
- [Practical Vim](https://pragprog.com/titles/dnvim2/) (หนังสือ)

[Vim]: https://www.vim.org/

# Code intelligence และ language server

IDE โดยทั่วไปจะให้การสนับสนุนเฉพาะภาษาที่ต้องอาศัยความเข้าใจเชิง semantic ของโค้ดผ่าน IDE extension ที่เชื่อมต่อกับ _language server_ ซึ่ง implement [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) ตัวอย่างเช่น [Python extension สำหรับ VS Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python) พึ่งพา [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance) และ [Go extension สำหรับ VS Code](https://marketplace.visualstudio.com/items?itemName=golang.go) พึ่งพา [gopls](https://go.dev/gopls/) ซึ่งเป็น first-party การติดตั้ง extension และ language server สำหรับภาษาที่ใช้งาน จะช่วยเปิดใช้ feature เฉพาะภาษาจำนวนมากใน IDE เช่น:

- **Code completion** autocomplete และ autosuggest ที่ดีขึ้น เช่น สามารถเห็น field และ method ของ object หลังจากพิมพ์ `object.`
- **Inline documentation** เห็นเอกสารประกอบเมื่อ hover เมาส์และจาก autosuggest
- **Jump-to-definition** กระโดดจากจุดที่ใช้งานไปยังจุดที่นิยาม เช่น สามารถไปจาก field reference `object.field` ไปยังจุดที่นิยาม field นั้น
- **Find references** ตรงกันข้ามกับข้อข้างต้น ค้นหาทุกจุดที่มีการอ้างอิงถึง item เช่น field หรือ type
- **ช่วยจัดการ import** จัดระเบียบ import, ลบ import ที่ไม่ได้ใช้, แจ้งเตือน import ที่ขาดหายไป
- **Code quality** เครื่องมือเหล่านี้ใช้แยกเดี่ยวก็ได้ แต่ functionality นี้มักถูกจัดให้โดย language server ด้วยเช่นกัน code formatting จัดย่อหน้าและจัดรูปแบบโค้ดอัตโนมัติ ส่วน type checker และ linter ช่วยหาข้อผิดพลาดในโค้ดขณะพิมพ์ เราจะครอบคลุม functionality กลุ่มนี้อย่างละเอียดใน[บทเรียนเรื่อง code quality](/2026/code-quality/)

## การตั้งค่า language server

สำหรับบางภาษา สิ่งที่ต้องทำคือติดตั้ง extension และ language server เท่านั้นก็พร้อมใช้งาน แต่สำหรับภาษาอื่น ๆ เพื่อให้ได้ประโยชน์สูงสุดจาก language server ต้องบอก IDE เกี่ยวกับ environment ของเรา ตัวอย่างเช่น การชี้ VS Code ไปยัง [Python environment](https://code.visualstudio.com/docs/python/environments) จะทำให้ language server เห็น package ที่ติดตั้งไว้ เรื่อง environment จะครอบคลุมอย่างละเอียดใน[บทเรียนเรื่อง packaging และ shipping code](/2026/shipping-code/)

ขึ้นอยู่กับภาษา อาจมี setting บางอย่างที่ตั้งค่าได้สำหรับ language server ตัวอย่างเช่น การใช้ Python support ใน VS Code สามารถปิด static type checking สำหรับ project ที่ไม่ได้ใช้ optional type annotation ของ Python

# การพัฒนาซอฟต์แวร์ด้วย AI

ตั้งแต่การเปิดตัว [GitHub Copilot][github-copilot] ที่ใช้ [Codex model](https://openai.com/index/openai-codex/) ของ OpenAI ในกลางปี 2021 [LLM](https://en.wikipedia.org/wiki/Large_language_model) ได้ถูกนำมาใช้อย่างแพร่หลายในวงการวิศวกรรมซอฟต์แวร์ ปัจจุบันมีรูปแบบการใช้งานหลัก 3 แบบ: autocomplete, inline chat และ coding agent

[github-copilot]: https://github.com/features/copilot/ai-code-editor

## Autocomplete

AI-powered autocomplete มีรูปแบบเดียวกับ autocomplete แบบดั้งเดิมใน IDE โดยแนะนำการเติมข้อความที่ตำแหน่ง cursor ขณะพิมพ์ บางครั้งใช้เป็น feature แบบ passive ที่ "ทำงานเอง" ได้เลย นอกเหนือจากนั้น AI autocomplete โดยทั่วไปจะถูก [prompt](https://en.wikipedia.org/wiki/Prompt_engineering) ด้วย code comment

ตัวอย่างเช่น ลองเขียน script เพื่อดาวน์โหลดเนื้อหาของบทเรียนนี้และดึง link ทั้งหมดออกมา เราเริ่มด้วย:

```python
import requests

def download_contents(url: str) -> str:
```

model จะ autocomplete ส่วนเนื้อหาของ function:

```python
    response = requests.get(url)
    return response.text
```

เราสามารถนำทาง completion เพิ่มเติมได้ด้วย comment ตัวอย่างเช่น ถ้าเริ่มเขียน function สำหรับดึง Markdown link ทั้งหมด แต่ชื่อ function ไม่ได้อธิบายความหมายชัดเจนนัก:

```python
def extract(contents: str) -> list[str]:
```

model จะ autocomplete ออกมาประมาณนี้:

```python
    lines = contents.splitlines()
    return [line for line in lines if line.strip()]
```

เราสามารถนำทาง completion ด้วย code comment:

```python
def extract(content: str) -> list[str]:
    # extract all Markdown links from the content
```

คราวนี้ model ให้ completion ที่ดีขึ้น:

```python
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)
```

ตรงนี้จะเห็นข้อเสียอย่างหนึ่งของ AI coding tool ตัวนี้: มันสามารถให้ completion ที่ตำแหน่ง cursor เท่านั้น ในกรณีนี้ จะเป็น practice ที่ดีกว่าถ้าวาง `import re` ไว้ที่ระดับ module แทนที่จะอยู่ภายใน function

ตัวอย่างข้างต้นใช้ function ที่ตั้งชื่อไม่ดีเพื่อสาธิตว่า code completion สามารถถูกนำทางด้วย comment ได้อย่างไร ในทางปฏิบัติ ควรเขียนโค้ดโดยตั้งชื่อ function ให้อธิบายความหมายมากกว่านี้ เช่น `extract_links` และควรเขียน docstring ด้วย (จากข้อมูลนี้ model ควรจะสร้าง completion ที่คล้ายกับข้างต้นได้)

เพื่อวัตถุประสงค์ในการสาธิต เราเขียน script ให้สมบูรณ์:

```python
print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

## Inline chat

Inline chat ให้เลือกบรรทัดหรือบล็อกของโค้ด แล้ว prompt AI model โดยตรงเพื่อเสนอการแก้ไข ในรูปแบบการทำงานนี้ model สามารถแก้ไขโค้ดที่มีอยู่แล้วได้ (ซึ่งต่างจาก autocomplete ที่เติมโค้ดหลัง cursor เท่านั้น)

ต่อจากตัวอย่างข้างต้น สมมติว่าตัดสินใจไม่ใช้ library `requests` ของ third-party เราสามารถเลือกโค้ด 3 บรรทัดที่เกี่ยวข้อง เรียกใช้ inline chat แล้วพิมพ์ประมาณว่า:

```
use built-in libraries instead
```

model เสนอ:

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')
```

## Coding agent

Coding agent จะถูกครอบคลุมอย่างละเอียดในบทเรียน [Agentic Coding](/2026/agentic-coding/)

## ซอฟต์แวร์ที่แนะนำ

AI IDE ที่เป็นที่นิยม ได้แก่ [VS Code][vs-code] พร้อม extension [GitHub Copilot][github-copilot] และ [Cursor](https://cursor.com/) GitHub Copilot ปัจจุบัน[ให้ใช้ฟรีสำหรับนักศึกษา](https://github.com/education/students) อาจารย์ และผู้ดูแล open source project ที่เป็นที่นิยม นี่เป็นพื้นที่ที่พัฒนาอย่างรวดเร็ว ผลิตภัณฑ์ชั้นนำหลายตัวมี functionality ที่ใกล้เคียงกัน

# Extension และ feature อื่น ๆ ของ IDE

IDE เป็นเครื่องมือที่ทรงพลัง และยิ่งทรงพลังขึ้นด้วย _extension_ เราไม่สามารถครอบคลุม feature ทั้งหมดเหล่านี้ได้ในบทเดียว แต่จะชี้แนะไปยัง extension ยอดนิยมบางตัว แนะนำให้สำรวจพื้นที่นี้ด้วยตัวเอง มีรายการ IDE extension ยอดนิยมออนไลน์มากมาย เช่น [Vim Awesome](https://vimawesome.com/) สำหรับ Vim plugin และ [VS Code extension เรียงตามความนิยม](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Installs)

- [Development container](https://containers.dev/): รองรับโดย IDE ยอดนิยม (เช่น [รองรับโดย VS Code](https://code.visualstudio.com/docs/devcontainers/containers)) dev container ให้ใช้ container เพื่อรันเครื่องมือพัฒนา สิ่งนี้มีประโยชน์ในเรื่อง portability หรือ isolation [บทเรียนเรื่อง packaging และ shipping code](/2026/shipping-code/) จะครอบคลุมเรื่อง container อย่างละเอียดยิ่งขึ้น
- Remote development: พัฒนาบนเครื่อง remote ผ่าน SSH (เช่น ด้วย [Remote SSH plugin สำหรับ VS Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)) สิ่งนี้มีประโยชน์ ตัวอย่างเช่น เมื่อต้องการพัฒนาและรันโค้ดบนเครื่อง GPU แรง ๆ บน cloud
- Collaborative editing: แก้ไขไฟล์เดียวกันแบบ Google Docs (เช่น ด้วย [Live Share plugin สำหรับ VS Code](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare))

# แบบฝึกหัด

1. เปิดใช้ Vim mode ในซอฟต์แวร์ทุกตัวที่ใช้อยู่ที่รองรับ เช่น editor และ shell แล้วใช้ Vim mode สำหรับการแก้ไขข้อความทุกอย่างตลอดเดือนหน้า เมื่อไหร่ก็ตามที่รู้สึกว่าทำอะไรไม่คล่อง หรือคิดว่า "น่าจะมีวิธีที่ดีกว่านี้" ให้ลอง Google ดู มักจะมีวิธีที่ดีกว่าจริง ๆ
2. ทำ challenge จาก [VimGolf](https://www.vimgolf.com/) ให้สำเร็จ
3. ตั้งค่า IDE extension และ language server สำหรับ project ที่กำลังทำอยู่ ตรวจสอบว่า functionality ที่คาดหวังทั้งหมด เช่น jump-to-definition สำหรับ library dependency ทำงานได้ตามที่คาดไว้ หากไม่มีโค้ดที่จะใช้สำหรับแบบฝึกหัดนี้ สามารถใช้ open-source project จาก GitHub ได้ (เช่น [ตัวนี้](https://github.com/spf13/cobra))
4. สำรวจรายการ IDE extension แล้วติดตั้งตัวที่ดูเป็นประโยชน์
