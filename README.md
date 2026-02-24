<!doctype html>
<html lang="pt-BR" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>JDS Three in a Row - Multiplayer</title>
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@400;500;600;700&amp;display=swap" rel="stylesheet">
  <style>
    * { font-family: 'Fredoka', sans-serif; }
    
    .piece {
      cursor: pointer;
      transition: all 0.2s ease;
      user-select: none;
    }
    
    .piece:hover:not(.disabled) {
      transform: scale(1.15);
      filter: brightness(1.2);
    }
    
    .piece.selected {
      animation: pulse 0.8s infinite;
      filter: drop-shadow(0 0 12px currentColor);
    }
    
    @keyframes pulse {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.15); }
    }
    
    .board-cell {
      transition: all 0.2s ease;
      cursor: pointer;
    }
    
    .board-cell:hover {
      background: rgba(255,255,255,0.15);
    }
    
    .winning-cell {
      animation: celebrate 0.6s ease-out;
    }
    
    @keyframes celebrate {
      0% { transform: scale(1); }
      25% { transform: scale(1.2) rotate(5deg); }
      50% { transform: scale(1.3) rotate(-5deg); }
      75% { transform: scale(1.2) rotate(3deg); }
      100% { transform: scale(1); }
    }
    
    .score-pop {
      animation: scorePop 0.5s ease-out;
    }
    
    @keyframes scorePop {
      0% { transform: scale(1); }
      50% { transform: scale(1.4); }
      100% { transform: scale(1); }
    }
    
    .player-active {
      box-shadow: 0 0 30px rgba(251, 191, 36, 0.6);
      border-color: #fbbf24 !important;
      transform: scale(1.05);
    }
    
    .confetti {
      position: fixed;
      width: 10px;
      height: 10px;
      pointer-events: none;
      animation: confetti-fall 3s linear forwards;
    }
    
    @keyframes confetti-fall {
      0% { transform: translateY(-100vh) rotate(0deg); opacity: 1; }
      100% { transform: translateY(100vh) rotate(720deg); opacity: 0; }
    }

    .modal-overlay {
      animation: fadeIn 0.3s ease-out;
    }

    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }

    .modal-content {
      animation: slideUp 0.3s ease-out;
    }

    @keyframes slideUp {
      from { transform: translateY(20px); opacity: 0; }
      to { transform: translateY(0); opacity: 1; }
    }
  </style>
  <style>body { box-sizing: border-box; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full bg-gradient-to-br from-indigo-900 via-purple-900 to-pink-900 overflow-auto">
  <div id="app" class="w-full h-full flex flex-col items-center justify-center p-4 py-6">
   <h1 id="game-title" class="text-3xl md:text-4xl font-bold text-white mb-2 text-center drop-shadow-lg">🎮 JDS Three in a Row</h1>
   <p class="text-purple-200 text-sm md:text-base mb-4 text-center max-w-md px-2">Coloque suas peças no tabuleiro 3x3. Forme 3 em linha para pontuar!</p>
   <div class="flex gap-4 md:gap-8 mb-4 flex-col md:flex-row md:items-center md:justify-center">
    <div id="player1-card" class="bg-gradient-to-br from-rose-500 to-rose-700 rounded-2xl p-3 md:p-4 border-4 border-rose-400 transition-all duration-300">
     <div class="flex items-center gap-2 mb-1"><span class="text-2xl">🔴</span> <span id="player1-name" class="text-white font-semibold text-sm md:text-base">Jogador 1</span>
     </div>
     <div id="player1-score" class="text-3xl md:text-4xl font-bold text-white text-center">
      0
     </div>
    </div>
    <div class="flex items-center justify-center"><span class="text-2xl md:text-3xl text-yellow-400 font-bold">VS</span>
    </div>
    <div id="player2-card" class="bg-gradient-to-br from-blue-500 to-blue-700 rounded-2xl p-3 md:p-4 border-4 border-blue-400 transition-all duration-300">
     <div class="flex items-center gap-2 mb-1"><span class="text-2xl">🔵</span> <span id="player2-name" class="text-white font-semibold text-sm md:text-base">Jogador 2</span>
     </div>
     <div id="player2-score" class="text-3xl md:text-4xl font-bold text-white text-center">
      0
     </div>
    </div>
   </div>
   <div id="turn-indicator" class="bg-black/30 backdrop-blur rounded-full px-4 py-2 mb-4"><span class="text-white text-sm md:text-base">Vez de: <span id="current-player" class="font-bold text-yellow-400">Jogador 1 🔴</span></span>
   </div>
   <div class="relative mb-4">
    <div id="board" class="grid grid-cols-3 gap-2 bg-gradient-to-br from-amber-800 to-amber-950 p-4 rounded-2xl shadow-2xl border-4 border-amber-600"></div>
   </div>
   <div class="flex flex-col items-center gap-3">
    <div id="message" class="text-yellow-300 text-sm md:text-base font-medium text-center min-h-6">
     Clique em uma casa vazia para colocar sua peça
    </div>
    <div class="flex gap-3">
     <button id="reset-btn" class="bg-gradient-to-r from-emerald-500 to-emerald-700 hover:from-emerald-400 hover:to-emerald-600 text-white font-bold py-2 px-4 md:px-6 rounded-full transition-all duration-300 hover:scale-105 shadow-lg text-sm md:text-base">🔄 Novo Jogo</button> <button id="rules-btn" class="bg-gradient-to-r from-violet-500 to-violet-700 hover:from-violet-400 hover:to-violet-600 text-white font-bold py-2 px-4 md:px-6 rounded-full transition-all duration-300 hover:scale-105 shadow-lg text-sm md:text-base">📖 Regras</button>
    </div>
   </div>
  </div>
  <div id="rules-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden items-center justify-center z-50 p-4 modal-overlay">
   <div class="bg-gradient-to-br from-indigo-800 to-purple-900 rounded-2xl p-6 max-w-md w-full shadow-2xl border-2 border-purple-400 modal-content">
    <h2 class="text-2xl font-bold text-white mb-4 text-center">📖 Regras do Jogo</h2>
    <ul class="text-purple-100 space-y-3 text-sm md:text-base">
     <li class="flex gap-2"><span>🎯</span><span>Cada jogador coloca 3 peças no tabuleiro 3x3</span></li>
     <li class="flex gap-2"><span>♟️</span><span>Clique em uma casa vazia para colocar sua peça</span></li>
     <li class="flex gap-2"><span>🎲</span><span>Depois que todas 6 peças estão no tabuleiro, você pode mover suas peças para casas adjacentes</span></li>
     <li class="flex gap-2"><span>⭐</span><span>Forme uma linha de 3 peças suas (horizontal, vertical ou diagonal) para pontuar</span></li>
     <li class="flex gap-2"><span>🔄</span><span>Após pontuar, todas as peças voltam para o início</span></li>
     <li class="flex gap-2"><span>🏆</span><span>Primeiro a 5 pontos vence!</span></li>
    </ul><button id="close-rules" class="mt-6 w-full bg-gradient-to-r from-yellow-500 to-orange-500 hover:from-yellow-400 hover:to-orange-400 text-white font-bold py-3 rounded-full transition-all duration-300 hover:scale-105">Entendi! 👍</button>
   </div>
  </div>
  <div id="victory-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden items-center justify-center z-50 p-4 modal-overlay">
   <div class="bg-gradient-to-br from-yellow-600 to-orange-700 rounded-2xl p-6 max-w-md w-full shadow-2xl border-4 border-yellow-400 modal-content">
    <div class="text-6xl text-center mb-4">
     🏆
    </div>
    <h2 id="winner-text" class="text-2xl md:text-3xl font-bold text-white mb-2 text-center">Jogador 1 Venceu!</h2>
    <p class="text-yellow-100 text-center mb-6">Parabéns pela vitória!</p><button id="play-again" class="w-full bg-gradient-to-r from-emerald-500 to-emerald-700 hover:from-emerald-400 hover:to-emerald-600 text-white font-bold py-3 rounded-full transition-all duration-300 hover:scale-105 text-lg">🎮 Jogar Novamente</button>
   </div>
  </div>
  <script>
    const defaultConfig = {
      game_title: '🎮 JDS Three in a Row',
      player1_name: 'Jogador 1',
      player2_name: 'Jogador 2'
    };

    let board = [];
    let currentPlayer = 1;
    let piecesPlaced = { 1: 0, 2: 0 };
    let scores = { 1: 0, 2: 0 };
    let gamePhase = 'placement';
    let selectedPiece = null;
    const BOARD_SIZE = 3;
    const PIECES_PER_PLAYER = 3;
    const WIN_SCORE = 5;

    function initBoard() {
      board = Array(BOARD_SIZE).fill(null).map(() => Array(BOARD_SIZE).fill(0));
      
      // Posicionar peças automaticamente
      // Jogador 1 (vermelho) - linha de baixo
      board[2][0] = 1;
      board[2][1] = 1;
      board[2][2] = 1;
      
      // Jogador 2 (azul) - linha de cima
      board[0][0] = 2;
      board[0][1] = 2;
      board[0][2] = 2;
      
      piecesPlaced = { 1: 3, 2: 3 };
      currentPlayer = 1;
      gamePhase = 'movement';
      selectedPiece = null;
      renderBoard();
      updateTurnIndicator();
      updateMessage('🎮 Selecione uma de suas peças para movê-la!');
    }
    
    function renderBoard() {
      const boardEl = document.getElementById('board');
      boardEl.innerHTML = '';
      
      for (let r = 0; r < BOARD_SIZE; r++) {
        for (let c = 0; c < BOARD_SIZE; c++) {
          const cell = document.createElement('div');
          const isDark = (r + c) % 2 === 1;
          cell.className = `board-cell w-24 h-24 md:w-32 md:h-32 rounded-lg flex items-center justify-center text-4xl md:text-6xl ${isDark ? 'bg-amber-900/80' : 'bg-amber-700/60'}`;
          cell.dataset.row = r;
          cell.dataset.col = c;
          
          if (selectedPiece && selectedPiece[0] === r && selectedPiece[1] === c) {
            cell.style.boxShadow = '0 0 20px rgba(251, 191, 36, 0.8) inset';
            cell.style.border = '3px solid #fbbf24';
          }
          
          if (board[r][c] !== 0) {
            const isPlayer1 = board[r][c] === 1;
            const piece = document.createElement('div');
            piece.className = `piece w-20 h-20 md:w-28 md:h-28 rounded-full flex items-center justify-center text-4xl md:text-6xl shadow-lg`;
            piece.style.background = isPlayer1 
              ? 'linear-gradient(135deg, #f43f5e, #be123c)' 
              : 'linear-gradient(135deg, #3b82f6, #1d4ed8)';
            piece.style.border = `3px solid ${isPlayer1 ? '#fda4af' : '#93c5fd'}`;
            piece.innerHTML = isPlayer1 ? '🔴' : '🔵';
            cell.appendChild(piece);
          }
          
          cell.addEventListener('click', () => {
            if (gamePhase === 'placement') {
              handleCellClickPlacement(r, c);
            } else {
              handleCellClickMovement(r, c);
            }
          });
          boardEl.appendChild(cell);
        }
      }
    }

    function handleCellClickPlacement(r, c) {
      if (board[r][c] !== 0) {
        updateMessage('❌ Esta casa já está ocupada!');
        return;
      }
      
      board[r][c] = currentPlayer;
      piecesPlaced[currentPlayer]++;
      
      if (piecesPlaced[1] === PIECES_PER_PLAYER && piecesPlaced[2] === PIECES_PER_PLAYER) {
        gamePhase = 'movement';
        currentPlayer = 1;
        selectedPiece = null;
        renderBoard();
        updateTurnIndicator();
        updateMessage('🎮 Selecione uma de suas peças para movê-la!');
      } else {
        currentPlayer = currentPlayer === 1 ? 2 : 1;
        renderBoard();
        updateTurnIndicator();
        const playerName = currentPlayer === 1 ? getPlayer1Name() : getPlayer2Name();
        updateMessage(`${playerName}, é sua vez! Coloque sua peça`);
      }
    }

    function handleCellClickMovement(r, c) {
      if (board[r][c] === currentPlayer) {
        selectedPiece = [r, c];
        renderBoard();
        updateMessage('✅ Peça selecionada! Clique em uma casa adjacente para movê-la');
        return;
      }
      
      if (!selectedPiece) {
        updateMessage('❌ Selecione uma de suas peças primeiro!');
        return;
      }
      
      if (board[r][c] !== 0) {
        updateMessage('❌ Esta casa está ocupada!');
        return;
      }
      
      const [fromR, fromC] = selectedPiece;
      const distance = Math.max(Math.abs(r - fromR), Math.abs(c - fromC));
      if (distance !== 1) {
        updateMessage('❌ Só pode mover para uma casa adjacente!');
        return;
      }
      
      board[r][c] = currentPlayer;
      board[fromR][fromC] = 0;
      
      const lineResult = checkForLine(currentPlayer);
      
      if (lineResult) {
        scores[currentPlayer]++;
        updateScores();
        highlightWinningLine(lineResult);
        
        if (scores[currentPlayer] >= WIN_SCORE) {
          setTimeout(() => showVictoryModal(), 800);
        } else {
          setTimeout(() => {
            initBoard();
            const playerName = currentPlayer === 1 ? getPlayer1Name() : getPlayer2Name();
            updateMessage(`🎉 Ponto para ${playerName}! Reiniciando...`);
          }, 1500);
        }
      } else {
        selectedPiece = null;
        currentPlayer = currentPlayer === 1 ? 2 : 1;
        renderBoard();
        updateTurnIndicator();
        const playerName = currentPlayer === 1 ? getPlayer1Name() : getPlayer2Name();
        updateMessage(`${playerName}, é sua vez!`);
      }
    }

    function checkForLine(player) {
      // Linha 0 (topo - zona proibida para Jogador 2)
      // Linha 2 (baixo - zona proibida para Jogador 1)
      // Apenas a linha 1 (meio) é permitida para pontuação
      
      // Horizontal - APENAS na linha do meio (linha 1)
      if (board[1][0] === player && board[1][1] === player && board[1][2] === player) {
        return [[1, 0], [1, 1], [1, 2]];
      }
      
      // Vertical
      for (let c = 0; c < BOARD_SIZE; c++) {
        if (board[0][c] === player && board[1][c] === player && board[2][c] === player) {
          return [[0, c], [1, c], [2, c]];
        }
      }
      
      // Diagonal
      if (board[0][0] === player && board[1][1] === player && board[2][2] === player) {
        return [[0, 0], [1, 1], [2, 2]];
      }
      if (board[0][2] === player && board[1][1] === player && board[2][0] === player) {
        return [[0, 2], [1, 1], [2, 0]];
      }
      
      return null;
    }

    function highlightWinningLine(cells) {
      renderBoard();
      cells.forEach(([r, c]) => {
        const cellEl = document.querySelector(`[data-row="${r}"][data-col="${c}"]`);
        if (cellEl) {
          cellEl.classList.add('winning-cell');
          cellEl.style.background = 'rgba(251, 191, 36, 0.5)';
        }
      });
    }

    function updateTurnIndicator() {
      const name = currentPlayer === 1 ? getPlayer1Name() : getPlayer2Name();
      const emoji = currentPlayer === 1 ? '🔴' : '🔵';
      document.getElementById('current-player').textContent = `${name} ${emoji}`;
      
      document.getElementById('player1-card').classList.toggle('player-active', currentPlayer === 1);
      document.getElementById('player2-card').classList.toggle('player-active', currentPlayer === 2);
    }

    function updateScores() {
      const scoreEl = document.getElementById(`player${currentPlayer}-score`);
      scoreEl.textContent = scores[currentPlayer];
      scoreEl.classList.add('score-pop');
      setTimeout(() => scoreEl.classList.remove('score-pop'), 500);
    }

    function updateMessage(msg) {
      document.getElementById('message').textContent = msg;
    }

    function getPlayer1Name() {
      return window.elementSdk?.config?.player1_name || defaultConfig.player1_name;
    }

    function getPlayer2Name() {
      return window.elementSdk?.config?.player2_name || defaultConfig.player2_name;
    }

    function showVictoryModal() {
      const winnerName = currentPlayer === 1 ? getPlayer1Name() : getPlayer2Name();
      document.getElementById('winner-text').textContent = `${winnerName} Venceu! 🏆`;
      document.getElementById('victory-modal').classList.remove('hidden');
      document.getElementById('victory-modal').classList.add('flex');
      createConfetti();
    }

    function createConfetti() {
      const colors = ['#f43f5e', '#3b82f6', '#fbbf24', '#10b981', '#8b5cf6', '#f97316'];
      for (let i = 0; i < 50; i++) {
        setTimeout(() => {
          const confetti = document.createElement('div');
          confetti.className = 'confetti';
          confetti.style.left = Math.random() * 100 + '%';
          confetti.style.background = colors[Math.floor(Math.random() * colors.length)];
          confetti.style.animationDuration = (2 + Math.random() * 2) + 's';
          document.body.appendChild(confetti);
          setTimeout(() => confetti.remove(), 4000);
        }, i * 50);
      }
    }

    function resetGame() {
      scores = { 1: 0, 2: 0 };
      document.getElementById('player1-score').textContent = '0';
      document.getElementById('player2-score').textContent = '0';
      document.getElementById('victory-modal').classList.add('hidden');
      document.getElementById('victory-modal').classList.remove('flex');
      initBoard();
    }

    document.getElementById('rules-btn').addEventListener('click', () => {
      document.getElementById('rules-modal').classList.remove('hidden');
      document.getElementById('rules-modal').classList.add('flex');
    });

    document.getElementById('close-rules').addEventListener('click', () => {
      document.getElementById('rules-modal').classList.add('hidden');
      document.getElementById('rules-modal').classList.remove('flex');
    });

    document.getElementById('rules-modal').addEventListener('click', (e) => {
      if (e.target.id === 'rules-modal') {
        document.getElementById('rules-modal').classList.add('hidden');
        document.getElementById('rules-modal').classList.remove('flex');
      }
    });

    document.getElementById('play-again').addEventListener('click', resetGame);
    document.getElementById('reset-btn').addEventListener('click', resetGame);

    async function onConfigChange(config) {
      const title = config.game_title || defaultConfig.game_title;
      document.getElementById('game-title').textContent = title;
      document.getElementById('player1-name').textContent = getPlayer1Name();
      document.getElementById('player2-name').textContent = getPlayer2Name();
      updateTurnIndicator();
    }

    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange,
        mapToCapabilities: (config) => ({
          recolorables: [],
          borderables: [],
          fontEditable: undefined,
          fontSizeable: undefined
        }),
        mapToEditPanelValues: (config) => new Map([
          ['game_title', config.game_title || defaultConfig.game_title],
          ['player1_name', config.player1_name || defaultConfig.player1_name],
          ['player2_name', config.player2_name || defaultConfig.player2_name]
        ])
      });
    }

    initBoard();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9d2bdbd16678f630',t:'MTc3MTkwMzU1Ny4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
