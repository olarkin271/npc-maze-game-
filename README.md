<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Maze Escape Game</title>
<style>
/* ——— existing CSS unchanged ——— */
body{margin:0;padding:20px;display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);font-family:'Segoe UI',Tahoma,Geneva,Verdana,sans-serif;}
h1{color:white;text-shadow:2px 2px 4px rgba(0,0,0,0.3);margin-bottom:20px;}
#game-container{background:white;padding:25px;border-radius:15px;box-shadow:0 10px 40px rgba(0,0,0,0.3);}
#maze{display:inline-block;border:3px solid #333;background:#f0f0f0;}
#info{margin-top:15px;text-align:center;font-size:18px;color:#333;font-weight:bold;}
#controls{margin-top:10px;text-align:center;color:#666;font-size:14px;}
#score{margin-top:10px;text-align:center;font-size:22px;color:#f59e0b;font-weight:bold;}
#status{margin-top:10px;text-align:center;font-size:16px;min-height:25px;}
.win{color:#22c55e;font-weight:bold;animation:pulse .5s ease-in-out;}
.lose{color:#ef4444;font-weight:bold;animation:shake .5s ease-in-out;}
@keyframes pulse{0%,100%{transform:scale(1);}50%{transform:scale(1.1);}}
@keyframes shake{0%,100%{transform:translateX(0);}25%{transform:translateX(-10px);}75%{transform:translateX(10px);}}
button{margin-top:15px;padding:12px 30px;font-size:16px;background:#667eea;color:white;border:none;border-radius:8px;cursor:pointer;font-weight:bold;transition:all .3s;display:block;margin-left:auto;margin-right:auto;}
button:hover{background:#5568d3;transform:translateY(-2px);box-shadow:0 5px 15px rgba(102,126,234,.4);}
#instructions{margin-top:15px;padding:15px;background:#f8fafc;border-radius:8px;text-align:left;font-size:14px;color:#475569;line-height:1.6;}
#instructions h3{margin:0 0 10px 0;color:#1e293b;}
footer{margin-top:20px;color:white;text-align:center;font-size:14px;opacity:.9;}
</style>
</head>
<body>
<h1>🎮 Maze Escape</h1>

<div id="game-container">
  <canvas id="maze"></canvas>
  <div id="info">Use Arrow Keys to Move!</div>
  <div id="controls">🟦 You | 🟥 Enemy NPC | 🟩 Goal | 🪙 Coins</div>
  <div id="score">💰 Score: 0</div>
  <div id="status"></div>
  <button onclick="restartGame()">🔄 New Game</button>

  <!-- === AI Buttons Added Below Canvas === -->
  <div style="text-align:center; margin-top:10px;">
    <button onclick="trainAgent()">🤖 Train AI</button>
    <button onclick="runTrainedAgent()">▶️ Run AI</button>
  </div>

  <div id="instructions">
    <h3>📋 How to Play</h3>
    <p><strong>Objective:</strong> Collect coins and reach the green goal before the enemy catches you!</p>
    <p><strong>Controls:</strong> Use arrow keys (↑ ↓ ← →) to navigate the maze</p>
    <p><strong>Scoring:</strong> Each coin is worth 10 points. Collect all coins for a perfect score!</p>
    <p><strong>Tips:</strong> The enemy only chases you when you're close. Plan your route carefully!</p>
  </div>
</div>

<footer><p>Made with ❤️ | Perfect for GitHub Pages</p></footer>

<script>
/* === Your original game logic (unchanged) === */
const canvas=document.getElementById('maze'),ctx=canvas.getContext('2d');
const cellSize=40,rows=12,cols=15;
canvas.width=cols*cellSize;canvas.height=rows*cellSize;
let player,npc,goal,maze,gameOver,coins,score,npcMoveCounter;

function createMaze(){const m=Array(rows).fill().map(()=>Array(cols).fill(1));
function carve(x,y){const dirs=[[0,-2],[2,0],[0,2],[-2,0]];dirs.sort(()=>Math.random()-.5);
m[y][x]=0;for(let[dx,dy]of dirs){const nx=x+dx,ny=y+dy;if(nx>0&&nx<cols-1&&ny>0&&ny<rows-1&&m[ny][nx]===1){m[y+dy/2][x+dx/2]=0;carve(nx,ny);}}}
carve(1,1);for(let i=0;i<15;i++){const x=Math.floor(Math.random()*(cols-2))+1,y=Math.floor(Math.random()*(rows-2))+1;m[y][x]=0;}return m;}

function initGame(){maze=createMaze();player={x:1,y:1};goal={x:cols-2,y:rows-2};
npc={x:Math.floor(cols*.75),y:Math.floor(rows*.75)};
while(maze[npc.y][npc.x]===1){npc.x=Math.floor(Math.random()*(cols-2))+1;npc.y=Math.floor(Math.random()*(rows-2))+1;}
score=0;coins=[];npcMoveCounter=0;
for(let i=0;i<15;i++){let coinX,coinY,attempts=0;do{coinX=Math.floor(Math.random()*(cols-2))+1;coinY=Math.floor(Math.random()*(rows-2))+1;attempts++;}while(attempts<100&&(maze[coinY][coinX]===1|
