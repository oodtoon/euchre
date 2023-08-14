<script lang="ts">
  import type { Card } from "../types"

  export let playedCards: Card[];
</script>

<div class="played-container">
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

<style>
  .played-container {
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
