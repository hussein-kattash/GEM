<script>
  const winningLines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];

  const cellDescriptions = [
    'row 1 column 1',
    'row 1 column 2',
    'row 1 column 3',
    'row 2 column 1',
    'row 2 column 2',
    'row 2 column 3',
    'row 3 column 1',
    'row 3 column 2',
    'row 3 column 3'
  ];

  let board = Array(9).fill('');
  let currentPlayer = 'X';
  let startingPlayer = 'X';
  let winner = null;
  let winningLine = [];
  let isDraw = false;
  let scores = {
    X: 0,
    O: 0,
    draws: 0
  };

  function handleSquareClick(index) {
    if (board[index] || winner) {
      return;
    }

    const updatedBoard = board.slice();
    updatedBoard[index] = currentPlayer;
    board = updatedBoard;

    const result = calculateWinner(updatedBoard);

    if (result) {
      winner = result.player;
      winningLine = result.line;
      scores = { ...scores, [result.player]: scores[result.player] + 1 };
      startingPlayer = startingPlayer === 'X' ? 'O' : 'X';
    } else if (updatedBoard.every(Boolean)) {
      isDraw = true;
      winningLine = [];
      scores = { ...scores, draws: scores.draws + 1 };
      startingPlayer = startingPlayer === 'X' ? 'O' : 'X';
    } else {
      currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
    }
  }

  function calculateWinner(boardState) {
    for (const line of winningLines) {
      const [a, b, c] = line;
      if (boardState[a] && boardState[a] === boardState[b] && boardState[a] === boardState[c]) {
        return { player: boardState[a], line };
      }
    }
    return null;
  }

  function startNextRound() {
    board = Array(9).fill('');
    winner = null;
    winningLine = [];
    isDraw = false;
    currentPlayer = startingPlayer;
  }

  function resetScores() {
    scores = { X: 0, O: 0, draws: 0 };
    startingPlayer = 'X';
    startNextRound();
  }

  $: gameOver = Boolean(winner) || isDraw;
  $: statusMessage = winner
    ? `Player ${winner} wins!`
    : isDraw
    ? "It's a draw!"
    : `Player ${currentPlayer}, it's your turn.`;
  $: nextRoundHint = gameOver
    ? `Next round will start with player ${startingPlayer}.`
    : `Place three of your marks in a row to win.`;
  $: winningCells = new Set(winningLine ?? []);
</script>

<main>
  <h1>Tic-Tac-Toe</h1>
  <p class="status" aria-live="polite">{statusMessage}</p>
  <p class="instructions">{nextRoundHint}</p>

  <div class="board" role="grid" aria-label="Tic-tac-toe board">
    {#each board as value, index}
      <button
        class="cell"
        class:winner={winningCells.has(index)}
        on:click={() => handleSquareClick(index)}
        disabled={Boolean(value) || gameOver}
        aria-label={`Cell ${index + 1}, ${cellDescriptions[index]}, ${value ? `contains ${value}` : 'empty'}`}
        role="gridcell"
      >
        {value || ''}
      </button>
    {/each}
  </div>

  <section class="scoreboard" aria-label="Scoreboard">
    <h2>Scoreboard</h2>
    <ul class="score-list">
      <li>
        <span class="score-label">Player X</span>
        <span class="score-value">{scores.X}</span>
      </li>
      <li>
        <span class="score-label">Draws</span>
        <span class="score-value">{scores.draws}</span>
      </li>
      <li>
        <span class="score-label">Player O</span>
        <span class="score-value">{scores.O}</span>
      </li>
    </ul>
    <p class="next-player">Next round starts with player {startingPlayer}</p>
  </section>

  <div class="controls">
    <button class="action" type="button" on:click={startNextRound}>Start next round</button>
    <button class="action secondary" type="button" on:click={resetScores}>Reset scores</button>
  </div>
</main>
