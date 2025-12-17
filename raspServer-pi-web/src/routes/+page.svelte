<script>
  import { onMount, tick } from 'svelte';
  import mapa from '$lib/assets/Mapa_IES_vidreres_v2.svg';

  let mapContainer;
  let mapLoaded = false;

  let players = [];
  let playerPositions = [];

  function relToXY(latRel, lonRel) {
    if (!mapContainer || !mapLoaded) return { x: 0, y: 0 };

    const rect = mapContainer.getBoundingClientRect();

    return {
      x: lonRel * rect.width,
      y: latRel * rect.height   
    };
  }

  async function fetchLocations() {
    try {
      const res = await fetch('http://192.168.0.100:3000/mapa');
      const data = await res.json();

      players = data.map((p, i) => ({
        id: p.jugador_id ?? `p_${i}`,
        lat: Number(p.lat),
        lon: Number(p.lon)
      }));

      await tick();
      updatePositions();
    } catch (err) {
      console.error('Error fetching locations', err);
    }
  }

  function updatePositions() {
    if (!mapLoaded) return;

    playerPositions = players.map(p => ({
      ...p,
      pos: relToXY(p.lat, p.lon)
    }));
  }

  function onMapLoaded() {
    mapLoaded = true;
    updatePositions();
  }

  onMount(() => {
    fetchLocations();
    const interval = setInterval(fetchLocations, 1000);
    return () => clearInterval(interval);
  });
</script>

<div bind:this={mapContainer} class="mapa">
  <img
    src={mapa}
    alt="Mapa del campo"
    on:load={onMapLoaded}
  />

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
