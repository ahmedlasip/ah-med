<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>النظام الشمسي</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: black;
            color: white;
            font-family: Arial, sans-serif;
        }
        canvas {
            display: block;
        }
        #header {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            text-align: center;
            z-index: 10;
        }
        #header h1 {
            font-size: 36px;
            font-weight: bold;
            margin: 0;
        }
        #phenomena {
            position: absolute;
            top: 60px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 18px;
            text-align: center;
            color: white; /* لون النص أبيض */
            white-space: nowrap; /* للحفاظ على النص في سطر واحد */
        }
        .planet-info-box {
            position: absolute;
            top: 120px;
            right: 10px;
            width: 250px;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            font-size: 16px;
            padding: 15px;
            border-radius: 10px;
            border: 2px solid #ffffff;
            margin-bottom: 10px;
            box-sizing: border-box;
            text-align: center;
        }
        .planet-name {
            font-size: 20px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .orbit-time-box {
            position: absolute;
            top: 120px;
            left: 0;
            width: 300px;
            height: calc(100vh - 120px);
            overflow-y: auto;
            background: rgba(0, 0, 0, 0.9);
            color: white;
            font-size: 16px;
            padding: 15px;
            border-right: 2px solid #ffffff;
            box-sizing: border-box;
            text-align: center;
        }
        .info-panel {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 80%;
            max-width: 600px;
            height: 80%;
            max-height: 400px;
            background: rgba(0, 0, 0, 0.9);
            color: white;
            font-size: 18px;
            padding: 20px;
            box-sizing: border-box;
            display: none;
            z-index: 100;
            border-radius: 10px;
            border: 2px solid #ffffff;
        }
        .info-panel button {
            position: absolute;
            top: 10px;
            left: 10px;
            padding: 10px 20px;
            font-size: 16px;
            background: #007BFF;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .info-panel button:hover {
            background: #0056b3;
        }
    </style>
</head>
<body>
<div id="header">
    <h1>النظام الشمسي</h1>
    <div id="phenomena">الظواهر التي تحدث داخل النظام...</div>
</div>
<div class="info-panel" id="infoPanel">
    <button id="backButton">رجوع إلى المدار الشمسي</button>
    <h1 id="infoTitle"></h1>
    <p id="infoContent"></p>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // إضافة الإضاءة المتغيرة من الشمس
    const sunLight = new THREE.PointLight(0xffffff, 1.5, 100);
    scene.add(sunLight);

    // إنشاء النجوم في الخلفية
    const starGeometry = new THREE.BufferGeometry();
    const starMaterial = new THREE.PointsMaterial({ color: 0xffffff });
    const starVertices = [];
    for (let i = 0; i < 3000; i++) {
        const x = (Math.random() - 0.5) * 2000;
        const y = (Math.random() - 0.5) * 2000;
        const z = (Math.random() - 0.5) * 2000;
        starVertices.push(x, y, z);
    }
    starGeometry.setAttribute('position', new THREE.Float32BufferAttribute(starVertices, 3));
    const stars = new THREE.Points(starGeometry, starMaterial);
    scene.add(stars);

    // إنشاء الشمس
    const sunGeometry = new THREE.SphereGeometry(3, 32, 32);
    const sunMaterial = new THREE.MeshStandardMaterial({ emissive: 0xFFFF00, emissiveIntensity: 1 });
    const sun = new THREE.Mesh(sunGeometry, sunMaterial);
    sunLight.position.copy(sun.position);
    scene.add(sun);

    // بيانات الكواكب مع تحويل مدة الدوران إلى سنوات أرضية
    const planetsData = [
        { name: "عطارد", radius: 0.5, distance: 6, speed: 0.02, color: 0x8B4513, moons: 0, orbitTimeYears: 0.24, mass: "3.3 × 10²³ كغ", gravity: "3.7 م/ث²", dayNightHours: "1407 ساعة", sunDistance: "57.9 مليون كم", tilt: 0 },
        { name: "الزهرة", radius: 0.6, distance: 9, speed: 0.015, color: 0xFFA500, moons: 0, orbitTimeYears: 0.62, mass: "4.8 × 10²⁴ كغ", gravity: "8.87 م/ث²", dayNightHours: "5832 ساعة", sunDistance: "108.2 مليون كم", tilt: 0 },
        { name: "الأرض", radius: 0.7, distance: 12, speed: 0.01, color: 0x0000FF, moons: 1, orbitTimeYears: 1, mass: "5.9 × 10²⁴ كغ", gravity: "9.8 م/ث²", dayNightHours: "24 ساعة", sunDistance: "149.6 مليون كم", tilt: 23.5 },
        { name: "المريخ", radius: 0.4, distance: 15, speed: 0.007, color: 0xFF4500, moons: 2, orbitTimeYears: 1.88, mass: "6.4 × 10²³ كغ", gravity: "3.71 م/ث²", dayNightHours: "24.6 ساعة", sunDistance: "227.9 مليون كم", tilt: 25 },
        { name: "المشتري", radius: 1.2, distance: 20, speed: 0.004, color: 0xFFD700, moons: 4, orbitTimeYears: 11.86, mass: "1.9 × 10²⁷ كغ", gravity: "24.79 م/ث²", dayNightHours: "9.9 ساعات", sunDistance: "778.5 مليون كم", tilt: 3 },
        { name: "زحل", radius: 1, distance: 25, speed: 0.003, color: 0xF5F5DC, moons: 3, orbitTimeYears: 29.46, mass: "5.7 × 10²⁶ كغ", gravity: "10.44 م/ث²", dayNightHours: "10.7 ساعات", sunDistance: "1.4 مليار كم", tilt: 26.7 },
        { name: "أورانوس", radius: 0.8, distance: 30, speed: 0.002, color: 0xADD8E6, moons: 2, orbitTimeYears: 84.01, mass: "8.7 × 10²⁵ كغ", gravity: "8.69 م/ث²", dayNightHours: "17.2 ساعة", sunDistance: "2.9 مليار كم", tilt: 97.8 },
        { name: "نبتون", radius: 0.7, distance: 35, speed: 0.0015, color: 0x00FFFF, moons: 1, orbitTimeYears: 164.79, mass: "1.0 × 10²⁶ كغ", gravity: "11.15 م/ث²", dayNightHours: "16.1 ساعة", sunDistance: "4.5 مليار كم", tilt: 28.3 }
    ];

    const planets = [];
    planetsData.forEach((data, index) => {
        const planetGeometry = new THREE.SphereGeometry(data.radius, 32, 32);
        const planetMaterial = data.type === "rocky" ?
            new THREE.MeshStandardMaterial({ color: data.color }) :
            new THREE.MeshPhongMaterial({ color: data.color, shininess: 50 });
        const planet = new THREE.Mesh(planetGeometry, planetMaterial);

        const orbitRadius = data.distance;
        planet.position.x = orbitRadius;

        const orbit = new THREE.Object3D();
        orbit.add(planet);
        scene.add(orbit);

        // إنشاء مدار وهمي
        const orbitGeometry = new THREE.RingGeometry(orbitRadius - 0.05, orbitRadius + 0.05, 64);
        const orbitMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide, transparent: true, opacity: 0.2 });
        const orbitMesh = new THREE.Mesh(orbitGeometry, orbitMaterial);
        orbitMesh.rotation.x = Math.PI / 2;
        scene.add(orbitMesh);

        // إنشاء الأقمار
        const moons = [];
        for (let i = 0; i < data.moons; i++) {
            const moonGeometry = new THREE.SphereGeometry(0.1, 16, 16);
            const moonMaterial = new THREE.MeshStandardMaterial({ color: 0xcccccc });
            const moon = new THREE.Mesh(moonGeometry, moonMaterial);
            moon.position.set(Math.random() * 2 - 1, Math.random() * 2 - 1, Math.random() * 2 - 1).normalize().multiplyScalar(data.radius + 0.5);
            planet.add(moon);
            moons.push(moon);
        }

        planets.push({ orbit, planet, speed: data.speed, rotations: 0, name: data.name, moons, orbitTimeYears: data.orbitTimeYears, mass: data.mass, gravity: data.gravity, dayNightHours: data.dayNightHours, sunDistance: data.sunDistance, tilt: data.tilt });
    });

    // إنشاء حزام الكويكبات بين المريخ والمشتري
    const asteroidBelt = [];
    for (let i = 0; i < 200; i++) {
        const asteroidGeometry = new THREE.SphereGeometry(Math.random() * 0.05 + 0.05, 8, 8);
        const asteroidMaterial = new THREE.MeshStandardMaterial({ color: 0x808080 });
        const asteroid = new THREE.Mesh(asteroidGeometry, asteroidMaterial);
        asteroid.position.set(
            Math.random() * 10 - 5,
            Math.random() * 10 - 5,
            Math.random() * 10 - 5
        ).normalize().multiplyScalar(18 + Math.random() * 2); // بين المريخ والمشتري
        asteroid.velocity = new THREE.Vector3(
            (Math.random() - 0.5) * 0.001,
            (Math.random() - 0.5) * 0.001,
            (Math.random() - 0.5) * 0.001
        );
        scene.add(asteroid);
        asteroidBelt.push(asteroid);
    }

    // لوحة المعلومات
    const infoPanel = document.getElementById('infoPanel');
    const infoTitle = document.getElementById('infoTitle');
    const infoContent = document.getElementById('infoContent');
    const backButton = document.getElementById('backButton');

    backButton.addEventListener('click', () => {
        infoPanel.style.display = 'none';
    });

    // تعديل الإضاءة بناءً على المسافة
    function updateLightIntensity(planet) {
        const distance = planet.position.distanceTo(sun.position);
        planet.material.emissiveIntensity = Math.max(0.1, 1 / (distance * distance));
    }

    // تحريك الكاميرا باستخدام الماوس
    let isDragging = false;
    let lastMouseX = 0;
    let lastMouseY = 0;
    let currentAngleX = 0;
    let currentAngleY = 0;
    let cameraDistance = 50;

    document.addEventListener('mousedown', (event) => {
        if (event.button === 0) {
            isDragging = true;
            lastMouseX = event.clientX;
            lastMouseY = event.clientY;
        }
    });

    document.addEventListener('mouseup', () => {
        isDragging = false;
    });

    document.addEventListener('mousemove', (event) => {
        if (isDragging) {
            const deltaX = event.clientX - lastMouseX;
            const deltaY = event.clientY - lastMouseY;

            currentAngleX -= deltaX * 0.005;
            currentAngleY -= deltaY * 0.005;

            lastMouseX = event.clientX;
            lastMouseY = event.clientY;
        }
    });

    document.addEventListener('wheel', (event) => {
        cameraDistance -= event.deltaY * 0.01;
        cameraDistance = Math.max(10, Math.min(100, cameraDistance));
    });

    // حلقة الرسم
    function animate() {
        requestAnimationFrame(animate);

        let phenomenaText = ""; // نص الظواهر

        planets.forEach((planet, index) => {
            planet.orbit.rotation.y += planet.speed;

            // دوران الكوكب حول نفسه
            planet.planet.rotation.y += planet.speed * 0.1;

            // تطبيق ميلان المحور
            planet.planet.rotation.z = (Math.PI / 180) * planet.tilt;

            // تحديث الإضاءة بناءً على المسافة
            updateLightIntensity(planet.planet);

            // تسجيل الظواهر (إكمال دورة أو اصطفاف)
            if (planet.orbit.rotation.y >= Math.PI * 2) {
                planet.rotations++;
                planet.orbit.rotation.y -= Math.PI * 2;

                // إكمال دورة
                phenomenaText += `اكمل ${planet.name} دورة حول الشمس `;
            }

            // تحديث عداد الدورات
            const rotationCounter = document.getElementById(`rotations-${index}`);
            if (rotationCounter) {
                planet.rotations = Math.floor(planet.orbit.rotation.y / (Math.PI * 2));
                rotationCounter.textContent = planet.rotations;
            }

            // دوران الأقمار حول الكواكب
            planet.moons.forEach(moon => {
                moon.position.applyAxisAngle(new THREE.Vector3(0, 1, 0), 0.05);
            });
        });

        // اصطفاف الكواكب
        const alignmentThreshold = 0.1;
        const alignedPlanets = planets.filter(planet => Math.abs(planet.orbit.rotation.y % (Math.PI * 2)) < alignmentThreshold);
        if (alignedPlanets.length > 1) {
            phenomenaText += `حدث اصطفاف بين الكواكب: ${alignedPlanets.map(p => p.name).join(', ')} `;
        }

        // تحديث نص الظواهر
        const phenomenaDiv = document.getElementById('phenomena');
        phenomenaDiv.textContent = phenomenaText.trim(); // إزالة الفراغات الزائدة

        // تحديث حركة حزام الكويكبات
        asteroidBelt.forEach(asteroid => {
            asteroid.position.add(asteroid.velocity);
            if (asteroid.position.distanceTo(scene.position) > 20 || asteroid.position.distanceTo(scene.position) < 16) {
                asteroid.velocity.negate();
            }
        });

        // تحديث موقع الكاميرا
        const camX = cameraDistance * Math.sin(currentAngleX) * Math.cos(currentAngleY);
        const camZ = cameraDistance * Math.cos(currentAngleX) * Math.cos(currentAngleY);
        const camY = cameraDistance * Math.sin(currentAngleY);
        camera.position.set(camX, camY, camZ);
        camera.lookAt(scene.position);

        renderer.render(scene, camera);
    }

    animate();

    // تعديل الحجم عند تغيير نافذة المتصفح
    window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // معالجة النقر على الكواكب
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();

    window.addEventListener('click', (event) => {
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

        raycaster.setFromCamera(mouse, camera);

        const intersects = raycaster.intersectObjects(planets.map(p => p.planet));
        if (intersects.length > 0) {
            const clickedPlanet = planets.find(p => p.planet === intersects[0].object);
            if (clickedPlanet) {
                infoTitle.textContent = clickedPlanet.name;
                infoContent.innerHTML = `
                    <p><strong>عدد الأقمار:</strong> ${clickedPlanet.moons.length}</p>
                    <p><strong>مدة الدورة حول الشمس:</strong> ${clickedPlanet.orbitTimeYears.toFixed(2)} سنة أرضية</p>
                    <p><strong>الكتلة:</strong> ${clickedPlanet.mass}</p>
                    <p><strong>الجاذبية:</strong> ${clickedPlanet.gravity}</p>
                    <p><strong>ساعات الليل والنهار:</strong> ${clickedPlanet.dayNightHours}</p>
                    <p><strong>المسافة من الشمس:</strong> ${clickedPlanet.sunDistance}</p>
                    <p><strong>ميلان المحور:</strong> ${clickedPlanet.tilt} درجة</p>
                `;
                infoPanel.style.display = 'block';
            }
        }
    });

    // إنشاء العدادات على الجانب الأيمن
    planetsData.forEach((data, index) => {
        const infoBox = document.createElement('div');
        infoBox.className = 'planet-info-box';
        infoBox.style.top = `${120 + index * 120}px`;
        infoBox.innerHTML = `
            <div class="planet-name">${data.name}</div>
            <div>دورات حول الشمس: <span id="rotations-${index}">0</span></div>
        `;
        document.body.appendChild(infoBox);
    });

    // إنشاء القائمة على الجانب الأيسر
    const orbitTimeBox = document.createElement('div');
    orbitTimeBox.className = 'orbit-time-box';
    let orbitTimeContent = '<div style="font-size: 20px; font-weight: bold;">معلومات الكواكب</div>';
    planetsData.forEach(data => {
        orbitTimeContent += `
            <div><strong>${data.name}</strong></div>
            <div>مدة الدوران حول الشمس: ${data.orbitTimeYears.toFixed(2)} سنة أرضية</div>
            <div>الكتلة: ${data.mass}</div>
            <div>الجاذبية: ${data.gravity}</div>
            <div>ساعات الليل والنهار: ${data.dayNightHours}</div>
            <div>المسافة من الشمس: ${data.sunDistance}</div>
            <div>ميلان المحور: ${data.tilt} درجة</div>
            <hr style="border-color: white;">
        `;
    });
    orbitTimeBox.innerHTML = orbitTimeContent;
    document.body.appendChild(orbitTimeBox);
</script>
</body>
</html>
