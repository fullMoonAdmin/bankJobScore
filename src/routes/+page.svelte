<script>
	import { onMount } from 'svelte';

	// Score API (escapeRoomMongoAPI). Change here if the server moves.
	const API_URL = 'http://192.168.3.54:8000/records/';
	// How many crews fit on the board.
	const BOARD_SIZE = 8;
	// Reload interval for new scores (ms).
	const REFRESH_MS = 60000;

	const STAMP = '#a3312b';

	let leaders = [];
	let recent = null;
	let error = '';
	let scale = 1;

	const money = (n) => Number(n || 0).toLocaleString('en-US');

	async function load() {
		try {
			const res = await fetch(API_URL);
			if (!res.ok) throw new Error(`API returned ${res.status}`);
			const records = await res.json();
			recent = records.length ? records[records.length - 1] : null;
			leaders = [...records].sort((a, b) => b.score - a.score).slice(0, BOARD_SIZE);
			error = '';
		} catch (e) {
			error = 'Vault records unavailable';
			console.error(e);
		}
	}

	function fit() {
		scale = Math.min(window.innerWidth / 1280, window.innerHeight / 720);
	}

	onMount(() => {
		fit();
		load();
		window.addEventListener('resize', fit);
		const timer = setInterval(load, REFRESH_MS);
		return () => {
			clearInterval(timer);
			window.removeEventListener('resize', fit);
		};
	});
</script>

<svelte:head>
	<title>Merchant's Bank — Bank Heist Leaderboard</title>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;800&family=Libre+Caslon+Text:wght@400;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="screen">
	<div class="stage" style="transform: scale({scale})">
		<div class="frame">
			<div class="frame-inner">
				<header>
					<svg class="stamp" width="104" height="104" viewBox="0 0 104 104" role="img" aria-label="Merchant's Bank paid stamp" style="color: {STAMP}">
						<defs>
							<path id="stampRing" d="M52 52 m-36 0 a36 36 0 1 1 72 0 a36 36 0 1 1 -72 0" />
						</defs>
						<circle cx="52" cy="52" r="49" fill="none" stroke="currentColor" stroke-width="3.5" />
						<circle cx="52" cy="52" r="27" fill="none" stroke="currentColor" stroke-width="2" />
						<text fill="currentColor" font-family="Libre Caslon Text, serif" font-size="9" font-weight="700" letter-spacing="1">
							<textPath href="#stampRing" textLength="220" lengthAdjust="spacing">MERCHANT'S BANK ★ KANSAS CITY ★</textPath>
						</text>
						<rect x="18" y="41" width="68" height="22" fill="#efe8d2" stroke="currentColor" stroke-width="2.5" />
						<text x="52" y="58" text-anchor="middle" fill="currentColor" font-family="Playfair Display, serif" font-size="18" font-weight="800" letter-spacing="3">PAID</text>
					</svg>

					<div class="titles">
						<div class="bank">Merchant's Bank</div>
						<h1>Bank Heist Leaderboard</h1>
						<div class="tagline">Full Moon Escape · Payable to the boldest crew in Kansas City</div>
					</div>

					<svg class="seal" width="84" height="84" viewBox="0 0 84 84" aria-hidden="true" fill="none" stroke="currentColor">
						<circle cx="42" cy="42" r="39" stroke-width="2" />
						<circle cx="42" cy="42" r="33" stroke-width="1" stroke-dasharray="2 3" />
						<circle cx="42" cy="42" r="24" stroke-width="2" />
						<path d="M42 26l4.7 9.6 10.6 1.5-7.7 7.5 1.8 10.5L42 50.1l-9.4 5 1.8-10.5-7.7-7.5 10.6-1.5z" stroke-width="2" stroke-linejoin="round" />
					</svg>
				</header>

				{#if recent}
					<section class="deposit">
						<div class="label">Latest Deposit</div>
						<div class="team">{recent.teamName}</div>
						<div class="figures">
							<span class="take">${money(recent.score)}</span>
							<span>{recent.timeInRoom}</span>
							<span>{recent.dateOfGame}</span>
						</div>
					</section>
				{/if}

				<section class="board">
					<div class="row head">
						<span>No.</span><span>Crew</span><span class="r">Take</span><span class="r">Time</span><span class="r">Date</span>
					</div>
					{#if error}
						<div class="message">{error}</div>
					{:else if !leaders.length}
						<div class="message">Opening the vault…</div>
					{:else}
						{#each leaders as record, i}
							<div class="row" class:top={i < 3}>
								<span class="rank">{i + 1}</span>
								<span class="name">{record.teamName}</span>
								<span class="r take">${money(record.score)}</span>
								<span class="r">{record.timeInRoom}</span>
								<span class="r">{record.dateOfGame}</span>
							</div>
						{/each}
					{/if}
				</section>
			</div>
		</div>
	</div>
</div>

<style>
	:global(body) { margin: 0; background: #efe8d2; overflow: hidden; }
	.screen { width: 100vw; height: 100vh; display: flex; align-items: center; justify-content: center; background: #efe8d2; }
	.stage { width: 1280px; height: 720px; flex: none; transform-origin: center; box-sizing: border-box; padding: 18px; font-family: 'Libre Caslon Text', serif; color: #1d2b22; }
	.frame { height: 100%; box-sizing: border-box; border: 3px solid #1f4a34; padding: 6px; }
	.frame-inner { height: 100%; box-sizing: border-box; border: 1px solid #1f4a34; padding: 26px 44px; display: flex; flex-direction: column; gap: 20px; }
	header { display: flex; align-items: center; gap: 28px; }
	.stamp { flex: none; transform: rotate(-14deg); }
	.seal { flex: none; color: #1f4a34; }
	.titles { flex-grow: 1; display: flex; flex-direction: column; align-items: center; gap: 4px; color: #1f4a34; }
	.bank { font-size: 15px; letter-spacing: 0.35em; text-transform: uppercase; font-weight: 700; }
	h1 { margin: 0; font-family: 'Playfair Display', serif; font-size: 50px; font-weight: 800; letter-spacing: 0.02em; }
	.tagline { font-size: 15px; font-style: italic; color: #3d4a40; }
	.deposit { display: flex; align-items: center; gap: 28px; padding: 12px 24px; background: #e4dbbd; border: 1px dashed #1f4a34; }
	.deposit .label { width: 170px; font-size: 14px; letter-spacing: 0.25em; text-transform: uppercase; font-weight: 700; color: #1f4a34; }
	.deposit .team { flex-grow: 1; font-family: 'Playfair Display', serif; font-size: 28px; font-weight: 700; }
	.figures { display: flex; gap: 36px; font-size: 22px; font-variant-numeric: tabular-nums; }
	.take { font-weight: 700; color: #1f4a34; }
	.board { display: flex; flex-direction: column; flex-grow: 1; }
	.row { display: grid; grid-template-columns: 80px minmax(0, 1fr) 210px 130px 160px; align-items: center; height: 44px; padding: 0 24px; border-bottom: 1px dotted #8a8e7a; font-size: 22px; font-variant-numeric: tabular-nums; }
	.row.head { height: auto; padding-bottom: 8px; font-size: 13px; letter-spacing: 0.25em; text-transform: uppercase; font-weight: 700; color: #1f4a34; border-bottom: 2px solid #1f4a34; }
	.row.top { background: #e7dfc4; font-weight: 700; }
	.rank { font-family: 'Playfair Display', serif; font-weight: 800; color: #1f4a34; }
	.name { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
	.r { text-align: right; }
	.message { padding: 24px; font-size: 22px; font-style: italic; color: #3d4a40; }
</style>
