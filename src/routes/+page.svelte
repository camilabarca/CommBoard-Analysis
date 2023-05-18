<script>

	import { onMount } from 'svelte';
	import { createEventDispatcher } from 'svelte';

  	import firebase from '../lib/firebase/firebase';

	let data = [];
	let filteredData = [];

	let sortColumn = "date";
    let sortAsc = true;

	let filters = {
		date: "",
		time: "",
		guardian: "",
		subject: "",
		user: "",
		sound: ""
	};

	let filtersApplied = false;

	const dispatch = createEventDispatcher();

	onMount(async () => {
		const snapshot = await firebase.database().ref('interactions').once('value');
		data = Object.values(snapshot.val());
		filteredData = [...data];
		for (let item in data){
			const [date, time] = (data[item].date).split(", ");
			data[item].date = date
			data[item].time = time
		}
		sortTable();
	});

	function downloadCsv() {
        const rows = filteredData
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
        filteredData = filteredData.sort((a, b) => {
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

	function handleFilterChange(column, event) {
		const value = event.target.value;
		console.log(value);
		filters[column] = value ? value.trim().toLowerCase() : '';
		applyFilters();
	}

	function applyFilters() {
		if (Object.values(filters).some(value => value !== '')) {
			filteredData = data.filter(item => {
				for (const column in filters) {
					const filterValue = filters[column];
					const itemValue = item[column];

					if (filterValue && (itemValue === undefined || itemValue === null)) {
						return false;
					}

					if (filterValue && !itemValue.toLowerCase().includes(filterValue)) {
						return false;
					}
				}
				return true;
			});
			filtersApplied = true;
		} else {
			filteredData = data;
			filtersApplied = false;
		}
	}

</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Commboard Analysis" />
</svelte:head>

<section>
	<div class="filter-container"> 
		<div class="filter-input">
			<label>Date:</label>
			<input type="text" bind:value={filters.date} on:input={() => handleFilterChange('date', event)} />
		</div>
		<div class="filter-input">
			<label>Time:</label>
			<input type="text" bind:value={filters.time} on:input={(event) => handleFilterChange('time', event)} />
		</div>
		<div class="filter-input">
			<label>Guardian:</label>
			<input type="text" bind:value={filters.guardian} on:input={() => handleFilterChange('guardian', event)} />
		</div>
		<div class="filter-input">
			<label>Subject:</label>
			<input type="text" bind:value={filters.subject} on:input={() => handleFilterChange('subject', event)} />
		</div>
		<div class="filter-input">
			<label>User:</label>
			<input type="text" bind:value={filters.user} on:input={(event) => handleFilterChange('user', event)} />
		</div>
		<div class="filter-input">
			<label>Sound:</label>
			<input type="text" bind:value={filters.sound} on:input={() => handleFilterChange('sound', event)} />
		</div>
	</div>
	
	
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
				{#each filteredData as item}
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
		height: 300px; 
		overflow-y: auto;
		border: 1px solid #ccc;
		padding: 10px;
	}

	.filter-container {
		display: flex;
		flex-wrap: wrap;
		align-items: center;
		margin-bottom: 20px;
	}

	.filter-input {
		margin-right: 10px;
		margin-bottom: 10px;
		display: flex;
		align-items: center;
		flex: 1 1 200px; /* Adjust the width as needed */
	}

	.filter-input label {
		margin-right: 5px;
	}

	.filter-input input {
		padding: 5px;
		border: 1px solid #ccc;
		border-radius: 4px;
		width: 100%;
	}



</style>
