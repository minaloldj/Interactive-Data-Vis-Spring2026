---
title: "Lab 3: Mayoral Mystery"
toc: false
---

## Lab 3: Mayoral Mystery

A mayoral candidate, despite having some exciting momentum, lost the vote in NYC. The campaign team is assessing what went wrong, what went right, and what could be improved upon for next time. 

First, let's look at the results across NYC districts. 

```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
results.forEach(d => d.boro = +String(d.boro_cd)[0]);
results.forEach(d => {
  d.total_votes = d.votes_candidate + d.votes_opponent;
  d.pct_yes = Math.round(100 * (d.votes_candidate / d.total_votes));
});
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });
events.forEach(d => d.boro = +String(d.boro_cd)[0]);
events.forEach(d => d.event_date = new Date(d.event_date))
```

```js
const districts = topojson.feature(nyc, nyc.objects.districts);
const votesByDistrict = new Map(results.map(d => [d.boro_cd, d.pct_yes]));

const income = topojson.feature(nyc, nyc.objects.districts);

const incomeColorMap = { low: "red", med: "yellow", high: "green" };
const incomeFillByDistrict = new Map(results.map(d => [Number(d.boro_cd), incomeColorMap[d.income_category]]));
```
```js
Plot.plot({
  title: "Results by District",
  projection: { type: "mercator", domain: income },
  color: {
    type: "quantile",
    n: 9,
    scheme: "oranges",
    label: "% Votes for Candidate",
    legend: true
  },
  marks: [
    Plot.geo(income, {
      fill: d => votesByDistrict.get(d.properties.BoroCD),
      stroke: "#404040",
      title: d => `District ${d.properties.BoroCD}: ${votesByDistrict.get(d.properties.BoroCD)}% votes for candidate`,
      tip: true
    }),
    Plot.dot(income.features, Plot.centroid({
      stroke: d => incomeFillByDistrict.get(d.properties.BoroCD),
      fill: "none",
      symbol: "triangle",
      r: 8,
      strokeWidth: 2
    }))
  ]
})
```
The Bronx and Brooklyn had the strongest turnout for the candidate, whereas Lower Manhattan and Staten Island had the lowest. To understand why this may be, let's assess the Get Out the Vote (GOTV) campaign. 
```js
const selectedIncome = view(Inputs.select(["All", ...Array.from(new Set(results.map(d => d.income_category)))]));
```
```js
Plot.plot({
  title: "Success of the GOTV campaign",
  color: { scheme: "oranges", label: "% Yes Votes", legend: true },
  marks: [
    Plot.dot(selectedIncome === "All" ? results : results.filter(d => d.income_category === selectedIncome), {
      x: "gotv_doors_knocked",
      y: "candidate_hours_spent",
      fill: "pct_yes",
      r: 5,
      tip: true
    })
  ],
  x: { label: "Doors Knocked" },
  y: { label: "Candidate Hours" }
})
```

The GOTV campaign was not very strong. The candidate spending time in each district is not strongly correlated with % yes votes - the candidate spent very little time in a number of districts and did relatively well, and spent a lot of time in other districts and did poorly. Alternatively, door knocking does seem to have helped get stronger turnouts in some districts, whereas others seem to have already been in support of the candidate. Interestingly, the districts where the candidate did the worst is where they spent the most time, yet had very little door knocking. 

To unpack this more, we can use the ticker to filter by income levels. The candidate seems to have spent considerably more time in high income districts (18+ hours), but there were not a lot of doors knocked, comparatively.Still, voter turnout for the candidate was relatively low despite the time spent there by the candidate. Once we look at middle, there seems to be a clearer picture showing the success of the door knocking campaign, and the little to no effect that candidate hours had on voting. Among low income districts, it's varied. These districts had the most doors knocked, but it doesn't seem to have a clean correlation with voter turnout. However, we can conclude that candidate time spent was not as effective overall as doors knocked. 

Next, we should look at how voters felt about the candidate's policies. 


```js
const issues = ["affordable_housing_alignment", "public_transit_alignment", "childcare_support_alignment", "small_business_tax_alignment", "police_reform_alignment"];
survey.forEach(d => {
  if (d.voted_for === "" || d.voted_for == null) d.voted_for = "No Vote";
});
```

```js
const selectedIssue = view(Inputs.select(["All", ...issues], {
  format: d => d === "All" ? "All Issues" : d.replace("_alignment", "").replaceAll("_", " ")
}));
```

```js
Plot.plot({
  title: selectedIssue === "All" ? "All Issues" : selectedIssue.replace("_alignment", "").replaceAll("_", " "),
  fx: { domain: ["Candidate", "Opponent", "No Vote"] },
  color: { type: "ordinal", scheme: "oranges", legend: true },
  marks: [
    Plot.barY(
      selectedIssue === "All"
        ? survey.flatMap(d => issues.map(issue => ({ ...d, score: +d[issue] })))
        : survey.map(d => ({ ...d, score: +d[selectedIssue] })),
      Plot.groupX({y: (group) => {
        const issueCount = selectedIssue === "All" ? issues.length : 1;
        const total = survey.filter(d => d.voted_for === group[0].voted_for).length * issueCount;
        return (group.length / total) * 100;
      }}, {
        x: "score",
        fx: "voted_for",
        fill: "score",
        tip: true
      })
    ),
    Plot.ruleY([0])
  ],
  x: { label: "Score (1-5)", tickFormat: d => d },
  y: { label: "% of Voters", domain: [0, 100] }
})
```
This plot uses survey responses from voters for the candidate, voters against the candidate, and non-voters, and their opinion on the candidate's take on the major issues in the city, where 1 is not aligned and 5 is in support. This plot takes a percentage of each group of voters surveyed to understand the general attitude towards the candidate's policies - however, it is important to note that there was a relatively small sample size of voters surveyed, and so we should not assume these takeaways are true across the voter blocs. 

First, we look at an aggregate across all issues. Interestingly, there isn't a clearly stronger alignment from candidate voters to the candidate's issues compared to the other voter blocs. To better unpack this, we have to look at each issue. 

#### Affordable housing
Candidate voters were largely aligned (4+) with the candidate's affordable housing policies. Opponent voters also were somewhat aligned with the candidate's policies, but features more people who were not aligned (<=3) with the policies. The no vote bloc was also largely aligned with the candidate's policies. 

#### Public transit
Somewhat similar distribution as the affordable housing policy. Candidate voters were largely aligned, opponent voters were still fairly strongly aligned but features more poeple who were not aligned (<=3), whereas the no vote block was actually the strongest aligned. 

#### Childcare support
Candidate voters were strongly aligned with the candidate's childcare policies, with nearly 50% of those surveyed responding 5 to the survey. Opponent voters were much less aligned, with over 30% of them responding 2. Again, the no vote bloc was still strongly aligned with the candidate, with nearly 80% answering either 4 or 5. 

#### Small business tax 
While voters were somewhat aligned with the candidate's policies on small business taxes, no one in any voter bloc responded 5, meaning there was no strong alignment. However, nearly 90% of candidate voters responded either 3 or 4, showing that there was a general support, but not a strong one. Opponent voters largely responded 3, meaning they were not for or against the policies, and gain the no vote block was somewhat in support, answering largely 3 or 4. 

#### Police reform
This was clearly the least supported policy of the candidate's. Candidate voters are split evenly between 1 or 2, meaning very low support for the candidate's policies. Both the opponent voters and the no voting bloc mostly answered 1, with a smaller group responding 2. 

## Takeaways: 

Door knocking was more effective than candidate hours spent. In the future, more effort should be put in grassroots canvassing (maybe a phone campaign?) since that has largely worked more. 

The candidate can then spend more time explaining their policies, especially to individuals/districts that did not vote. The no vote bloc was not necessarily against the candidate, but clearly, despite moderate alignment with the candidate's policies, they were not motivated enough to go vote. More effort should be spent on that bloc than the opponent voters, since they are less aligned with the candidate's policies.