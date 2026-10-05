<template>
  <header class="topo">
    <nav class="container topo-inner">
      <router-link to="/" class="brand-link">
        <span class="brand-mark brand-emoji" aria-label="Bandeira do Brasil">🇧🇷</span>
        <span>Deputados Federais</span>
      </router-link>
    </nav>

    <div class="hero-shell">
      <div class="container">
        <div class="hero-panel">
          <div class="hero-copy">
            <span class="eyebrow">Poder Legislativo Brasileiro</span>
            <h1 class="hero-title">Conheça a atuação e o perfil dos deputados em exercício.</h1>
            <p class="hero-text">
              Explore nomes, estados, partidos e perfis por meio de uma pesquisa rápida e intuitiva,
              com foco em transparência, dados públicos e acesso à informação.
            </p>
            <div class="hero-actions">
              <button type="button" class="btn btn-primaria" @click="irParaBusca">Buscar agora</button>
              <a href="https://dadosabertos.camara.leg.br/swagger/api.html" target="_blank" rel="noopener" class="btn btn-secundaria">Fonte oficial</a>
            </div>
          </div>

          <div class="stats-grid">
            <div class="stat-card">
              <span class="stat-label">Consulta</span>
              <strong>Nome, estado e partido</strong>
            </div>
            <div class="stat-card">
              <span class="stat-label">Detalhes</span>
              <strong>Gabinete, dados e redes</strong>
            </div>
            <div class="stat-card">
              <span class="stat-label">Favoritos</span>
              <strong>Salve perfis em destaque</strong>
            </div>
          </div>
        </div>
      </div>
    </div>
  </header>

  <main class="container py-4 flex-grow-1">
    <router-view />
  </main>

  <rodape-site />
</template>

<script>
import RodapeSite from './components/RodapeSite.vue'

export default {
  name: 'App',
  components: {
    RodapeSite
  },
  methods: {
    irParaBusca() {
      this.$router.push({ path: '/', hash: '#busca-nome' }).then(() => {
        this.$nextTick(() => {
          const campo = document.getElementById('busca-nome')
          if (!campo) return

          campo.focus({ preventScroll: true })
          campo.scrollIntoView({ behavior: 'smooth', block: 'center' })
        })
      })
    }
  }
}
</script>
