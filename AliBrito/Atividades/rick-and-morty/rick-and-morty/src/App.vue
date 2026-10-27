<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

type Character = {
  id: number
  name: string
  status: 'Alive' | 'Dead' | 'unknown'
  image: string
  location: { name: string }
}

type ApiInfo = {
  count: number
  pages: number
  next: string | null
  prev: string | null
}

const characters = ref<Character[]>([])
const info = ref<ApiInfo>({ count: 0, pages: 0, next: null, prev: null })
const currentPage = ref(1)
const loading = ref(false)
const error = ref('')

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

const pageNumbers = computed(() => {
  const totalPages = info.value.pages || 1
  return Array.from({ length: totalPages }, (_, index) => index + 1)
})

const fetchCharacters = async (page = 1) => {
  loading.value = true
  error.value = ''

  try {
    const response = await fetch(`https://rickandmortyapi.com/api/character?page=${page}`)

    if (!response.ok) {
      throw new Error('Não foi possível carregar os personagens.')
    }

    const data = await response.json()
    characters.value = data.results
    info.value = data.info
    currentPage.value = page
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Erro ao carregar os personagens.'
  } finally {
    loading.value = false
  }
}

const changePage = (page: number) => {
  if (page < 1 || page > (info.value.pages || 1)) return
  fetchCharacters(page)
}

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

    <section v-if="error" class="alert" role="alert">
      {{ error }}
    </section>

    <section v-if="loading" class="loading" aria-live="polite">
      Carregando personagens...
    </section>

    <section v-else class="content">
      <nav class="pagination" aria-label="Paginação de personagens">
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

        <button type="button" :disabled="currentPage === info.pages" @click="changePage(currentPage + 1)">
          Próxima
        </button>
      </nav>

      <div class="cards-grid">
        <article v-for="character in characters" :key="character.id" class="card">
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
  margin-bottom: 24px;
}

.pagination button {
  border: none;
  border-radius: 999px;
  background: #22c55e;
  color: #052e16;
  font-weight: 700;
  padding: 10px 16px;
  cursor: pointer;
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.pagination button:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.pagination button:not(:disabled):hover {
  transform: translateY(-1px);
}

.page-numbers {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
}

.page-numbers button {
  background: rgba(148, 163, 184, 0.15);
  color: #e2e8f0;
  min-width: 42px;
}

.page-numbers button.active {
  background: #38bdf8;
  color: #082f49;
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
}
</style>
