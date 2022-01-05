<script lang="ts">
  export let show: boolean;

  export let playAgain: () => void;
  export let won: boolean;
</script>

<div class="modal {show ? 'show' : 'hide'}">
  <div class="content">
    You {won ? "Win" : "Lose"}! click to play again
    <button on:click|preventDefault={playAgain} class="play-again-button"
      >PLAY AGAIN</button
    >
  </div>
</div>

<style>
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    flex-direction: column;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 9999;
  }

  .content {
    display: flex;
    background-color: #eee;
    padding: 0.5rem;
    flex-direction: column;

    justify-content: center;
    align-items: center;
  }

  .play-again-button {
    margin: 0.5rem;
    background-color: #111;
    color: white;
  }

  .hide {
    display: none;
  }
</style>
