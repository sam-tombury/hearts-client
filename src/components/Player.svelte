<script>

	export let name;
	export let score;
	export let currentHandScore;
	export let currentHandScoringCards;

</script>

<p>
	{name}:
	<span class='score'>{score}</span>
	<span class='grey hand-score'>
		{currentHandScore}
	</span>
	{#each currentHandScoringCards as {suit, value}}
		<span class={suit.colour + ' hand-score'}>{value.shortName}{suit.icon}</span>
	{/each}
</p>

<style>
	.hand-score {
		font-size: 0.75rem;
		padding-left: 0.5rem;
	}
	.grey {
		color: #666
	}
</style>