<script>
	// @ts-nocheck
	import { onMount } from 'svelte';
	import { scale } from 'svelte/transition';
	import { anyId } from 'promptparse/generate';
	import { renderSVG } from 'uqr';
	import { Moon, Sun, Wallet } from 'lucide-svelte';

	let amount = $state(200);
	let person = $state(2);
	let target = $state('');

	let amountPerPerson = $derived(person > 0 ? amount / person : 0);
	let cleanTarget = $derived(target.replace(/\D/g, ''));
	let isValidPhone = $derived(cleanTarget.length === 10);

	let qrImage = $derived(
		cleanTarget
			? btoa(renderSVG(anyId({ type: 'MSISDN', target: cleanTarget, amount: amountPerPerson })))
			: ''
	);

	let theme = $state('default');

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

		if (savedAmount) amount = Number(savedAmount);
		if (savedPerson) person = Number(savedPerson);
		if (savedTarget) target = savedTarget;
	});

	$effect(() => {
		localStorage.setItem('amount', String(amount));
		localStorage.setItem('person', String(person));
		localStorage.setItem('target', target);
	});

	function onlyNumber(e) {
		e.target.value = e.target.value.replace(/\D/g, '');
	}

	function applyTheme(value) {
		document.documentElement.classList.toggle('dark', value === 'dark');
	}

	function toggleTheme() {
		theme = theme === 'dark' ? 'light' : 'dark';
		localStorage.setItem('theme', theme);
		applyTheme(theme);
	}

	function downloadSVG() {
		const svg = renderSVG(anyId({ type: 'MSISDN', target: cleanTarget, amount: amountPerPerson }));
		const blob = new Blob([svg], { type: 'image/svg+xml;charset=utf-8;' });
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = `promptpay-${cleanTarget}.svg`;
		a.click();
		URL.revokeObjectURL(url);
	}

	function downloadPNG() {
		const svg = renderSVG(anyId({ type: 'MSISDN', target: cleanTarget, amount: amountPerPerson }));
		const img = new Image();
		const svgBlob = new Blob([svg], { type: 'image/svg+xml;charset=utf-8;' });
		const url = URL.createObjectURL(svgBlob);

		img.onload = () => {
			const canvas = document.createElement('canvas');
			canvas.width = 512;
			canvas.height = 512;

			const ctx = canvas.getContext('2d');
			ctx.fillStyle = '#ffffff';
			ctx.fillRect(0, 0, canvas.width, canvas.height);
			ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

			canvas.toBlob((blob) => {
				const pngUrl = URL.createObjectURL(blob);
				const a = document.createElement('a');
				a.href = pngUrl;
				a.download = `promptpay-${cleanTarget}.png`;
				a.click();
				URL.revokeObjectURL(pngUrl);
			});

			URL.revokeObjectURL(url);
		};

		img.src = url;
	}

	async function shareQR() {
		if (!isValidPhone) return;

		const svg = renderSVG(anyId({ type: 'MSISDN', target: cleanTarget, amount: amountPerPerson }));
		const img = new Image();
		const svgBlob = new Blob([svg], { type: 'image/svg+xml;charset=utf-8;' });
		const url = URL.createObjectURL(svgBlob);

		img.onload = async () => {
			const canvas = document.createElement('canvas');
			canvas.width = 512;
			canvas.height = 512;
			const ctx = canvas.getContext('2d');
			ctx.fillStyle = '#ffffff';
			ctx.fillRect(0, 0, canvas.width, canvas.height);
			ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

			canvas.toBlob(async (blob) => {
				const file = new File([blob], `promptpay-${cleanTarget}.png`, { type: 'image/png' });

				if (navigator.canShare && navigator.canShare({ files: [file] })) {
					await navigator.share({
						files: [file],
						title: 'PromptPay QR',
						text: `TH: รหัส QR PromptPay สำหรับหมายเลข ${cleanTarget}\nจำนวนเงิน ฿${amountPerPerson.toFixed(2)}\n\nEN: PromptPay QR for phone ${cleanTarget}\nAmount ฿${amountPerPerson.toFixed(2)}`
					});
				} else {
					// fallback desktop: ดาวน์โหลด
					const pngUrl = URL.createObjectURL(blob);
					const a = document.createElement('a');
					a.href = pngUrl;
					a.download = `promptpay-${cleanTarget}.png`;
					a.click();
					URL.revokeObjectURL(pngUrl);
				}
			});

			URL.revokeObjectURL(url);
		};

		img.src = url;
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
	<meta property="og:url" content="https://workshop-svelte-half-and-half.vercel.app/" />
	<meta property="og:image" content="/wallet.png" />

	<!-- Twitter Card -->
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:title" content="Half & Half | PromptPay" />
	<meta
		name="twitter:description"
		content="คำนวณจำนวนเงินต่อคนและสร้าง PromptPay QR Code ดาวน์โหลด PNG/SVG"
	/>
	<meta name="twitter:image" content="/wallet.png" />

	<!-- Favicon -->
	<link rel="icon" href="/favicon.ico" type="image/x-icon" />

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
					bind:value={amount}
					oninput={(e) => {
						onlyNumber(e);
						amount = Number(e.target.value || 0);
					}}
					class="w-full rounded-md border border-zinc-300 bg-white px-3 py-2 text-sm focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:bg-zinc-800"
					aria-label="Amount in Thai Baht"
				/>
			</div>

			<!-- People -->
			<div class="space-y-2">
				<label for="people" class="text-sm text-zinc-600 dark:text-zinc-300">
					Number of People
				</label>
				<span class="text-xs text-zinc-600 dark:text-zinc-400">(จำนวนคน)</span>
				<input
					id="people"
					type="text"
					inputmode="numeric"
					pattern="[0-9]*"
					bind:value={person}
					oninput={(e) => {
						onlyNumber(e);
						person = Number(e.target.value || 0);
					}}
					class="w-full rounded-md border border-zinc-300 bg-white px-3 py-2 text-sm focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:bg-zinc-800"
					aria-label="Number of people"
				/>
			</div>

			<!-- Phone -->
			<div class="space-y-2">
				<label for="phone" class="text-sm text-zinc-600 dark:text-zinc-300">PromptPay Phone</label>
				<div class="flex justify-between text-xs">
					<span id="phone-help" class="text-zinc-600 dark:text-zinc-400"
						>(ใส่เบอร์โทรศัพท์ 10 หลัก)</span
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
					maxlength="10"
					bind:value={target}
					oninput={(e) => {
						onlyNumber(e);
						target = e.target.value.slice(0, 10);
					}}
					placeholder="08xxxxxxxx"
					class="w-full rounded-md border border-zinc-300 bg-white px-3 py-2 text-sm focus:outline focus:outline-2 focus:outline-blue-500 dark:border-zinc-700 dark:bg-zinc-800"
					aria-describedby="phone-help"
				/>
			</div>

			{#if isValidPhone}
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
