---
title: "Lab 1: Passing Pollinators"
toc: true
---

  1. What is the body mass and wing span distribution of each pollinator species observed?

```js
const pollinator_activity_data = await FileAttachment("data/pollinator_activity_data.csv").csv({typed: true});
const nectarCutoff = d3.quantile(
  pollinator_activity_data.map(d => d.nectar_production).sort(d3.ascending),
  0.85
);
const top15NectarData = pollinator_activity_data.filter(d => d.nectar_production >= nectarCutoff);
```

```js 
Plot.plot({
  title: "Body Mass vs. Wing Span",
  grid: true,
  marks: [
    Plot.ruleY([0], { stroke: "gray", strokeDasharray: "2 2" }),
    Plot.dot(pollinator_activity_data, {
      x: "avg_wing_span_mm", 
      y: "avg_body_mass_g", 
      r: 6,  
      tip: true,
      fill:"pollinator_species"
    })
  ],
  color:{
    legend: true,
    label: "pollinator_species"
  }
})

```
There is a clear positive relationship between body mass and wing span across bee types, but within each bee type the body mass stays fairly consistent though the wing span varies. The carpenter bee is the largest both in wing span and body mass, followed by the bumblebee, then by the honeybee. 

 2. What is the ideal weather (conditions, temperature, etc) for pollinating?

```js
const humidityMin = d3.min(pollinator_activity_data, d => d.humidity);
const humidityMax = d3.max(pollinator_activity_data, d => d.humidity);
Inputs.table([{humidityMin, humidityMax}])
```

```js
Plot.plot({
  title: "Ideal Weather for Pollinating",
  grid: true,
  color: { legend: true, label: "Weather Condition" },
  r: {
    legend: false,
    domain: [humidityMin, humidityMax],
    range: [2, 18]
  },
  marks: [
    Plot.dot(top15NectarData, {
      x: "nectar_production",
      y: "temperature",
      fill: "weather_condition",
      r: "humidity",
      opacity: 0.8,
      stroke: "white",
      strokeWidth: 0.6,
      tip: true
    }),
  ],
  x: { label: "Nectar Production" },
  y: { label: "Temperature (°C)"}
})
```

Ideal weather conditions for pollinating are not so cut and dry. Focusing only on the top 15% of nectar production instances, we see that there is a slight negative relationship between temperature and nectar production, and that there are more instances of cloudy weather in that top nectar production block compared to sunny. Humidity (represented by the size of the bubbles) does not seem to play a big part in creating ideal conditions for nectar production. 

  3. Which flower has the most nectar production?

```js
const beeImage = await FileAttachment("bee.png").url()
const nectarByFlower = d3
  .rollups(
    pollinator_activity_data,
    v => d3.max(v, d => d.nectar_production),
    d => d.flower_species
  )
  .map(([flower_species, max_nectar]) => ({flower_species, max_nectar}))
  .sort((a, b) => d3.descending(a.max_nectar, b.max_nectar));

const meanByFlower = d3
  .rollups(
    pollinator_activity_data,
    v => d3.mean(v, d => d.nectar_production),
    d => d.flower_species
  )
  .map(([flower_species, mean_nectar]) => ({flower_species, mean_nectar}));
```

```js
Plot.plot({
  title: "Nectar Production by Flower Type",
  grid: true,
   color: {
    legend: true,
    domain: ["Coneflower", "Lavender", "Sunflower"],
    range: ["#db2eb0", "#8e44ad", "#f0cf3e"]
  },
  marks: [
    Plot.dot(pollinator_activity_data, {
      x: "flower_species",
      y: "nectar_production",
      tip: true,
      fill: "flower_species"
    }),
    Plot.image(meanByFlower, {
      x: "flower_species",
      y: "mean_nectar",
      src: () => beeImage,
      width: 24,
      height: 24,
      dx: 14,
      dy: -8
    })
  ],
  x: { label: "Flower Species"},
  y: { label: "Nectar Production" , domain: [0.4, 1.3]}
})
```
Flower species' impact on nectar production is more clear. Sunflowers have much higher levels of nectar production than coneflowers or lavender. The hovering bee indicates the mean nectar production of each flower type, showing that coneflowers produce slightly more nectar on average than lavender. 

This page is where you can iterate. Follow the lab instructions in the [readme.md](./README).

