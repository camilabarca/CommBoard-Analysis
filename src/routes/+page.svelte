<script>
	import Counter from './Counter.svelte';

	import { onMount } from 'svelte';
  	import firebase from '../lib/firebase/firebase';

	let data = [];

	onMount(async () => {
		const snapshot = await firebase.database().ref('interactions').once('value');
		data = Object.values(snapshot.val());
		for (let item in data){
			const [date, time] = (data[item].date).split(", ");
			data[item].date = date
			data[item].time = time
		}
		console.log(data)
	});
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Commboard Analysis" />
</svelte:head>

<section>

	<button>Download</button>
	<table>
		<thead>
			<tr>
				<th>Date</th>
				<th>Time</th>
				<th>Guardian</th>
				<th>Subject</th>
				<th>Sound</th>
			</tr>
		</thead>
		<tbody>
			{#each data as item}
			  <tr>
				<td>{item.date}</td>
				<td>{item.time}</td>
				<td>{item.guardian}</td>
				<td>{item.subject}</td>
				<td>{item.sound}</td>
				<!-- Add more columns as needed -->
			  </tr>
			{/each}
		  </tbody>
	</table>

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
	}

	tr:nth-child(even) {
	background-color: #f9f9f9;
	}

	tr:hover {
	background-color: #e9e9e9;
	}



</style>
