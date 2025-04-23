<!DOCTYPE html>
<html lang="en">
<head>
    
    <title>Galaxy Generator</title>
    <style>
        body { margin: 0; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
    <script src="https://unpkg.com/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/dat-gui/0.7.9/dat.gui.min.js"></script>

    <script>
        // Scene setup
        const scene = new THREE.Scene();
        const canvas = document.querySelector('canvas.webgl');
        const sizes = { width: window.innerWidth, height: window.innerHeight };

        // Camera
        const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100);
        camera.position.set(3, 3, 3);
        scene.add(camera);

        // Renderer
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setClearColor(0x000000)
        renderer.setSize(sizes.width, sizes.height);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        document.body.appendChild(renderer.domElement);

        // Controls
        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;

        // Galaxy parameters
        const parameters = {
            count: 300000,
            size: 0.01,
            radius: 5,
            branches: 5,
            spin: 1,
            randomness: 0.9,
            randomnessPower: 3,
            insideColor: '#ff6030',
            outsideColor: '#1b3984',
            rotationSpeed: 0.3,
            pulseSpeed: 1
        };

        // GUI
        let gui = new dat.gui.GUI();
        gui.closed = true
        gui.add(parameters, 'count', 1000, 1000000, 100).onFinishChange(generateGalaxy);
        gui.add(parameters, 'size', 0.001, 0.1, 0.001);
        gui.add(parameters, 'radius', 0.01, 20, 0.01);
        gui.add(parameters, 'branches', 2, 20, 1);
        gui.add(parameters, 'spin', -5, 5, 0.01);
        gui.add(parameters, 'randomness', 0, 2, 0.001);
        gui.add(parameters, 'randomnessPower', 1, 10, 0.1);
        gui.addColor(parameters, 'insideColor').onFinishChange(generateGalaxy);
        gui.addColor(parameters, 'outsideColor').onFinishChange(generateGalaxy);
        gui.add(parameters, 'rotationSpeed', 0, 2, 0.01);
        gui.add(parameters, 'pulseSpeed', 0, 5, 0.01);

        let galaxyGeometry = null;
        let galaxyMaterial = null;
        let galaxy = null;
        let colorInside = new THREE.Color(parameters.insideColor);
        let colorOutside = new THREE.Color(parameters.outsideColor);

        // Galaxy generation
        function generateGalaxy() {
            if (galaxy) {
                galaxyGeometry.dispose();
                galaxyMaterial.dispose();
                scene.remove(galaxy);
            }

            galaxyGeometry = new THREE.BufferGeometry();
            const positions = new Float32Array(parameters.count * 3);
            const colors = new Float32Array(parameters.count * 3);

            colorInside = new THREE.Color(parameters.insideColor);
            colorOutside = new THREE.Color(parameters.outsideColor);

            for (let i = 0; i < parameters.count; i++) {
                const i3 = i * 3;
                const radius = Math.random() * parameters.radius;
                const spinAngle = radius * parameters.spin;
                const branchAngle = (i % parameters.branches) / parameters.branches * Math.PI * 2;
                
                const randomX = Math.pow(Math.random(), parameters.randomnessPower) * parameters.randomness * (Math.random() < 0.5 ? 1 : -1);
                const randomY = Math.pow(Math.random(), parameters.randomnessPower) * parameters.randomness * (Math.random() < 0.5 ? 1 : -1);
                const randomZ = Math.pow(Math.random(), parameters.randomnessPower) * parameters.randomness * (Math.random() < 0.5 ? 1 : -1);

                positions[i3] = Math.cos(branchAngle + spinAngle) * radius + randomX;
                positions[i3 + 1] = randomY;
                positions[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius + randomZ;

                const mixedColor = colorInside.clone().lerp(colorOutside, radius / parameters.radius);
                colors[i3] = mixedColor.r;
                colors[i3 + 1] = mixedColor.g;
                colors[i3 + 2] = mixedColor.b;
            }

            galaxyGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            galaxyGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

            galaxyMaterial = new THREE.PointsMaterial({
                size: parameters.size,
                sizeAttenuation: true,
                depthWrite: false,
                blending: THREE.AdditiveBlending,
                vertexColors: true
            });

            galaxy = new THREE.Points(galaxyGeometry, galaxyMaterial);
            galaxy.rotation.x = Math.PI * 0.5;
            scene.add(galaxy);
        }

        // Ambient stars
        const starGeometry = new THREE.BufferGeometry();
        const starMaterial = new THREE.PointsMaterial({ color: 0xffffff, size: 0.02 });
        const starPositions = new Float32Array(1000 * 3);

        for (let i = 0; i < 1000; i++) {
            starPositions[i * 3] = (Math.random() - 0.5) * 50;
            starPositions[i * 3 + 1] = (Math.random() - 0.5) * 50;
            starPositions[i * 3 + 2] = (Math.random() - 0.5) * 50;
        }

        starGeometry.setAttribute('position', new THREE.BufferAttribute(starPositions, 3));
        const stars = new THREE.Points(starGeometry, starMaterial);
        scene.add(stars);

        // Animation
        const clock = new THREE.Clock();
        let elapsedTime = 0;

        const createdBySword = () => {
            elapsedTime = clock.getElapsedTime();
            
            // Galaxy animation
            if (galaxy) {
                galaxy.rotation.y = elapsedTime * parameters.rotationSpeed;
                const scale = Math.sin(elapsedTime * parameters.pulseSpeed) * 0.1 + 1;
                galaxy.scale.set(scale, scale, scale);
            }

            // Update controls
            controls.update();
            
            renderer.render(scene, camera);
            requestAnimationFrame(createdBySword);
        };

        
        generateGalaxy();

        
        window.addEventListener('resize', () => {
            sizes.width = window.innerWidth;
            sizes.height = window.innerHeight;
            camera.aspect = sizes.width / sizes.height;
            camera.updateProjectionMatrix();
            renderer.setSize(sizes.width, sizes.height);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        });

        
        createdBySword();
        
    
    </script>
</body>
</html>
