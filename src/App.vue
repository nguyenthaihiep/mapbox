<template>
	<div id="map"></div>
</template>

<script>
	import 'mapbox-gl/dist/mapbox-gl.css';
	import '@mapbox/mapbox-gl-directions/dist/mapbox-gl-directions.css'
	import mapboxgl from 'mapbox-gl';
	import MapboxDirections from '@mapbox/mapbox-gl-directions/dist/mapbox-gl-directions';
	
	export default {
		created() {
			setTimeout(function() {
				mapboxgl.accessToken = 'pk.eyJ1IjoiaGllcG5ndXllbjk0IiwiYSI6ImNrcHFiajN3cTBibDYyd2xlZnI2aTY4d2QifQ.aW648xC7HA6RDOKrRejAFg';

				const map = new mapboxgl.Map({
					container: 'map',
					style: 'mapbox://styles/mapbox/streets-v11',
					center: [106.6509628, 10.7323728],
					zoom: 14
				});

				map.addControl(
					new MapboxDirections({
						accessToken: mapboxgl.accessToken
					}),
					'top-left'
				);
			},0);
		}
	}
</script>

<style>
	* {
		box-sizing: border-box;
	}
	body {
		padding: 0;
		margin: 0;
	}
	#map {
		width: 100%;
		height: 100vh;
		display: inline-block;
	}
</style>