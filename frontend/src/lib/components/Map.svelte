<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { browser } from '$app/environment';

  export let mapData: any; // The big JSON object from MuleSoft

  let mapElement: HTMLElement;
  let map: any;

  onMount(async () => {
    if (browser) {
      // 1. Dynamic import of Leaflet to avoid SSR errors
      const L = (await import('leaflet')).default;

      // 2. Initialize Map
      // We center it roughly on the start point initially
      const startLat = mapData.Weather[0].Latitude;
      const startLon = mapData.Weather[0].Longitude;
      
      map = L.map(mapElement).setView([startLat, startLon], 9);

      // 3. Add OpenStreetMap Tiles
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
      }).addTo(map);

      // --- VISUALIZATION LOGIC ---

      // A. DRAW THE ROUTE (Blue Line)
      // MuleSoft sends GeoJSON (Lon, Lat), but Leaflet wants (Lat, Lon)
      // We extract coordinates from the Route FeatureCollection
      if (mapData.Route && mapData.Route.features) {
        const routeCoords = mapData.Route.features[0].geometry.coordinates.map(
          (coord: number[]) => [coord[1], coord[0]] // Swap Lon, Lat to Lat, Lon
        );

        const routeLine = L.polyline(routeCoords, {
          color: 'blue',
          weight: 5,
          opacity: 0.7
        }).addTo(map);

        // Fit map view to show the whole route
        map.fitBounds(routeLine.getBounds(), { padding: [50, 50] });
      }

      // B. ADD WEATHER POINTS (Emojis)
      if (mapData.Weather) {
        mapData.Weather.forEach((point: any) => {
          // Determine emoji based on message text (simple logic)
          let emoji = '🌤️'; 
          const msg = point.Message.toLowerCase();
          if (msg.includes('cold')) emoji = '❄️';
          else if (msg.includes('hot')) emoji = '☀️';
          else if (msg.includes('rain')) emoji = '🌧️';

          // Create a custom HTML Icon for the emoji
          const weatherIcon = L.divIcon({
            className: 'weather-icon',
            html: `<div style="font-size: 24px; text-shadow: 0 0 3px white;">${emoji}</div>`,
            iconSize: [30, 30],
            iconAnchor: [15, 15] // Center the emoji
          });

          // Add marker
          const marker = L.marker([point.Latitude, point.Longitude], { icon: weatherIcon })
            .addTo(map);
          
          // Add popup with the message
          marker.bindPopup(`<b>Weather Alert</b><br>${point.Message}`);
        });
      }
    }
  });

  onDestroy(() => {
    if (map) {
      map.remove();
    }
  });
</script>

<div class="map-container" bind:this={mapElement}></div>

<svelte:head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" 
     integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" 
     crossorigin=""/>
</svelte:head>

<style>
  .map-container {
    width: 100%;
    height: 500px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    margin-top: 20px;
    z-index: 1; /* Keep it below dropdowns */
  }
</style>