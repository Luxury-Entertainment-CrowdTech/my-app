<!-- src/components/Tangle/TopNavBar.vue-->

<template>
    <v-app-bar class="color-green" app color="primary" dark>
        <!-- Logo e ícono de menú para pantallas pequeñas -->
        <v-app-bar-nav-icon @click="$emit('toggleDrawer')"></v-app-bar-nav-icon>
        <v-img :src="logo" alt="Logo" height="40" max-width="5%"></v-img>
        <v-col cols="auto" class="subtitle-1 white--text">
            Luxen
        </v-col>
        <v-spacer></v-spacer>
        <!-- Enlaces de navegación -->
        <v-btn text href="/profile">Mi Perfil</v-btn>
        <v-btn text @click="$emit('logout')">Cerrar Sesión</v-btn>
    </v-app-bar>
</template>
  
<script>
export default {
    name: 'TopNavBar',
    props: {
        logo: String,
    },
    // Aquí puedes incluir cualquier lógica adicional o métodos si es necesario
};
</script>
  
<style scoped>

.color-green {
    background-color: rgb(86,147,44) !important;
}

</style>