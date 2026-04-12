---
title: "Lab 2: Subway Staffing"
toc: true
---

## Lab 2: Subway Staffing
### Mina Loldj

<!-- Import Data -->
```js
const incidents = FileAttachment("./data/incidents.csv").csv({ typed: true })
const local_events = FileAttachment("./data/local_events.csv").csv({ typed: true })
const upcoming_events = FileAttachment("./data/upcoming_events.csv").csv({ typed: true })
const ridership = FileAttachment("./data/ridership.csv").csv({ typed: true })
```

<!-- Include current staffing counts from the prompt -->
```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
}
```
<i>How did local events impact ridership in summer 2025? What effect did the July 15th fare increase have?</i>

#### Number of riders by station
<select id="station-select"></select>
<div id="plot"></div>

```js
const stations = Array.from(
  new Set(local_events.map(d => d.nearby_station))
).sort();

const colorScale = {
  type: "categorical",
  domain: stations
};

const select = document.getElementById("station-select");

stations.forEach(st => {
  const opt = document.createElement("option");
  opt.value = st;
  opt.textContent = st;
  select.appendChild(opt);
});

function drawPlot(selectedStation) {
  const plot = Plot.plot({
    height: 500,
    color: { ...colorScale, legend: true },
    marks: [
      Plot.line(ridership, {
        x: "date",
        y: "exits",
        stroke: "station",
        strokeOpacity: d => d.station === selectedStation ? 1 : 0.1,
        strokeWidth: d => d.station === selectedStation ? 2.5 : 1,
        pointerEvents: "none"
      }),
      Plot.line(
        ridership.filter(d => d.station === selectedStation),
        {
          x: "date",
          y: "exits",
          stroke: "station",
          strokeWidth: 3,
          tip: true
        }
      ),
      Plot.dot(local_events, {
        x: "date",
        y: "estimated_attendance",
        stroke: "nearby_station",
        fill: "nearby_station",
        opacity: d => d.nearby_station === selectedStation ? 1 : 0.1,
        r: d => d.nearby_station === selectedStation ? 5 : 3,
        pointerEvents: "none"
      }),
      Plot.dot(
        local_events.filter(d => d.nearby_station === selectedStation),
        { x: "date",
          y: "estimated_attendance",
          stroke: "nearby_station",
          fill: "nearby_station",
          r: 5,
          channels: {
            "Event": { value: "event_name", label: "Event" },
          },
          tip: {
            format: {
              Event: true,
              x: true,
              y: true,
              stroke: false,
              fill: false,
            }
        }}
      ),
      Plot.ruleX(
        ridership.filter(d => d.station === selectedStation),
        Plot.pointer({
          x: "date",
          stroke: "black",
          strokeOpacity: 0.4,
          strokeWidth: 1
        })
      ),
      Plot.ruleX([new Date("2025-07-15")], {
        stroke: "black",
        strokeOpacity: 0.5,
        strokeDasharray: "4 4",
        strokeWidth: 1.5
      }),
      Plot.text(
        [{ date: new Date("2025-07-15"), label: "Fare increase" }],
        {
          x: "date",
          y: () => 30000,
          text: "label",
          dy: -10,
          textAnchor: "middle",
          fontWeight: "bold"
        }),
    ]
  });
  document.getElementById("plot").innerHTML = "";
  document.getElementById("plot").appendChild(plot);
}

drawPlot(stations[0]);

select.addEventListener("change", e => {
  drawPlot(e.target.value);
});
```
This visualization allows us to choose a specific station to investigate its ridership rates, the attendance at nearby events, and compare any changes directly to the July 15 fare increase. Looking at a few stations, while there do seem to be spikes because of events, there are clearly other catalysts impacting ridership rates. 

For example, 28th St station has two clear spikes due to the June 21 and 27 events. There also seems to be a decrease the day of the fare increase. However, there are a number of other spikes and dips, not explained by nearby events nor the fare increase. Grand Central similarly has experienced spikes due to nearby events, but most of the variation is unexplained.  

96th St is slightly more interpretable, with two clear major spikes on dates of nearby events, and a clear drop right after the fare increase. 50th St also has clear spikes in relation to nearby events, with a slight drop in ridership the day of the fare increase, but recovered the next day. 

<i>How do the stations compare when it comes to response time? Which are the best, which are the worst?</i>

```js
Plot.plot({
  title: "Median Response Times by Severity, per Station",
  subtitle: "with minimum and maximum ranges",
  marginLeft: 140,
    x: { label: "Median response time (minutes)", grid: true },
    y: { label: "Station" },
  color: {
    legend: true,
    domain: ["low", "medium", "high"],
    range: ["#378ADD", "#EF9F27", "#E24B4A"]
  },
  marks: [
    Plot.ruleY(incidents,
      Plot.groupY(
        { x1: "min", x2: "max" },
        {
          x: "response_time_minutes",
          y: "station",
          stroke: "#D3D1C7",
          strokeWidth: 2,
          sort: { y: "x", reduce: "median", reverse: false },
        }
      )
    ),
    Plot.dot(incidents,
      Plot.groupY(
        { x: "median" },
        {
          x: "response_time_minutes",
          y: "station",
          fill: "severity",
          r: 6,
          sort: { y: "x", reduce: "median", reverse: false },
          tip: true,
          channels: {
            "Station": { value: "station", label: "Station" },
            "Severity": { value: "severity", label: "Severity" },
          }
        }
      )
    ),
  ]
})
```
This visualization shows us the median response time for low, medium, and high severity incidents. The gray line indicates the minimum and maximum response time per station. 

Focusing on high severity incidents, the stations with the most prompt median responses are Herald Square and 50th St, and the slowest are West 4th and Bowling Green. Astor Pl has the widest range of response times, and overall slow median response rates. 
  

<i>Which three stations need the most staffing help for next summer based on the 2026 event calendar?</i>

```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
};

const attendanceByStation = d3.rollup(
  upcoming_events,
  v => d3.sum(v, d => d.expected_attendance),
  d => d.nearby_station
);

const responseByStation = d3.rollup(
  incidents,
  v => d3.median(v, d => d.response_time_minutes),
  d => d.station
);

const stationSummary = Array.from(
  attendanceByStation,
  ([station, total_attendance]) => ({
    station,
    total_attendance,
    median_response: responseByStation.get(station) ?? null,
    staffing: currentStaffing[station] ?? null,
  })
)
  .filter(d => d.median_response != null && d.staffing != null)
  .sort((a, b) => b.total_attendance - a.total_attendance);
```

```js
Plot.plot({
  title: "Staff count and expected event attendance, by station",
  subtitle: "with minimum and maximum ranges",
  width: 1100,
  height: 700,
  marginLeft: 60,
  marginBottom: 130,
  marginRight: 40,
  marginTop: 40,
  x: {
    label: "Station →",
    tickRotate: -45,
    domain: stationSummary.map(d => d.station),
  },
  y: {
    label: "Total expected attendance (2026)",
    grid: true,
  },
  color: {
    type: "linear",
    domain: [
      d3.min(stationSummary, d => d.median_response),
      d3.max(stationSummary, d => d.median_response),
    ],
    range: ["#378ADD", "#E24B4A"], 
    label: "Median response time (min)",
    legend: true,
  },
  r: {
    domain: [
      d3.min(stationSummary, d => d.staffing),
      d3.max(stationSummary, d => d.staffing),
    ],
    range: [5, 28],
    label: "Current staffing count",
    legend: true,
  },
  marks: [
   Plot.dot(stationSummary, {
      x: "station",
      y: "total_attendance",
      fill: "median_response",
      r: "staffing",
      stroke: "white",
      strokeWidth: 1,
      fillOpacity: 0.9,
      tip: true
    }),
    // inside marks array
    Plot.text(
      [{ x: stationSummary[stationSummary.length - 1].station, y: d3.max(stationSummary, d => d.total_attendance) }],
      {
        x: "x",
        y: "y",
        text: () => "Bubble size = staffing count\n(larger = more staff)",
        fontSize: 15,
        textAnchor: "end",
        lineAnchor: "top",
      }
    ),
  ],
})
```
This graph shows us the staffing count and average response time for each station, as well as the total expected attendance nearby each station for 2026 events. The stations that would need the most additional support are those with high total expected attendance, small bubble size indicating a small staff, and a purple to red color indicating slower average response time. Canal St is the most evident station that needs support, with 42nd st and 23st following. While Chambers St is slightly purple, meaning it has a middle-range average response time, it already has a high staff, so I wouldn't fixate additional efforts there.  

<i>[BONUS] If you had to prioritize one station to get increased staffing, which would it be and why?</i>

Canal St needs the most support, given that nearby events are expected to attract over 70,000 visitors, there is currently only a staff of 4, and it has a slow response time compared to other stations. 