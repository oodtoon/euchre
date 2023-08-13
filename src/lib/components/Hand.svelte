<script lang="ts">
  import Card from "./Card.svelte";
  import PlayedCards from "./PlayedCards.svelte";

  interface Card {
    type: string;
    suite: string;
    value: number;
  }

  export let playersHands: Card[][];
  export let blackTricks: number;
  export let redTricks: number;
  export let dealerIndex: number;
  export let hasDealerPickedUp: boolean;
  export let playerToCallTrump: number;
  export let hasEveryPlayerPassed: boolean;
  export let playedCards: Card[];
  export let playerTurn: Number;
  export let trump: string | null;

  export let handleCardClick: () => void;
  export let handleDiscard: () => void;
</script>

<div class="player-container">
  {#each playersHands as player, i}
    <div class={`p${i + 1}`}>
      Player {i + 1}.
      {#if i < 2}
        {#if i % 2 === 0}
          <span>Team 1</span>
          <div>Black tricks: {blackTricks}</div>
        {:else}
          <span>Team 2</span>
          <div>Red tricks: {redTricks}</div>
        {/if}
      {/if}

      {#if dealerIndex === i}
        <div>Dealer</div>
      {/if}

      {#if playerTurn === i}
        {#if !hasDealerPickedUp && dealerIndex === i && playerToCallTrump !== null}
          <div>Discard</div>
        {:else}
          <div>Turn</div>
        {/if}
      {/if}

      <div>
        {#each player as card}
          <span>
            {#if playedCards.includes(card)}
              <Card
                {i}
                {card}
                handleCard={() => {
                  return;
                }}
                isPlayed={true}
              />
            {/if}
            {#if trump !== "" && !hasEveryPlayerPassed && !hasDealerPickedUp}
              <Card {i} {card} handleCard={handleDiscard} isPlayed={false} />
            {:else}
              <Card {i} {card} handleCard={handleCardClick} isPlayed={false} />
            {/if}
          </span>
        {/each}
      </div>
    </div>
  {/each}

  <PlayedCards {playedCards} />
</div>

<style>
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

  .played-card-container {
    grid-area: played;
  }

  .red {
    color: rgb(181, 9, 9);
  }
</style>
