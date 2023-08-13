<script lang="ts">

  interface Card {
    type: string;
    suite: string;
    value: number;
  }

  export let fullDeck: Card[];
  export let trump: string | null;
  export let passCount: number;
  export let blackScore: number;
  export let redScore: number;
  export let hasEveryPlayerPassed: boolean;
  export let hasDealerPickedUp: boolean;
  export let leftBowerSuite: string;
  export let playerTurn: number
  export let suites: string[];
 

  export let handlePass: () => void
  export let handleNext: () => void
  export let handlePickUp: () => void

</script>

{#if fullDeck.length > 20}
  <div>Kitty</div>
{:else}
  <div>Trump: {trump}</div>
  {#if hasEveryPlayerPassed || hasDealerPickedUp}
    <div>Kitty: Back of Card</div>
  {:else}
    <div>Kitty {fullDeck[0].type + fullDeck[0].suite}</div>
  {/if}
  <div>Pass count: {passCount}</div>
  <button on:click={handlePass}>Pass</button>

  {#if !trump && hasEveryPlayerPassed}
    {#each suites as suite}
      {#if suite !== fullDeck[0].suite && !trump}
        <span>
          <button
            class:red={suite === "♥" || suite === "♦"}
            on:click={handlePickUp}>{suite}</button
          >
        </span>
      {/if}
    {/each}
  {:else}
    <button on:click={handlePickUp}>Pick It Up</button>
  {/if}

  <div><button on:click={handleNext}>Next Hand</button></div>
  <div>
    Left Bower:
    {#if leftBowerSuite}
      <span>J{leftBowerSuite}</span>
    {/if}
  </div>
  <div>Turn: {playerTurn + 1}</div>
{/if}

<div>
  Black score: {blackScore}
  {#if blackScore === 10}<span>Winner</span>{/if}
</div>
<div>
  Red score: {redScore}
  {#if redScore === 10}<span>Winner!</span>{/if}
</div>

<style>
  button {
    cursor: pointer;
  }
  .red {
    color: rgb(181, 9, 9);
  }
</style>
