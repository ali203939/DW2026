<template>
  <div class="resumo-card">
    <div class="card-header">
      <div>
        <span class="fw-semibold"><i class="bi bi-bar-chart-fill me-2 texto-primario"></i>Deputados por partido</span>
      </div>
      <div class="small text-muted">Resultado atual</div>
    </div>

    <div class="resumo-body">
      <div v-for="item in ranking" :key="item.sigla" class="ranking-item">
        <div class="ranking-topo">
          <div class="ranking-nome">
            <span class="sigla-badge">{{ item.sigla }}</span>
          </div>
          <span class="ranking-total">{{ item.total }}</span>
        </div>

        <div class="barra-wrap">
          <div class="barra-preenchida" :style="{ width: `${(item.total / maior) * 100}%` }"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ResumoPartidos',
  props: {
    deputados: { type: Array, required: true },
    limite: { type: Number, default: 8 }
  },
  computed: {
    ranking() {
      const contagem = {}
      for (const deputado of this.deputados) {
        contagem[deputado.siglaPartido] = (contagem[deputado.siglaPartido] || 0) + 1
      }
      return Object.entries(contagem)
        .map(([sigla, total]) => ({ sigla, total }))
        .sort((a, b) => b.total - a.total)
        .slice(0, this.limite)
    },
    maior() {
      return this.ranking.length ? this.ranking[0].total : 1
    }
  }
}
</script>
