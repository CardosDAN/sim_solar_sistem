<script>
    import * as THREE from 'three';
    import { onMount } from 'svelte';
    import * as dat from 'dat.gui';
    import SceneInit from "./entities/SceneInit.js";
    import Planet from "./entities/Planet.js";
    import Rotation from "./entities/Rotation.js";


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

        const mars = new Planet(3, 64, 'mars.jpeg');
        const marsMesh = mars.getMesh();
        let marsSystem = new THREE.Group();
        marsSystem.add(marsMesh);

        solarSystem.add(mercurySystem, venusSystem, earthSystem, marsSystem);

        const EarthRotation = 2 * Math.PI * (1/ 60) * (1/ 60);
        const animate = () => {
            sun.rotation.y +=  0.001;
            mercurySystem.rotation.y += EarthRotation * 4;
            venusSystem.rotation.y += EarthRotation * 2;
            earthSystem.rotation.y += EarthRotation;
            marsSystem.rotation.y += EarthRotation * 0.5;
            test.renderer.render( test.scene, test.camera );
            requestAnimationFrame( animate );
        };
        animate();
    }, []);
</script>

<div class="flex justify-center items-center h-screen">
    <canvas id="myThreeJsCanvas"></canvas>
</div>
