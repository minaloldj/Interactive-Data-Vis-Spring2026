---
title: "Lab 1: Passing Pollinators"
toc: true
---

  1. What is the body mass and wing span distribution of each pollinator species observed?

```js
const pollinator_activity_data = await FileAttachment("data/pollinator_activity_data.csv").csv({typed: true});
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
 2. What is the ideal weather (conditions, temperature, etc) for pollinating?

temperature,
humidity,
wind_speed,
weather_condition,
nectar_production,

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
    legend: true,
    label: "Humidity",
    domain: [humidityMin, humidityMax],
    range: [2, 18]
  },
  marks: [
    Plot.dot(pollinator_activity_data, {
      x: "temperature",
      y: "nectar_production",
      fill: "weather_condition",
      r: "humidity",
      opacity: 0.8,
      stroke: "white",
      strokeWidth: 0.6,
      tip: true
    })
  ],
  x: { label: "Temperature (°C)" },
  y: { label: "Nectar Production" }
})
```

  3. Which flower has the most nectar production?

```js
const flowerImage = await FileAttachment("flower.jpg").url()
const nectarByFlower = d3
  .rollups(
    pollinator_activity_data,
    v => d3.max(v, d => d.nectar_production),
    d => d.flower_species
  )
  .map(([flower_species, max_nectar]) => ({flower_species, max_nectar}))
  .sort((a, b) => d3.descending(a.max_nectar, b.max_nectar));
```
  
```js
Plot.plot({
  title: "Max Nectar Production by Flower Species",
  grid: true,
  marks: [
    Plot.barY(nectarByFlower, {
      x: "flower_species",
      y: "max_nectar",
      fill: "flower_species",
      tip: true
    }),
    Plot.image(nectarByFlower, {
      x: "flower_species",
      y: "max_nectar",
      src: flowerImage,
      width: 24,
      height: 24,
      dy: -14
    })
  ],
  x: { label: "Flower Species" },
  y: { label: "Max Nectar Production" }
})
```

This page is where you can iterate. Follow the lab instructions in the [readme.md](./README).

