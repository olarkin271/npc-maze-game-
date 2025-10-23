<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Maze Escape Game</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        h1 {
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            margin-bottom: 20px;
        }
        
        #game-container {
            background: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
        }
        
        #maze {
            display: inline-block;
            border: 3px solid #333;
            background: #f0f0f0;
        }
        
        #info {
            margin-top: 15px;
            text-align: center;
            font-size: 18px;
            color: #333;
            font-weight: bold;
        }
        
        #controls {
            margin-top: 10px;
            text-align: center;
            color: #666;
            font-size: 14px;
        }
        
        #score {
            margin-top: 10px;
            text-align: center;
            font-size: 22px;
            color: #f59e0b;
            font-weight: bold;
        }
        
        #status {
            margin-top: 10px;
            text-align: center;
            font-size: 16px;
            min-height: 25px;
        }
        
        .win {
            color: #22c55e;
            font-weight: bold;
            animation: pulse 0.5s ease-in-out;
        }
        
        .lose {
            color: #ef4444;
            font-weight: bold;
            animation: shake 0.5s ease-in-out;
        }
        
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.1); }
        }
        
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            75% { transform: translateX(10px); }
        }
        
        button {
            margin-top: 15px;
            padding: 12px 30px;
            font-size: 16px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            transition: all 0.3s;
            display: block;
            margin-left: auto;
            margin-right: auto;
        }
        
        button:hover {
            background: #5568d3;
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
        }
        
        #instructions {
            margin-top: 15px;
            padding: 15px;
            background: #f8fafc;
            border-radius: 8px;
            text-align: left;
            font-size: 14px;
            color: #475569;
            line-height: 1.6;
        }
        
        #instructions h3 {
            margin: 0 0 10px 0;
            color: #1e293b;
        }
        
        footer {
            margin-top: 20px;
            color: white;
            text-align: center;
            font-size: 14px;
            opacity: 0.9;
        }
    </style>
</head>
<body>
    <h1>🎮 Maze Escape</h1>
    
    <div id="game-container">
        <canvas id="maze"></canvas>
        <button onclick="trainAgent()">Train AI</button>
<button onclick="runTrainedAgent()">Run AI</button>
        <div id="info">Use Arrow Keys to Move!</div>
        <div id="controls">🟦 You | 🟥 Enemy NPC | 🟩 Goal | 🪙 Coins</div>
        <div id="score">💰 Score: 0</div>
        <div id="status"></div>
        <button onclick="restartGame()">🔄 New Game</button>
        
        <div id="instructions">
            <h3>📋 How to Play</h3>
            <p><strong>Objective:</strong> Collect coins and reach the green goal before the enemy catches you!</p>
            <p><strong>Controls:</strong> Use arrow keys (↑ ↓ ← →) to navigate the maze</p>
            <p><strong>Scoring:</strong> Each coin is worth 10 points. Collect all coins for a perfect score!</p>
            <p><strong>Tips:</strong> The enemy only chases you when you're close. Plan your route carefully!</p>
        </div>
    </div>
    
    <footer>
        <p>Made with ❤️ | Perfect for GitHub Pages</p>
    </footer>

    <script>
        const canvas = document.getElementById('maze');
        const ctx = canvas.getContext('2d');
        const cellSize = 40;
        const rows = 12;
        const cols = 15;
        
        canvas.width = cols * cellSize;
        canvas.height = rows * cellSize;
        
        let player, npc, goal, maze, gameOver, coins, score, npcMoveCounter;
        
        function createMaze() {
            // Create maze with 1 = wall, 0 = path
            const m = Array(rows).fill().map(() => Array(cols).fill(1));
            
            function carve(x, y) {
                const dirs = [[0, -2], [2, 0], [0, 2], [-2, 0]];
                dirs.sort(() => Math.random() - 0.5);
                
                m[y][x] = 0;
                
                for (let [dx, dy] of dirs) {
                    const nx = x + dx;
                    const ny = y + dy;
                    
                    if (nx > 0 && nx < cols - 1 && ny > 0 && ny < rows - 1 && m[ny][nx] === 1) {
                        m[y + dy/2][x + dx/2] = 0;
                        carve(nx, ny);
                    }
                }
            }
            
            carve(1, 1);
            
            // Add extra paths to make maze more open
            for (let i = 0; i < 15; i++) {
                const x = Math.floor(Math.random() * (cols - 2)) + 1;
                const y = Math.floor(Math.random() * (rows - 2)) + 1;
                m[y][x] = 0;
            }
            
            return m;
        }
        
        function initGame() {
            maze = createMaze();
            player = {x: 1, y: 1};
            goal = {x: cols - 2, y: rows - 2};
            
            // Place NPC far from player (in opposite quadrant)
            npc = {
                x: Math.floor(cols * 0.75),
                y: Math.floor(rows * 0.75)
            };
            
            // Ensure NPC position is valid
            while (maze[npc.y][npc.x] === 1) {
                npc.x = Math.floor(Math.random() * (cols - 2)) + 1;
                npc.y = Math.floor(Math.random() * (rows - 2)) + 1;
            }
            
            score = 0;
            coins = [];
            npcMoveCounter = 0;
            
            // Generate coins in random valid positions
            for (let i = 0; i < 15; i++) {
                let coinX, coinY;
                let attempts = 0;
                do {
                    coinX = Math.floor(Math.random() * (cols - 2)) + 1;
                    coinY = Math.floor(Math.random() * (rows - 2)) + 1;
                    attempts++;
                } while (
                    attempts < 100 &&
                    (maze[coinY][coinX] === 1 || 
                    (coinX === player.x && coinY === player.y) ||
                    (coinX === goal.x && coinY === goal.y) ||
                    (Math.abs(coinX - npc.x) < 2 && Math.abs(coinY - npc.y) < 2) ||
                    coins.some(c => c.x === coinX && c.y === coinY))
                );
                if (attempts < 100) {
                    coins.push({x: coinX, y: coinY});
                }
            }
            
            maze[goal.y][goal.x] = 0;
            gameOver = false;
            document.getElementById('status').textContent = '';
            document.getElementById('status').className = '';
            updateScore();
        }
        
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw maze
            for (let y = 0; y < rows; y++) {
                for (let x = 0; x < cols; x++) {
                    if (maze[y][x] === 1) {
                        ctx.fillStyle = '#1e293b';
                        ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
                    } else {
                        ctx.fillStyle = '#f8fafc';
                        ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
                    }
                    
                    // Grid lines
                    ctx.strokeStyle = '#cbd5e1';
                    ctx.strokeRect(x * cellSize, y * cellSize, cellSize, cellSize);
                }
            }
            
            // Draw goal
            ctx.fillStyle = '#22c55e';
            ctx.fillRect(goal.x * cellSize + 5, goal.y * cellSize + 5, cellSize - 10, cellSize - 10);
            
            // Draw coins
            coins.forEach(coin => {
                ctx.fillStyle = '#f59e0b';
                ctx.beginPath();
                ctx.arc(coin.x * cellSize + cellSize/2, coin.y * cellSize + cellSize/2, 8, 0, Math.PI * 2);
                ctx.fill();
                ctx.strokeStyle = '#d97706';
                ctx.lineWidth = 2;
                ctx.stroke();
            });
            
            // Draw NPC
            ctx.fillStyle = '#ef4444';
            ctx.beginPath();
            ctx.arc(npc.x * cellSize + cellSize/2, npc.y * cellSize + cellSize/2, cellSize/2 - 5, 0, Math.PI * 2);
            ctx.fill();
            
            // Draw player
            ctx.fillStyle = '#3b82f6';
            ctx.fillRect(player.x * cellSize + 5, player.y * cellSize + 5, cellSize - 10, cellSize - 10);
        }
        
        function moveNPC() {
            if (gameOver) return;
            
            // NPC only moves every other player move (slower)
            npcMoveCounter++;
            if (npcMoveCounter < 2) return;
            npcMoveCounter = 0;
            
            const dx = player.x - npc.x;
            const dy = player.y - npc.y;
            const dist = Math.abs(dx) + Math.abs(dy);
            
            // Only chase if player is within 8 cells
            if (dist > 8) {
                // Random walk when far away
                const moves = [{x: 1, y: 0}, {x: -1, y: 0}, {x: 0, y: 1}, {x: 0, y: -1}];
                moves.sort(() => Math.random() - 0.5);
                
                for (let move of moves) {
                    const newX = npc.x + move.x;
                    const newY = npc.y + move.y;
                    
                    if (newX >= 0 && newX < cols && newY >= 0 && newY < rows && maze[newY][newX] === 0) {
                        npc.x = newX;
                        npc.y = newY;
                        break;
                    }
                }
            } else {
                // Chase player when close
                let moves = [];
                if (Math.abs(dx) > Math.abs(dy)) {
                    moves = dx > 0 ? [{x: 1, y: 0}, {x: 0, y: dy > 0 ? 1 : -1}] : [{x: -1, y: 0}, {x: 0, y: dy > 0 ? 1 : -1}];
                } else {
                    moves = dy > 0 ? [{x: 0, y: 1}, {x: dx > 0 ? 1 : -1, y: 0}] : [{x: 0, y: -1}, {x: dx > 0 ? 1 : -1, y: 0}];
                }
                
                for (let move of moves) {
                    const newX = npc.x + move.x;
                    const newY = npc.y + move.y;
                    
                    if (newX >= 0 && newX < cols && newY >= 0 && newY < rows && maze[newY][newX] === 0) {
                        npc.x = newX;
                        npc.y = newY;
                        break;
                    }
                }
            }
            
            checkCollisions();
        }
        
        function checkCollisions() {
            // Check coin collection
            const coinIndex = coins.findIndex(c => c.x === player.x && c.y === player.y);
            if (coinIndex !== -1) {
                coins.splice(coinIndex, 1);
                score += 10;
                updateScore();
            }
            
            if (player.x === npc.x && player.y === npc.y) {
                gameOver = true;
                const status = document.getElementById('status');
                status.textContent = '💀 Caught by the enemy! Try again!';
                status.className = 'lose';
            }
            
            if (player.x === goal.x && player.y === goal.y) {
                gameOver = true;
                const status = document.getElementById('status');
                const bonus = coins.length === 0 ? ' Perfect score! 🌟' : '';
                status.textContent = `🎉 You escaped! Final Score: ${score}${bonus}`;
                status.className = 'win';
            }
        }
        
        function updateScore() {
            document.getElementById('score').textContent = `💰 Score: ${score}`;
        }
        
        document.addEventListener('keydown', (e) => {
            if (gameOver) return;
            
            let newX = player.x;
            let newY = player.y;
            
            switch(e.key) {
                case 'ArrowUp': newY--; break;
                case 'ArrowDown': newY++; break;
                case 'ArrowLeft': newX--; break;
                case 'ArrowRight': newX++; break;
                default: return;
            }
            
            e.preventDefault();
            
            if (newX >= 0 && newX < cols && newY >= 0 && newY < rows && maze[newY][newX] === 0) {
                player.x = newX;
                player.y = newY;
                checkCollisions();
                draw();
                
                // NPC moves after player
                setTimeout(() => {
                    moveNPC();
                    draw();
                }, 100);
            }
        });
        
        function restartGame() {
            initGame();
            draw();
        }
        
        initGame();
        draw();
        <!-- === Q-Learning Agent Integration === -->
<script>
class QAgent {
  constructor(alpha=0.1, gamma=0.9, epsilon=0.2) {
    this.alpha = alpha;
    this.gamma = gamma;
    this.epsilon = epsilon;
    this.q = JSON.parse(localStorage.getItem("qTable") || "{}");
  }
  key(s,a){ return `${s.x},${s.y}_${a}`; }
  getQ(s,a){ return this.q[this.key(s,a)] || 0; }
  chooseAction(s,actions){
    if(Math.random()<this.epsilon) return actions[Math.floor(Math.random()*actions.length)];
    let best=actions[0],bestVal=-Infinity;
    for(let a of actions){ let v=this.getQ(s,a); if(v>bestVal){best=a;bestVal=v;} }
    return best;
  }
  update(s,a,r,ns,actions){
    let maxNext=Math.max(...actions.map(a2=>this.getQ(ns,a2)));
    let old=this.getQ(s,a);
    this.q[this.key(s,a)]=old+this.alpha*(r+this.gamma*maxNext-old);
  }
  save(){ localStorage.setItem("qTable",JSON.stringify(this.q)); }
}

const ACTIONS=["UP","DOWN","LEFT","RIGHT"];
const agent=new QAgent();

function envStep(action){
  // Use your movePlayer logic here:
  let reward=-1, done=false;
  movePlayer(action); // existing movement
  if(player.hitEnemy){ reward=-100; done=true; }
  else if(player.gotCoin){ reward=+10; player.gotCoin=false; }
  else if(player.atGoal){ reward=+50; done=true; }
  return {nextState:{x:player.x,y:player.y},reward,done};
}

async function trainAgent(episodes=200){
  for(let ep=0;ep<episodes;ep++){
    resetGame(); // your existing reset/start
    let state={x:player.x,y:player.y},done=false;
    while(!done){
      let act=agent.chooseAction(state,ACTIONS);
      let {nextState,reward,done:finished}=envStep(act);
      agent.update(state,act,reward,nextState,ACTIONS);
      state=nextState;
      done=finished;
      await new Promise(r=>setTimeout(r,5)); // speed of training
    }
  }
  agent.save();
  alert("Training complete!");
}

function runTrainedAgent(){
  agent.epsilon=0;
  resetGame();
  let state={x:player.x,y:player.y};
  function stepLoop(){
    let act=agent.chooseAction(state,ACTIONS);
    let {nextState,reward,done}=envStep(act);
    state=nextState;
    if(!done) requestAnimationFrame(stepLoop);
  }
  stepLoop();
}
</script>
    </script>
</body>
</html>
