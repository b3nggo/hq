<script lang="ts">
  import { onMount } from 'svelte'
  import { browser } from '$app/environment'
  import Chart from 'chart.js/auto'

  // const urlTemplate = 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
  const urlTemplate = 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png'

  const data = [
    { year: 2010, revenue: 1000000 },
    { year: 2011, revenue: 2000000 },
    { year: 2012, revenue: 1500000 },
    { year: 2013, revenue: 2500000 },
    { year: 2014, revenue: 2200000 },
    { year: 2015, revenue: 3000000 },
    { year: 2016, revenue: 2800000 }
  ]

  const forReview = [
    {
      id: 0,
      imageUrl: 'https://img.daisyui.com/images/profile/demo/2@94.webp',
      name: 'Hart Hagerty',
      submissionDate: '1-03-2025',
      verified: false
    },
    {
      id: 1,
      imageUrl: 'https://img.daisyui.com/images/profile/demo/3@94.webp',
      name: 'Brice Swyre',
      submissionDate: '28-02-2025',
      verified: false
    },
    {
      id: 2,
      imageUrl: 'https://img.daisyui.com/images/profile/demo/4@94.webp',
      name: 'Marjy Ferencz',
      submissionDate: '27-02-2025',
      verified: false
    },
    {
      id: 3,
      imageUrl: 'https://img.daisyui.com/images/profile/demo/5@94.webp',
      name: 'Yancy Tear',
      submissionDate: '26-02-2025',
      verified: false
    }
  ]

  const fraudAlert = [
    {
      id: 0,
      rideId: 0,
      status: 'ongoing',
      location: 'Jl. Cihampelas No. 123, Bandung',
      date: '1-03-2025',
      driver: 'Hart Hagerty',
      imageUrl: 'https://img.daisyui.com/images/profile/demo/2@94.webp',
      passenger: 'Brice Swyre'
    }
  ]

  let revenueEl: HTMLCanvasElement,
    menuView: HTMLDivElement,
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

  const onMenu = async (name = 'stats') => {
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

    await new Promise(() => {
      setTimeout(() => {
        onMenuView()
      }, 210)
    })
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

  const onRevenueElement = () => {
    new Chart(revenueEl, {
      type: 'line',
      data: {
        labels: data.map((d) => d.year),
        datasets: [
          {
            label: 'Revenue',
            data: data.map((d) => d.revenue),
            fill: false,
            borderColor: 'rgb(65, 42, 213)',
            tension: 0.1
          }
        ]
      },
      options: {
        scales: {
          y: {
            beginAtZero: true
          }
        }
      }
    })
  }

  onMount(async () => {
    if (browser && mapElement) map = await createMap(mapElement)
    if (menuView) onMenuView()
    if (revenueEl) onRevenueElement()
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
      <div class={`sized-box mb-4`}>
        <p>8<br /><small>Active drivers</small></p>
      </div>
      <div class={`sized-box mb-4`}>
        <p>21<br /><small>Ongoing rides</small></p>
      </div>
      <div class={`sized-box mb-4`}>
        <p>198<br /><small>Completed rides</small></p>
      </div>
      <div class={`sized-box mb-4`}>
        <p>11<br /><small>Cancellations</small></p>
      </div>
    </div>
  {/if}
  <div class={`menu-view`} bind:this={menuView}>
    <div class={`menu-pages`} style={`min-height: ${menuViewHeight}px; min-width: ${menuViewWidth * 3}px; left: -${menuViewWidth * menuIndex}px;`}>
      <section class={`menu-page`} class:hide={!expanded} style={`min-width: ${menuViewWidth}px; min-height: ${menuViewHeight}px;`}>
        <header class={`menu-header sticky top-0`}>
          <h1 class={`font-bold`}>Stats</h1>
        </header>
        <canvas bind:this={revenueEl}></canvas>
      </section>
      <section class={`menu-page`} class:hide={!expanded} style={`min-width: ${menuViewWidth}px; min-height: ${menuViewHeight}px;`}>
        <header class={`menu-header sticky top-0`}>
          <h1 class={`font-bold`}>Approval</h1>
        </header>
        <div class="overflow-x-auto">
          <table class="table">
            <thead>
              <tr>
                <th>
                  <label>
                    <input type="checkbox" class="checkbox" />
                  </label>
                </th>
                <th>Name</th>
                <th>Submission date</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              {#each forReview as r}
                <tr>
                  <th>
                    <label>
                      <input type="checkbox" class="checkbox" />
                    </label>
                  </th>
                  <td>
                    <div class="flex items-center gap-3">
                      <div class="avatar">
                        <div class="mask mask-squircle h-12 w-12">
                          <img src={r.imageUrl} alt={r.name} />
                        </div>
                      </div>
                      <div>
                        <div class="text-sm opacity-50">{r.id}</div>
                        <div class="font-bold">{r.name}</div>
                      </div>
                    </div>
                  </td>
                  <td>{r.submissionDate}</td>
                  <th>
                    <button class="btn btn-ghost btn-xs">Review</button>
                  </th>
                </tr>
              {/each}
            </tbody>
            <tfoot>
              <tr>
                <th></th>
                <th>Name</th>
                <th>Submission date</th>
                <th></th>
              </tr>
            </tfoot>
          </table>
        </div>
      </section>
      <section class={`menu-page`} class:hide={!expanded} style={`min-width: ${menuViewWidth}px; min-height: ${menuViewHeight}px;`}>
        <header class={`menu-header sticky top-0`}>
          <h1 class={`font-bold`}>Fraud alert</h1>
        </header>
        <div class="overflow-x-auto">
          <table class="table">
            <thead>
              <tr>
                <th>
                  <label>
                    <input type="checkbox" class="checkbox" />
                  </label>
                </th>
                <th>Name</th>
                <th>Location</th>
                <th>Date</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              {#each fraudAlert as f}
                <tr>
                  <th>
                    <label>
                      <input type="checkbox" class="checkbox" />
                    </label>
                  </th>
                  <td>
                    <div class="flex items-center gap-3">
                      <div class="avatar">
                        <div class="mask mask-squircle h-12 w-12">
                          <img src={f.imageUrl} alt={f.driver} />
                        </div>
                      </div>
                      <div>
                        <div class="text-sm opacity-50">{f.id}</div>
                        <div class="font-bold">{f.driver}</div>
                      </div>
                    </div>
                  </td>
                  <td>{f.location}</td>
                  <td>{f.date}</td>
                  <th>
                    <button class="btn btn-ghost btn-xs">Whitelist</button>
                  </th>
                </tr>
              {/each}
            </tbody>
            <tfoot>
              <tr>
                <th></th>
                <th>Name</th>
                <th>Location</th>
                <th>Date</th>
                <th></th>
              </tr>
            </tfoot>
          </table>
        </div>
      </section>
    </div>
  </div>
</div>

<main>
  <div class={`leaflet-container`} bind:this={mapElement}></div>
</main>
