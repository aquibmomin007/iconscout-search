<template>
  <div>
    <!-- Top Navbar -->
    <nav class="navbar navbar-expand-lg navbar-light bg-white border-bottom fixed-top">
      <div>
        <a class="navbar-brand fw-bold" href="#">
          <img
            src="/iconscout-logo.svg"
            class="card-img-top"
            width="192"
          />
        </a>
      </div>
    </nav>

    <!-- Main Container -->
    <section class="main-layout">
      <div class="px-4 pb-4">
        <h1 class="mb-3">{{ totalResults }} Search {{assetTypeLabel}}</h1>
        <p class="text-muted">
          {{ totalResults }} {{assetTypeLabel}} exclusively selected by our designer community.
        </p>
      </div>

      <div class="row px-4">
        <!-- Sidebar Filters -->
        <div class="col-12 col-md-3 mb-4 sidebar ">
          <h4>Filters</h4>
          <div class="mb-3">
            <p class="mb-2 fw-bold">Price</p>
            <div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="free"
                  id="free"
                  v-model="filters.priceCat"
                />
                <label class="form-check-label" for="free">
                  Free
                </label>
              </div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="premium"
                  id="premium"
                  v-model="filters.priceCat"
                />
                <label class="form-check-label" for="premium">
                  Premium
                </label>
              </div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="all"
                  id="all"
                  v-model="filters.priceCat"
                />
                <label class="form-check-label mb-2" for="all">
                  All
                </label>
              </div>
            </div>
          </div>

          <!-- Sort By -->
          <div class="mb-3">
            <p class="mb-2 fw-bold">Sort By</p>
            <select class="form-select" v-model="filters.sortBy">
              <option value="popular">Popular</option>
              <option value="latest">Latest</option>
              <option value="relevant">Relevant</option>
            </select>
          </div>

          <!-- Asset Types -->
          <div>
            <p class="mb-2 fw-bold">Asset Type</p>
            <div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="lottie"
                  id="lottie"
                  v-model="filters.asset"
                />
                <label class="form-check-label" for="lottie">
                  Lottie Animations
                </label>
              </div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="illustration"
                  id="illustration"
                  v-model="filters.asset"
                />
                <label class="form-check-label" for="illustration">
                  Illustrations
                </label>
              </div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="3d"
                  id="3d"
                  v-model="filters.asset"
                />
                <label class="form-check-label" for="3d">
                  3D Illustrations
                </label>
              </div>
              <div class="form-check mb-2">
                <input
                  class="form-check-input"
                  type="radio"
                  value="all"
                  id="asset-all"
                  v-model="filters.asset"
                />
                <label class="form-check-label mb-2" for="asset-all">
                  All
                </label>
              </div>
            </div>
          </div>
        </div>

        <!-- Card Container -->
        <div class="col-12 px-4 scrollable-content">
          <div class="row g-4">
            <!-- Card item -->
            <div
              class="col-6 col-sm-4 col-md-4 col-lg-3"
              v-for="(item, index) in items"
              :key="index"
            >
              <div class="card illustration-card">
                <template v-if="item.asset === 'lottie'">
                  <video autoplay>
                    <source :src="item.image" type="video/mp4">
                    Your browser does not support the video tag.
                  </video>
                </template>
                <template v-else>
                  <img
                    :src="item.image"
                    class="card-img-top"
                    :alt="item.name"
                  />
                </template>
              </div>
            </div>
          </div>

          <!-- Infinite scroll -->
          <div ref="bottomObserver" v-if="hasMore" class="mt-3 text-muted">
            <p v-if="isLoading">Loading more items...</p>
            <p v-else>Scroll to load more</p>
          </div>
        </div>

      </div>
    </section>
  </div>
</template>


<script setup lang="ts">

import axios from 'axios';
import { ref, watch, onMounted, computed } from 'vue';

interface Item {
  name: string
  image: string
  asset: string
}

const filters = ref({
  priceCat: 'all',
  sortBy: 'relevant',
  asset: 'all'
})

const assetTypes = ref({
  '3d': '3D Illustrations',
  'illustration': 'Illustrations',
  'lottie': 'Lottie Animations',
  'all': 'All Assets'
})

const items = ref<Item[]>([])
const totalResults = ref(0);
const assetTypeLabel = ref("");

const page = ref(1)
const hasMore = ref(true)
const isLoading = ref(false)

const bottomObserver = ref<HTMLElement | null>(null)

async function fetchItems(reset = false) {
  if (reset) {
    page.value = 1
    items.value = []
    hasMore.value = true
  }

  if (!hasMore.value || isLoading.value) return
  isLoading.value = true

  try {
    const response = await axios.get('https://api.iconscout.com/v3/search', {
      params: {
        query: 'avatar',
        product_type: 'item',
        asset: filters.value.asset === 'all' ? '' : filters.value.asset,
        per_page: 12,
        page: page.value,
        sort: filters.value.sortBy,
        price: filters.value.priceCat === 'all' ? '' : filters.value.priceCat
      },
      headers: {
        accept: 'application/json',
        'Client-ID': '239474317514292'
      }
    })

    const fetchedData = response.data?.response?.items?.data || []
    const mappedItems: Item[] = fetchedData.map((apiItem: any) => {
      return {
        name: apiItem.name,
        subtitle: apiItem.slug ?? '',
        image: apiItem.urls?.thumb ?? apiItem.urls?.png_256 ?? apiItem.urls?.lottie ?? '',
        asset: apiItem.asset,
        uuid: apiItem.uuid
      }
    })
    
    items.value.push(...mappedItems)

    const total = response.data?.response?.items?.total || 0
    totalResults.value = total

    assetTypeLabel.value = assetTypes.value[filters.value.asset]

    const nextPageUrl = response.data?.response?.items?.next_page_url
    if (!nextPageUrl) {
      hasMore.value = false
    } else {
      page.value += 1
    }
  } catch (error) {
    console.error('Failed to fetch items:', error)
    hasMore.value = false
  } finally {
    isLoading.value = false
  }
}

watch(filters, () => {
  fetchItems(true)
}, { deep: true })

onMounted(() => {
  fetchItems(true)

  const observer = new IntersectionObserver(
    (entries) => {
      const [entry] = entries
      if (entry.isIntersecting && !isLoading.value && hasMore.value) {
        fetchItems(false)
      }
    },
    {
      rootMargin: '0px',
      threshold: 1.0
    }
  )

  if (bottomObserver.value) {
    observer.observe(bottomObserver.value)
  }
})
</script>


<style scoped>
  .navbar{
    padding: 12px 24px;
  }

  .sidebar{
    max-width: 257px;
    padding: 0 24px 0 12px;
  }

  .illustration-card {
    border: 1px solid #ddd;
    transition: box-shadow 0.2s ease;
    cursor: pointer;
    padding: 8px 12px;
    max-height: 210px;
    height: 100%;
  }
  .illustration-card:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    background: #F5F6FA;
  }
  .illustration-card img {
    object-fit: cover;
    margin: auto;
    display: block;
    width: auto;
    max-height: 190px;
  }

  .illustration-card video {
    object-fit: cover;
    margin: auto;
    display: block;
    width: auto;
    max-height: 190px;
  }

  .scrollable-content {
    overflow-y: auto;
    max-height: calc(100vh - 16rem);
    max-width:calc(100% - 257px);
  }

  .main-layout {
    padding-top: 6rem;
    height: 100vh;
    overflow: hidden;
  }
</style>
