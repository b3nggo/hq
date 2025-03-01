<script lang="ts">
  import { onMount } from 'svelte'
  import { browser } from '$app/environment'

  // const urlTemplate = 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
  const urlTemplate = 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png'

  let mapElement: HTMLDivElement | null = null,
    map: any = null,
    view: [number, number] = [-6.940583170466513, 107.68126364180127]

  const createMap = async (container: HTMLDivElement) => {
    const L = (await import('leaflet')).default

    let m = L.map(container, { preferCanvas: true }).setView(view, 20)

    L.tileLayer(urlTemplate, {
      attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OpenStreetMap</a>,
              &copy;<a href="https://carto.com/attributions" target="_blank">CARTO</a>`,
      maxZoom: 14
    }).addTo(m)

    return m
  }

  const onResize = () => {
    if (map) map.invalidateSize()
  }

  onMount(async () => {
    if (browser && mapElement) map = await createMap(mapElement)
  })
</script>

<svelte:window on:resize={onResize} />

<main>
  <div class={`leaflet-container`} bind:this={mapElement}></div>
</main>
