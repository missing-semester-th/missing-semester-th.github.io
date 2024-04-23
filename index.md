---
layout: page
title: The Missing Semester of Your CS Education ฉบับแปลภาษาไทย
nositetitle: true
---

วิชาเรียนในหลักสูตรสาขาวิชาวิทยาการคอมพิวเตอร์ (CS) จะเต็มไปด้วยวิชาที่ถือได้ว่ายากเยอะแยะเต็มไปหมด ตั้งแต่เรื่อง operating systems ไปจนถึง machine learning แต่ถึงอย่างนั้น มันดันมีเนื้อหาหนึ่งที่แทบไม่มีใครสอน คือนอกจากจะไม่ได้พูดถึงมันแล้ว ยังปล่อยให้นักเรียนคิดหาทางที่จะต้องเก่งขึ้นในเรื่องเหล่านั้นด้วยตัวเองอีกด้วย พวกเราเลยจะมาช่วยสอนเนื้อหาตรงนั้น เพื่อทำให้คุณมีความเชี่ยวชาญมากขึ้น โดยเนื้อหาดังกล่าวจะประกอบไปด้วย วิธีการใช้งาน command-line, ใช้งาน text editor, ตลอดจนการใช้งาน version control systems และอื่น ๆ อีกมายมาย!

สิ่งที่น่าตลกก็คือ เนื้อหาเหล่านั้นเป็นสิ่งที่นักเรียนได้นำไปใช้งานจริงตั้งแต่ที่อยู่ในรั้วการศึกษาไปจนถึงชีวิตการทำงาน ใช้เวลาไปกับมันหลายร้อยถึงหลายพันชั่วโมง ดังนั้น มันถึงเวลาแล้วล่ะที่พวกเราจะนำเอาเนื้อหาเหล่านั้นมาทำให้เป็นสิ่งที่นักเรียนสามารถเข้าถึงได้ง่ายและจับต้องได้มากขึ้น เพราะการเข้าใจในเนื้อหาเกี่ยวกับเครื่องมือเหล่านั้น นอกจากจะช่วยลดเวลาในการควานหาวิธีการใช้เครื่องมือ (ที่ไม่มีใครสอน) ด้วยตัวเองแล้ว ยังช่วยให้สามารถนำเครื่องมือเหล่านั้นไปแก้ไขปัญหาที่ซับซ้อนมากขึ้นได้อีกด้วย

อ่านเกี่ยวกับ [แรงบันดาลใจที่ทำให้วิชานี้เกิดขึ้นได้ที่นี่](/about/).

{% comment %}
# Registration

ลงทะเบียนเรียนวิชา IAP 2020 ด้วยการกรอกแบบฟอร์มนี้ [registration form](https://forms.gle/TD1KnwCSV52qexVt9). (ปิดรับแล้ว)
{% endcomment %}

# Schedule

{% comment %}
**Lecture**: 35-225, 2pm--3pm<br>
**Office hours**: 32-G9 lounge, 3pm--4pm (every day, right after lecture)
{% endcomment %}

<ul>
{% assign lectures = site['2020'] | sort: 'date' %}
{% for lecture in lectures %}
    {% if lecture.phony != true %}
        <li>
        <strong>{{ lecture.date | date: '%-m/%d/%y' }}</strong>:
        {% if lecture.ready %}
            <a href="{{ lecture.url }}">{{ lecture.title }}</a>
        {% else %}
            {{ lecture.title }} {% if lecture.noclass %}[no class]{% endif %}
        {% endif %}
        </li>
    {% endif %}
{% endfor %}
</ul>

วิดีโอบันทึกการสอนจะถูกเก็บไว้ที่ [on
YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J).

# About the class

**ผู้สอน**: วิชานี้สอนโดย [Anish](https://www.anishathalye.com/), [Jon](https://thesquareplanet.com/), and [Jose](http://josejg.com/).<br>
**หากมีคำถาม**: ให้ส่ง Email มาหาเราได้ที่ [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

# Beyond MIT

แหล่งเรียนรู้อื่น ๆ เผื่อว่าจะมีประโยชน์

 - [Hacker News](https://news.ycombinator.com/item?id=22226380)
 - [Lobsters](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit)
 - [/r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/)
 - [/r/programming](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/)
 - [Twitter](https://twitter.com/jonhoo/status/1224383452591509507)
 - [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J)

{% comment %}
Some more URLs:

- https://news.ycombinator.com/item?id=27154577
- https://news.ycombinator.com/item?id=34934216
- https://www.reddit.com/r/learnprogramming/comments/nca1v3/mit_the_missing_semester_of_your_cs_education/
- https://www.reddit.com/r/compsci/comments/eyywv8/the_missing_semester_of_your_cs_education_from_mit/
- https://www.reddit.com/r/programming/comments/io7nq3/the_missing_semester_of_your_cs_education_mit/
- https://twitter.com/MIT_CSAIL/status/1349766980413263873
- https://twitter.com/MIT_CSAIL/status/1481676163491659780
- https://twitter.com/MIT_CSAIL/status/1581313961093484545
{% endcomment %}

# Translations

- [Chinese (Simplified)](https://missing-semester-cn.github.io/)
- [Chinese (Traditional)](https://missing-semester-zh-hant.github.io/)
- [Japanese](https://missing-semester-jp.github.io/)
- [Korean](https://missing-semester-kr.github.io/)
- [Portuguese](https://missing-semester-pt.github.io/)
- [Russian](https://missing-semester-rus.github.io/)
- [Serbian](https://netboxify.com/missing-semester/)
- [Spanish](https://missing-semester-esp.github.io/)
- [Turkish](https://missing-semester-tr.github.io/)
- [Vietnamese](https://missing-semester-vn.github.io/)
- [Arabic](https://missing-semester-ar.github.io/)
- [Italian](https://missing-semester-it.github.io/)
- [Persian](https://missing-semester-fa.github.io/)
- [German](https://missing-semester-de.github.io/)
- [Thai (ภาษาไทย)](https://missing-semester-th.github.io/)

ป.ล. ลิงก์เหล่านี้เป็นลิงก์ที่มีคนใน community อาสาแปลให้ทั้งหมด ไม่ได้จัดทำโดยทางมหาวิทยาลัยแต่อย่างใด

หากต้องการช่วยแปลเนื้อหาวิชาเรียนเหล่านี้ให้เป็นภาษาไทย สามารถเปิด [pull request](https://github.com/missing-semester-th/missing-semester-th.github.io/pulls) มาได้เลย

## Acknowledgements

We thank Elaine Mello, Jim Cain, and [MIT Open
Learning](https://openlearning.mit.edu/) for making it possible for us to
record lecture videos; Anthony Zolnik and [MIT
AeroAstro](https://aeroastro.mit.edu/) for A/V equipment; and Brandi Adams and
[MIT EECS](https://www.eecs.mit.edu/) for supporting this class.

---

<div class="small center">
<p><a href="https://github.com/missing-semester/missing-semester">Source code</a>.</p>
<p>Licensed under CC BY-NC-SA.</p>
<p>See <a href="/license/">here</a> for contribution &amp; translation guidelines.</p>
</div>
