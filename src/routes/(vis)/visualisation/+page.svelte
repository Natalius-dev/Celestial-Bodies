<script>
    import * as THREE from "three";
    import { OrbitControls } from "three/addons/controls/OrbitControls.js";
    import { onMount } from "svelte";
    import "../../../app.css";

    function degrees_to_radians(degrees)
    {
        return degrees * (Math.PI/180);
    }

    onMount(() => {
        const params = new URLSearchParams(location.search);
        const system = JSON.parse(localStorage.getItem(params.get("system")));

        // Setup 3D scene
        const renderer = new THREE.WebGLRenderer({canvas: document.getElementById("graphics"), antialias: true, powerPreference: "high-performance"});
        const scene = new THREE.Scene();
        const parent = document.getElementById("graphicsParent");
        const camera = new THREE.PerspectiveCamera( 75, parent.clientWidth/parent.clientHeight, 0.01, 10000 );
        const controls = new OrbitControls( camera, renderer.domElement );
        controls.zoomSpeed = 1.5;
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize( parent.clientWidth, parent.clientHeight );
        parent.appendChild( renderer.domElement );

        window.addEventListener('resize', function(){
            camera.aspect = parent.clientWidth/parent.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(parent.clientWidth, parent.clientHeight);
        });

        let orbitMeshes = {};
        function generateOrbit(semi_major, eccentricity, inclination, name) {
            const centerX = (semi_major*eccentricity); const centerY = 0;
            const semi_minor = semi_major*(Math.sqrt(1-(Math.pow(eccentricity,2))));
            const angle = degrees_to_radians(inclination);

            let path = new THREE.EllipseCurve(
                centerX, centerY,
                semi_major, semi_minor,
                0, 2*Math.PI,
                false,
                0
            );
            let ellipse = path.getSpacedPoints(objects[name+" speed"]);

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
            for(let i = 0; i < objects[name+" speed"]; i++) {
                let pos = new THREE.Vector3(0,0,0);
                pos.x = ellipse[i].x * xAxis.x;
                pos.y = ellipse[i].x * xAxis.y;
                pos.z = ellipse[i].x * xAxis.z;

                pos.x = pos.x + ellipse[i].y * yAxis.x;
                pos.y = pos.y + ellipse[i].y * yAxis.y;
                pos.z = pos.z + ellipse[i].y * yAxis.z;

                pathArray.push(pos);
            }

            const orbitMesh = new THREE.Line(new THREE.BufferGeometry().setFromPoints( pathArray ), new THREE.LineBasicMaterial({color:"#ffffff",linewidth:1}) );
            orbitMeshes[name] = orbitMesh;
            scene.add(orbitMesh);

            return pathArray;
        }

        scene.add(new THREE.AmbientLight("#ffffff", 0.15));
        let names = [];
        let objects = {};

        const speedModifier = 1200;

        function load(obj) {
            const texture = new THREE.TextureLoader().load(obj.texture);
            names.push(obj.name);
            objects[obj.name+" speed"] = (obj.a !== 0) ? (Math.round((((Math.pow(Math.log10(obj.a), 6))/2000)+0.5)*speedModifier)) : 1;
            objects[obj.name+" orbit"] = generateOrbit(obj.a !== 0 ? (((Math.pow(Math.log10(obj.a), 6))/2000)+0.5) : 0, obj.e, obj.i, obj.name);
            objects[obj.name+" body"] = new THREE.Mesh(new THREE.IcosahedronGeometry(obj.radius !== 0 ? ((Math.pow(( Math.log10(obj.radius) < 0 ? 1 : Math.log10(obj.radius) ), 1.5)/8)+0.05) : 0, 8), new THREE.MeshStandardMaterial({map: texture}));
            if(obj.normal !== undefined) {
                const normal = new THREE.TextureLoader().load(obj.normal);
                objects[obj.name+" body"].material.normalMap = normal;
            }
            objects[obj.name+" body"].position.set(objects[obj.name+" orbit"][obj.shift === undefined ? 0 : Math.round(obj.shift*objects[obj.name+" speed"])].x, objects[obj.name+" orbit"][obj.shift === undefined ? 0 : Math.round(obj.shift*objects[obj.name+" speed"])].y, objects[obj.name+" orbit"][obj.shift === undefined ? 0 : Math.round(obj.shift*objects[obj.name+" speed"])].z);
            objects[obj.name+" body"].receiveShadow = true;
            objects[obj.name+" index"] = (obj.shift === undefined ? 0 : Math.round(obj.shift*objects[obj.name+" speed"]));
            objects[obj.name+" parent"] = obj.parent;
            scene.add(objects[obj.name+" body"]);
            if(obj.brightness !== undefined) {
                let light = new THREE.PointLight("#ffffff", obj.brightness);
                light.castShadow = true;
                light.shadow.mapSize = new THREE.Vector2(8, 8);
                objects[obj.name+" body"].material = new THREE.MeshBasicMaterial({map: texture});
                objects[obj.name+" body"].receiveShadow = false;
                objects[obj.name+" body"].add(light);
            }
            if(obj.children.length !== 0) {
                for(let i = 0; i < obj.children.length; i++) {
                    obj.children[i].parent = obj.name;
                    load(obj.children[i]);
                }
            }
        }
        camera.position.x = 5;
        camera.position.y = 5;
        camera.position.z = 7;

        function animate() {
            for(let i = 0; i < names.length; i++) {
                orbitMeshes[names[i]].position.set(objects[objects[names[i]+" parent"]+" body"] === undefined ? 0 : objects[objects[names[i]+" parent"]+" body"].position.x, objects[objects[names[i]+" parent"]+" body"] === undefined ? 0 : objects[objects[names[i]+" parent"]+" body"].position.y, objects[objects[names[i]+" parent"]+" body"] === undefined ? 0 : objects[objects[names[i]+" parent"]+" body"].position.z);
                if(objects[names[i]+" index"] === objects[names[i]+" speed"]-1) {
                    objects[names[i]+" index"] = 0;
                } else {
                    objects[names[i]+" index"] += 1;
                }
                let localPoint = objects[names[i]+" orbit"][objects[names[i]+" index"]];
                let parent = objects[names[i]+" parent"];
                objects[names[i]+" body"].position.set(
                    localPoint.x + (parent === undefined ? 0 : (objects[parent+" orbit"][objects[parent+" index"]]).x),
                    localPoint.y + (parent === undefined ? 0 : (objects[parent+" orbit"][objects[parent+" index"]]).y),
                    localPoint.z + (parent === undefined ? 0 : (objects[parent+" orbit"][objects[parent+" index"]]).z)
                );
            }
            controls.target = objects["Planet 3 body"].position;
            requestAnimationFrame( animate );
            controls.update();
            renderer.render( scene, camera );
        }
        load(system);
        console.log(objects);
        animate();
    });
</script>

<div id="graphicsParent" class="flex-grow w-full h-full"><canvas id="graphics" class="!w-full !h-full"></canvas></div>