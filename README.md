const status = document.getElementById('status');
const cells = document.querySelectorAll('.cell');
const startRed = document.getElementById('start-red');
const startBlue = document.getElementById('start-blue');
const restart = document.getElementById('restart');
const soundToggle = document.getElementById('soundToggle');
const placeSound = document.getElementById('placeSound');
const winSound = document.getElementById('winSound');

let currentPlayer = 'red';
let dragged = null;
let placed = Array(9).fill(null);
let redCount = 0;
let blueCount = 0;
let phase = 'placement';

function playSound(sound) {
  if (soundToggle.checked) sound.play();
}

function checkWin() {
  const winPatterns = [
    [0,1,2],[3,4,5],[6,7,8],
    [0,3,6],[1,4,7],[2,5,8],
    [0,4,8],[2,4,6]
  ];
  return winPatterns.some(pattern =>
    pattern.every(i => placed[i] === currentPlayer)
  );
}

function switchPlayer() {
  currentPlayer = currentPlayer === 'red' ? 'blue' : 'red';
  status.textContent = currentPlayer === 'red' ? "Player 1's Turn" : "Player 2's Turn";
}

function handleDrop(cell, index) {
  if (!dragged || placed[index]) return;

  cell.appendChild(dragged);
  placed[index] = currentPlayer;
  playSound(placeSound);

  if (checkWin()) {
    status.textContent = currentPlayer === 'red' ? "Player 1 Wins!" : "Player 2 Wins!";
    playSound(winSound);
    phase = 'ended';
    return;
  }

  if (currentPlayer === 'red') redCount++;
  else blueCount++;

  if (redCount >= 3 && blueCount >= 3) {
    phase = 'moving';
    status.textContent = "Move your pieces!";
  } else {
    switchPlayer();
  }
}

function initPieces() {
  startRed.innerHTML = '';
  startBlue.innerHTML = '';
  for (let i = 0; i < 3; i++) {
    const red = document.createElement('div');
    red.classList.add('ball', 'red');
    red.draggable = true;
    red.addEventListener('dragstart', () => dragged = red);
    startRed.appendChild(red);

    const blue = document.createElement('div');
    blue.classList.add('ball', 'blue');
    blue.draggable = true;
    blue.addEventListener('dragstart', () => dragged = blue);
    startBlue.appendChild(blue);
  }
}

cells.forEach((cell, index) => {
  cell.addEventListener('dragover', e => {
    e.preventDefault();
    cell.classList.add('drag-over');
  });

  cell.addEventListener('dragleave', () => {
    cell.classList.remove('drag-over');
  });

  cell.addEventListener('drop', () => {
    cell.classList.remove('drag-over');
    if (phase !== 'ended') handleDrop(cell, index);
  });
});

restart.addEventListener('click', () => {
  placed.fill(null);
  phase = 'placement';
  currentPlayer = 'red';
  redCount = 0;
  blueCount = 0;
  status.textContent = "Player 1's Turn";
  initPieces();
  cells.forEach(cell => cell.innerHTML = '');
});

initPieces();