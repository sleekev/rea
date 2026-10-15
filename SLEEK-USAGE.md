# REA — วิธีใช้งานของ Sleek EV

> fork นี้ใช้ติดตามเวอร์ชันของ [morluto/rea](https://github.com/morluto/rea) เท่านั้น ไม่มีการแก้โค้ด
> `main` sync จาก upstream แบบ fast-forward · คู่มือนี้คือไฟล์เดียวที่เพิ่มเข้ามา
> ผ่าน security review (static) ของ `rea-agents@6.5.0` แล้ว — 2026-10-15

## ใช้ทำอะไร

ใช้ REA กับของที่เรา**เป็นเจ้าของหรือได้รับอนุญาต**เท่านั้น เช่น

- firmware และ protocol ของอุปกรณ์ที่บริษัทซื้อมาใช้งาน (VCU, สถานีชาร์จ, แบตเตอรี่) เพื่อหาข้อมูลที่ vendor ไม่ได้ระบุในเอกสาร
- แอป Android/Electron ของ vendor ที่ระบบเราต้องเชื่อมต่อ (interoperability)
- binary ของระบบเราเองที่ไม่มี source แล้ว

🔴 **ห้ามใช้ REA กับ source code ของบริษัท** และของที่ build จากโค้ดของเรา (repo, APK/IPA, firmware ที่เราพัฒนา, binary, web bundle)
การใช้ REA ทุกครั้งถือเป็น **การส่งโค้ดออกนอกบริษัท** เพราะผลวิเคราะห์เข้าไปอยู่ใน context ของ AI agent และถูกส่งไปผู้ให้บริการ model
โค้ดของเราเองใช้เครื่องมือบนเครื่องตามปกติเท่านั้น · ไม่แน่ใจว่าเป้าหมายเป็นของเราหรือไม่ → หยุดแล้วถามก่อน

ก่อนเริ่ม ตรวจสัญญาหรือ license ของ vendor ว่าไม่ได้ห้าม reverse engineering
ห้ามใช้เพื่อหลบ DRM, license check หรือระบบยืนยันตัวตนของคนอื่น (ดู [DISCLAIMER.md](DISCLAIMER.md))

## ติดตั้ง (บังคับตามนี้)

```bash
# 1) ตรวจ integrity ของเวอร์ชันที่จะลง
npm view rea-agents@6.5.0 dist.integrity
#    ต้องได้ sha512-C6BG7LyrAdR74mSjertM6PrxJkcNZ2bfQNOJornPo9erDEp2tyPRVYPnV5Yodr+1RqewCQla68WulXlVnveD+A==

# 2) ล็อกเวอร์ชัน · ไม่รัน install script · ไม่ลง Frida
npm install --global --ignore-scripts --omit=optional rea-agents@6.5.0
npm audit signatures
rea --help
```

| ห้าม | เหตุผล |
|---|---|
| `curl … \| bash` (install.sh) | ติดตั้งเวอร์ชันล่าสุดแบบไม่ล็อก และรัน install script ของ dependency |
| `npx rea-agents@latest …` | รันโค้ด release ใหม่ทันทีโดยไม่ผ่าน review (package ใหม่ ผู้ดูแลคนเดียว ออกเวอร์ชันถี่) |
| `rea setup` แบบเต็มชุด | เขียน MCP ลง config ระดับ user (เปิดในทุก project) และติดตั้ง skill ลงโฟลเดอร์ skill ระดับ user |
| ติดตั้ง `frida` | Frida แทรกตัวเข้า process ใดก็ได้ของ user เดียวกัน และไม่มีสวิตช์ปิดใน MCP |

## ลง MCP เฉพาะ project ที่ทำงาน RE

ดูแผนก่อนเสมอ แล้วลงเองใน `.mcp.json` ของ repo งาน RE ไม่ลงระดับ user

```bash
rea setup --client claude_code --dry-run --json   # อ่านอย่างเดียว ดูว่าจะเขียนอะไร
```

```json
{
  "mcpServers": {
    "rea": { "command": "rea", "args": ["mcp"] }
  }
}
```

- skill `reverse-engineer-anything` ถ้าจะใช้ ให้ copy ไปไว้ที่ `.claude/skills/` **ของ repo งาน RE** เท่านั้น
- ถ้าเผลอรัน setup แล้ว ให้ลบไฟล์ `*.rea.backup` ที่เกิดข้าง config (เป็นสำเนา config ทั้งไฟล์)

## สภาพแวดล้อม

- **ห้ามเปิด REA ในเครื่องหรือ session ที่ถือ credential ของ production** หรือมี tunnel ไปฐานข้อมูล production
  tool `process`/lldb รันคำสั่งที่ agent เลือก และ process ลูกได้ environment ทั้งหมดของเครื่องไปด้วย
- binary หรือแอปที่ไม่น่าไว้ใจ วิเคราะห์ใน VM หรือ user แยกเท่านั้น — REA ไม่ใช่ sandbox ([SECURITY.md](SECURITY.md))
- ถือผลวิเคราะห์เป็นข้อมูลที่ไม่น่าไว้ใจ: ข้อความใน binary อาจหลอกให้ agent ทำเกินขอบเขต (prompt injection)
- ไม่ auto-allow tool `process`, lldb และ runtime capture ของ REA ใน permission ของ agent

## ใช้งาน

- static ก่อนเสมอ: firmware (`inspect_firmware_regions`), APK (`inspect_android_package`), JS/Electron (`analyze_javascript_application`), native (`open_binary` + Ghidra/IDA)
- เก็บ Evidence ID ไว้ทุกข้อสรุป และแยก "สังเกตเห็น / อนุมาน / ไม่ทราบ" ตามที่ skill กำหนด
- ผลวิเคราะห์ที่มีข้อมูลของ vendor หรือข้อมูลภายใน เก็บใน repo ภายในของทีม **ไม่ commit ลง fork นี้** (repo นี้เป็น public)

## อัปเดต

```bash
rea update          # ใช้ --ignore-scripts และ pin version ให้เอง · อ่าน CHANGELOG ก่อนทุกครั้ง
```

- ห้ามตั้งให้อัปเดตอัตโนมัติ
- ขึ้นเวอร์ชันใหม่ = ตรวจ integrity ใหม่ และแก้เลขเวอร์ชัน/hash ในไฟล์นี้
- sync fork: `git fetch upstream && git push origin upstream/main:main` (fast-forward เท่านั้น — ไฟล์นี้อยู่ branch `sleek`)
