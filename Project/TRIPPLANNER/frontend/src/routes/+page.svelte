<script lang="ts">
  import Map from '$lib/components/Map.svelte';

  // --- Configuration ---
  const MULESOFT_ENDPOINT = 'http://localhost:8086/api/trips'; 

  // --- State ---
  let startCoords = { lat: '', lon: '', label: '' };
  let endCoords = { lat: '', lon: '', label: '' };
  
  let transport = 'driving-car';
  let points = 5;
  
  let startQuery = '';
  let endQuery = '';
  let startResults: any[] = [];
  let endResults: any[] = [];
  
  let loading = false;
  let muleResponse: any = null;
  let errorMsg = '';

  // --- Geocoding Helpers ---
  async function searchPlace(query: string) {
    if (query.length < 3) return [];
    try {
      const res = await fetch(
        `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query)}`
      );
      return await res.json();
    } catch (e) {
      return [];
    }
  }

  async function handleStartInput() {
    startResults = await searchPlace(startQuery);
  }

  async function handleEndInput() {
    endResults = await searchPlace(endQuery);
  }

  function selectStart(place: any) {
    startCoords = { lat: place.lat, lon: place.lon, label: place.display_name };
    startQuery = place.display_name; 
    startResults = []; 
  }

  function selectEnd(place: any) {
    endCoords = { lat: place.lat, lon: place.lon, label: place.display_name };
    endQuery = place.display_name;
    endResults = [];
  }

  // --- Submission Logic ---
  async function planJourney() {
    if (!startCoords.lat || !endCoords.lat) {
      errorMsg = "Please select valid start and end points from the dropdowns.";
      return;
    }

    loading = true;
    errorMsg = '';
    muleResponse = null;

    // Derive "Text" context (e.g., "Bhopal, India")
    const areaText = startCoords.label.split(',').slice(0, 2).join(',');

    const payload = {
      text: areaText,
      start: `${startCoords.lon}, ${startCoords.lat}`, 
      end: `${endCoords.lon}, ${endCoords.lat}`,
      transport: transport,
      points: Number(points)
    };

    try {
      const res = await fetch(MULESOFT_ENDPOINT, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!res.ok) throw new Error(`API Error: ${res.statusText}`);
      
      muleResponse = await res.json();
    } catch (e: any) {
      console.error(e);
      errorMsg = "Failed to plan journey. Check console for details.";
    } finally {
      loading = false;
    }
  }
</script>

<div class="layout">
  <div class="sidebar">
    <h1>Travel Planner</h1>
    
    <div class="form-group">
      <label for="start">Start Location</label>
      <input 
        id="start"
        type="text" 
        bind:value={startQuery} 
        on:input={handleStartInput} 
        placeholder="Search starting point..."
        autocomplete="off"
      />
      {#if startResults.length > 0}
        <ul class="dropdown">
          {#each startResults as place}
            <li>
              <button on:click={() => selectStart(place)}>
                {place.display_name}
              </button>
            </li>
          {/each}
        </ul>
      {/if}
    </div>

    <div class="form-group">
      <label for="end">Destination</label>
      <input 
        id="end"
        type="text" 
        bind:value={endQuery} 
        on:input={handleEndInput} 
        placeholder="Search destination..."
        autocomplete="off"
      />
      {#if endResults.length > 0}
        <ul class="dropdown">
          {#each endResults as place}
            <li>
              <button on:click={() => selectEnd(place)}>
                {place.display_name}
              </button>
            </li>
          {/each}
        </ul>
      {/if}
    </div>

    <div class="row">
      <div class="form-group">
        <label for="transport">Mode</label>
        <select id="transport" bind:value={transport}>
          <option value="driving-car">Car</option>
          <option value="cycling-regular">Bike</option>
          <option value="foot-walking">Walk</option>
        </select>
      </div>

      <div class="form-group">
        <label for="points">Stops</label>
        <input id="points" type="number" bind:value={points} min="1" max="10" />
      </div>
    </div>

    {#if errorMsg}
      <p class="error">{errorMsg}</p>
    {/if}

    <button class="cta-button" on:click={planJourney} disabled={loading}>
      {loading ? 'Planning...' : 'Plan Journey'}
    </button>
  </div>

  <div class="map-area">
    {#if muleResponse}
      <Map mapData={muleResponse} />
    {:else}
      <div class="placeholder">
        <p>Enter your journey details to view the route.</p>
      </div>
    {/if}
  </div>
</div>

<style>
  :global(body) { margin: 0; font-family: sans-serif; }

  .layout {
    display: grid;
    grid-template-columns: 350px 1fr;
    height: 100vh;
  }

  .sidebar {
    padding: 2rem;
    background: #f9fafb;
    border-right: 1px solid #e5e7eb;
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    overflow-y: auto;
  }

  .map-area {
    position: relative;
    background: #e0e0e0;
  }

  h1 { margin: 0 0 1rem 0; font-size: 1.5rem; color: #1f2937; }

  .form-group { position: relative; display: flex; flex-direction: column; gap: 0.5rem; }
  .row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }

  label { font-size: 0.875rem; font-weight: 600; color: #374151; }
  
  input, select {
    padding: 0.75rem;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    font-size: 1rem;
  }

  .dropdown {
    position: absolute;
    top: 100%; left: 0; right: 0;
    background: white;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    list-style: none;
    padding: 0; margin: 0.25rem 0 0 0;
    max-height: 200px;
    overflow-y: auto;
    z-index: 50;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }

  .dropdown button {
    width: 100%;
    text-align: left;
    padding: 0.75rem;
    background: none;
    border: none;
    border-bottom: 1px solid #f3f4f6;
    cursor: pointer;
  }
  .dropdown button:hover { background: #f3f4f6; }

  .cta-button {
    background: #2563eb;
    color: white;
    padding: 1rem;
    border: none;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    margin-top: auto;
  }
  .cta-button:disabled { background: #93c5fd; cursor: not-allowed; }

  .error { color: #dc2626; font-size: 0.875rem; }

  .placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    color: #6b7280;
  }
</style>
