<script>
	import Card from "../components/Card.svelte";
	import Scores from "../components/Scores.svelte";
	import Trick from "../components/Trick.svelte";

	let hand = [];
	let trick = [];
	let lastTrick = [];
	let events = [];
	let players = [];

	let passing = false;
	let passCount;

	let playing = false;

	$: selectedCards = hand.filter((h) => h.selected).map((h) => h.card);

	export let params;
	const gameID = params.gameID;
	const seatID = params.seatID;
	let name = new URLSearchParams(window.location.search).get("name") || "";
	const join = async () => {
		const fetchData = {
			method: "POST",
			body: name,
			headers: new Headers(),
		};
		const url = `/api/games/${gameID}/seats/${seatID}`;
		await fetch(url, fetchData);
		const socket = new WebSocket(
			(window.location.protocol === "https:" ? "wss://" : "ws://") +
				window.location.host +
				url +
				"/ws"
		);
		socket.onopen = () =>
			showEvent(
				`Welcome, ${name}. The game will start once all players have joined.`
			);
		socket.onmessage = (msg) => onMessage(JSON.parse(msg.data));
	};

	const showEvent = (text) => {
		const event = {
			message: text,
			timestamp: new Date(),
		};
		events = [event, ...events];
	};

	const onMessage = (message) => {
		const { eventType, data } = message;
		switch (eventType) {
			case "GameStarted":
				initGame(data);
				break;
			case "ReceiveHand":
				receiveHand(data);
				break;
			case "ReceivePass":
				receivePass(data);
				break;
			case "CardPlayed":
				cardPlayed(data);
				break;
			case "RequestLead":
				requestPlay();
				break;
			case "RequestPlay":
				requestPlay(data);
				break;
			case "TrickEnded":
				endTrick(data);
				break;
			case "HandEnded":
				endHand(data);
				break;
		}
	};

	const initGame = (data) => {
		players = data.map((gamePlayer) => {
			const player = {
				name: gamePlayer.name,
				id: gamePlayer.id,
				score: 0,
				currentHandScore: 0,
				currentHandScoringCards: [],
			};
			return player;
		});
	};

	const receiveHand = (data) => {
		trick = [];
		hand = data.hand.map((card) => {
			const holding = {
				card: card,
				selected: false,
			};
			return holding;
		});
		showEvent(
			`Choose ${data.passCount} cards to pass to ${data.passTo}. ${data.firstLead} will lead first.`
		);
		passCount = data.passCount;
		passing = true;
	};

	const endTrick = (data) => {
		players = players.map((player) => {
			if (player.id === data.winner.id) {
				player.currentHandScoringCards = [
					...player.currentHandScoringCards,
					...data.scoringCards,
				];
				player.currentHandScore += data.pointsTaken;
			}
			return player;
		});
		showEvent("Trick won by " + data.winner.name);
		lastTrick = trick;
		trick = [];
		setTimeout(() => {
			lastTrick = [];
		}, 7000);
	};

	const endHand = (data) => {
		players = players.map((player) => {
			player.score += data.find(
				(result) => result.player.id === player.id
			).pointsTaken;
			player.currentHandScore = 0;
			player.currentHandScoringCards = [];
			return player;
		});
	};

	const receivePass = (cards) => {
		const newHolding = cards.map((card) => {
			const holding = {
				card: card,
				selected: false,
			};
			return holding;
		});
		hand = [...hand, ...newHolding];
		passing = false;
		showEvent("Pass received");
	};

	const sortHand = () => {
		hand.sort(
			(card1, card2) =>
				(card1.card.suit.order - card2.card.suit.order) * 100 +
				(card1.card.value.order - card2.card.value.order)
		);
		hand = hand;
	};

	const requestPlay = (suit) => {
		playing = true;
		showEvent(`Your ${suit ? "turn, the lead is " + suit.name : "lead"}`);
	};

	const cardPlayed = (data) => {
		playing = false;
		trick = [...trick, data.card];
		showEvent(
			`The ${data.card.value.name} of ${data.card.suit.name} was played`
		);
		data.toPlay && showEvent(`${data.toPlay}'s turn`);
	};

	const toggleSelected = (card) => {
		hand = hand.map((c) => {
			if (c.card === card) {
				c.selected = !c.selected;
			}
			return c;
		});
	};

	const toPost = (card) => {
		const postData = {
			value: card.value.name,
			suit: card.suit.name,
		};
		return postData;
	};

	const pass = () => {
		if (selectedCards.length !== passCount) {
			showEvent(`Please select ${passCount} cards to pass`);
			return;
		}
		const fetchData = {
			method: "POST",
			body: JSON.stringify(selectedCards.map(toPost)),
			headers: {
				"Content-Type": "application/json",
			},
		};
		const url = `/api/games/${gameID}/seats/${seatID}/passes`;
		fetch(url, fetchData)
			.then((resp) => {
				if (!resp.ok) {
					throw Error(resp.text());
				} else {
					return resp.text();
				}
			})
			.then(() => {
				passing = false;
				hand = hand.filter((h) => !h.selected);
			})
			.catch((err) => console.log(err));
	};

	const play = () => {
		if (selectedCards.length !== 1) {
			showEvent("Please select a single card to play");
			return;
		}
		const card = selectedCards[0];
		const fetchData = {
			method: "POST",
			body: JSON.stringify(toPost(card)),
			headers: {
				"Content-Type": "application/json",
			},
		};
		const url = `/api/games/${gameID}/seats/${seatID}/plays`;
		fetch(url, fetchData)
			.then((resp) => {
				if (!resp.ok) {
					throw Error(resp.text());
				} else {
					return resp.text();
				}
			})
			.then(() => {
				hand = hand.filter((h) => h.card != card);
			})
			.catch((err) => console.log(err));
	};
</script>

<main>
	{#if events.length === 0}
		<h1>Hearts</h1>
		<h3>Join game</h3>
		<input bind:value={name} placeholder="Name" />
		<button on:click={join}> Join </button>
	{:else}
		<h2>Events</h2>
		{#each events.slice(0, 3) as { message, timestamp }}
			<p>
				<span class="timestamp">{timestamp.toLocaleTimeString("en-gb")}</span>
				{message}
			</p>
		{/each}
	{/if}
	<Scores {players} />
	<Trick {trick} {lastTrick} />
	{#if hand.length > 0}
		<h2>Hand<span on:click={sortHand} class="sort">⇅</span></h2>
		<p>
			{#if passing}
				<button disabled={selectedCards.length !== passCount} on:click={pass}
					>Pass</button
				>
			{:else if playing}
				<button disabled={selectedCards.length !== 1} on:click={play}
					>Play</button
				>
			{/if}
		</p>
		{#each hand as { card, selected }}
			<Card on:click={() => toggleSelected(card)} {...card} {selected} />
		{/each}
	{/if}
</main>

<style>
	main {
		padding: 1em;
		margin: 0 auto;
	}
	.timestamp {
		padding-right: 0.25rem;
		font-size: 0.75em;
		color: #666;
	}
	.sort {
		cursor: pointer;
		padding-left: 0.5rem;
	}
</style>
