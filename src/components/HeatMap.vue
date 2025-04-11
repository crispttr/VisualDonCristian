<template>
  <section>
    <div class="heatmap">
      <h3>Augmentation du nombre d’usager·ère·s (2020 - 2024)</h3>

      <!-- 🔥 Heatmap -->
      <div class="heatmap-grid">
        <!-- En-têtes (années) -->
        <div class="header-cell"></div>
        <div v-for="year in years" :key="year" class="header-cell year-label">
          {{ year }}
        </div>

        <!-- Gares + cellules -->
        <template v-for="row in data" :key="row.gare">
          <div class="station-cell">{{ row.gare }}</div>
          <div
            v-for="year in years"
            :key="`${row.gare}-${year}`"
            class="data-cell"
            :style="{ backgroundColor: getColor(row[year]) }"
            @mousemove="showTooltip($event, row[year], row.gare, year)"
            @mouseleave="hideTooltip"
          ></div>
        </template>
      </div>

      <!-- 🧠 Tooltip custom -->
      <div
        v-if="tooltip.visible"
        class="tooltip"
        :style="{ top: tooltip.y + 'px', left: tooltip.x + 'px' }"
      >
        {{ tooltip.content }}
      </div>
      <!-- 🟡 Légende -->
      <div class="legend">
        <span>Peu</span>
        <div class="legend-bar"></div>
        <span>Beaucoup</span>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: 'HeatMap',
  props: {
    data: Array, // [{ gare: 'Genève', 2020: 123456, ... }]
  },
  data() {
    return {
      tooltip: {
        visible: false,
        x: 0,
        y: 0,
        content: '',
      },
    }
  },
  computed: {
    years() {
      return ['2020', '2021', '2022', '2023', '2024']
    },
  },
  methods: {
    getColor(value) {
      if (!value) return '#eee'
      const min = 50000
      const max = 500000
      const ratio = (value - min) / (max - min)
      const clamped = Math.min(Math.max(ratio, 0), 1)
      const red = 230
      const greenBlue = Math.floor(230 * (1 - clamped))
      return `rgb(${red},${greenBlue},${greenBlue})`
    },
    showTooltip(event, value, gare, year) {
      if (!value) return
      this.tooltip.content = `${gare} (${year}) : ${value.toLocaleString()} passagers`
      this.tooltip.visible = true
      this.tooltip.x = event.clientX + 10
      this.tooltip.y = event.clientY + 10
    },
    hideTooltip() {
      this.tooltip.visible = false
    },
  },
}
</script>

<style scoped>
:root {
  --sbb-blue: #2d327d;
  --sbb-red: #eb0000;
  --txt-font-display: Helvetica, sans-serif;
}

* {
  background-color: white;
}

.heatmap {
  font-family: var(--txt-font-display);
  padding: 1.5rem;
}

.legend {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-size: 0.9rem;
  margin-bottom: 1.5rem;
}

.legend-bar {
  flex-grow: 1;
  height: 1rem;
  min-width: 8rem;
  background: linear-gradient(to right, rgb(255, 230, 230), rgb(230, 0, 0));
  border-radius: 0.5rem;
}

.heatmap-grid {
  display: grid;
  grid-template-columns: 10rem repeat(5, 1fr);
  gap: 1rem;
  align-items: center;
}

.header-cell {
  font-weight: bold;
  text-align: center;
  padding: 0.5rem;
  font-size: 0.95rem;
}

.year-label {
  transform: rotate(-45deg);
}

.station-cell {
  font-weight: bold;
  padding: 0.5rem;
  background-color: #f4f4f4;
  font-size: 0.95rem;
}

.data-cell {
  height: 2.5rem;
  border-radius: 0.4rem;
  transition: background-color 0.3s ease;
  cursor: pointer;
}

.tooltip {
  position: fixed;
  background-color: #333;
  color: white;
  padding: 0.4rem 0.6rem;
  border-radius: 0.4rem;
  font-size: 0.85rem;
  pointer-events: none;
  white-space: nowrap;
  z-index: 999;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}
</style>
