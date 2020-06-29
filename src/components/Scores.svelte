<script>
	import Player from './Player.svelte'
	export let players;
</script>


{#if players.length > 0}
	<h2>Scores</h2>
	{#each players as player}
		<Player {...player}/>
	{/each}
{/if}