<script lang="ts">
  import Footer from "./Footer.svelte";

  import Game from "./Game.svelte";
</script>

<main>
  <h1>MineSweeper</h1>
  <Game />
</main>
<Footer />

<style>
  main {
    font-family: "Press Start 2P";
    display: flex;
    align-items: center;
    flex-direction: column;
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
