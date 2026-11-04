[EN](README.md) | [TH](README.th.md)

<h1 align="center">Half &amp; Half | PromptPay</h1>

<p align="center">
  A split-the-bill tool that calculates the amount per person and generates a PromptPay QR Code.
</p>

<p align="center">
  <a href="https://workshop-svelte-half-and-half.vercel.app"><strong>workshop-svelte-half-and-half.vercel.app</strong></a>
</p>

<p align="center">
  <a href="#overview">Overview</a> ·
  <a href="#features">Features</a> ·
  <a href="#tech-stack">Tech stack</a> ·
  <a href="#project-structure">Structure</a> ·
  <a href="#getting-started">Getting started</a> ·
  <a href="#environment-variables">Environment</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#notes">Notes</a>
</p>

---

## Overview

**Half & Half | PromptPay** is a small, client-side web application built with SvelteKit. You enter a total amount, the number of people, and a Thai mobile number; the app divides the total evenly (rounded to two decimal places) and renders a PromptPay QR Code that can be scanned to pay that amount.

The QR payload is generated in the browser using [`promptparse`](https://www.npmjs.com/package/promptparse) (MSISDN / mobile-number PromptPay) and rendered as an SVG via [`uqr`](https://www.npmjs.com/package/uqr). No backend is involved: every calculation and QR generation happens locally in the browser, and inputs are remembered in `localStorage`.

The interface uses English and Thai labels and supports light and dark themes. The live site is deployed at `https://workshop-svelte-half-and-half.vercel.app`.

## Features

- **Split calculation** — computes the amount per person and rounds to two decimal places.
- **PromptPay QR generation** — builds an MSISDN (mobile number) PromptPay payload with `promptparse` and renders it as a QR image with `uqr`.
- **Download** — export the QR code as **PNG** (512×512, rasterised via `<canvas>`) or **SVG**.
- **Share** — uses the Web Share API to share the PNG file; falls back to a download when sharing is unavailable or fails.
- **Dark / Light theme** — toggleable in the UI, remembered in `localStorage`, and applied before paint with an inline script in `app.html` to avoid a flash of the wrong theme (FOUC).
- **Input validation** — amount must be greater than 0, the number of people must be an integer ≥ 1, and the phone number must be a 10-digit Thai mobile number starting with `0` (`0xxxxxxxxx`).
- **Input persistence** — amount, number of people, and phone number are saved to `localStorage` and restored on the next visit.
- **Responsive, accessible UI** — Tailwind CSS styling, ARIA labels, live regions, and keyboard-focus styles.
- **SEO / social metadata** — page title, description, keywords, Open Graph, Twitter Card, and JSON-LD structured data.
- **Vercel Speed Insights** — performance telemetry injected through `@vercel/speed-insights`.

## Tech stack

| Area | Technology |
| --- | --- |
| Framework | [Svelte 5](https://svelte.dev) + [SvelteKit 2](https://svelte.dev/docs/kit) |
| Build tool | [Vite 7](https://vitejs.dev) |
| Language | TypeScript 5 |
| Styling | [Tailwind CSS 4](https://tailwindcss.com) (+ `@tailwindcss/forms`) |
| QR / payment | [`promptparse`](https://www.npmjs.com/package/promptparse), [`uqr`](https://www.npmjs.com/package/uqr) |
| Icons | [`lucide-svelte`](https://www.npmjs.com/package/lucide-svelte) |
| Analytics | [`@vercel/speed-insights`](https://www.npmjs.com/package/@vercel/speed-insights) |
| Adapter | [`@sveltejs/adapter-auto`](https://svelte.dev/docs/kit/adapter-auto) |
| Package manager | [pnpm](https://pnpm.io) (`pnpm-lock.yaml`) |
| Lint / format | ESLint 9 (flat config) + Prettier 3 (with Svelte and Tailwind plugins) |
| Type checking | `svelte-check` |

## Project structure

```
tool-promptpay-generator/
├── src/
│   ├── app.html              # HTML shell + inline theme script (prevents FOUC)
│   ├── app.d.ts              # Global app type declarations
│   ├── lib/
│   │   ├── index.ts          # $lib barrel (currently a placeholder)
│   │   └── assets/
│   │       ├── favicon.svg
│   │       └── wallet.png
│   └── routes/
│       ├── +layout.svelte    # Root layout: favicon, global CSS, Speed Insights
│       ├── +page.svelte      # Main app: split logic, QR, downloads, sharing
│       └── layout.css        # Tailwind entry + dark variant + font
├── static/
│   ├── robots.txt            # Allows all crawlers
│   └── wallet.png            # Social share (OG/Twitter) image
├── eslint.config.js          # ESLint flat config
├── svelte.config.js          # SvelteKit config (vite preprocess, adapter-auto)
├── vite.config.ts            # Vite config (Tailwind + SvelteKit plugins)
├── tsconfig.json             # TypeScript config (extends generated .svelte-kit config)
├── pnpm-workspace.yaml       # pnpm workspace / build settings
└── package.json              # Scripts and dependencies
```

## Getting started

Requirements: **Node.js >=22.23.4** (`.nvmrc` pins 24.22.1) and **pnpm**. `.npmrc` sets `engine-strict=true`, so an unsupported Node version fails the install.

```bash
pnpm install
pnpm dev
```

Open the URL printed by Vite. `pnpm build` produces a production build and `pnpm preview` serves it locally.

### Scripts

| Command | Description |
| --- | --- |
| `pnpm run dev` | Start the Vite dev server with hot reload |
| `pnpm run build` | Produce a production build |
| `pnpm run preview` | Preview the production build locally |
| `pnpm run check` | Run `svelte-kit sync` and `svelte-check` |
| `pnpm run check:watch` | Same as `check`, in watch mode |
| `pnpm run lint` | Prettier check + ESLint across the project |
| `pnpm run format` | Format the codebase with Prettier |
| `pnpm run prepare` | `svelte-kit sync` (runs automatically on install) |

## Environment variables

None. The project reads no `.env` values; behaviour is entirely client-side.

## Usage

1. Enter the **Amount** (number of Thai Baht).
2. Enter the **Number of People** to split the bill between.
3. Enter the **PromptPay Phone** — a 10-digit Thai mobile number starting with `0`.
4. The app immediately shows the **amount per person** and the **PromptPay QR Code**.
5. Use **Download PNG**, **Download SVG**, or **Share QR** to export or share the code.

## Notes

- **No backend** — all computation and QR generation run in the browser; there is no server-side logic or API.
- **Privacy** — amount, number of people, and phone number are stored only in your browser's `localStorage` and are not sent to a server.
- **Phone format** — only 10-digit Thai mobile numbers beginning with `0` are accepted; the field strips non-digits and caps input at 10 characters.
- **Deployment** — the project uses `@sveltejs/adapter-auto`, so it deploys to platforms such as Vercel without adapter changes. Speed Insights is wired in for Vercel.
- **License** — MIT (see [LICENSE](LICENSE)).
