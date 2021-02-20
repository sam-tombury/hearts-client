<script>
	let games = [];
	let playerNames = Array(4);
	let hands = 5;
	$: validNames = playerNames.filter((p) => p).length;
	const createGame = async () => {
		const fetchData = {
			method: "POST",
			body: JSON.stringify({
				players: playerNames.filter((p) => p),
				hands,
			}),
			headers: new Headers(),
		};
		const resp = await fetch("/api/games", fetchData);
		const body = await resp.json();
		games = [
			...games,
			{
				gameID: body.gameID,
				seats: body.seats,
			},
		];
		playerNames = Array(4);
	};
</script>

<main>
	<h1>Hearts</h1>
	<h3>New game</h3>
	<h4>Players</h4>
	{#each playerNames as player, row}
		<input
			bind:value={player}
			placeholder="Player {row + 1} name{row > 2 ? ' (optional)' : ''}"
		/>
	{/each}
	<h4>Hands</h4>
	<input bind:value={hands} type="number" min={1} />
	<button on:click={createGame} disabled={validNames < 3}>Create</button>
	{#if games.length !== 0}
		<h3>Games</h3>
	{/if}
	{#each games as game}
		<p>Game {game.gameID}</p>
		{#each game.seats as { name, href }, i}
			<a target="_blank" {href}>{name}</a>
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
		padding-right: 2rem;
	}
	input {
		margin-right: 0.5rem;
	}
</style>
