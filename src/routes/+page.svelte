<script>
	const cardVals = ["9", "10", "J", "Q", "K", "A"]
	const suites = ["♠", "♦", "♣", "♥"]

	const createDeck = (suites, vals) => {
		return suites.flatMap((suite) => {
			const suiteVal = []
			vals.forEach((val) => {
				suiteVal.push(val + suite)
			})
			return suiteVal
		})
	}

	let fullDeck = createDeck(suites, cardVals)

	let players = [[], [], [], []]
	let shuffledDeck = []
	let playedCards = ["", "", "", ""]

	const shuffleDeck = (deck) => {
		if (deck.length < 5) {
			return
		}
		const randomIndex = Math.floor(Math.random() * (deck.length - 1))
		const randomCard = deck.splice(randomIndex, 1)
		shuffledDeck.push(randomCard)
		shuffleDeck(deck)
	}



	const dealCards = (deck) => {
		for (let i = 0; i < deck.length; i++) {
				const index = i % 4
				players[index].push(deck[i])
		}
	}

const reset = () => {
fullDeck = createDeck(suites, cardVals)
players = [[], [], [], []]
shuffledDeck = []
playedCards = ["", "", "", ""]
}

const handleShuffle = () => {
reset()
shuffleDeck(fullDeck)
dealCards(shuffledDeck)

}

const handlePass = () => {
	console.log("pass")
}

const handlePickUp = () => {
	console.log("pick up", fullDeck[0])
}

const handleCardClick = (event) => {
	const index= (event.target.value)

	if (playedCards[index] === "") {
	event.target.remove()
			playedCards.splice(index, 1, event.target.textContent)
	playedCards = [...playedCards]
	}

}
</script>

<button on:click={handleShuffle}>Shuffle</button>
<button on:click={reset}>Reset</button>

<div class="player-container">
{#each players as player, i}
	<div class={`p${i+1}`}>Player {i +1}.
	{#if i % 2 === 0}
	<span>Team 1</span>
	{:else}
	<span>Team 2</span>
	{/if}
<div>
{#each player as card}
	<span><button value={i} on:click={handleCardClick} class:red={card[0].slice(-1 ) === ("♥") || card[0].slice(-1) === "♦"}>{`${card}`}</button></span>
{/each}
</div>
</div>
{/each}

<div class="played-card-container">
{#if playedCards.length < 5}
	{#each playedCards as card, i}
		<div class={`c${i + 1}`} class:red={card.slice(-1 ) === ("♥") || card.slice(-1) === "♦"}>{card}</div>
	{/each}
{/if}
</div>
</div>



{#if fullDeck.length > 20}
<div>Kitty</div>
{:else}
<div>Kitty {fullDeck[0]}</div>
<button on:click={handlePass}>Pass</button>
<button on:click={handlePickUp}>Pick It Up</button>
{/if}

<style>
	.player-container {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr;
		grid-template-areas: "p3 p3 p3" "p2 played p4" "p1 p1 p1";
	}

	.p1 {
		grid-area: p1;
    place-self: center;
	}

	.p2 {
		grid-area: p2;
    justify-self: end;
    
	}

	.p3 {
		grid-area: p3;
    place-self: center;
	}

	.p4 {
		grid-area: p4
	}

	.played-card-container {
		grid-area: played;
		display: grid;
		margin: auto;
		grid-auto-columns: 1fr 1fr;
		grid-template-areas: "c3 c3" "c2 c4" "c1 c1";
	}

		.c1 {
		grid-area: c1;
		place-self: center;
	}

	.c2 {
		grid-area: c2;
		justify-self: start;
	}

	.c3 {
		grid-area: c3;
		place-self: center;
	}

	.c4 {
		grid-area: c4;
		justify-self: end;
	}

  .red {
    color: red;
  }

</style>
