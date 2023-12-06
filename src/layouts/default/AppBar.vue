<template>
	<v-app-bar order="1" :elevation="11">
		<v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-app-bar-title>
        <v-icon icon="mdi mdi-rocket-launch" />
        Rocket GO
      </v-app-bar-title>
	</v-app-bar>
	<v-navigation-drawer order="0" v-model="drawer" temporary floating>
		<v-list :items="items"></v-list>
	</v-navigation-drawer>
</template>

<script setup>
  import { ref } from 'vue'
	let drawer = ref(false);
	let items = ref([
		{title: 'Opciones' },
		{type: 'divider'},
		{title: 'Registrarse', value: 'inicio', props:{prependIcon:'mdi mdi-account', to:'/'}},
		{title: 'Iniciar Sesion', value: 'tabla', props:{prependIcon:'mdi mdi-account-key', to:'/Tabla'}},
	]);
</script>
