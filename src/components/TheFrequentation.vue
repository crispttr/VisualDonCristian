<template>
  <div>
    <HeatMap :data="heatmapData" v-if="heatmapData.length" />
    <p v-else>Chargement des données...</p>
  </div>
</template>

<script>
import axios from 'axios'
import HeatMap from './HeatMap.vue'

export default {
  name: 'TheFrequentation',
  components: { HeatMap },
  data() {
    return {
      heatmapData: [],
    }
  },
  async mounted() {
    try {
      // 🔁 Charger le fichier JSON local
      const res = await axios.get('src/data/frequentationData.json')
      const rawData = res.data

      // 🎯 Filtrer les années 2020 à 2024 et données valides
      const filtered = rawData.filter((entry) => {
        const year = parseInt(entry.jahr)
        return (
          year >= 2020 && year <= 2024 && entry.anzahl_bahnhofbenutzer !== null
        )
      })

      // 🎯 Grouper par gare + année
      const grouped = {}
      filtered.forEach((entry) => {
        const gare = entry.bahnhof_gare_stazione
        const year = entry.jahr
        const count = entry.anzahl_bahnhofbenutzer

        if (!grouped[gare]) {
          grouped[gare] = { gare }
        }

        if (!grouped[gare][year]) {
          grouped[gare][year] = 0
        }

        grouped[gare][year] += count
      })

      // 🎯 Trier les gares par total fréquentation et garder top 5
      const totalPerGare = Object.values(grouped).map((gareData) => {
        const total = Object.entries(gareData)
          .filter(([key]) => key !== 'gare')
          .reduce((sum, [_, val]) => sum + val, 0)
        return { ...gareData, total }
      })

      const top5 = totalPerGare
        .sort((a, b) => b.total - a.total)
        .slice(0, 5)
        .map(({ total, ...rest }) => rest)

      this.heatmapData = top5
    } catch (err) {
      console.error('Erreur chargement du fichier JSON :', err)
    }
  },
}
</script>
