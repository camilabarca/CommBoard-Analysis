<script>

	import { onMount } from 'svelte';
  	import firebase from '../lib/firebase/firebase';

	let data = [];
	let sortColumn = "date";
    let sortAsc = true;

	onMount(async () => {
		const snapshot = await firebase.database().ref('interactions').once('value');
		data = Object.values(snapshot.val());
		for (let item in data){
			const [date, time] = (data[item].date).split(", ");
			data[item].date = date
			data[item].time = time
		}
		sortTable();
	});

	function downloadCsv() {
        const rows = data
            .map(
                (data) =>
                    `${data.date},${data.time},${data.guardian},${data.subject},${data.sound}`
            )
            .join("\n");
        const csv = `Date,Time,Guardian,Subject,User,Sound\n${rows}`;
        const blob = new Blob([csv], { type: "text/csv" });
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.href = url;
        link.download = "interactions.csv";
        link.click();
    }

	function sortTable() {
        data = data.sort((a, b) => {
            const aValue = a[sortColumn];
            const bValue = b[sortColumn];

            if (aValue < bValue) {
                return sortAsc ? -1 : 1;
            } else if (aValue > bValue) {
                return sortAsc ? 1 : -1;
            } else {
                return 0;
            }
        });
    }

	function handleTitleClick(column){
		if (column === sortColumn){
			sortAsc = !sortAsc;
		} else {
			sortColumn = column;
			sortAsc = true;
		}
		sortTable();
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Commboard Analysis" />
</svelte:head>

<section>
	<div class="table-container">
		<table>
			<thead>
				<tr>
					<th on:click={() => handleTitleClick("date")}>Date {sortColumn === "date" ? (sortAsc ? "▲" : "▼") : ""}</th>
					<th on:click={() => handleTitleClick("time")}>Time {sortColumn === "time" ? (sortAsc ? "▲" : "▼") : ""}</th>
					<th on:click={() => handleTitleClick("guardian")}>Guardian {sortColumn === "guardian" ? (sortAsc ? "▲" : "▼") : ""}</th>
					<th on:click={() => handleTitleClick("subject")}>Subject {sortColumn === "subject" ? (sortAsc ? "▲" : "▼") : ""}</th>
					<th on:click={() => handleTitleClick("user")}>User {sortColumn === "user" ? (sortAsc ? "▲" : "▼") : ""}</th>
					<th on:click={() => handleTitleClick("sound")}>Sound {sortColumn === "sound" ? (sortAsc ? "▲" : "▼") : ""}</th>
				</tr>
			</thead>
			<tbody>
				{#each data as item}
				<tr>
					<td>{item.date}</td>
					<td>{item.time}</td>
					<td>{item.guardian}</td>
					<td>{item.subject}</td>
					<td>{item.user}</td>
					<td>{item.sound}</td>
					<!-- Add more columns as needed -->
				</tr>
				{/each}
			</tbody>
		</table>
	</div>
	<button on:click={downloadCsv}>Download</button>

</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}
	table {
		width: 100%;
		border-collapse: collapse;
	}

	th, td {
		padding: 12px;
		text-align: left;
		border-bottom: 1px solid #ddd;
	}

	th {
		background-color: #f2f2f2;
		color: #333;
		font-weight: bold;
		cursor: pointer;
	}

	tr:nth-child(even) {
		background-color: #f9f9f9;
	}

	tr:hover {
		background-color: #e9e9e9;
	}
	button {
		padding: 10px 20px;
		background-color: #4a90e2;
		color: #fff;
		border: none;
		border-radius: 5px;
		font-size: 16px;
		position: absolute;
		bottom: 10px;
		left: 10px;
		cursor: pointer;
	}
	.table-container {
		width: 100%;
		height: 500px; 
		overflow-y: auto;
		border: 1px solid #ccc;
		padding: 10px;
	}



</style>
