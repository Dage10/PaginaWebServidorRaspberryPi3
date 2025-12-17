<script>
  import { onMount } from 'svelte';
  import mapa from '$lib/assets/Mapa_IES_vidreres_v2.svg';

  let mapContainer;
  let mapLoaded = false;

  let players = [];
  let playerPositions = [];

  const MARKER_SIZE = 20; 

  function relToXY(latRel, lonRel) {
    if (!mapContainer || !mapLoaded) return { x: 0, y: 0 };
    const rect = mapContainer.getBoundingClientRect();

    
    let lat = Math.min(Math.max(latRel, 0), 1);
    let lon = Math.min(Math.max(lonRel, 0), 1);

    
    const x = lon * rect.width;
    const y = (1 - lat) * rect.height;

    const halfMarker = MARKER_SIZE / 2;

    return {
      x: Math.min(Math.max(x, halfMarker), rect.width - halfMarker),
      y: Math.min(Math.max(y, halfMarker), rect.height - halfMarker)
    };
  }

  async function fetchLocations() {
    try {
      const res = await fetch('http://192.168.0.100:3000/mapa');
      const data = await res.json();

      if (!Array.isArray(data)) {
        players = [];
        playerPositions = [];
        return;
      }

      players = data.map((p, i) => ({
        id: p.jugador_id ?? `p_${i}`,
        lat: Number(p.lat),
        lon: Number(p.lon)
      }));

      if (mapLoaded) {
        setTimeout(() => updatePositions(), 10);
      }
    } catch (err) {
      console.error('Error fetching locations', err);
    }
  }

  function updatePositions() {
    if (!mapLoaded || !mapContainer) return;

    playerPositions = players.map(p => {
      const pos = relToXY(p.lat, p.lon);
      return { ...p, pos };
    });
  }

  function onMapLoaded() {
    mapLoaded = true;
    setTimeout(() => updatePositions(), 10);
  }

  onMount(() => {
    fetchLocations();
    const interval = setInterval(fetchLocations, 5000);
    return () => clearInterval(interval);
  });
</script>

<div bind:this={mapContainer} class="mapa">
  <img src={mapa} alt="Mapa" on:load={onMapLoaded} />

  {#each playerPositions as player (player.id)}
    <div
      class="marker"
      style="left: {player.pos.x}px; top: {player.pos.y}px;"
      title={player.id}
    ></div>
  {/each}
</div>

<style>
  .mapa {
    position: relative;
    width: 800px;
    height: 600px;
  }

  .mapa img {
    width: 100%;
    height: 100%;
    display: block;
  }

  .marker {
    position: absolute;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: blue;
    transform: translate(-50%, -50%);
    z-index: 10;
  }
</style>
