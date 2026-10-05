<template>
  <section class="empty-state empty-state-404">
    <i class="bi bi-signpost-split"></i>
    <h1>Página não encontrada</h1>
    <p class="text-muted mb-4">O endereço acessado não existe ou foi movido.</p>
    <router-link to="/" class="btn btn-primaria">Ir para a lista de deputados</router-link>
  </section>
</template>

<script>
export default {
  name: 'PaginaNaoEncontrada'
}
</script>
