<script lang="ts">
  import type { ITile } from "./Game.svelte";

  export let tile: ITile;
  export let reveal: () => void;
  export let rightClick: () => void;
</script>

<button
  class="tile tile-{tile.value} {tile.revealed || tile.flagged
    ? 'revealed'
    : ''}"
  on:click={reveal}
  on:contextmenu|preventDefault={rightClick}
>
  {#if tile.flagged}
    <div class="flag">
      <img src="assets/flag.svg" alt="flag icon" />
    </div>
  {:else if tile.revealed}
    {#if tile.bomb}
      <div class="bomb">
        <img src="./assets/bomb.svg" alt="bomb icon" />
      </div>
    {:else}
      <div>
        {tile.value > 0 ? tile.value : " "}
      </div>
    {/if}
  {/if}
</button>

<style>
  .flag {
    fill: red;
  }

  .tile-1 {
    color: blue;
  }

  .tile-2 {
    color: green;
  }

  .tile-3 {
    color: red;
  }

  .tile-4 {
    color: darkblue;
  }

  .tile-5 {
    color: brown;
  }

  .tile-6 {
    color: cyan;
  }

  .tile-7 {
    color: black;
  }

  .tile-8 {
    color: gray;
  }

  .tile {
    background: #ccc;
    margin: none;
    width: 30px;
    height: 30px;
    border: 2px solid #444;
    border-radius: 0;
    display: flex;
    justify-content: center;
    cursor: pointer;
    font-weight: 800;
    align-items: center;
    text-align: center;
  }

  .tile:active {
    background: #bbb;
    border: 4px solid #333;
  }

  .tile:hover {
    background: #ddd;
    border: 3px solid #434;
  }

  .tile.revealed {
    background-color: #aaa;
  }
</style>
