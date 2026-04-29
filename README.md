<!DOCTYPE html>
<html>
<head>
    <title>Aim & Damage Shooter</title>
    <style>
        body { margin: 0; background: #222; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        canvas { background: #000; border: 2px solid #fff; cursor: crosshair; }
    </style>
</head>
<body>
<canvas id="gameCanvas" width="800" height="600"></canvas>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    let player = { x: 400, y: 550, w: 30, h: 30, speed: 5, health: 3 };
    let bullets = [];
    let enemies = [];
    let keys = {};
    let mouse = { x: 0, y: 0 };
    let isMouseDown = false;

    document.addEventListener("keydown", (e) => keys[e.code] = true);
    document.addEventListener("keyup", (e) => keys[e.code] = false);
    document.addEventListener("mousemove", (e) => {
        const rect = canvas.getBoundingClientRect();
        mouse.x = e.clientX - rect.left;
        mouse.y = e.clientY - rect.top;
    });
    document.addEventListener("mousedown", () => isMouseDown = true);
    document.addEventListener("mouseup", () => isMouseDown = false);

    function spawnEnemy() {
        enemies.push({ x: Math.random() * 780, y: -30, w: 30, h: 30, speed: 2 });
    }
    setInterval(spawnEnemy, 1000);

    function update() {
        if (player.health <= 0) { alert("Game Over!"); player.health = 3; enemies = []; }

        // Movement
        if (keys["KeyA"] && player.x > 0) player.x -= player.speed;
        if (keys["KeyD"] && player.x < canvas.width - player.w) player.x += player.speed;
        if (keys["KeyW"] && player.y > 0) player.y -= player.speed;
        if (keys["KeyS"] && player.y < canvas.height - player.h) player.y += player.speed;
        
        // Aim and Shoot
        if (isMouseDown) {
            let angle = Math.atan2(mouse.y - (player.y + 15), mouse.x - (player.x + 15));
            bullets.push({ 
                x: player.x + 12, y: player.y, 
                vx: Math.cos(angle) * 10, vy: Math.sin(angle) * 10 
            });
            isMouseDown = false; 
        }

        bullets.forEach((b, i) => {
            b.x += b.vx; b.y += b.vy;
            if (b.y < 0 || b.y > 600 || b.x < 0 || b.x > 800) bullets.splice(i, 1);
        });

        enemies.forEach((e, i) => {
            e.y += e.speed;
            // Damage Player
            if (e.x < player.x + player.w && e.x + e.w > player.x && e.y < player.y + player.h && e.y + e.h > player.y) {
                player.health--;
                enemies.splice(i, 1);
            }
            // Collision with Bullets
            bullets.forEach((b, bi) => {
                if (b.x > e.x && b.x < e.x + e.w && b.y > e.y && b.y < e.y + e.h) {
                    enemies.splice(i, 1);
                    bullets.splice(bi, 1);
                }
            });
        });

        draw();
        requestAnimationFrame(update);
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // Draw Player
        ctx.fillStyle = "#00ff00";
        ctx.fillRect(player.x, player.y, player.w, player.h);
        
        // Draw UI
        ctx.fillStyle = "white";
        ctx.fillText("Health: " + player.health, 10, 20);

        bullets.forEach(b => { ctx.fillStyle = "yellow"; ctx.fillRect(b.x, b.y, 5, 5); });
        enemies.forEach(e => { ctx.fillStyle = "red"; ctx.fillRect(e.x, e.y, e.w, e.h); });
    }

    update();
</script>
</body>
</html>