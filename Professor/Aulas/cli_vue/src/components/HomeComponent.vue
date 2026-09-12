<template>
  <div>
    <header>
      <p>Seja ben vindo: {{ nome }}</p>
      <img width="200" :height="altura" :src="urlImgBotafogo" />
    </header>
  </div>
</template>

<script>
export default {
  name: "HomeComponent",
  data() {
    return {
      nome: "Fernando",
      altura: 150,
      urlImgBotafogo:
        "https://botafogofrsocialolimpico.com.br/wp-content/uploads/2024/06/5.png",
    };
  },
  mounted() {
    setTimeout(() => {
      this.urlImgBotafogo =
        "https://png.pngtree.com/png-vector/20231019/ourmid/pngtree-user-profile-avatar-png-image_10211467.png";
    }, 5 * 1000);
  },
};
</script>
