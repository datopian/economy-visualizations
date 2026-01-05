---
toc: false
---

```js
const goldRaw = FileAttachment("data/annual-gold-prices.csv").csv({typed: true});
const goldData = (await goldRaw).map(d => ({
  year: +d.Date,
  price: +d.Price
})).filter(d => d.price > 0);

const latestPrice = goldData[goldData.length - 1].price;
const firstPrice = goldData[0].price;
const allTimeHigh = d3.max(goldData, d => d.price);
const allTimeHighYear = goldData.find(d => d.price === allTimeHigh).year;
const priceIn1970 = goldData.find(d => d.year === 1970)?.price || 35;
const totalIncrease = ((latestPrice - priceIn1970) / priceIn1970 * 100).toFixed(0);
const yearsOfData = goldData.length;

const goldStandardData = goldData.filter(d => d.year <= 1970);
const modernEra = goldData.filter(d => d.year > 1970);
```



```js
display(html`<div class="dashboard-container">
  <div class="dashboard-top">
    <div class="sidebar-section">
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">Current Price</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value" style="color: var(--color-accent-orange)">$${latestPrice.toLocaleString()}</div>
            <div class="stat-card-subvalue">USD / oz</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">Since 1970</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value" style="color: var(--color-accent-green)">+${totalIncrease}%</div>
            <div class="stat-card-subvalue">Post-Gold Std</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">All-Time High</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value">$${allTimeHigh.toLocaleString()}</div>
            <div class="stat-card-subvalue">In ${allTimeHighYear}</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">Dataset</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value">${yearsOfData}</div>
            <div class="stat-card-subvalue">Years</div>
          </div>
        </div>
      </div>

      <div class="insights-card">
        <div class="insights-header">
          <h4 class="insights-title">Key Insights</h4>
        </div>
        <ul class="insights-list">
          <li>Gold prices surged +${totalIncrease}% after the end of the Gold Standard in 1971</li>
          <li>Modern era volatility is significantly higher than the stable 19th century</li>
          <li>The 2000s and 2010s saw rapid appreciation during financial uncertainty</li>
          <li>Gold remains a key hedge against inflation and currency devaluation</li>
        </ul>
      </div>
    </div>

    <div class="chart-card">
      <div class="chart-header">
        <div class="chart-title-row">
          <h3 class="chart-title">Gold Price Timeline (1833-2024) - Log Scale</h3>
        </div>
      </div>
      <div class="chart-content">
        ${resize((width) => {
          const isMobile = width < 640;
          return Plot.plot({
            width,
            height: isMobile ? 400 : 500,
            marginLeft: isMobile ? 55 : 60,
            marginRight: isMobile ? 20 : 40,
            marginBottom: isMobile ? 50 : 60,
            marginTop: 30,
            style: { background: "transparent" },
            x: { label: "Year", grid: true, ticks: isMobile ? 5 : 12, tickFormat: d => String(d) },
            y: { label: "Price per Troy Ounce (USD)", grid: true, type: "log", ticks: [10, 20, 50, 100, 200, 500, 1000, 2000, 3000] },
            marks: [
              Plot.areaY(goldData, {x: "year", y: "price", fill: "#fbbf24", fillOpacity: 0.2, curve: "catmull-rom"}),
              Plot.line(goldData, {x: "year", y: "price", stroke: "#fbbf24", strokeWidth: 3, curve: "catmull-rom"}),
              Plot.dot(goldData, {x: "year", y: "price", r: 2, fill: "transparent", tip: true, title: d => `${d.year}: $${d.price.toFixed(2)}`}),
              Plot.ruleX([1971], {stroke: "#dc2626", strokeWidth: 2, strokeDasharray: "6,4"}),
              Plot.text([{year: 1971, price: 1500}], {x: "year", y: "price", text: isMobile ? ["1971"] : ["1971: Nixon Ends Gold Standard"], fill: "#dc2626", fontSize: isMobile ? 9 : 11, fontWeight: "bold", dx: -5, textAnchor: "end"}),
              Plot.ruleX([2008], {stroke: "#f59e0b", strokeWidth: 2, strokeDasharray: "6,4"}),
              Plot.text([{year: 2008, price: 600}], {x: "year", y: "price", text: isMobile ? ["2008"] : ["2008: Financial Crisis"], fill: "#f59e0b", fontSize: isMobile ? 9 : 11, fontWeight: "bold", dx: -5, textAnchor: "end"}),
              Plot.ruleY([35], {stroke: "#22c55e", strokeWidth: 1.5, strokeDasharray: "3,3", opacity: 0.6}),
              Plot.dot([goldData[goldData.length - 1]], {x: "year", y: "price", r: 8, fill: "#fbbf24", stroke: "white", strokeWidth: 3})
            ]
          });
        })}
      </div>
    </div>
  </div>

  <div class="dashboard-bottom" style="grid-template-columns: 1fr 1fr;">
    <div class="chart-card">
      <div class="chart-header"><h3 class="chart-title">Price Growth by Era</h3></div>
      <div class="chart-content">
        ${resize((width) => {
          const isMobile = width < 640;
          const eraData = [
            { era: "Gold Standard (1833-1970)", avgPrice: d3.mean(goldStandardData, d => d.price), color: "#22c55e" },
            { era: "Modern Era (1971-2024)", avgPrice: d3.mean(modernEra, d => d.price), color: "#fbbf24" }
          ];
          return Plot.plot({
            width, height: isMobile ? 200 : 250, marginLeft: isMobile ? 140 : 160, marginRight: 50, marginBottom: 40,
            style: { background: "transparent" },
            x: { label: "Average Price (USD)", grid: true },
            y: { label: null },
            marks: [
              Plot.barX(eraData, {x: "avgPrice", y: "era", fill: "color", rx: 6, tip: true}),
              Plot.text(eraData, {x: "avgPrice", y: "era", text: d => `$${d.avgPrice.toFixed(0)}`, dx: 8, textAnchor: "start", fontSize: 12, fontWeight: "600", fill: d => d.color}),
              Plot.ruleX([0])
            ]
          });
        })}
      </div>
    </div>

    <div class="chart-card">
      <div class="chart-header"><h3 class="chart-title">Decade-by-Decade Average Prices</h3></div>
      <div class="chart-content">
        ${resize((width) => {
          const isMobile = width < 640;
          const decadeData = d3.rollups(goldData, v => d3.mean(v, d => d.price), d => Math.floor(d.year / 10) * 10)
            .map(([decade, avg]) => ({ decade: `${decade}s`, average: avg, decadeNum: decade })).filter(d => d.decadeNum >= 1840);
          return Plot.plot({
            width, height: isMobile ? 280 : 320, marginLeft: isMobile ? 50 : 60, marginRight: 20, marginBottom: isMobile ? 70 : 60,
            style: { background: "transparent" },
            x: { label: null, tickRotate: isMobile ? -45 : -30 },
            y: { label: "Average Price (USD)", grid: true },
            marks: [
              Plot.barY(decadeData, {x: "decade", y: "average", fill: d => d.decadeNum >= 1970 ? "#fbbf24" : "#22c55e", rx: 4, tip: true, title: d => `${d.decade}: $${d.average.toFixed(2)}`}),
              Plot.ruleY([0])
            ]
          });
        })}
      </div>
    </div>
  </div>

  <div class="dashboard-footer">
    <span>Built with Observable Framework</span>
    <span>Source: <a href="https://datahub.io/core/gold-prices" target="_blank">World Gold Council via DataHub.io</a></span>
  </div>
</div>`)
```
