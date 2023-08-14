<script lang="ts">
  import Hand from "./Hand.svelte";
  import Info from "./Info.svelte";
  import PlayedCards from "./PlayedCards.svelte";
  import type { Card } from "../types";

  const cardVals = ["9", "10", "J", "Q", "K", "A"];
  const suites = ["♠", "♦", "♣", "♥"];

  let passCount: number = 0;
  let hasEveryPlayerPassed: boolean = false;
  let hasDealerPickedUp = false;
  let playerToCallTrump: number | null = null;

  let dealerIndex: number = 0;
  let playerTurn: number = 1;

  let redTricks: number = 0;
  let blackTricks: number = 0;

  let redScore: number = 0;
  let blackScore: number = 0;

  let trump: string = "";
  let suiteToFollow: string = "";

  $: trumpIndex = suites.indexOf(trump);

  $: leftBowerSuite = trump ? suites[(trumpIndex + 2) % suites.length] : "";

  $: leftBower = leftBowerSuite ? "J" + leftBowerSuite : "";

  $: if (passCount > 3) {
    passCount = 0;
    hasEveryPlayerPassed = true;
  }

  $: if (playerTurn > 3) {
    playerTurn = 0;
  }

  $: if (redTricks + blackTricks === 5) {
    console.log(redTricks, blackTricks);
    if (playerToCallTrump === 0 || playerToCallTrump === 2) {
      if (redTricks > 3) {
        redScore += 2;
      } else if (blackTricks === 5) {
        blackScore += 2;
      } else if (blackTricks < 5 && blackTricks > 2) {
        blackScore += 1;
      }
    } else if (playerToCallTrump === 1 || playerToCallTrump === 3) {
      if (blackTricks > 3) {
        blackScore += 2;
      } else if (redTricks === 5) {
        redScore += 2;
      } else if (redTricks < 5 && redTricks > 2) {
        redScore += 1;
      }
    }

    if (dealerIndex < 4) {
      dealerIndex++;
    } else {
      dealerIndex = 0;
    }

    setTimeout(() => {
      handleShuffle();
    }, 2500);
  }

  $: if (leftBower !== "") {
    const currentLeftBower = shuffledDeck.find(
      (card) => card.type + card.suite === leftBower
    );
    currentLeftBower.dynamicSuite = trump;
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
        let cardObj = {
          type: val,
          suite,
          value: getCardVal(val, suite),
          dynamicSuite: suite,
        };

        suiteVal.push(cardObj);
      });
      return suiteVal;
    });
  };

  let fullDeck = createDeck(suites, cardVals);

  let playersHands: [Card[], Card[], Card[], Card[]] = [[], [], [], []];
  let shuffledDeck: Card[];
  let playedCards: Card[] = [
    { type: "", suite: "", value: 0, dynamicSuite: "" },
    { type: "", suite: "", value: 0, dynamicSuite: "" },
    { type: "", suite: "", value: 0, dynamicSuite: "" },
    { type: "", suite: "", value: 0, dynamicSuite: "" },
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
    passCount = 0;
    trump = "";
    playerTurn = dealerIndex + 1;
    fullDeck = createDeck(suites, cardVals);
    playersHands = [[], [], [], []];
    shuffledDeck = [];
    playedCards = [
      { type: "", suite: "", value: 0, dynamicSuite: "" },
      { type: "", suite: "", value: 0, dynamicSuite: "" },
      { type: "", suite: "", value: 0, dynamicSuite: "" },
      { type: "", suite: "", value: 0, dynamicSuite: "" },
    ];
    hasEveryPlayerPassed = false;
    hasDealerPickedUp = false;
    playerToCallTrump = null;
  };

  const handleShuffle = () => {
    reset();
    shuffleDeck(fullDeck);
    dealCards(shuffledDeck);
  };

  const handlePass = () => {
    if (trump === "" && hasEveryPlayerPassed && passCount === 3) {
      return;
    } else {
      passCount++;
      playerTurn++;
    }
  };

  const handlePickUp = (event: MouseEvent) => {
    if (!hasEveryPlayerPassed) {
      playerToCallTrump = playerTurn;
      const trumpSuite = fullDeck[0].suite;
      trump = trumpSuite;
      playerTurn = dealerIndex;
    } else {
      playerToCallTrump = playerTurn;
      playerTurn = dealerIndex + 1;
      trump = event.target?.textContent;
    }
  };

  const handleCardClick = (event: MouseEvent) => {
    if (trump === "") {
      return;
    }

    const index = parseInt(event.target.value);

    if (index !== playerTurn) {
      return;
    }

    const matchingCard = shuffledDeck.find(
      (card) => card.type + card.suite === event.target.textContent
    );

    const hasLeadingSuite = playersHands[index].some(
      (card) => card.dynamicSuite === suiteToFollow
    );

    if (hasLeadingSuite && matchingCard!.dynamicSuite !== suiteToFollow) {
      return;
    }

    if (playedCards[index].type === "") {
      const playedCardIndex = playersHands[index].indexOf(matchingCard!);
      playedCards.splice(index, 1, matchingCard!);

      if (suiteToFollow === "") {
        suiteToFollow = playedCards[index].dynamicSuite;
      } else {
        if (matchingCard?.dynamicSuite === trump) {
          matchingCard!.value = matchingCard!.value * 2;
        } else if (
          matchingCard?.dynamicSuite !== trump &&
          matchingCard?.suite !== suiteToFollow
        ) {
          matchingCard!.value = 0;
        }
      }
      playersHands[index].splice(playedCardIndex, 1);
      playedCards = [...playedCards];
      playersHands = playersHands;
    }

    if (playerTurn > 3) {
      playerTurn = 0;
    } else {
      playerTurn++;
    }
  };

  const handleDiscard = (event: MouseEvent) => {
    if (hasDealerPickedUp) {
      return;
    }

    const index = event.target!.value;

    if (parseInt(index) !== dealerIndex) {
      return;
    } else {
      const discardCard = shuffledDeck.find(
        (card) => card.type + card.suite === event.target!.textContent
      );

      const pickUpCard = fullDeck[0];
      const discardCardIndex = playersHands[index].indexOf(discardCard!);

      playersHands[index].splice(discardCardIndex, 1, pickUpCard);
      fullDeck.splice(0, 1, discardCard!);

      fullDeck = fullDeck;
      playersHands = playersHands;
      hasDealerPickedUp = true;
      playerTurn = dealerIndex + 1;
    }
  };

  const handleNext = () => {
    getHandWinner(playedCards);
    suiteToFollow = "";
    playedCards = [
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
      { type: "", suite: "", value: 0 },
    ];
  };

  const getHandWinner = (cards: Card[]) => {
    const highestCard = cards.find(
      (card) => card.value === Math.max(...cards.map((c) => c.value))
    );
    const winningIndex = playedCards.indexOf(highestCard!);
    if (winningIndex === 1 || winningIndex === 3) {
      redTricks++;
    } else {
      blackTricks++;
    }
    playerTurn = winningIndex;
  };
</script>

<button on:click={handleShuffle}>Shuffle and Deal</button>
<button on:click={reset}>Reset</button>

<Hand
  {playersHands}
  {blackTricks}
  {redTricks}
  {dealerIndex}
  {hasDealerPickedUp}
  {playerToCallTrump}
  {hasEveryPlayerPassed}
  {playedCards}
  {playerTurn}
  {trump}
  {handleCardClick}
  {handleDiscard}
/>
<Info
  {fullDeck}
  {trump}
  {passCount}
  {blackScore}
  {redScore}
  {hasEveryPlayerPassed}
  {hasDealerPickedUp}
  {leftBowerSuite}
  {playerTurn}
  {suites}
  {handleNext}
  {handlePass}
  {handlePickUp}
/>
