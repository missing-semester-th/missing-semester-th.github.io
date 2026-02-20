---
layout: lecture
title: "Version Control and Git"
description: >
  เรียนรู้ data model ของ Git และวิธีใช้ Git สำหรับ version control และการทำงานร่วมกัน
thumbnail: /static/assets/thumbnails/2026/lec5.png
date: 2026-01-16
ready: true
video:
  aspect: 56.25
  id: 9K8lB61dl3Y
---

Version control system (VCS) คือเครื่องมือที่ใช้ติดตามการเปลี่ยนแปลงของ source code
(หรือชุดของ file และ folder อื่น ๆ) ตามชื่อเลย เครื่องมือเหล่านี้ช่วยเก็บประวัติการเปลี่ยนแปลง
และยังช่วยอำนวยความสะดวกในการทำงานร่วมกันอีกด้วย ในเชิงตรรกะแล้ว VCS จะติดตาม
การเปลี่ยนแปลงของ folder และเนื้อหาข้างในเป็นชุดของ _snapshot_ โดยแต่ละ snapshot
จะเก็บสถานะทั้งหมดของ file/folder ภายใต้ top-level directory นอกจากนี้ VCS ยังเก็บ
metadata ต่าง ๆ เช่น ใครเป็นคนสร้าง snapshot แต่ละอัน, ข้อความที่แนบมากับ snapshot แต่ละอัน เป็นต้น

ทำไม version control ถึงมีประโยชน์? แม้ตอนที่ทำงานคนเดียว มันก็ช่วยให้ดู snapshot เก่า ๆ
ของโปรเจกต์ได้ เก็บ log ว่าทำไมถึงแก้ไขอะไรบางอย่าง ทำงานบน branch ที่พัฒนาไปพร้อม ๆ กันได้
และอื่น ๆ อีกมาก พอทำงานร่วมกับคนอื่น มันยิ่งเป็นเครื่องมือที่ขาดไม่ได้ ทั้งการดูว่าคนอื่นแก้อะไรไปบ้าง
และการแก้ไข conflict ที่เกิดจากการพัฒนาพร้อมกัน

VCS สมัยใหม่ยังช่วยให้ตอบคำถามแบบนี้ได้อย่างง่ายดาย (และมักจะทำได้อัตโนมัติ):

- ใครเป็นคนเขียน module นี้?
- บรรทัดนี้ของ file นี้ถูกแก้ไขเมื่อไหร่? โดยใคร? แก้ทำไม?
- ในช่วง 1000 revision ที่ผ่านมา unit test ตัวนี้หยุดทำงานตอนไหนและเพราะอะไร?

แม้จะมี VCS ตัวอื่นอยู่ แต่ **Git** คือมาตรฐานโดยพฤตินัยของ version control
[XKCD comic](https://xkcd.com/1597/) นี้สะท้อนชื่อเสียงของ Git ได้ดี:

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

เนื่องจาก interface ของ Git เป็น leaky abstraction การเรียนรู้ Git แบบ top-down
(เริ่มจาก interface / command-line interface) อาจทำให้สับสนได้มาก มันเป็นไปได้ที่จะ
ท่องจำคำสั่งสักหยิบมือแล้วคิดว่ามันเป็นคาถาวิเศษ แล้วก็ทำตามแนวทางใน comic ด้านบน
ทุกครั้งที่มีอะไรผิดพลาด

แม้ว่า Git จะมี interface ที่ไม่สวยงามเท่าไหร่ แต่ design และแนวคิดที่อยู่เบื้องหลังนั้นสวยงามมาก
interface ที่ไม่สวยงามต้อง _ท่องจำ_ แต่ design ที่สวยงามสามารถ _เข้าใจ_ ได้ ด้วยเหตุนี้
เราจึงอธิบาย Git แบบ bottom-up โดยเริ่มจาก data model แล้วค่อยพูดถึง command-line interface
ทีหลัง เมื่อเข้าใจ data model แล้ว ก็จะเข้าใจคำสั่งต่าง ๆ ได้ดีขึ้นว่ามันจัดการกับ data model
ที่อยู่เบื้องหลังอย่างไร

# Data model ของ Git

ความชาญฉลาดของ Git อยู่ที่ data model ที่ออกแบบมาอย่างดี ซึ่งทำให้ฟีเจอร์ดี ๆ ของ
version control เป็นไปได้ทั้งหมด ไม่ว่าจะเป็นการเก็บประวัติ การรองรับ branch และการทำงานร่วมกัน

## Snapshot

Git จำลองประวัติของชุด file และ folder ภายใน top-level directory บางตัว ให้เป็นชุดของ
snapshot ในศัพท์ของ Git file เรียกว่า "blob" ซึ่งก็คือข้อมูลกลุ่มหนึ่งของ byte
directory เรียกว่า "tree" ซึ่ง map ชื่อไปยัง blob หรือ tree อื่น (ดังนั้น directory
สามารถมี directory อื่นข้างในได้) snapshot คือ top-level tree ที่ถูกติดตามอยู่
ตัวอย่างเช่น เราอาจมี tree แบบนี้:

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

top-level tree ประกอบด้วยสองสิ่ง: tree "foo" (ซึ่งข้างในมี blob "bar.txt" หนึ่งตัว) และ blob "baz.txt"

## การจำลองประวัติ: ความสัมพันธ์ระหว่าง snapshot

Version control system ควรเชื่อมโยง snapshot เข้าด้วยกันอย่างไร? model แบบง่ายอันหนึ่ง
คือเก็บประวัติแบบเป็นเส้นตรง ประวัติก็จะเป็นลิสต์ของ snapshot เรียงตามลำดับเวลา
ด้วยเหตุผลหลายประการ Git ไม่ได้ใช้ model แบบง่ายแบบนี้

ใน Git ประวัติคือ directed acyclic graph (DAG) ของ snapshot อาจฟังดูเป็นศัพท์คณิตศาสตร์หรู ๆ
แต่ไม่ต้องกลัว สิ่งที่มันหมายความก็แค่ว่า snapshot แต่ละตัวใน Git จะอ้างอิงไปยังชุดของ "parent"
ซึ่งคือ snapshot ที่มาก่อนหน้ามัน เป็นชุดของ parent แทนที่จะเป็น parent ตัวเดียว (อย่างที่จะเป็น
ในประวัติแบบเส้นตรง) เพราะ snapshot อาจสืบเชื้อสายมาจากหลาย parent ได้ เช่น เมื่อรวม
(merge) สอง branch ที่พัฒนาแยกกันเข้าด้วยกัน

Git เรียก snapshot เหล่านี้ว่า "commit" การมองประวัติ commit เป็นภาพอาจดูประมาณนี้:

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

ใน ASCII art ด้านบน `o` แต่ละตัวแทน commit (snapshot) แต่ละอัน ลูกศรชี้ไปที่ parent
ของแต่ละ commit (เป็นความสัมพันธ์แบบ "มาก่อน" ไม่ใช่ "มาทีหลัง") หลังจาก commit ที่สาม
ประวัติแตกออกเป็นสอง branch แยกกัน ซึ่งอาจหมายถึง เช่น สองฟีเจอร์ที่พัฒนาไปพร้อม ๆ กัน
อย่างอิสระจากกัน ในอนาคต branch เหล่านี้อาจถูก merge เข้าด้วยกันเพื่อสร้าง snapshot ใหม่
ที่รวมทั้งสองฟีเจอร์เข้าด้วยกัน ทำให้เกิดประวัติที่มีหน้าตาแบบนี้ โดย merge commit
ที่สร้างขึ้นใหม่แสดงเป็นตัวหนา:

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</code>
</pre>

Commit ใน Git นั้น immutable ไม่ได้หมายความว่าแก้ไขข้อผิดพลาดไม่ได้นะ แต่ "การแก้ไข"
ประวัติ commit จริง ๆ แล้วคือการสร้าง commit ใหม่ขึ้นมาทั้งหมด แล้ว reference (ดูด้านล่าง)
จะถูกอัปเดตให้ชี้ไปที่ commit ใหม่แทน

## Data model ในรูปแบบ pseudocode

การดู data model ของ Git ที่เขียนเป็น pseudocode อาจช่วยให้เข้าใจได้ดีขึ้น:

```
// file คือข้อมูลกลุ่มหนึ่งของ byte
type blob = array<byte>

// directory ประกอบด้วย file และ directory ที่มีชื่อ
type tree = map<string, tree | blob>

// commit มี parent, metadata, และ top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

เป็น model ที่สะอาดและเรียบง่ายสำหรับการเก็บประวัติ

## Object และ content-addressing

"object" คือ blob, tree, หรือ commit:

```
type object = blob | tree | commit
```

ใน data store ของ Git object ทั้งหมดจะถูก content-address ด้วย [SHA-1
hash](https://en.wikipedia.org/wiki/SHA-1)

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blob, tree, และ commit ถูกรวมเป็นหนึ่งในลักษณะนี้: ทั้งหมดคือ object เมื่อ object
อ้างอิงถึง object อื่น มันไม่ได้ _เก็บ_ object นั้นจริง ๆ ใน on-disk representation
แต่เก็บเป็น reference ไปยัง hash ของ object นั้นแทน

ตัวอย่างเช่น tree ของโครงสร้าง directory ตัวอย่าง[ด้านบน](#snapshots)
(แสดงผลด้วย `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`)
มีหน้าตาแบบนี้:

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

ตัว tree เองมี pointer ชี้ไปยังเนื้อหาข้างใน คือ `baz.txt` (blob) และ `foo`
(tree) ถ้าดูเนื้อหาที่ hash ของ baz.txt ชี้ไป ด้วยคำสั่ง
`git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85` จะได้ผลลัพธ์นี้:

```
git is wonderful
```

## Reference

ตอนนี้ snapshot ทั้งหมดสามารถระบุตัวตนได้ด้วย SHA-1 hash แต่นั่นไม่สะดวก
เพราะมนุษย์ไม่ถนัดจำ string ที่เป็นเลขฐานสิบหก 40 ตัวอักษร

วิธีแก้ปัญหานี้ของ Git คือชื่อที่มนุษย์อ่านได้สำหรับ SHA-1 hash เรียกว่า
"reference" reference คือ pointer ที่ชี้ไปยัง commit ต่างจาก object ที่เป็น
immutable ตรงที่ reference เป็น mutable (สามารถอัปเดตให้ชี้ไปที่ commit ใหม่ได้)
ตัวอย่างเช่น reference ชื่อ `master` มักจะชี้ไปที่ commit ล่าสุดใน branch หลักของการพัฒนา

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

ด้วยวิธีนี้ Git สามารถใช้ชื่อที่มนุษย์อ่านได้อย่าง "master" เพื่ออ้างถึง snapshot
หนึ่ง ๆ ในประวัติ แทนที่จะเป็น string เลขฐานสิบหกยาว ๆ

รายละเอียดอีกอย่างหนึ่งคือ เรามักต้องการแนวคิดของ "ตอนนี้เราอยู่ตรงไหน" ในประวัติ
เพื่อที่ว่าเมื่อสร้าง snapshot ใหม่ เราจะได้รู้ว่ามันสัมพันธ์กับอะไร (ค่าใน field `parents`
ของ commit ตั้งยังไง) ใน Git "ตอนนี้เราอยู่ตรงไหน" คือ reference พิเศษที่เรียกว่า "HEAD"

## Repository

สุดท้ายแล้ว เราสามารถนิยาม (คร่าว ๆ) ได้ว่า Git _repository_ คืออะไร: มันคือข้อมูล
`objects` และ `references`

บน disk สิ่งที่ Git เก็บทั้งหมดคือ object และ reference: นั่นคือทุกอย่างใน data model
ของ Git คำสั่ง `git` ทุกคำสั่ง map ไปยังการจัดการ commit DAG บางอย่าง ไม่ว่าจะเป็น
การเพิ่ม object หรือเพิ่ม/อัปเดต reference

ทุกครั้งที่พิมพ์คำสั่งใด ๆ ให้คิดว่าคำสั่งนั้นกำลังจัดการกับ graph data structure ที่อยู่เบื้องหลัง
อย่างไร ในทางกลับกัน ถ้าต้องการเปลี่ยนแปลง commit DAG แบบเฉพาะเจาะจง เช่น "ยกเลิก
uncommitted change และทำให้ reference 'master' ชี้ไปที่ commit `5d83f9e`" ก็น่าจะ
มีคำสั่งที่ทำแบบนั้นได้ (เช่น ในกรณีนี้คือ `git checkout master; git reset
--hard 5d83f9e`)

# Staging area

นี่คือแนวคิดอีกอันหนึ่งที่เป็นอิสระจาก data model แต่เป็นส่วนหนึ่งของ interface
สำหรับสร้าง commit

วิธีหนึ่งที่อาจนึกออกสำหรับการทำ snapshot ตามที่อธิบายไว้ข้างต้น คือมีคำสั่ง
"create snapshot" ที่สร้าง snapshot ใหม่จาก _สถานะปัจจุบัน_ ของ working directory
version control tool บางตัวทำงานแบบนี้ แต่ Git ไม่ได้ทำแบบนั้น เราต้องการ snapshot
ที่สะอาด และมันอาจไม่เหมาะเสมอไปที่จะสร้าง snapshot จากสถานะปัจจุบัน ตัวอย่างเช่น
ลองนึกภาพสถานการณ์ที่ implement สองฟีเจอร์แยกกัน แล้วต้องการสร้างสอง commit แยกกัน
โดย commit แรกเป็นฟีเจอร์แรก และ commit ถัดไปเป็นฟีเจอร์ที่สอง หรือลองนึกภาพ
สถานการณ์ที่มี debugging print statement กระจายอยู่ทั่ว code พร้อมกับ bugfix
ต้องการ commit เฉพาะ bugfix โดยทิ้ง print statement ทั้งหมด

Git รองรับสถานการณ์แบบนี้โดยให้ระบุได้ว่า modification ไหนบ้างที่ควรรวมอยู่ใน
snapshot ถัดไป ผ่านกลไกที่เรียกว่า "staging area"

# Git command-line interface

เพื่อไม่ให้ข้อมูลซ้ำซ้อน เราจะไม่อธิบายคำสั่งด้านล่างโดยละเอียดในเนื้อหาบทเรียนนี้
ดูหนังสือที่แนะนำอย่างยิ่ง [Pro Git](https://git-scm.com/book/en/v2) สำหรับข้อมูลเพิ่มเติม
หรือดูวิดีโอบทเรียน

## พื้นฐาน

- `git help <command>`: ดูข้อมูลช่วยเหลือของคำสั่ง git
- `git init`: สร้าง git repo ใหม่ โดยเก็บข้อมูลใน directory `.git`
- `git status`: บอกสถานะปัจจุบัน
- `git add <filename>`: เพิ่ม file ไปยัง staging area
- `git commit`: สร้าง commit ใหม่
    - เขียน [commit message ที่ดี](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
    - อีกเหตุผลที่ควรเขียน [commit message ที่ดี](https://chris.beams.io/posts/git-commit/)!
- `git log`: แสดง log ประวัติแบบเรียบ ๆ
- `git log --all --graph --decorate`: แสดงประวัติเป็น DAG
- `git diff <filename>`: แสดงการเปลี่ยนแปลงเทียบกับ staging area
- `git diff <revision> <filename>`: แสดงความแตกต่างของ file ระหว่าง snapshot
- `git checkout <revision>`: อัปเดต HEAD (และ branch ปัจจุบันถ้า checkout branch)

## Branching และ merging

- `git branch`: แสดง branch ทั้งหมด
- `git branch <name>`: สร้าง branch
- `git switch <name>`: สลับไปยัง branch
- `git checkout -b <name>`: สร้าง branch แล้วสลับไปทันที
    - เหมือนกับ `git branch <name>; git switch <name>`
- `git merge <revision>`: merge เข้า branch ปัจจุบัน
- `git mergetool`: ใช้เครื่องมือช่วยแก้ merge conflict
- `git rebase`: rebase ชุด patch ไปบน base ใหม่

## Remote

- `git remote`: แสดงรายการ remote
- `git remote add <name> <url>`: เพิ่ม remote
- `git push <remote> <local branch>:<remote branch>`: ส่ง object ไปยัง remote และอัปเดต remote reference
- `git branch --set-upstream-to=<remote>/<remote branch>`: ตั้งค่าความสัมพันธ์ระหว่าง local branch กับ remote branch
- `git fetch`: ดึง object/reference จาก remote
- `git pull`: เหมือน `git fetch; git merge`
- `git clone`: ดาวน์โหลด repository จาก remote

## Undo

- `git commit --amend`: แก้ไขเนื้อหา/ข้อความของ commit
- `git reset <file>`: unstage file
- `git restore`: ยกเลิกการเปลี่ยนแปลง

# Advanced Git

- `git config`: Git มี[ตัวเลือกปรับแต่งมากมาย](https://git-scm.com/docs/git-config)
- `git clone --depth=1`: shallow clone โดยไม่ดึงประวัติทั้งหมด
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: แสดงว่าใครแก้ไขแต่ละบรรทัดล่าสุด
- `git stash`: เก็บการแก้ไขใน working directory ไว้ชั่วคราว
- `git bisect`: binary search ประวัติ (เช่น หา regression)
- `git revert`: สร้าง commit ใหม่ที่ย้อนกลับผลของ commit ก่อนหน้า
- `git worktree`: checkout หลาย branch พร้อมกัน
- `.gitignore`: [กำหนด](https://git-scm.com/docs/gitignore)ว่า file ที่ไม่ต้องการติดตามตัวไหนบ้างที่จะให้ ignore

# เบ็ดเตล็ด

- **GUI**: มี [GUI client](https://git-scm.com/downloads/guis) สำหรับ Git มากมาย
เราเองไม่ได้ใช้ และใช้ command-line interface แทน
- **Shell integration**: การมี Git status เป็นส่วนหนึ่งของ shell prompt นั้นสะดวกมาก
([zsh](https://github.com/olivierverdier/zsh-git-prompt),
[bash](https://github.com/magicmonty/bash-git-prompt)) มักรวมอยู่ใน
framework อย่าง [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)
- **Editor integration**: เช่นเดียวกับข้อบน มี integration กับ editor ที่มีฟีเจอร์มากมาย
[fugitive.vim](https://github.com/tpope/vim-fugitive) เป็นตัวมาตรฐานสำหรับ Vim
- **Workflow**: เราสอน data model และคำสั่งพื้นฐานไปแล้ว แต่ยังไม่ได้บอกว่าควรใช้
practice อะไรตอนทำงานในโปรเจกต์ใหญ่ (ซึ่งมี[แนวทาง](https://nvie.com/posts/a-successful-git-branching-model/)
[ที่แตกต่างกัน](https://www.endoflineblog.com/gitflow-considered-harmful)
[หลายแบบ](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow))
- **GitHub**: Git ไม่ใช่ GitHub นะ GitHub มีวิธีเฉพาะในการ contribute code ไปยัง
โปรเจกต์อื่น เรียกว่า [pull
request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)
- **Git provider อื่น ๆ**: GitHub ไม่ใช่ตัวเลือกเดียว มี Git repository host อื่นอีกมาก
เช่น [GitLab](https://about.gitlab.com/) และ
[BitBucket](https://bitbucket.org/)

# แหล่งข้อมูล

- [Pro Git](https://git-scm.com/book/en/v2) **แนะนำอย่างยิ่งให้อ่าน**
อ่าน Chapter 1--5 ก็น่าจะเพียงพอให้ใช้ Git ได้อย่างคล่องแคล่ว เมื่อเข้าใจ data model แล้ว
บทหลัง ๆ มีเนื้อหา advanced ที่น่าสนใจ
- [Oh Shit, Git!?!](https://ohshitgit.com/) เป็นคู่มือสั้น ๆ เรื่องวิธีกู้คืนจาก
ข้อผิดพลาดที่พบบ่อยใน Git
- [Git for Computer
Scientists](https://eagain.net/articles/git-for-computer-scientists/) เป็น
คำอธิบายสั้น ๆ เกี่ยวกับ data model ของ Git ที่มี pseudocode น้อยกว่าแต่มีแผนภาพสวย ๆ
มากกว่าเนื้อหาบทเรียนนี้
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)
เป็นคำอธิบายโดยละเอียดเกี่ยวกับรายละเอียดการ implement ของ Git ที่ลึกกว่าแค่ data model
สำหรับคนที่อยากรู้เพิ่มเติม
- [How to explain git in simple
words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) เป็นเกมบน browser
ที่สอนการใช้ Git

# แบบฝึกหัด

1. ถ้ายังไม่เคยใช้ Git มาก่อน ลองอ่านสองสามบทแรกของ [Pro Git](https://git-scm.com/book/en/v2)
   หรือทำ tutorial อย่าง [Learn Git Branching](https://learngitbranching.js.org/)
   ระหว่างทำ ให้ลองเชื่อมโยงคำสั่ง Git กับ data model
2. Clone [repository ของ
เว็บไซต์วิชานี้](https://github.com/missing-semester/missing-semester)
    1. สำรวจประวัติ version โดยแสดงเป็น graph
    2. ใครเป็นคนแก้ไข `README.md` คนสุดท้าย? (คำใบ้: ใช้ `git log` กับ
       argument)
    3. commit message ของการแก้ไขบรรทัด `collections:` ใน `_config.yml` ครั้งล่าสุด
       คืออะไร? (คำใบ้: ใช้ `git blame` และ `git show`)
3. ข้อผิดพลาดที่พบบ่อยตอนเรียน Git คือการ commit file ขนาดใหญ่ที่ไม่ควรจัดการด้วย Git
   หรือเพิ่มข้อมูลที่เป็นความลับเข้าไป ลอง add file เข้า repository ทำ commit สักสองสาม
   ครั้ง แล้วลบ file นั้นออกจาก _ประวัติ_ (ไม่ใช่แค่จาก commit ล่าสุด) อาจต้องดู
   [บทความนี้](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)
4. Clone repository จาก GitHub สักอัน แล้วแก้ไข file ที่มีอยู่ เกิดอะไรขึ้นเมื่อรัน `git stash`?
   เห็นอะไรเมื่อรัน `git log --all --oneline`? รัน `git stash pop` เพื่อ undo สิ่งที่ทำกับ
   `git stash` ในสถานการณ์ไหนที่คำสั่งนี้อาจมีประโยชน์?
5. เหมือน command line tool อื่น ๆ หลายตัว Git มี configuration file (หรือ dotfile)
   ชื่อ `~/.gitconfig` สร้าง alias ใน `~/.gitconfig` เพื่อให้เมื่อรัน `git graph`
   จะได้ผลลัพธ์ของ `git log --all --graph --decorate --oneline` ทำได้โดย
   [แก้ไข](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias)
   file `~/.gitconfig` โดยตรง หรือใช้คำสั่ง `git config` เพื่อเพิ่ม alias
   ข้อมูลเกี่ยวกับ git alias ดูได้
   [ที่นี่](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)
6. สามารถกำหนด global ignore pattern ใน `~/.gitignore_global` ได้หลังจากรัน
   `git config --global core.excludesfile ~/.gitignore_global` คำสั่งนี้ตั้งค่า
   ตำแหน่งของ global ignore file ที่ Git จะใช้ แต่ยังต้องสร้าง file นั้นที่ path ดังกล่าว
   ด้วยตัวเอง ตั้งค่า global gitignore file ให้ ignore file ชั่วคราวเฉพาะ OS หรือ
   editor เช่น `.DS_Store`
7. Fork [repository ของ
   เว็บไซต์วิชานี้](https://github.com/missing-semester/missing-semester) หา typo
   หรือสิ่งที่ปรับปรุงได้ แล้ว submit pull request บน GitHub
   (อาจต้องดู[บทความนี้](https://github.com/firstcontributions/first-contributions))
   กรุณา submit เฉพาะ PR ที่มีประโยชน์ (อย่า spam นะ!) ถ้าหาสิ่งที่ต้องปรับปรุงไม่ได้
   ข้ามแบบฝึกหัดข้อนี้ได้
8. ฝึกแก้ merge conflict โดยจำลองสถานการณ์ทำงานร่วมกัน:
    1. สร้าง repository ใหม่ด้วย `git init` แล้วสร้าง file ชื่อ
       `recipe.txt` ที่มีเนื้อหาสักสองสามบรรทัด (เช่น สูตรอาหารง่าย ๆ)
    2. Commit แล้วสร้างสอง branch: `git branch salty` และ `git branch
       sweet`
    3. ใน branch `salty` แก้ไขบรรทัดหนึ่ง (เช่น เปลี่ยน "1 cup sugar" เป็น "1
       cup salt") แล้ว commit
    4. ใน branch `sweet` แก้ไขบรรทัดเดียวกันให้ต่างออกไป (เช่น เปลี่ยน "1
       cup sugar" เป็น "2 cups sugar") แล้ว commit
    5. ตอนนี้สลับไปที่ `master` แล้วลอง `git merge salty` จากนั้น `git merge
       sweet` เกิดอะไรขึ้น? ดูเนื้อหาของ `recipe.txt` -- เครื่องหมาย
       `<<<<<<<`, `=======`, และ `>>>>>>>` หมายความว่าอะไร?
    6. แก้ conflict โดยแก้ไข file ให้เหลือเฉพาะเนื้อหาที่ต้องการ ลบ conflict marker ออก
       แล้วทำ merge ให้เสร็จด้วย `git add` และ `git commit` (หรือ `git merge --continue`)
       อีกทางเลือกหนึ่งคือลองใช้ `git mergetool` เพื่อแก้ conflict ด้วยเครื่องมือ merge
       แบบกราฟิกหรือแบบ terminal
    7. ใช้ `git log --graph --oneline` เพื่อดู merge history ที่เพิ่งสร้างขึ้น
