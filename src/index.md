---
toc: false
---

<div class="portaljs-banner">
  <div class="portaljs-banner-content">
    <div class="portaljs-banner-text">
      <p class="portaljs-banner-title">Create beautiful data portals with PortalJS 🌀</p>
    </div>
  </div>
  <a href="https://www.portaljs.com/" target="_blank" class="portaljs-banner-cta">
    Get started free
  </a>
</div>

```js
// Load data for high level stats
const goldRaw = FileAttachment("data/annual-gold-prices.csv").csv({typed: true});
const goldData = (await goldRaw).map(d => ({
  year: +d.Date,
  price: +d.Price
})).filter(d => d.price > 0);

const latestPrice = goldData[goldData.length - 1].price;
const allTimeHigh = d3.max(goldData, d => d.price);
const startPrice = goldData[0].price;
const totalYears = goldData.length;
const totalGrowth = ((latestPrice - startPrice) / startPrice * 100).toLocaleString(undefined, {maximumFractionDigits: 0});
```

<div class="hero-large">
  <h1>The Golden Century</h1>
  <p>Explore 191 years of historical gold price data, analyzing trends from the Gold Standard era to modern economic volatility.</p>
</div>

<div class="key-stats">
  <div class="key-stat">
    <div class="value">$${latestPrice.toLocaleString('en-US', {minimumFractionDigits: 0, maximumFractionDigits: 0})}</div>
    <div class="label">Current Price (USD)</div>
  </div>
  <div class="key-stat">
    <div class="value">${totalYears}</div>
    <div class="label">Years of History</div>
  </div>
  <div class="key-stat">
    <div class="value">$${allTimeHigh.toLocaleString('en-US', {minimumFractionDigits: 0, maximumFractionDigits: 0})}</div>
    <div class="label">All-Time High</div>
  </div>
  <div class="key-stat">
    <div class="value">+${totalGrowth}%</div>
    <div class="label">Total Growth</div>
  </div>
</div>

<div class="landing-grid">
  <div class="landing-card">
    <a href="./gold-prices">
      <h3>Gold Prices Dashboard 🪙</h3>
      <p>Interactive visualization of historical gold prices versus major economic events. Analyze annual returns and era-specific performance.</p>
    </a>
  </div>
</div>

---

<div class="dashboard-footer">
  <span>Built with <a href="https://www.portaljs.com/" target="_blank">PortalJS</a> and Observable Framework</span>
  <span>Data Source: <a href="https://datahub.io/core/gold-prices" target="_blank">DataHub</a></span>
</div>
