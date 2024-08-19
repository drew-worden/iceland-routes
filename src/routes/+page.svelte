<script lang="ts">
	// Imports
	import { onMount } from 'svelte';
	import { PUBLIC_MAPBOX_TOKEN } from '$env/static/public';
	import mapboxgl from 'mapbox-gl';
	import landmannalaugarGeoJSON from '$lib/assets/landmannalaugar.json';

	mapboxgl.accessToken = PUBLIC_MAPBOX_TOKEN;

	let map;

	// Lifecycle
	onMount(() => {
		mapboxgl.accessToken = PUBLIC_MAPBOX_TOKEN;
		map = new mapboxgl.Map({
			container: 'map', // Container ID
			style: 'mapbox://styles/mapbox/satellite-streets-v12', // Style URL
			center: [-19.060623, 63.990666], // Initial center based on the first point
			zoom: 1, // Initial zoom
			pitch: 60, // Tilt angle for 3D terrain
			bearing: -17.6,
			antialias: true, // Improve the rendering quality
			interactive: false,
			hash: false
		});

		map.on('load', () => {
			map.addSource('mapbox-dem', {
				type: 'raster-dem',
				url: 'mapbox://mapbox.terrain-rgb',
				tileSize: 512,
				maxzoom: 14
			});

			map.setTerrain({ source: 'mapbox-dem', exaggeration: 1.5 });

			map.addLayer({
				id: 'sky',
				type: 'sky',
				paint: {
					'sky-type': 'atmosphere',
					'sky-atmosphere-sun': [0.0, 0.0],
					'sky-atmosphere-sun-intensity': 15
				}
			});

			// Start the animation
			drawRoute();
		});
	});

	function drawRoute() {
		// Initialize the route source with an empty LineString
		map.addSource('route', {
			type: 'geojson',
			data: {
				type: 'Feature',
				geometry: {
					type: 'LineString',
					coordinates: []
				}
			}
		});

		map.addLayer({
			id: 'route',
			type: 'line',
			source: 'route',
			layout: {
				'line-join': 'round',
				'line-cap': 'round'
			},
			paint: {
				'line-color': '#FF0000',
				'line-width': 8
			}
		});

		animateRoute(landmannalaugarGeoJSON.features.map((feature) => feature.geometry.coordinates));
	}

	function animateRoute(coordinates) {
		let i = 0;

		function animate() {
			if (i < coordinates.length) {
				// Update the route with the next coordinate
				const currentRoute = map.getSource('route')._data;
				currentRoute.geometry.coordinates.push(coordinates[i]);
				map.getSource('route').setData(currentRoute);

				// Fly to the next point
				map.flyTo({
					center: coordinates[i],
					zoom: 15,
					speed: 2,
					curve: 1,
					easing: (t) => t,
					essential: true
				});

				i++;
				setTimeout(animate, 1); // Adjust the speed of animation
			}
		}

		animate();
	}
</script>

<div class="map" id="map"></div>

<style>
	.map {
		position: absolute;
		top: 0;
		bottom: 0;
		width: 100%;
	}
</style>
