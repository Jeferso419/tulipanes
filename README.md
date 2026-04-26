<!-- Dev de Corazon 
     El codigo de la galaxia de tulipanes morados
     -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Galaxia de Tulipanes Morados 3D</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Poppins', 'Segoe UI', 'Quicksand', cursive, sans-serif;
            color: #fff;
        }
        canvas { display: block; }

        .greeting {
            position: absolute;
            top: 30px;
            left: 0;
            right: 0;
            text-align: center;
            pointer-events: none;
            z-index: 20;
            font-size: 2.5rem;
            font-weight: 800;
            text-shadow: 0 0 20px rgba(0,0,0,0.5), 0 0 8px #cc66ff;
            background: linear-gradient(135deg, #e066ff, #aa44ff, #cc88ff);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            letter-spacing: -0.5px;
            animation: fadeInDown 1s ease-out;
        }
        .greeting small {
            font-size: 1.1rem;
            display: block;
            color: #dda0ff;
            background: none;
            -webkit-background-clip: unset;
            background-clip: unset;
            text-shadow: 0 0 10px rgba(180,0,255,0.5);
            margin-top: 5px;
            font-weight: 500;
        }
        @keyframes fadeInDown {
            from { opacity: 0; transform: translateY(-30px); }
            to   { opacity: 1; transform: translateY(0); }
        }
        .badge-control {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(60,0,80,0.55);
            backdrop-filter: blur(5px);
            border-radius: 40px;
            padding: 6px 16px;
            font-size: 0.7rem;
            pointer-events: none;
            font-weight: 500;
            color: #dda0ff;
            border: 1px solid #cc66ff80;
            z-index: 15;
        }
        .control-hint {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(40,0,60,0.55);
            backdrop-filter: blur(4px);
            padding: 5px 12px;
            border-radius: 30px;
            font-size: 0.7rem;
            pointer-events: none;
            white-space: nowrap;
            font-family: monospace;
            letter-spacing: 1px;
            color: #cc99ff;
            z-index: 15;
        }
        @media (max-width: 700px) {
            .greeting { font-size: 1.6rem; top: 20px; }
            .greeting small { font-size: 0.8rem; }
        }
    </style>
</head>
<body>

    <div class="greeting">
        🌷 Galaxia de Tulipanes Morados 🌷
        <small>✨ Un universo de pétalos para ti ✨</small>
    </div>

    <div class="badge-control">🌀 360° | Galaxia de Tulipanes</div>
    <div class="control-hint">🖱️ Arrastra para rotar 360° | Zoom con scroll</div>

    <script type="importmap">
    {
        "imports": {
            "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
            "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
        }
    }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
        import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
        import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';

        // --- Escena ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x04010f);   // negro violáceo profundo
        scene.fog = new THREE.FogExp2(0x06010f, 0.00045);

        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 2000);
        camera.position.set(13, 6, 18);
        camera.lookAt(0, 0, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.toneMapping = THREE.ReinhardToneMapping;
        renderer.toneMappingExposure = 1.3;
        document.body.appendChild(renderer.domElement);

        // --- Bloom ---
        const renderScene = new RenderPass(scene, camera);
        const bloomPass = new UnrealBloomPass(
            new THREE.Vector2(window.innerWidth, window.innerHeight),
            1.3, 0.45, 0.80
        );
        bloomPass.threshold = 0.07;
        bloomPass.strength  = 0.85;
        bloomPass.radius    = 0.65;
        const effectComposer = new EffectComposer(renderer);
        effectComposer.addPass(renderScene);
        effectComposer.addPass(bloomPass);

        // --- Controles ---
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping    = true;
        controls.dampingFactor    = 0.06;
        controls.rotateSpeed      = 1.2;
        controls.zoomSpeed        = 1.0;
        controls.enableZoom       = true;
        controls.autoRotate       = true;
        controls.autoRotateSpeed  = 0.65;
        controls.target.set(0, 0.5, 0);

        // =====================================================
        // 1. TEXTURAS DE TULIPÁN MORADO (partículas galaxia)
        // =====================================================
        function createTulipTexture(variant) {
            const canvas = document.createElement('canvas');
            canvas.width = 128; canvas.height = 128;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, 128, 128);
            const cx = 64, cy = 68;

            // Paleta de morados
            const purples = ['#9b30d0','#b44fe8','#7b2ab5','#c070f0','#6a1fa0'];
            const pCol = purples[variant % purples.length];
            const pCol2 = ['#d080ff','#e0a0ff','#a040cc','#bf66ee','#8833bb'][variant % 5];

            if (variant === 0) {
                // tulipán clásico: 6 pétalos en copa
                for (let i = 0; i < 6; i++) {
                    const angle = (i / 6) * Math.PI * 2;
                    const px = cx + Math.cos(angle) * 26;
                    const py = cy + Math.sin(angle) * 26;
                    const grad = ctx.createRadialGradient(px, py, 2, px, py, 28);
                    grad.addColorStop(0, pCol2);
                    grad.addColorStop(1, pCol);
                    ctx.beginPath();
                    ctx.ellipse(px, py, 18, 28, angle + Math.PI/2, 0, Math.PI*2);
                    ctx.fillStyle = grad;
                    ctx.fill();
                }
            } else if (variant === 1) {
                // tulipán alargado: 3 pétalos internos + 3 externos
                for (let i = 0; i < 3; i++) {
                    const angle = (i / 3) * Math.PI * 2;
                    ctx.beginPath();
                    ctx.ellipse(
                        cx + Math.cos(angle)*20, cy + Math.sin(angle)*20,
                        14, 30, angle + Math.PI/2, 0, Math.PI*2
                    );
                    ctx.fillStyle = pCol2;
                    ctx.fill();
                }
                for (let i = 0; i < 3; i++) {
                    const angle = (i / 3) * Math.PI * 2 + Math.PI/3;
                    ctx.beginPath();
                    ctx.ellipse(
                        cx + Math.cos(angle)*26, cy + Math.sin(angle)*26,
                        12, 26, angle + Math.PI/2, 0, Math.PI*2
                    );
                    ctx.fillStyle = pCol;
                    ctx.fill();
                }
            } else {
                // tulipán mini estrella con muchos pétalos
                for (let i = 0; i < 10; i++) {
                    const angle = (i / 10) * Math.PI * 2;
                    ctx.beginPath();
                    ctx.ellipse(
                        cx + Math.cos(angle)*28, cy + Math.sin(angle)*28,
                        11, 20, angle + Math.PI/2, 0, Math.PI*2
                    );
                    ctx.fillStyle = i%2===0 ? pCol : pCol2;
                    ctx.fill();
                }
            }

            // pistilo central
            const cGrad = ctx.createRadialGradient(cx, cy, 2, cx, cy, 13);
            cGrad.addColorStop(0, '#ffe0ff');
            cGrad.addColorStop(0.5, '#cc55ee');
            cGrad.addColorStop(1, '#6600aa');
            ctx.beginPath();
            ctx.arc(cx, cy, 13, 0, Math.PI*2);
            ctx.fillStyle = cGrad;
            ctx.fill();

            // brilitos del pistilo
            ctx.fillStyle = '#ffe8ff';
            for (let i = 0; i < 12; i++) {
                const r = Math.random() * 10;
                const a = Math.random() * Math.PI * 2;
                ctx.beginPath();
                ctx.arc(cx + Math.cos(a)*r, cy + Math.sin(a)*r, 1.2, 0, Math.PI*2);
                ctx.fill();
            }

            const tex = new THREE.CanvasTexture(canvas);
            tex.needsUpdate = true;
            return tex;
        }

        const tulipTextures = [
            createTulipTexture(0),
            createTulipTexture(1),
            createTulipTexture(2)
        ];

        // =====================================================
        // 2. PARTÍCULAS EN FORMA DE GALAXIA (tulipanes)
        // =====================================================
        const TOTAL_FLOWERS = 18500;
        const positions  = new Float32Array(TOTAL_FLOWERS * 3);
        const colors     = new Float32Array(TOTAL_FLOWERS * 3);
        const sizes      = new Float32Array(TOTAL_FLOWERS);
        const texIndices = new Float32Array(TOTAL_FLOWERS);

        const arms            = 4;
        const spiralTightness = 2.6;
        const armSpread       = 0.58;
        const innerRadius     = 1.0;
        const outerRadius     = 16.2;

        for (let i = 0; i < TOTAL_FLOWERS; i++) {
            let radius, angle, yOffset;
            const isCore = Math.random() < 0.2;

            if (isCore) {
                radius  = Math.pow(Math.random(), 1.3) * 3.8;
                angle   = Math.random() * Math.PI * 2;
                yOffset = (Math.random() - 0.5) * 1.9 * (1 - radius / 5);
            } else {
                const armIdx       = Math.floor(Math.random() * arms);
                const armAngleOff  = (armIdx / arms) * Math.PI * 2;
                const t            = Math.pow(Math.random(), 1.1);
                radius             = innerRadius + t * (outerRadius - innerRadius);
                const spiralAngle  = spiralTightness * Math.log(radius + 0.6) + armAngleOff;
                const randAngleOff = (Math.random() - 0.5) * armSpread * (1 + radius * 0.08);
                angle   = spiralAngle + randAngleOff;
                yOffset = Math.sin(radius * 1.4 + armIdx) * 0.85 + (Math.random() - 0.5) * 0.8;
                if (Math.random() > 0.7) yOffset += (Math.random() - 0.5) * 0.9;
            }

            positions[i*3]   = Math.cos(angle) * radius;
            positions[i*3+1] = yOffset * 1.3;
            positions[i*3+2] = Math.sin(angle) * radius;

            // Paleta morada para partículas
            const dist = Math.min(1, Math.sqrt(
                positions[i*3]**2 + positions[i*3+1]**2 + positions[i*3+2]**2
            ) / 14);

            // r, g, b en rango morado
            let r = 0.45 + Math.random() * 0.35;   // componente roja moderada
            let g = 0.0  + Math.random() * 0.15;   // poca verde
            let b = 0.65 + Math.random() * 0.35;   // mucha azul → morado

            if (dist < 0.3) { r = 0.6; g = 0.05; b = 0.9; }  // núcleo: morado brillante
            if (dist > 0.7) { r = 0.7; g = 0.2;  b = 1.0; }  // borde: lila

            colors[i*3]   = r;
            colors[i*3+1] = g;
            colors[i*3+2] = b;

            let sizeVal = 0.28 + Math.random() * 0.48;
            if (dist < 0.3) sizeVal *= 0.85;
            if (dist > 0.8) sizeVal *= 1.25;
            if (Math.random() > 0.93) sizeVal *= 1.55;
            sizes[i]      = sizeVal;
            texIndices[i] = Math.floor(Math.random() * tulipTextures.length);
        }

        const flowerGeometry = new THREE.BufferGeometry();
        flowerGeometry.setAttribute('position',  new THREE.BufferAttribute(positions, 3));
        flowerGeometry.setAttribute('color',     new THREE.BufferAttribute(colors, 3));
        flowerGeometry.setAttribute('size',      new THREE.BufferAttribute(sizes, 1));
        flowerGeometry.setAttribute('texIndex',  new THREE.BufferAttribute(texIndices, 1));

        const vertexShader = `
            attribute float size;
            attribute vec3 color;
            attribute float texIndex;
            varying vec3 vColor;
            varying float vTexIndex;
            void main() {
                vColor = color;
                vTexIndex = texIndex;
                vec4 mvPosition = modelViewMatrix * vec4(position, 1.0);
                gl_PointSize = size * ( 300.0 / ( - mvPosition.z ) ) * 0.95;
                gl_PointSize = clamp(gl_PointSize, 7.0, 48.0);
                gl_Position = projectionMatrix * mvPosition;
            }
        `;

        const fragmentShader = `
            uniform sampler2D uTextures[3];
            varying vec3 vColor;
            varying float vTexIndex;
            void main() {
                int idx = int(round(vTexIndex));
                vec4 texColor;
                if (idx == 0)      texColor = texture2D(uTextures[0], gl_PointCoord);
                else if (idx == 1) texColor = texture2D(uTextures[1], gl_PointCoord);
                else               texColor = texture2D(uTextures[2], gl_PointCoord);
                if (texColor.a < 0.2) discard;
                vec3 finalColor = vColor * texColor.rgb;
                finalColor += vec3(0.08, 0.02, 0.18);   // tinte morado oscuro en sombras
                gl_FragColor = vec4(finalColor, texColor.a * 0.95);
            }
        `;

        const flowerMaterial = new THREE.ShaderMaterial({
            uniforms:       { uTextures: { value: tulipTextures } },
            vertexShader,
            fragmentShader,
            transparent:    true,
            depthWrite:     false,
            blending:       THREE.AdditiveBlending
        });

        scene.add(new THREE.Points(flowerGeometry, flowerMaterial));

        // =====================================================
        // 3. TULIPANES 3D (objetos con geometría)
        // =====================================================
        function createTulip3D(x, z, yOff = 0, scale = 1.0) {
            const group = new THREE.Group();

            // Tallo verde
            const stemMat = new THREE.MeshStandardMaterial({ color: 0x3a8a3a, roughness: 0.6 });
            const stem    = new THREE.Mesh(
                new THREE.CylinderGeometry(0.10*scale, 0.16*scale, 1.2*scale, 6),
                stemMat
            );
            stem.position.y = -0.6 * scale;
            group.add(stem);

            // Hojas lanceoladas
            const leafMat = new THREE.MeshStandardMaterial({ color: 0x4db04d, side: THREE.DoubleSide });
            [-1, 1].forEach(side => {
                const leaf = new THREE.Mesh(
                    new THREE.ConeGeometry(0.18*scale, 0.6*scale, 5),
                    leafMat
                );
                leaf.position.set(side * 0.28*scale, -0.25*scale, 0);
                leaf.rotation.z = side * 0.55;
                leaf.rotation.x = 0.15;
                group.add(leaf);
            });

            // Copa del tulipán: 6 pétalos en forma de cucharita
            const petalColors = [0x8822cc, 0x9933dd, 0x7711bb, 0xaa44ee, 0x6600aa, 0xbb55ff];
            const petalMat = new THREE.MeshStandardMaterial({
                color: petalColors[Math.floor(Math.random() * petalColors.length)],
                emissive: 0x440088,
                emissiveIntensity: 0.3,
                roughness: 0.4,
                side: THREE.DoubleSide
            });

            // 6 pétalos externos (copa cerrada característica del tulipán)
            for (let i = 0; i < 6; i++) {
                const angle = (i / 6) * Math.PI * 2;
                const petal = new THREE.Mesh(
                    new THREE.CylinderGeometry(0.05*scale, 0.22*scale, 0.7*scale, 5),
                    petalMat
                );
                // posición: inclinado hacia afuera en la base, cerrándose arriba
                petal.position.set(
                    Math.cos(angle) * 0.28*scale,
                    0.22*scale,
                    Math.sin(angle) * 0.28*scale
                );
                petal.rotation.z  = angle + Math.PI / 2;
                petal.rotation.x  = Math.cos(angle) * 0.35;
                group.add(petal);
            }

            // Pistilo / estambre
            const pistilMat = new THREE.MeshStandardMaterial({ color: 0xffccff, emissive: 0xcc66ff, emissiveIntensity: 0.5 });
            const pistil = new THREE.Mesh(
                new THREE.SphereGeometry(0.10*scale, 8, 8),
                pistilMat
            );
            pistil.position.y = 0.55 * scale;
            group.add(pistil);

            group.position.set(x, yOff, z);
            group.rotation.y = Math.random() * Math.PI * 2;
            return group;
        }

        const tulipPositions = [];

        // Núcleo central
        for (let i = 0; i < 8; i++) {
            const angle = (i / 8) * Math.PI * 2;
            const r = 1.8 + Math.random() * 1.2;
            tulipPositions.push({ x: Math.cos(angle)*r, z: Math.sin(angle)*r, y: -0.2 + Math.random()*0.5, scale: 0.9 + Math.random()*0.3 });
        }

        // Brazos espirales
        const armAngles = [0, Math.PI/2, Math.PI, 3*Math.PI/2];
        for (let a = 0; a < armAngles.length; a++) {
            for (let r = 3.5; r <= 12; r += 2.2) {
                const base  = spiralTightness * Math.log(r + 0.8);
                const angle = base + armAngles[a] + (Math.random()-0.5) * 0.8;
                const x     = Math.cos(angle) * r;
                const z     = Math.sin(angle) * r;
                const y     = Math.sin(r * 1.2) * 0.4 + (Math.random()-0.5) * 0.4;
                tulipPositions.push({ x, z, y: y - 0.1, scale: 0.8 + Math.random()*0.5 });
            }
        }

        // Flores adicionales esparcidas
        for (let i = 0; i < 12; i++) {
            const rad = 6 + Math.random() * 7;
            const ang = Math.random() * Math.PI * 2;
            tulipPositions.push({
                x: Math.cos(ang)*rad, z: Math.sin(ang)*rad,
                y: (Math.random()-0.5)*1.2,
                scale: 0.7 + Math.random()*0.6
            });
        }

        const tulipsGroup = new THREE.Group();
        tulipPositions.forEach(p => tulipsGroup.add(createTulip3D(p.x, p.z, p.y, p.scale)));
        scene.add(tulipsGroup);

        // Partículas de polen — ahora color lila
        const pollenCount = 1800;
        const pollenPos   = new Float32Array(pollenCount * 3);
        for (let i = 0; i < pollenCount; i++) {
            const base = tulipPositions[Math.floor(Math.random() * tulipPositions.length)];
            pollenPos[i*3]   = base.x + (Math.random()-0.5)*1.2;
            pollenPos[i*3+1] = base.y + (Math.random()-0.5)*1.0 + 0.2;
            pollenPos[i*3+2] = base.z + (Math.random()-0.5)*1.2;
        }
        const pollenGeo = new THREE.BufferGeometry();
        pollenGeo.setAttribute('position', new THREE.BufferAttribute(pollenPos, 3));
        const pollenMat = new THREE.PointsMaterial({
            color: 0xcc88ff, size: 0.05,
            transparent: true, blending: THREE.AdditiveBlending
        });
        scene.add(new THREE.Points(pollenGeo, pollenMat));

        // =====================================================
        // 4. ESTRELLAS DE FONDO — tono morado/azul
        // =====================================================
        const starCount = 4500;
        const starPos   = new Float32Array(starCount * 3);
        const starCol   = new Float32Array(starCount * 3);
        for (let i = 0; i < starCount; i++) {
            const rad   = 40 + Math.random() * 55;
            const theta = Math.random() * Math.PI * 2;
            const phi   = Math.acos(2 * Math.random() - 1);
            starPos[i*3]   = rad * Math.sin(phi) * Math.cos(theta);
            starPos[i*3+1] = rad * Math.sin(phi) * Math.sin(theta) * 0.5;
            starPos[i*3+2] = rad * Math.cos(phi);
            const k = 0.4 + Math.random() * 0.8;
            starCol[i*3]   = k * (Math.random() > 0.5 ? 0.7 : 0.4);   // rojo bajo
            starCol[i*3+1] = k * 0.2;                                   // verde casi nulo
            starCol[i*3+2] = k * (0.8 + Math.random() * 0.2);          // azul alto → violeta
        }
        const starGeo = new THREE.BufferGeometry();
        starGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
        starGeo.setAttribute('color',    new THREE.BufferAttribute(starCol, 3));
        scene.add(new THREE.Points(starGeo, new THREE.PointsMaterial({
            size: 0.16, vertexColors: true,
            transparent: true, opacity: 0.75,
            blending: THREE.AdditiveBlending
        })));

        // Polvo estelar morado
        const dustCount = 8000;
        const dustPos   = new Float32Array(dustCount * 3);
        for (let i = 0; i < dustCount; i++) {
            const r   = 5 + Math.random() * 14;
            const ang = Math.random() * Math.PI * 2;
            const y   = (Math.random()-0.5) * 4.5;
            dustPos[i*3]   = Math.cos(ang) * r * (0.6 + Math.random()*0.7);
            dustPos[i*3+1] = y + (Math.random()-0.5)*1.5;
            dustPos[i*3+2] = Math.sin(ang) * r * (0.6 + Math.random()*0.7);
        }
        const dustGeo = new THREE.BufferGeometry();
        dustGeo.setAttribute('position', new THREE.BufferAttribute(dustPos, 3));
        const dustCloud = new THREE.Points(dustGeo, new THREE.PointsMaterial({
            color: 0xaa44ff, size: 0.055,
            transparent: true, opacity: 0.4,
            blending: THREE.AdditiveBlending
        }));
        scene.add(dustCloud);

        // =====================================================
        // 5. NÚCLEO Y HALO — morado intenso
        // =====================================================
        const coreGlow = new THREE.Mesh(
            new THREE.SphereGeometry(1.4, 32, 32),
            new THREE.MeshStandardMaterial({
                color: 0xaa00ff,
                emissive: 0x7700cc,
                emissiveIntensity: 1.8,
                roughness: 0.2
            })
        );
        scene.add(coreGlow);

        const haloCount = 1500;
        const haloPos   = new Float32Array(haloCount * 3);
        for (let i = 0; i < haloCount; i++) {
            const r     = 1.8 + Math.random() * 2.2;
            const theta = Math.random() * Math.PI * 2;
            const phi   = Math.acos(2 * Math.random() - 1);
            haloPos[i*3]   = r * Math.sin(phi) * Math.cos(theta);
            haloPos[i*3+1] = r * Math.sin(phi) * Math.sin(theta) * 0.7;
            haloPos[i*3+2] = r * Math.cos(phi);
        }
        const haloGeo = new THREE.BufferGeometry();
        haloGeo.setAttribute('position', new THREE.BufferAttribute(haloPos, 3));
        const haloRing = new THREE.Points(haloGeo, new THREE.PointsMaterial({
            color: 0xcc66ff, size: 0.09,
            blending: THREE.AdditiveBlending
        }));
        scene.add(haloRing);

        // Luces — todas violáceas
        scene.add(new THREE.AmbientLight(0x110022));
        const pLight = new THREE.PointLight(0xaa44ff, 1.3, 30);
        pLight.position.set(2, 1.5, 2);
        scene.add(pLight);
        const bLight = new THREE.PointLight(0x8800cc, 0.8);
        bLight.position.set(-3, 2, -5);
        scene.add(bLight);
        const fLight = new THREE.PointLight(0xcc88ff, 0.6);
        fLight.position.set(1, 3, 3);
        scene.add(fLight);

        // =====================================================
        // 6. ANIMACIÓN
        // =====================================================
        let time = 0;
        function animate() {
            requestAnimationFrame(animate);
            time += 0.008;

            // Núcleo pulsante
            coreGlow.material.emissiveIntensity = 1.5 + Math.sin(time * 2.5) * 0.5;
            pLight.intensity = 1.0 + Math.sin(time * 2.2) * 0.3;

            dustCloud.rotation.y  = time * 0.03;
            haloRing.rotation.y   = time * 0.1;
            haloRing.rotation.x   = Math.sin(time * 0.2) * 0.1;

            // Balanceo suave de tulipanes
            tulipsGroup.children.forEach((tulip, idx) => {
                tulip.rotation.z = Math.sin(time * 1.1 + idx) * 0.035;
                tulip.rotation.x = Math.cos(time * 0.9 + idx) * 0.02;
            });

            controls.update();
            effectComposer.render();
        }
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            effectComposer.setSize(window.innerWidth, window.innerHeight);
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        console.log('🌷 Galaxia de Tulipanes Morados - lista');
    </script>
</body>
</html>
