# Freemaxshot
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Jogo de Tiro Mobile</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: none;
        }
        
        body {
            overflow: hidden;
            background-color: #111;
            position: fixed;
            width: 100%;
            height: 100%;
            font-family: Arial, sans-serif;
        }
        
        #gameCanvas {
            display: block;
            width: 100%;
            height: 100%;
            background-color: #000;
        }
        
        #uiContainer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }
        
        #scoreDisplay {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-size: 24px;
            text-shadow: 1px 1px 2px black;
        }
        
        #healthBar {
            position: absolute;
            top: 10px;
            right: 10px;
            width: 100px;
            height: 20px;
            background-color: rgba(255, 0, 0, 0.3);
            border: 2px solid white;
            border-radius: 5px;
            overflow: hidden;
        }
        
        #healthFill {
            height: 100%;
            width: 100%;
            background-color: red;
            transition: width 0.3s;
        }
        
        #joystick {
            position: absolute;
            bottom: 30px;
            left: 30px;
            width: 80px;
            height: 80px;
            background-color: rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            pointer-events: auto;
        }
        
        #joystickKnob {
            position: absolute;
            width: 40px;
            height: 40px;
            background-color: rgba(255, 255, 255, 0.5);
            border-radius: 50%;
            top: 20px;
            left: 20px;
        }
        
        #shootButton {
            position: absolute;
            bottom: 30px;
            right: 30px;
            width: 80px;
            height: 80px;
            background-color: rgba(255, 0, 0, 0.5);
            border-radius: 50%;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 24px;
            pointer-events: auto;
            user-select: none;
        }
        
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 10;
        }
        
        #startButton {
            margin-top: 20px;
            padding: 15px 30px;
            background-color: #4CAF50;
            border: none;
            border-radius: 5px;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    
    <div id="uiContainer">
        <div id="scoreDisplay">Pontos: 0</div>
        <div id="healthBar"><div id="healthFill"></div></div>
        <div id="joystick"><div id="joystickKnob"></div></div>
        <div id="shootButton">ATIRAR</div>
    </div>
    
    <div id="startScreen">
        <h1>Jogo de Tiro Mobile</h1>
        <p>Use o joystick para mover e o botão para atirar</p>
        <button id="startButton">Começar</button>
    </div>

    <script>
        // Configurações do jogo
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const healthFill = document.getElementById('healthFill');
        const joystick = document.getElementById('joystick');
        const joystickKnob = document.getElementById('joystickKnob');
        const shootButton = document.getElementById('shootButton');
        const startScreen = document.getElementById('startScreen');
        const startButton = document.getElementById('startButton');

        // Ajustar tamanho do canvas
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // Variáveis do jogo
        let score = 0;
        let gameRunning = false;
        let player;
        let bullets = [];
        let enemies = [];
        let lastEnemySpawn = 0;
        let joystickActive = false;
        let joystickAngle = 0;
        let joystickDistance = 0;
        let joystickMaxDistance = 30;
        let lastShot = 0;
        let shootCooldown = 300; // ms

        // Classe do Jogador
        class Player {
            constructor() {
                this.x = canvas.width / 2;
                this.y = canvas.height / 2;
                this.radius = 20;
                this.color = '#3498db';
                this.speed = 5;
                this.health = 100;
                this.maxHealth = 100;
                this.shootDelay = 300; // ms
                this.lastShot = 0;
            }

            draw() {
                // Desenhar jogador
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
                
                // Desenhar "arma" ou direção
                ctx.beginPath();
                ctx.moveTo(this.x, this.y);
                ctx.lineTo(
                    this.x + Math.cos(this.angle) * this.radius * 1.5,
                    this.y + Math.sin(this.angle) * this.radius * 1.5
                );
                ctx.strokeStyle = 'white';
                ctx.lineWidth = 3;
                ctx.stroke();
                ctx.closePath();
            }

            update() {
                // Atualizar ângulo para mirar na direção do toque de tiro
                this.angle = Math.atan2(
                    shootTouch.y - this.y,
                    shootTouch.x - this.x
                );
                
                // Movimento com joystick
                if (joystickActive) {
                    this.x += Math.cos(joystickAngle) * this.speed * (joystickDistance / joystickMaxDistance);
                    this.y += Math.sin(joystickAngle) * this.speed * (joystickDistance / joystickMaxDistance);
                }
                
                // Manter jogador dentro do canvas
                this.x = Math.max(this.radius, Math.min(canvas.width - this.radius, this.x));
                this.y = Math.max(this.radius, Math.min(canvas.height - this.radius, this.y));
            }
            
            takeDamage(amount) {
                this.health -= amount;
                healthFill.style.width = `${(this.health / this.maxHealth) * 100}%`;
                
                if (this.health <= 0) {
                    gameOver();
                }
            }
            
            shoot() {
                const now = Date.now();
                if (now - this.lastShot < this.shootDelay) return;
                
                this.lastShot = now;
                
                bullets.push(new Bullet(
                    this.x,
                    this.y,
                    this.angle
                ));
            }
        }

        // Classe da Bala
        class Bullet {
            constructor(x, y, angle) {
                this.x = x;
                this.y = y;
                this.radius = 5;
                this.color = '#f1c40f';
                this.speed = 10;
                this.angle = angle;
                this.damage = 10;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }

            update() {
                this.x += Math.cos(this.angle) * this.speed;
                this.y += Math.sin(this.angle) * this.speed;
                
                // Verificar se a bala saiu da tela
                return this.x < -this.radius || this.x > canvas.width + this.radius || 
                       this.y < -this.radius || this.y > canvas.height + this.radius;
            }
        }

        // Classe do Inimigo
        class Enemy {
            constructor() {
                // Posição aleatória nas bordas
                const side = Math.floor(Math.random() * 4);
                
                if (side === 0) { // Topo
                    this.x = Math.random() * canvas.width;
                    this.y = -20;
                } else if (side === 1) { // Direita
                    this.x = canvas.width + 20;
                    this.y = Math.random() * canvas.height;
                } else if (side === 2) { // Baixo
                    this.x = Math.random() * canvas.width;
                    this.y = canvas.height + 20;
                } else { // Esquerda
                    this.x = -20;
                    this.y = Math.random() * canvas.height;
                }
                
                this.radius = 15 + Math.random() * 15; // Tamanho variado
                this.color = `hsl(${Math.random() * 60}, 100%, 50%)`; // Tons de vermelho/amarelo
                this.speed = 1 + Math.random() * 2;
                this.health = this.radius * 1.5;
                this.damage = 5;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }

            update() {
                // Mover em direção ao jogador
                const dx = player.x - this.x;
                const dy = player.y - this.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                this.x += dx / distance * this.speed;
                this.y += dy / distance * this.speed;
                
                // Verificar colisão com o jogador
                if (distance < this.radius + player.radius) {
                    player.takeDamage(this.damage);
                    return true; // Remover inimigo após causar dano
                }
                
                return false;
            }
        }

        // Variáveis de controle touch
        let joystickTouch = { id: null, x: 0, y: 0 };
        let shootTouch = { x: canvas.width - 100, y: canvas.height - 100 };

        // Configurar controles touch
        joystick.addEventListener('touchstart', handleJoystickStart);
        joystick.addEventListener('touchmove', handleJoystickMove);
        joystick.addEventListener('touchend', handleJoystickEnd);
        
        shootButton.addEventListener('touchstart', handleShootStart);
        shootButton.addEventListener('touchmove', handleShootMove);
        shootButton.addEventListener('touchend', handleShootEnd);
        
        // Também funciona para mouse (para teste em desktop)
        joystick.addEventListener('mousedown', handleJoystickStart);
        window.addEventListener('mousemove', handleJoystickMove);
        window.addEventListener('mouseup', handleJoystickEnd);
        
        shootButton.addEventListener('mousedown', handleShootStart);
        window.addEventListener('mousemove', handleShootMove);
        window.addEventListener('mouseup', handleShootEnd);

        function handleJoystickStart(e) {
            e.preventDefault();
            const touch = e.type === 'touchstart' ? e.touches[0] : e;
            const rect = joystick.getBoundingClientRect();
            
            joystickTouch.id = touch.identifier;
            joystickTouch.x = touch.clientX - rect.left - rect.width / 2;
            joystickTouch.y = touch.clientY - rect.top - rect.height / 2;
            
            joystickActive = true;
            updateJoystick();
        }

        function handleJoystickMove(e) {
            if (!joystickActive) return;
            e.preventDefault();
            
            let touch;
            if (e.type === 'touchmove') {
                for (let i = 0; i < e.touches.length; i++) {
                    if (e.touches[i].identifier === joystickTouch.id) {
                        touch = e.touches[i];
                        break;
                    }
                }
                if (!touch) return;
            } else {
                touch = e;
            }
            
            const rect = joystick.getBoundingClientRect();
            joystickTouch.x = touch.clientX - rect.left - rect.width / 2;
            joystickTouch.y = touch.clientY - rect.top - rect.height / 2;
            
            updateJoystick();
        }

        function handleJoystickEnd(e) {
            if (e.type === 'touchend') {
                for (let i = 0; i < e.changedTouches.length; i++) {
                    if (e.changedTouches[i].identifier === joystickTouch.id) {
                        joystickActive = false;
                        joystickKnob.style.transform = 'translate(20px, 20px)';
                        break;
                    }
                }
            } else {
                joystickActive = false;
                joystickKnob.style.transform = 'translate(20px, 20px)';
            }
        }

        function updateJoystick() {
            const distance = Math.sqrt(joystickTouch.x * joystickTouch.x + joystickTouch.y * joystickTouch.y);
            const angle = Math.atan2(joystickTouch.y, joystickTouch.x);
            
            joystickDistance = Math.min(distance, joystickMaxDistance);
            joystickAngle = angle;
            
            const limitedX = Math.cos(angle) * joystickDistance;
            const limitedY = Math.sin(angle) * joystickDistance;
            
            joystickKnob.style.transform = `translate(${20 + limitedX}px, ${20 + limitedY}px)`;
        }

        function handleShootStart(e) {
            e.preventDefault();
            const touch = e.type === 'touchstart' ? (e.touches[0] || e.changedTouches[0]) : e;
            const rect = shootButton.getBoundingClientRect();
            
            shootTouch.x = touch.clientX;
            shootTouch.y = touch.clientY;
            
            // Atirar imediatamente ao tocar
            player.shoot();
        }

        function handleShootMove(e) {
            e.preventDefault();
            
            let touch;
            if (e.type === 'touchmove') {
                // Encontrar o toque correspondente ao botão de tiro
                for (let i = 0; i < e.touches.length; i++) {
                    const rect = shootButton.getBoundingClientRect();
                    const x = e.touches[i].clientX - rect.left;
                    const y = e.touches[i].clientY - rect.top;
                    
                    if (x >= 0 && x <= rect.width && y >= 0 && y <= rect.height) {
                        touch = e.touches[i];
                        break;
                    }
                }
                if (!touch) return;
            } else {
                touch = e;
            }
            
            shootTouch.x = touch.clientX;
            shootTouch.y = touch.clientY;
            
            // Atirar continuamente enquanto mantém pressionado
            const now = Date.now();
            if (now - lastShot > shootCooldown) {
                player.shoot();
                lastShot = now;
            }
        }

        function handleShootEnd(e) {
            // Nada necessário aqui para este exemplo
        }

        // Função de game over
        function gameOver() {
            gameRunning = false;
            startScreen.style.display = 'flex';
            startButton.textContent = 'Jogar Novamente';
        }

        // Função para iniciar o jogo
        function startGame() {
            score = 0;
            scoreDisplay.textContent = `Pontos: ${score}`;
            player = new Player();
            bullets = [];
            enemies = [];
            healthFill.style.width = '100%';
            gameRunning = true;
            startScreen.style.display = 'none';
            lastEnemySpawn = Date.now();
        }

        // Event listener para o botão de início
        startButton.addEventListener('click', startGame);

        // Loop do jogo
        function gameLoop() {
            if (!gameRunning) {
                requestAnimationFrame(gameLoop);
                return;
            }

            // Limpar canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Atualizar jogador
            player.update();
            player.draw();
            
            // Atualizar e desenhar balas
            for (let i = bullets.length - 1; i >= 0; i--) {
                const bullet = bullets[i];
                if (bullet.update()) {
                    bullets.splice(i, 1);
                } else {
                    bullet.draw();
                }
            }
            
            // Spawn de inimigos
            const now = Date.now();
            if (now - lastEnemySpawn > 1000) { // 1 segundo
                enemies.push(new Enemy());
                lastEnemySpawn = now;
                
                // Aumentar dificuldade com o tempo
                if (score > 0 && score % 10 === 0) {
                    enemies.push(new Enemy());
                }
            }
            
            // Atualizar e desenhar inimigos
            for (let i = enemies.length - 1; i >= 0; i--) {
                const enemy = enemies[i];
                const shouldRemove = enemy.update();
                
                if (shouldRemove) {
                    enemies.splice(i, 1);
                } else {
                    enemy.draw();
                    
                    // Verificar colisão com balas
                    for (let j = bullets.length - 1; j >= 0; j--) {
                        const bullet = bullets[j];
                        const dx = enemy.x - bullet.x;
                        const dy = enemy.y - bullet.y;
                        const distance = Math.sqrt(dx * dx + dy * dy);
                        
                        if (distance < enemy.radius + bullet.radius) {
                            // Colisão detectada
                            enemy.health -= bullet.damage;
                            bullets.splice(j, 1);
                            
                            if (enemy.health <= 0) {
                                enemies.splice(i, 1);
                                score += Math.floor(30 - enemy.radius); // Inimigos menores valem mais
                                scoreDisplay.textContent = `Pontos: ${score}`;
                            }
                            break;
                        }
                    }
                }
            }
            
            requestAnimationFrame(gameLoop);
        }

        // Iniciar loop do jogo
        requestAnimationFrame(gameLoop);
    </script>
</body>
</html><!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Jogo de Tiro Mobile</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: none;
        }
        
        body {
            overflow: hidden;
            background-color: #111;
            position: fixed;
            width: 100%;
            height: 100%;
            font-family: Arial, sans-serif;
        }
        
        #gameCanvas {
            display: block;
            width: 100%;
            height: 100%;
            background-color: #000;
        }
        
        #uiContainer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }
        
        #scoreDisplay {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-size: 24px;
            text-shadow: 1px 1px 2px black;
        }
        
        #healthBar {
            position: absolute;
            top: 10px;
            right: 10px;
            width: 100px;
            height: 20px;
            background-color: rgba(255, 0, 0, 0.3);
            border: 2px solid white;
            border-radius: 5px;
            overflow: hidden;
        }
        
        #healthFill {
            height: 100%;
            width: 100%;
            background-color: red;
            transition: width 0.3s;
        }
        
        #joystick {
            position: absolute;
            bottom: 30px;
            left: 30px;
            width: 80px;
            height: 80px;
            background-color: rgba(255, 255, 255, 0.2);
            border-radius: 50%;
            pointer-events: auto;
        }
        
        #joystickKnob {
            position: absolute;
            width: 40px;
            height: 40px;
            background-color: rgba(255, 255, 255, 0.5);
            border-radius: 50%;
            top: 20px;
            left: 20px;
        }
        
        #shootButton {
            position: absolute;
            bottom: 30px;
            right: 30px;
            width: 80px;
            height: 80px;
            background-color: rgba(255, 0, 0, 0.5);
            border-radius: 50%;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 24px;
            pointer-events: auto;
            user-select: none;
        }
        
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 10;
        }
        
        #startButton {
            margin-top: 20px;
            padding: 15px 30px;
            background-color: #4CAF50;
            border: none;
            border-radius: 5px;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    
    <div id="uiContainer">
        <div id="scoreDisplay">Pontos: 0</div>
        <div id="healthBar"><div id="healthFill"></div></div>
        <div id="joystick"><div id="joystickKnob"></div></div>
        <div id="shootButton">ATIRAR</div>
    </div>
    
    <div id="startScreen">
        <h1>Jogo de Tiro Mobile</h1>
        <p>Use o joystick para mover e o botão para atirar</p>
        <button id="startButton">Começar</button>
    </div>

    <script>
        // Configurações do jogo
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const healthFill = document.getElementById('healthFill');
        const joystick = document.getElementById('joystick');
        const joystickKnob = document.getElementById('joystickKnob');
        const shootButton = document.getElementById('shootButton');
        const startScreen = document.getElementById('startScreen');
        const startButton = document.getElementById('startButton');

        // Ajustar tamanho do canvas
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // Variáveis do jogo
        let score = 0;
        let gameRunning = false;
        let player;
        let bullets = [];
        let enemies = [];
        let lastEnemySpawn = 0;
        let joystickActive = false;
        let joystickAngle = 0;
        let joystickDistance = 0;
        let joystickMaxDistance = 30;
        let lastShot = 0;
        let shootCooldown = 300; // ms

        // Classe do Jogador
        class Player {
            constructor() {
                this.x = canvas.width / 2;
                this.y = canvas.height / 2;
                this.radius = 20;
                this.color = '#3498db';
                this.speed = 5;
                this.health = 100;
                this.maxHealth = 100;
                this.shootDelay = 300; // ms
                this.lastShot = 0;
            }

            draw() {
                // Desenhar jogador
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
                
                // Desenhar "arma" ou direção
                ctx.beginPath();
                ctx.moveTo(this.x, this.y);
                ctx.lineTo(
                    this.x + Math.cos(this.angle) * this.radius * 1.5,
                    this.y + Math.sin(this.angle) * this.radius * 1.5
                );
                ctx.strokeStyle = 'white';
                ctx.lineWidth = 3;
                ctx.stroke();
                ctx.closePath();
            }

            update() {
                // Atualizar ângulo para mirar na direção do toque de tiro
                this.angle = Math.atan2(
                    shootTouch.y - this.y,
                    shootTouch.x - this.x
                );
                
                // Movimento com joystick
                if (joystickActive) {
                    this.x += Math.cos(joystickAngle) * this.speed * (joystickDistance / joystickMaxDistance);
                    this.y += Math.sin(joystickAngle) * this.speed * (joystickDistance / joystickMaxDistance);
                }
                
                // Manter jogador dentro do canvas
                this.x = Math.max(this.radius, Math.min(canvas.width - this.radius, this.x));
                this.y = Math.max(this.radius, Math.min(canvas.height - this.radius, this.y));
            }
            
            takeDamage(amount) {
                this.health -= amount;
                healthFill.style.width = `${(this.health / this.maxHealth) * 100}%`;
                
                if (this.health <= 0) {
                    gameOver();
                }
            }
            
            shoot() {
                const now = Date.now();
                if (now - this.lastShot < this.shootDelay) return;
                
                this.lastShot = now;
                
                bullets.push(new Bullet(
                    this.x,
                    this.y,
                    this.angle
                ));
            }
        }

        // Classe da Bala
        class Bullet {
            constructor(x, y, angle) {
                this.x = x;
                this.y = y;
                this.radius = 5;
                this.color = '#f1c40f';
                this.speed = 10;
                this.angle = angle;
                this.damage = 10;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }

            update() {
                this.x += Math.cos(this.angle) * this.speed;
                this.y += Math.sin(this.angle) * this.speed;
                
                // Verificar se a bala saiu da tela
                return this.x < -this.radius || this.x > canvas.width + this.radius || 
                       this.y < -this.radius || this.y > canvas.height + this.radius;
            }
        }

        // Classe do Inimigo
        class Enemy {
            constructor() {
                // Posição aleatória nas bordas
                const side = Math.floor(Math.random() * 4);
                
                if (side === 0) { // Topo
                    this.x = Math.random() * canvas.width;
                    this.y = -20;
                } else if (side === 1) { // Direita
                    this.x = canvas.width + 20;
                    this.y = Math.random() * canvas.height;
                } else if (side === 2) { // Baixo
                    this.x = Math.random() * canvas.width;
                    this.y = canvas.height + 20;
                } else { // Esquerda
                    this.x = -20;
                    this.y = Math.random() * canvas.height;
                }
                
                this.radius = 15 + Math.random() * 15; // Tamanho variado
                this.color = `hsl(${Math.random() * 60}, 100%, 50%)`; // Tons de vermelho/amarelo
                this.speed = 1 + Math.random() * 2;
                this.health = this.radius * 1.5;
                this.damage = 5;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }

            update() {
                // Mover em direção ao jogador
                const dx = player.x - this.x;
                const dy = player.y - this.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                this.x += dx / distance * this.speed;
                this.y += dy / distance * this.speed;
                
                // Verificar colisão com o jogador
                if (distance < this.radius + player.radius) {
                    player.takeDamage(this.damage);
                    return true; // Remover inimigo após causar dano
                }
                
                return false;
            }
        }

        // Variáveis de controle touch
        let joystickTouch = { id: null, x: 0, y: 0 };
        let shootTouch = { x: canvas.width - 100, y: canvas.height - 100 };

        // Configurar controles touch
        joystick.addEventListener('touchstart', handleJoystickStart);
        joystick.addEventListener('touchmove', handleJoystickMove);
        joystick.addEventListener('touchend', handleJoystickEnd);
        
        shootButton.addEventListener('touchstart', handleShootStart);
        shootButton.addEventListener('touchmove', handleShootMove);
        shootButton.addEventListener('touchend', handleShootEnd);
        
        // Também funciona para mouse (para teste em desktop)
        joystick.addEventListener('mousedown', handleJoystickStart);
        window.addEventListener('mousemove', handleJoystickMove);
        window.addEventListener('mouseup', handleJoystickEnd);
        
        shootButton.addEventListener('mousedown', handleShootStart);
        window.addEventListener('mousemove', handleShootMove);
        window.addEventListener('mouseup', handleShootEnd);

        function handleJoystickStart(e) {
            e.preventDefault();
            const touch = e.type === 'touchstart' ? e.touches[0] : e;
            const rect = joystick.getBoundingClientRect();
            
            joystickTouch.id = touch.identifier;
            joystickTouch.x = touch.clientX - rect.left - rect.width / 2;
            joystickTouch.y = touch.clientY - rect.top - rect.height / 2;
            
            joystickActive = true;
            updateJoystick();
        }

        function handleJoystickMove(e) {
            if (!joystickActive) return;
            e.preventDefault();
            
            let touch;
            if (e.type === 'touchmove') {
                for (let i = 0; i < e.touches.length; i++) {
                    if (e.touches[i].identifier === joystickTouch.id) {
                        touch = e.touches[i];
                        break;
                    }
                }
                if (!touch) return;
            } else {
                touch = e;
            }
            
            const rect = joystick.getBoundingClientRect();
            joystickTouch.x = touch.clientX - rect.left - rect.width / 2;
            joystickTouch.y = touch.clientY - rect.top - rect.height / 2;
            
            updateJoystick();
        }

        function handleJoystickEnd(e) {
            if (e.type === 'touchend') {
                for (let i = 0; i < e.changedTouches.length; i++) {
                    if (e.changedTouches[i].identifier === joystickTouch.id) {
                        joystickActive = false;
                        joystickKnob.style.transform = 'translate(20px, 20px)';
                        break;
                    }
                }
            } else {
                joystickActive = false;
                joystickKnob.style.transform = 'translate(20px, 20px)';
            }
        }

        function updateJoystick() {
            const distance = Math.sqrt(joystickTouch.x * joystickTouch.x + joystickTouch.y * joystickTouch.y);
            const angle = Math.atan2(joystickTouch.y, joystickTouch.x);
            
            joystickDistance = Math.min(distance, joystickMaxDistance);
            joystickAngle = angle;
            
            const limitedX = Math.cos(angle) * joystickDistance;
            const limitedY = Math.sin(angle) * joystickDistance;
            
            joystickKnob.style.transform = `translate(${20 + limitedX}px, ${20 + limitedY}px)`;
        }

        function handleShootStart(e) {
            e.preventDefault();
            const touch = e.type === 'touchstart' ? (e.touches[0] || e.changedTouches[0]) : e;
            const rect = shootButton.getBoundingClientRect();
            
            shootTouch.x = touch.clientX;
            shootTouch.y = touch.clientY;
            
            // Atirar imediatamente ao tocar
            player.shoot();
        }

        function handleShootMove(e) {
            e.preventDefault();
            
            let touch;
            if (e.type === 'touchmove') {
                // Encontrar o toque correspondente ao botão de tiro
                for (let i = 0; i < e.touches.length; i++) {
                    const rect = shootButton.getBoundingClientRect();
                    const x = e.touches[i].clientX - rect.left;
                    const y = e.touches[i].clientY - rect.top;
                    
                    if (x >= 0 && x <= rect.width && y >= 0 && y <= rect.height) {
                        touch = e.touches[i];
                        break;
                    }
                }
                if (!touch) return;
            } else {
                touch = e;
            }
            
            shootTouch.x = touch.clientX;
            shootTouch.y = touch.clientY;
            
            // Atirar continuamente enquanto mantém pressionado
            const now = Date.now();
            if (now - lastShot > shootCooldown) {
                player.shoot();
                lastShot = now;
            }
        }

        function handleShootEnd(e) {
            // Nada necessário aqui para este exemplo
        }

        // Função de game over
        function gameOver() {
            gameRunning = false;
            startScreen.style.display = 'flex';
            startButton.textContent = 'Jogar Novamente';
        }

        // Função para iniciar o jogo
        function startGame() {
            score = 0;
            scoreDisplay.textContent = `Pontos: ${score}`;
            player = new Player();
            bullets = [];
            enemies = [];
            healthFill.style.width = '100%';
            gameRunning = true;
            startScreen.style.display = 'none';
            lastEnemySpawn = Date.now();
        }

        // Event listener para o botão de início
        startButton.addEventListener('click', startGame);

        // Loop do jogo
        function gameLoop() {
            if (!gameRunning) {
                requestAnimationFrame(gameLoop);
                return;
            }

            // Limpar canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Atualizar jogador
            player.update();
            player.draw();
            
            // Atualizar e desenhar balas
            for (let i = bullets.length - 1; i >= 0; i--) {
                const bullet = bullets[i];
                if (bullet.update()) {
                    bullets.splice(i, 1);
                } else {
                    bullet.draw();
                }
            }
            
            // Spawn de inimigos
            const now = Date.now();
            if (now - lastEnemySpawn > 1000) { // 1 segundo
                enemies.push(new Enemy());
                lastEnemySpawn = now;
                
                // Aumentar dificuldade com o tempo
                if (score > 0 && score % 10 === 0) {
                    enemies.push(new Enemy());
                }
            }
            
            // Atualizar e desenhar inimigos
            for (let i = enemies.length - 1; i >= 0; i--) {
                const enemy = enemies[i];
                const shouldRemove = enemy.update();
                
                if (shouldRemove) {
                    enemies.splice(i, 1);
                } else {
                    enemy.draw();
                    
                    // Verificar colisão com balas
                    for (let j = bullets.length - 1; j >= 0; j--) {
                        const bullet = bullets[j];
                        const dx = enemy.x - bullet.x;
                        const dy = enemy.y - bullet.y;
                        const distance = Math.sqrt(dx * dx + dy * dy);
                        
                        if (distance < enemy.radius + bullet.radius) {
                            // Colisão detectada
                            enemy.health -= bullet.damage;
                            bullets.splice(j, 1);
                            
                            if (enemy.health <= 0) {
                                enemies.splice(i, 1);
                                score += Math.floor(30 - enemy.radius); // Inimigos menores valem mais
                                scoreDisplay.textContent = `Pontos: ${score}`;
                            }
                            break;
                        }
                    }
                }
            }
            
            requestAnimationFrame(gameLoop);
        }

        // Iniciar loop do jogo
        requestAnimationFrame(gameLoop);
    </script>
</body>
</html>
