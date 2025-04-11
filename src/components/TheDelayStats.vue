<script setup></script>

<template>
  <section>
    <div class="wrapper">
      <h2>Le saviez-vous ?</h2>
      <p>
        🚆 Hier, la gare de ... a enregistré x retards par rapport aux horaires
        prévus
      </p>
    </div>
  </section>
</template>

<style scoped>
section {
  background-color: var(--clr-primary-bg);
}

.wrapper {
  height: inherit;
  grid-column: 7 / span 3;
  display: flex;
  flex-direction: column;

  justify-content: center;
}
</style>
