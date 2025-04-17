<template>
  <section class="section-heatmap">
    <div class="wrapper">
      <h2>Augmentation du nombre d'usager.ères</h2>
      <h3>2020 à 2024</h3>
      <div ref="heatmapContainer" class="heatmap-container"></div>
      <div ref="tooltip" class="tooltip" />
    </div>
  </section>
</template>

<script setup>
import * as d3 from 'd3'
import { onMounted, ref } from 'vue'

const tooltip = ref(null)
const heatmapContainer = ref(null)

onMounted(async () => {
  const data = await d3.json('/frequentationData.json')

  const grouped = d3.rollup(
    data,
    (v) => d3.sum(v, (d) => d.anzahl_bahnhofbenutzer),
    (d) => d.bahnhof_gare_stazione,
    (d) => d.jahr
  )

  const allYears = ['2020', '2021', '2022', '2023', '2024']
  const sumYears = ['2020', '2021', '2022', '2023']

  // Calculer la somme de 2020 à 2023 pour chaque gare
  const garesTotals = Array.from(grouped.entries()).map(
    ([gare, valuesByYear]) => {
      const sum = sumYears.reduce(
        (acc, year) => acc + (valuesByYear.get(year) || 0),
        0
      )
      return {
        gare,
        total: sum,
      }
    }
  )

  // Trier et sélectionner le top 5
  const top5 = garesTotals
    .sort((a, b) => b.total - a.total)
    .slice(0, 5)
    .map((d) => d.gare)

  // Créer les données pour la heatmap sur toutes les années
  const heatmapData = []
  top5.forEach((gare, y) => {
    allYears.forEach((year, x) => {
      heatmapData.push({
        gare,
        year,
        value: grouped.get(gare)?.get(year) || 0,
        x,
        y,
      })
    })
  })

  const width = 700
  const height = 450
  const cellSize = 60

  const svg = d3
    .select(heatmapContainer.value)
    .append('svg')
    .attr('width', width)
    .attr('height', height)

  const color = d3
    .scaleLinear()
    .domain(d3.extent(heatmapData, (d) => d.value))
    .range(['#fae8e6', '#D72E20'])

  // Dessiner les cellules de la heatmap
  svg
    .selectAll('rect')
    .data(heatmapData)
    .join('rect')
    .attr('x', (d) => d.x * cellSize + 100)
    .attr('y', (d) => d.y * cellSize + 80)
    .attr('width', cellSize)
    .attr('height', cellSize)
    .attr('rx', 10)
    .attr('fill', (d) => color(d.value))
    .attr('stroke', 'white')
    .on('mouseover', (event, d) => {
      tooltip.value.style.opacity = 1
      tooltip.value.innerHTML = `
        <h3>📍 ${d.gare}</h3>
        <p>Année : ${d.year}</p>
        <p>Usagers : ${d3.format(',')(d.value)}</p>
      `
      tooltip.value.style.left = `${event.pageX + 10}px`
      tooltip.value.style.top = `${event.pageY - 40}px`
    })
    .on('mouseout', () => {
      tooltip.value.style.opacity = 0
    })

  // Affichage des années en haut
  svg
    .selectAll('text.year')
    .data(allYears)
    .join('text')
    .attr('class', 'year')
    .attr('x', (d, i) => i * cellSize + 130)
    .attr('y', 50)
    .attr('text-anchor', 'middle')
    .attr('transform', (d, i) => `rotate(-45 ${i * cellSize + 130},50)`)
    .text((d) => d)

  // Affichage des gares à gauche
  svg
    .selectAll('text.gare')
    .data(top5)
    .join('text')
    .attr('class', 'gare')
    .attr('x', 90)
    .attr('y', (d, i) => i * cellSize + 115)
    .attr('text-anchor', 'end')
    .text((d) => d)

  // Légende
  const legend = svg
    .append('g')
    .attr('transform', `translate(100, ${top5.length * cellSize + 100})`)

  const legendScale = d3
    .scaleLinear()
    .domain(d3.extent(heatmapData, (d) => d.value))
    .range([0, 200])

  const defs = svg.append('defs')
  const gradient = defs
    .append('linearGradient')
    .attr('id', 'legendGradient')
    .attr('x1', '0%')
    .attr('x2', '100%')

  gradient.append('stop').attr('offset', '0%').attr('stop-color', '#fae8e6')
  gradient.append('stop').attr('offset', '100%').attr('stop-color', '#D72E20')

  legend
    .append('rect')
    .attr('width', 220)
    .attr('height', 15)
    .style('fill', 'url(#legendGradient)')
    .attr('rx', 10)

  svg
    .append('text')
    .attr('x', 70)
    .attr('y', top5.length * cellSize + 112)
    .attr('text-anchor', 'end')
    .style('font-weight', 'bold')
    .text('Peu')

  svg
    .append('text')
    .attr('x', 350)
    .attr('y', top5.length * cellSize + 112)
    .style('font-weight', 'bold')
    .text('Beaucoup')
})
</script>

<style scoped>
.section-heatmap {
  background-color: var(--clr-primary-bg);
  font-family: var(--txt-font-txt);
  display: flex;
  justify-content: flex-end; /* Aligne tout à droite */
}
.wrapper {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-end; /* Centre le contenu à droite */
  padding: 2rem;
  text-align: right; /* Aligne le texte à droite */
}

.heatmap-container svg {
  max-width: 100%;
  height: auto;
  margin-left: auto; /* Pousse le SVG vers la droite */
}

.tooltip {
  position: absolute;
  background: white;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.3s ease;
  font-family: var(--txt-font-txt);
  font-size: 0.9rem;
  text-align: left; /* Aligne le texte de l'infobulle à droite */
}
</style>
