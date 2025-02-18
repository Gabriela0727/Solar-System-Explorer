# Solar-System-Explorer

This project is an interactive 3D solar system visualization built using Three.js that creates an immersive educational experience. The simulation features accurate representations of all eight planets with proper scaling and orbital mechanics, complete with a detailed sun displaying corona effects and dynamic glowing, realistic moon systems, an asteroid belt between Mars and Jupiter, comets with directional tails, twinkling stars, and distant rotating galaxies. The visualization incorporates sophisticated visual effects including day/night cycles on planets based on their position relative to the sun, dynamic sun glow and corona animations, realistic planet textures, atmospheric effects, and visible orbital paths. Users can interact with the system through orbit controls for zooming, panning, and rotating the view, while an information panel appears when hovering over objects, providing comprehensive details about each celestial body's classification, orbital characteristics, physical properties, and interesting facts. The technical implementation leverages Three.js for 3D rendering, employs raycasting for object selection, implements complex material systems, and features proper lighting with a point light for the sun and ambient light for general illumination. The project also includes responsive design elements and an animation speed control slider, making it both an educational tool and an engaging way to explore the solar system.

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Solar System with Day/Night Cycles and Twinkling Stars</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128/examples/js/controls/OrbitControls.js"></script>
    <style>
      body {
        margin: 0;
        overflow: hidden;
        background: #000000;
        font-family: Arial, sans-serif;
        color: white;
      }
      canvas {
        display: block;
      }

      .info-panel {
        position: absolute;
        top: 20px;
        right: 20px;
        background: rgba(0, 0, 0, 0.85);
        padding: 20px;
        border-radius: 12px;
        border: 2px solid rgba(255, 255, 255, 0.1);
        max-width: 300px;
        backdrop-filter: blur(10px);
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        transition: all 0.3s ease;
      }

      .info-panel:hover {
        border-color: rgba(255, 255, 255, 0.2);
        transform: translateY(-2px);
      }

      .info-panel h2 {
        margin: 0 0 15px;
        font-size: 24px;
        color: #ACE1AF;
        border-bottom: 1px solid rgba(255, 255, 255, 0.1);
        padding-bottom: 10px;
      }

      .info-panel .stats {
        display: grid;
        grid-template-columns: auto 1fr;
        gap: 8px 12px;
        margin-bottom: 15px;
      }

      .info-panel .label {
        color: #FFCCCC;
        font-size: 14px;
      }

      .info-panel .value {
        color: #CCCCFF;
        font-size: 14px;
        font-weight: 500;
      }

      .info-panel .description {
        font-size: 14px;
        line-height: 1.5;
        color: #FFCCCC;
        margin-top: 15px;
      }

      .info-panel .moons-list {
        margin-top: 15px;
        padding-top: 15px;
        border-top: 1px solid rgba(255, 255, 255, 0.1);
      }

      .info-panel h3 {
        color: #ACE1AF;
        font-size: 18px;
        margin: 0 0 10px;
      }

      .info-panel .moon-item {
        font-size: 14px;
        color: #CCCCFF;
        margin: 5px 0;
        padding: 5px 10px;
        background: rgba(255, 255, 255, 0.1);
        border-radius: 4px;
        transition: background 0.2s ease;
      }

      .info-panel .moon-item:hover {
        background: rgba(255, 255, 255, 0.2);
      }

      .info-panel .additional-info {
        margin-top: 15px;
        padding-top: 15px;
        border-top: 1px solid rgba(255, 255, 255, 0.1);
        font-size: 13px;
        color: #FFCCCC;
      }
      .speed-control {
        position: absolute;
        bottom: 20px;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0, 0, 0, 0.85);
        padding: 15px 25px;
        border-radius: 12px;
        border: 2px solid rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(10px);
        display: flex;
        align-items: center;
        gap: 15px;
        z-index: 1000;
      }

      .speed-control label {
        color: #ACE1AF;
        font-size: 14px;
      }

      .speed-control input[type="range"] {
        width: 200px;
        cursor: pointer;
      }

      .speed-control span {
        color: white;
        min-width: 40px;
      }
    </style>
  </head>
  <body>
    <div class="info-panel" id="infoPanel">
      <h2>Celestial Info</h2>
      <p>Hover over a planet, moon, or asteroid to see details here.</p>
    </div>
    <div class="speed-control">
      <label for="speedSlider">Animation Speed</label>
      <input
        type="range"
        id="speedSlider"
        min="0.1"
        max="5"
        step="0.1"
        value="1"
      />
      <span id="speedValue">1x</span>
    </div>
    <script>
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        10000
      );
      let speedMultiplier = 1;
      // slider elements
      const speedSlider = document.getElementById("speedSlider");
      const speedValue = document.getElementById("speedValue");

      // updates speed when slider changes
      speedSlider.addEventListener("input", (event) => {
        speedMultiplier = parseFloat(event.target.value);
        speedValue.textContent = `${speedMultiplier.toFixed(1)}x`;
      });
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setClearColor(0x000000);
      document.body.appendChild(renderer.domElement);

      const controls = new THREE.OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;

      camera.position.set(0, 50, 150);
      camera.lookAt(scene.position);

      const sunLight = new THREE.PointLight(0xffffff, 1.5, 1000);
      sunLight.position.set(0, 0, 0);
      scene.add(sunLight);

      const ambientLight = new THREE.AmbientLight(0x404040, 0.5);
      scene.add(ambientLight);

      function createStars() {
        const starGeometry = new THREE.BufferGeometry();
        const starCount = 70000;
        const starVertices = [];
        const starOpacities = new Float32Array(starCount);
        const starTwinkleSpeed = new Float32Array(starCount);

        for (let i = 0; i < starCount; i++) {
          const x = (Math.random() - 0.5) * 4000;
          const y = (Math.random() - 0.5) * 4000;
          const z = (Math.random() - 0.5) * 4000;
          starVertices.push(x, y, z);

          starOpacities[i] = Math.random();
          starTwinkleSpeed[i] = 0.3 + Math.random() * 0.5;
        }

        starGeometry.setAttribute(
          "position",
          new THREE.Float32BufferAttribute(starVertices, 3)
        );
        starGeometry.setAttribute(
          "opacity",
          new THREE.Float32BufferAttribute(starOpacities, 1)
        );
        starGeometry.setAttribute(
          "twinkleSpeed",
          new THREE.Float32BufferAttribute(starTwinkleSpeed, 1)
        );

        const starMaterial = new THREE.PointsMaterial({
          color: 0xffffff,
          size: 1,
          transparent: true,
          opacity: 1,
          vertexColors: false,
        });

        const stars = new THREE.Points(starGeometry, starMaterial);
        stars.userData.opacities = starOpacities;
        stars.userData.twinkleSpeeds = starTwinkleSpeed;
        scene.add(stars);
        return stars;
      }

      const stars = createStars();

      function createSunGlow(size, opacity, color) {
        const glowMaterial = new THREE.MeshBasicMaterial({
          color: color,
          transparent: true,
          opacity: opacity,
        });
        const glowGeometry = new THREE.SphereGeometry(size, 32, 32);
        return new THREE.Mesh(glowGeometry, glowMaterial);
      }
      function createGalaxy(params = {}) {
        const {
          particles = 50000,
          radius = 1000,
          branches = 5,
          spin = 1,
          randomness = 0.2,
          randomnessPower = 3,
          insideColor = "#ff6030",
          outsideColor = "#1b3984",
          size = 0.01,
        } = params;

        // creates buffer geometry
        const geometry = new THREE.BufferGeometry();
        const positions = new Float32Array(particles * 3);
        const colors = new Float32Array(particles * 3);

        const colorInside = new THREE.Color(insideColor);
        const colorOutside = new THREE.Color(outsideColor);

        for (let i = 0; i < particles; i++) {
          const i3 = i * 3;

          //position calculation
          const branchAngle = ((i % branches) / branches) * Math.PI * 2;
          const radiusPosition = Math.random() * radius;
          const spinAngle = radiusPosition * spin;

          // adds randomness to position
          const randomX =
            Math.pow(Math.random(), randomnessPower) *
            (Math.random() < 0.5 ? 1 : -1) *
            randomness *
            radiusPosition;
          const randomY =
            Math.pow(Math.random(), randomnessPower) *
            (Math.random() < 0.5 ? 1 : -1) *
            randomness *
            radiusPosition;
          const randomZ =
            Math.pow(Math.random(), randomnessPower) *
            (Math.random() < 0.5 ? 1 : -1) *
            randomness *
            radiusPosition;

          // sets positions
          positions[i3] =
            Math.cos(branchAngle + spinAngle) * radiusPosition + randomX;
          positions[i3 + 1] = randomY;
          positions[i3 + 2] =
            Math.sin(branchAngle + spinAngle) * radiusPosition + randomZ;

          // color gradient
          const mixColor = colorInside.clone();
          mixColor.lerp(colorOutside, radiusPosition / radius);

          colors[i3] = mixColor.r;
          colors[i3 + 1] = mixColor.g;
          colors[i3 + 2] = mixColor.b;
        }

        geometry.setAttribute(
          "position",
          new THREE.BufferAttribute(positions, 3)
        );
        geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));

      
        const material = new THREE.PointsMaterial({
          size,
          sizeAttenuation: true,
          depthWrite: false,
          blending: THREE.AdditiveBlending,
          vertexColors: true,
          transparent: true,
          opacity: 0.5,
        });

        const galaxy = new THREE.Points(geometry, material);

        // adds rotation data
        galaxy.userData.rotationSpeed = 0.0001 + Math.random() * 0.0001;

        return galaxy;
      }

      // creates and position galaxies
      const galaxies = [
        {
          params: {
            particles: 50000,
            radius: 1000,
            branches: 5,
            spin: 1,
            randomness: 0.2,
            randomnessPower: 3,
            insideColor: "#ff6030",
            outsideColor: "#1b3984",
            size: 0.3,
            position: new THREE.Vector3(1000, 500, -1500),
          },
        },
        {
          params: {
            particles: 40000,
            radius: 800,
            branches: 3,
            spin: -1.5,
            randomness: 0.3,
            randomnessPower: 2,
            insideColor: "#1b3984",
            outsideColor: "#ff6030",
            size: 0.08,
            position: new THREE.Vector3(-1500, -500, -1000),
          },
        },
        {
          params: {
            particles: 30000,
            radius: 600,
            branches: 4,
            spin: 2,
            randomness: 0.2,
            randomnessPower: 4,
            insideColor: "#ff8060",
            outsideColor: "#2b4994",
            size: 0.12,
            position: new THREE.Vector3(-1250, 1000, -500),
          },
        },
      ].map(({ params }) => {
        const galaxy = createGalaxy(params);
        galaxy.position.copy(params.position);
        galaxy.rotation.x = Math.random() * Math.PI;
        galaxy.rotation.y = Math.random() * Math.PI;
        scene.add(galaxy);
        return galaxy;
      });

      function createSunCorona() {
        const points = [];
        const coronaSegments = 32;
        for (let i = 0; i < coronaSegments; i++) {
          const angle = (i / coronaSegments) * Math.PI * 2;
          const radius = 12 + Math.random() * 4;
          points.push(
            new THREE.Vector2(
              Math.cos(angle) * radius,
              Math.sin(angle) * radius
            )
          );
        }
        points.push(points[0]);

        const coronaGeometry = new THREE.LatheGeometry(points, 32);
        const coronaMaterial = new THREE.MeshBasicMaterial({
          color: 0xff7700,
          transparent: true,
          opacity: 0.3,
          side: THREE.DoubleSide,
        });
        return new THREE.Mesh(coronaGeometry, coronaMaterial);
      }

      const sunGeometry = new THREE.SphereGeometry(8, 64, 64);
      const sunMaterial = new THREE.MeshStandardMaterial({
        emissive: 0xffaa00,
        emissiveIntensity: 1,
        roughness: 0.2,
        metalness: 0,
      });
      const sun = new THREE.Mesh(sunGeometry, sunMaterial);
      scene.add(sun);

      const glow1 = createSunGlow(9, 0.5, 0xffaa00);
      const glow2 = createSunGlow(10, 0.3, 0xffdd00);
      const glow3 = createSunGlow(11, 0.1, 0xffff00);
      scene.add(glow1);
      scene.add(glow2);
      scene.add(glow3);

      const corona = createSunCorona();
      scene.add(corona);

      const planetData = [
        {
          name: "Mercury",
          size: 2,
          color: 0xaaaaaa,
          distance: 20,
          speed: 0.02,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/8k_mercury.jpg?v=1739218972511",
          atmosphereColor: 0xaaaaaa,
          moons: [],
        },
        {
          name: "Venus",
          size: 3,
          color: 0xffcc99,
          distance: 30,
          speed: 0.015,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/4k_venus_atmosphere.jpg?v=1739218936369",
          atmosphereColor: 0xffcc99,
          moons: [],
        },
        {
          name: "Earth",
          size: 4,
          color: 0x4444ff,
          distance: 40,
          speed: 0.01,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/8k_earth_daymap.jpg?v=1739218943949",
          atmosphereColor: 0x4444ff,
          moons: [
            {
              name: "Moon",
              size: 1,
              distance: 6,
              speed: 0.05,
              color: 0xaaaaaa,
            },
          ],
        },
        {
          name: "Mars",
          size: 3.5,
          color: 0xff4444,
          distance: 50,
          speed: 0.008,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/8k_mars.jpg?v=1739218965450",
          atmosphereColor: 0xff4444,
          moons: [
            {
              name: "Phobos",
              size: 0.7,
              distance: 5,
              speed: 0.07,
              color: 0x888888,
            },
            {
              name: "Deimos",
              size: 0.5,
              distance: 8,
              speed: 0.04,
              color: 0x999999,
            },
          ],
        },
        {
          name: "Jupiter",
          size: 8,
          color: 0xffaa88,
          distance: 70,
          speed: 0.004,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/8k_jupiter.jpg?v=1739218954364",
          atmosphereColor: 0xffaa88,
          moons: [
            {
              name: "Io",
              size: 1.2,
              distance: 10,
              speed: 0.06,
              color: 0xffcc88,
            },
            {
              name: "Europa",
              size: 1,
              distance: 14,
              speed: 0.05,
              color: 0xbbbbbb,
            },
            {
              name: "Ganymede",
              size: 1.5,
              distance: 18,
              speed: 0.03,
              color: 0xdddddd,
            },
            {
              name: "Callisto",
              size: 1.4,
              distance: 22,
              speed: 0.02,
              color: 0xcccccc,
            },
          ],
        },
        {
          name: "Saturn",
          size: 7,
          color: 0xffdd99,
          distance: 90,
          speed: 0.003,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/8k_saturn.jpg?v=1739219290213",
          atmosphereColor: 0xffdd99,
          moons: [],
        },
        {
          name: "Uranus",
          size: 6,
          color: 0x66ccff,
          distance: 110,
          speed: 0.002,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/2k_uranus.jpg?v=1739218939048",
          atmosphereColor: 0x66ccff,
          moons: [],
        },
        {
          name: "Neptune",
          size: 6,
          color: 0x3366ff,
          distance: 130,
          speed: 0.0015,
          texture:
            "https://cdn.glitch.global/bc88ebd6-ae9a-43e7-963f-45a604480200/2k_neptune.jpg?v=1739218930401",
          atmosphereColor: 0x3366ff,
          moons: [],
        },
      ];

      const raycaster = new THREE.Raycaster();
      const mouse = new THREE.Vector2();
      const infoPanel = document.getElementById("infoPanel");

      function loadTexture(url) {
        const textureLoader = new THREE.TextureLoader();
        return new Promise((resolve, reject) => {
          textureLoader.load(
            url,
            (texture) => resolve(texture),
            undefined,
            (error) => reject(error)
          );
        });
      }

      function createPlanetMaterial(color, textureUrl) {
        if (textureUrl) {
          const material = new THREE.MeshPhongMaterial({
            map: null, // will set when texture loads
            bumpScale: 0.05,
            specular: new THREE.Color("grey"),
            shininess: 10,
          });

          // load and place the texture
          loadTexture(textureUrl)
            .then((texture) => {
              material.map = texture;
              material.needsUpdate = true;
            })
            .catch((error) => {
              console.error("Error loading texture:", error);
              // fall back to basic material if texture doesn't load
              material.color = new THREE.Color(color);
            });

          return { dayTexture: material, nightTexture: material };
        }

        // fall back to basic materials if no texture URL is placed
        const dayTexture = new THREE.MeshStandardMaterial({
          color: color,
          roughness: 0.7,
          metalness: 0.3,
        });

        const nightTexture = new THREE.MeshStandardMaterial({
          color: new THREE.Color(color).multiplyScalar(0.2),
          roughness: 0.9,
          metalness: 0.1,
          emissive: new THREE.Color(color).multiplyScalar(0.1),
        });

        return { dayTexture, nightTexture };
      }

      const planets = planetData.map((data) => {
        const planetGroup = new THREE.Group();
        const { dayTexture, nightTexture } = createPlanetMaterial(
          data.color,
          data.texture
        );

        const geometry = new THREE.SphereGeometry(data.size, 32, 32);
        const planet = new THREE.Mesh(geometry, dayTexture);
        planet.name = data.name;

        planet.userData.rotationSpeed = 0.01 + Math.random() * 0.01;
        planet.userData.axialTilt = (Math.random() - 0.5) * 0.5;

        const atmosphereGeometry = new THREE.SphereGeometry(
          data.size * 1.1,
          32,
          32
        );
        const atmosphereMaterial = new THREE.MeshBasicMaterial({
          color: data.atmosphereColor,
          transparent: true,
          opacity: 0.3,
        });
        const atmosphere = new THREE.Mesh(
          atmosphereGeometry,
          atmosphereMaterial
        );

        const orbitGeometry = new THREE.RingGeometry(
          data.distance - 0.05,
          data.distance + 0.05,
          64
        );
        const orbitMaterial = new THREE.MeshBasicMaterial({
          color: 0xffffff,
          side: THREE.DoubleSide,
          transparent: true,
          opacity: 0.3,
        });
        const orbit = new THREE.Mesh(orbitGeometry, orbitMaterial);
        orbit.rotation.x = Math.PI / 2;
        scene.add(orbit);

        planetGroup.add(planet);
        planetGroup.add(atmosphere);

        const moons = (data.moons || []).map((moonData) => {
          const moonGroup = new THREE.Group();
          const moonGeometry = new THREE.SphereGeometry(moonData.size, 16, 16);
          const moonMaterial = new THREE.MeshStandardMaterial({
            color: moonData.color,
          });
          const moon = new THREE.Mesh(moonGeometry, moonMaterial);
          moon.name = moonData.name;
          moonGroup.add(moon);

          const moonOrbitGeometry = new THREE.RingGeometry(
            moonData.distance - 0.1,
            moonData.distance + 0.1,
            32
          );
          const moonOrbitMaterial = new THREE.MeshBasicMaterial({
            color: 0xffffff,
            side: THREE.DoubleSide,
            transparent: true,
            opacity: 0.2,
          });
          const moonOrbit = new THREE.Mesh(
            moonOrbitGeometry,
            moonOrbitMaterial
          );
          moonOrbit.rotation.x = Math.PI / 2;
          planetGroup.add(moonOrbit);

          moonGroup.userData = {
            ...moonData,
            angle: Math.random() * Math.PI * 2,
          };
          planetGroup.add(moonGroup);
          return moonGroup;
        });

        planetGroup.userData = {
          ...data,
          angle: Math.random() * Math.PI * 2,
          moons,
          dayMaterial: dayTexture,
          nightMaterial: nightTexture,
        };

        return planetGroup;
      });

      planets.forEach((planet) => scene.add(planet));

      function createComet() {
        const cometGroup = new THREE.Group();

        // creates comet nucleus
        const cometGeometry = new THREE.SphereGeometry(1, 32, 32);
        const cometMaterial = new THREE.MeshPhongMaterial({
          color: 0xcccccc,
          emissive: 0x444444,
        });
        const comet = new THREE.Mesh(cometGeometry, cometMaterial);

        // create comet tail
        const tailGeometry = new THREE.ConeGeometry(2, 20, 32);
        const tailMaterial = new THREE.MeshBasicMaterial({
          color: 0x88ccff,
          transparent: true,
          opacity: 0.3,
        });
        const tail = new THREE.Mesh(tailGeometry, tailMaterial);
        tail.position.z = -10;
        tail.rotation.x = Math.PI / 2;

        cometGroup.add(comet);
        cometGroup.add(tail);

        // set initial position and properties
        const angle = Math.random() * Math.PI * 2;
        const distance = 150 + Math.random() * 50;
        cometGroup.position.x = Math.cos(angle) * distance;
        cometGroup.position.z = Math.sin(angle) * distance;
        cometGroup.position.y = (Math.random() - 0.5) * 50;

        cometGroup.userData = {
          angle: angle,
          distance: distance,
          speed: 0.001 + Math.random() * 0.002,
          inclination: (Math.random() * Math.PI) / 4,
        };

        scene.add(cometGroup);
        return cometGroup;
      }

      function createEnhancedAsteroidBelt() {
        const asteroidBelt = new THREE.Group();
        const asteroidCount = 1000;

        // creates a couple of asteroid geometries for variety
        const geometries = [
          new THREE.DodecahedronGeometry(0.3),
          new THREE.DodecahedronGeometry(0.5),
          new THREE.DodecahedronGeometry(0.7),
          new THREE.IcosahedronGeometry(0.4),
          new THREE.TetrahedronGeometry(0.4),
        ];

        const materials = [
          new THREE.MeshStandardMaterial({ color: 0x888888, roughness: 0.8 }),
          new THREE.MeshStandardMaterial({ color: 0x777777, roughness: 0.7 }),
          new THREE.MeshStandardMaterial({ color: 0x666666, roughness: 0.9 }),
        ];

        for (let i = 0; i < asteroidCount; i++) {
          const geometry =
            geometries[Math.floor(Math.random() * geometries.length)];
          const material =
            materials[Math.floor(Math.random() * materials.length)];
          const asteroid = new THREE.Mesh(geometry, material);

          // position
          const angle = Math.random() * Math.PI * 2;
          const distance = 55 + Math.random() * 15; // Between Mars and Jupiter
          const height = (Math.random() - 0.5) * 5;

          asteroid.position.x = Math.cos(angle) * distance;
          asteroid.position.z = Math.sin(angle) * distance;
          asteroid.position.y = height;

          // random rotation
          asteroid.rotation.x = Math.random() * Math.PI;
          asteroid.rotation.y = Math.random() * Math.PI;
          asteroid.rotation.z = Math.random() * Math.PI;

          // adds orbital properties
          asteroid.userData = {
            angle: angle,
            distance: distance,
            rotationSpeed: Math.random() * 0.02,
            orbitSpeed: 0.001 + Math.random() * 0.002,
          };

          asteroidBelt.add(asteroid);
        }

        scene.add(asteroidBelt);
        return asteroidBelt;
      }

      const comets = Array(3)
        .fill(null)
        .map(() => createComet());
      const asteroidBelt = createEnhancedAsteroidBelt();

      function animate() {
        requestAnimationFrame(animate);

        // animates stars twinkling
        const time = Date.now() * 0.001;
        const opacities = stars.userData.opacities;
        const twinkleSpeeds = stars.userData.twinkleSpeeds;

        for (let i = 0; i < opacities.length; i++) {
          opacities[i] = 0.5 + Math.sin(time * twinkleSpeeds[i]) * 0.5;
        }
        stars.geometry.attributes.opacity.needsUpdate = true;
        stars.material.opacity = 1;

        // animates sun
        sun.rotation.y += 0.001 * speedMultiplier;

        // animates sun intensity
        const pulseIntensity = 1 + Math.sin(time) * 0.2;
        sunLight.intensity = 2 * pulseIntensity;
        sunMaterial.emissiveIntensity = pulseIntensity;

        // animates glow layers
        glow1.scale.set(
          1 + Math.sin(time * 1.1) * 0.04,
          1 + Math.sin(time * 1.1) * 0.04,
          1 + Math.sin(time * 1.1) * 0.04
        );
        glow2.scale.set(
          1 + Math.sin(time * 0.8) * 0.06,
          1 + Math.sin(time * 0.8) * 0.06,
          1 + Math.sin(time * 0.8) * 0.06
        );
        glow3.scale.set(
          1 + Math.sin(time * 0.6) * 0.08,
          1 + Math.sin(time * 0.6) * 0.08,
          1 + Math.sin(time * 0.6) * 0.08
        );

        // rotates corona
        corona.rotation.y += 0.0005 * speedMultiplier;
        corona.rotation.x = Math.sin(time * 0.3) * 0.1;

        // animates planets
        planets.forEach((planetGroup) => {
          const data = planetGroup.userData;
          // orbital movement
          data.angle += data.speed * speedMultiplier;
          planetGroup.position.x = Math.cos(data.angle) * data.distance;
          planetGroup.position.z = Math.sin(data.angle) * data.distance;

          // planet rotation and day/night cycle
          const planet = planetGroup.children[0];
          planet.rotation.y += planet.userData.rotationSpeed * speedMultiplier;

          // angle to sun for day/night effect
          const angleToSun = Math.atan2(
            planetGroup.position.z,
            planetGroup.position.x
          );

          // update material based on angle to sun
          const sunFacing = (Math.cos(angleToSun - planet.rotation.y) + 1) / 2;
          planet.material.color.lerpColors(
            new THREE.Color(data.nightMaterial.color),
            new THREE.Color(data.dayMaterial.color),
            sunFacing
          );

          // animates moons
          data.moons?.forEach((moon) => {
            const moonData = moon.userData;
            moonData.angle += moonData.speed * speedMultiplier;
            moon.position.x = Math.cos(moonData.angle) * moonData.distance;
            moon.position.z = Math.sin(moonData.angle) * moonData.distance;

            // adds rotation to moons
            const moonMesh = moon.children[0];
            moonMesh.rotation.y += 0.005 * speedMultiplier;
          });
        });
        comets.forEach((cometGroup) => {
          const data = cometGroup.userData;
          data.angle += data.speed * speedMultiplier;

          // updates position
          cometGroup.position.x = Math.cos(data.angle) * data.distance;
          cometGroup.position.z = Math.sin(data.angle) * data.distance;
          cometGroup.position.y = Math.sin(data.angle + data.inclination) * 50;

          // orients tail away from sun
          cometGroup.lookAt(0, 0, 0);
          cometGroup.rotateY(Math.PI);
        });

        // animates asteroid
        asteroidBelt.children.forEach((asteroid) => {
          const data = asteroid.userData;
          data.angle += data.orbitSpeed * speedMultiplier;

          // updates position
          asteroid.position.x = Math.cos(data.angle) * data.distance;
          asteroid.position.z = Math.sin(data.angle) * data.distance;

          // rotates asteroid
          asteroid.rotation.x += data.rotationSpeed * speedMultiplier;
          asteroid.rotation.y += data.rotationSpeed * speedMultiplier;
        });

        galaxies.forEach((galaxy) => {
          galaxy.rotation.y += galaxy.userData.rotationSpeed * speedMultiplier;
        });

        controls.update();
        renderer.render(scene, camera);
      }
      function getRandomFact(objectName) {
        const facts = {
          Mercury: [
            "Closest planet to the Sun",
            "Smallest planet in our solar system",
            "Has no moons",
          ],
          Venus: [
            "Hottest planet in our solar system",
            "Rotates backwards compared to most planets",
            "Often called Earth's sister planet",
          ],
          Earth: [
            "Only known planet with life",
            "Has one natural satellite - the Moon",
            "71% of surface covered by water",
          ],
          Mars: [
            "Known as the Red Planet",
            "Has the largest volcano in the solar system",
            "Home to Olympus Mons",
          ],
          Jupiter: [
            "Largest planet in our solar system",
            "Has the Great Red Spot storm",
            "Has at least 79 moons",
          ],
          Saturn: [
            "Famous for its spectacular ring system",
            "Least dense planet in the solar system",
            "Has 82 confirmed moons",
          ],
          Uranus: [
            "Rotates on its side",
            "First planet discovered using a telescope",
            "Has 27 known moons",
          ],
          Neptune: [
            "The windiest planet",
            "Last planet in our solar system",
            "Has 14 known moons",
          ],
        };
        return facts[objectName]
          ? facts[objectName][
              Math.floor(Math.random() * facts[objectName].length)
            ]
          : null;
      }

      function formatDistance(distance) {
        return `${distance.toFixed(1)} AU`;
      }

      function formatSpeed(speed) {
        return `${(speed * 100).toFixed(2)} km/s`;
      }

      function onMouseMove(event) {
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

        raycaster.setFromCamera(mouse, camera);

        const allObjects = [];
        scene.traverse((object) => {
          if (object instanceof THREE.Mesh && object.name) {
            allObjects.push(object);
          }
        });

        const intersects = raycaster.intersectObjects(allObjects, false);

        if (intersects.length > 0) {
          const object = intersects[0].object;
          let parentGroup = object.parent;

          while (parentGroup && !parentGroup.userData.distance) {
            parentGroup = parentGroup.parent;
          }

          const data = parentGroup ? parentGroup.userData : {};
          const isMoon =
            object.parent.userData.distance !== undefined &&
            parentGroup &&
            parentGroup.userData.name !== object.name;

          let infoHTML = `<h2>${object.name}</h2><div class="stats">`;

          if (isMoon) {
            const parentName = parentGroup.userData.name;
            infoHTML += `
            <span class="label">Classification:</span>
            <span class="value">Natural Satellite</span>
            <span class="label">Parent Planet:</span>
            <span class="value">${parentName}</span>
            <span class="label">Orbital Distance:</span>
            <span class="value">${formatDistance(
              object.parent.userData.distance
            )}</span>
            <span class="label">Orbital Speed:</span>
            <span class="value">${formatSpeed(
              object.parent.userData.speed
            )}</span>
            <span class="label">Diameter:</span>
            <span class="value">${(
              object.geometry.parameters.radius * 2
            ).toFixed(1)} km</span>
            <span class="label">Surface Gravity:</span>
            <span class="value">${(
              object.geometry.parameters.radius * 0.5
            ).toFixed(2)} m/s²</span>
          </div>
          <div class="additional-info">
            <p>This moon is in a stable orbit around ${parentName}. Click and drag to explore its orbital path.</p>
          </div>`;
          } else if (data.distance) {
            const fact = getRandomFact(object.name);
            infoHTML += `
            <span class="label">Classification:</span>
            <span class="value">Planet</span>
            <span class="label">Distance from Sun:</span>
            <span class="value">${formatDistance(data.distance)}</span>
            <span class="label">Orbital Speed:</span>
            <span class="value">${formatSpeed(data.speed)}</span>
            <span class="label">Diameter:</span>
            <span class="value">${(
              object.geometry.parameters.radius * 2
            ).toFixed(1)} km</span>
            <span class="label">Rotation Period:</span>
            <span class="value">${(24 / object.userData.rotationSpeed).toFixed(
              1
            )} hours</span>
            <span class="label">Surface Temperature:</span>
            <span class="value">${Math.round(300 - data.distance * 2)}°C</span>
          </div>`;

            if (fact) {
              infoHTML += `
            <div class="additional-info">
              <p>Did you know? ${fact}</p>
            </div>`;
            }

            if (data.moons && data.moons.length > 0) {
              infoHTML += `
            <div class="moons-list">
              <h3>Moons (${data.moons.length})</h3>
              ${data.moons
                .map(
                  (moon) => `
                <div class="moon-item">
                  • ${moon.children[0].name}
                </div>
              `
                )
                .join("")}
            </div>`;
            }
          }

          infoPanel.innerHTML = infoHTML;
        } else {
          infoPanel.innerHTML = `
          <h2>Solar System Explorer</h2>
          <div class="description">
            <p>Hover over any planet or moon to discover detailed information about its properties,
            orbital characteristics, and satellite system.</p>
            <p>Click and drag to rotate the view. Scroll to zoom in/out.</p>
          </div>`;
        }
      }

      window.addEventListener("mousemove", onMouseMove);

      window.addEventListener("resize", () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      });

      animate();
    </script>
  </body>
</html>

