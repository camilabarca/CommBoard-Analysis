<script>

	import { onMount } from 'svelte';
	import { createEventDispatcher } from 'svelte';

  	import firebase from '../lib/firebase/firebase';
	import flatpickr from 'flatpickr';

	import 'flatpickr/dist/flatpickr.css';

	/**
     * @type {any[]}
     */
	// list for all data
	let data = [];

	/**
     * @type {any[]}
     */
	// list for all filtered data
	let filteredData = [];

	// column to sort by
	let sortColumn = "date";
	// sort ascending 
    let sortAsc = true;

	// filters values for each column
	let filters = {
		time: "",
		guardian: "",
		subject: "",
		user: "",
		sound: ""
	};

	// date filter
	let dateFilter = {
		startDate: null,
		endDate: null,
	};

	/**
     * @type {import("flatpickr/dist/types/instance").Instance | import("flatpickr/dist/types/instance").Instance[]}
     */
	let datepicker;

	let filtersApplied = false;

	// when application starts 
	onMount(async () => {
		// @ts-ignore
		datepicker = flatpickr('#date-filter', {
			mode: 'range',
			dateFormat: 'd/m/Y',
			onChange: handleDateFilterChange,
		});
		// get data from database and pass it to list
		const snapshot = await firebase.database().ref('interactions').once('value');
		data = Object.values(snapshot.val());
		for (let item in data){
			const [date, time] = data[item].date.includes(", ")
			? data[item].date.split(", ")
			: data[item].date.split(" ");
			data[item].date = date;
			data[item].time = time;
		}
		filteredData = [...data];
		sortTable();
	});

	// download csv with data
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

	// sort table by selected column
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

	/**
     * @param {string} column
     */
	// change column to sort by when clicked
	function handleTitleClick(column){
		if (column === sortColumn){
			sortAsc = !sortAsc;
		} else {
			sortColumn = column;
			sortAsc = true;
		}
		sortTable();
	}

	/**
     * @param {string} column
     * @param {Event | undefined} event
     */
	// handle filters
	function handleFilterChange(column, event) {
		// @ts-ignore
		const value = event.target.value;
		console.log(value);
		// @ts-ignore
		filters[column] = value ? value.trim().toLowerCase() : '';
		applyFilters();
	}
	/**
     * @param {null[]} selectedDates
     * @param {any} dateStr
     * @param {any} instance
     */
	// handle date filters
	function handleDateFilterChange(selectedDates, dateStr, instance) {
		dateFilter.startDate = selectedDates[0] || null;
		dateFilter.endDate = selectedDates[1] || null;
		console.log(dateFilter);
		applyFilters();
	}

	// apply filters
	function applyFilters() {
		filteredData = data.filter(item => {
		for (const column in filters) {
			// @ts-ignore
			const filterValue = filters[column];
			if (filterValue && !item[column].toLowerCase().includes(filterValue)) {
			return false;
			}
		}

		const dateString = item.date;
		const parts = dateString.split("/");
		const day = parseInt(parts[0], 10);
		const month = parseInt(parts[1], 10) - 1; // Month is zero-based (0-11)
		const year = parseInt(parts[2], 10);

		const itemDate = new Date(year, month, day);

		const startDate = dateFilter.startDate;
		const endDate = dateFilter.endDate;

		// @ts-ignore
		if (startDate && itemDate < startDate) {
			return false;
		}

		// @ts-ignore
		if (endDate && itemDate > endDate) {
			return false;
		}

		return true;
		});

		// @ts-ignore
		filtersApplied = Object.values(filters).some(value => value !== '') || dateFilter.startDate || dateFilter.endDate;
	}

	// clear date filters
	function clearDateFilter() {
		// @ts-ignore
		datepicker.clear();
		dateFilter.startDate = null;
		dateFilter.endDate = null;
		applyFilters();
	}

	/**
     * @param {string} filter
     */
	// clear other filters
	function clearFilter(filter){
		// @ts-ignore
		filters[filter] = "";
		applyFilters();
	}

</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Commboard Analysis" />
</svelte:head>

<section>
	<!-- container for all filters -->
	<div class="filter-container"> 
		<div class="filter-input">
			<!-- svelte-ignore a11y-label-has-associated-control -->
			<label>Date:</label>
			<input type="text" id="date-filter" readonly />
			<button class="clear-button" on:click={clearDateFilter}></button>
		</div>
		<div class="filter-input">
			<!-- svelte-ignore a11y-label-has-associated-control -->
			<label>Time:</label>
			<input type="text" bind:value={filters.time} on:input={(event) => handleFilterChange('time', event)} />
			<button class="clear-button" on:click={() => clearFilter('time')}></button>
		</div>
			<!-- svelte-ignore a11y-label-has-associated-control -->
		<div class="filter-input">
			<label>Guardian:</label>
			<input type="text" bind:value={filters.guardian} on:input={() => handleFilterChange('guardian', event)} />
			<button class="clear-button" on:click={() => clearFilter('guardian')}></button>
		</div>
		<div class="filter-input">
			<!-- svelte-ignore a11y-label-has-associated-control -->
			<label>Subject:</label>
			<input type="text" bind:value={filters.subject} on:input={() => handleFilterChange('subject', event)} />
			<button class="clear-button" on:click={() => clearFilter('subject')}></button>
		</div>
		<div class="filter-input">
			<!-- svelte-ignore a11y-label-has-associated-control -->
			<label>User:</label>
			<input type="text" bind:value={filters.user} on:input={(event) => handleFilterChange('user', event)} />
			<button class="clear-button" on:click={() => clearFilter('user')}></button>
		</div>
		<div class="filter-input">
			<!-- svelte-ignore a11y-label-has-associated-control -->
			<label>Sound:</label>
			<input type="text" bind:value={filters.sound} on:input={() => handleFilterChange('sound', event)} />
			<button class="clear-button" on:click={() => clearFilter('sound')}></button>
		</div>
	</div>
	
	<!-- container for the table -->
	<div class="table-container">
		<table>
			<!-- table titles -->
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
			<!-- table data -->
			<tbody>
				{#each filteredData as item}
				<tr>
					<td>{item.date}</td>
					<td>{item.time}</td>
					<td>{item.guardian}</td>
					<td>{item.subject}</td>
					<td>{item.user}</td>
					<td>{item.sound}</td>
				</tr>
				{/each}
			</tbody>
		</table>
	</div>
	<!-- button for download -->
	<button class="download" on:click={downloadCsv}>Download</button>

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
	.download {
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
		height: 350px; 
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
	.clear-button {
		background: none;
		border: none;
		padding: 5px;
		font: inherit;
		color: #666;
		cursor: pointer;
		outline: none;
	}

	.clear-button::before {
		content: "×";
	}



</style>
