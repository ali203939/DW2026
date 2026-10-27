<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'

type Character = {
  id: number
  name: string
  status: 'Alive' | 'Dead' | 'unknown'
  image: string
  location: {
    name: string
  }
}

type ApiInfo = {
  count: number
  pages: number
  next: string | null
  prev: string | null
}

const characters = ref<Character[]>([])
const allCharacters = ref<Character[]>([])
const info = ref<ApiInfo>({ count: 0, pages: 0, next: null, prev: null })
const currentPage = ref(1)
const pageInput = ref('1')
const loading = ref(false)
const retryDelayMs = 1500
const maxRetries = 5
const itemsPerPage = 10
const pageWindowSize = 10

const statusIcon = (status: Character['status']) => {
  switch (status) {
    case 'Alive':
      return { label: 'Vivo', icon: '🟢', color: '#22c55e' }
    case 'Dead':
      return { label: 'Morto', icon: '🔴', color: '#ef4444' }
    default:
      return { label: 'Desconhecido', icon: '⚪', color: '#9ca3af' }
  }
}

const totalPages = computed(() => Math.max(1, Math.ceil(allCharacters.value.length / itemsPerPage)))

const paginatedCharacters = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  return allCharacters.value.slice(start, start + itemsPerPage)
})

const pageNumbers = computed(() => {
  const total = totalPages.value

  if (total <= pageWindowSize) {
    return Array.from({ length: total }, (_, index) => index + 1)
  }

  const startPage = Math.floor((currentPage.value - 1) / pageWindowSize) * pageWindowSize + 1
  const endPage = Math.min(startPage + pageWindowSize - 1, total)

  return Array.from({ length: endPage - startPage + 1 }, (_, index) => startPage + index)
})

const fetchCharacters = async (retries = 0) => {
  loading.value = true

  try {
    const firstPageResponse = await fetch('https://rickandmortyapi.com/api/character?page=1')

    if (!firstPageResponse.ok) {
      throw new Error('Resposta inválida da API.')
    }

    const firstPageData = await firstPageResponse.json()
    const totalApiPages = firstPageData.info.pages

    const pageResponses = await Promise.all(
      Array.from({ length: totalApiPages }, (_, index) =>
        fetch(`https://rickandmortyapi.com/api/character?page=${index + 1}`),
      ),
    )

    const pagesData = await Promise.all(
      pageResponses.map(async (response) => {
        if (!response.ok) {
          throw new Error('Resposta inválida da API.')
        }

        return response.json()
      }),
    )

    allCharacters.value = pagesData.flatMap((pageData) => pageData.results)
    characters.value = paginatedCharacters.value
    info.value = firstPageData.info
    currentPage.value = 1
  } catch (err) {
    const shouldRetry = retries < maxRetries

    if (shouldRetry) {
      window.setTimeout(() => {
        fetchCharacters(retries + 1)
      }, retryDelayMs)
      return
    }

    loading.value = false
    return
  }

  characters.value = paginatedCharacters.value
  loading.value = false
}

const changePage = (page: number) => {
  if (page < 1 || page > totalPages.value) return
  currentPage.value = page
  pageInput.value = String(page)
  characters.value = paginatedCharacters.value
}

const goToPage = () => {
  const pageNumber = Number(pageInput.value)

  if (!Number.isInteger(pageNumber)) {
    pageInput.value = String(currentPage.value)
    return
  }

  changePage(pageNumber)
}

watch(currentPage, (value) => {
  pageInput.value = String(value)
})

onMounted(() => {
  fetchCharacters(1)
})
</script>

<template>
  <main class="page">
    <header class="header">
      <div>
        <p class="eyebrow">Rick and Morty</p>
        <h1>Personagens</h1>
      </div>
    </header>

    <section v-if="loading" class="loading" aria-live="polite">
      Carregando personagens...
    </section>

    <section v-else class="content">
      <nav class="pagination" aria-label="Paginação de personagens">
        <div class="pagination-meta">
          <span>Página</span>
          <strong>{{ currentPage }}</strong>
          <span>de</span>
          <strong>{{ totalPages }}</strong>
        </div>

        <div class="pagination-actions">
          <button type="button" :disabled="currentPage === 1" @click="changePage(currentPage - 1)">
            Anterior
          </button>

          <div class="page-numbers" aria-label="Páginas">
            <button
              v-for="page in pageNumbers"
              :key="page"
              type="button"
              :class="{ active: page === currentPage }"
              @click="changePage(page)"
            >
              {{ page }}
            </button>
          </div>

          <button type="button" :disabled="currentPage === totalPages" @click="changePage(currentPage + 1)">
            Próxima
          </button>
        </div>

        <div class="jump-to-page">
          <label for="page-jump">Ir para</label>
          <input
            id="page-jump"
            v-model="pageInput"
            type="number"
            min="1"
            :max="totalPages"
            @keyup.enter="goToPage"
          />
          <button type="button" @click="goToPage">Ir</button>
        </div>
      </nav>

      <div class="cards-grid">
        <article v-for="character in paginatedCharacters" :key="character.id" class="card">
          <img :src="character.image" :alt="character.name" class="character-image" />

          <div class="card-body">
            <div class="card-top">
              <h2>{{ character.name }}</h2>
              <span class="status" :style="{ color: statusIcon(character.status).color }">
                <span class="status-icon">{{ statusIcon(character.status).icon }}</span>
                {{ statusIcon(character.status).label }}
              </span>
            </div>

            <p class="location-label">Local onde vive</p>
            <p class="location-name">{{ character.location.name }}</p>
          </div>
        </article>
      </div>
    </section>
  </main>
</template>

<style scoped>
:global(body) {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(135deg, #0f172a, #111827 35%, #1f2937);
  color: #f3f4f6;
}

* {
  box-sizing: border-box;
}

button {
  font: inherit;
}

.page {
  min-height: 100vh;
  padding: 32px 20px 48px;
}

.header {
  max-width: 1200px;
  margin: 0 auto 24px;
}

.eyebrow {
  margin: 0;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  font-size: 0.7rem;
  color: #a5f3fc;
}

h1 {
  margin: 10px 0 0;
  font-size: clamp(2.2rem, 4vw, 3.5rem);
}

.content,
.alert,
.loading {
  max-width: 1200px;
  margin: 0 auto;
}

.alert,
.loading {
  padding: 18px 20px;
  border-radius: 12px;
  background: rgba(15, 23, 42, 0.9);
  border: 1px solid rgba(148, 163, 184, 0.2);
  margin-bottom: 20px;
}

.pagination {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
  margin-bottom: 28px;
  padding: 14px 18px;
  border: 1px solid rgba(148, 163, 184, 0.2);
  border-radius: 18px;
  background: rgba(15, 23, 42, 0.72);
  box-shadow: 0 18px 30px rgba(15, 23, 42, 0.16);
}

.pagination-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  color: #cbd5e1;
}

.pagination-meta strong {
  color: #f8fafc;
  font-size: 1rem;
}

.pagination-actions {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
  flex: 1;
}

.jump-to-page {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
  border-radius: 12px;
  background: rgba(15, 23, 42, 0.9);
  border: 1px solid rgba(148, 163, 184, 0.18);
}

.jump-to-page label {
  font-size: 0.8rem;
  color: #cbd5e1;
}

.jump-to-page input {
  width: 72px;
  padding: 8px 10px;
  border: 1px solid rgba(148, 163, 184, 0.2);
  border-radius: 10px;
  background: rgba(15, 23, 42, 0.8);
  color: #f8fafc;
  font: inherit;
}

.jump-to-page input:focus {
  outline: 2px solid rgba(56, 189, 248, 0.7);
  outline-offset: 1px;
}

.jump-to-page button {
  padding: 8px 12px;
  min-width: unset;
}

.pagination button {
  border: 1px solid transparent;
  border-radius: 12px;
  background: linear-gradient(135deg, #22c55e, #16a34a);
  color: #ecfdf5;
  font-weight: 700;
  padding: 10px 18px;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease, opacity 0.2s ease;
  box-shadow: 0 10px 18px rgba(34, 197, 94, 0.2);
}

.pagination button:disabled {
  opacity: 0.45;
  cursor: not-allowed;
  box-shadow: none;
}

.pagination button:not(:disabled):hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 22px rgba(34, 197, 94, 0.25);
}

.page-numbers {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  align-items: center;
  flex: 1;
}

.page-numbers button {
  background: rgba(148, 163, 184, 0.08);
  color: #e2e8f0;
  border-color: rgba(148, 163, 184, 0.16);
  min-width: 42px;
  min-height: 42px;
  padding: 0 12px;
  box-shadow: none;
}

.page-numbers button.active {
  background: linear-gradient(135deg, #38bdf8, #0ea5e9);
  color: #f0f9ff;
  border-color: rgba(56, 189, 248, 0.7);
  box-shadow: 0 12px 18px rgba(14, 165, 233, 0.22);
}

.page-numbers button:not(.active):hover {
  background: rgba(148, 163, 184, 0.12);
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 22px;
}

.card {
  background: rgba(15, 23, 42, 0.82);
  border: 1px solid rgba(148, 163, 184, 0.2);
  border-radius: 18px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.35);
}

.character-image {
  width: 100%;
  height: 280px;
  object-fit: cover;
  display: block;
}

.card-body {
  padding: 18px 18px 20px;
}

.card-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 10px;
  margin-bottom: 18px;
}

.card-top h2 {
  margin: 0;
  font-size: 1.4rem;
  line-height: 1.2;
}

.status {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.8rem;
  font-weight: 600;
  white-space: nowrap;
}

.status-icon {
  font-size: 0.9rem;
}

.location-label {
  margin: 0 0 6px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-size: 0.68rem;
  color: #94a3b8;
}

.location-name {
  margin: 0;
  color: #e2e8f0;
  font-size: 1rem;
}

@media (max-width: 640px) {
  .card-top {
    flex-direction: column;
    align-items: flex-start;
  }

  .pagination {
    justify-content: center;
  }

  .pagination-actions {
    width: 100%;
  }

  .pagination button {
    width: 100%;
  }

  .jump-to-page {
    width: 100%;
    justify-content: center;
  }

  .page-numbers {
    width: 100%;
  }
}
</style>
