<script lang="ts">
	import { onMount } from 'svelte';
	import { scale } from 'svelte/transition';
	import { anyId } from 'promptparse/generate';
	import { renderSVG } from 'uqr';
	import { Moon, Sun, Wallet } from 'lucide-svelte';

	const SITE_URL = 'https://workshop-svelte-half-and-half.vercel.app';

	// Use strings for inputs to avoid number/text binding coercion issues.
	let amountStr = $state('200');
	let personStr = $state('2');
	let target = $state('');
	let theme: 'light' | 'dark' = $state('light');
	let hydrated = $state(false);

	let amount = $derived(Number(amountStr) || 0);
	let person = $derived(Number(personStr) || 0);
	let amountValid = $derived(Number.isFinite(amount) && amount > 0);
	let personValid = $derived(Number.isInteger(person) && person >= 1);
	let cleanTarget = $derived(target.replace(/\D/g, '').slice(0, 10));
	let isValidPhone = $derived(/^0\d{9}$/.test(cleanTarget));

	let amountPerPerson = $derived(
		amountValid && personValid ? Math.round((amount / person) * 100) / 100 : 0
	);
	let canShowQr = $derived(isValidPhone && amountValid && personValid);

	// Only build the payload when inputs are valid; anyId can throw on bad input.
	let qrSvg = $derived.by(() => {
		if (!canShowQr) return '';
		try {
			return renderSVG(anyId({ type: 'MSISDN', target: cleanTarget, amount: amountPerPerson }));
		} catch {
			return '';
		}
	});

	function svgToBase64(svg: string): string {
		const bytes = new TextEncoder().encode(svg);
		let binary = '';
		for (const b of bytes) binary += String.fromCharCode(b);
		return btoa(binary);
	}

	let qrImage = $derived(qrSvg ? svgToBase64(qrSvg) : '');

	function buildQrSvg(): string {
		return renderSVG(anyId({ type: 'MSISDN', target: cleanTarget, amount: amountPerPerson }));
	}

	function svgToPngBlob(svg: string): Promise<Blob | null> {
		return new Promise((resolve) => {
			const svgBlob = new Blob([svg], { type: 'image/svg+xml;charset=utf-8;' });
			const url = URL.createObjectURL(svgBlob);
			const img = new Image();
			img.onload = () => {
				URL.revokeObjectURL(url);
				const canvas = document.createElement('canvas');
				canvas.width = 512;
				canvas.height = 512;
				const ctx = canvas.getContext('2d');
				if (!ctx) {
					resolve(null);
					return;
				}
				ctx.fillStyle = '#ffffff';
				ctx.fillRect(0, 0, canvas.width, canvas.height);
				ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
				canvas.toBlob((blob) => resolve(blob), 'image/png');
			};
			img.onerror = () => {
				URL.revokeObjectURL(url);
				resolve(null);
			};
			img.src = url;
		});
	}

	function triggerDownload(url: string, filename: string) {
		const a = document.createElement('a');
		a.href = url;
		a.download = filename;
		document.body.appendChild(a);
		a.click();
		a.remove();
		// Revoke async so the download has time to start.
		setTimeout(() => URL.revokeObjectURL(url), 1000);
	}

	onMount(() => {
		const saved = localStorage.getItem('theme');
		theme =
			saved === 'dark' || saved === 'light'
				? saved
				: window.matchMedia('(prefers-color-scheme: dark)').matches
					? 'dark'
					: 'light';

		applyTheme(theme);

		const savedAmount = localStorage.getItem('amount');
		const savedPerson = localStorage.getItem('person');
		const savedTarget = localStorage.getItem('target');

		if (savedAmount !== null && /^\d+$/.test(savedAmount)) amountStr = savedAmount;
		if (savedPerson !== null && /^\d+$/.test(savedPerson)) personStr = savedPerson;
		if (savedTarget !== null) target = savedTarget.replace(/\D/g, '').slice(0, 10);

		hydrated = true;
	});

	$effect(() => {
		if (!hydrated) return;
		localStorage.setItem('amount', amountStr);
		localStorage.setItem('person', personStr);
		localStorage.setItem('target', target);
	});

	function applyTheme(value: 'light' | 'dark') {
		document.documentElement.classList.toggle('dark', value === 'dark');
	}

	function toggleTheme() {
		theme = theme === 'dark' ? 'light' : 'dark';
		localStorage.setItem('theme', theme);
		applyTheme(theme);
	}

	function downloadSVG() {
		if (!canShowQr) return;
		try {
			const svg = buildQrSvg();
			const blob = new Blob([svg], { type: 'image/svg+xml;charset=utf-8;' });
			triggerDownload(URL.createObjectURL(blob), `promptpay-${cleanTarget}.svg`);
		} catch {
			// Invalid payload — nothing to download.
		}
	}

	async function downloadPNG() {
		if (!canShowQr) return;
		try {
			const blob = await svgToPngBlob(buildQrSvg());
			if (!blob) return;
			triggerDownload(URL.createObjectURL(blob), `promptpay-${cleanTarget}.png`);
		} catch {
			// Invalid payload or canvas failure — nothing to download.
		}
	}

	async function shareQR() {
		if (!canShowQr) return;
		try {
			const blob = await svgToPngBlob(buildQrSvg());
			if (!blob) return;
			const file = new File([blob], `promptpay-${cleanTarget}.png`, { type: 'image/png' });

			if (navigator.canShare && navigator.canShare({ files: [file] })) {
				try {
					await navigator.share({
						files: [file],
						title: 'PromptPay QR',
						text: `TH: รหัส QR PromptPay สำหรับหมายเลข ${cleanTarget}\nจำนวนเงิน ฿${amountPerPerson.toFixed(2)}\n\nEN: PromptPay QR for phone ${cleanTarget}\nAmount ฿${amountPerPerson.toFixed(2)}`
					});
				} catch (e) {
					// AbortError = user cancelled, ignore. Other errors fall back to download.
					if (e instanceof DOMException && e.name === 'AbortError') return;
					triggerDownload(URL.createObjectURL(blob), `promptpay-${cleanTarget}.png`);
				}
			} else {
				// Fallback desktop: download
				triggerDownload(URL.createObjectURL(blob), `promptpay-${cleanTarget}.png`);
			}
		} catch {
			// Invalid payload — nothing to share.
		}
	}
</script>

<svelte:head>
	<title>Half & Half | PromptPay</title>
	<meta
		name="description"
		content="Half & Half แอปคนละครึ่งสำหรับคำนวณจำนวนเงินต่อคนและสร้าง PromptPay QR Code แบบง่ายๆ รองรับดาวน์โหลด PNG และ SVG"
	/>
	<meta
		name="keywords"
		content="PromptPay, คนละครึ่ง, QR Code, คำนวณเงิน, download QR, generate QR, ไทย"
	/>
	<meta name="author" content="netpramad" />
	<meta name="robots" content="index, follow" />

	<!-- Open Graph -->
	<meta property="og:title" content="Half & Half | PromptPay" />
	<meta
		property="og:description"
		content="แอปคนละครึ่งสำหรับคำนวณจำนวนเงินต่อคนและสร้าง PromptPay QR Code แบบง่ายๆ"
	/>
	<meta property="og:type" content="website" />
	<meta property="og:url" content="{SITE_URL}/" />
	<meta property="og:image" content="{SITE_URL}/wallet.png" />

	<!-- Twitter Card -->
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:title" content="Half & Half | PromptPay" />
	<meta
		name="twitter:description"
		content="คำนวณจำนวนเงินต่อคนและสร้าง PromptPay QR Code ดาวน์โหลด PNG/SVG"
	/>
	<meta name="twitter:image" content="{SITE_URL}/wallet.png" />

	<!-- JSON-LD Structured Data -->
	<script type="application/ld+json">
		{
			"@context": "https://schema.org",
			"@type": "WebApplication",
			"name": "Half & Half",
			"url": "https://workshop-svelte-half-and-half.vercel.app/",
			"description": "แอปคนละครึ่งสำหรับคำนวณจำนวนเงินต่อคนและสร้าง PromptPay QR Code แบบง่ายๆ รองรับ PNG และ SVG",
			"applicationCategory": "FinanceApplication",
			"operatingSystem": "Web"
		}
	</script>
</svelte:head>

<main
	class="flex min-h-screen items-center justify-center bg-zinc-50 p-4 text-zinc-900 transition-colors duration-300 dark:bg-zinc-950 dark:text-zinc-100"
>
	<div
		class="w-full max-w-md rounded-xl border border-zinc-300 bg-white shadow-sm transition-colors duration-300 dark:border-zinc-800 dark:bg-zinc-900"
	>
		<div class="space-y-6 p-6">
			<header class="flex items-center justify-between">
				<div class="flex items-center gap-2">
					<Wallet class="text-lg" aria-hidden="true" />
					<h1 class="text-lg font-semibold tracking-tight uppercase">Half & Half</h1>
					<span
						class="rounded-full border border-zinc-300 px-2 py-0.5 text-xs text-zinc-500 dark:border-zinc-700 dark:text-zinc-400"
					>
						คนละครึ่ง
					</span>
				</div>

				<button
					onclick={toggleTheme}
					aria-label={`Switch to ${theme === 'dark' ? 'light' : 'dark'} mode`}
					class="rounded-md border border-zinc-300 p-2 text-xs text-zinc-600 transition hover:bg-zinc-100 focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:text-zinc-300 dark:hover:bg-zinc-800"
				>
					{#if theme === 'dark'}
						<Moon class="h-4 w-4" aria-hidden="true" />
					{:else}
						<Sun class="h-4 w-4" aria-hidden="true" />
					{/if}
				</button>
			</header>

			<!-- Amount -->
			<div class="space-y-2">
				<label for="amount" class="text-sm text-zinc-600 dark:text-zinc-300">
					Amount <span class="text-xs text-zinc-600 dark:text-zinc-400">(จำนวนเงิน)</span>
				</label>
				<input
					id="amount"
					type="text"
					inputmode="numeric"
					pattern="[0-9]*"
					value={amountStr}
					oninput={(e) => {
						amountStr = e.currentTarget.value.replace(/\D/g, '');
					}}
					class="w-full rounded-md border border-zinc-300 bg-white px-3 py-2 text-sm focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:bg-zinc-800"
					aria-label="Amount in Thai Baht"
				/>
				{#if !amountValid}
					<p class="text-xs text-red-500">กรุณาใส่จำนวนเงินมากกว่า 0</p>
				{/if}
			</div>

			<!-- People -->
			<div class="space-y-2">
				<label for="people" class="text-sm text-zinc-600 dark:text-zinc-300">
					Number of People
					<span class="text-xs text-zinc-600 dark:text-zinc-400">(จำนวนคน)</span>
				</label>
				<input
					id="people"
					type="text"
					inputmode="numeric"
					pattern="[0-9]*"
					value={personStr}
					oninput={(e) => {
						personStr = e.currentTarget.value.replace(/\D/g, '');
					}}
					class="w-full rounded-md border border-zinc-300 bg-white px-3 py-2 text-sm focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:bg-zinc-800"
					aria-label="Number of people"
				/>
				{#if !personValid}
					<p class="text-xs text-red-500">กรุณาใส่จำนวนคนตั้งแต่ 1 ขึ้นไป</p>
				{/if}
			</div>

			<!-- Phone -->
			<div class="space-y-2">
				<label for="phone" class="text-sm text-zinc-600 dark:text-zinc-300">PromptPay Phone</label>
				<div class="flex justify-between text-xs">
					<span id="phone-help" class="text-zinc-600 dark:text-zinc-400"
						>(ใส่เบอร์โทรศัพท์ 10 หลัก ขึ้นต้นด้วย 0)</span
					>
					<span class={cleanTarget.length === 10 ? 'text-green-500' : 'text-zinc-600'}>
						{cleanTarget.length}/10
					</span>
				</div>
				<input
					id="phone"
					type="tel"
					inputmode="numeric"
					pattern="[0-9]*"
					maxlength={10}
					value={target}
					oninput={(e) => {
						target = e.currentTarget.value.replace(/\D/g, '').slice(0, 10);
					}}
					placeholder="08xxxxxxxx"
					class="w-full rounded-md border border-zinc-300 bg-white px-3 py-2 text-sm focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:bg-zinc-800"
					aria-describedby="phone-help"
				/>
			</div>

			{#if canShowQr && qrSvg}
				<div
					class="space-y-4 text-center"
					in:scale={{ start: 0.95, duration: 200 }}
					aria-live="polite"
				>
					<p class="text-xs tracking-wide text-zinc-500 uppercase dark:text-zinc-300">
						Amount per person <span class="text-zinc-600 dark:text-zinc-400">(จำนวนเงินต่อคน)</span>
					</p>
					<p class="text-2xl font-bold">฿{amountPerPerson.toFixed(2)}</p>

					<div class="flex justify-center">
						<div
							class="rounded-lg border border-zinc-300 bg-white p-3 shadow-sm dark:border-zinc-700"
						>
							<img
								src={'data:image/svg+xml;base64,' + qrImage}
								alt={`PromptPay QR for phone ${cleanTarget}, amount ฿${amountPerPerson.toFixed(2)}`}
								class="h-40 w-40"
							/>
						</div>
					</div>
				</div>

				<div class="flex justify-center gap-2">
					<button
						onclick={downloadPNG}
						aria-label="Download QR as PNG"
						class="rounded-md border border-zinc-300 px-3 py-1 text-xs text-zinc-700 transition hover:bg-zinc-100 focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:text-zinc-300 dark:hover:bg-zinc-800"
					>
						Download PNG
					</button>

					<button
						onclick={downloadSVG}
						aria-label="Download QR as SVG"
						class="rounded-md border border-zinc-300 px-3 py-1 text-xs text-zinc-700 transition hover:bg-zinc-100 focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:text-zinc-300 dark:hover:bg-zinc-800"
					>
						Download SVG
					</button>
					<button
						onclick={shareQR}
						aria-label="Share QR Code"
						class="rounded-md border border-zinc-300 px-3 py-1 text-xs text-zinc-700 transition hover:bg-zinc-100 focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:text-zinc-300 dark:hover:bg-zinc-800"
					>
						Share QR
					</button>
				</div>
			{/if}

			<footer>
				<p class="text-center text-xs text-zinc-500">netpramad © 2025 All rights reserved</p>
			</footer>
		</div>
	</div>
</main>
