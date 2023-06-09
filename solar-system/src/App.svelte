<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';
    import {OrbitControls} from "three/examples/jsm/controls/OrbitControls.js";
    import * as dat from 'dat.gui';

    let gui;

    const initGui = () => {
        gui = new dat.GUI();
    }

	onMount(() => {
		// canvas pentru a afișa simularea
		const canvas = document.createElement('canvas');
		document.body.appendChild(canvas);

		// scena este locul unde se desfășoară simularea
		const scene = new THREE.Scene();

		// Creez o cameră
		const camera = new THREE.PerspectiveCamera(
				60, // Unghiul de vizualizare
				window.innerWidth / window.innerHeight, // Raportul de aspect
				0.1, // Planul de apropiere
				4000 // Planul de depărtare
		);
		camera.position.set(0, 30, 500);

		// Creez un renderer
		const renderer = new THREE.WebGLRenderer({ canvas });
		renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

		// Adaug o lumină ambientală
		const ambientLight = new THREE.AmbientLight(0x222222);
		scene.add(ambientLight);

        const pointLight = new THREE.PointLight(0xffffff);
        scene.add(pointLight);

        //OrbitControls pentru a putea roti camera cu mouse-ul
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;

        // *****************************************************************************************
        // *****************************************************************************************
        // Creez sistemul Solar
        // *****************************************************************************************

        // Creez texture loader pentru a incarca texturile
        const textureLoader = new THREE.TextureLoader();

        // *****************************************************************************************
        // Fundalul
        scene.background = textureLoader.load('images/background/space.jpg');

        // *****************************************************************************************
        // Soarele
        const sunMaterial = new THREE.MeshStandardMaterial({
            emissive: 0xffd700, // culoarea straluciri
            emissiveMap: textureLoader.load('images/sun/8k_sun.jpg'),
            emissiveIntensity: 1, // intensitatea straluciri
        });
        const sunGeometry = new THREE.SphereGeometry(109, 400, 200); // geometria Soarelui
        const sun = new THREE.Mesh(sunGeometry, sunMaterial); // creez Soarele
        scene.add(sun); // adaug Soarele în scena

        // *****************************************************************************************
        // Mercur

        const mercuryMaterial = new THREE.MeshPhongMaterial({ // materialul Mercurului
            map: textureLoader.load('images/mercury/mercurymap.jpg'),
            bumpMap: textureLoader.load('images/mercury/mercurybump.jpg'),
            bumpScale: 0.05
        });
        const mercuryGeometry = new THREE.SphereGeometry(10, 40);
        const mercury = new THREE.Mesh(mercuryGeometry, mercuryMaterial);
        scene.add(mercury); // adaug Mercurul în scena

        //axa de rotație a Mercurului
        const mercuryAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const mercuryAxisGeometry = new THREE.BufferGeometry().setFromPoints(mercuryAxisPoints); // geometria axei de rotație
        const mercuryAxis = new THREE.Line(mercuryAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(mercuryAxis); // adaug axa de rotație în scena

        // *****************************************************************************************

        // Venus
        const venusMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/venus/venusmap.jpg'),
            bumpMap: textureLoader.load('images/venus/venusbump.jpg'),
            bumpScale: 0.05
        });
        const venusGeometry = new THREE.SphereGeometry(12, 40);
        const venus = new THREE.Mesh(venusGeometry, venusMaterial);
        scene.add(venus); // adaug Venus în scena

        //axa de rotație a Venusului
        const venusAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const venusAxisGeometry = new THREE.BufferGeometry().setFromPoints(venusAxisPoints); // geometria axei de rotație
        const venusAxis = new THREE.Line(venusAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(venusAxis); // adaug axa de rotație în scena


        // *****************************************************************************************

        //Pământ
        const earthSystem = new THREE.Group();

        const radius = 25; // raza Pământului
        const segments = 50; // numarul de segmente
        const tilt = 0.41; // înclinarea axei de rotație

        const earthColor = textureLoader.load('images/earth/earthmap1k.jpg');
        const earthBump = textureLoader.load('images/earth/earthbump1k.jpg');
        const earthSpecular = textureLoader.load('images/earth/earthspec1k.jpg');

        const earthGeometry = new THREE.SphereGeometry(radius, segments, segments);
        const earthMaterial = new THREE.MeshPhongMaterial({
            map: earthColor, // textura
            bumpMap: earthBump,// textura pentru bump
            bumpScale: 0.05, // scalarea bump-ului
            specularMap: earthSpecular, // textura pentru specular
            shininess: 0.5 // stralucirea
        });
        const earth = new THREE.Mesh(earthGeometry, earthMaterial);
        earth.rotation.z = tilt;
        earthSystem.add(earth); // adaug Pământul în sistemul solar
        // Norii Pământului
        const cloudGeometry = new THREE.SphereGeometry(radius + 1, segments, segments);
        const cloudMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/earth/earthcloudmap.jpg'),
            transparent: true,
            opacity: 0.5
        });
        const cloud = new THREE.Mesh(cloudGeometry, cloudMaterial);
        cloud.rotation.z = tilt;
        earthSystem.add(cloud); // adaug norii în sistemul solar

        // Axa de rotație a Pământului
        const axisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const axisGeometry = new THREE.BufferGeometry().setFromPoints(axisPoints); // geometria axei de rotație
        const axis = new THREE.Line(axisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        axis.rotation.z = tilt; // înclinarea axei de rotație
        earthSystem.add(axis); // adaug axa de rotație în sistemul solar

        // Luna
        const moonGeometry = new THREE.SphereGeometry(5, 40, 20);
        const moonMaterial = new THREE.MeshStandardMaterial({
            map: textureLoader.load('images/moon/moonmap1k.jpg'),
            bumpMap: textureLoader.load('images/moon/moonbump1k.jpg'),
            bumpScale: 0.5
        });
        const moon = new THREE.Mesh(moonGeometry, moonMaterial);
        moon.position.set(40, 0, 0); // poziția Lunii
        earthSystem.add(moon); // adaug Luna în sistemul solar

        scene.add(earthSystem); // adaug sistemul solar în scena

        // *****************************************************************************************

        // Marte
        const marsMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/mars/marsmap1k.jpg'),
            bumpMap: textureLoader.load('images/mars/marsbump1k.jpg'),
            bumpScale: 0.05
        });
        const marsGeometry = new THREE.SphereGeometry(15, 40);
        const mars = new THREE.Mesh(marsGeometry, marsMaterial);
        scene.add(mars); // adaug Marte în scena

        //axa de rotație a Marte
        const marsAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const marsAxisGeometry = new THREE.BufferGeometry().setFromPoints(marsAxisPoints); // geometria axei de rotație
        const marsAxis = new THREE.Line(marsAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(marsAxis); // adaug axa de rotație în scena

        // *****************************************************************************************

        // Jupiter

        const jupiterMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/jupiter/jupitermap.jpg'),
        });

        const jupiterGeometry = new THREE.SphereGeometry(50, 40);
        const jupiter = new THREE.Mesh(jupiterGeometry, jupiterMaterial);
        scene.add(jupiter); // adaug Jupiter în scena

        //axa de rotație a lui Jupiter
        const jupiterAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const jupiterAxisGeometry = new THREE.BufferGeometry().setFromPoints(jupiterAxisPoints); // geometria axei de rotație
        const jupiterAxis = new THREE.Line(jupiterAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(jupiterAxis); // adaug axa de rotație în scena

        // *****************************************************************************************

        // Saturn

        const saturnMaterial = new THREE.MeshPhongMaterial({ // materialul lui Saturn
            map: textureLoader.load('images/saturn/saturnmap.jpg'),
        });

        const saturnGeometry = new THREE.SphereGeometry(40, 40);
        const saturn = new THREE.Mesh(saturnGeometry, saturnMaterial);
        scene.add(saturn); // adaug Saturn în scena

        //axa de rotație a lui Saturn
        const saturnAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const saturnAxisGeometry = new THREE.BufferGeometry().setFromPoints(saturnAxisPoints); // geometria axei de rotație
        const saturnAxis = new THREE.Line(saturnAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(saturnAxis); // adaug axa de rotație în scena

        // Inelele lui Saturn

        const saturnRingsGeometry = new THREE.RingGeometry(53, 65, 40);
        const saturnRingsMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/saturn/saturnringcolor.jpg'),
            side: THREE.DoubleSide,
            opacity: 0.5,
            transparent: 0.5,
            shininess: 100
        });
        const saturnRings = new THREE.Mesh(saturnRingsGeometry, saturnRingsMaterial);
        saturnRings.rotation.x = Math.PI / 4; // Înclinarea inelelor lui Saturn cu 45 de grade
        saturn.add(saturnRings); // adaug inelele lui Saturn în scena

        // *****************************************************************************************

        // Uranus

        const uranusMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/uranus/uranusmap.jpg'),
        });

        const uranusGeometry = new THREE.SphereGeometry(30, 40);
        const uranus = new THREE.Mesh(uranusGeometry, uranusMaterial);
        scene.add(uranus); // adaug Uranus în scena

        //axa de rotație a lui Uranus
        const uranusAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const uranusAxisGeometry = new THREE.BufferGeometry().setFromPoints(uranusAxisPoints); // geometria axei de rotație
        const uranusAxis = new THREE.Line(uranusAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(uranusAxis); // adaug axa de rotație în scena

        // Inelele lui Uranus

        const uranusRingsGeometry = new THREE.RingGeometry(33, 40, 40);
        const uranusRingsMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/uranus/uranusringcolour.jpg'),
            side: THREE.DoubleSide,
            opacity: 0.5,
            transparent: true,
            shininess: 100
        });

        const uranusRings = new THREE.Mesh(uranusRingsGeometry, uranusRingsMaterial);
        uranusRings.rotation.x = Math.PI / 4; // Înclinarea inelelor lui Uranus cu 45 de grade
        uranus.add(uranusRings); // adaug inelele lui Uranus în scena

        // *****************************************************************************************

        // Neptun

        const neptuneMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/neptune/neptunemap.jpg'),
        });

        const neptuneGeometry = new THREE.SphereGeometry(30, 40);
        const neptune = new THREE.Mesh(neptuneGeometry, neptuneMaterial);
        scene.add(neptune); // adaug Neptun în scena

        //axa de rotație a lui Neptun

        const neptuneAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const neptuneAxisGeometry = new THREE.BufferGeometry().setFromPoints(neptuneAxisPoints); // geometria axei de rotație
        const neptuneAxis = new THREE.Line(neptuneAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(neptuneAxis); // adaug axa de rotație în scena

        // *****************************************************************************************

        // Pluto

        const plutoMaterial = new THREE.MeshPhongMaterial({
            map: textureLoader.load('images/pluto/plutomap1k.jpg'),
            bumpMap: textureLoader.load('images/pluto/plutobump1k.jpg'),
            bumpScale: 0.05
        });

        const plutoGeometry = new THREE.SphereGeometry(10, 40);
        const pluto = new THREE.Mesh(plutoGeometry, plutoMaterial);
        scene.add(pluto); // adaug Pluto în scena

        //axa de rotație a lui Pluto

        const plutoAxisPoints = [
            new THREE.Vector3(0, 35, 0),
            new THREE.Vector3(0, -35, 0)
        ];

        const plutoAxisGeometry = new THREE.BufferGeometry().setFromPoints(plutoAxisPoints); // geometria axei de rotație
        const plutoAxis = new THREE.Line(plutoAxisGeometry, new THREE.LineBasicMaterial({ color: 0x330000, transparent: true, opacity: 0.5 })); // axa de rotație
        scene.add(plutoAxis); // adaug axa de rotație în scena

        // *****************************************************************************************
        // SadowMap

        renderer.shadowMap.enabled = true;
        pointLight.castShadow = true;

        // setez proprietatile pt point light
        pointLight.shadow.mapSize.width = 512;
        pointLight.shadow.mapSize.height = 512;
        pointLight.shadow.camera.near = 150;
        pointLight.shadow.camera.far = 350;

        // cum pot obiectele interactiona cu umbra
        earth.castShadow = true;
        earth.receiveShadow = true;
        cloud.receiveShadow = true;
        moon.castShadow = true;
        moon.receiveShadow = true;

        // *****************************************************************************************

        //Orbita Mercurului

        const mercuryCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            150, 150,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const mercuryPoints = mercuryCurve.getSpacedPoints( 200 );

        const mercuryOrbitGeometry = new THREE.BufferGeometry().setFromPoints( mercuryPoints ); // geometria orbitei
        const mercuryOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const mercuryOrbit = new THREE.Line( mercuryOrbitGeometry, mercuryOrbitMaterial );
        mercuryOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( mercuryOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        //Orbita Venusului

        const venusCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            200, 200,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const venusPoints = venusCurve.getSpacedPoints( 200 );

        const venusOrbitGeometry = new THREE.BufferGeometry().setFromPoints( venusPoints ); // geometria orbitei
        const venusOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const venusOrbit = new THREE.Line( venusOrbitGeometry, venusOrbitMaterial );
        venusOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( venusOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        // Orbita Pământului

        const curve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            270, 320,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const points = curve.getSpacedPoints( 200 );

        const orbitGeometry = new THREE.BufferGeometry().setFromPoints( points ); // geometria orbitei
        const orbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const orbit = new THREE.Line( orbitGeometry, orbitMaterial );
        orbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( orbit ); // adaug orbita in scena

        // *****************************************************************************************

        // Orbita Marte

        const marsCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            350, 400,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const marsPoints = marsCurve.getSpacedPoints( 200 );

        const marsOrbitGeometry = new THREE.BufferGeometry().setFromPoints( marsPoints ); // geometria orbitei
        const marsOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const marsOrbit = new THREE.Line( marsOrbitGeometry, marsOrbitMaterial );
        marsOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( marsOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        // Orbita lui Jupiter

        const jupiterCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            500, 550,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const jupiterPoints = jupiterCurve.getSpacedPoints( 200 );

        const jupiterOrbitGeometry = new THREE.BufferGeometry().setFromPoints( jupiterPoints ); // geometria orbitei
        const jupiterOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const jupiterOrbit = new THREE.Line( jupiterOrbitGeometry, jupiterOrbitMaterial );
        jupiterOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( jupiterOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        // Orbita lui Saturn

        const saturnCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            700, 750,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const saturnPoints = saturnCurve.getSpacedPoints( 200 );

        const saturnOrbitGeometry = new THREE.BufferGeometry().setFromPoints( saturnPoints ); // geometria orbitei
        const saturnOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const saturnOrbit = new THREE.Line( saturnOrbitGeometry, saturnOrbitMaterial );
        saturnOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( saturnOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        // Orbita lui Uranus

        const uranusCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            900, 950,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const uranusPoints = uranusCurve.getSpacedPoints( 200 );

        const uranusOrbitGeometry = new THREE.BufferGeometry().setFromPoints( uranusPoints ); // geometria orbitei
        const uranusOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const uranusOrbit = new THREE.Line( uranusOrbitGeometry, uranusOrbitMaterial );
        uranusOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( uranusOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        // Orbita lui Neptun

        const neptuneCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            1100, 1150,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const neptunePoints = neptuneCurve.getSpacedPoints( 200 );

        const neptuneOrbitGeometry = new THREE.BufferGeometry().setFromPoints( neptunePoints ); // geometria orbitei
        const neptuneOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const neptuneOrbit = new THREE.Line( neptuneOrbitGeometry, neptuneOrbitMaterial );
        neptuneOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( neptuneOrbit ); // adaug orbita in scena

        // *****************************************************************************************
        const loopTime = 1; // timpul de rotatie al Pământului
        const earthOrbitSpeed = 0.00001; // viteza de rotatie a Pământului
        const moonOrbitRadius = 55; // raza orbitei Lunii
        const moonOrbitSpeed = 80; // viteza de rotatie a Lunii

        // *****************************************************************************************

        // Orbita lui Pluto

        const plutoCurve = new THREE.EllipseCurve(
            0, 0,            // axa x, y
            1300, 1350,           // raza x, y
            0, 2 * Math.PI,  // unghiul de start si unghiul final
        );

        const plutoPoints = plutoCurve.getSpacedPoints( 200 );

        const plutoOrbitGeometry = new THREE.BufferGeometry().setFromPoints( plutoPoints ); // geometria orbitei
        const plutoOrbitMaterial = new THREE.LineBasicMaterial( { color : 0x333333, transparent: true, opacity: 0.5 } ); // materialul orbitei

        const plutoOrbit = new THREE.Line( plutoOrbitGeometry, plutoOrbitMaterial );
        plutoOrbit.rotateX( -Math.PI / 2 ); // rotesc orbita
        scene.add( plutoOrbit ); // adaug orbita in scena

        // *****************************************************************************************

        let selectedPlanet; // planeta selectata

        // functie pentru a selecta o planeta
        function focusCameraOnPlanet(planet) {
            selectedPlanet = planet;
        }

        initGui(); // apelez functia pentru interfata grafica
       // adaug optiunile pentru selectie planeta in interfata grafica

        const planetOptions = {
            "Mercury": function() { focusCameraOnPlanet(mercury); },
            "Venus": function() { focusCameraOnPlanet(venus); },
            "Earth": function() { focusCameraOnPlanet(earthSystem); },
            "Mars": function() { focusCameraOnPlanet(mars); },
            "Jupiter": function() { focusCameraOnPlanet(jupiter); },
            "Saturn": function() { focusCameraOnPlanet(saturn); },
            "Uranus": function() { focusCameraOnPlanet(uranus); },
            "Neptune": function() { focusCameraOnPlanet(neptune); },
            "Pluto": function() { focusCameraOnPlanet(pluto); },
        };

        gui.add(planetOptions, "Mercury").name("Mercury").listen();
        gui.add(planetOptions, "Venus").name("Venus").listen();
        gui.add(planetOptions, "Earth").name("Earth").listen();
        gui.add(planetOptions, "Mars").name("Mars").listen();
        gui.add(planetOptions, "Jupiter").name("Jupiter").listen();
        gui.add(planetOptions, "Saturn").name("Saturn").listen();
        gui.add(planetOptions, "Uranus").name("Uranus").listen();
        gui.add(planetOptions, "Neptune").name("Neptune").listen();
        gui.add(planetOptions, "Pluto").name("Pluto").listen();



        // *****************************************************************************************
		// Funția pentru animație
		function animate() {
            // *****************************************************************************************

            // rotatia Mercurului in jurul soarelui

            const mercuryOrbitSpeed = 0.0001; // viteza de rotatie a Mercurului

            const mercuryTime = mercuryOrbitSpeed * performance.now(); // timpul de rotatie al Mercurului
            const mercuryT = mercuryTime % loopTime;

            let mercuryP = mercuryCurve.getPoint(mercuryT); // punctul de pe orbita
            console.log(mercuryP, mercuryT);

            mercury.position.x = mercuryP.x; // pozitia Mercurului pe orbita
            mercury.position.z = mercuryP.y; // pozitia Mercurului pe orbita

            // *****************************************************************************************

            // rotatia Venusului in jurul soarelui

            const venusOrbitSpeed = 0.00005; // viteza de rotatie a Venusului

            const venusTime = venusOrbitSpeed * performance.now(); // timpul de rotatie al Venusului
            const venusT = venusTime % loopTime;

            let venusP = venusCurve.getPoint(venusT); // punctul de pe orbita
            console.log(venusP, venusT);

            venus.position.x = venusP.x; // pozitia Venusului pe orbita
            venus.position.z = venusP.y; // pozitia Venusului pe orbita

            // *****************************************************************************************

            // rotatia Pământului in jurul soarelui

            const time = earthOrbitSpeed * performance.now(); // timpul de rotatie al Pământului
            const t = time % loopTime;

            let p = curve.getPoint(t); // punctul de pe orbita
            console.log(p, t);

            earthSystem.position.x = p.x; // pozitia Pământului pe orbita
            earthSystem.position.z = p.y; // pozitia Pământului pe orbita

            // *****************************************************************************************
            // rotatia Lunii in jurul Pământului
            moon.position.x = -Math.cos(time * moonOrbitSpeed) * moonOrbitRadius; // pozitia Lunii pe orbita
		    moon.position.z = -Math.sin(time * moonOrbitSpeed) * moonOrbitRadius; // pozitia Lunii pe orbita

            // *****************************************************************************************

            // rotatia Marte in jurul soarelui

            const marsOrbitSpeed = 0.00003; // viteza de rotatie a Marte

            const marsTime = marsOrbitSpeed * performance.now(); // timpul de rotatie al Marte
            const marsT = marsTime % loopTime;

            let marsP = marsCurve.getPoint(marsT); // punctul de pe orbita
            console.log(marsP, marsT);

            mars.position.x = marsP.x; // pozitia Marte pe orbita
            mars.position.z = marsP.y; // pozitia Marte pe orbita

            // *****************************************************************************************

            // rotatia lui Jupiter in jurul soarelui

            const jupiterOrbitSpeed = 0.00002; // viteza de rotatie a lui Jupiter

            const jupiterTime = jupiterOrbitSpeed * performance.now(); // timpul de rotatie al lui Jupiter
            const jupiterT = jupiterTime % loopTime;

            let jupiterP = jupiterCurve.getPoint(jupiterT); // punctul de pe orbita
            console.log(jupiterP, jupiterT);

            jupiter.position.x = jupiterP.x; // pozitia lui Jupiter pe orbita
            jupiter.position.z = jupiterP.y; // pozitia lui Jupiter pe orbita

            // *****************************************************************************************

            // rotatia lui Saturn in jurul soarelui

            const saturnOrbitSpeed = 0.00001; // viteza de rotatie a lui Saturn

            const saturnTime = saturnOrbitSpeed * performance.now(); // timpul de rotatie al lui Saturn
            const saturnT = saturnTime % loopTime;

            let saturnP = saturnCurve.getPoint(saturnT); // punctul de pe orbita
            console.log(saturnP, saturnT);

            saturn.position.x = saturnP.x; // pozitia lui Saturn pe orbita
            saturn.position.z = saturnP.y; // pozitia lui Saturn pe orbita

            // *****************************************************************************************

            // rotatia lui Uranus in jurul soarelui

            const uranusOrbitSpeed = 0.000005; // viteza de rotatie a lui Uranus

            const uranusTime = uranusOrbitSpeed * performance.now(); // timpul de rotatie al lui Uranus
            const uranusT = uranusTime % loopTime;

            let uranusP = uranusCurve.getPoint(uranusT); // punctul de pe orbita
            console.log(uranusP, uranusT);

            uranus.position.x = uranusP.x; // pozitia lui Uranus pe orbita
            uranus.position.z = uranusP.y; // pozitia lui Uranus pe orbita

            // *****************************************************************************************

            // rotatia lui Neptun in jurul soarelui

            const neptuneOrbitSpeed = 0.000002; // viteza de rotatie a lui Neptun

            const neptuneTime = neptuneOrbitSpeed * performance.now(); // timpul de rotatie al lui Neptun
            const neptuneT = neptuneTime % loopTime;

            let neptuneP = neptuneCurve.getPoint(neptuneT); // punctul de pe orbita
            console.log(neptuneP, neptuneT);

            neptune.position.x = neptuneP.x; // pozitia lui Neptun pe orbita
            neptune.position.z = neptuneP.y; // pozitia lui Neptun pe orbita

            // *****************************************************************************************

            // rotatia lui Pluto in jurul soarelui

            const plutoOrbitSpeed = 0.000001; // viteza de rotatie a lui Pluto

            const plutoTime = plutoOrbitSpeed * performance.now(); // timpul de rotatie al lui Pluto
            const plutoT = plutoTime % loopTime;

            let plutoP = plutoCurve.getPoint(plutoT); // punctul de pe orbita
            console.log(plutoP, plutoT);

            pluto.position.x = plutoP.x; // pozitia lui Pluto pe orbita
            pluto.position.z = plutoP.y; // pozitia lui Pluto pe orbita

            // *****************************************************************************************

            // roatatia mercur
            mercury.rotation.y += 0.001; // rotatia mercur

            // *****************************************************************************************

            // rotatia venus
            venus.rotation.y += 0.0005; // rotatia venus

            // *****************************************************************************************

            // rotatia  pamantului
            earth.rotation.y += 0.0015; // rotatia sistemului solar

            // *****************************************************************************************

            // rotatia Marte
            mars.rotation.y += 0.0005; // rotatia Marte

            // *****************************************************************************************

            // rotatia lui Jupiter
            jupiter.rotation.y += 0.0005; // rotatia lui Jupiter

            // *****************************************************************************************

            // rotatia lui Saturn
            saturn.rotation.y += 0.0005; // rotatia lui Saturn

            // *****************************************************************************************

            // rotatia lui Uranus
            uranus.rotation.y += 0.0005; // rotatia lui Uranus

            // *****************************************************************************************

            // rotatia lui Neptun
            neptune.rotation.y += 0.0005; // rotatia lui Neptun

            // *****************************************************************************************

            // rotatia lui Pluto
            pluto.rotation.y += 0.0005; // rotatia lui Pluto

            // *****************************************************************************************
            // rotatia norilor
            cloud.rotation.y += 0.0025; // rotatia norilor

            // *****************************************************************************************
            // rotatia Lunii
            moon.rotation.y += 0.0005; // rotatia Lunii

            // *****************************************************************************************
            // rotatia soarelui
            sun.rotation.y += 0.0008; // rotatia soarelui

            if (selectedPlanet){
                camera.position.set(selectedPlanet.position.x, selectedPlanet.position.y, selectedPlanet.position.z + 100);

                // Actualizați orientarea camerei
                camera.lookAt(selectedPlanet.position);
            }
            else {
               camera.lookAt(scene.position);

            }

            renderer.render(scene, camera); // randez scena
            requestAnimationFrame(animate); // apelez functia animate
        }

		animate();
	});
</script>

<style>
	canvas {
		display: block;
	}
</style>
