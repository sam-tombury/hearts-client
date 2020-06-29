<script>
	import Card from './Card.svelte'
	export let trick;
	export let lastTrick;
</script>


{#if trick.length > 0}
	<h2>Trick</h2>
	{#each trick as card}
		<Card {...card}/>
	{/each}
	{#if lastTrick.length > 0}
		<p>
			Last trick:
			{#each lastTrick as {suit, value}}
				<span class={suit.colour + ' small-card'}>{value.shortName}{suit.icon}</span>
			{/each}
		</p>
	{/if}
{:else if lastTrick.length > 0}
	<h2>Last trick</h2>
	{#each lastTrick as card}
		<Card {...card}/>
	{/each}
{/if}

<style>
	.small-card {
		font-size: 0.75rem;
		padding-left: 0.5rem;
	}
</style>