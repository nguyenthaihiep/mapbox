<template>
	<div id="map"></div>
	<div class="box-search">
		<input type="text" class="search" v-model="search"/>
		<div class="search-result" v-if="search.length > 1">
			<ul v-if="resultSearch.length > 0">
				<li v-for="(data, index) in resultSearch" :key="index" @click="setEndPoint(data.center)">
					<div>
						<div class="address">{{ data.placeName }}</div>
					</div>
				</li>
			</ul>
			<p v-if="resultSearch.length < 1">No result</p>
		</div>
	</div>
</template>

<script>
	import 'mapbox-gl/dist/mapbox-gl.css';
	import mapboxgl from 'mapbox-gl';
	
	export default {
		data() {
			return {
				map: null,
				location: [106.6631168, 10.7347968],
				search: '',
				token: 'pk.eyJ1IjoiaGllcG5ndXllbjk0IiwiYSI6ImNrcHFiajN3cTBibDYyd2xlZnI2aTY4d2QifQ.aW648xC7HA6RDOKrRejAFg',
				resultSearch: []
			}
		},
		created() {
			setTimeout(() => {
				mapboxgl.accessToken = this.token;
				this.map = new mapboxgl.Map({
					container: 'map',
					style: 'mapbox://styles/mapbox/streets-v10',
					center: this.location,
					zoom: 12
				});
				this.getRoute(this.location);

				navigator.geolocation.getCurrentPosition(pos => {
					this.location = [pos.coords.longitude, pos.coords.latitude];
				}, err => {
					this.errorStr = err.message;
				})
			},0);
		},
		methods: {
			getRoute(end) {
				var start = this.location;
				var url = 'https://api.mapbox.com/directions/v5/mapbox/cycling/' + start[0] + ',' + start[1] + ';' + end[0] + ',' + end[1] + '?steps=true&geometries=geojson&access_token=' + this.token;

				var req = new XMLHttpRequest();
				req.open('GET', url, true);
				req.onload = () => {
					var json = JSON.parse(req.response);
					var data = json.routes[0];
					var route = data.geometry.coordinates;
					var geojson = {
						type: 'Feature',
						properties: {},
						geometry: {
							type: 'LineString',
							coordinates: route
						}
					};
					if (this.map.getSource('route')) {
						this.map.getSource('route').setData(geojson);
					} else { 
						this.map.addLayer({
							id: 'route',
							type: 'line',
							source: {
								type: 'geojson',
								data: {
									type: 'Feature',
									properties: {},
									geometry: {
										type: 'LineString',
										coordinates: geojson
									}
								}
							},
							layout: {
								'line-join': 'round',
								'line-cap': 'round'
							},
							paint: {
								'line-color': '#3887be',
								'line-width': 5,
								'line-opacity': 0.75
							}
						});
					}
				};
				req.send();
			},
			setEndPoint(coords) {
				var end = {
					type: 'FeatureCollection',
					features: [{
						type: 'Feature',
						properties: {},
						geometry: {
							type: 'Point',
							coordinates: coords
						}
					}]
				};

				if (this.map.getLayer('end')) {
					this.map.getSource('end').setData(end);
				} else {
					this.map.addLayer({
						id: 'end',
						type: 'circle',
						source: {
							type: 'geojson',
							data: {
								type: 'FeatureCollection',
								features: [{
									type: 'Feature',
									properties: {},
									geometry: {
										type: 'Point',
										coordinates: coords
									}
								}]
							}
						},
						paint: {
							'circle-radius': 10,
							'circle-color': '#f30'
						}
					});
				}
				this.getRoute(coords);
			},
			async fetchDatas() {
				const response = await fetch(
					'https://api.mapbox.com/geocoding/v5/mapbox.places/'+this.search+'.json?access_token='+this.token+'&cachebuster=1623469152097&autocomplete=true'
				);
				const responseData = await response.json();
				if(!response.ok) {
					const error = new Error(responseData.message || 'Fail to fetch datas');
					throw error;
				}

				const datas = [];

				for(const key in responseData.features) {
					const data = {
						center: responseData.features[key].center,
						placeName: responseData.features[key].place_name,
					};
					datas.push(data);
				}
				this.resultSearch = datas;
			}
		},
		computed: {
			newCenter() {
				this.map.on('load', () => {
					this.map.addLayer({
						id: 'point',
						type: 'circle',
						source: {
							type: 'geojson',
							data: {
								type: 'FeatureCollection',
								features: [{
									type: 'Feature',
									properties: {},
									geometry: {
										type: 'Point',
										coordinates: this.location
									}
								}]
							}
						},
						paint: {
							'circle-radius': 10,
							'circle-color': '#3887be'
						}
					});
				});
				this.map.flyTo({
					center: this.location,
					essential: true
				})
				return true;
			}
		},
		watch: {
			location() {
				this.newCenter;
			},
			search(newValue) {
				if(newValue.length > 1) {
					this.fetchDatas();
				} else{
					this.resultSearch = [];
				}
			}
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
	.search {
		width: 100%;
		height: 30px;
		border: none;
	}
	.search:focus {
		outline: none;
	}
	.box-search {
		position: absolute;
		left: 20px;
		top: 20px;
		width: 300px;
	}
	.search-result ul {
		list-style: none;
		padding: 0;
		background-color: #fff;
		border-radius: 2px;
	}
	.search-result p {
		background-color: #fff;
		padding: 10px;
		text-align: center;
	}
	.search-result ul li > div {
		padding: 10px;
		cursor: pointer;
	}
	.search-result ul li.active > div, .search-result ul li:hover > div {
		background-color: #f3f3f3;
	}
	.search-result .title {
		font-weight: bold;
	}
	.search-result .title, .search-result .address {
		/*text-overflow: ellipsis;
		overflow: hidden;
		white-space: nowrap;*/
	}
</style>