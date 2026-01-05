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
// Load gold price data
const goldRaw = FileAttachment("data/annual-gold-prices.csv").csv({typed: true});
const goldData = (await goldRaw).map(d => ({
  year: +d.Date,
  price: +d.Price
})).filter(d => d.price > 0);

// Calculate key metrics
const latestPrice = goldData[goldData.length - 1].price;
const firstPrice = goldData[0].price;
const allTimeHigh = d3.max(goldData, d => d.price);
const allTimeHighYear = goldData.find(d => d.price === allTimeHigh).year;
const totalIncrease = ((latestPrice - firstPrice) / firstPrice * 100).toFixed(0);

// Find the gold standard era (fixed price period)
const goldStandardEnd = 1971; // Nixon ended gold standard
const priceIn1970 = goldData.find(d => d.year === 1970)?.price || 35;

// Categorize eras
const goldStandardData = goldData.filter(d => d.year <= 1970);
const modernEra = goldData.filter(d => d.year > 1970);
```

```js
const years = d3.range(1833, 2025);
const selectedYear = view(Scrubber(years, {
  delay: 150,
  loop: false,
  initial: years.length - 1,
  autoplay: false,
  format: d => d
}));

const selectedData = goldData.filter(d => d.year <= selectedYear);
const currentPrice = selectedData[selectedData.length - 1];
```


<div class="dashboard-container">
  <div class="dashboard-top">
    <div class="sidebar-section">
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">Current Price</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value">$${latestPrice.toLocaleString('en-US', {minimumFractionDigits: 0, maximumFractionDigits: 0})}</div>
            <div class="stat-card-subvalue">USD/oz</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">Growth</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value" style="color: var(--color-accent-green)">+${parseInt(totalIncrease).toLocaleString()}%</div>
            <div class="stat-card-subvalue">Since 1833</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">All Time High</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value" style="color: var(--color-accent-orange)">$${allTimeHigh.toLocaleString('en-US', {minimumFractionDigits: 0, maximumFractionDigits: 0})}</div>
            <div class="stat-card-subvalue">In ${allTimeHighYear}</div>
          </div>
        </div>

        <div class="stat-card">
          <div class="stat-card-header">
            <span class="stat-card-label">Data Points</span>
          </div>
          <div class="stat-card-content">
            <div class="stat-card-value">191</div>
            <div class="stat-card-subvalue">Years</div>
          </div>
        </div>
      </div>

      <div class="insights-card">
        <div class="insights-header">
          <h4 class="insights-title">Key Insights</h4>
        </div>
        <ul class="insights-list">
          <li>For over a century (1833-1970), gold prices remained stable under the Gold Standard ($20-$35).</li>
          <li>The abandonment of the Gold Standard in 1971 triggered a massive era of price discovery and volatility.</li>
          <li>Financial crises (2008, 2020) act as catalysts, driving investors to gold as a safe haven asset.</li>
          <li>Recent highs reflect global economic uncertainty and inflation hedging dynamics.</li>
        </ul>
      </div>
    </div>

    <div class="chart-card">
      <div class="chart-header">
        <div class="chart-title-row">
          <h3 class="chart-title">191 Years of Gold Prices (1833-2024)</h3>
          <span class="chart-badge">Interactive Timeline</span>
        </div>
      </div>
      <div class="chart-content">
        ${resize((width) => goldTimeline({width}))}
      </div>
      <div style="padding-top: 1rem;">
        ${view(Scrubber(years, {
          delay: 150,
          loop: false,
          initial: years.length - 1,
          autoplay: false,
          format: d => d
        }))}
      </div>
    </div>
  </div>
</div>

```js
function goldTimeline({width} = {}) {
  const isMobile = width < 640;
  return Plot.plot({
    width,
    height: isMobile ? 500 : 600,
    marginLeft: isMobile ? 60 : 60,
    marginRight: isMobile ? 20 : 60,
    marginBottom: isMobile ? 60 : 40,
    marginTop: 30,
    style: {
      background: "transparent",
      fontFamily: "var(--font-family)",
      fontSize: "13px",
      overflow: "visible"
    },
    x: {
      label: "Year",
      grid: true,
      ticks: isMobile ? 5 : 10,
      tickFormat: "d"
    },
    y: {
      label: isMobile ? "Price (USD, log)" : "Price (USD, log scale)",
      grid: true,
      type: "log",
      ticks: [10, 20, 50, 100, 200, 500, 1000, 2000, 3000],
      labelAnchor: "top",
      tickFormat: d => d >= 1000 ? `$${d/1000}k` : `$${d}`
    },
    marks: [
      Plot.areaY(selectedData, {
        x: "year",
        y: "price",
        fill: "url(#goldGradient)",
        fillOpacity: 0.5,
        curve: "catmull-rom"
      }),
      Plot.line(selectedData, {
        x: "year",
        y: "price",
        stroke: "var(--color-accent-orange)",
        strokeWidth: 3,
        curve: "catmull-rom",
        tip: true,
        title: d => `${d.year}: $${d.price.toFixed(2)}`
      }),
      Plot.dot([currentPrice], {
        x: "year",
        y: "price",
        r: 6,
        fill: "var(--color-accent-orange)",
        stroke: "#ffffff",
        strokeWidth: 2
      }),
      Plot.text([currentPrice], {
        x: "year",
        y: "price",
        text: d => isMobile ? `$${d.price.toFixed(0)}` : `${d.year}: $${d.price.toFixed(2)}`,
        dx: isMobile ? -30 : -15,
        dy: -15,
        fill: "var(--color-text-primary)",
        fontSize: isMobile ? 11 : 13,
        fontWeight: 600,
        textAnchor: "end"
      }),
      Plot.ruleX([1971], {
        stroke: "var(--color-accent-red)",
        strokeWidth: 2,
        strokeDasharray: "4,4",
        opacity: 0.8
      }),
      Plot.text([{year: 1971, price: 1800}], {
        x: "year",
        y: "price",
        text: isMobile ? ["1971: Nixon"] : ["1971: Nixon Ends Gold Standard"],
        fill: "var(--color-accent-red)",
        fontSize: isMobile ? 10 : 12,
        fontWeight: 500,
        dx: isMobile ? -5 : -5,
        textAnchor: "end"
      }),
      Plot.ruleY([35], {
        stroke: "var(--color-accent-green)",
        strokeWidth: 2,
        strokeDasharray: "4,4",
        opacity: 0.6
      }),
      ...(isMobile ? [] : [Plot.text([{year: 1900, price: 35}], {
        x: "year",
        y: "price",
        text: ["Gold Standard Era ($35) →"],
        fill: "var(--color-accent-green)",
        fontSize: 12,
        dy: -8,
        textAnchor: "middle"
      })])
    ]
  });
}
```

```js
function Scrubber(values, {format = value => value, initial = 0, direction = 1, delay = null, autoplay = true, loop = true, loopDelay = null, alternate = false} = {}) {
  values = Array.from(values);
  const form = html`<form style="font: 12px var(--font-family); display: flex; height: 33px; align-items: center;"><button name=b type=button style="margin-right: 0.4em; width: 5em; background:var(--color-background-card); border:1px solid var(--color-border); border-radius:6px; cursor:pointer"></button><label style="display: flex; align-items: center;"><input name=i type=range min=0 max=${values.length - 1} value=${initial} step=1 style="width: 180px; accent-color: var(--color-accent-blue)"><output name=o style="margin-left: 0.4em; color:var(--color-text-primary)"></output></label></form>`;
  let frame = null, timer = null, interval = null;
  const start = () => { form.b.textContent = "Pause"; if (delay === null) frame = requestAnimationFrame(tick); else interval = setInterval(tick, delay); };
  const stop = () => { form.b.textContent = "Play"; if (frame !== null) cancelAnimationFrame(frame), frame = null; if (timer !== null) clearTimeout(timer), timer = null; if (interval !== null) clearInterval(interval), interval = null; };
  const running = () => frame !== null || timer !== null || interval !== null;
  const step = () => { form.i.valueAsNumber = (form.i.valueAsNumber + direction + values.length) % values.length; form.i.dispatchEvent(new CustomEvent("input", {bubbles: true})); };
  const tick = () => { if (form.i.valueAsNumber === (direction > 0 ? values.length - 1 : direction < 0 ? 0 : NaN)) { if (!loop) return stop(); if (alternate) direction = -direction; if (loopDelay !== null) { if (frame !== null) cancelAnimationFrame(frame), frame = null; if (interval !== null) clearInterval(interval), interval = null; timer = setTimeout(() => (step(), start()), loopDelay); return; } } if (delay === null) frame = requestAnimationFrame(tick); step(); };
  form.i.oninput = event => { if (event && event.isTrusted && running()) stop(); form.value = values[form.i.valueAsNumber]; form.o.value = format(form.value, form.i.valueAsNumber, values); };
  form.b.onclick = () => { if (running()) return stop(); direction = alternate && form.i.valueAsNumber === values.length - 1 ? -1 : 1; form.i.valueAsNumber = (form.i.valueAsNumber + direction) % values.length; form.i.dispatchEvent(new CustomEvent("input", {bubbles: true})); start(); };
  form.i.oninput();
  if (autoplay) start(); else stop();
  Inputs.disposal(form).then(stop);
  return form;
}
```

<svg width="0" height="0">
  <defs>
    <linearGradient id="goldGradient" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%" style="stop-color:var(--color-accent-orange);stop-opacity:0.9" />
      <stop offset="80%" style="stop-color:var(--color-accent-orange);stop-opacity:0.1" />
      <stop offset="100%" style="stop-color:var(--color-background-card);stop-opacity:0" />
    </linearGradient>
  </defs>
</svg>
