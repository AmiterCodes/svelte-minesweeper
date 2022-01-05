<script lang="ts">
  export let restart: () => void;
  export let time: number;
  export let bombsLeft: number;
</script>

<div class="toolbar">
  <h2>{time}</h2>

  <button on:click|preventDefault={() => restart()}>Restart</button>

  <h2>{bombsLeft}</h2>
</div>

<style>
  h2 {
    font-size: 2em;
    text-align: center;
  }

  .toolbar {
    display: flex;
    justify-content: space-between;
  }
</style>
