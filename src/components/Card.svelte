<script>
	export let value;
	export let suit;
	export let selected = false;
</script>

<div on:click class:selected class={suit.colour}>
	<span class='top-left'>{suit.icon}</span>
	<span class='value'>{value.shortName}</span>
	<span class='bottom-right'>{suit.icon}</span>
</div>

<style>
	.top-left {
		position: absolute;
		left: 10px;
		top: 10px;
	}

	.bottom-right {
		position: absolute;
		right: 10px;
		bottom: 10px;
	}

	.value {
		position: relative;
		top: calc(50% - 1.25rem);
		font-size: 2em;
	}

	.selected {
		transform: translateY(-0.5rem);
		border-color: #666
	}

	div {
		width: 75px;
		height: 110px;
		background: #f4f4f4;
		border-radius: 5px;
		border: 1px #ccc solid;
		position: relative;
		text-align: center;
		margin: 5px;
		display: inline-block;
		cursor: pointer;
	}

</style>