# Teleprompter PWA

จอบอกบท (teleprompter) เลื่อนอัตโนมัติสำหรับอ่านหน้ากล้อง — ปรับความเร็ว/ขนาดอักษรได้, โหมดกระจก, กล้องหน้าเป็นพื้นหลัง, อัดวิดีโอแล้วเซฟลง Photos
ทำเป็น **PWA** ติดตั้งลงมือถือได้ทั้ง **iPhone และ Android** ใช้งานออฟไลน์ได้หลังติดตั้ง

---

## 1. โครงสร้างโปรเจกต์

```
teleprompter/
├── index.html            # ตัวแอป (HTML/CSS/JS อยู่ในไฟล์เดียว)
├── manifest.json         # ข้อมูลแอปสำหรับติดตั้ง (ชื่อ, ไอคอน, สี, โหมดเต็มจอ)
├── sw.js                 # service worker: cache ให้ใช้งานออฟไลน์ + ทำให้ Android ติดตั้งได้
├── icons/
│   ├── icon-192.png      # ไอคอน 192×192
│   ├── icon-512.png      # ไอคอน 512×512
│   └── maskable-512.png  # ไอคอน maskable (สำหรับ adaptive icon ของ Android)
└── README.md
```

> **สำคัญ:** ทุก path ในโปรเจกต์เป็นแบบ **relative (`./`)** ตั้งใจไว้เพื่อให้ทำงานได้เมื่อ host บน GitHub Pages ที่อยู่ใต้ subpath เช่น `https://<user>.github.io/teleprompter/` — อย่าเปลี่ยนเป็น path แบบขึ้นต้นด้วย `/`

---

## 2. เงื่อนไขสำคัญก่อนเริ่ม

- ต้อง serve ผ่าน **HTTPS** เท่านั้น (GitHub Pages ให้ฟรีอยู่แล้ว) — ไม่งั้น **กล้อง / ไมค์ / service worker จะไม่ทำงาน**
- ต้องมี `git` และบัญชี GitHub — ถ้าใช้ GitHub CLI ต้องมี `gh` และล็อกอินไว้ (`gh auth login`)

ตรวจเครื่องก่อน:

```bash
git --version
gh --version      # ถ้าจะใช้ GitHub CLI (ไม่บังคับ)
```

---

## 3. เอาขึ้น GitHub ผ่าน CLI

เข้าไปในโฟลเดอร์โปรเจกต์ก่อน:

```bash
cd teleprompter
git init
git add .
git commit -m "Teleprompter PWA: initial version"
git branch -M main
```

### วิธี A — ใช้ GitHub CLI (`gh`) เร็วสุด

สร้าง repo + push ในคำสั่งเดียว:

```bash
gh repo create teleprompter --public --source=. --remote=origin --push
```

### วิธี B — git ธรรมดา (สร้าง repo บนเว็บก่อน)

1. ไปสร้าง repo เปล่าชื่อ `teleprompter` ที่ https://github.com/new (ไม่ต้องติ๊ก add README)
2. แล้วรัน (แทน `<user>` ด้วยชื่อผู้ใช้ของคุณ):

```bash
git remote add origin https://github.com/<user>/teleprompter.git
git push -u origin main
```

---

## 4. เปิด GitHub Pages

### วิธี A — ผ่าน `gh` (CLI)

```bash
gh api -X POST repos/<user>/teleprompter/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

ดูสถานะ/URL:

```bash
gh api repos/<user>/teleprompter/pages --jq '.html_url, .status'
```

### วิธี B — ผ่านหน้าเว็บ

`repo → Settings → Pages → Build and deployment`
ตั้ง **Source = Deploy from a branch**, **Branch = `main`**, **Folder = `/ (root)`** แล้ว **Save**

รอสัก 1–2 นาที จะได้ URL:

```
https://<user>.github.io/teleprompter/
```

เปิด URL นี้ในเบราว์เซอร์มือถือได้เลย

---

## 5. ติดตั้งลงมือถือ

### iPhone / iPad (ต้องใช้ Safari)

1. เปิด URL ใน **Safari**
2. แตะปุ่ม **แชร์** (สี่เหลี่ยมมีลูกศรขึ้น)
3. เลือก **เพิ่มไปยังหน้าจอโฮม (Add to Home Screen)**
4. ได้ไอคอนบนหน้าจอ เปิดแบบเต็มจอเหมือนแอปจริง

> iOS ติดตั้งด้วยการ "Add to Home Screen" เท่านั้น (ไม่มีปุ่ม Install อัตโนมัติ) และต้องเป็น Safari — Chrome บน iOS ทำไม่ได้

### Android (Chrome / Edge)

1. เปิด URL ใน **Chrome**
2. จะมีแถบ/ป็อปอัป **"ติดตั้งแอป (Install app)"** ขึ้นมาเอง — แตะติดตั้ง
   - ถ้าไม่ขึ้น: แตะเมนู `⋮` มุมขวาบน → **ติดตั้งแอป / เพิ่มไปที่หน้าจอหลัก**
3. ได้แอปจริงในเครื่อง เปิดเต็มจอ ใช้ออฟไลน์ได้

---

## 6. การใช้งานแอปโดยย่อ

- วางสคริปต์ → กด **เริ่มอ่าน** ตัวหนังสือเลื่อนขึ้นอัตโนมัติ
- **แตะจอ** = พัก/เล่น • **แถบล่าง** = ปรับความเร็ว/ขนาด/กระจกระหว่างอ่าน
- เปิด toggle **อัดวิดีโอ** → ในหน้าอ่านจะมี **ปุ่มแดงมุมซ้ายบน** กดเริ่ม/หยุดอัด
- อัดเสร็จ → หน้าพรีวิว → **บันทึกลง Photos** → เลือก **Save Video**
- วิดีโอที่อัดเป็น **ภาพจากกล้องล้วน ไม่มีตัวหนังสือติดไปด้วย**

---

## 7. อัปเดตแอปภายหลัง

หลังแก้ไฟล์ ให้ **เปลี่ยนเลขเวอร์ชันใน `sw.js`** เพื่อบังคับให้เครื่องโหลดของใหม่ (ไม่งั้นจะติด cache เก่า):

```js
// sw.js บรรทัดแรก
const CACHE = 'teleprompter-v2';   // เดิม v1 → เปลี่ยนเป็น v2 ทุกครั้งที่ปล่อยเวอร์ชันใหม่
```

แล้ว push ตามปกติ:

```bash
git add .
git commit -m "update: <อธิบายสั้นๆ>"
git push
```

ฝั่งมือถือ: ปิด-เปิดแอป 1–2 รอบ service worker จะสลับมาเวอร์ชันใหม่เอง

---

## 8. แก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุ / วิธีแก้ |
|---|---|
| กล้อง/ไมค์ไม่ทำงาน | ต้องเปิดผ่าน **HTTPS** และกด **อนุญาต** สิทธิ์กล้อง/ไมค์ในเบราว์เซอร์ |
| Android ไม่ขึ้นปุ่ม Install | ต้องมี `manifest.json` + `sw.js` โหลดสำเร็จ และเป็น HTTPS — เช็ก DevTools → Application → Manifest |
| ไอคอนไม่ขึ้น / เพี้ยน | เช็กว่าไฟล์ใน `icons/` อัปครบและชื่อตรงกับใน `manifest.json` |
| แก้โค้ดแล้วแอปยังเป็นของเก่า | ยังไม่ได้ bump `CACHE` ใน `sw.js` (ดูข้อ 7) หรือ hard-refresh |
| หน้า 404 บน Pages | รอ build ให้เสร็จ หรือเช็กว่า Branch/Folder ตั้งเป็น `main` `/ (root)` |
| อัดวิดีโอไม่ได้บนมือถือรุ่นเก่า | เบราว์เซอร์ไม่รองรับ `MediaRecorder` — แอปจะแจ้งเตือน ใช้เครื่อง/เบราว์เซอร์ใหม่ขึ้น |

---

## 9. ความเข้ากันได้ทุกรุ่น (สรุป)

| ฟีเจอร์ | iPhone (Safari) | Android (Chrome) |
|---|---|---|
| ติดตั้งเป็นแอป | Add to Home Screen | Install app (อัตโนมัติ) |
| ใช้งานออฟไลน์ | ได้ | ได้ |
| กล้องหน้า/พื้นหลัง | ได้ (HTTPS) | ได้ (HTTPS) |
| อัดวิดีโอ | ได้ (iOS 14.3+ → .mp4) | ได้ (→ .webm/.mp4) |
| เซฟลง Photos | Web Share → Save Video | Web Share / ดาวน์โหลด |
| เครื่องเก่ามากที่ไม่มี Web Share | ดาวน์โหลดไฟล์แทน แล้วเซฟลงคลังเอง | เหมือนกัน |

แอปเช็กความสามารถของเครื่องให้อัตโนมัติ ถ้าเครื่องไหนทำบางอย่างไม่ได้ จะ fallback ไปทางที่รองรับแทน — ไม่พังทั้งแอป
