<script context="module" lang="ts">
  export interface ITile {
    bomb: boolean;
    flagged: boolean;
    revealed: boolean;
    value: number;
  }

  export interface Board {
    opened: boolean;
    tiles: ITile[][];
    bombs: number;
    width: number;
    height: number;
    gameOver: boolean;
  }
</script>

<script lang="ts">
  import { onDestroy, onMount } from "svelte";

  import Tile from "./Tile.svelte";
  import Toolbar from "./Toolbar.svelte";
  import WinModal from "./WinModal.svelte";

  const defaultWidth = 30;
  const defaultHeight = 16;
  const defaultBombCount = 99;

  let time: number = 0;

  let board = generateEmptyBoard();

  let won: boolean = false;

  function generateEmptyBoard() {
    return { ...generateBoard(0), opened: false };
  }

  let intervalId: number;

  let flagged = 0;

  onMount(() => {
    intervalId = setInterval(() => {
      if (!board.gameOver) {
        time++;
      }
    }, 1000) as unknown as number;
  });

  onDestroy(() => {
    clearInterval(intervalId);
  });

  function restart() {
    board = generateEmptyBoard();
    won = false;
    flagged = 0;
    time = 0;
  }

  function generateBoard(
    bombCount: number = defaultBombCount,
    width: number = defaultWidth,
    height: number = defaultHeight,
    startingPosition: [number, number] = [0, 0]
  ): Board {
    let board = {
      tiles: [],
      bombs: bombCount,
      width,
      height,
      gameOver: false,
      opened: false,
    };

    for (let i = 0; i < board.height; i++) {
      board.tiles[i] = [];
      for (let j = 0; j < board.width; j++) {
        board.tiles[i][j] = {
          bomb: false,
          flagged: false,
          revealed: false,
        };
      }
    }

    // add mines, not near starting position
    for (let i = 0; i < bombCount; i++) {
      let x = Math.floor(Math.random() * height);
      let y = Math.floor(Math.random() * width);
      if (
        (Math.abs(x - startingPosition[0]) <= 1 &&
          Math.abs(y - startingPosition[1]) <= 1) ||
        board.tiles[x][y].bomb
      ) {
        i--;
        continue;
      }
      board.tiles[x][y].bomb = true;
    }

    // add values
    for (let i = 0; i < board.height; i++) {
      for (let j = 0; j < board.width; j++) {
        if (!board.tiles[i][j].bomb) {
          let value = 0;
          for (let x = i - 1; x <= i + 1; x++) {
            for (let y = j - 1; y <= j + 1; y++) {
              if (
                x >= 0 &&
                x < board.height &&
                y >= 0 &&
                y < board.width &&
                board.tiles[x][y].bomb
              ) {
                value++;
              }
            }
          }
          board.tiles[i][j].value = value;
        }
      }
    }

    return board;
  }

  function reveal(i: number, j: number, tile: ITile, notRecursive: boolean) {
    if (board.gameOver) {
      restart();
      return;
    }

    if (!board.opened) {
      board = {
        ...generateBoard(defaultBombCount, defaultWidth, defaultHeight, [i, j]),
        opened: true,
      };
      tile = board.tiles[i][j];
    }
    if (tile.flagged) return;

    if (tile.bomb) {
      board.gameOver = true;
      // reveal the board
      for (let i = 0; i < board.height; i++) {
        for (let j = 0; j < board.width; j++) {
          if (board.tiles[i][j].bomb) {
            board.tiles[i][j].revealed = true;
          }
        }
      }
      return;
    }

    if (tile.revealed) {
      let count = 0;
      let flaggedAround = 0;
      let revealedAround = 0;
      for (let x = i - 1; x <= i + 1; x++) {
        for (let y = j - 1; y <= j + 1; y++) {
          if (x >= 0 && x < board.height && y >= 0 && y < board.width) {
            if (board.tiles[x][y].flagged) {
              flaggedAround++;
            }
            if (board.tiles[x][y].revealed) {
              revealedAround++;
            }
            count++;
          }
        }
      }

      if (flaggedAround === tile.value) {
        for (let x = i - 1; x <= i + 1; x++) {
          for (let y = j - 1; y <= j + 1; y++) {
            if (
              x >= 0 &&
              x < board.height &&
              y >= 0 &&
              y < board.width &&
              !board.tiles[x][y].flagged &&
              !board.tiles[x][y].revealed
            ) {
              reveal(x, y, board.tiles[x][y], false);
            }
          }
        }
      } else if (revealedAround === count - tile.value && notRecursive) {
        for (let x = i - 1; x <= i + 1; x++) {
          for (let y = j - 1; y <= j + 1; y++) {
            if (
              x >= 0 &&
              x < board.height &&
              y >= 0 &&
              y < board.width &&
              !board.tiles[x][y].flagged &&
              !board.tiles[x][y].revealed
            ) {
              flag(x, y, board.tiles[x][y]);
              board = board;
            }
          }
        }
      }
    } else {
      if (tile.value === 0) {
        for (let x = i - 1; x <= i + 1; x++) {
          for (let y = j - 1; y <= j + 1; y++) {
            if (
              x >= 0 &&
              x < board.height &&
              y >= 0 &&
              y < board.width &&
              !board.tiles[x][y].revealed
            ) {
              board.tiles[x][y].revealed = true;
              reveal(x, y, board.tiles[x][y], false);
            }
          }
        }
      } else {
        tile.revealed = true;
      }
    }
    board = board;

    if (notRecursive && isBoardDone()) {
      finishGame();
    }
  }

  function finishGame() {
    board.gameOver = true;
    // flag all bombs
    for (let i = 0; i < board.height; i++) {
      for (let j = 0; j < board.width; j++) {
        if (board.tiles[i][j].bomb) {
          board.tiles[i][j].flagged = true;
        }
      }
    }
    board = board;
  }

  function isBoardDone() {
    let count = 0;
    for (let i = 0; i < board.height; i++) {
      for (let j = 0; j < board.width; j++) {
        if (board.tiles[i][j].revealed) {
          count++;
        }
      }
    }

    return (won = count === board.width * board.height - board.bombs);
  }

  function flag(i: number, j: number, tile: ITile) {
    if (tile.revealed) {
      return;
    }

    board.tiles[i][j].flagged = !board.tiles[i][j].flagged;
    if (board.tiles[i][j].flagged) {
      flagged++;
    } else {
      flagged--;
    }

    board = board;
  }

  function onKey(e: KeyboardEvent) {
    if (e.key === "r") {
      restart();
    }
  }
</script>

<WinModal playAgain={restart} {won} show={board.gameOver} />

<div on:keypress={(e) => onKey(e)}>
  <Toolbar {time} {restart} bombsLeft={board.bombs - flagged} />
  <table>
    {#each board.tiles as row, i}
      <tr class="row">
        {#each row as tile, j}
          <Tile
            {tile}
            reveal={() => reveal(i, j, tile, true)}
            rightClick={() => flag(i, j, tile)}
          />
        {/each}
      </tr>
    {/each}
  </table>
</div>

<style>
  .row {
    height: 30px;
    display: flex;
    margin: none;
  }
</style>
