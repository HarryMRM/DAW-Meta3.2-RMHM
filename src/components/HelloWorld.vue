<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <v-img height="300" src="@/assets/logo.svg" />
      <h1 class="text-h2 font-weight-bold">Haz despegar esos sueños</h1>

      <div class="py-14" />

      <v-row class="d-flex align-center justify-center">
        <v-col cols="auto">
        </v-col>

        <v-col cols="auto">
          <router-link to="/login">
            <v-btn
            color="primary"
            min-width="228"
            rel="noopener noreferrer"
            size="x-large"
            target="_blank"
            variant="flat"
            >
            <v-icon
              icon="mdi mdi-currency-usd"
              size="large"
              start
            />
            Apoyar
            </v-btn>
          </router-link>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script setup>
  //
</script>
