# The Missing Semester of Your CS Education ฉบับแปลภาษาไทย (Thai language)

[![Build Status](https://github.com/missing-semester/missing-semester/workflows/Build/badge.svg)](https://github.com/missing-semester/missing-semester/actions?query=workflow%3ABuild) [![Links Status](https://github.com/missing-semester/missing-semester/workflows/Links/badge.svg)](https://github.com/missing-semester/missing-semester/actions?query=workflow%3ALinks)

เว็บไซต์สำหรับ [The Missing Semester of Your CS Education](https://missing-semester-th.github.io/) ฉบับแปลภาษาไทย

หากใครต้องการแก้ไขหรือเพิ่มเนื้อหาใหม่ สามารถเปิด issue หรือ pull request มาได้เลย!

## Development

หากต้องการ build เพื่อดูเว็บไซต์บนเครื่อง local ของตัวเอง ให้ run คำสั่งข้างล่างนี้:

```bash
bundle exec jekyll serve -w
```

*(แนะนำวิธีนี้)* หากต้องการ build แบบไม่ต้องการติดตั้ง Ruby และ dependencies ต่าง ๆ บนเครื่อง local ของตัวเอง ให้ build ผ่าน Docker โดย run คำสั่งนี้:

```bash
docker-compose up --build
```

หลังจาก build เสร็จแล้วให้ไปที่ <http://localhost:4000> บนเครื่อง local ของตัวเองเพื่อดูเว็บไซต์ได้เลย ทันทีที่มีการแก้ไขไฟล์ เว็บไซต์นี้จะถูก build ใหม่ ซึ่งจะทำให้เห็นการเปลี่ยนแปลงบนเว็บไซต์ทันที

## License

All the contents in this course, including the website source code, lecture notes, exercises, and lecture videos are licensed under Attribution-NonCommercial-ShareAlike 4.0 International [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). See [here](https://missing.csail.mit.edu/license) for more information on contributions or translations.
