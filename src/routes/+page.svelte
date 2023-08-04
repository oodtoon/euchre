<script lang="ts">
  const cardVals = ["9", "10", "J", "Q", "K", "A"];
  const suites = ["♠", "♦", "♣", "♥"];

  let passCount: number = 0;
  let trump: string = "";

  $: if (passCount > 4) {
    passCount = 0;
  }

  const createDeck = (suites: string[], vals: string[]) => {
    return suites.flatMap((suite) => {
      const suiteVal: string[] = [];
      vals.forEach((val) => {
        suiteVal.push(val + suite);
      });
      return suiteVal;
    });
  };

  let fullDeck = createDeck(suites, cardVals);

  let players: [string[], string[], string[], string[]] = [[], [], [], []];
  let shuffledDeck: string[];
  let playedCards = ["", "", "", ""];

  const shuffleDeck = (deck: string[]) => {
    if (deck.length < 5) {
      return;
    }
    const randomIndex = Math.floor(Math.random() * (deck.length - 1));
    const randomCard = deck.splice(randomIndex, 1)[0];
    shuffledDeck.push(randomCard);
    shuffleDeck(deck);
  };

  const dealCards = (deck: string[]) => {
    for (let i = 0; i < deck.length; i++) {
      const index = i % 4;
      players[index].push(deck[i]);
    }
  };

  const reset = () => {
    fullDeck = createDeck(suites, cardVals);
    players = [[], [], [], []];
    shuffledDeck = [];
    playedCards = ["", "", "", ""];
  };

  const handleShuffle = () => {
    reset();
    shuffleDeck(fullDeck);
    dealCards(shuffledDeck);
  };

  const handlePass = () => {
    passCount++;
    console.log("pass");
  };

  const handlePickUp = () => {
    const trumpSuite = fullDeck[0].slice(-1);
    trump = trumpSuite;
  };

  const handleCardClick = (event: MouseEvent) => {
    const index = event.target!.value;

    if (playedCards[index] === "") {
      // getCardVal(event.target!.textContent)
      event.target!.remove()!;
      playedCards.splice(index, 1, event.target!.textContent);
      playedCards = [...playedCards];
    }
  };

  const handleNext = () => {
    console.log("nextHand");
    getHighestCard(playedCards);
  };

  const getCardVal = (card: string) => {
    let val: string;
    if (card.length > 2) {
      val = card[0] + card[1];
    } else {
      val = card[0];
    }
    console.log(val);
  };

  const getHighestCard = (cards: string[]) => {
    if (cards.includes("A")) {
      console.log("A");
    } else if (cards.includes("K")) {
      console.log("K");
    }
  };
</script>

<button on:click={handleShuffle}>Shuffle</button>
<button on:click={reset}>Reset</button>

<div class="player-container">
  {#each players as player, i}
    <div class={`p${i + 1}`}>
      Player {i + 1}.
      {#if i % 2 === 0}
        <span>Team 1</span>
      {:else}
        <span>Team 2</span>
      {/if}
      <div>
        {#each player as card}
          <span
            ><button
              class="card"
              value={i}
              on:click={handleCardClick}
              class:red={card.slice(-1) === "♥" || card.slice(-1) === "♦"}
              >{`${card}`}</button
            ></span
          >
        {/each}
      </div>
    </div>
  {/each}

  <div class="played-card-container">
    {#if playedCards.length < 5}
      {#each playedCards as card, i}
        {#if card !== ""}
          <div
            class={`c${i + 1}`}
            class:red={card.slice(-1) === "♥" || card.slice(-1) === "♦"}
          >
            {card}
          </div>
        {/if}
      {/each}
    {/if}
  </div>
</div>

{#if fullDeck.length > 20}
  <div>Kitty</div>
{:else}
  <div>Trump: {trump}</div>
  <div>Kitty {fullDeck[0]}</div>
  <div>Pass count: {passCount}</div>
  <button on:click={handlePass}>Pass</button>
  <button on:click={handlePickUp}>Pick It Up</button>
  <div><button on:click={handleNext}>Next Hand</button></div>
{/if}

<style>
  button {
    cursor: pointer;
  }
  .player-container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-areas: "p3 p3 p3" "p2 played p4" "p1 p1 p1";
  }

  .p1 {
    grid-area: p1;
    place-self: center;
    min-height: 3em;
  }

  .p2 {
    grid-area: p2;
    justify-self: end;
    align-self: center;
    min-height: 3em;
  }

  .p3 {
    grid-area: p3;
    place-self: center;
    min-height: 3em;
  }

  .p4 {
    grid-area: p4;
    min-height: 3em;
    align-self: center;
  }

  .card {
    margin: 0.25em;
    width: 3rem;
    height: 4rem;
  }
  .played-card-container {
    grid-area: played;
    display: grid;
    margin: auto;
    grid-auto-columns: 1fr 1fr;
    grid-template-areas: ". c3 ." "c2 c3 c4" "c2 c1 c4" ". c1 .";
    place-items: center;
    min-height: 10rem;
  }

  .c1 {
    grid-area: c1;
  }

  .c2 {
    grid-area: c2;
  }

  .c3 {
    grid-area: c3;
  }

  .c4 {
    grid-area: c4;
  }

  .c1,
  .c3 {
    height: 4rem;
    width: 3rem;
  }

  .c2,
  .c4 {
    height: 3rem;
    width: 4rem;
  }

  .c1,
  .c2,
  .c3,
  .c4 {
    border-radius: 4px;
    border: 1px solid black;
    display: grid;
    place-content: center;
    margin: 0.1rem;
    background-color: rgb(241, 241, 241);
  }

  .red {
    color: rgb(181, 9, 9);
  }
</style>
