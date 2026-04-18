<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Hand Tracking AR</title>
    
    <!-- MediaPipe Hands CDN -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>

    <style>
        :root {
            --bg-color: #050510;
            --text-color: #ffffff;
            --glass-bg: rgba(20, 25, 40, 0.4);
            --glass-border: rgba(255, 255, 255, 0.15);
            --glass-blur: blur(16px);
            --accent: #00ffcc;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            overflow: hidden;
        }

        .video-container {
            position: absolute;
            width: 100vw;
            height: 100vh;
        }

        .input_video {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1); /* Mirror */
            z-index: 0; /* Let the raw camera sit behind */
            filter: brightness(0.6) contrast(1.1); /* Subtle dark to make neon pop */
        }

        /* 
         * Background Canvas captures the video frame and applies the matrix rain/starfield 
         */
        #bgCanvas, #mainCanvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1); /* Mirror */
        }
        
        #bgCanvas {
            z-index: 1;
            pointer-events: none;
        }

        #mainCanvas {
            z-index: 5;
            pointer-events: none;
            /* We use mix-blend-mode for the glowing effects in canvas */
        }

        /* UI Overlays */
        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 10;
            display: flex;
            flex-direction: column;
            gap: 15px;
            pointer-events: none;
        }

        .panel {
            background: var(--glass-bg);
            backdrop-filter: var(--glass-blur);
            -webkit-backdrop-filter: var(--glass-blur);
            border: 1px solid var(--glass-border);
            border-radius: 12px;
            padding: 15px 20px;
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
        }

        .stat {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            font-size: 0.9rem;
            color: rgba(255, 255, 255, 0.8);
            min-width: 180px;
        }
        .stat:last-child { margin-bottom: 0; }
        .stat span.val { font-weight: bold; color: var(--accent); }

        /* Themes */
        #themes {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
            display: flex;
            gap: 10px;
            background: var(--glass-bg);
            backdrop-filter: var(--glass-blur);
            border: 1px solid var(--glass-border);
            border-radius: 30px;
            padding: 8px;
            transition: opacity 0.5s;
        }

        .theme-btn {
            background: transparent;
            border: 1px solid transparent;
            color: white;
            padding: 8px 16px;
            border-radius: 20px;
            cursor: pointer;
            transition: 0.3s ease;
            font-weight: 500;
        }
        
        .theme-btn.active {
            background: rgba(255,255,255,0.15);
            border-color: rgba(255,255,255,0.3);
            text-shadow: 0 0 10px white;
        }

        /* Start Overlay */
        #startOverlay {
            position: absolute;
            inset: 0;
            background: rgba(0,0,0,0.85);
            z-index: 100;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            backdrop-filter: blur(10px);
            transition: opacity 0.5s ease;
        }

        .start-btn {
            padding: 16px 40px;
            font-size: 1.2rem;
            background: linear-gradient(45deg, #ff00cc, #3333ff);
            border: none;
            color: white;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(255, 0, 204, 0.5);
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            transition: transform 0.2s;
        }
        .start-btn:hover { transform: scale(1.05); }

        .hidden { opacity: 0; pointer-events: none; }
    </style>
</head>
<body>

    <!-- Start Overlay (Required for AudioContext initialization) -->
    <div id="startOverlay">
        <h1 style="margin-bottom: 20px; font-weight: 300; letter-spacing: 4px;">NEON AURA AR</h1>
        <p style="margin-bottom: 40px; color: #888;">Grant camera permissions and click to start audio/visual experience</p>
        <button class="start-btn" id="startBtn">Enter Experience</button>
    </div>

    <div class="video-container">
        <video class="input_video" autoplay playsinline></video>
        <canvas id="bgCanvas"></canvas>
        <canvas id="mainCanvas"></canvas>
    </div>

    <!-- UI Overlay -->
    <div id="hud" class="hidden">
        <div class="panel">
            <div class="stat">Hands Detected: <span class="val" id="ui-hands">0</span></div>
            <div class="stat">FPS: <span class="val" id="ui-fps">0</span></div>
        </div>
        <div class="panel">
            <div class="stat">Gesture: <span class="val" id="ui-gesture">None</span></div>
            <div class="stat">Spread: <span class="val" id="ui-spread">0%</span></div>
        </div>
    </div>

    <div id="themes" class="hidden">
        <button class="theme-btn active" data-theme="Rainbow">Rainbow</button>
        <button class="theme-btn" data-theme="Cyberpunk">Cyberpunk</button>
        <button class="theme-btn" data-theme="Lava">Lava</button>
        <button class="theme-btn" data-theme="Ocean">Ocean</button>
        <button class="theme-btn" data-theme="Galaxy">Galaxy</button>
    </div>

<script>
/** 
 * GLOBALS & CONFIG
 */
const videoElement = document.querySelector('.input_video');
const bgCanvas = document.getElementById('bgCanvas');
const mainCanvas = document.getElementById('mainCanvas');
const bgCtx = bgCanvas.getContext('2d');
const ctx = mainCanvas.getContext('2d');

let width = window.innerWidth;
let height = window.innerHeight;

let time = 0;
let lastTime = performance.now();
let framesThisSecond = 0;
let lastFpsTime = performance.now();

let currentHands = []; // Latest data from MediaPipe
let handVelocities = 0; // Average hand movement speed

// Theme Config
let currentTheme = 'Rainbow';
const themes = {
    'Rainbow':   (t, index, total) => `hsl(${(t * 100 + index * (360/total)) % 360}, 100%, 60%)`,
    'Cyberpunk': (t, index, total) => (index % 2 === 0) ? '#ff003c' : '#00f0ff',
    'Lava':      (t, index, total) => `hsl(${(10 + (index * 10)) % 40}, 100%, ${50 + Math.sin(t)*10}%)`,
    'Ocean':     (t, index, total) => `hsl(${180 + (index * 20)}, 100%, 60%)`,
    'Galaxy':    (t, index, total) => `hsl(${260 + Math.sin(t*2 + index)*40}, 100%, 65%)`
};

// Physics Engines Data
let particles = [];
let ripples = [];
const FINGER_TIPS = [4, 8, 12, 16, 20];

// Matrix Background
let matrixColumns = [];
const fontSize = 16;
let maxColumns = 0;

// Audio Node References
let audioCtx = null;
let humOsc = null;
let humGain = null;

// UI Elements
const uiHands = document.getElementById('ui-hands');
const uiFps = document.getElementById('ui-fps');
const uiGesture = document.getElementById('ui-gesture');
const uiSpread = document.getElementById('ui-spread');

/**
 * INITIALIZATION
 */
function resize() {
    width = window.innerWidth;
    height = window.innerHeight;
    bgCanvas.width = width;
    bgCanvas.height = height;
    mainCanvas.width = width;
    mainCanvas.height = height;
    
    maxColumns = Math.floor(width / fontSize);
    matrixColumns = new Array(maxColumns).fill(1).map(() => Math.random() * height/fontSize);
}
window.addEventListener('resize', resize);
resize();

// UI Theme Switcher
document.querySelectorAll('.theme-btn').forEach(btn => {
    btn.addEventListener('click', (e) => {
        document.querySelectorAll('.theme-btn').forEach(b => b.classList.remove('active'));
        e.target.classList.add('active');
        currentTheme = e.target.getAttribute('data-theme');
        document.documentElement.style.setProperty('--accent', themes[currentTheme](0, 1, 1));
    });
});

// Start button triggers AudioContext and hides overlay
document.getElementById('startBtn').addEventListener('click', () => {
    document.getElementById('startOverlay').classList.add('hidden');
    document.getElementById('hud').classList.remove('hidden');
    document.getElementById('themes').classList.remove('hidden');
    initAudio();
    initMediaPipe();
    requestAnimationFrame(renderLoop);
});


/**
 * AUDIO ENGINE
 */
function initAudio() {
    try {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        // Continuous Hum
        humOsc = audioCtx.createOscillator();
        humGain = audioCtx.createGain();
        
        humOsc.type = 'sine';
        humOsc.frequency.value = 100;
        
        humGain.gain.value = 0; // Mute until hands are seen
        
        humOsc.connect(humGain);
        humGain.connect(audioCtx.destination);
        humOsc.start();
    } catch(e) {
        console.error("Web Audio API failed", e);
    }
}

function triggerZap() {
    if (!audioCtx) return;
    const osc = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();
    
    // Zap sound profile
    osc.type = 'sawtooth';
    osc.frequency.setValueAtTime(800, audioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(40, audioCtx.currentTime + 0.1);
    
    gainNode.gain.setValueAtTime(0.5, audioCtx.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.1);
    
    osc.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    osc.start();
    osc.stop(audioCtx.currentTime + 0.15);
}

function updateHum(activeHands) {
    if (!audioCtx || !humGain) return;
    if (activeHands.length < 2) {
        humGain.gain.setTargetAtTime(0, audioCtx.currentTime, 0.1);
        return;
    }

    // Measure distance between index fingers to modulate volume
    const p1 = activeHands[0][8];
    const p2 = activeHands[1][8];
    const dx = p1.x - p2.x;
    const dy = p1.y - p2.y;
    const dist = Math.sqrt(dx*dx + dy*dy);
    
    // The closer they are, the higher the pitch and volume
    const targetFreq = 100 + (1 - Math.min(dist, 1)) * 300;
    const targetVolume = 0.05 + (1 - Math.min(dist, 1)) * 0.15;
    
    humOsc.frequency.setTargetAtTime(targetFreq, audioCtx.currentTime, 0.1);
    humGain.gain.setTargetAtTime(targetVolume, audioCtx.currentTime, 0.1);
}

/**
 * MATH & STATE LOGIC
 */
function getDist(p1, p2) {
    return Math.hypot(p1.x - p2.x, p1.y - p2.y);
}

// Convert normalized landmark to specific canvas scale (Note: canvas is horizontally flipped)
function mapToCanvas(point) {
    return { x: point.x * width, y: point.y * height };
}

let lastPinchState = [false, false]; // Prevents rapid re-triggering

function detectGestures() {
    if (!currentHands.length) return;
    
    currentHands.forEach((hand, idx) => {
        // Pinch Detection: Thumb (4) and Index (8)
        const thumb = hand[4];
        const index = hand[8];
        const dist = getDist(thumb, index);
        
        const isPinching = dist < 0.05; // 5% of screen screen distance
        
        if (isPinching && !lastPinchState[idx]) {
            const midpoint = {
                x: (thumb.x + index.x) / 2, 
                y: (thumb.y + index.y) / 2
            };
            createShockwave(mapToCanvas(midpoint), themes[currentTheme](time, 1, 1));
            triggerZap();
            uiGesture.innerText = "PINCH !";
        }
        lastPinchState[idx] = isPinching;
    });

    // Spread Percentage roughly estimated by distance from Palm(0) to Index(8) and Pinky(20)
    if (currentHands[0]) {
        const spread = getDist(currentHands[0][8], currentHands[0][20]);
        // Normalizing spread so max is around 100%
        let spreadPct = Math.min(Math.round(spread * 300), 100);
        uiSpread.innerText = spreadPct + '%';
        if (!lastPinchState.includes(true)) {
            uiGesture.innerText = spreadPct > 50 ? "Open Hand" : "Fist";
        }
    }
}

/**
 * EFFECTS & PHYSICS
 */
function createParticles(pos, color, count = 3) {
    for (let i=0; i<count; i++) {
        particles.push({
            x: pos.x,
            y: pos.y,
            vx: (Math.random() - 0.5) * 8,
            vy: (Math.random() - 0.5) * 8,
            life: 1.0,
            color: color,
            size: Math.random() * 3 + 1
        });
    }
}

function createShockwave(pos, color) {
    ripples.push({
        x: pos.x,
        y: pos.y,
        radius: 0,
        maxRadius: 150 + Math.random() * 100,
        life: 1.0,
        color: color
    });
}

// Background Effect Engine
function drawBackground() {
    // Use destination-out to fade out the previous frame's drops, leaving a transparent trail
    bgCtx.globalCompositeOperation = 'destination-out';
    bgCtx.fillStyle = `rgba(0, 0, 0, ${0.15 + Math.min(handVelocities*10, 0.5)})`;
    bgCtx.fillRect(0, 0, width, height);
    bgCtx.globalCompositeOperation = 'source-over';

    // Matrix Rain Effect mapping to hand speed
    bgCtx.fillStyle = themes[currentTheme](time, 1, 1);
    bgCtx.font = fontSize + "px monospace";
    
    // Matrix speed boosts when hands move fast
    let speedMult = 1 + (handVelocities * 100);

    for (let i = 0; i < matrixColumns.length; i++) {
        // Only draw randomly to keep it sparse like stars/rain
        if (Math.random() > 0.95) {
            const char = String.fromCharCode(0x30A0 + Math.random() * 96);
            bgCtx.fillText(char, i * fontSize, matrixColumns[i] * fontSize);
        }
        
        matrixColumns[i] += Math.random() * speedMult;
        
        if (matrixColumns[i] * fontSize > height && Math.random() > 0.9) {
            matrixColumns[i] = 0;
        }
    }
}

function updatePhysics() {
    // Particles
    for (let i = particles.length - 1; i >= 0; i--) {
        let p = particles[i];
        p.x += p.vx;
        p.y += p.vy;
        p.life -= 0.02;     // Fade out
        p.vy += 0.1;        // Gravity
        
        if (p.life <= 0) {
            particles.splice(i, 1);
        } else {
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
            ctx.fillStyle = p.color;
            ctx.globalAlpha = p.life;
            ctx.fill();
        }
    }
    
    // Ripples / Shockwaves
    for (let i = ripples.length - 1; i >= 0; i--) {
        let r = ripples[i];
        r.radius += (r.maxRadius - r.radius) * 0.1; // Ease out
        r.life -= 0.03;
        
        if (r.life <= 0) {
            ripples.splice(i, 1);
        } else {
            ctx.beginPath();
            ctx.arc(r.x, r.y, r.radius, 0, Math.PI * 2);
            ctx.strokeStyle = r.color;
            ctx.lineWidth = 4 * r.life;
            ctx.globalAlpha = r.life;
            ctx.stroke();
        }
    }
    ctx.globalAlpha = 1.0; // Reset
}

/**
 * MAIN RENDER PIPELINE
 */
function renderLoop(timestamp) {
    requestAnimationFrame(renderLoop);
    
    let dt = (timestamp - lastTime) / 1000;
    lastTime = timestamp;
    time += dt;

    // Update FPS Counter
    framesThisSecond++;
    if (timestamp > lastFpsTime + 1000) {
        uiFps.innerText = framesThisSecond;
        framesThisSecond = 0;
        lastFpsTime = timestamp;
    }

    drawBackground();

    // The main canvas will clear fully each frame since we handle ghosting via bgCanvas 
    // BUT user requested trailing motion blur for fingertips.
    // Instead of clearRect, we fade the main canvas using destination-out to keep it transparent
    ctx.globalCompositeOperation = 'destination-out';
    ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
    ctx.fillRect(0, 0, width, height);

    // Enable Screen mode for additive light effects (like neon bloom)
    ctx.globalCompositeOperation = 'screen'; // Creates glowy overlapping effects

    // Render Physics layer
    updatePhysics();

    // Process Hand Logic if present
    if (currentHands.length > 0) {
        
        // 1. Draw Skeleton
        currentHands.forEach((hand, handIndex) => {
            const glowColor = themes[currentTheme](time, handIndex, 2);
            
            // Draw MediaPipe skeleton connectors using custom styles
            drawConnectors(ctx, hand, HAND_CONNECTIONS, {
                color: glowColor,
                lineWidth: 2
            });
            
            // Draw Landmarks with neon bloom
            ctx.shadowBlur = 15;
            ctx.shadowColor = glowColor;
            
            // Only draw fingertips with intense bloom and spawn particles
            FINGER_TIPS.forEach((tipIndex, idx) => {
                const pt = mapToCanvas(hand[tipIndex]);
                const tipCol = themes[currentTheme](time, idx, FINGER_TIPS.length);
                
                ctx.beginPath();
                ctx.arc(pt.x, pt.y, 4, 0, Math.PI * 2);
                ctx.fillStyle = '#fff';
                ctx.fill();

                // Generate constant spark particles at fingertips
                if (Math.random() > 0.6) {
                    createParticles(pt, tipCol, 1);
                }
            });
            ctx.shadowBlur = 0; // Reset
        });

        // 2. Cross-Hand Interactions (Lightning & Gradients)
        if (currentHands.length >= 2) {
            const h1 = currentHands[0];
            const h2 = currentHands[1];

            // A. Rainbow Connecting Lines
            FINGER_TIPS.forEach((tipIndex, idx) => {
                const pt1 = mapToCanvas(h1[tipIndex]);
                const pt2 = mapToCanvas(h2[tipIndex]);
                const dist = getDist(pt1, pt2);
                
                const col = themes[currentTheme](time, idx, FINGER_TIPS.length);
                
                // Lightning electric arc when very close (but not touching)
                if (dist < 150 && Math.random() > 0.5) {
                    // Draw jagged lightning
                    ctx.beginPath();
                    ctx.moveTo(pt1.x, pt1.y);
                    // Midpoint jitter
                    const midX = (pt1.x + pt2.x)/2 + (Math.random() - 0.5) * 50;
                    const midY = (pt1.y + pt2.y)/2 + (Math.random() - 0.5) * 50;
                    ctx.lineTo(midX, midY);
                    ctx.lineTo(pt2.x, pt2.y);
                    
                    ctx.strokeStyle = '#ffffff';
                    ctx.shadowBlur = 20;
                    ctx.shadowColor = col;
                    ctx.lineWidth = 3;
                    ctx.stroke();
                }

                // Normal flowing gradient line
                ctx.beginPath();
                ctx.moveTo(pt1.x, pt1.y);
                ctx.lineTo(pt2.x, pt2.y);
                
                // Create gradient that shifts over time
                let grad = ctx.createLinearGradient(pt1.x, pt1.y, pt2.x, pt2.y);
                grad.addColorStop(0, themes[currentTheme](time, idx, 5));
                grad.addColorStop(0.5, themes[currentTheme](time, idx + 1, 5));
                grad.addColorStop(1, themes[currentTheme](time, idx + 2, 5));
                
                ctx.strokeStyle = grad;
                ctx.lineWidth = 4;
                ctx.shadowBlur = 10;
                ctx.shadowColor = col;
                ctx.stroke();
                ctx.shadowBlur = 0;
            });

            // B. Mandala drawing if 10 tips are perfectly detected
            // (Assuming if we have 2 hands, we draw lines connecting all tips in a star)
            if (h1 && h2) {
                // Combine all 10 tips
                let allTips = FINGER_TIPS.map(t => mapToCanvas(h1[t])).concat(
                              FINGER_TIPS.map(t => mapToCanvas(h2[t])) );
                
                ctx.save();
                // Find center point to draw Mandala
                let cx = allTips.reduce((sum, p) => sum + p.x, 0) / 10;
                let cy = allTips.reduce((sum, p) => sum + p.y, 0) / 10;
                
                ctx.translate(cx, cy);
                ctx.rotate(time * 0.5); // Slow rotation
                
                ctx.beginPath();
                for (let i=0; i<10; i++) {
                    const t1 = { x: allTips[i].x - cx, y: allTips[i].y - cy };
                    const t2 = { x: allTips[(i+3)%10].x - cx, y: allTips[(i+3)%10].y - cy };
                    ctx.moveTo(t1.x, t1.y);
                    ctx.lineTo(t2.x, t2.y);
                }
                ctx.strokeStyle = `rgba(255, 255, 255, 0.2)`;
                ctx.lineWidth = 1;
                ctx.stroke();
                ctx.restore();
            }
        }
        
        detectGestures();
    }
    
    ctx.globalCompositeOperation = 'source-over'; // Restore
}

/**
 * MEDIAPIPE INITIALIZATION
 */
function initMediaPipe() {
    const hands = new Hands({locateFile: (file) => {
        return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;
    }});

    hands.setOptions({
        maxNumHands: 2,
        modelComplexity: 1, 
        minDetectionConfidence: 0.7,
        minTrackingConfidence: 0.7
    });

    hands.onResults((results) => {
        if (!audioCtx) return; // Wait for initialization

        // Update global state for render loop to read from
        uiHands.innerText = results.multiHandLandmarks ? results.multiHandLandmarks.length : 0;
        
        // Calculate velocity (rudimentary)
        if (currentHands.length > 0 && results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
            let vSum = 0;
            // check distance difference on index finger of hand 0 
            const oldP = currentHands[0][8];
            const newP = results.multiHandLandmarks[0][8];
            if (oldP && newP) {
                vSum += getDist(oldP, newP);
                handVelocities = vSum; 
            }
        } else {
            handVelocities = 0;
        }

        currentHands = results.multiHandLandmarks || [];
        updateHum(currentHands);
    });

    const camera = new Camera(videoElement, {
        onFrame: async () => {
            await hands.send({image: videoElement});
        },
        width: 1280,
        height: 720,
        facingMode: 'user'
    });
    
    camera.start();
}

</script>
</body>
</html>
