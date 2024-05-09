<script>
	import { onMount } from 'svelte';

	onMount(() => {
		setInterval('window.location.reload();', 60000);
	});

	async function loadEscapeData() {
		const res = await fetch('http://192.168.3.54:8000/records/');
		const records = await res.json();

		if (!res.ok) {
			throw new Error("Something's wrong");
		}
		return records.sort((a, b) => b.score - a.score);
	}

	const escapeRecordsPromise = loadEscapeData();
</script>

<div class="title">Full Moon Escape</div>
<div class="header">The Bank Heist High Scores</div>
{#await escapeRecordsPromise}
	<p>Loading...</p>
{:then records}
	<table class="recordTable">
		<tr>
			<th class="table_header">Team Name</th>
			<th class="table_header">Time In Room (Min:Sec)</th>
			<th class="table_header">Score</th>
		</tr>
		{#each records as record}
			<tr>
				<td>{record.teamName}</td>
				<td>{record.timeInRoom}</td>
				<td class="score">{record.score}</td>
			</tr>
		{/each}
	</table>
{/await}

<style>
	:global(*) {
		background-color: black;
		font-family: Georgia, serif;
		margin: 0;
		color: whitesmoke;
	}
	* {
		margin: 0%;
	}
	td {
		text-align: center;
	}
	.table_header {
		border: 1px solid red;
	}
	.header {
		font-size: 4rem;
		background-color: red;
		text-align: center;
	}
	.title {
		font-size: 5rem;
		background-color: red;
		text-align: center;
	}

	.recordTable {
		min-width: 100%;
		font-size: 2rem;
		border-bottom: 1px solid red;
		padding: 4px;
	}
</style>
