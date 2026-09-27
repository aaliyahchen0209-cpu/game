<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的夢幻可愛跳舞機遊戲！</title>
    <style>
        body {
            margin: 0; padding: 0;
            background-color: #1a1525;
            font-family: Arial, sans-serif;
            display: flex; justify-content: center; align-items: center;
            height: 100vh; overflow: hidden;
            user-select: none;
        }

        /* 遊戲舞台 - 夢幻星空紫色漸層背景，100% 絕不黑屏 */
        #game-stage {
            width: 500px; height: 650px;
            background: linear-gradient(180deg, #4a3b7a 0%, #251d3a 100%);
            position: relative; border: 4px solid #ffb6c1; border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.6); overflow: hidden;
        }

        /* 頂部雙軌道 */
        .ui-header {
            position: absolute; top: 25px; left: 0; width: 100%;
            display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; z-index: 10;
        }
        .lane-zone { display: flex; gap: 10px; width: 210px; justify-content: space-between; }
        
        /* 判定框容器 */
        .receiver-container { position: relative; width: 45px; height: 45px; }
        .arrow-receiver { width: 100%; height: 100%; opacity: 0.3; transition: transform 0.05s, opacity 0.05s; }
        
        /* 🎯 判定框旋轉：使用內建白色朝上立體箭頭，自動轉向左、下、上、右 */
        .rec-l { transform: rotate(270deg); } 
        .rec-d { transform: rotate(180deg); } 
        .rec-u { transform: rotate(0deg); }   
        .rec-r { transform: rotate(90deg); }  
        
        .receiver-container.active .arrow-receiver { opacity: 1; filter: brightness(1.6) drop-shadow(0 0 8px #fff); }
        .receiver-container.active .rec-l { transform: rotate(270deg) scale(0.85); }
        .receiver-container.active .rec-d { transform: rotate(180deg) scale(0.85); }
        .receiver-container.active .rec-u { transform: rotate(0deg) scale(0.85); }
        .receiver-container.active .rec-r { transform: rotate(90deg) scale(0.85); }

        /* 🚀 飄浮音符箭頭：改用內建 SVG 向量碼，100% 免外部圖片連線，保證不卡死 */
        .note { width: 45px; height: 45px; position: absolute; z-index: 5; }
        
        /* 8 條軌道的 X 座標 */
        .x-opp-left  { left: 20px; }  .x-opp-down  { left: 74px; }  .x-opp-up    { left: 128px; }  .x-opp-right { left: 182px; }
        .x-plr-left  { left: 273px; } .x-plr-down  { left: 327px; } .x-plr-up    { left: 381px; } .x-plr-right { left: 435px; }

        /* 🎯 金色音符旋轉：因為內建的金色箭頭原圖是「朝右 ➡️」 */
        .rot-left  { transform: rotate(180deg); } /* 朝右轉180度變朝左 */
        .rot-down  { transform: rotate(90deg); }  /* 朝右轉90度變朝下 */
        .rot-up    { transform: rotate(270deg); } /* 朝右轉270度變朝上 */
        .rot-right { transform: rotate(0deg); }   /* 保持原樣朝右 */

        /* FNF 血條平衡條 */
        #health-bar-container { position: absolute; bottom: 250px; left: 50%; transform: translateX(-50%); width: 380px; height: 14px; background: #ff4757; border: 3px solid #1a1525; border-radius: 10px; z-index: 12; }
        #health-bar-fill { height: 100%; width: 50%; background: #2ed573; float: right; transition: width 0.05s ease; border-radius: 6px; }
        
        /* 雙角色預留視覺區 */
        .character-zone { position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: space-between; padding: 0 35px; box-sizing: border-box; pointer-events: none; z-index: 2; }
        .char-box { width: 120px; height: 160px; border-radius: 15px; display: flex; justify-content: center; align-items: center; color: white; font-weight: bold; font-size: 14px; border: 2px dashed rgba(255,255,255,0.3); }
        #char-l { background-color: rgba(255, 107, 129, 0.4); } /* 美少女位置 */
        #char-r { background-color: rgba(123, 237, 159, 0.4); align-self: flex-end; height: 140px; } /* 機器人位置 */

        /* UI 計分板 */
        #ui-layer { position: absolute; bottom: 280px; width: 100%; text-align: center; z-index: 10; display: flex; flex-direction: column; align-items: center; }
        #score-board { font-size: 18px; font-weight: bold; color: #fff; text-shadow: 2px 2px 4px #000; font-family: monospace; }
        #judgment-text { font-size: 28px; font-weight: bold; min-height: 35px; text-shadow: 2px 2px 4px #000; }
    </style>
</head>
<body>

    <div id="game-stage">
        <div class="ui-header">
            <!-- 左邊對手軌道（內建白色朝上判定框 SVG） -->
            <div class="lane-zone">
                <div class="receiver-container" id="cont-opp-left"><svg class="arrow-receiver rec-l" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
                <div class="receiver-container" id="cont-opp-down"><svg class="arrow-receiver rec-d" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
                <div class="receiver-container" id="cont-opp-up"><svg class="arrow-receiver rec-u" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
                <div class="receiver-container" id="cont-opp-right"><svg class="arrow-receiver rec-r" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
            </div>
            <!-- 右邊玩家軌道（內建白色朝上判定框 SVG） -->
            <div class="lane-zone">
                <div class="receiver-container" id="cont-plr-left"><svg class="arrow-receiver rec-l" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
                <div class="receiver-container" id="cont-plr-down"><svg class="arrow-receiver rec-d" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
                <div class="receiver-container" id="cont-plr-up"><svg class="arrow-receiver rec-u" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
                <div class="receiver-container" id="cont-plr-right"><svg class="arrow-receiver rec-r" viewBox="0 0 100 100"><path fill="#ffffff" d="M50 10 L15 55 L40 55 L40 90 L60 90 L60 55 L85 55 Z"/></svg></div>
            </div>
        </div>

        <div id="ui-layer">
            <div id="judgment-text"></div>
            <div id="score-board">SCORE: <span id="score">0</span></div>
        </div>

        <!-- 血條平衡條 -->
        <div id="health-bar-container"><div id="health-bar-fill"></div></div>

        <!-- 雙角色動態保留區 -->
        <div class="character-zone">
            <div id="char-l" class="char-box">美少女對手<br>(保底圖層)</div>
            <div id="char-r" class="char-box">機器人主角<br>(保底圖層)</div>
        </div>
    </div>

    <script>
        document.addEventListener("DOMContentLoaded", function() {
            const stage = document.getElementById('game-stage');
            const scoreDisplay = document.getElementById('score');
            const judgmentDisplay = document.getElementById('judgment-text');
            const barFill = document.getElementById('health-bar-fill');
            
            let score = 0; let health = 50;
            const speed = 4; const activeNotes = []; let spawnInterval;

            const keyMap = {
                'ArrowLeft': 'left', 'a': 'left', 'A': 'left',
                'ArrowDown': 'down', 's': 'down', 'S': 'down',
                'ArrowUp': 'up',     'w': 'up',   'W': 'up',
                'ArrowRight': 'right','d': 'right','D': 'right'
            };

            // 🎯 核心修正：將金色「朝右 ➡️」箭頭轉換為嵌入式 SVG 代碼，完全不需要外網連線，防跨網域卡死！
            const goldArrowSvg = `<svg viewBox="0 0 100 100" style="width:100%; height:100%;"><path fill="%23e5bc5d" stroke="%23b89336" stroke-width="3" d="M10 50 L55 15 L55 40 L90 40 L90 60 L55 60 L55 85 Z"/></svg>`;

            function initGame() {
                spawnInterval = setInterval(spawnNote, 800);
                updateHealthBar(); 
                gameLoop();
            }

            window.addEventListener('keydown', (e) => {
                if (keyMap[e.key]) {
                    const dir = keyMap[e.key];
                    const targetContainer = document.getElementById(`cont-plr-${dir}`);
                    if (targetContainer) targetContainer.classList.add('active');
                    checkHit(dir);
                }
            });

            window.addEventListener('keyup', (e) => {
                if (keyMap[e.key]) {
                    const dir = keyMap[e.key];
                    const targetContainer = document.getElementById(`cont-plr-${dir}`);
                    if (targetContainer) targetContainer.classList.remove('active');
                }
            });

            function updateHealthBar() {
                if (health < 0) health = 0; if (health > 100) health = 100;
                barFill.style.width = health + '%';
            }

            function spawnNote() {
                const directions = ['left', 'down', 'up', 'right'];
                const randomDir = directions[Math.floor(Math.random() * directions.length)];
                const targetSide = Math.random() > 0.45 ? 'plr' : 'opp';

                // 建立標準 <img> 載入內置的金色朝右箭頭
                const noteElement = document.createElement('img');
                noteElement.src = "data:image/svg+xml;utf8," + goldArrowSvg;
                noteElement.className = `note x-${targetSide}-${randomDir} rot-${randomDir}`;
                
                // 安全出生位置
                let initialY = 580; 
                noteElement.style.top = initialY + 'px';
                stage.appendChild(noteElement);

                activeNotes.push({ element: noteElement, direction: randomDir, side: targetSide, y: initialY });
            }

            function gameLoop() {
                for (let i = activeNotes.length - 1; i >= 0; i--) {
                    const note = activeNotes[i]; 
                    note.y -= speed; 
                    
