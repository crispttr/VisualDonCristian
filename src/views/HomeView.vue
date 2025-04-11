<script setup>
import TheHeader from '../components/TheHeader.vue'
import TheIntro from '../components/TheIntro.vue'
import TheMap from '../components/TheMap.vue'
import TheAttendance from '../components/TheAttendance.vue'
import TheMainStations from '../components/TheMainStations.vue'
import TheMostCrowded from '../components/TheMostCrowded.vue'
import TheEmployees from '../components/TheEmployees.vue'
import TheEmployeesRate from '../components/TheEmployeesRate.vue'
import TheBoard from '../components/TheBoard.vue'
import TheDelayStats from '../components/TheDelayStats.vue'
import TheFrequentation from '../components/TheFrequentation.vue'
</script>

<template>
  <TheHeader />
  <TheBoard />
  <TheDelayStats />
  <TheIntro />
  <TheMap />
  <TheAttendance />
  <TheMainStations />
  <TheFrequentation />
  <TheMostCrowded />
  <TheEmployees />
</template>

<style scoped></style>
