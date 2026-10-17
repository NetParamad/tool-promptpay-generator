# Half & Half | PromptPay

แอปคนละครึ่งสำหรับคำนวณจำนวนเงินต่อคนและสร้าง PromptPay QR Code

## Features

- คำนวณยอดต่อคน (ปัดทศนิยม 2 ตำแหน่ง)
- สร้าง PromptPay QR (MSISDN) ด้วย `promptparse` + `uqr`
- ดาวน์โหลด PNG / SVG, แชร์ผ่าน Web Share API (fallback เป็นดาวน์โหลด)
- Dark/Light theme (จำค่าผ่าน `localStorage`, กัน FOUC ด้วย inline script ใน `app.html`)
- Validate: ยอด > 0, จำนวนคน ≥ 1, เบอร์ `0xxxxxxxxx` (10 หลัก)

## Developing

```sh
pnpm install
pnpm run dev
```

## Checks

```sh
pnpm run check
pnpm run lint
pnpm run build
```
