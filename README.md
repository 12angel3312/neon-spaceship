# neon-spaceship <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Spaceship v6.1 - Battle Pass & Reactive Skins</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
        body { background-color: #05050d; color: #fff; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; overflow: hidden; }
        #gameCanvas { display: block; background: #020208; width: 100vw; height: 100vh; }
        
        /* UI Overlay */
        .ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
        .interactive { pointer-events: auto; }
        
        /* HUD */
        #hud { position: absolute; top: 15px; left: 20px; right: 20px; display: flex; justify-content: space-between; align-items: flex-start; font-weight: bold; text-shadow: 0 0 10px rgba(0,255,255,0.7); }
        .stat-box { background: rgba(10, 15, 30, 0.75); border: 1px solid #00f3ff; padding: 10px 18px; border-radius: 8px; box-shadow: 0 0 15px rgba(0, 243, 255, 0.2); }
        .bar-container { width: 180px; height: 12px; background: rgba(255,255,255,0.1); border-radius: 6px; overflow: hidden; margin-top: 5px; border: 1px solid rgba(0,243,255,0.4); }
        .bar-fill { height: 100%; transition: width 0.2s ease-out; }
        #hpBar { background: linear-gradient(90deg, #ff0055, #ff5500); }
        #shieldBar { background: linear-gradient(90deg, #00aaff, #00ffff); }
        #cooldownBar { background: linear-gradient(90deg, #aa00ff, #00f3ff); }
        
        /* Modals & Screens */
        .screen-modal { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(5, 5, 15, 0.88); backdrop-filter: blur(8px); display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 10; }
        .hidden { display: none !important; }
        
        h1 { font-size: 3rem; text-transform: uppercase; letter-spacing: 4px; color: #00f3ff; text-shadow: 0 0 20px #00f3ff, 0 0 40px #aa00ff; margin-bottom: 10px; }
        h2 { font-size: 1.8rem; color: #ff007f; text-shadow: 0 0 10px #ff007f; margin-bottom: 20px; }
        
        .btn { background: linear-gradient(135deg, #00f3ff, #aa00ff); color: #fff; border: none; padding: 12px 28px; font-size: 1.1rem; font-weight: bold; border-radius: 6px; cursor: pointer; text-transform: uppercase; letter-spacing: 1px; box-shadow: 0 0 15px rgba(0,243,255,0.4); transition: all 0.2s; margin: 8px; }
        .btn:hover { transform: translateY(-2px) scale(1.03); box-shadow: 0 0 25px rgba(0,243,255,0.8); }
        
        /* Battle Pass UI */
        .bp-container { width: 85%; max-width: 1000px; background: rgba(12, 16, 35, 0.9); border: 2px solid #aa00ff; border-radius: 12px; padding: 20px; box-shadow: 0 0 30px rgba(170,0,255,0.3); margin-top: 15px; }
        .bp-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
        .bp-track { display: flex; gap: 12px; overflow-x: auto; padding: 15px 5px; scrollbar-color: #00f3ff rgba(0,0,0,0.5); }
        .bp-tier { min-width: 90px; height: 110px; background: rgba(255,255,255,0.05); border: 1px solid rgba(0,243,255,0.3); border-radius: 8px; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; font-size: 0.85rem; }
        .bp-tier.unlocked { border-color: #00ff88; background: rgba(0,255,136,0.15); box-shadow: 0 0 10px rgba(0,255,136,0.3); }
        .bp-tier.featured { border-color: #ffaa00; background: rgba(255,170,0,0.2); }
        .bp-level-tag { position: absolute; top: 4px; left: 6px; font-size: 0.7rem; color: #aaa; }
        
        /* Ship Selection Cards */
        .ship-selector { display: flex; gap: 20px; margin: 20px 0; }
        .ship-card { background: rgba(20, 25, 50, 0.7); border: 2px solid #00f3ff; border-radius: 10px; padding: 20px; width: 220px; text-align: center; cursor: pointer; transition: 0.3s; }
        .ship-card.active { border-color: #ff007f; box-shadow: 0 0 25px rgba(255,0,127,0.6); transform: scale(1.05); }
        .ship-card.locked { opacity: 0.5; border-color: #555; }
    </style>
</head>
<body>

    <canvas id="gameCanvas"></canvas>

    <div class="ui-layer">
        <!-- HUD In-Game -->
        <div id="hud" class="hidden">
            <div class="stat-box">
                <div>SALUD</div>
                <div class="bar-container"><div id="hpBar" class="bar-fill"></div></div>
                <div style="margin-top: 6px;">ESCUDO</div>
                <div class="bar-container"><div id="shieldBar" class="bar-fill"></div></div>
            </div>
            <div class="stat-box" style="text-align: center;">
                <div id="levelDisplay" style="font-size: 1.4rem; color: #00f3ff;">NIVEL 1</div>
                <div id="scoreDisplay" style="font-size: 0.9rem; color: #aaa;">PUNTOS: 0</div>
            </div>
            <div class="stat-box" style="text-align: right;">
                <div id="gadgetLabel">GADGET [E]</div>
                <div class="bar-container"><div id="cooldownBar" class="bar-fill"></div></div>
                <div style="margin-top: 6px; font-size: 0.8rem; color: #888;">[ESC] PAUSA</div>
            </div>
        </div>

        <!-- Main Menu -->
        <div id="mainMenu" class="screen-modal interactive">
            <h1>Neon Spaceship 6.1</h1>
            <h2>Edición Pase de Batalla & Skins Reactivas</h2>

            <div class="ship-selector">
                <div id="cardSolari" class="ship-card active" onclick="selectShip('solari')">
                    <h3 style="color: #ffaa00;">SOLARI</h3>
                    <p style="font-size: 0.8rem; margin-top: 8px;">Tanque Pesado (320 HP)<br>Ataque de Área Solar<br>Cohetes Teledirigidos<br><span id="solariGadgetText" style="color:#00ff88;">Gadget [E]: Curación</span></p>
                </div>
                <div id="cardChronos" class="ship-card" onclick="selectShip('chronos')">
                    <h3 style="color: #00f3ff;">CHRONOS</h3>
                    <p style="font-size: 0.8rem; margin-top: 8px;">Control Temporal<br>Disparo Ciber-Taquión<br>Desbloqueo en Pase Niv. 50<br><span id="chronosGadgetText" style="color:#00ff88;">Gadget [E]: Invencibilidad</span></p>
                </div>
            </div>

            <button class="btn" onclick="startGame()">INICIAR MISIÓN</button>

            <!-- Pase de Batalla -->
            <div class="bp-container">
                <div class="bp-header">
                    <span style="font-weight: bold; color: #aa00ff;">PASE DE BATALLA (100 NIVELES)</span>
                    <span id="bpLevelText" style="color: #00f3ff;">Nivel del Pase: 1</span>
                </div>
                <div class="bp-track" id="bpTrack">
                    <!-- Generado dinámicamente -->
                </div>
            </div>
        </div>

        <!-- Pause Menu -->
        <div id="pauseMenu" class="screen-modal interactive hidden">
            <h1>JUEGO EN PAUSA</h1>
            <button class="btn" onclick="togglePause()">REANUDAR</button>
            <button class="btn" onclick="returnToMenu()">VOLVER AL MENÚ</button>
        </div>

        <!-- Game Over -->
        <div id="gameOverMenu" class="screen-modal interactive hidden">
            <h1 style="color: #ff0055;">MISIÓN FALLIDA</h1>
            <p id="finalScore" style="margin-bottom: 20px; font-size: 1.2rem;"></p>
            <button class="btn" onclick="startGame()">REINTENTAR</button>
            <button class="btn" onclick="returnToMenu()">MENÚ PRINCIPAL</button>
        </div>
    </div>

    <script>
        // Declaración Global y Configuración de Canvas
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // Estado del Juego
        let isGameRunning = false;
        let isPaused = false;
        let selectedShip = 'solari';
        let bpLevel = 1;
        let bpXP = 0;
        let score = 0;
        let currentLevel = 1;

        // Niveles de Nave y Progreso
        let shipLevels = { solari: 1, chronos: 1 };
        let shipXP = { solari: 0, chronos: 0 };

        // Entidades
        let player;
        let enemies = [];
        let bullets = [];
        let missiles = [];
        let particles = [];
        let shockwaves = [];
        let keys = {};

        // Manejo de Teclado y Ratón
        window.addEventListener('keydown', e => {
            keys[e.key.toLowerCase()] = true;
            keys[e.code] = true;
            if (e.key === 'Escape' && isGameRunning) togglePause();
        });
        window.addEventListener('keyup', e => {
            keys[e.key.toLowerCase()] = false;
            keys[e.code] = false;
        });
        window.addEventListener('mousemove', e => {
            if (player) {
                player.mouseX = e.clientX;
                player.mouseY = e.clientY;
            }
        });
        window.addEventListener('mousedown', () => { if (player) player.isFiring = true; });
        window.addEventListener('mouseup', () => { if (player) player.isFiring = false; });

        // Clase Onda Expansiva (Shockwave)
        class Shockwave {
            constructor(x, y, maxRadius, color) {
                this.x = x;
                this.y = y;
                this.radius = 5;
                this.maxRadius = maxRadius;
                this.color = color;
                this.alpha = 1;
            }
            update() {
                this.radius += 4;
                this.alpha = 1 - (this.radius / this.maxRadius);
            }
            draw() {
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.strokeStyle = this.color;
                ctx.globalAlpha = Math.max(0, this.alpha);
                ctx.lineWidth = 4;
                ctx.stroke();
                ctx.fillStyle = this.color;
                ctx.globalAlpha = Math.max(0, this.alpha * 0.2);
                ctx.fill();
                ctx.restore();
            }
        }

        // Nave Jugador
        class Player {
            constructor(type) {
                this.type = type;
                this.x = canvas.width / 2;
                this.y = canvas.height / 2;
                this.radius = 22;
                this.angle = 0;
                this.speed = 5.5;
                this.level = shipLevels[type] || 1;
                
                this.maxHp = (type === 'solari' ? 320 : 180) + (this.level - 1) * 20;
                this.hp = this.maxHp;
                this.maxShield = 100 + (this.level - 1) * 10;
                this.shield = this.maxShield;
                
                this.cooldown = 0;
                this.maxCooldown = 600; // 10s
                this.isInvulnerable = false;
                this.invulnerableTimer = 0;
                
                this.mouseX = canvas.width / 2;
                this.mouseY = canvas.height / 2;
                this.isFiring = false;
                this.fireTimer = 0;
                this.reactiveBoost = 0;
            }

            update() {
                // Movimiento WASD / Flechas
                if (keys['w'] || keys['arrowup'] || keys['KeyW']) this.y -= this.speed;
                if (keys['s'] || keys['arrowdown'] || keys['KeyS']) this.y += this.speed;
                if (keys['a'] || keys['arrowleft'] || keys['KeyA']) this.x -= this.speed;
                if (keys['d'] || keys['arrowright'] || keys['KeyD']) this.x += this.speed;

                this.x = Math.max(this.radius, Math.min(canvas.width - this.radius, this.x));
                this.y = Math.max(this.radius, Math.min(canvas.height - this.radius, this.y));

                // Rotación hacia el cursor
                this.angle = Math.atan2(this.mouseY - this.y, this.mouseX - this.x);

                // Disparo Básico
                this.fireTimer++;
                let fireRate = (this.type === 'chronos' && this.isInvulnerable) ? 3 : 10;
                if (this.isFiring && this.fireTimer > fireRate) {
                    this.fireTimer = 0;
                    this.shootBasic();
                }

                // Activar Gadget [E]
                if (this.cooldown > 0) this.cooldown--;
                if ((keys['e'] || keys['KeyE']) && this.cooldown === 0) {
                    this.useGadget();
                }

                // Temporizador de invulnerabilidad (Gadget Chronos)
                if (this.invulnerableTimer > 0) {
                    this.invulnerableTimer--;
                    if (this.invulnerableTimer <= 0) {
                        this.isInvulnerable = false;
                    }
                }

                // Recarga progresiva de escudo
                if (this.shield < this.maxShield) this.shield += 0.05;
                if (this.reactiveBoost > 0) this.reactiveBoost -= 0.02;
            }

            shootBasic() {
                const dmgMultiplier = 1.2;
                const baseDmg = (this.type === 'solari' ? 35 : 22) * dmgMultiplier;
                
                if (this.type === 'solari') {
                    // Disparo Esférico Solar Doble con Explosión
                    for (let side of [-10, 10]) {
                        const perpAngle = this.angle + Math.PI / 2;
                        bullets.push({
                            x: this.x + Math.cos(this.angle) * 20 + Math.cos(perpAngle) * side,
                            y: this.y + Math.sin(this.angle) * 20 + Math.sin(perpAngle) * side,
                            vx: Math.cos(this.angle) * 11,
                            vy: Math.sin(this.angle) * 11,
                            radius: 10,
                            color: '#ffaa00',
                            damage: baseDmg,
                            isPlayer: true,
                            isSolar: true
                        });
                    }
                } else {
                    // Disparo Ciber-Taquión
                    bullets.push({
                        x: this.x + Math.cos(this.angle) * 25,
                        y: this.y + Math.sin(this.angle) * 25,
                        vx: Math.cos(this.angle) * (this.isInvulnerable ? 18 : 14),
                        vy: Math.sin(this.angle) * (this.isInvulnerable ? 18 : 14),
                        radius: 6,
                        color: '#00f3ff',
                        damage: baseDmg,
                        isPlayer: true,
                        isSolar: false
                    });
                }
            }

            useGadget() {
                if (this.level < 4) {
                    createFloatingText(this.x, this.y - 30, '🔒 Gadget requiere Nivel 4', '#ff0055');
                    return;
                }

                this.cooldown = this.maxCooldown;

                if (this.type === 'solari') {
                    // Gadget Solari: Restaura 35% de Vida y Escudo
                    const healHp = this.maxHp * 0.35;
                    const healShield = this.maxShield * 0.35;
                    this.hp = Math.min(this.maxHp, this.hp + healHp);
                    this.shield = Math.min(this.maxShield, this.shield + healShield);
                    shockwaves.push(new Shockwave(this.x, this.y, 120, '#00ff88'));
                    createFloatingText(this.x, this.y - 30, '+35% CURACIÓN', '#00ff88');
                } else {
                    // Gadget Chronos: Invencibilidad y Super Velocidad por 5s
                    this.isInvulnerable = true;
                    this.invulnerableTimer = 300; // 5 segundos (60 fps * 5)
                    shockwaves.push(new Shockwave(this.x, this.y, 150, '#00f3ff'));
                    createFloatingText(this.x, this.y - 30, '¡MODO TAQUIÓN ACTIVO!', '#00f3ff');
                }
            }

            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);

                // Escudo Neón / Gadget Invulnerable
                if (this.isInvulnerable) {
                    ctx.beginPath();
                    ctx.arc(0, 0, this.radius + 15, 0, Math.PI * 2);
                    ctx.strokeStyle = '#00ffff';
                    ctx.lineWidth = 5;
                    ctx.shadowBlur = 20;
                    ctx.shadowColor = '#00ffff';
                    ctx.stroke();
                } else if (this.shield > 5) {
                    ctx.beginPath();
                    ctx.arc(0, 0, this.radius + 10, 0, Math.PI * 2);
                    ctx.strokeStyle = `rgba(0, 243, 255, ${0.3 + Math.sin(Date.now() * 0.01) * 0.2})`;
                    ctx.lineWidth = 3;
                    ctx.stroke();
                }

                // Render Nave Base
                ctx.beginPath();
                ctx.moveTo(20, 0);
                ctx.lineTo(-15, -15);
                ctx.lineTo(-8, 0);
                ctx.lineTo(-15, 15);
                ctx.closePath();
                ctx.fillStyle = this.type === 'solari' ? '#ffaa00' : '#00f3ff';
                if (this.reactiveBoost > 0) ctx.fillStyle = '#ff007f';
                ctx.fill();
                ctx.shadowBlur = 15 + this.reactiveBoost * 20;
                ctx.shadowColor = ctx.fillStyle;
                ctx.strokeStyle = '#ffffff';
                ctx.stroke();

                ctx.restore();
            }
        }

        // Texto Flotante
        let floatingTexts = [];
        function createFloatingText(x, y, text, color) {
            floatingTexts.push({ x, y, text, color, alpha: 1, life: 60 });
        }

        function updateFloatingTexts() {
            floatingTexts.forEach((ft, i) => {
                ft.y -= 1;
                ft.life--;
                ft.alpha = ft.life / 60;
                ctx.save();
                ctx.globalAlpha = Math.max(0, ft.alpha);
                ctx.font = 'bold 14px Segoe UI';
                ctx.fillStyle = ft.color;
                ctx.textAlign = 'center';
                ctx.fillText(ft.text, ft.x, ft.y);
                ctx.restore();
                if (ft.life <= 0) floatingTexts.splice(i, 1);
            });
        }

        // Sistema de Enemigos
        function spawnEnemy() {
            if (enemies.length >= 12) return;
            const side = Math.floor(Math.random() * 4);
            let x, y;
            if (side === 0) { x = Math.random() * canvas.width; y = -30; }
            else if (side === 1) { x = canvas.width + 30; y = Math.random() * canvas.height; }
            else if (side === 2) { x = Math.random() * canvas.width; y = canvas.height + 30; }
            else { x = -30; y = Math.random() * canvas.height; }

            enemies.push({
                x: x,
                y: y,
                hp: 40 + currentLevel * 8,
                maxHp: 40 + currentLevel * 8,
                speed: 1.8 + Math.random() * 1.2,
                radius: 16
            });
        }

        // Detonación por Área
        function triggerAreaExplosion(x, y, radius, damage) {
            shockwaves.push(new Shockwave(x, y, radius, '#ff5500'));
            enemies.forEach((e, eIdx) => {
                const dist = Math.hypot(e.x - x, e.y - y);
                if (dist <= radius) {
                    e.hp -= damage;
                    createFloatingText(e.x, e.y, `-${Math.round(damage)}`, '#ffaa00');
                    if (e.hp <= 0) {
                        enemies.splice(eIdx, 1);
                        score += 100;
                        player.reactiveBoost = 1.0;
                        addXP(25);
                        addShipXP(player.type, 15);
                    }
                }
            });
        }

        // Bucle Principal
        function gameLoop() {
            if (!isGameRunning || isPaused) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Fondo Neón
            ctx.strokeStyle = 'rgba(0, 243, 255, 0.05)';
            ctx.lineWidth = 1;
            for (let x = 0; x < canvas.width; x += 50) {
                ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, canvas.height); ctx.stroke();
            }

            // Actualizar Jugador
            player.update();
            player.draw();

            // Generar Enemigos
            if (Math.random() < 0.025) spawnEnemy();

            // Actualizar Ondas Expansivas
            shockwaves.forEach((sw, idx) => {
                sw.update();
                sw.draw();
                if (sw.alpha <= 0) shockwaves.splice(idx, 1);
            });

            // Actualizar Enemigos
            enemies.forEach((e, index) => {
                const angle = Math.atan2(player.y - e.y, player.x - e.x);
                e.x += Math.cos(angle) * e.speed;
                e.y += Math.sin(angle) * e.speed;

                // Render
                ctx.beginPath();
                ctx.arc(e.x, e.y, e.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#ff0055';
                ctx.fill();

                // Colisión con jugador
                const dist = Math.hypot(player.x - e.x, player.y - e.y);
                if (dist < player.radius + e.radius) {
                    if (!player.isInvulnerable) {
                        if (player.shield > 0) player.shield -= 20;
                        else player.hp -= 20;
                    }
                    enemies.splice(index, 1);
                }
            });

            // Actualizar Balas
            bullets.forEach((b, bIdx) => {
                b.x += b.vx;
                b.y += b.vy;

                ctx.beginPath();
                ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2);
                ctx.fillStyle = b.color;
                ctx.fill();

                // Colisión con límites de pantalla
                if (b.x < 0 || b.x > canvas.width || b.y < 0 || b.y > canvas.height) {
                    bullets.splice(bIdx, 1);
                    return;
                }

                // Impacto con Enemigos
                enemies.forEach((e, eIdx) => {
                    if (Math.hypot(b.x - e.x, b.y - e.y) < e.radius + b.radius) {
                        e.hp -= b.damage;
                        createFloatingText(e.x, e.y, `-${Math.round(b.damage)}`, '#ffffff');

                        if (b.isSolar) {
                            // Detonación en área por disparo solar
                            triggerAreaExplosion(b.x, b.y, 80, b.damage * 0.7);
                        }

                        bullets.splice(bIdx, 1);

                        if (e.hp <= 0) {
                            enemies.splice(eIdx, 1);
                            score += 100;
                            player.reactiveBoost = 1.0;
                            addXP(25);
                            addShipXP(player.type, 15);
                        }
                    }
                });
            });

            // Textos Flotantes
            updateFloatingTexts();

            // Actualizar HUD
            document.getElementById('hpBar').style.width = Math.max(0, (player.hp / player.maxHp * 100)) + '%';
            document.getElementById('shieldBar').style.width = Math.max(0, (player.shield / player.maxShield * 100)) + '%';
            document.getElementById('cooldownBar').style.width = ((1 - player.cooldown / player.maxCooldown) * 100) + '%';
            document.getElementById('scoreDisplay').innerText = `PUNTOS: ${score}`;
            document.getElementById('levelDisplay').innerText = `NIVEL ${currentLevel}`;

            const gadgetLabel = document.getElementById('gadgetLabel');
            if (player.level < 4) {
                gadgetLabel.innerText = '🔒 GADGET (NIV 4)';
                gadgetLabel.style.color = '#ff0055';
            } else {
                gadgetLabel.innerText = 'GADGET [E]';
                gadgetLabel.style.color = '#00ff88';
            }

            if (player.hp <= 0) {
                isGameRunning = false;
                document.getElementById('gameOverMenu').classList.remove('hidden');
                document.getElementById('finalScore').innerText = `Puntuación final: ${score}`;
            } else {
                requestAnimationFrame(gameLoop);
            }
        }

        // Sistema de Pase de Batalla
        function renderBattlePass() {
            const track = document.getElementById('bpTrack');
            track.innerHTML = '';
            for (let i = 1; i <= 100; i++) {
                const tier = document.createElement('div');
                tier.className = `bp-tier ${i <= bpLevel ? 'unlocked' : ''} ${i % 10 === 0 ? 'featured' : ''}`;
                tier.innerHTML = `<span class="bp-level-tag">NIV ${i}</span><span>${i === 50 ? 'CHRONOS' : (i === 100 ? 'SKIN LEG.' : 'RECOMPENSA')}</span>`;
                track.appendChild(tier);
            }
            document.getElementById('bpLevelText').innerText = `Nivel del Pase: ${bpLevel}`;
        }

        function addXP(amount) {
            bpXP += amount;
            if (bpXP >= 100) {
                bpXP -= 100;
                if (bpLevel < 100) bpLevel++;
                renderBattlePass();
            }
        }

        function addShipXP(type, amount) {
            shipXP[type] = (shipXP[type] || 0) + amount;
            if (shipXP[type] >= 100) {
                shipXP[type] -= 100;
                shipLevels[type] = (shipLevels[type] || 1) + 1;
                updateShipCardsUI();
            }
        }

        function updateShipCardsUI() {
            const solariLvl = shipLevels['solari'] || 1;
            const chronosLvl = shipLevels['chronos'] || 1;
            
            document.getElementById('solariGadgetText').innerText = `Nivel Nave: ${solariLvl} | Gadget [E]: ${solariLvl >= 4 ? 'Desbloqueado' : '🔒 Requiere Niv. 4'}`;
            document.getElementById('chronosGadgetText').innerText = `Nivel Nave: ${chronosLvl} | Gadget [E]: ${chronosLvl >= 4 ? 'Desbloqueado' : '🔒 Requiere Niv. 4'}`;
            
            const chronosCard = document.getElementById('cardChronos');
            if (bpLevel < 50) {
                chronosCard.classList.add('locked');
            } else {
                chronosCard.classList.remove('locked');
            }
        }

        // Control UI
        function selectShip(type) {
            if (type === 'chronos' && bpLevel < 50) return;
            selectedShip = type;
            document.getElementById('cardSolari').classList.toggle('active', type === 'solari');
            document.getElementById('cardChronos').classList.toggle('active', type === 'chronos');
        }

        function startGame() {
            player = new Player(selectedShip);
            enemies = [];
            bullets = [];
            missiles = [];
            shockwaves = [];
            score = 0;
            currentLevel = 1;
            isGameRunning = true;
            isPaused = false;

            document.getElementById('mainMenu').classList.add('hidden');
            document.getElementById('gameOverMenu').classList.add('hidden');
            document.getElementById('pauseMenu').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');

            requestAnimationFrame(gameLoop);
        }

        function togglePause() {
            isPaused = !isPaused;
            document.getElementById('pauseMenu').classList.toggle('hidden', !isPaused);
            if (!isPaused) requestAnimationFrame(gameLoop);
        }

        function returnToMenu() {
            isGameRunning = false;
            document.getElementById('pauseMenu').classList.add('hidden');
            document.getElementById('gameOverMenu').classList.add('hidden');
            document.getElementById('hud').classList.add('hidden');
            document.getElementById('mainMenu').classList.remove('hidden');
        }

        // Inicialización
        renderBattlePass();
        updateShipCardsUI();
    </script>
</body>
</html>
