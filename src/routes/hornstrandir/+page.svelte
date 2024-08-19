<script lang="ts">
	// Imports
	import { onMount } from 'svelte';
	import { PUBLIC_MAPBOX_TOKEN } from '$env/static/public';
	import mapboxgl from 'mapbox-gl';
	import hornstrandirGeoJSON from '$lib/assets/hornstrandir.json';

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
			pitch: 70, // Tilt angle for 3D terrain
			bearing: 0,
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
			zoomToStart();
		});
	});

	function zoomToStart() {
		const startCoords = hornstrandirGeoJSON.features[0].geometry.coordinates;

		map.flyTo({
			center: startCoords,
			zoom: 15,
			speed: 0.8,
			curve: 1,
			easing: (t) => t,
			essential: true
		});

		// When the map finishes moving, start drawing the route
		map.once('moveend', () => {
			drawRoute();
		});
	}

	function drawRoute() {
		const coordinates = hornstrandirGeoJSON.features.map((feature) => feature.geometry.coordinates);

		// Add the route line
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

		// Add start and end markers
		addStartEndMarkers(coordinates);

		// Start animating the route
		animateRoute(coordinates);
	}

	function addStartEndMarkers(coordinates) {
		// Add the start marker
		new mapboxgl.Marker({ color: 'green' }).setLngLat(coordinates[0]).addTo(map);

		// Add the end marker
		new mapboxgl.Marker({ color: 'red' }).setLngLat(coordinates[coordinates.length - 1]).addTo(map);
	}

	function animateRoute(coordinates) {
		let i = 0;

		function animate() {
			if (i < coordinates.length - 1) {
				// Update the route with the next coordinate
				const currentRoute = map.getSource('route')._data;
				currentRoute.geometry.coordinates.push(coordinates[i]);
				map.getSource('route').setData(currentRoute);

				// Calculate the smoothed bearing over the next few points
				const bearing = calculateSmoothedBearing(coordinates, i, 7); // Average over 5 points

				// Incrementally adjust the camera to smoothly follow the route and turn to face the direction
				const [lng, lat] = coordinates[i];
				const offsetCenter = [lng, lat + 0.005]; // Adjust this value to keep the line closer to the center

				map.easeTo({
					center: offsetCenter,
					bearing: bearing, // Rotate the camera to face the smoothed direction of travel
					zoom: 15,
					speed: 0.5, // Slower speed for smoother transitions
					curve: 1.42, // Smoother curve
					easing: (t) => t,
					essential: true
				});

				i++;
				requestAnimationFrame(animate); // Use requestAnimationFrame for smoother animation
			}
		}

		animate();
	}

	function calculateSmoothedBearing(coordinates, index, numPoints) {
		let bearings = [];
		for (let j = index; j < index + numPoints - 1 && j < coordinates.length - 1; j++) {
			const bearing = calculateBearing(coordinates[j], coordinates[j + 1]);
			bearings.push(bearing);
		}
		// Average the bearings
		const smoothedBearing = bearings.reduce((a, b) => a + b, 0) / bearings.length;
		return smoothedBearing;
	}

	function calculateBearing(start, end) {
		const [startLng, startLat] = start;
		const [endLng, endLat] = end;
		const deltaLng = endLng - startLng;

		const y = Math.sin(deltaLng) * Math.cos(endLat);
		const x =
			Math.cos(startLat) * Math.sin(endLat) -
			Math.sin(startLat) * Math.cos(endLat) * Math.cos(deltaLng);
		const bearing = (Math.atan2(y, x) * 180) / Math.PI;

		return bearing;
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
