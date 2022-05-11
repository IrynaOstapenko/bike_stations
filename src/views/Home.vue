<template>
	<div id="map" class="map"></div>
</template>

<script>
//Creating map with use of mapbox
import mapboxgl from 'mapbox-gl';

export default {
	data() {
		return {
			mapbox_id: import.meta.env.VITE_MAPBOX_ID,
			bikeCoordinates: [],
		}
					
	},
	created() {
		this.fetchData();
	},

	methods: {
		async fetchData() {
			//Fetching API data with coordinates of bike rent places in Oslo
            const url = `https://api.allorigins.win/get?url=${encodeURIComponent('https://data-legacy.urbansharing.com/legacy-api/stations.json')}`;
            const res = await fetch(url);
            const results = await res.json();
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

