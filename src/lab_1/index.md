---
title: "Lab 1: Passing Pollinators"
toc: true
---

  1. What is the body mass and wing span distribution of each pollinator species observed?

## simple plot
##avg_wing_span_mm
##avg_body_mass_g

```js
Inputs.table(pollinator_activity_data)
```

```js 
Plot.plot({
  grid: true,
  marks: [
    Plot.ruleY([0], { stroke: "gray", strokeDasharray: "2 2" }),
    Plot.dot(sample, {
      x: "avg_wing_span_mm",      // ← which key to read for x positioning
      y: "avg_body_mass_g",     // ← which key to read for y positioning
      r: 6,           // ← set value for radius
      fill: "blue",  // ← which key to read for fill
      tip: true
    })
  ]
})

```

## scatterplot
```js
Plot.plot({
  marks: [
    Plot.dot(pollinator_activity_data,
      {x:"avg_wing_span_mm", y:"avg_body_mass_g", tip: true, fill:"species"}
    )
  ]
})

```

  2. What is the ideal weather (conditions, temperature, etc) for pollinating?
  3. Which flower has the most nectar production?


This page is where you can iterate. Follow the lab instructions in the [readme.md](./README).

