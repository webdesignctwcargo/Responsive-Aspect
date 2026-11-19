# Hero Media — Responsive Aspect Ratio & Focus Point

เอกสารสำหรับ dev อธิบายโค้ดที่ใช้ปรับสัดส่วนภาพ/วิดีโอ Hero ตามขนาดหน้าจอ และวิธีใช้เครื่องมือ preview (`picture-preview.html`)

---

## 1. Breakpoints และสัดส่วนภาพ

| Breakpoint | Media query | aspect-ratio |
|---|---|---|
| Mobile | `max-width: 767px` | `4/3` |
| Tablet | `max-width: 1023px` | `16/8` |
| Desktop | ขนาดที่เหลือทั้งหมด | `16/6` |

---

## 2. HTML

ใช้ได้ 2 แบบ ภายใต้ wrapper `.hero-media`

### แบบรูปภาพ (คนละไฟล์ต่อ breakpoint)

```html
<div class="hero-media">
  <picture class="hero-picture">
    <source media="(max-width:767px)"  srcset="/images/about-mobile.gif">
    <source media="(max-width:1023px)" srcset="/images/about-tablet.gif">
    <img src="/images/about-desktop.gif" alt="CTW Cargo">
  </picture>
</div>
```

### แบบวิดีโอ (ไฟล์เดียว ครอปด้วย CSS)

```html
<div class="hero-media">
  <video class="hero-video" autoplay muted loop playsinline>
    <source src="/videos/about-desktop.mp4" type="video/mp4">
  </video>
</div>
```

---

## 3. CSS

```css
.hero-media {
  width: 100%;
  border-radius: 20px;
  overflow: hidden;
  position: relative;
}

/* Desktop */
.hero-picture,
.hero-video {
  display: block;
  width: 100%;
  aspect-ratio: 16/6;
}
.hero-picture img,
.hero-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* Tablet */
@media (max-width: 1023px) {
  .hero-picture,
  .hero-video { aspect-ratio: 16/8; }
}

/* Mobile */
@media (max-width: 767px) {
  .hero-picture,
  .hero-video { aspect-ratio: 4/3; }
}
```

`object-fit: cover` จะครอปส่วนเกินของภาพออกให้พอดีกรอบ โดยค่าเริ่มต้นครอปจากกึ่งกลาง

---

## 4. Focus point (จุดโฟกัสภาพ)

กำหนดจุดที่ต้องการให้อยู่กลางภาพเมื่อถูกครอป ค่าเป็นเปอร์เซ็นต์ของภาพต้นฉบับ (`focusX`, `focusY` ช่วง 0–100)

วิธีที่ง่ายที่สุดสำหรับ production คือกำหนด `object-position` ตรงๆ:

```css
.hero-picture img,
.hero-video {
  object-position: 30% 40%; /* x y */
}
```

> หมายเหตุ: `object-position` คือตำแหน่ง *หน้าต่างครอป* ไม่ใช่ตัวจุดโฟกัสโดยตรง
> หากต้องการให้จุดโฟกัสอยู่กลางกรอบพอดี (เมื่อเป็นไปได้) ให้คำนวณด้วยฟังก์ชันด้านล่าง

```js
// คำนวณกรอบครอปแบบ "cover" โดยให้จุดโฟกัสอยู่กลางกรอบ (clamp ไม่ให้เลยขอบภาพ)
function computeCoverCrop(sourceW, sourceH, ratioW, ratioH, fx = 50, fy = 50) {
  const target = ratioW / ratioH;
  const source = sourceW / sourceH;
  let cropW, cropH;
  if (source > target) { cropH = sourceH; cropW = sourceH * target; }
  else                 { cropW = sourceW; cropH = sourceW / target; }

  const clamp = (v, lo, hi) => Math.min(Math.max(v, lo), hi);
  return {
    sx: clamp((fx / 100) * sourceW - cropW / 2, 0, sourceW - cropW),
    sy: clamp((fy / 100) * sourceH - cropH / 2, 0, sourceH - cropH),
    sw: cropW,
    sh: cropH,
  };
}

// แปลงเป็นค่า object-position (%) สำหรับ CSS
function toObjectPosition(sourceW, sourceH, crop) {
  const px = sourceW > crop.sw ? (crop.sx / (sourceW - crop.sw)) * 100 : 50;
  const py = sourceH > crop.sh ? (crop.sy / (sourceH - crop.sh)) * 100 : 50;
  return `${px}% ${py}%`;
}
```

ตัวอย่างใช้งาน:

```js
const img = document.querySelector('.hero-picture img');
const crop = computeCoverCrop(img.naturalWidth, img.naturalHeight, 16, 6, 30, 40);
img.style.objectPosition = toObjectPosition(img.naturalWidth, img.naturalHeight, crop);
```

---

## 5. ขนาดตายตัว (Fixed-size banner)

เครื่องมือ preview มีพรีเซ็ตขนาดพิกเซลตายตัว 2 แบบ:

| ขนาด | aspect-ratio |
|---|---|
| 1920×520 | `1920/520` |
| 700×520 | `700/520` |

ใน preview จะล็อกสัดส่วนนี้ให้เท่านั้น หากต้องการใช้จริงในระบบ ให้เพิ่ม class แยก เช่น:

```css
.hero-picture--banner-lg,
.hero-video--banner-lg { aspect-ratio: 1920/520; }

.hero-picture--banner-sm,
.hero-video--banner-sm { aspect-ratio: 700/520; }
```

---

## 6. วิธีใช้เครื่องมือ Preview (`picture-preview.html`)

1. **อัปโหลดไฟล์** — คลิกหรือลากไฟล์มาวางในกรอบ หรือกดปุ่ม "อัปโหลด / เปลี่ยนไฟล์"
   รองรับ JPEG / PNG / GIF / MP4 (ประมวลผลในเบราว์เซอร์เท่านั้น ไม่มีการอัปโหลดขึ้นเซิร์ฟเวอร์)
2. **สลับขนาดหน้าจอ** — แถบพรีเซ็ต: Auto, Mobile S 320, Mobile M 375, Mobile L 425, Tablet 768,
   Laptop 1024, Laptop L 1440, 4K 2560, 1920×520, 700×520
   - Auto = ตามความกว้างจริงของหน้าต่าง
   - พรีเซ็ตที่กว้างกว่าพื้นที่แสดงผลจะถูกย่อให้พอดี (แสดงเปอร์เซ็นต์ที่ย่อใต้กรอบ)
3. **กำหนดจุดโฟกัส** — คลิก/ลากบนภาพต้นฉบับขนาดเล็ก, กรอก X/Y (%) หรือใช้ปุ่มกริด 3×3
   กรอบสีเหลืองคือพื้นที่ที่ถูกครอปไว้จริง
4. **ดาวน์โหลดไฟล์ที่ครอปแล้ว** — ครอปตามสัดส่วนของขนาดที่เลือกอยู่ และใช้จุดโฟกัสที่กำหนด
5. **สลับภาษา** — ปุ่ม TH / EN มุมขวาบน

### การทำงานของปุ่มดาวน์โหลด

| กรณี | ผลลัพธ์ |
|---|---|
| รูปภาพ | ครอปด้วย canvas ส่งออกนามสกุลเดียวกับต้นฉบับ (`.png`, `.jpg`, `.webp`) |
| `.gif` | canvas เข้ารหัส GIF ไม่ได้ จึง fallback เป็น `.png` (เฟรมเดียว ไม่เคลื่อนไหว) |
| วิดีโอ | บันทึกเฟรมที่ครอปด้วย `MediaRecorder` ได้ `.mp4` หากเบราว์เซอร์รองรับ มิฉะนั้น fallback เป็น `.webm` |
| ขนาดตายตัว (1920×520 / 700×520) | ส่งออกขนาดพิกเซลตรงตามที่เลือก |
| พรีเซ็ตอื่น | ครอปตามสัดส่วน คงความละเอียดต้นฉบับ |

ข้อจำกัดของการส่งออกวิดีโอ:

- ต้องเล่นวิดีโอจนจบ 1 รอบเพื่อบันทึก (ใช้เวลาเท่าความยาววิดีโอ)
- ไฟล์ที่ได้ไม่มีเสียง
- ต้องใช้เบราว์เซอร์ที่รองรับ `MediaRecorder` และ `canvas.captureStream()` (Chrome / Edge / Firefox); Safari อาจมีปัญหา

---

## 7. Browser support

ฟีเจอร์ CSS ที่ใช้: `aspect-ratio`, `object-fit`, `object-position`, `<picture>`/`srcset`
และใน preview ใช้ container query units (`cqw`) กับ `ResizeObserver` — ทุกตัวรองรับในเบราว์เซอร์รุ่นปัจจุบัน
