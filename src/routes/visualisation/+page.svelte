<script>
    import * as THREE from "three";
    import { OrbitControls } from "three/addons/controls/OrbitControls.js";
    import { onMount } from "svelte";
    import "../../app.css";

    function degrees_to_radians(degrees)
    {
        return degrees * (Math.PI/180);
    }

    onMount(() => {
        const params = new URLSearchParams(location.search);
        const system = JSON.parse(localStorage.getItem(params.get("system")));

        // Setup 3D scene
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.01, 10000 );
        
        const renderer = new THREE.WebGLRenderer();
        const controls = new OrbitControls( camera, renderer.domElement );
        renderer.setSize( window.innerWidth, window.innerHeight );
        renderer.shadowMap.enabled = true;
        document.body.appendChild( renderer.domElement );

        function generateOrbit(semi_major, eccentricity, inclination) {
            const centerX = (semi_major*eccentricity); const centerY = 0;
            const semi_minor = semi_major*(Math.sqrt(1-(Math.pow(eccentricity,2))));
            const angle = degrees_to_radians(inclination);
            const orbitQuality = 500;

            let path = new THREE.EllipseCurve(
                centerX, centerY,
                semi_major, semi_minor,
                0, 2*Math.PI,
                false,
                0
            );
            let ellipse = [];
            const period = Math.pow(semi_major, (3/2));
            for(let i = 0; i < orbitQuality; i++) {
                ellipse.push(path.getPointAt(i/orbitQuality));
            }

            const xAxis = { x : 1, y : 0, z : 0 }; // define the x axis
            const yAxis = { x : 0, y : 1, z : 0 }; // define the y axis
            const zAxis = { x : 0, y : 0, z : 1 }; // define the z axis

            // get the x axis at 45 deg
            xAxis.x = Math.cos(angle);
            xAxis.y = -Math.sin(angle);
            xAxis.z = 0;

            // get the y axis at 90 deg (PI / 2) from the x axis
            zAxis.x = Math.sin(angle);
            zAxis.y = Math.cos(angle);
            zAxis.z = 0;

            // get the y axis at 90 deg (PI / 2) from the x axis
            yAxis.x = 0;
            yAxis.y = 0;
            yAxis.z = 1;

            // Converts 2D coordinates in ellipse variable to 3D coordinates
            let pathArray = [];
            for(let i = 0; i < orbitQuality; i++) {
                let pos = new THREE.Vector3(0,0,0);
                pos.x = ellipse[i].x * xAxis.x;
                pos.y = ellipse[i].x * xAxis.y;
                pos.z = ellipse[i].x * xAxis.z;

                pos.x = pos.x + ellipse[i].y * yAxis.x;
                pos.y = pos.y + ellipse[i].y * yAxis.y;
                pos.z = pos.z + ellipse[i].y * yAxis.z;
                pathArray.push(pos);
                const node = new THREE.Mesh(new THREE.BoxGeometry(0.01, 0.01, 0.01), new THREE.MeshBasicMaterial({color:"#ff0000"}));
                node.position.set(pos.x, pos.y, pos.z);
                scene.add(node);
            }

            const orbitMesh = new THREE.Line(new THREE.BufferGeometry().setFromPoints( pathArray ), new THREE.LineBasicMaterial({color:"#ffffff",linewidth:1}) );
            scene.add(orbitMesh);

            return pathArray;
        }

        scene.add(new THREE.AmbientLight("#ffffff", 0.3));
        const orbit = generateOrbit(4, 0.99, 20);
        let i = 0;

        camera.position.z = 15;
        camera.position.y = 0;

        const cube = new THREE.Mesh(new THREE.BoxGeometry(0.15,0.15,0.15), new THREE.MeshStandardMaterial({color:"#00ff00"}));
        scene.add(cube);

        function animate() {
            requestAnimationFrame( animate );
            controls.update();
            cube.position.set(orbit[i].x, orbit[i].y, orbit[i].z);
            i < 499 ? i += 1 : i = 0;
            renderer.render( scene, camera );
        }
        animate();
    });
</script>

<h1 class="bg-slate-800">AAAA</h1>