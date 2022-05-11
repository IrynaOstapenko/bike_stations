<template>
	<div v-if="!error" id="map" class="map"></div>
	<div v-if="error">{{ error }}</div>
</template>

<script>
	//Creating map with use of mapbox
	import mapboxgl from 'mapbox-gl';

	export default {
		data() {
			return {
				mapbox_id: import.meta.env.VITE_MAPBOX_ID,
				bikeCoordinates: [],
				error: ''
			}					
		},

		created() {
			this.fetchData();
		},

		methods: {
			async fetchData() {
				//Fetching API data with coordinates of bike rent places in Oslo
            	const url = `https://api.allorigins.win/get?url=${encodeURIComponent('https://data-legacy.urbansharing.com/legacy-api/stations.json')}`;
           	 	const response = await fetch(url);
				try {
					await this.handleResponse(response);
				} catch (error) {
					console.log(error);
					this.error = error.message;
				}            
			},

			async handleResponse(response) {
				if (response.status >= 200 && response.status < 300) {
					const results = await response.json();
					const contents = results.contents;
					const parsedResults = JSON.parse(contents);
					const stations = parsedResults.stations;
					const coordinatesArray = stations.map(element => element.center);
					this.bikeCoordinates = coordinatesArray;

					//Creating the map 
					mapboxgl.accessToken = this.mapbox_id;

					const map = new mapboxgl.Map({
					container: 'map', 
					style: 'mapbox://styles/mapbox/streets-v11', 
					center: [10.76, 59.91], 
					zoom: 9 
					});

					//Using the coordinates received in API for creating markers on the map
					this.bikeCoordinates.map(bikeCoordinate => {
					const el = document.createElement('div');
					el.className = 'marker';
					el.style.backgroundImage = 'url(/images/bicycle1.png)';
					el.style.width = '30px';
					el.style.height = '30px';
					new mapboxgl.Marker(el)
					.setLngLat([bikeCoordinate.longitude, bikeCoordinate.latitude])
					.addTo(map);
				})
			 	} else {
				  	if (response.status === 404) {
                        throw new Error('Url not found');
                    }
                    if (response.status === 401) {
                        throw new Error('Unauthorized');
                    }
                    if (response.status > 500) {
                        throw new Error('Server error');
                    }
                        throw new Error('No data');
			 	}
			}
		}
	}
</script>

<style>
 	.map {
		width: 100%;
		height: 600px;
 	}

	 .marker {
		display: block;
		border: none;
		border-radius: 50%;
		cursor: pointer;
		padding: 0;
	}

</style>

