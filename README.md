<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Esfera Runner</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">

<style>
    body {
        margin: 0;
        overflow: hidden;
        background: linear-gradient(#0f2027, #203a43, #2c5364);
        font-family: Arial, sans-serif;
    }
    canvas {
        display: block;
    }
</style>
</head>
<body>

<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// Cenário
const groundHeight = 120;

// Jogador
const player = {
    x: 140,
    y: canvas.height - groundHeight - 22,
    radius: 22,
    vy: 0,
    gravity: 1,
    jumpForce: -18,
    onGround: true
};

// Jogo
let obstacles = [];
let score = 0;
let speed = 6;
let gameOver = false;

// Pulo
function jump() {
    if (player.onGround) {
        player.vy = player.jumpForce;
        player.onGround = false;
    }
}

canvas.addEventListener("touchstart", jump);
canvas.addEventListener("mousedown", jump);
document.addEventListener("keydown", e => {
    if (e.code === "Space") jump();
});

// 👉 CRIA OBSTÁCULOS CONFORME A PONTUAÇÃO
function spawnObstacle() {
    let quantidade = 1;

    if (score >= 10000) {
        quantidade = 3;
    } else if (score >= 5000) {
        quantidade = 2;
    }

    for (let i = 0; i < quantidade; i++) {
        obstacles.push({
            x: canvas.width + i * 55,
            y: canvas.height - groundHeight - 45,
            size: 45
        });
    }
}

setInterval(spawnObstacle, 1400);

// Atualizações
function updatePlayer() {
    player.vy += player.gravity;
    player.y += player.vy;

    if (player.y >= canvas.height - groundHeight - player.radius) {
        player.y = canvas.height - groundHeight - player.radius;
        player.vy = 0;
        player.onGround = true;
    }
}

function updateObstacles() {
    obstacles.forEach(o => o.x -= speed);
    obstacles = obstacles.filter(o => o.x + o.size > 0);
}

// Colisão
function checkCollision() {
    obstacles.forEach(o => {
        const dx = Math.abs(player.x - (o.x + o.size / 2));
        const dy = Math.abs(player.y - (o.y + o.size / 2));
        if (dx < player.radius + o.size / 2 &&
            dy < player.radius + o.size / 2) {
            gameOver = true;
        }
    });
}

// Desenhos
function drawBackground() {
    const grad = ctx.createLinearGradient(0, 0, 0, canvas.height);
    grad.addColorStop(0, "#74ebd5");
    grad.addColorStop(1, "#ACB6E5");
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
}

function drawPanel() {
    ctx.fillStyle = "rgba(0,0,0,0.45)";
    ctx.fillRect(0, 0, canvas.width, 70);

    ctx.fillStyle = "#fff";
    ctx.font = "22px Arial";
    ctx.fillText("Pontos: " + score, 20, 45);
}

function drawGround() {
    const grad = ctx.createLinearGradient(0, canvas.height - groundHeight, 0, canvas.height);
    grad.addColorStop(0, "#2c3e50");
    grad.addColorStop(1, "#000");
    ctx.fillStyle = grad;
    ctx.fillRect(0, canvas.height - groundHeight, canvas.width, groundHeight);
}

function drawPlayer() {
    const grad = ctx.createRadialGradient(
        player.x - 6, player.y - 6, 5,
        player.x, player.y, player.radius
    );
    grad.addColorStop(0, "#ffffff");
    grad.addColorStop(1, "#ff9800");

    ctx.beginPath();
    ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
    ctx.fillStyle = grad;
    ctx.fill();
}

function drawObstacles() {
    obstacles.forEach(o => {
        ctx.fillStyle = "#111";
        ctx.shadowColor = "rgba(0,0,0,0.6)";
        ctx.shadowBlur = 10;
        ctx.fillRect(o.x, o.y, o.size, o.size);
        ctx.shadowBlur = 0;
    });
}

// Loop
function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(0,0,0,0.65)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#ff4444";
        ctx.font = "48px Arial";
        ctx.fillText("GAME OVER", canvas.width / 2 - 140, canvas.height / 2);
        ctx.font = "22px Arial";
        ctx.fillStyle = "#fff";
        ctx.fillText("Toque para reiniciar", canvas.width / 2 - 120, canvas.height / 2 + 50);
        return;
    }

    drawBackground();
    drawPanel();
    drawGround();

    updatePlayer();
    updateObstacles();
    checkCollision();

    drawPlayer();
    drawObstacles();

    score++;
    speed = 6 + Math.floor(score / 800);

    requestAnimationFrame(gameLoop);
}

// Reiniciar
canvas.addEventListener("click", () => {
    if (gameOver) location.reload();
});

// Resize
window.addEventListener("resize", () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
});

// Start
gameLoop();
</script>

</body>
</html>