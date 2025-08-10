
<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>2048 Game</title>
  <style>
    html, body {
      overflow: hidden;
      height: 100%;
      touch-action: none;
    }
    body {
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      background: #faf8ef;
      margin: 0;
      padding: 20px;
    }
    h1 {
      margin-bottom: 10px;
      color: #776e65;
    }
    #score-box {
      display: flex;
      gap: 20px;
      margin-bottom: 10px;
    }
    .score {
      background: #bbada0;
      color: #fff;
      padding: 10px 20px;
      border-radius: 5px;
      font-size: 18px;
    }
    #game-board {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
      background-color: #bbada0;
      padding: 10px;
      border-radius: 10px;
      width: 90vw;
      max-width: 400px;
      aspect-ratio: 1 / 1;
    }
    .tile {
      background: #cdc1b4;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 5vw;
      font-weight: bold;
      border-radius: 5px;
      color: #776e65;
      animation: pop 0.3s ease;
    }
    @keyframes pop {
      0% { transform: scale(0.8); opacity: 0.5; }
      100% { transform: scale(1); opacity: 1; }
    }
    .tile-2 { background: #eee4da; }
    .tile-4 { background: #ede0c8; }
    .tile-8 { background: #f2b179; color: #f9f6f2; }
    .tile-16 { background: #f59563; color: #f9f6f2; }
    .tile-32 { background: #f67c5f; color: #f9f6f2; }
    .tile-64 { background: #f65e3b; color: #f9f6f2; }
    .tile-128 { background: #edcf72; color: #f9f6f2; }
    .tile-256 { background: #edcc61; color: #f9f6f2; }
    .tile-512 { background: #edc850; color: #f9f6f2; }
    .tile-1024 { background: #edc53f; color: #f9f6f2; }
    .tile-2048 { background: #edc22e; color: #f9f6f2; }
    #new-game {
      margin: 10px;
      padding: 10px 20px;
      font-size: 16px;
      background: #8f7a66;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #game-over {
      display: none;
      margin-top: 15px;
      font-size: 24px;
      color: red;
    }
  </style>
</head>
<body>
  <h1>2048 Game</h1>
  <div id="score-box">
    <div class="score">Score: <span id="score">0</span></div>
    <div class="score">Best: <span id="best-score">0</span></div>
  </div>
  <button id="new-game">New Game</button>
  <div id="game-board"></div>
  <div id="game-over">Game Over! <button onclick="restartGame()">Restart</button></div><audio id="merge-sound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_e3fc3cba3b.mp3"></audio> <audio id="bg-music" src="https://cdn.pixabay.com/download/audio/2022/10/04/audio_8571f0d8aa.mp3?filename=lofi-study-112191.mp3" loop autoplay></audio>

  <script>
    const board = document.getElementById('game-board');
    const scoreDisplay = document.getElementById('score');
    const bestScoreDisplay = document.getElementById('best-score');
    const newGameButton = document.getElementById('new-game');
    const gameOverDisplay = document.getElementById('game-over');
    const mergeSound = document.getElementById('merge-sound');
    const bgMusic = document.getElementById('bg-music');

    let grid;
    let score = 0;
    let bestScore = localStorage.getItem('bestScore2048') || 0;
    bestScoreDisplay.textContent = bestScore;

    function initializeGrid() {
      grid = Array.from({ length: 4 }, () => Array(4).fill(0));
      score = 0;
      updateScore();
      addRandomTile();
      addRandomTile();
      drawBoard();
      gameOverDisplay.style.display = 'none';
      bgMusic.play().catch(() => {});
    }

    function drawBoard() {
      board.innerHTML = '';
      grid.forEach(row => {
        row.forEach(cell => {
          const tile = document.createElement('div');
          tile.classList.add('tile');
          if (cell !== 0) {
            tile.classList.add(`tile-${cell}`);
            tile.textContent = cell;
          }
          board.appendChild(tile);
        });
      });
    }

    function addRandomTile() {
      const empty = [];
      for (let r = 0; r < 4; r++) {
        for (let c = 0; c < 4; c++) {
          if (grid[r][c] === 0) empty.push([r, c]);
        }
      }
      if (empty.length === 0) return;
      const [r, c] = empty[Math.floor(Math.random() * empty.length)];
      grid[r][c] = Math.random() < 0.9 ? 2 : 4;
    }

    function updateScore() {
      scoreDisplay.textContent = score;
      if (score > bestScore) {
        bestScore = score;
        localStorage.setItem('bestScore2048', bestScore);
        bestScoreDisplay.textContent = bestScore;
      }
    }

    function move(dir) {
      let rotated = rotateGrid(grid, dir);
      let moved = false;
      for (let r = 0; r < 4; r++) {
        let row = rotated[r].filter(x => x !== 0);
        for (let i = 0; i < row.length - 1; i++) {
          if (row[i] === row[i + 1]) {
            row[i] *= 2;
            score += row[i];
            row[i + 1] = 0;
            moved = true;
            mergeSound.play();
          }
        }
        row = row.filter(x => x !== 0);
        while (row.length < 4) row.push(0);
        if (rotated[r].join() !== row.join()) moved = true;
        rotated[r] = row;
      }
      if (moved) {
        grid = rotateGrid(rotated, (4 - dir) % 4);
        addRandomTile();
        updateScore();
        drawBoard();
        if (isGameOver()) {
          gameOverDisplay.style.display = 'block';
        }
      }
    }

    function rotateGrid(g, times) {
      let result = g.map(row => row.slice());
      for (let t = 0; t < times; t++) {
        result = result[0].map((_, i) => result.map(row => row[i]).reverse());
      }
      return result;
    }

    function isGameOver() {
      for (let r = 0; r < 4; r++) {
        for (let c = 0; c < 4; c++) {
          if (grid[r][c] === 0) return false;
          if (c < 3 && grid[r][c] === grid[r][c + 1]) return false;
          if (r < 3 && grid[r][c] === grid[r + 1][c]) return false;
        }
      }
      return true;
    }

    // Keyboard
    document.addEventListener('keydown', e => {
      if (e.key === 'ArrowLeft') move(0);
      if (e.key === 'ArrowUp') move(1);
      if (e.key === 'ArrowRight') move(2);
      if (e.key === 'ArrowDown') move(3);
    });

    // Swipe for mobile
    let startX = 0, startY = 0;
    document.addEventListener('touchstart', e => {
      startX = e.touches[0].clientX;
      startY = e.touches[0].clientY;
    });

    document.addEventListener('touchend', e => {
      const dx = e.changedTouches[0].clientX - startX;
      const dy = e.changedTouches[0].clientY - startY;
      if (Math.abs(dx) > Math.abs(dy)) {
        if (dx > 30) move(2);
        else if (dx < -30) move(0);
      } else {
        if (dy > 30) move(3);
        else if (dy < -30) move(1);
      }
    });

    newGameButton.addEventListener('click', initializeGrid);

    function restartGame() {
      initializeGrid();
    }

    initializeGrid();
  </script></body>
</html>
