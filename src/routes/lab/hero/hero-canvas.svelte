<script lang="ts">
	import * as THREE from 'three';
	import { RoomEnvironment } from 'three/examples/jsm/Addons.js';

	let canvasElement: HTMLCanvasElement;

	$effect(() => {
		const canvas: HTMLCanvasElement = canvasElement;

		if (!canvas) return;

		function getWidth() {
			return parseInt(window.getComputedStyle(canvas).width);
		}

		function getHeight() {
			return parseInt(window.getComputedStyle(canvas).height);
		}

		//
		// Scene
		//
		const scene = new THREE.Scene();

		const geometry = new THREE.BoxGeometry(2, 2, 2);
		let material = new THREE.MeshPhysicalMaterial({
			transmission: 1.33,
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

		const initialAspect = getWidth() / getHeight();

		//
		//  Camera
		//
		const camera = new THREE.PerspectiveCamera(75, initialAspect, 0.1, 1000);
		camera.position.set(0, 0, 5);

		//
		// Renderer
		//
		const renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
		renderer.setSize(getWidth(), getHeight(), false); // 'false' prevents setting inline styles, letting CSS rule

		//
		// Environment
		//
		const environment = new RoomEnvironment();
		const pmremGenerator = new THREE.PMREMGenerator(renderer);
		scene.backgroundBlurriness = 0.5;

		const env = pmremGenerator.fromScene(environment).texture;
		scene.background = env;
		scene.environment = env;
		environment.dispose();

		// Animation loop
		let animationId: number;

		const animate = () => {
			animationId = requestAnimationFrame(animate);
			cube.rotation.x += 0.01;
			cube.rotation.y += 0.01;
			renderer.render(scene, camera);
		};

		animate();

		//
		// Resize handling and observation
		//
		const resizeObserver = new ResizeObserver(() => {
			const width = getWidth();
			const height = getHeight();

			// Guard against 0 height (prevents crash when element is hidden)
			if (height === 0) return;

			camera.aspect = width / height;
			camera.updateProjectionMatrix();

			renderer.setSize(width, height, false);
		});

		resizeObserver.observe(canvas);

		// Cleanup
		return () => {
			cancelAnimationFrame(animationId);
			resizeObserver.disconnect();
			renderer.dispose();
			geometry.dispose();
			material.dispose();
		};
	});
</script>

<canvas class="block h-full w-full" bind:this={canvasElement}></canvas>
