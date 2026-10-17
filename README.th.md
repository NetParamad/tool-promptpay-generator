[English](README.md) | [ไทย](README.th.md)

# Half & Half | PromptPay

เครื่องมือหารบิลที่คำนวณยอดเงินต่อคนและสร้าง PromptPay QR Code สำหรับระบบชำระเงินของไทย

## ภาพรวม

**Half & Half | PromptPay** เป็นเว็บแอปขนาดเล็กที่ทำงานฝั่งไคลเอนต์ทั้งหมด สร้างด้วย SvelteKit ผู้ใช้กรอกยอดเงินรวม จำนวนคน และเบอร์มือถือไทย ระบบจะหารยอดให้เท่ากันทุกคน (ปัดทศนิยม 2 ตำแหน่ง) แล้วสร้าง PromptPay QR Code ที่สแกนจ่ายยอดนั้นได้ทันที

ข้อมูล QR ถูกสร้างในเบราว์เซอร์ด้วย [`promptparse`](https://www.npmjs.com/package/promptparse) (แบบ MSISDN หรือเบอร์มือถือ) แล้วเรนเดอร์เป็นภาพ SVG ด้วย [`uqr`](https://www.npmjs.com/package/uqr) ทั้งหมดนี้ทำในเครื่อง ไม่มีการต่อกับเซิร์ฟเวอร์ ค่าที่กรอกจะถูกจำไว้ใน `localStorage`

หน้าจอมีป้ายกำกับสองภาษา (อังกฤษ + ไทย) และรองรับธีมสว่าง/มืด ตัวเว็บที่ใช้งานจริงอยู่ที่ `https://workshop-svelte-half-and-half.vercel.app`

## ฟีเจอร์

- **คำนวณยอดหาร** — คำนวณจำนวนเงินต่อคนและปัดทศนิยม 2 ตำแหน่ง
- **สร้าง PromptPay QR** — สร้างข้อมูล PromptPay แบบ MSISDN (เบอร์มือถือ) ด้วย `promptparse` แล้วเรนเดอร์เป็น QR ด้วย `uqr`
- **ดาวน์โหลด** — บันทึก QR เป็น **PNG** (ขนาด 512×512 เรนเดอร์ผ่าน `<canvas>`) หรือ **SVG**
- **แชร์** — แชร์ไฟล์ PNG ผ่าน Web Share API และถ้าเครื่องไม่รองรับหรือแชร์ไม่สำเร็จ จะดาวน์โหลดให้แทน
- **ธีมสว่าง/มืด** — สลับได้ในหน้าแอป จำค่าไว้ใน `localStorage` และตั้งค่าก่อนวาดหน้าจอด้วย inline script ใน `app.html` เพื่อกันไม่ให้เห็นธีมผิดก่อนโหลดเสร็จ (FOUC)
- **ตรวจสอบข้อมูลนำเข้า** — ยอดเงินต้องมากกว่า 0 จำนวนคนต้องเป็นจำนวนเต็มตั้งแต่ 1 ขึ้นไป และเบอร์โทรต้องเป็นมือถือไทย 10 หลักที่ขึ้นต้นด้วย `0` (`0xxxxxxxxx`)
- **จำค่าที่กรอก** — ยอดเงิน จำนวนคน และเบอร์โทร ถูกบันทึกใน `localStorage` และกลับมาแสดงอีกครั้งเมื่อเข้าใช้ครั้งถัดไป
- **UI ตอบสนองและเข้าถึงได้** — ตกแต่งด้วย Tailwind CSS มี ARIA label, live region และสไตล์โฟกัสด้วยคีย์บอร์ด
- **SEO / ข้อมูลโซเชียล** — ตั้งค่าหัวข้อ คำอธิบาย คีย์เวิร์ด, Open Graph, Twitter Card และ JSON-LD structured data
- **Vercel Speed Insights** — เก็บข้อมูลประสิทธิภาพผ่าน `@vercel/speed-insights`

## เทคโนโลยีที่ใช้

| ส่วนงาน | เทคโนโลยี |
| --- | --- |
| เฟรมเวิร์ก | [Svelte 5](https://svelte.dev) + [SvelteKit 2](https://svelte.dev/docs/kit) |
| เครื่องมือ build | [Vite 7](https://vitejs.dev) |
| ภาษา | TypeScript 5 |
| ตกแต่ง | [Tailwind CSS 4](https://tailwindcss.com) (ร่วมกับ `@tailwindcss/forms`) |
| QR / การชำระเงิน | [`promptparse`](https://www.npmjs.com/package/promptparse), [`uqr`](https://www.npmjs.com/package/uqr) |
| ไอคอน | [`lucide-svelte`](https://www.npmjs.com/package/lucide-svelte) |
| วิเคราะห์การใช้งาน | [`@vercel/speed-insights`](https://www.npmjs.com/package/@vercel/speed-insights) |
| Adapter | [`@sveltejs/adapter-auto`](https://svelte.dev/docs/kit/adapter-auto) |
| ตัวจัดการแพ็กเกจ | [pnpm](https://pnpm.io) |
| Lint / จัดรูปแบบ | ESLint 9 (flat config) + Prettier 3 (พร้อมปลั๊กอิน Svelte และ Tailwind) |
| ตรวจชนิดข้อมูล | `svelte-check` |

## โครงสร้างโปรเจกต์

```
tool-promptpay-generator/
├── src/
│   ├── app.html              # โครง HTML + inline script ตั้งค่าธีม (กัน FOUC)
│   ├── app.d.ts              # ประกาศชนิดข้อมูลระดับแอป
│   ├── lib/
│   │   ├── index.ts          # barrel ของ $lib (ตอนนี้ยังเป็นไฟล์ว่าง)
│   │   └── assets/
│   │       ├── favicon.svg
│   │       └── wallet.png
│   └── routes/
│       ├── +layout.svelte    # เลย์เอาต์หลัก: favicon, CSS ทั้งหมด, Speed Insights
│       ├── +page.svelte      # แอปหลัก: คำนวณหาร, QR, ดาวน์โหลด, แชร์
│       └── layout.css        # จุดรวม Tailwind + dark variant + ฟอนต์
├── static/
│   ├── robots.txt            # อนุญาตให้ crawler ทั้งหมด
│   └── wallet.png            # รูปสำหรับแชร์โซเชียล (OG/Twitter)
├── eslint.config.js          # ตั้งค่า ESLint แบบ flat config
├── svelte.config.js          # ตั้งค่า SvelteKit (vite preprocess, adapter-auto)
├── vite.config.ts            # ตั้งค่า Vite (ปลั๊กอิน Tailwind + SvelteKit)
├── tsconfig.json             # ตั้งค่า TypeScript (extends config ที่ .svelte-kit สร้าง)
├── pnpm-workspace.yaml       # ตั้งค่า workspace / build ของ pnpm
└── package.json              # script และ dependencies
```

## เริ่มต้นใช้งาน

### สิ่งที่ต้องมีก่อน

- **Node.js** (แนะนำเวอร์ชัน LTS)
- **pnpm** — โปรเจกต์มีไฟล์ `pnpm-lock.yaml` และ `pnpm-workspace.yaml` มาพร้อมแล้ว

### ติดตั้ง

```sh
pnpm install
```

### คำสั่งที่มีให้ใช้

| คำสั่ง | คำอธิบาย |
| --- | --- |
| `pnpm run dev` | เปิด dev server ของ Vite พร้อม hot reload |
| `pnpm run build` | สร้าง build สำหรับ production |
| `pnpm run preview` | ดูตัวอย่าง production build ในเครื่อง |
| `pnpm run check` | รัน `svelte-kit sync` และ `svelte-check` เพื่อตรวจชนิด/ข้อผิดพลาดตอนคอมไพล์ |
| `pnpm run check:watch` | เหมือน `check` แต่ทำงานแบบ watch |
| `pnpm run lint` | ตรวจด้วย Prettier และ ESLint ทั่วโปรเจกต์ |
| `pnpm run format` | จัดรูปแบบโค้ดทั้งโปรเจกต์ด้วย Prettier |
| `pnpm run prepare` | `svelte-kit sync` (รันอัตโนมัติตอนติดตั้ง) |

### รันในเครื่อง

```sh
pnpm run dev
```

## วิธีใช้งาน

1. กรอก **Amount** (จำนวนเงินเป็นบาท)
2. กรอก **Number of People** (จำนวนคนที่ต้องการหารบิล)
3. กรอก **PromptPay Phone** — เบอร์มือถือไทย 10 หลัก ขึ้นต้นด้วย `0`
4. ระบบจะแสดง **จำนวนเงินต่อคน** และ **PromptPay QR Code** ให้ทันที
5. ใช้ปุ่ม **Download PNG**, **Download SVG** หรือ **Share QR** เพื่อบันทึกหรือแชร์โค้ด

## หมายเหตุ

- **ไม่ต้องมี backend** การคำนวณและสร้าง QR ทำในเบราว์เซอร์ทั้งหมด ไม่มีลอจิกหรือ API ฝั่งเซิร์ฟเวอร์
- **ไม่มี environment variable** โปรเจกต์ไม่อ่านค่าจาก `.env` การทำงานทั้งหมดอยู่ฝั่งไคลเอนต์
- **ความเป็นส่วนตัว** ยอดเงิน จำนวนคน และเบอร์โทร ถูกเก็บไว้ใน `localStorage` ของเบราว์เซอร์เท่านั้น ไม่ถูกส่งไปเซิร์ฟเวอร์
- **รูปแบบเบอร์โทร** รับเฉพาะมือถือไทย 10 หลักที่ขึ้นต้นด้วย `0` ช่องกรอกจะตัดอักขระที่ไม่ใช่ตัวเลขออกและจำกัดความยาวไม่เกิน 10 ตัวอักษร
- **การ deploy** โปรเจกต์ใช้ `@sveltejs/adapter-auto` จึง deploy ขึ้นแพลตฟอร์มอย่าง Vercel ได้โดยไม่ต้องเปลี่ยน adapter และมีการต่อ Speed Insights สำหรับ Vercel ไว้แล้ว

## สัญญาอนุญาต

TODO: โปรเจกต์นี้ยังไม่ได้ระบุสัญญาอนุญาต (License)
