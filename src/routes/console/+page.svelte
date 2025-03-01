<script lang="ts">
  import { onMount } from 'svelte'
  import { browser } from '$app/environment'

  // const urlTemplate = 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
  const urlTemplate = 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png'

  let menuView: HTMLDivElement,
    mapElement: HTMLDivElement | null = null,
    map: any = null,
    view: [number, number] = [-6.940583170466513, 107.68126364180127]

  let activeMenu = $state('stats'),
    expanded = $state(false),
    menuViewHeight = $state(0),
    menuViewWidth = $state(0),
    menuIndex = $state(0)

  const createMap = async (container: HTMLDivElement) => {
    const L = (await import('leaflet')).default

    let m = L.map(container, { preferCanvas: true, zoomControl: false }).setView(view, 20)

    L.tileLayer(urlTemplate, {
      attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OpenStreetMap</a>,
              &copy;<a href="https://carto.com/attributions" target="_blank">CARTO</a>`
    }).addTo(m)

    L.control.zoom({ position: 'bottomleft' }).addTo(m)

    return m
  }

  const onMenu = (name = 'stats') => {
    activeMenu = name
    switch (name) {
      case 'approval':
        menuIndex = 1
        break
      case 'alert':
        menuIndex = 2
        break
      default:
        menuIndex = 0
        break
    }
    expanded = true
  }

  const onResize = () => {
    if (map) map.invalidateSize()
    if (menuView) onMenuView()
  }

  const onMenuView = () => {
    const rect = menuView.getBoundingClientRect()
    menuViewHeight = rect.height
    menuViewWidth = rect.width
  }

  onMount(async () => {
    if (browser && mapElement) map = await createMap(mapElement)
    if (menuView) onMenuView()
  })
</script>

<svelte:window on:resize={onResize} />

<div class={`dynamic-island rounded-2xl`} class:expanded>
  <div class={`dynamic-island-control`}>
    <div class={`join join-horizontal p-4`}>
      <div class="tooltip tooltip-bottom" data-tip="Stats">
        <button class={`btn join-item rounded-l-full`} class:btn-neutral={activeMenu === 'stats'} onclick={() => onMenu('stats')}>
          <span class={`material-icons`}>show_chart</span>
        </button>
      </div>
      <div class="tooltip tooltip-bottom" data-tip="Approval">
        <button class={`btn join-item`} class:btn-neutral={activeMenu === 'approval'} onclick={() => onMenu('approval')}>
          <span class={`material-icons`}>approval</span>
        </button>
      </div>
      <div class="tooltip tooltip-bottom" data-tip="Fraud alert">
        <button class={`btn join-item`} class:btn-neutral={activeMenu === 'alert'} onclick={() => onMenu('alert')}>
          <span class={`material-icons`}>crisis_alert</span>
        </button>
      </div>
      <label class={`btn swap swap-rotate join-item rounded-r-full`}>
        <input type="checkbox" bind:checked={expanded} />
        <span class={`swap-off material-icons`}>menu</span>
        <span class={`swap-on material-icons`}>close</span>
      </label>
    </div>
  </div>
  {#if !expanded}
    <div class={`scroll-view`}>
      {#each { length: 10 } as _, i}
        <div class={`sized-box mb-4`}>
          <span>Quick look {i}</span>
        </div>
      {/each}
    </div>
  {/if}
  <div class={`menu-view`} bind:this={menuView}>
    <div class={`menu-pages`} style={`min-height: ${menuViewHeight}px; min-width: ${menuViewWidth * 3}px; left: -${menuViewWidth * menuIndex}px;`}>
      <section class={`menu-page`} style={`min-width: ${menuViewWidth}px; min-height: ${menuViewHeight}px;`}>
        <header class={`menu-header sticky top-0`}>
          <h1 class={`font-bold`}>Stats</h1>
        </header>
        {#each { length: 10 } as _, i}
          <p class={`p-4`}>{i} Lorem ipsum dolor sit amet consectetur adipisicing elit. Ipsam officia magnam quia et eius, laudantium maxime alias eveniet facilis. Totam cupiditate iste quo itaque ea nulla vero repellat aspernatur dignissimos?</p>
        {/each}
      </section>
      <section class={`menu-page`} style={`min-width: ${menuViewWidth}px; min-height: ${menuViewHeight}px;`}>
        <header class={`menu-header sticky top-0`}>
          <h1 class={`font-bold`}>Approval</h1>
        </header>
        {#each { length: 10 } as _, i}
          <p class={`p-4`}>{i} Lorem ipsum dolor sit amet consectetur adipisicing elit. Ipsam officia magnam quia et eius, laudantium maxime alias eveniet facilis. Totam cupiditate iste quo itaque ea nulla vero repellat aspernatur dignissimos?</p>
        {/each}
      </section>
      <section class={`menu-page`} style={`min-width: ${menuViewWidth}px; min-height: ${menuViewHeight}px;`}>
        <header class={`menu-header sticky top-0`}>
          <h1 class={`font-bold`}>Fraud alert</h1>
        </header>
        {#each { length: 10 } as _, i}
          <p class={`p-4`}>{i} Lorem ipsum dolor sit amet consectetur adipisicing elit. Ipsam officia magnam quia et eius, laudantium maxime alias eveniet facilis. Totam cupiditate iste quo itaque ea nulla vero repellat aspernatur dignissimos?</p>
        {/each}
      </section>
    </div>
  </div>
</div>

<main>
  <div class={`leaflet-container`} bind:this={mapElement}></div>
</main>
