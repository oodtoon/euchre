<script lang="ts">
  const cardVals = ["9", "10", "J", "Q", "K", "A"];
  const suites = ["♠", "♦", "♣", "♥"];

  let passCount: number = 0;

  let dealerIndex: number = 0;
  let playerTurn: number = 1
  let cardsPlayed: number = 0

  let redTricks: number = 0;
  let blackTricks: number = 0;

  let trump: string = "";

  $: trumpIndex = suites.indexOf(trump);

  $: leftBowerSuite = trump ? suites[(trumpIndex + 2) % suites.length] : "";

  $: leftBower = leftBowerSuite ? "J" + leftBowerSuite : "";
  let suiteToFolllow: string = "";

  $: if (passCount > 4) {
    passCount = 0;
  }

  $: if (redTricks + blackTricks === 5) {
    if (dealerIndex < 4) {
      dealerIndex++;
    } else {
      dealerIndex = 0;
    }
  }

  interface Card {
    type: string;
    suite: string;
    value: number;
  }

  let cardsValues: {
    9: number;
    10: number;
    J: number;
    Q: number;
    K: number;
    A: number;
  };

  cardsValues = {
    9: 9,
    10: 10,
    J: 11,
    Q: 12,
    K: 13,
    A: 14,
  };

  const getCardVal = (cardType: string, suite: string) => {
    if (cardType === "J") {
      if (suite === trump) {
        return 16;
      } else if (suite === leftBowerSuite) {
        return 15;
      } else {
        return 11;
      }
    }
    return cardsValues[cardType as keyof typeof cardsValues];
  };

  const createDeck = (suites: string[], vals: string[]) => {
    return suites.flatMap((suite) => {
      const suiteVal: Card[] = [];
      vals.forEach((val) => {
        let cardObj = { type: val, suite, value: getCardVal(val, suite) };

        suiteVal.push(cardObj);
      });
      return suiteVal;
    });
  };

  let fullDeck = createDeck(suites, cardVals);

  let playersHands: [Card[], Card[], Card[], Card[]] = [[], [], [], []];
  let shuffledDeck: Card[];
  let playedCards: Card[] = [
    { type: "", suite: "", value: 0 },
    { type: "", suite: "", value: 0 },
    { type: "", suite: "", value: 0 },
    { type: "", suite: "", value: 0 },
  ];

  $: if (shuffledDeck) {
    if (shuffledDeck.length < 24) {
      shuffledDeck = [...shuffledDeck, ...fullDeck];
    }

    shuffledDeck.forEach((card) => {
      if (card.type !== "J") {
        return;
      }

      if (card.suite === trump) {
        card.value = 16;
      } else if (card.suite === leftBowerSuite) {
        card.value = 15;
      }
    });
    shuffledDeck = shuffledDeck;
  }

  const shuffleDeck = (deck: Card[]) => {
    console.log(playersHands);
    if (deck.length < 5) {
      return;
    }
    const randomIndex = Math.floor(Math.random() * (deck.length - 1));
    const randomCard = deck.splice(randomIndex, 1)[0];

    shuffledDeck.push(randomCard);
    shuffleDeck(deck);
  };

  const dealCards = (deck: Card[]) => {
    for (let i = 0; i < deck.length; i++) {
      const index = i % 4;
      playersHands[index].push(deck[i]);
    }
  };

  const reset = () => {
    trump = "";
    fullDeck = createDeck(suites, cardVals);
    playersHands = [[], [], [], []];
    shuffledDeck = [];
    playedCards = [
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
    ];
  };

  const handleShuffle = () => {
    reset();
    shuffleDeck(fullDeck);
    dealCards(shuffledDeck);
  };

  const handlePass = () => {
    passCount++;
  };

  const handlePickUp = () => {
    const trumpSuite = fullDeck[0].suite;
    trump = trumpSuite;
  };

  const handleCardClick = (event: MouseEvent) => {
    if (trump === "") {
      return;
    }

    const index = event.target!.value;

    if (index !== playerTurn) {
      return
    }

    if (playedCards[index].type === "") {
      const matchingCard = shuffledDeck.find(
        (card) => card.type + card.suite === event.target!.textContent
      );

      const playedCardIndex = playersHands[index].indexOf(matchingCard!);
      playersHands[index].splice(playedCardIndex, 1);

      if (suiteToFolllow !== "") {
        const hasLeadingSuite = playersHands[index].some(
          (card) => card.suite === suiteToFolllow
        );
        if (matchingCard?.suite !== suiteToFolllow && hasLeadingSuite) {
          return;
        }

        if (
          matchingCard?.suite !== trump &&
          matchingCard?.type! + matchingCard?.suite! !== leftBower &&
          matchingCard?.suite !== suiteToFolllow
        ) {
          matchingCard!.value = 0;
        }
      }

      if (
        matchingCard?.suite === trump ||
        (matchingCard?.suite === leftBowerSuite && matchingCard.type === "J")
      ) {
        matchingCard.value = matchingCard.value * 2;
      }

      playedCards.splice(index, 1, matchingCard!);
      playedCards = [...playedCards];
    }

    if (suiteToFolllow === "") {
      if (
        playedCards[index].suite === leftBowerSuite &&
        playedCards[index].type === "J"
      ) {
        suiteToFolllow = trump;
      } else {
        suiteToFolllow = playedCards[index].suite;
      }
    }

    playerTurn++
  };

  const handleNext = () => {
    console.log("nextHand");
    getHighestCard(playedCards);
    suiteToFolllow = "";
    playedCards = [
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
    ];
  };

  const getHighestCard = (cards: Card[]) => {
    const highestCard = cards.find(
      (card) => card.value === Math.max(...cards.map((c) => c.value))
    );
    const winningIndex = playedCards.indexOf(highestCard!);
    if (winningIndex === 1 || winningIndex === 3) {
      redTricks++;
    } else {
      blackTricks++;
    }
  };
</script>

<button on:click={handleShuffle}>Shuffle</button>
<button on:click={reset}>Reset</button>

<div class="player-container">
  {#each playersHands as player, i}
    <div class={`p${i + 1}`}>
      Player {i + 1}.
      {#if i % 2 === 0}
        <span>Team 1</span>
        <div>Black tricks: {blackTricks}</div>
      {:else}
        <span>Team 2</span>
        <div>Red tricks: {redTricks}</div>
      {/if}
      {#if dealerIndex === i}
        <div>Dealer</div>
      {/if}

      <div>
        {#each player as card}
          <span>
            {#if playedCards.includes(card)}
              <button style="display: none">{card}</button>
            {/if}
            <button
              class="card"
              value={i}
              on:click={handleCardClick}
              class:red={card.suite === "♥" || card.suite === "♦"}
              >{`${card.type + card.suite}`}</button
            ></span
          >
        {/each}
      </div>
    </div>
  {/each}

  <div class="played-card-container">
    {#if playedCards.length < 5}
      {#each playedCards as card, i}
        {#if card.type !== ""}
          <div
            class={`c${i + 1}`}
            class:red={card.suite === "♥" || card.suite === "♦"}
          >
            {card.type + card.suite}
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
  <div>Kitty {fullDeck[0].type + fullDeck[0].suite}</div>
  <div>Pass count: {passCount}</div>
  <button on:click={handlePass}>Pass</button>
  <button on:click={handlePickUp}>Pick It Up</button>
  <div><button on:click={handleNext}>Next Hand</button></div>
  <div>
    Left Bower:
    {#if leftBowerSuite}
      <span>J{leftBowerSuite}</span>
    {/if}
  </div>
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
