<script lang="ts">
  import type { Card } from "../types"

  export let i: number;
  export let card: Card;
  export let isPlayed: boolean

  export let handleCard: () => void

</script>

<button
  class="card"
  value={i}
  on:click={handleCard}
  class:red={card.suite === "♥" || card.suite === "♦"}
  class:isPlayed
  >{`${card.type + card.suite}`}</button
>

<style>
  .card {
    margin: 0.25em;
    width: 3rem;
    height: 4rem;
    cursor: pointer;
  }

  .red {
    color: rgb(181, 9, 9);
  }

  .isPlayed {
    display: none;
  }

</style>
