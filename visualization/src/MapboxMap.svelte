<script>
    import { onMount } from 'svelte';
    import mapboxgl from 'mapbox-gl';
    import * as d3 from 'd3';

    mapboxgl.accessToken = 'pk.eyJ1IjoiY2hhcmxpZS0yNzkiLCJhIjoiY2xzbXhmbm5yMHJsNzJrcDU3N2djZXB0YyJ9.NDuLBdFNAlnOX-t9Uvtatg';

    const getColor = (count) => {
        if (count > 100) {
            return 'red';
        } else if (count > 20) {
            return 'orange';
        } else if (count > 10) {
            return 'green';
        } else if (count > 2) {
            return 'blue';
        } else {
            return 'purple';
        }
    };

    onMount(() => {
        const map = new mapboxgl.Map({
            container: 'map',
            style: 'mapbox://styles/mapbox/light-v10',
            center: [0, 0],
            zoom: 1
        });

        d3.csv("data/yc_companies.csv").then(function(data) {
            const locationCounts = {};
            data.forEach(function(company) {
                const location = `${company.Latitude},${company.Longitude}`;
                locationCounts[location] = (locationCounts[location] || 0) + 1;
            });

            Object.keys(locationCounts).forEach(function(location) {
                const [latitude, longitude] = location.split(',').map(parseFloat);
                const count = locationCounts[location];
                const radius = 3 + 0.02 * count;

                if (!isNaN(latitude) && !isNaN(longitude)) {
                    const color = getColor(count);

                    const el = document.createElement('div');
                    el.className = 'circle-marker';
                    el.style.width = el.style.height = `${2 * radius}px`;
                    el.style.backgroundColor = color;
                    el.style.borderRadius = '50%';
                    el.style.opacity = '0.7';

                    // Add circle marker to the map
                    new mapboxgl.Marker(el)
                        .setLngLat([longitude, latitude])
                        .addTo(map);
                }
            });
        }).catch(function(error) {
            console.error("Error loading CSV file:", error);
        });
    });
</script>

<style>
    #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
    }
</style>

<div id="map"></div>
