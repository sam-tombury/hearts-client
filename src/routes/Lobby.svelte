<script>
	let games = []
	let players = 4
	const createGame = () => {
        const fetchData = {
            method: 'POST',
            body: {},
            headers: new Headers()
        }
        const url = "/api/games?players=" + players
        fetch(url, fetchData)
            .then(resp => resp.json())
            .then(body => {
				games = [...games, {
					gameID: body.gameID,
					seats: body.seatHrefs
				}]
            })
	}
</script>

<main>
	<h1>Hearts</h1>
	<h3>New game</h3>
	<select bind:value={players}>
		<option value={3}>3 players</option>
		<option value={4}>4 players</option>
	</select>
	<button on:click={createGame}>Create</button>
	{#if games.length !== 0}
		<h3>Games</h3>
	{/if}
	{#each games as game}
		<p>Game {game.gameID}</p>
		{#each game.seats as seat, i}
			<a target="_blank" href={seat}>
				Seat {i + 1}
			</a>
		{/each}
	{/each}
</main>

<style>
	main {
		padding: 1em;
		margin: 0 auto;
	}
	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}
	a {
		padding-right: 2rem
	}
</style>