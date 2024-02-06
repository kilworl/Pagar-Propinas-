<!-- app.vue -->
<template>
  <div id="app">
    <TipCapture @tip-submitted="handleTipSubmitted" />
    <!-- Otros componentes y vistas aquí -->
  </div>
</template>

<script>
import TipCapture from "@/components/TipCapture.vue";



export default {
  components: {
    TipCapture,
  
  },
  methods: {
    handleTipSubmitted(tipAmount) {
      console.log("Monto Total de Propinas Capturado:", tipAmount);
      // Puedes almacenar el monto total de propinas en el estado si es necesario
    },
  },
};
</script>

<style>
/* Estilos opcionales para la vista principal */
</style>
