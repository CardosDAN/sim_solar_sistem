<script>
    import * as THREE from 'three';
    import { onMount } from 'svelte';
    import * as dat from 'dat.gui';
    import SceneInit from "./entities/SceneInit.js";
    import Planet from "./entities/Planet.js";
    import Rotation from "./entities/Rotation.js";

    let gui;

    const initGui = () => {
        gui = new dat.GUI();
    };

    onMount( () => {
        const test = new SceneInit();
        test.initScene();
        test.animate();

        const sunGeometry = new THREE.SphereGeometry(8);
        const sunTexture = new THREE.TextureLoader().load( 'sun.jpeg' );
        const sunMaterial = new THREE.MeshBasicMaterial( { map: sunTexture } );
        const sun = new THREE.Mesh( sunGeometry, sunMaterial );
        const solarSystem = new THREE.Group();
        solarSystem.add( sun );
        test.scene.add( solarSystem );

        const mercury = new Planet(2, 16, 'mercury.jpeg');
        const mercuryMesh = mercury.getMesh();
        let mercurySystem = new THREE.Group();
        mercurySystem.add(mercuryMesh);

        const venus = new Planet(3, 32, 'venus.jpeg');
        const venusMesh = venus.getMesh();
        let venusSystem = new THREE.Group();
        venusSystem.add(venusMesh);

        const earth = new Planet(4, 48, 'earth.jpeg');
        const earthMesh = earth.getMesh();
        let earthSystem = new THREE.Group();
        earthSystem.add(earthMesh);

        const mars = new Planet(3, 64, 'mars.jpg');
        const marsMesh = mars.getMesh();
        let marsSystem = new THREE.Group();
        marsSystem.add(marsMesh);

        const jupiter = new Planet(11, 90, 'jupiter.jpeg');
        const jupiterMesh = jupiter.getMesh();
        let jupiterSystem = new THREE.Group();
        jupiterSystem.add(jupiterMesh);


        const saturn = new Planet(9, 130, 'saturn.jpeg');
        const saturnMesh = saturn.getMesh();
        let saturnSystem = new THREE.Group();
        saturnSystem.add(saturnMesh);

        const innerRadius1 = 10; // Raza interioară a primului inel
        const outerRadius1 = 13; // Raza exterioară a primului inel
        const ringSegments = 32; // Numărul de segmente ale inelului
        const ringGeometry1 = new THREE.RingGeometry(innerRadius1, outerRadius1, ringSegments);

        const innerRadius2 = 15; // Raza interioară a celui de-al doilea inel
        const outerRadius2 = 18; // Raza exterioară a celui de-al doilea inel
        const ringGeometry2 = new THREE.RingGeometry(innerRadius2, outerRadius2, ringSegments);

        const textureLoader = new THREE.TextureLoader();
        const saturnRingTexture = textureLoader.load('saturnRing.jpeg');

        const ringMaterial1 = new THREE.MeshBasicMaterial({ map: saturnRingTexture, side: THREE.DoubleSide });
        const ringMaterial2 = new THREE.MeshBasicMaterial({ map: saturnRingTexture, side: THREE.DoubleSide });

        const ringMesh1 = new THREE.Mesh(ringGeometry1, ringMaterial1);
        const ringMesh2 = new THREE.Mesh(ringGeometry2, ringMaterial2);

        ringMesh1.rotation.x = Math.PI / 2;
        ringMesh2.rotation.x = Math.PI / 2;
        ringMesh1.position.set(0, 0, 1);
        ringMesh2.position.set(0, 0, 1);


        saturnMesh.add(ringMesh1, ringMesh2);

        const uranus = new Planet(6, 160, 'uranus.jpeg');
        const uranusMesh = uranus.getMesh();
        let uranusSystem = new THREE.Group();
        uranusSystem.add(uranusMesh);

        const neptune = new Planet(5, 180, 'neptune.jpeg');
        const neptuneMesh = neptune.getMesh();
        let neptuneSystem = new THREE.Group();
        neptuneSystem.add(neptuneMesh);


        solarSystem.add(mercurySystem, venusSystem, earthSystem, marsSystem, jupiterSystem, saturnSystem, uranusSystem, neptuneSystem);

        const mercuryRotation = new Rotation(mercuryMesh);
        const mercuryRotationMesh = mercuryRotation.getMesh();
        mercurySystem.add(mercuryRotationMesh);

        const venusRotation = new Rotation(venusMesh);
        const venusRotationMesh = venusRotation.getMesh();
        venusSystem.add(venusRotationMesh);

        const earthRotation = new Rotation(earthMesh);
        const earthRotationMesh = earthRotation.getMesh();
        earthSystem.add(earthRotationMesh);

        const marsRotation = new Rotation(marsMesh);
        const marsRotationMesh = marsRotation.getMesh();
        marsSystem.add(marsRotationMesh);

        const jupiterRotation = new Rotation(jupiterMesh);
        const jupiterRotationMesh = jupiterRotation.getMesh();
        jupiterSystem.add(jupiterRotationMesh);

        const saturnRotation = new Rotation(saturnMesh);
        const saturnRotationMesh = saturnRotation.getMesh();
        saturnSystem.add(saturnRotationMesh);

        const uranusRotation = new Rotation(uranusMesh);
        const uranusRotationMesh = uranusRotation.getMesh();
        uranusSystem.add(uranusRotationMesh);

        const neptuneRotation = new Rotation(neptuneMesh);
        const neptuneRotationMesh = neptuneRotation.getMesh();
        neptuneSystem.add(neptuneRotationMesh);


        initGui();
        const solarSystemGui = gui.addFolder('Solar System');
        solarSystemGui.add(mercuryRotationMesh, "visible").name("mercury").listen();
        solarSystemGui.add(venusRotationMesh, "visible").name("venus").listen();
        solarSystemGui.add(earthRotationMesh, "visible").name("earth").listen();
        solarSystemGui.add(marsRotationMesh, "visible").name("mars").listen();
        solarSystemGui.add(jupiterRotationMesh, "visible").name("jupiter").listen();
        solarSystemGui.add(saturnRotationMesh, "visible").name("saturn").listen();
        solarSystemGui.add(uranusRotationMesh, "visible").name("uranus").listen();
        solarSystemGui.add(neptuneRotationMesh, "visible").name("neptune").listen();


        const EarthRotation = 2 * Math.PI * (1/ 60) * (1/ 60);
        const animate = () => {
            sun.rotation.y +=  0.001;
            mercurySystem.rotation.y += EarthRotation * 4;
            venusSystem.rotation.y += EarthRotation * 2;
            earthSystem.rotation.y += EarthRotation;
            marsSystem.rotation.y += EarthRotation * 0.5;
            jupiterSystem.rotation.y += EarthRotation * 2.5;
            saturnSystem.rotation.y += EarthRotation * 1.5;
            uranusSystem.rotation.y += EarthRotation * 1.2;
            neptuneSystem.rotation.y += EarthRotation * 1.1;
            test.renderer.render( test.scene, test.camera );
            requestAnimationFrame( animate );
        };
        animate();
    }, []);
</script>

<div class="flex justify-center items-center h-screen">
    <canvas id="myThreeJsCanvas"></canvas>
</div>
