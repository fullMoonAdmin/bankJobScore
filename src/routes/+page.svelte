<script>
	import { onMount } from 'svelte';
	import logo from '../images/fullmoonescapelogo.png';
	import seperator from '../images/seperator.png';

	onMount(() => {
		setInterval('window.location.reload();', 60000);
	});

	async function loadEscapeData() {
		const res = await fetch('http://192.168.3.54:8000/records/');
		const records = await res.json();

		if (!res.ok) {
			throw new Error("Something's wrong");
		}
		console.log(records.sort((a, b) => b.score - a.score));
		return records.sort((a, b) => b.score - a.score);
	}
	async function loadMostRecentRecord() {
		const res = await fetch('http://192.168.3.54:8000/records/');
		const mostRecentRecord = await res.json();

		if (!res.ok) {
			throw new Error("Something's wrong");
		}

		return mostRecentRecord.pop();
	}
	const mostRecentRecordPromise = loadMostRecentRecord();
	const escapeRecordsPromise = loadEscapeData();

	let valueName;

	function showName() {
		console.log(valueName);
	}
</script>

<div class="header">
	<img id="logo" src={logo} alt="" />
	<h1>The Bank Heist Score Board</h1>
	<img id="seperator" src={seperator} alt="" />
</div>

{#await escapeRecordsPromise}
	<p>Loading...</p>
{:then records}
	{#await mostRecentRecordPromise then mostRecentRecord}
		<h1 id="mostRecentRecordHeading">Most Recent Room</h1>

		<table class="recordTable" id="mostRecentRecordTable">
			<tr>
				<th class="table_header">Team</th>
				<th class="table_header">Score</th>
				<th class="table_header">Time</th>
				<th class="table_header">Date</th>
			</tr>
			<tr>
				<td id="team_name">{mostRecentRecord.teamName}</td>
				<td>{mostRecentRecord.score}</td>
				<td class="score">{mostRecentRecord.timeInRoom}</td>
				<td>{mostRecentRecord.dateOfGame}</td>
			</tr>
		</table>
	{/await}
	<h1 id="recordScoresHeading">Leader Board</h1>
	<table class="recordTable">
		<tr>
			<th class="table_header position" id="position_head">#</th>
			<th class="table_header">Team</th>
			<th class="table_header">Score</th>
			<th class="table_header">Time</th>
			<th class="table_header">Date</th>
		</tr>
		{#each records as record, i}
			<tr>
				<td class="position">{i + 1}</td>
				<td id="team_name">{record.teamName}</td>
				<td>{record.score}</td>
				<td class="score">{record.timeInRoom}</td>
				<td>{record.dateOfGame}</td>
			</tr>
		{/each}
	</table>
{/await}

<style>
	@import url('https://fonts.googleapis.com/css2?family=Limelight&display=swap');

	@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');

	:global(body) {
		/*background-color: black;*/
		font-family: Poppins;
		background-color: aliceblue;
		color: rgb(237, 242, 244);
		margin: 0;
	}

	#mostRecentRecordHeading {
		background-color: hsl(224, 89%, 64%);
		text-align: center;
		font-size: 20px;
		width: max-content;
		position: relative;
	}

	#mostRecentRecordHeading,
	#recordScoresHeading {
		color: rgb(237, 242, 244);
		font-family: Limelight;
	}
	#seperator {
		background-color: transparent;
		margin-bottom: -30px;
		margin-top: -30px;
	}

	#position_head {
		text-align: center;
	}
	td {
		text-align: center;
		max-width: min-content;
	}

	.table_header {
		border-bottom: 1px solid navy;
	}
	#team_name {
		min-width: 50%;
	}
	#logo {
		width: 215px;
		position: absolute;
		top: 13px;
		left: -10px;
	}

	.header {
		height: 5rem;
		font-family: Limelight;
		font-size: 4rem;
		background-color: rgb(67, 67, 172);
		text-align: center;
		padding-top: 30px;
		padding-bottom: 1.9rem;
		border-bottom: solid 3px navy;
		border-radius: 0 0 2px 2px;
		z-index: -2;
	}
	.header > h1 {
		font-family: Limelight;
		font-size: 4rem;
		background-color: rgb(67, 67, 172);
		text-align: center;
		margin-top: -20px;
		margin-bottom: -80px;
		padding-bottom: 1.8rem;
	}

	.recordTable {
		min-width: 100%;
		font-size: 25px;
		padding: 4px;
		border-radius: 6px;
	}

	tr:nth-child(even) {
		background-color: rgb(43, 42, 42);
	}

	tr:nth-child(odd) {
		background-color: black;
	}
	.position {
		min-width: 4%;
		text-align: center;
	}

	#mostRecentRecordTable {
		font-size: 25px;
		border-radius: 6px;
	}

	#mostRecentRecordHeading {
		width: 100%;
		font-size: 35px;
		background-color: #517ef5;
		margin-top: 0;
		margin-bottom: 0;
		padding-top: 5px;
		padding-bottom: 5px;
		text-decoration: underline;
		z-index: 1;
		border-radius: 0 0 6px 6px;
	}
	#recordScoresHeading {
		width: 100%;
		font-size: 35px;
		background-color: #517ef5;
		margin-top: 0;
		margin-bottom: 0;
		padding-top: 5px;
		padding-bottom: 5px;

		text-decoration: underline;
		text-align: center;
		border-radius: 0 0 6px 6px;
	}
	td,
	th {
		border-radius: 6px;
	}
	tr {
		border-radius: 6px;
	}
</style>
