<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Slope Pro Ultimate Deluxe v18 - Neon Edition</title>
<style>
    body {
        margin: 0;
        background: #03050f;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        overflow: hidden;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        color: white;
        -webkit-user-select: none;
        user-select: none;
    }
    canvas {
        background: #000000;
        border-radius: 24px;
        box-shadow: 0 0 120px rgba(0,242,255,0.35), inset 0 0 60px rgba(0,0,0,0.95);
        border: 2px solid rgba(0,242,255,0.5);
        touch-action: none;
    }
</style>
</head>
<body>

<canvas id="game" width="530" height="700"></canvas>

<script>
// ---------------------------------------------------------
// SISTEMA DE RÉCORDS Y DATOS GLOBALES V18
// ---------------------------------------------------------
let recordsFacil = JSON.parse(localStorage.getItem("slope_records_facil_v18")) || [];
let recordsDificil = JSON.parse(localStorage.getItem("slope_records_dificil_v18")) || [];

let unlockedEasySkins = JSON.parse(localStorage.getItem("slope_easy_skins_v18")) || [0];
let unlockedHardSkins = JSON.parse(localStorage.getItem("slope_hard_skins_v18")) || [0];
let selectedEasySkin = parseInt(localStorage.getItem("slope_sel_easy_v18")) || 0;
let selectedHardSkin = parseInt(localStorage.getItem("slope_sel_hard_v18")) || 0;

let playerName = prompt("Introduce tu nombre de Piloto:") || "Piloto_01";
let isAdminPro = false;

let oClickCount = 0;
let oClickTimer = 0;

// ---------------------------------------------------------
// PANEL DE HACKS CON NOMBRES ESPACIADOS
// ---------------------------------------------------------
let showAdminPanel = false;
let adminPanelAnim = 0;
let adminScrollY = 0;

let hacks = {
    invencibilidad: false,    
    multiplicadorVel: 1.0,  
    multiplicadorPts: 1,    
    sinLasers: false,        
    atravesarBloques: false,
    ralentizarBloques: false,
    congelarBloques: false,  
    jugadorMini: false,      
    jugadorGigante: false,  
    controlesInvertidos: false,
    pilotoAutomatico: false,
    modoEspejo: false,      
    estelaExtrarlarga: false,
    vibracionPantalla: true,
    lluviaBloques: false,    
    colorEstela: "#00f2ff",  
    impulsoAgilidad: 1.0,    
    modoArcoiris: false,    
    inmunidadLasers: false,  
    deslizamientoPuro: false
};

const hackDisplayNames = {
    invencibilidad: "INVENCIBILIDAD",
    multiplicadorVel: "MULTIPLICADOR VEL.",
    multiplicadorPts: "MULTIPLICADOR PTS.",
    sinLasers: "SIN LÁSERS",
    atravesarBloques: "ATRAVESAR BLOQUES",
    ralentizarBloques: "RALENTIZAR BLOQUES",
    congelarBloques: "CONGELAR BLOQUES",
    jugadorMini: "JUGADOR MINI",
    jugadorGigante: "JUGADOR GIGANTE",
    controlesInvertidos: "CONTROLES INVERTIDOS",
    pilotoAutomatico: "PILOTO AUTOMÁTICO",
    modoEspejo: "MODO ESPEJO",
    estelaExtrarlarga: "ESTELA EXTRA LARGA",
    vibracionPantalla: "VIBRACIÓN PANTALLA",
    lluviaBloques: "LLUVIA BLOQUES",
    colorEstela: "COLOR ESTELA",
    impulsoAgilidad: "IMPULSO AGILIDAD",
    modoArcoiris: "MODO ARCOIRIS",
    inmunidadLasers: "INMUNIDAD LÁSERS",
    deslizamientoPuro: "DESLIZAMIENTO PURO"
};

function checkSecretAdminTrigger(textToCheck){
    oClickCount++;
    if(oClickTimer) clearTimeout(oClickTimer);
    oClickTimer = setTimeout(() => { oClickCount = 0; }, 2000);

    if(oClickCount >= 3 || textToCheck.toLowerCase() === "o"){
        oClickCount = 0;
        let pass = prompt("🔒 Acceso Restringido. Introduce la contraseña de Administrador:");
        if(pass === "admin pro"){
            isAdminPro = true;
            unlockedEasySkins = [0,1,2,3,4,5,6,7,8,9];
            unlockedHardSkins = [0,1,2,3,4,5,6,7,8,9];
            localStorage.setItem("slope_easy_skins_v18", JSON.stringify(unlockedEasySkins));
            localStorage.setItem("slope_hard_skins_v18", JSON.stringify(unlockedHardSkins));
            alert("¡Contraseña correcta! Modo Admin Pro activado y todas las skins desbloqueadas.");
            showAdminPanel = true;
        } else if(pass !== null){
            alert("❌ Contraseña incorrecta.");
        }
    }
}

function checkSkinUnlocks(scoreVal, diff){
    let skinsEarned = Math.min(Math.floor(scoreVal / 1000), 10);
    if(diff === "facil"){
        let updated = false;
        for(let i=0; i<=skinsEarned; i++){
            if(!unlockedEasySkins.includes(i)){ unlockedEasySkins.push(i); updated = true; }
        }
        if(updated) localStorage.setItem("slope_easy_skins_v18", JSON.stringify(unlockedEasySkins));
    } else {
        let updated = false;
        for(let i=0; i<=skinsEarned; i++){
            if(!unlockedHardSkins.includes(i)){ unlockedHardSkins.push(i); updated = true; }
        }
        if(updated) localStorage.setItem("slope_hard_skins_v18", JSON.stringify(unlockedHardSkins));
    }
}

function saveRecord(name, scoreVal, diff){
    let list = diff === "facil" ? recordsFacil : recordsDificil;
    let existing = list.find(r => r.name === name);
    if(existing){
        if(scoreVal > existing.score) existing.score = scoreVal;
    } else {
        list.push({name: name, score: scoreVal});
    }
    list.sort((a, b) => b.score - a.score);
    if(list.length > 5) list = list.slice(0, 5);

    if(diff === "facil"){
        recordsFacil = list;
        localStorage.setItem("slope_records_facil_v18", JSON.stringify(recordsFacil));
    } else {
        recordsDificil = list;
        localStorage.setItem("slope_records_dificil_v18", JSON.stringify(recordsDificil));
    }
    checkSkinUnlocks(scoreVal, diff);
}

// ---------------------------------------------------------
// DEFINICIÓN DE SKINS
// ---------------------------------------------------------
let gameState = "menu";
let difficulty = "facil";

const EASY_SKINS = [
    { name: "Cian Neón (0k)", colors: ["#ffffff", "#00ffff", "#004466"] },
    { name: "Esmeralda Viva (1k)", colors: ["#ffffff", "#00ff88", "#005522"] },
    { name: "Fucsia Galáctico (2k)", colors: ["#ffffff", "#ff00cc", "#660055"] },
    { name: "Oro Estelar (3k)", colors: ["#ffffff", "#ffcc00", "#664400"] },
    { name: "Amatista Oscura (4k)", colors: ["#ffffff", "#9933ff", "#330066"] },
    { name: "Fuego Solar (5k)", colors: ["#ffffff", "#ff6600", "#662200"] },
    { name: "Azul Eléctrico (6k)", colors: ["#ffffff", "#3366ff", "#001166"] },
    { name: "Verde Matrix (7k)", colors: ["#ffffff", "#33ff33", "#003300"] },
    { name: "Rosa Neón Suave (8k)", colors: ["#ffffff", "#ff99cc", "#661133"] },
    { name: "Prisma Supremo (9k-10k+)", colors: ["#ffffff", "#ff0055", "#ffff00"] }
];

const HARD_SKINS = [
    { name: "Magma Carmesí (0k)", colors: ["#ffffff", "#ff1a00", "#660000"], shape: "magma" },
    { name: "Anillo Cuántico (1k)", colors: ["#ffffff", "#ff0044", "#440011"], shape: "ring" },
    { name: "Cristal Rubí (2k)", colors: ["#ffcccc", "#ff3333", "#550000"], shape: "hex" },
    { name: "Plasma Volátil (3k)", colors: ["#ffff00", "#ff0022", "#440000"], shape: "spike" },
    { name: "Ojo del Abismo (4k)", colors: ["#ff0055", "#cc0022", "#220008"], shape: "target" },
    { name: "Supernova Roja (5k)", colors: ["#ffffff", "#ff2222", "#660022"], shape: "star" },
    { name: "Triada Sangrienta (6k)", colors: ["#ff4444", "#880000", "#220000"], shape: "triangle" },
    { name: "Cubo Antimateria (7k)", colors: ["#ff5500", "#551100", "#1a0500"], shape: "square" },
    { name: "Espectro Infernal (8k)", colors: ["#ff6688", "#660022", "#22000a"], shape: "cross" },
    { name: "Singularidad Roja (9k-10k+)", colors: ["#ffffff", "#ff0000", "#110000"], shape: "diamond" }
];

// ---------------------------------------------------------
// CANVAS Y VARIABLES GLOBALES
// ---------------------------------------------------------
let c = document.getElementById("game");
let x = c.getContext("2d");
let CANVAS_WIDTH = 530;

let left=false, right=false, vel=0;
let player, obs, lasers, parts, bgParticles, speed, score, dead;
let bgOffset=0;
let trail=[];
let shakeTimer = 0;
let globalAnimTimer = 0;

function initBgParticles(){
    bgParticles = [];
    for(let i=0; i<70; i++){
        bgParticles.push({
            x: Math.random() * CANVAS_WIDTH,
            y: Math.random() * 700,
            size: Math.random() * 2.5 + 0.8,
            speedY: Math.random() * 3.0 + 1.2,
            alpha: Math.random() * 0.8 + 0.2
        });
    }
}

function startGame(diff){
    difficulty = diff;
    player = {x:CANVAS_WIDTH/2, y:560, r:16};
    obs = [];
    lasers = [];
    parts = [];
    trail = [];
    initBgParticles();
    speed = difficulty === "dificil" ? 1.5 : 1.4;
    score = 0;
    dead = false;
    gameState = "playing";
}

// ---------------------------------------------------------
// CONTROLES
// ---------------------------------------------------------
document.addEventListener("keydown", e => {
    if(e.key.toLowerCase() === "o"){
        checkSecretAdminTrigger("o");
        return;
    }

    if(e.key === "p" || e.key === "P"){
        if(isAdminPro && (gameState === "menu" || gameState === "playing")) {
            showAdminPanel = !showAdminPanel;
        }
        return;
    }

    if(showAdminPanel) return;

    if(gameState === "menu"){
        if(e.key === "1") startGame("facil");
        if(e.key === "2") startGame("dificil");
        if(e.key === "3") gameState = "skinselect";
        return;
    }
    if(gameState === "skinselect"){
        if(e.key === "3" || e.key === "m" || e.key === "M" || e.key === "Escape") gameState = "menu";
        return;
    }
    if(gameState === "gameover"){
        if(e.key === "1") startGame("facil");
        if(e.key === "2") startGame("dificil");
        if(e.key === "3" || e.key === "m" || e.key === "M") gameState = "menu";
        return;
    }
    if(gameState === "playing"){
        let lKey = hacks.controlesInvertidos ? e.key==="d"||e.key==="ArrowRight" : e.key==="a"||e.key==="ArrowLeft";
        let rKey = hacks.controlesInvertidos ? e.key==="a"||e.key==="ArrowLeft" : e.key==="d"||e.key==="ArrowRight";
        if(lKey) left=true;
        if(rKey) right=true;
    }
});

document.addEventListener("keyup", e => {
    let lKey = hacks.controlesInvertidos ? e.key==="d"||e.key==="ArrowRight" : e.key==="a"||e.key==="ArrowLeft";
    let rKey = hacks.controlesInvertidos ? e.key==="a"||e.key==="ArrowLeft" : e.key==="d"||e.key==="ArrowRight";
    if(lKey) left=false;
    if(rKey) right=false;
});

window.addEventListener("pointerdown", e => {
    let rect = c.getBoundingClientRect();
    let touchX = e.clientX - rect.left;
    let touchY = e.clientY - rect.top;

    if(isAdminPro && showAdminPanel){
        if(touchX >= 400 && touchX <= 485 && touchY >= 45 && touchY <= 85) { showAdminPanel = false; return; }
       
        let localY = touchY - 95 + adminScrollY;
        let hackKeys = Object.keys(hacks);
        let btnHeight = 42;
        let gap = 8;

        for(let i=0; i<hackKeys.length; i++){
            let by = i * (btnHeight + gap);
            if(localY >= by && localY <= by + btnHeight && touchX >= 45 && touchX <= 485){
                let k = hackKeys[i];
                if(k === "multiplicadorVel"){
                    hacks.multiplicadorVel = hacks.multiplicadorVel === 1.0 ? 0.5 : (hacks.multiplicadorVel === 0.5 ? 2.0 : (hacks.multiplicadorVel === 2.0 ? 5.0 : 1.0));
                } else if(k === "multiplicadorPts"){
                    hacks.multiplicadorPts = hacks.multiplicadorPts === 1 ? 5 : (hacks.multiplicadorPts === 5 ? 10 : 1);
                } else if(k === "impulsoAgilidad"){
                    hacks.impulsoAgilidad = hacks.impulsoAgilidad === 1.0 ? 2.0 : (hacks.impulsoAgilidad === 2.0 ? 0.5 : 1.0);
                } else if(k === "colorEstela"){
                    let colors = ["#00f2ff", "#ff0055", "#00ff88", "#ffcc00", "#ff0000"];
                    let idx = colors.indexOf(hacks.colorEstela);
                    hacks.colorEstela = colors[(idx + 1) % colors.length];
                } else if(typeof hacks[k] === "boolean"){
                    hacks[k] = !hacks[k];
                }
                return;
            }
        }
        return;
    }

    if(gameState === "menu"){
        if(touchX >= 50 && touchX <= 480 && touchY >= 105 && touchY <= 150) startGame("facil");
        if(touchX >= 50 && touchX <= 480 && touchY >= 158 && touchY <= 203) startGame("dificil");
        if(touchX >= 50 && touchX <= 480 && touchY >= 212 && touchY <= 252) gameState = "skinselect";
    } else if(gameState === "skinselect"){
        if(touchX >= 165 && touchX <= 365 && touchY >= 610 && touchY <= 650) gameState = "menu";
        let isHardTab = difficulty === "dificil";
        let skinList = isHardTab ? HARD_SKINS : EASY_SKINS;
        let unlocked = isHardTab ? unlockedHardSkins : unlockedEasySkins;
       
        if(touchY >= 95 && touchY <= 130){
            difficulty = difficulty === "facil" ? "dificil" : "facil";
            return;
        }

        let startY = 155;
        for(let i=0; i<skinList.length; i++){
            let col = i % 2;
            let row = Math.floor(i / 2);
            let bx = 55 + col * 210;
            let by = startY + row * 76;
            if(touchX >= bx && touchX <= bx + 195 && touchY >= by && touchY <= by + 68){
                if(unlocked.includes(i) || isAdminPro){
                    if(isHardTab) {
                        selectedHardSkin = i;
                        localStorage.setItem("slope_sel_hard_v18", i);
                    } else {
                        selectedEasySkin = i;
                        localStorage.setItem("slope_sel_easy_v18", i);
                    }
                }
            }
        }
    } else if(gameState === "playing"){
        let moveLeft = hacks.controlesInvertidos ? touchX >= CANVAS_WIDTH / 2 : touchX < CANVAS_WIDTH / 2;
        if(moveLeft) { left = true; right = false; }
        else { right = true; left = false; }
    } else if(gameState === "gameover"){
        if(touchX >= 65 && touchX <= 250 && touchY >= 290 && touchY <= 335) startGame("facil");
        if(touchX >= 280 && touchX <= 465 && touchY >= 290 && touchY <= 335) startGame("dificil");
        if(touchX >= 165 && touchX <= 365 && touchY >= 355 && touchY <= 395) gameState = "menu";
    }
});

window.addEventListener("pointerup", () => { left = false; right = false; });

window.addEventListener("wheel", e => {
    if(isAdminPro && showAdminPanel){
        adminScrollY += e.deltaY * 0.5;
        if(adminScrollY < 0) adminScrollY = 0;
        if(adminScrollY > 800) adminScrollY = 800;
    }
});

// ---------------------------------------------------------
// FÍSICAS Y GENERACIÓN DE OBSTÁCULOS
// ---------------------------------------------------------
function boom(px,py){
    if(hacks.vibracionPantalla) shakeTimer = 18;
    for(let i=0; i<65; i++){
        parts.push({
            x:px, y:py,
            dx:(Math.random()-.5)*16,
            dy:(Math.random()-.5)*16,
            life:60,
            col: difficulty === "dificil" ? "rgba(255,30,30,1)" : "rgba(0,242,255,1)"
        });
    }
}

function drawParts(){
    parts.forEach(p=>{
        x.fillStyle = p.col.replace("1)", (p.life/60)+")");
        x.fillRect(p.x, p.y, 5, 5);
        p.x += p.dx; p.y += p.dy; p.life--;
    });
    parts = parts.filter(p => p.life > 0);
}

function spawnWave(){
    let count = Math.random() < 0.6 ? 1 : 2;
    let w = 85 + Math.random() * 40;
    let availableWidth = CANVAS_WIDTH - 60;
   
    let blockDx = (Math.random() < 0.5 ? -1 : 1) * 1.6;

    if(count === 1){
        let px = 30 + Math.random() * (availableWidth - w);
        let speedFactor = 0.8 + Math.random() * 0.4;
        obs.push({
            x: px,
            y: -110,
            w: w,
            h: 34,
            dx: blockDx,
            speedFactor: speedFactor,
            hue: difficulty === "dificil" ? Math.random() * 20 + 345 : Math.random() * 360,
            bounceAnim: 0
        });
    } else {
        let w1 = 70 + Math.random() * 20;
        let w2 = 70 + Math.random() * 20;
        let gap = 75 + Math.random() * 35;
       
        let px1 = 30 + Math.random() * 30;
        let px2 = px1 + w1 + gap;
        if(px2 + w2 > CANVAS_WIDTH - 30) {
            px2 = CANVAS_WIDTH - 30 - w2;
            px1 = px2 - gap - w1;
        }

        [px1, px2].forEach(px => {
            let speedFactor = 0.8 + Math.random() * 0.4;
            obs.push({
                x: px,
                y: -110,
                w: w1,
                h: 34,
                dx: blockDx,
                speedFactor: speedFactor,
                hue: difficulty === "dificil" ? Math.random() * 20 + 345 : Math.random() * 360,
                bounceAnim: 0
            });
        });
    }
}

function drawBlocks(){
    obs.forEach(o => {
        o.bounceAnim = Math.max(0, o.bounceAnim - 0.08);
        let scaleOffset = o.bounceAnim * 7;

        x.shadowBlur = 15 + o.bounceAnim * 10;
        x.shadowColor = difficulty === "dificil" ? "#ff2222" : "hsl(" + o.hue + ", 95%, 50%)";
       
        let grad = x.createLinearGradient(o.x - scaleOffset, o.y - scaleOffset, o.x, o.y + o.h + scaleOffset);
        grad.addColorStop(0, difficulty === "dificil" ? "#ff6666" : "hsl(" + o.hue + ", 90%, 80%)");
        grad.addColorStop(0.5, difficulty === "dificil" ? "#cc0000" : "hsl(" + o.hue + ", 80%, 50%)");
        grad.addColorStop(1, difficulty === "dificil" ? "#550000" : "hsl(" + o.hue + ", 70%, 28%)");
       
        x.fillStyle = grad;
        x.fillRect(o.x - scaleOffset/2, o.y - scaleOffset/2, o.w + scaleOffset, o.h + scaleOffset);
       
        x.fillStyle = "rgba(255,255,255,0.65)";
        x.fillRect(o.x - scaleOffset/2, o.y - scaleOffset/2, o.w + scaleOffset, 5);
       
        x.shadowBlur = 0;
    });
}

function spawnLaser(){
    if(hacks.sinLasers) return;
    let w = 70 + Math.random() * 60;
    let xPos = Math.random() * (CANVAS_WIDTH - w);
    lasers.push({ x: xPos, y: -60, w: w, h: 12, timer: 0, phase: 0, hue: difficulty === "dificil" ? 0 : 190 });
}

function updateLasers(){
    if(difficulty === "dificil"){
        let laserChance = 0.0035;
        if(Math.random() < laserChance) spawnLaser();
    }

    lasers.forEach(l => {
        l.timer++;
        let spd = hacks.congelarBloques ? 0 : (hacks.ralentizarBloques ? speed * 0.35 : speed);
        if(l.timer < 50) l.phase = 0;
        else if(l.timer < 110){ l.phase = 1; l.y += spd * 0.5 * hacks.multiplicadorVel; }
        else { l.phase = 2; l.y += spd * 1.2 * hacks.multiplicadorVel; }
    });
    lasers = lasers.filter(l => l.y < 760);
}

function drawLasers(){
    lasers.forEach(l => {
        let col = difficulty === "dificil" || l.hue === 0 ? "rgba(255, 30, 30, " : "rgba(0, 242, 255, ";
        if(l.phase === 0){
            x.strokeStyle = col + "0.7)";
            x.lineWidth = 2.5;
            x.setLineDash([7, 7]);
            x.beginPath(); x.moveTo(l.x - 5, l.y); x.lineTo(l.x + l.w + 5, l.y); x.stroke();
            x.setLineDash([]);
        } else {
            x.shadowBlur = 18;
            x.shadowColor = difficulty === "dificil" || l.hue === 0 ? "#ff0000" : "#00f2ff";
            x.fillStyle = col + "0.9)";
            x.fillRect(l.x, l.y, l.w, l.h);
            x.shadowBlur = 0;
        }
    });
}

function hitCircleRect(px, py, pr, rx, ry, rw, rh){
    let cx = Math.max(rx, Math.min(px, rx + rw));
    let cy = Math.max(ry, Math.min(py, ry + rh));
    let dx = px - cx, dy = py - cy;
    return dx * dx + dy * dy < pr * pr;
}

// ---------------------------------------------------------
// FONDO Y ESTELA
// ---------------------------------------------------------
function drawBackground(){
    let bgSpd = hacks.congelarBloques ? 0 : speed * 1.4 * hacks.multiplicadorVel;
    bgOffset += bgSpd;

    let g = x.createLinearGradient(0, 0, 0, 700);
    if(difficulty === "dificil"){
        g.addColorStop(0, "#2b0308");
        g.addColorStop(0.5, "#140103");
        g.addColorStop(1, "#000000");
    } else {
        g.addColorStop(0, "#080b1e");
        g.addColorStop(0.5, "#030612");
        g.addColorStop(1, "#000000");
    }
    x.fillStyle = g;
    x.fillRect(0, 0, CANVAS_WIDTH, 700);

    x.strokeStyle = difficulty === "dificil" ? "rgba(255, 30, 50, 0.15)" : "rgba(0,242,255,0.12)";
    x.lineWidth = 1.4;
    let step = 50, off = bgOffset % step;
    for(let i = -100; i < 700; i += step){
        x.beginPath(); x.moveTo(0, i + off); x.lineTo(CANVAS_WIDTH, i + off); x.stroke();
    }
    for(let i = 0; i <= CANVAS_WIDTH; i += 70){
        x.beginPath(); x.moveTo(CANVAS_WIDTH / 2, 150); x.lineTo(i, 700); x.stroke();
    }

    if(bgParticles){
        bgParticles.forEach(p => {
            x.fillStyle = difficulty === "dificil" ? `rgba(255,80,80,${p.alpha})` : `rgba(120,245,255,${p.alpha})`;
            x.fillRect(p.x, p.y, p.size, p.size * 4);
            p.y += (p.speedY + bgSpd * 0.4);
            if(p.y > 700){ p.y = -10; p.x = Math.random() * CANVAS_WIDTH; }
        });
    }
}

function updateTrail(){
    let maxT = hacks.estelaExtrarlarga ? 70 : 26;
    trail.push({ x: player.x, y: player.y, r: player.r });
    if(trail.length > maxT) trail.shift();
}

function drawTrail(){
    for(let i = 0; i < trail.length; i++){
        let t = trail[i], a = i / trail.length * 0.5;
        let col;
        if(difficulty === "dificil"){
            col = `rgba(255, 20, 20, ${a * 2.2})`;
        } else if(hacks.modoArcoiris){
            let hueAngle = (globalAnimTimer * 6 + i * 8) % 360;
            col = `hsla(${hueAngle}, 100%, 50%, ${a * 1.5})`;
        } else if(hacks.invencibilidad){
            col = `rgba(255,215,0,${a})`;
        } else {
            let r = parseInt(hacks.colorEstela.slice(1,3), 16);
            let g = parseInt(hacks.colorEstela.slice(3,5), 16);
            let b = parseInt(hacks.colorEstela.slice(5,7), 16);
            col = `rgba(${r},${g},${b},${a})`;
        }
        x.fillStyle = col;
        x.beginPath(); x.arc(t.x, t.y, t.r * 0.65, 0, Math.PI * 2); x.fill();
    }
}

function drawPlayerSkin(){
    let skinData = difficulty === "dificil" ? HARD_SKINS[selectedHardSkin] : EASY_SKINS[selectedEasySkin];
    let skinColors = skinData.colors;
    let shape = difficulty === "dificil" ? skinData.shape : "circle";

    player.r = hacks.jugadorMini ? 9 : (hacks.jugadorGigante ? 26 : 16);

    x.shadowBlur = 22;
    x.shadowColor = difficulty === "dificil" ? "#ff0000" : (hacks.modoArcoiris ? `hsl(${globalAnimTimer * 4 % 360}, 100%, 50%)` : (hacks.invencibilidad ? "#ffd700" : skinColors[1]));

    if(shape === "magma"){
        let grad = x.createRadialGradient(player.x, player.y, 2, player.x, player.y, player.r);
        grad.addColorStop(0, skinColors[0]);
        grad.addColorStop(0.5, skinColors[1]);
        grad.addColorStop(1, skinColors[2]);
        x.fillStyle = grad;
        x.beginPath(); x.arc(player.x, player.y, player.r, 0, Math.PI * 2); x.fill();
        x.strokeStyle = "#ffffff"; x.lineWidth = 2;
        x.beginPath(); x.arc(player.x, player.y, player.r * 0.65, 0, Math.PI * 2); x.stroke();
    } else if(shape === "ring"){
        x.strokeStyle = skinColors[1]; x.lineWidth = 4;
        x.beginPath(); x.arc(player.x, player.y, player.r, 0, Math.PI * 2); x.stroke();
        x.fillStyle = skinColors[2];
        x.beginPath(); x.arc(player.x, player.y, player.r * 0.5, 0, Math.PI * 2); x.fill();
    } else if(shape === "hex"){
        x.fillStyle = skinColors[1];
        x.beginPath();
        for(let i = 0; i < 6; i++){
            let angle = i * Math.PI / 3 + (globalAnimTimer * 0.02);
            let hx = player.x + player.r * Math.cos(angle);
            let hy = player.y + player.r * Math.sin(angle);
            i === 0 ? x.moveTo(hx, hy) : x.lineTo(hx, hy);
        }
        x.closePath(); x.fill();
        x.strokeStyle = skinColors[0]; x.lineWidth = 2; x.stroke();
    } else if(shape === "spike"){
        x.fillStyle = skinColors[1];
        x.beginPath();
        for(let i = 0; i < 8; i++){
            let angle = i * Math.PI / 4;
            let dist = i % 2 === 0 ? player.r * 1.2 : player.r * 0.5;
            let sx = player.x + dist * Math.cos(angle);
            let sy = player.y + dist * Math.sin(angle);
            i === 0 ? x.moveTo(sx, sy) : x.lineTo(sx, sy);
        }
        x.closePath(); x.fill();
    } else if(shape === "target"){
        x.fillStyle = skinColors[2];
        x.fillRect(player.x - player.r, player.y - player.r, player.r * 2, player.r * 2);
        x.strokeStyle = skinColors[1]; x.lineWidth = 3;
        x.strokeRect(player.x - player.r, player.y - player.r, player.r * 2, player.r * 2);
        x.fillStyle = skinColors[0];
        x.beginPath(); x.arc(player.x, player.y, player.r * 0.4, 0, Math.PI * 2); x.fill();
    } else if(shape === "star"){
        x.fillStyle = skinColors[1];
        x.beginPath();
        for(let i = 0; i < 5; i++){
            let angle = i * 2 * Math.PI / 5 - Math.PI / 2;
            let angleInner = angle + Math.PI / 5;
            let x1 = player.x + player.r * 1.3 * Math.cos(angle);
            let y1 = player.y + player.r * 1.3 * Math.sin(angle);
            let x2 = player.x + (player.r * 0.5) * Math.cos(angleInner);
            let y2 = player.y + (player.r * 0.5) * Math.sin(angleInner);
            if(i === 0) x.moveTo(x1, y1); else x.lineTo(x1, y1);
            x.lineTo(x2, y2);
        }
        x.closePath(); x.fill();
    } else if(shape === "triangle"){
        x.fillStyle = skinColors[1];
        x.beginPath();
        x.moveTo(player.x, player.y - player.r * 1.3);
        x.lineTo(player.x + player.r * 1.1, player.y + player.r);
        x.lineTo(player.x - player.r * 1.1, player.y + player.r);
        x.closePath(); x.fill();
        x.strokeStyle = skinColors[0]; x.lineWidth = 2.5; x.stroke();
    } else if(shape === "square"){
        x.fillStyle = skinColors[1];
        x.save();
        x.translate(player.x, player.y);
        x.rotate(globalAnimTimer * 0.03);
        x.fillRect(-player.r * 0.8, -player.r * 0.8, player.r * 1.6, player.r * 1.6);
        x.strokeStyle = skinColors[0]; x.lineWidth = 2;
        x.strokeRect(-player.r * 0.8, -player.r * 0.8, player.r * 1.6, player.r * 1.6);
        x.restore();
    } else if(shape === "cross"){
        x.fillStyle = skinColors[1];
        x.beginPath();
        let tSize = player.r * 0.45;
        x.rect(player.x - tSize, player.y - player.r, tSize * 2, player.r * 2);
        x.rect(player.x - player.r, player.y - tSize, player.r * 2, tSize * 2);
        x.fill();
        x.strokeStyle = skinColors[0]; x.lineWidth = 2;
        x.stroke();
    } else if(shape === "diamond"){
        x.fillStyle = skinColors[1];
        x.beginPath();
        x.moveTo(player.x, player.y - player.r * 1.4);
        x.lineTo(player.x + player.r * 1.1, player.y);
        x.lineTo(player.x, player.y + player.r * 1.4);
        x.lineTo(player.x - player.r * 1.1, player.y);
        x.closePath(); x.fill();
        x.fillStyle = skinColors[0];
        x.beginPath(); x.arc(player.x, player.y, player.r * 0.35, 0, Math.PI * 2); x.fill();
    } else {
        let grad = x.createRadialGradient(player.x - 4, player.y - 4, 2, player.x, player.y, player.r);
        grad.addColorStop(0, skinColors[0]);
        grad.addColorStop(0.5, skinColors[1]);
        grad.addColorStop(1, skinColors[2]);
        x.fillStyle = grad;
        x.beginPath(); x.arc(player.x, player.y, player.r, 0, Math.PI * 2); x.fill();
    }
    x.shadowBlur = 0;
}

// ---------------------------------------------------------
// BUCLE DE ACTUALIZACIÓN PRINCIPAL
// ---------------------------------------------------------
function update(){
    globalAnimTimer++;

    if(showAdminPanel){
        if(adminPanelAnim < 1) adminPanelAnim += 0.15;
    } else {
        if(adminPanelAnim > 0) adminPanelAnim -= 0.15;
    }

    if(gameState === "playing"){
        score += Math.round(1 * hacks.multiplicadorPts * hacks.multiplicadorVel);
        speed = ((difficulty === "dificil" ? 1.5 : 1.4) + score / 1600);

        checkSkinUnlocks(score, difficulty);

        let accel = 0.4 * hacks.impulsoAgilidad;
        let friction = hacks.deslizamientoPuro ? 1.0 : 0.87;

        if(hacks.pilotoAutomatico){
            let targetX = CANVAS_WIDTH / 2;
            for(let o of obs){
                if(o.y > 300 && o.y < 520){
                    if(player.x < o.x + o.w / 2) targetX = o.x - 30;
                    else targetX = o.x + o.w + 30;
                }
            }
            if(player.x < targetX) vel += accel;
            if(player.x > targetX) vel -= accel;
        } else {
            if(left) vel -= accel;
            if(right) vel += accel;
        }

        vel *= friction;
        player.x += vel;

        if(player.x < 30) player.x = 30;
        if(player.x > CANVAS_WIDTH - 30) player.x = CANVAS_WIDTH - 30;

        // Frecuencia de aparición de bloques reducida muy poco en difícil (0.011 -> 0.040)
        let spawnRate = hacks.lluviaBloques ? 0.07 : (difficulty === "dificil" ? 0.0095 : 0.015);
        if(Math.random() < spawnRate) spawnWave();

        let blockSpeedMul = hacks.congelarBloques ? 0 : (hacks.ralentizarBloques ? 0.4 : 1.0);
       
        for (let i = 0; i < obs.length; i++) {
            let o = obs[i];
            o.y += speed * o.speedFactor * blockSpeedMul * hacks.multiplicadorVel;
            o.x += o.dx * hacks.multiplicadorVel;
           
            // 1. Rebotes contra las paredes laterales (para ambos modos)
            if(o.dx !== 0){
                if(o.x < 30){
                    o.x = 30;
                    o.dx *= -1;
                    o.bounceAnim = 1.0;
                }
                if(o.x + o.w > CANVAS_WIDTH - 30){
                    o.x = CANVAS_WIDTH - 30 - o.w;
                    o.dx *= -1;
                    o.bounceAnim = 1.0;
                }

                // 2. Rebotes entre bloques (SOLO EN MODO DIFÍCIL)
                if(difficulty === "dificil"){
                    for (let j = i + 1; j < obs.length; j++) {
                        let o2 = obs[j];
                        if (Math.abs(o.y - o2.y) < 40) {
                            if (o.x < o2.x + o2.w && o.x + o.w > o2.x) {
                                let tempDx = o.dx;
                                o.dx = o2.dx;
                                o2.dx = tempDx;

                                if (o.x < o2.x) {
                                    o.x -= 3;
                                    o2.x += 3;
                                } else {
                                    o.x += 3;
                                    o2.x -= 3;
                                }

                                o.bounceAnim = 1.0;
                                o2.bounceAnim = 1.0;
                            }
                        }
                    }

                    // 3. Rebotes de bloques contra láseres (SOLO EN MODO DIFÍCIL)
                    lasers.forEach(l => {
                        if (l.phase >= 1 && Math.abs(o.y - l.y) < 30) {
                            if (o.x < l.x + l.w && o.x + o.w > l.x) {
                                o.dx *= -1;
                                o.bounceAnim = 1.0;
                                if (o.x < l.x) o.x = l.x - o.w - 2;
                                else o.x = l.x + l.w + 2;
                            }
                        }
                    });
                }
            }
        }

        obs = obs.filter(o => o.y < 760);

        updateLasers();
        updateTrail();

        if(!hacks.invencibilidad){
            if(!hacks.atravesarBloques){
                obs.forEach(o => {
                    if(hitCircleRect(player.x, player.y, player.r * 0.9, o.x, o.y, o.w, o.h)){
                        dead = true; boom(player.x, player.y); saveRecord(playerName, score, difficulty); gameState = "gameover";
                    }
                });
            }
            if(!hacks.inmunidadLasers){
                lasers.forEach(l => {
                    if(l.phase >= 1 && hitCircleRect(player.x, player.y, player.r * 0.9, l.x, l.y, l.w, l.h)){
                        dead = true; boom(player.x, player.y); saveRecord(playerName, score, difficulty); gameState = "gameover";
                    }
                });
            }
        }
    }

    if(shakeTimer > 0) shakeTimer--;

    draw();
    requestAnimationFrame(update);
}

function drawRoundedCard(xPos, yPos, w, h, r, fill, stroke) {
    x.beginPath();
    x.moveTo(xPos + r, yPos);
    x.arcTo(xPos + w, yPos, xPos + w, yPos + h, r);
    x.arcTo(xPos + w, yPos + h, xPos, yPos + h, r);
    x.arcTo(xPos, yPos + h, xPos, yPos, r);
    x.arcTo(xPos, yPos, xPos + w, yPos, r);
    x.closePath();
    x.fillStyle = fill; x.fill();
    if(stroke){ x.strokeStyle = stroke; x.lineWidth = 2; x.stroke(); }
}

        // TASA DE APARICIÓN EQUILIBRADA (NIVEL INTERMEDIO)
        let spawnRate = 0.020;
        if (hacks.lluviaBloques) {
            spawnRate = 0.053;
        } else if (difficulty === "facil") {
            spawnRate = 0.0087 + (speed * 0.0005); 
        } else {
            // Un poco más bajo que el anterior para que sea retador pero más accesible
            spawnRate = 0.042 + (speed * 0.0027);
        }


// ---------------------------------------------------------
// RENDERIZADO VISUAL
// ---------------------------------------------------------
function drawMenu(){
    drawRoundedCard(35, 30, CANVAS_WIDTH - 70, 640, 20, "rgba(5, 10, 25, 0.85)", "rgba(0, 242, 255, 0.35)");

    x.shadowBlur = 20;
    x.shadowColor = "#00f2ff";
    x.fillStyle = "#ffffff";
    x.font = "bold 26px 'Segoe UI', sans-serif";
    x.textAlign = "center";
    x.fillText("SLOPE PRO ULTIMATE", CANVAS_WIDTH / 2, 68);
   
    x.font = "bold 12px 'Segoe UI', sans-serif";
    x.fillStyle = "#00f2ff";
    x.fillText("NEON EDITION v18", CANVAS_WIDTH / 2, 88);

    // Botón Modo Fácil
    drawRoundedCard(50, 105, CANVAS_WIDTH - 100, 45, 10, "rgba(0, 242, 255, 0.15)", "#00f2ff");
    x.fillStyle = "#ffffff";
    x.font = "bold 16px 'Segoe UI', sans-serif";
    x.fillText("MODO FÁCIL", CANVAS_WIDTH / 2, 126);
    x.font = "11px 'Segoe UI', sans-serif";
    x.fillStyle = "#aae0ff";
    x.fillText("Movimiento lateral sin rebote entre bloques", CANVAS_WIDTH / 2, 142);

    // Botón Modo Difícil
    drawRoundedCard(50, 158, CANVAS_WIDTH - 100, 45, 10, "rgba(255, 30, 50, 0.15)", "#ff1e32");
    x.fillStyle = "#ffffff";
    x.font = "bold 16px 'Segoe UI', sans-serif";
    x.fillText("MODO DIFÍCIL", CANVAS_WIDTH / 2, 179);
    x.font = "11px 'Segoe UI', sans-serif";
    x.fillStyle = "#ffaacc";
    x.fillText("Bloques rebotan entre sí, láseres y más", CANVAS_WIDTH / 2, 195);

    // Botón Selector de Skins
    drawRoundedCard(50, 212, CANVAS_WIDTH - 100, 40, 10, "rgba(150, 50, 255, 0.15)", "#9632ff");
    x.fillStyle = "#ffffff";
    x.font = "bold 14px 'Segoe UI', sans-serif";
    x.fillText("SELECTOR DE SKINS", CANVAS_WIDTH / 2, 237);

    // TOP 5 FÁCIL
    x.fillStyle = "#00f2ff";
    x.font = "bold 12px 'Segoe UI', sans-serif";
    x.textAlign = "left";
    x.fillText("🏆 TOP 5 - MODO FÁCIL", 55, 275);
   
    let startYF = 290;
    if(recordsFacil.length === 0){
        x.fillStyle = "#888888";
        x.font = "11px 'Segoe UI', sans-serif";
        x.fillText("Sin registros aún", 55, startYF + 12);
    } else {
        for(let i = 0; i < Math.min(recordsFacil.length, 5); i++){
            x.fillStyle = "#ffffff";
            x.font = "11px 'Segoe UI', sans-serif";
            x.fillText(`${i+1}. ${recordsFacil[i].name}`, 55, startYF + (i * 18));
            x.textAlign = "right";
            x.fillStyle = "#00ffcc";
            x.fillText(`${recordsFacil[i].score} pts`, CANVAS_WIDTH - 55, startYF + (i * 18));
            x.textAlign = "left";
        }
    }

    // TOP 5 DIFÍCIL
    x.fillStyle = "#ff4444";
    x.font = "bold 12px 'Segoe UI', sans-serif";
    x.fillText("🏆 TOP 5 - MODO DIFÍCIL", 55, 395);
   
    let startYHD = 410;
    if(recordsDificil.length === 0){
        x.fillStyle = "#888888";
        x.font = "11px 'Segoe UI', sans-serif";
        x.fillText("Sin registros aún", 55, startYHD + 12);
    } else {
        for(let i = 0; i < Math.min(recordsDificil.length, 5); i++){
            x.fillStyle = "#ffffff";
            x.font = "11px 'Segoe UI', sans-serif";
            x.fillText(`${i+1}. ${recordsDificil[i].name}`, 55, startYHD + (i * 18));
            x.textAlign = "right";
            x.fillStyle = "#ff6666";
            x.fillText(`${recordsDificil[i].score} pts`, CANVAS_WIDTH - 55, startYHD + (i * 18));
            x.textAlign = "left";
        }
    }

    x.shadowBlur = 0;
}

function drawSkinSelect(){
    drawRoundedCard(35, 45, CANVAS_WIDTH - 70, 610, 20, "rgba(5, 10, 25, 0.85)", "rgba(150, 50, 255, 0.4)");

    x.fillStyle = "#ffffff";
    x.font = "bold 24px 'Segoe UI', sans-serif";
    x.textAlign = "center";
    x.fillText("SELECCIÓN DE SKINS", CANVAS_WIDTH / 2, 85);

    drawRoundedCard(165, 95, 200, 35, 10, "rgba(0,0,0,0.5)", "#9632ff");
    x.fillStyle = "#ffcc00";
    x.font = "bold 14px 'Segoe UI', sans-serif";
    x.fillText(difficulty === "facil" ? "◄ MODO FÁCIL ►" : "◄ MODO DIFÍCIL ►", CANVAS_WIDTH / 2, 118);

    let isHardTab = difficulty === "dificil";
    let skinList = isHardTab ? HARD_SKINS : EASY_SKINS;
    let unlocked = isHardTab ? unlockedHardSkins : unlockedEasySkins;
    let selected = isHardTab ? selectedHardSkin : selectedEasySkin;

    let startY = 155;
    for(let i=0; i<skinList.length; i++){
        let col = i % 2;
        let row = Math.floor(i / 2);
        let bx = 55 + col * 210;
        let by = startY + row * 76;
        let isUnlocked = unlocked.includes(i) || isAdminPro;
        let isSelected = selected === i;

        let cardBg = isSelected ? "rgba(0, 242, 255, 0.2)" : (isUnlocked ? "rgba(20, 30, 60, 0.6)" : "rgba(10, 10, 15, 0.6)");
        let cardBorder = isSelected ? "#00f2ff" : (isUnlocked ? "rgba(0, 242, 255, 0.3)" : "rgba(255,255,255,0.1)");

        drawRoundedCard(bx, by, 195, 68, 10, cardBg, cardBorder);

        x.fillStyle = isUnlocked ? "#ffffff" : "#666666";
        x.font = "bold 13px 'Segoe UI', sans-serif";
        x.textAlign = "left";
        x.fillText(skinList[i].name, bx + 12, by + 26);

        x.font = "11px 'Segoe UI', sans-serif";
        x.fillStyle = isUnlocked ? (isSelected ? "#00f2ff" : "#00ff88") : "#ff4444";
        x.fillText(isSelected ? "✓ SELECCIONADA" : (isUnlocked ? "DESBLOQUEADA" : "🔒 BLOQUEADA"), bx + 12, by + 48);
    }

    drawRoundedCard(165, 610, 200, 40, 12, "rgba(255,255,255,0.1)", "#ffffff");
    x.fillStyle = "#ffffff";
    x.font = "bold 15px 'Segoe UI', sans-serif";
    x.textAlign = "center";
    x.fillText("VOLVER AL MENÚ", CANVAS_WIDTH / 2, 635);
}

function drawGameOver(){
    drawRoundedCard(50, 160, CANVAS_WIDTH - 100, 310, 20, "rgba(15, 5, 10, 0.9)", "#ff1e32");

    x.shadowBlur = 15;
    x.shadowColor = "#ff0000";
    x.fillStyle = "#ff3333";
    x.font = "bold 32px 'Segoe UI', sans-serif";
    x.textAlign = "center";
    x.fillText("¡FIN DEL JUEGO!", CANVAS_WIDTH / 2, 215);

    x.shadowBlur = 0;
    x.fillStyle = "#ffffff";
    x.font = "bold 18px 'Segoe UI', sans-serif";
    x.fillText("Puntuación Final: " + score, CANVAS_WIDTH / 2, 260);

    drawRoundedCard(65, 290, 185, 45, 10, "rgba(0, 242, 255, 0.2)", "#00f2ff");
    x.fillStyle = "#ffffff";
    x.font = "bold 14px 'Segoe UI', sans-serif";
    x.fillText("REINTENTAR FÁCIL", 157, 318);

    drawRoundedCard(280, 290, 185, 45, 10, "rgba(255, 30, 50, 0.2)", "#ff1e32");
    x.fillText("REINTENTAR DIFÍCIL", 372, 318);

    drawRoundedCard(165, 355, 200, 40, 10, "rgba(255,255,255,0.1)", "#ffffff");
    x.fillText("MENÚ PRINCIPAL", CANVAS_WIDTH / 2, 381);
}

function drawAdminPanel(){
    if(adminPanelAnim <= 0) return;

    x.save();
    x.globalAlpha = adminPanelAnim;
    drawRoundedCard(30, 30, CANVAS_WIDTH - 60, 640, 16, "rgba(5, 5, 15, 0.95)", "#00ffcc");

    x.fillStyle = "#00ffcc";
    x.font = "bold 18px 'Segoe UI', sans-serif";
    x.textAlign = "left";
    x.fillText("⚙️ PANEL ADMIN PRO", 50, 68);

    drawRoundedCard(400, 45, 85, 40, 8, "rgba(255,30,50,0.3)", "#ff1e32");
    x.fillStyle = "#ffffff";
    x.font = "bold 13px 'Segoe UI', sans-serif";
    x.textAlign = "center";
    x.fillText("CERRAR", 442, 70);

    x.save();
    x.beginPath();
    x.rect(40, 90, CANVAS_WIDTH - 80, 560);
    x.clip();

    let localY = 95 - adminScrollY;
    let hackKeys = Object.keys(hacks);
    let btnHeight = 42;
    let gap = 8;

    for(let i=0; i<hackKeys.length; i++){
        let k = hackKeys[i];
        let by = localY + i * (btnHeight + gap);
       
        if(by > -50 && by < 700){
            let val = hacks[k];
            let active = typeof val === "boolean" ? val : val !== 1.0 && val !== 1;
            let bgCol = active ? "rgba(0, 255, 204, 0.2)" : "rgba(255,255,255,0.05)";
            let borderCol = active ? "#00ffcc" : "rgba(255,255,255,0.15)";

            drawRoundedCard(45, by, CANVAS_WIDTH - 90, btnHeight, 8, bgCol, borderCol);

            x.fillStyle = "#ffffff";
            x.font = "bold 13px 'Segoe UI', sans-serif";
            x.textAlign = "left";
            x.fillText(hackDisplayNames[k] || k, 65, by + 26);

            x.textAlign = "right";
            x.fillStyle = active ? "#00ffcc" : "#888888";
            let displayVal = typeof val === "boolean" ? (val ? "ACTIVADO" : "DESACTIVADO") : val;
            x.fillText(displayVal, CANVAS_WIDTH - 65, by + 26);
        }
    }
    x.restore();
    x.restore();
}

function drawHUD(){
    x.fillStyle = "#ffffff";
    x.font = "bold 16px 'Segoe UI', sans-serif";
    x.textAlign = "left";
    x.fillText("SCORE: " + score, 25, 35);
   
    x.textAlign = "right";
    x.fillStyle = difficulty === "dificil" ? "#ff4444" : "#00f2ff";
    x.fillText(difficulty.toUpperCase(), CANVAS_WIDTH - 25, 35);

    if(isAdminPro){
        x.fillStyle = "#00ffcc";
        x.font = "11px 'Segoe UI', sans-serif";
        x.textAlign = "left";
        x.fillText("💡 Presiona 'P' para abrir Panel Admin", 25, 55);
    }
}

function draw(){
    x.save();
    if(shakeTimer > 0){
        let shakeX = (Math.random() - 0.5) * 8;
        let shakeY = (Math.random() - 0.5) * 8;
        x.translate(shakeX, shakeY);
    }

    x.clearRect(0, 0, CANVAS_WIDTH, 700);

    drawBackground();

    if(gameState === "playing"){
        drawTrail();
        drawPlayerSkin();
        drawBlocks();
        drawLasers();
        drawParts();
        drawHUD();
    } else if(gameState === "menu"){
        drawMenu();
    } else if(gameState === "skinselect"){
        drawSkinSelect();
    } else if(gameState === "gameover"){
        drawTrail();
        drawPlayerSkin();
        drawBlocks();
        drawLasers();
        drawGameOver();
    }

    drawAdminPanel();
    x.restore();
}

initBgParticles();
requestAnimationFrame(update);
</script>
</body>
</html>
