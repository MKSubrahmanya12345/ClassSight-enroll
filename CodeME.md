# Dino Story Mode Game

This project implements a single-file, client-side 'Dino Story Mode' game, featuring bikes, dragons, and power-ups appearing sequentially over a 4-minute easy gameplay experience. The core game logic will be handled by JavaScript manipulating an HTML Canvas.

## Project Structure

```
.
├── index.html
├── assets/
│   ├── images/      # Dino, obstacles, bikes, dragons, power-up sprites
│   └── audio/       # Game sounds, background music
├── js/
│   ├── game.js      # Main game loop, canvas, story progression
│   ├── player.js    # Dino character logic
│   └── obstacles.js # Obstacle and enemy (bike, dragon) logic
└── css/
    └── style.css    # Basic styling
```

## `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dino Story Mode</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div id="game-container">
        <canvas id="gameCanvas" width="900" height="300"></canvas>
        <div id="score">Score: 0</div>
        <div id="message">Press Space to Start!</div>
    </div>

    <script src="js/player.js"></script>
    <script src="js/obstacles.js"></script>
    <script src="js/game.js"></script>
</body>
</html>
```

## `css/style.css`

```css
body {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background-color: #f0f0f0;
    margin: 0;
    font-family: Arial, sans-serif;
}

#game-container {
    border: 2px solid #333;
    background-color: #fff;
    position: relative;
    overflow: hidden; /* Hide anything outside the canvas */
}

#gameCanvas {
    display: block;
    background-color: #e0e0e0;
}

#score, #message {
    position: absolute;
    color: #333;
    font-size: 20px;
    padding: 10px;
}

#score {
    top: 0;
    left: 0;
}

#message {
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: rgba(255, 255, 255, 0.8);
    padding: 15px 30px;
    border-radius: 5px;
    display: none; /* Hidden by default, shown for start/game over */
}
```

## `js/player.js` (Example Snippet)

```javascript
class Player {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.x = 50;
        this.y = canvas.height - 70;
        this.width = 40;
        this.height = 60;
        this.velocityY = 0;
        this.gravity = 0.6;
        this.isJumping = false;
        // Add properties for bikes/dragons/powerups
        this.mode = 'dino'; // 'dino', 'bike', 'dragon'
        this.powerUpActive = false;
    }

    jump() {
        if (!this.isJumping) {
            this.velocityY = -12; // Adjust jump strength
            this.isJumping = true;
        }
    }

    update() {
        this.velocityY += this.gravity;
        this.y += this.velocityY;

        if (this.y > this.canvas.height - 70) {
            this.y = this.canvas.height - 70;
            this.velocityY = 0;
            this.isJumping = false;
        }
    }

    draw() {
        this.ctx.fillStyle = 'green';
        this.ctx.fillRect(this.x, this.y, this.width, this.height);
        // Draw different sprites based on this.mode (dino, bike, dragon)
    }

    // Methods to switch modes or activate power-ups
    activateBike() { this.mode = 'bike'; /* ... */ }
    activateDragon() { this.mode = 'dragon'; /* ... */ }
    activatePowerUp() { this.powerUpActive = true; /* ... */ }
}
```

## `js/obstacles.js` (Example Snippet)

```javascript
class Obstacle {
    constructor(canvas, type) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.x = canvas.width;
        this.y = canvas.height - 50; // Example position
        this.width = 30;
        this.height = 50;
        this.speed = 5;
        this.type = type; // 'cactus', 'pterodactyl', 'bike', 'dragon', 'powerup'
        // Load different images based on type
    }

    update() {
        this.x -= this.speed;
    }

    draw() {
        this.ctx.fillStyle = 'red'; // Placeholder color
        this.ctx.fillRect(this.x, this.y, this.width, this.height);
        // Draw specific image for cactus, bike, dragon, etc.
    }

    // Collision detection method
    collidesWith(player) {
        // ... collision logic ...
    }
}

// Logic to spawn obstacles, bikes, dragons, power-ups based on game progress/time
```

## `js/game.js` (Example Snippet)

```javascript
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreDisplay = document.getElementById('score');
const messageDisplay = document.getElementById('message');

let player;
let obstacles = [];
let gameFrame = 0;
let score = 0;
let gameOver = false;
let gameStarted = false;
let gameTimer = 0; // Max 4 minutes (240 seconds)

function initGame() {
    player = new Player(canvas);
    obstacles = [];
    score = 0;
    gameFrame = 0;
    gameOver = false;
    gameStarted = false;
    gameTimer = 0;
    scoreDisplay.textContent = `Score: ${score}`;
    messageDisplay.textContent = 'Press Space to Start!';
    messageDisplay.style.display = 'block';
}

function animate() {
    if (gameOver || !gameStarted) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    gameFrame++;
    gameTimer++;

    // Update and draw player
    player.update();
    player.draw();

    // Spawn obstacles/enemies/power-ups based on gameFrame or gameTimer
    // Logic for story progression: After X seconds, introduce bikes. After Y seconds, introduce dragons. After Z seconds, introduce power-ups.
    if (gameTimer === 60 * 30) { // Example: After 30 seconds, introduce bikes
        obstacles.push(new Obstacle(canvas, 'bike'));
    }
    // ... similar logic for dragons and power-ups ...

    // Update and draw obstacles
    obstacles.forEach((obstacle, index) => {
        obstacle.update();
        obstacle.draw();

        if (obstacle.x + obstacle.width < 0) {
            obstacles.splice(index, 1); // Remove off-screen obstacles
            score++;
            scoreDisplay.textContent = `Score: ${score}`;
        }

        if (obstacle.collidesWith(player)) {
            // Handle collision (e.g., game over, or power-up effect)
            if (obstacle.type === 'powerup') {
                player.activatePowerUp();
                obstacles.splice(index, 1); // Remove power-up after collection
            } else {
                gameOver = true;
                messageDisplay.textContent = 'Game Over! Press Space to Restart.';
                messageDisplay.style.display = 'block';
            }
        }
    });

    // Game end condition (4 minutes)
    if (gameTimer >= 60 * 240) { // 240 seconds = 4 minutes
        gameOver = true;
        messageDisplay.textContent = 'Congratulations! You finished the story! Press Space to Restart.';
        messageDisplay.style.display = 'block';
    }

    requestAnimationFrame(animate);
}

document.addEventListener('keydown', (e) => {
    if (e.code === 'Space') {
        if (!gameStarted && !gameOver) {
            gameStarted = true;
            messageDisplay.style.display = 'none';
            animate();
        } else if (gameOver) {
            initGame();
            animate();
        }
        player.jump();
    }
});

initGame();
```
