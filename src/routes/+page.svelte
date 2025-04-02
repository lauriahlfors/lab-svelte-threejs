<script lang="ts">
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import { RoomEnvironment } from 'three/examples/jsm/Addons.js';

	// Canvas
	let canvas: HTMLCanvasElement;

	onMount(() => {
		// Scene
		const scene = new THREE.Scene();

		// Camera
		const camera = new THREE.PerspectiveCamera(
			75,
			window.innerWidth / window.innerHeight,
			0.1,
			1000
		);
		camera.position.set(0, 0, 5);

		// Renderer
		const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
		renderer.setSize(window.innerWidth, window.innerHeight);

		const environment = new RoomEnvironment();
		const pmremGenerator = new THREE.PMREMGenerator(renderer);

		scene.backgroundBlurriness = 0.0;

		// Environment
		const env = pmremGenerator.fromScene(environment).texture;
		scene.background = env;
		scene.environment = env;
		environment.dispose();

		//
		const geometry = new THREE.BoxGeometry(2, 2, 2);
		let material = new THREE.MeshPhysicalMaterial({
			transmission: 1.0,
			thickness: 0.5,
			roughness: 0.15,
			clearcoat: 1.0,
			ior: 1.5,
			dispersion: 18.0,
			envMapIntensity: 1.0,
			side: THREE.DoubleSide
		});
		const cube = new THREE.Mesh(geometry, material);

		scene.add(cube);

		// Resize handling
		const handleResize = () => {
			const width = window.innerWidth;
			const height = window.innerHeight;
			renderer.setSize(width, height);
			camera.aspect = width / height;
			camera.updateProjectionMatrix();
		};

		window.addEventListener('resize', handleResize);

		const clock = new THREE.Clock();

		function render() {
			const delta = clock.getDelta();

			cube.rotation.y += delta * 0.25;
			cube.rotation.x += delta * 0.25;

			renderer.render(scene, camera);
			renderer.setRenderTarget(null);
			requestAnimationFrame(render);
		}
		render();
	});
</script>

<canvas bind:this={canvas} class=""></canvas>
