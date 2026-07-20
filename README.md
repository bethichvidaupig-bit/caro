[index.html](https://github.com/user-attachments/files/30183682/index.html)
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<title>Cờ Caro Online - 5 quân liên tiếp</title>
<style>
  :root{
    --paper:#f6f1e4; --ink:#2b2620; --line:#c9bfa5;
    --x-color:#b5442e; --o-color:#2a6f6f; --win:#e0a72e;
  }
  *{box-sizing:border-box;}
  body{
    margin:0; min-height:100vh; display:flex; align-items:center; justify-content:center;
    background:var(--paper);
    background-image:radial-gradient(circle at 1px 1px, rgba(43,38,32,0.06) 1px, transparent 0);
    background-size:14px 14px;
    font-family:'Georgia','Times New Roman',serif;
    color:var(--ink); padding:16px;
  }
  .wrap{ text-align:center; max-width:520px; margin:0 auto; }
  h1{ font-size:2rem; letter-spacing:2px; margin:0 0 4px; font-weight:700; }
  h1 .x{color:var(--x-color);} h1 .o{color:var(--o-color);}
  .subtitle{ font-size:0.82rem; color:#7a7160; margin-bottom:16px; font-style:italic; }

  .lobby-card{ background:rgba(255,255,255,0.4); border:1.5px solid var(--line); border-radius:12px; padding:20px; }
  .lobby-card h2{ font-size:1rem; margin:0 0 12px; }
  .lobby-btns{ display:flex; flex-direction:column; gap:10px; margin-bottom:14px; }
  input[type=text]{
    font-family:inherit; font-size:1rem; padding:9px 12px; border-radius:8px;
    border:1.5px solid var(--line); background:var(--paper); color:var(--ink);
    text-align:center; width:100%; margin-bottom:10px;
  }
  button{
    font-family:inherit; font-size:0.9rem; padding:9px 18px; border-radius:20px;
    border:1.5px solid var(--ink); background:transparent; color:var(--ink);
    cursor:pointer; transition:background 0.15s, color 0.15s;
  }
  button:hover{ background:var(--ink); color:var(--paper); }
  .room-code{ font-size:1.6rem; letter-spacing:3px; font-weight:bold; margin:10px 0; color:var(--x-color); user-select:all; }
  .status-line{ font-size:0.85rem; color:#7a7160; margin-top:8px; min-height:1.4em; }

  #status{ font-size:1.05rem; margin-bottom:10px; min-height:1.5em; }
  .turn-x{color:var(--x-color); font-weight:bold;}
  .turn-o{color:var(--o-color); font-weight:bold;}
  .you-tag{ font-size:0.78rem; color:#7a7160; margin-bottom:10px;}

  .board-scroll{ max-width:100%; overflow:auto; display:flex; justify-content:center; }
  .board{
    position:relative; width:450px; height:450px; flex:0 0 auto;
    background:rgba(255,255,255,0.25); border:1.5px solid var(--line); border-radius:6px;
  }
  .board svg.grid{ position:absolute; top:0; left:0; width:100%; height:100%; pointer-events:none; }
  .board svg.grid line{ stroke:var(--line); stroke-width:1; }
  .cells{
    position:relative; display:grid;
    grid-template-columns:repeat(15,1fr); grid-template-rows:repeat(15,1fr);
    width:100%; height:100%;
  }
  .cell{ display:flex; align-items:center; justify-content:center; cursor:pointer; position:relative; }
  .cell svg{ width:70%; height:70%; overflow:visible; }
  .cell path, .cell circle{ fill:none; stroke-width:6; stroke-linecap:round; stroke-dasharray:200; stroke-dashoffset:200; animation:draw 0.25s ease forwards; }
  .cell.x path{stroke:var(--x-color);} .cell.o circle{stroke:var(--o-color);}
  @keyframes draw{ to{stroke-dashoffset:0;} }
  .cell.win{ background:rgba(224,167,46,0.35); border-radius:4px; }

  .scoreboard{ display:flex; justify-content:center; gap:22px; margin:16px 0 6px; font-size:0.9rem; }
  .score-item{ display:flex; flex-direction:column; align-items:center; min-width:56px; }
  .score-item .label{ font-weight:bold; }
  .score-item.x .label{color:var(--x-color);} .score-item.o .label{color:var(--o-color);}
  .score-item .val{ font-size:1.2rem; }
  .controls{ margin-top:14px; display:flex; gap:10px; justify-content:center; }
  .hidden{ display:none !important; }
</style>
</head>
<body>
<div class="wrap">
  <h1><span class="x">X</span> · <span class="o">O</span></h1>
  <div class="subtitle">cờ caro online — 15×15, thắng khi có 5 quân liên tiếp</div>

  <div id="lobby" class="lobby-card">
    <div id="lobbyHome">
      <h2>Bạn muốn:</h2>
      <div class="lobby-btns">
        <button id="createBtn">Tạo phòng mới</button>
        <button id="joinBtn">Nhập mã để tham gia</button>
      </div>
      <div class="status-line">Người tạo phòng chơi X, người tham gia chơi O.</div>
    </div>

    <div id="lobbyCreate" class="hidden">
      <h2>Gửi mã này cho bạn của bạn:</h2>
      <div class="room-code" id="roomCodeDisplay">...</div>
      <div class="status-line" id="createStatus">Đang chờ người chơi kia tham gia...</div>
    </div>

    <div id="lobbyJoin" class="hidden">
      <h2>Nhập mã phòng bạn nhận được:</h2>
      <input type="text" id="joinInput" placeholder="Mã phòng">
      <button id="joinConfirmBtn">Kết nối</button>
      <div class="status-line" id="joinStatus"></div>
    </div>
  </div>

  <div id="game" class="hidden">
    <div id="status">Đang chờ...</div>
    <div class="you-tag" id="youTag"></div>

    <div class="board-scroll">
      <div class="board">
        <svg class="grid" id="gridSvg" viewBox="0 0 450 450"></svg>
        <div class="cells" id="cells"></div>
      </div>
    </div>

    <div class="scoreboard">
      <div class="score-item x"><span class="label">X</span><span class="val" id="scoreX">0</span></div>
      <div class="score-item"><span class="label">Hòa</span><span class="val" id="scoreD">0</span></div>
      <div class="score-item o"><span class="label">O</span><span class="val" id="scoreO">0</span></div>
    </div>

    <div class="controls">
      <button id="newRoundBtn">Ván mới</button>
    </div>
  </div>
</div>

<script>
const N = 15;
const WIN_COUNT = 5;
const TOTAL = N * N;

let ws;
let mySymbol = null;
let board = Array(TOTAL).fill("");
let current = "X";
let gameOver = false;
let score = { X:0, O:0, D:0 };

const el = id => document.getElementById(id);
function shortId(){ return Math.random().toString(36).substring(2,7).toUpperCase(); }

function buildGrid(){
  const svg = el("gridSvg");
  const cell = 450 / N;
  let lines = "";
  for(let i=0;i<=N;i++){
    const pos = i*cell;
    lines += `<line x1="${pos}" y1="0" x2="${pos}" y2="450"/>`;
    lines += `<line x1="0" y1="${pos}" x2="450" y2="${pos}"/>`;
  }
  svg.innerHTML = lines;
}
buildGrid();

function connectWS(roomId){
  const proto = location.protocol === "https:" ? "wss://" : "ws://";
  ws = new WebSocket(proto + location.host);

  ws.addEventListener("open", () => {
    ws.send(JSON.stringify({ type:"join", room: roomId }));
  });

  ws.addEventListener("message", (event) => {
    const data = JSON.parse(event.data);
    if(data.type === "joined"){
      mySymbol = data.symbol;
    } else if(data.type === "start"){
      startGame();
    } else if(data.type === "room_full"){
      el("joinStatus").textContent = "Phòng đã đầy, thử mã khác.";
    } else if(data.type === "opponent_left"){
      el("status").textContent = "Đối thủ đã rời phòng.";
    } else if(data.type === "move"){
      applyMove(data.index, false);
    } else if(data.type === "newRound"){
      newRound(false);
    }
  });

  ws.addEventListener("error", () => {
    el("createStatus").textContent = "Lỗi kết nối tới server.";
    el("joinStatus").textContent = "Lỗi kết nối tới server.";
  });
}

el("createBtn").addEventListener("click", () => {
  el("lobbyHome").classList.add("hidden");
  el("lobbyCreate").classList.remove("hidden");
  const roomId = shortId();
  el("roomCodeDisplay").textContent = roomId;
  connectWS(roomId);
});

el("joinBtn").addEventListener("click", () => {
  el("lobbyHome").classList.add("hidden");
  el("lobbyJoin").classList.remove("hidden");
});

el("joinConfirmBtn").addEventListener("click", () => {
  const code = el("joinInput").value.trim().toUpperCase();
  if(!code) return;
  el("joinStatus").textContent = "Đang kết nối...";
  connectWS(code);
});

function startGame(){
  el("lobby").classList.add("hidden");
  el("game").classList.remove("hidden");
  el("youTag").textContent = "Bạn là: " + mySymbol;
  render();
  updateStatus();
}

function xSVG(){ return `<svg viewBox="0 0 60 60"><path d="M8,8 L52,52"/><path d="M52,8 L8,52"/></svg>`; }
function oSVG(){ return `<svg viewBox="0 0 60 60"><circle cx="30" cy="30" r="22" style="stroke-dasharray:140;stroke-dashoffset:140;"/></svg>`; }

function render(){
  const cellsEl = el("cells");
  cellsEl.innerHTML = "";
  board.forEach((val, i) => {
    const div = document.createElement("div");
    div.className = "cell" + (val ? " " + val.toLowerCase() : "");
    if(val === "X") div.innerHTML = xSVG();
    if(val === "O") div.innerHTML = oSVG();
    div.addEventListener("click", () => handleClick(i));
    cellsEl.appendChild(div);
  });
}

function handleClick(i){
  if(gameOver || board[i] !== "" || current !== mySymbol) return;
  applyMove(i, true);
}

function applyMove(i, sendIt){
  if(board[i] !== "" || gameOver) return;
  board[i] = current;
  render();

  const line = checkWinFrom(i);
  if(line){
    gameOver = true;
    highlightWin(line);
    score[current]++;
    updateScore();
    el("status").innerHTML = `<span class="turn-${current.toLowerCase()}">${current}</span> đã thắng! 🎉`;
  } else if(!board.includes("")){
    gameOver = true;
    score.D++;
    updateScore();
    el("status").textContent = "Hòa!";
  } else {
    current = current === "X" ? "O" : "X";
    updateStatus();
  }

  if(sendIt && ws && ws.readyState === WebSocket.OPEN){
    ws.send(JSON.stringify({ type:"move", index:i }));
  }
}

function checkWinFrom(idx){
  const symbol = board[idx];
  const row = Math.floor(idx / N);
  const col = idx % N;
  const directions = [[0,1],[1,0],[1,1],[1,-1]];

  for(const [dr,dc] of directions){
    let cells = [idx];
    let r=row+dr, c=col+dc;
    while(r>=0 && r<N && c>=0 && c<N && board[r*N+c]===symbol){
      cells.push(r*N+c); r+=dr; c+=dc;
    }
    r=row-dr; c=col-dc;
    while(r>=0 && r<N && c>=0 && c<N && board[r*N+c]===symbol){
      cells.push(r*N+c); r-=dr; c-=dc;
    }
    if(cells.length >= WIN_COUNT) return cells;
  }
  return null;
}

function highlightWin(cellsIdx){
  const cellsEl = el("cells").children;
  cellsIdx.forEach(i => cellsEl[i].classList.add("win"));
}
function updateScore(){
  el("scoreX").textContent = score.X;
  el("scoreO").textContent = score.O;
  el("scoreD").textContent = score.D;
}
function updateStatus(){
  if(gameOver) return;
  const turnText = current === mySymbol ? "Đến lượt bạn" : "Đang chờ đối thủ";
  el("status").innerHTML = `${turnText} (<span class="turn-${current.toLowerCase()}">${current}</span>)`;
}

function newRound(sendIt){
  board = Array(TOTAL).fill("");
  current = "X";
  gameOver = false;
  render();
  updateStatus();
  if(sendIt && ws && ws.readyState === WebSocket.OPEN){
    ws.send(JSON.stringify({ type:"newRound" }));
  }
}
el("newRoundBtn").addEventListener("click", () => newRound(true));
</script>
</body>
</html>
