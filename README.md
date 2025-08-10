<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Forex AI Chart Pro - TradingView Style, Multi-Year</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="https://unpkg.com/lightweight-charts@4.1.0/dist/lightweight-charts.standalone.production.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <style>
    body { margin: 0; font-family: Arial, sans-serif; background: #131722; color: #d1d4dc; }
    #controls, #indicator-controls, #strategy-controls, #concept-controls {
      padding: 12px; background: #181c25; display: flex; flex-wrap: wrap; gap: 12px; align-items: center; }
    select, button, input[type="checkbox"] {
      padding: 7px; font-size: 16px; background: #222631;
      color: #eee; border: 1px solid #23283b; border-radius: 4px;
      transition: background 0.2s; }
    select:focus, button:focus { outline: 2px solid #546eef; }
    #chart-container {
      width: 100vw; max-width: 100%; height: 600px; background: #131722;
      display: flex; align-items: center; justify-content: center; margin: 0;
    }
    #chart { width: 96vw; max-width: 1400px; height: 100%; }
    .ai-box, .log-box { padding: 1em; background: #181c25; border-top: 1px solid #23283b; }
    .ai-box { color: #6ee7b7; font-size: 17px; }
    .log-box { color: #76a9fa; font-size: 14px; max-height: 200px; overflow-y: auto; }
    .section-title { font-weight: bold; color: #f8c74a; margin-top: 18px; }
    label { margin-right: 10px; }
    .checkbox-group { display: flex; flex-wrap: wrap; gap: 14px; }
    .signal-marker-buy { color: #10e57b; font-weight: bold; }
    .signal-marker-sell { color: #fa3a3a; font-weight: bold; }
    .theme-toggle { margin-left: auto; }
    .legend {
      margin: 0 0 10px 10px; font-size: 15px; background: #222631; color: #d1d4dc;
      padding: 6px 12px; border-radius: 7px; display: inline-block; letter-spacing: .5px;
    }
    @media (max-width: 900px) {
      #chart-container { height: 350px; }
      .ai-box, .log-box { font-size: 13px; }
      select, button, input[type="checkbox"] { font-size: 13px; }
      .legend { font-size: 12px; }
    }
  </style>
</head>
<body>

<div id="controls">
  <label>Symbol:
    <select id="symbol">
      <option value="XAU/USD">XAU/USD</option>
      <option value="EUR/USD">EUR/USD</option>
      <option value="GBP/USD">GBP/USD</option>
      <option value="USD/JPY">USD/JPY</option>
      <option value="BTC/USD">BTC/USD</option>
    </select>
  </label>
  <label>Timeframe:
    <select id="interval">
      <option value="1min">1m</option>
      <option value="5min">5m</option>
      <option value="15min">15m</option>
      <option value="1h">1h</option>
      <option value="4h">4h</option>
      <option value="1day">1d</option>
      <option value="1week">1w</option>
      <option value="1month">1mo</option>
    </select>
  </label>
  <label>Chart Range:
    <select id="range">
      <option value="1y">1 Year</option>
      <option value="2y">2 Years</option>
      <option value="5y">5 Years</option>
      <option value="10y">10 Years</option>
      <option value="max">Max</option>
    </select>
  </label>
  <button onclick="takeScreenshot()">📸 Screenshot</button>
  <button onclick="exportLog()">📤 Export Log</button>
  <button class="theme-toggle" onclick="toggleTheme()">🌗 Theme</button>
</div>

<div class="legend" id="legend"></div>

<div class="section-title">Indicators</div>
<div id="indicator-controls" class="checkbox-group"></div>

<div class="section-title">Strategies</div>
<div id="strategy-controls" class="checkbox-group"></div>

<div class="section-title">AI Concepts</div>
<div id="concept-controls" class="checkbox-group"></div>

<div id="chart-container"><div id="chart"></div></div>
<div class="ai-box" id="ai-signal">⏳ AI signal loading...</div>
<div class="log-box" id="log">📜 Trade History:<br /></div>
<audio id="alert-sound" src="https://www.soundjay.com/buttons/beep-07.wav" preload="auto"></audio>

<script>
  // === API Keys ===
  const apiKeys = {
    twelveData: '8706afaf419746a2b4b1db7eac8d4411'
  };

  // === Indicator, Strategy & Concept List ===
  const indicatorsList = [
    "RSI", "MACD", "SMA", "EMA", "Bollinger Bands", "Stochastic", "Ichimoku", "VWAP",
    "ATR", "CCI", "Parabolic SAR", "Volume Profile", "Fibonacci", "OBV", "Pivot Points"
  ];
  const strategiesList = [
    "ICT/SMC", "Order Block", "Breaker Block", "Liquidity Grab", "BOS", "Fair Value Gap",
    "RSI+MACD Divergence", "Golden/Death Cross", "Breakout & Retest", "Supply/Demand Zone",
    "VWAP Strategy", "BB Reversal", "Stochastic OB/OS", "Pin Bar", "EMA Crossover",
    "OB+CHoCH", "Fibonacci+S&R", "Trendline Break", "EMA Bounce", "RSI 50", "Volume Spike"
  ];
  const conceptsList = [
    "Smart Money Concepts (SMC)", "Technical Analysis Concepts", "Fundamental Analysis Concepts",
    "Market Structure Concepts", "Risk Management Concepts", "Order Flow and Market Microstructure Concepts",
    "Sentiment Analysis Concepts", "Volume Spread Analysis (VSA)", "Elliott Wave Theory (EWT)",
    "Fibonacci Retracement and Extensions", "Wyckoff Method", "Market Profile",
    "Algorithmic and Quantitative Trading Concepts", "Sentiment and Behavioral Finance Concepts",
    "Market Cycles and Phases", "Order Types and Execution"
  ];

  // === UI Controls ===
  function populateCheckboxes(list, containerId) {
    const container = document.getElementById(containerId);
    container.innerHTML = '';
    list.forEach(name => {
      const id = name.replace(/[^a-zA-Z0-9]/g, '') + '-cb';
      const label = document.createElement('label');
      label.innerHTML = `<input type="checkbox" id="${id}" checked /> ${name}`;
      container.appendChild(label);
    });
  }
  populateCheckboxes(indicatorsList, 'indicator-controls');
  populateCheckboxes(strategiesList, 'strategy-controls');
  populateCheckboxes(conceptsList, 'concept-controls');

  // === TradingView-Style Chart Setup ===
  let theme = 'dark';
  let chart, candleSeries, overlaySeries = {};
  let chartIsReady = false;
  function chartBg()  { return theme === 'dark' ? '#131722' : '#fff'; }
  function chartTxt() { return theme === 'dark' ? '#d1d4dc' : '#23283b'; }
  function chartGrid() { return theme === 'dark' ? '#23283b' : '#eee'; }
  function chartUp()   { return theme === 'dark' ? '#26a69a' : '#0f9d58'; }
  function chartDown() { return theme === 'dark' ? '#ef5350' : '#d32f2f'; }

  function initChart() {
    if (chart) chart.remove();
    chart = LightweightCharts.createChart(document.getElementById('chart'), {
      layout: { background: { color: chartBg() }, textColor: chartTxt(), fontFamily: 'Trebuchet MS,Arial,sans-serif' },
      grid: { vertLines: { color: chartGrid() }, horzLines: { color: chartGrid() } },
      width: document.getElementById('chart-container').offsetWidth,
      height: document.getElementById('chart-container').offsetHeight,
      handleScroll: true,
      handleScale: true,
      crosshair: { mode: 1 }
    });
    // Candlestick with TradingView colors
    candleSeries = chart.addCandlestickSeries({
      upColor: chartUp(),
      downColor: chartDown(),
      borderUpColor: chartUp(),
      borderDownColor: chartDown(),
      wickUpColor: chartUp(),
      wickDownColor: chartDown()
    });
    overlaySeries = {};
    chart.timeScale().fitContent();
    chartIsReady = true;
  }
  initChart();

  function toggleTheme() {
    theme = (theme === 'dark') ? 'light' : 'dark';
    document.body.style.background = chartBg();
    document.body.style.color = chartTxt();
    if (chart) {
      chart.applyOptions({
        layout: { background: { color: chartBg() }, textColor: chartTxt() },
        grid: { vertLines: { color: chartGrid() }, horzLines: { color: chartGrid() } }
      });
      candleSeries.applyOptions({
        upColor: chartUp(),
        downColor: chartDown(),
        borderUpColor: chartUp(),
        borderDownColor: chartDown(),
        wickUpColor: chartUp(),
        wickDownColor: chartDown()
      });
    }
    document.getElementById('legend').style.background = theme === 'dark' ? "#222631" : "#eee";
    document.getElementById('legend').style.color = theme === 'dark' ? "#d1d4dc" : "#23283b";
  }

  window.addEventListener('resize', ()=>{
    if (!chart) return;
    chart.resize(document.getElementById('chart-container').offsetWidth, document.getElementById('chart-container').offsetHeight);
  });

  // === Data Fetcher, with Multi-Year Support ===
  async function fetchData(symbol, interval, range) {
    // Try to fetch as much data as possible, given API limitations
    // For 1d, 1w, 1mo, will use max outputsize (5000) and truncate locally
    let apiInterval = interval;
    let url = `https://api.twelvedata.com/time_series?symbol=${symbol}&interval=${apiInterval}&outputsize=5000&apikey=${apiKeys.twelveData}`;
    let res = await fetch(url);
    let data = await res.json();
    if (!data.values) {
      alert("❌ Data error: " + (data.message || "Unknown error"));
      return null;
    }
    let bars = data.values.reverse().map(d => ({
      time: new Date(d.datetime).getTime() / 1000,
      open: parseFloat(d.open),
      high: parseFloat(d.high),
      low: parseFloat(d.low),
      close: parseFloat(d.close),
      volume: parseFloat(d.volume || 0)
    }));
    // Filter for range (years) if possible
    if (interval === "1day" || interval === "1week" || interval === "1month") {
      let now = Date.now();
      let years = (range === "1y") ? 1 : (range === "2y" ? 2 : (range === "5y" ? 5 : (range === "10y" ? 10 : 100)));
      let cutoff = now - years * 365.25 * 24 * 3600 * 1000;
      bars = bars.filter(b => b.time * 1000 >= cutoff);
    }
    return bars;
  }

  // === Indicator Calculators (condensed) ===
  function calcSMA(bars, len=20) {
    return bars.map((bar,i,arr)=>{
      if(i<len-1) return null;
      let avg = arr.slice(i-len+1,i+1).reduce((a,b)=>a+b.close,0)/len;
      return { time: bar.time, value: avg };
    }).filter(Boolean);
  }
  function calcEMA(bars, len=20) {
    let ema = [], k = 2/(len+1);
    bars.forEach((bar,i)=>{ if(i===0) ema.push(bar.close); else ema.push(bar.close*k + ema[i-1]*(1-k)); });
    return bars.map((bar,i)=>({ time: bar.time, value: ema[i] })).slice(len-1);
  }
  function calcRSI(bars, len=14) {
    let rsi = [];
    for(let i=len; i<bars.length; i++){
      let up=0, down=0;
      for(let j=i-len+1; j<=i;j++){
        let diff = bars[j].close - bars[j-1].close;
        if(diff>0) up+=diff; else down-=diff;
      }
      let rs = up/(down||1);
      rsi.push({ time: bars[i].time, value: 100-(100/(1+rs)) });
    }
    return rsi;
  }
  function calcMACD(bars, fast=12, slow=26, signal=9){
    let emaFast = calcEMA(bars, fast).map(e=>e.value);
    let emaSlow = calcEMA(bars, slow).map(e=>e.value);
    let macd = emaFast.map((v,i)=>v-(emaSlow[i]||0));
    let signalLine = [];
    let k = 2/(signal+1);
    macd.forEach((v,i)=>{ if(i===0) signalLine.push(v); else signalLine.push(v*k + signalLine[i-1]*(1-k)); });
    let hist = macd.map((v,i)=>v-(signalLine[i]||0));
    return macd.map((v,i)=>({
      time: bars[i+slow-1]?.time || bars[i]?.time,
      macd: v,
      signal: signalLine[i],
      hist: hist[i]
    })).filter(Boolean);
  }
  function calcBB(bars, len=20, mult=2){
    return bars.map((bar,i,arr)=>{
      if(i<len-1) return null;
      let closes = arr.slice(i-len+1,i+1).map(b=>b.close);
      let avg = closes.reduce((a,b)=>a+b,0)/len;
      let std = Math.sqrt(closes.map(x=>(x-avg)**2).reduce((a,b)=>a+b,0)/len);
      return { time: bar.time, upper: avg+mult*std, lower: avg-mult*std, middle: avg };
    }).filter(Boolean);
  }

  // === AI Logic: Concepts + Strategies Integration (condensed) ===
  function aiSignalEnsemble({ bars, overlays, enabledConcepts, enabledStrategies }) {
    let signals = [];
    function last(a) { return a[a.length-1]; }
    // SMC + ICT/OrderBlock
    if (enabledConcepts['SmartMoneyConceptsSMC'] && enabledStrategies['ICTSMC']) {
      if (bars.length >= 3) {
        let b1 = bars[bars.length-3], b2 = bars[bars.length-2], b3 = bars[bars.length-1];
        if (b1.low > b2.low && b2.low < b3.low && b2.close > b2.open) {
          signals.push({ time: b3.time, type: "BUY", reason: "SMC: Liquidity Sweep + Bullish Order Block" });
        }
      }
    }
    // Technical Analysis Example: RSI + MACD
    if (enabledConcepts['TechnicalAnalysisConcepts']) {
      let rsi = overlays['RSI']?.slice(-1)[0]?.value;
      let macd = overlays['MACD']?.slice(-1)[0]?.macd;
      if (rsi < 30 && macd > 0)
        signals.push({ time: bars[bars.length-1].time, type: "BUY", reason: "TA: RSI<30 + MACD Bullish" });
      if (rsi > 70 && macd < 0)
        signals.push({ time: bars[bars.length-1].time, type: "SELL", reason: "TA: RSI>70 + MACD Bearish" });
    }
    // Risk Management: Only allow signal if ATR is not too high (pseudo)
    if (enabledConcepts['RiskManagementConcepts'] && overlays['ATR']) {
      let atr = overlays['ATR'].slice(-1)[0]?.value;
      if (atr > 2) {
        signals.push({ time: bars[bars.length-1].time, type: "NO TRADE", reason: "ATR too high - Risk Control" });
      }
    }
    // Sentiment Analysis (stub, as real sentiment needs API)
    if (enabledConcepts['SentimentAnalysisConcepts']) {
      let sentiment = "bullish";
      if (sentiment === "bullish")
        signals.push({ time: bars[bars.length-1].time, type: "BUY", reason: "Sentiment: Bullish" });
    }
    // Wyckoff/Market Structure (pseudo, for demo)
    if (enabledConcepts['WyckoffMethod']) {
      if (bars.length > 5) {
        let lows = bars.slice(-5).map(b=>b.low);
        if (Math.min(...lows) === lows[0]) {
          signals.push({ time: bars[bars.length-1].time, type: "BUY", reason: "Wyckoff: Spring Phase" });
        }
      }
    }
    return signals;
  }

  // === Overlay Renderer ===
  function renderOverlays(bars, overlays, indEnabled) {
    // Remove previous overlays
    Object.values(overlaySeries).forEach(s=>{ if(s && s.setData) s.setData([]); });
    // Re-render overlays
    Object.keys(overlays).forEach(k=>{
      if (indEnabled[k]) {
        if (!overlaySeries[k]) {
          let color = k.includes('EMA') ? '#f8c74a' : k.includes('SMA') ? '#6ee7b7' : k.includes('RSI') ? '#5686f5' :
            k.includes('MACD') ? '#c084fc' : k.includes('BB') ? '#76a9fa' : '#aaa';
          overlaySeries[k] = chart.addLineSeries({ color, lineWidth: 2 });
        }
        overlaySeries[k].setData(overlays[k]);
      }
    });
    // Bollinger Bands overlay
    if(overlays['BB'] && indEnabled['BollingerBands']){
      if(!overlaySeries['BB_upper']) overlaySeries['BB_upper'] = chart.addLineSeries({ color: '#76a9fa', lineWidth: 1 });
      if(!overlaySeries['BB_lower']) overlaySeries['BB_lower'] = chart.addLineSeries({ color: '#76a9fa', lineWidth: 1 });
      overlaySeries['BB_upper'].setData(overlays['BB'].map(b=>({time:b.time,value:b.upper})));
      overlaySeries['BB_lower'].setData(overlays['BB'].map(b=>({time:b.time,value:b.lower})));
    }
  }

  // === Main Update ===
  async function updateChart() {
    let symbol = document.getElementById('symbol').value;
    let interval = document.getElementById('interval').value;
    let range = document.getElementById('range').value;
    if (!chartIsReady) return;
    let bars = await fetchData(symbol, interval, range);
    if (!bars) return;
    candleSeries.setData(bars);

    // Collect enabled indicators
    let indEnabled = {};
    indicatorsList.forEach(name => {
      let id = name.replace(/[^a-zA-Z0-9]/g, '') + '-cb';
      indEnabled[name.replace(/[^a-zA-Z0-9]/g, '')] = document.getElementById(id)?.checked;
    });

    // Calculate overlays
    let overlays = {};
    if (indEnabled['SMA']) overlays['SMA'] = calcSMA(bars, 20);
    if (indEnabled['EMA']) overlays['EMA'] = calcEMA(bars, 20);
    if (indEnabled['RSI']) overlays['RSI'] = calcRSI(bars, 14);
    if (indEnabled['MACD']) overlays['MACD'] = calcMACD(bars, 12, 26, 9);
    if (indEnabled['BollingerBands']) overlays['BB'] = calcBB(bars, 20, 2);

    // TODO: More overlays for other indicators

    renderOverlays(bars, overlays, indEnabled);

    // Collect enabled strategies
    let stratEnabled = {};
    strategiesList.forEach(name => {
      let id = name.replace(/[^a-zA-Z0-9]/g, '') + '-cb';
      stratEnabled[name.replace(/[^a-zA-Z0-9]/g, '')] = document.getElementById(id)?.checked;
    });

    // Collect enabled concepts
    let conceptEnabled = {};
    conceptsList.forEach(name => {
      let id = name.replace(/[^a-zA-Z0-9]/g, '') + '-cb';
      conceptEnabled[name.replace(/[^a-zA-Z0-9]/g, '')] = document.getElementById(id)?.checked;
    });

    // === AI Ensemble Signal (Concepts + Strategies) ===
    let signals = aiSignalEnsemble({
      bars, overlays,
      enabledConcepts: conceptEnabled,
      enabledStrategies: stratEnabled
    });

    // Render signals
    let markers = [];
    signals.forEach(sig => {
      markers.push({
        time: sig.time,
        position: sig.type === "BUY" ? 'belowBar' : (sig.type === "SELL" ? 'aboveBar' : 'inBar'),
        color: sig.type === "BUY" ? '#10e57b' : (sig.type === "SELL" ? '#fa3a3a' : '#f8c74a'),
        shape: sig.type === "BUY" ? 'arrowUp' : (sig.type === "SELL" ? 'arrowDown' : 'circle'),
        text: sig.type
      });
      logSignal(symbol, interval, sig.type, bars[bars.length-1].close, sig.reason);
    });
    candleSeries.setMarkers(markers);

    document.getElementById("ai-signal").innerHTML =
      signals.length ? signals.map(s=>`<span class="signal-marker-${s.type.toLowerCase()}">[${s.type}]</span> ${s.reason}`).join('<br>') : "🔍 No clear signal.";

    // Update legend
    let l = [];
    if (indEnabled['SMA']) l.push('SMA');
    if (indEnabled['EMA']) l.push('EMA');
    if (indEnabled['RSI']) l.push('RSI');
    if (indEnabled['MACD']) l.push('MACD');
    if (indEnabled['BollingerBands']) l.push('BB');
    document.getElementById("legend").textContent = l.length ? 'Active: ' + l.join(', ') : 'No overlays';

    if (signals.length) document.getElementById("alert-sound").play();
  }

  function logSignal(sym, tf, dir, price, reason) {
    const now = new Date().toLocaleString();
    const line = `<div>📌 ${now} | ${sym} | ${tf} | <b>${dir}</b> @ ${price.toFixed(2)} <span style="color:#f8c74a">${reason||""}</span></div>`;
    document.getElementById("log").innerHTML += line;
  }

  // === Screenshot/Export ===
  function takeScreenshot() {
    html2canvas(document.getElementById("chart")).then(canvas => {
      const link = document.createElement("a");
      link.download = "chart.png";
      link.href = canvas.toDataURL();
      link.click();
    });
  }
  function exportLog() {
    let log = document.getElementById('log').innerText;
    let blob = new Blob([log], { type: "text/plain" });
    let link = document.createElement("a");
    link.download = "trade_log.txt";
    link.href = URL.createObjectURL(blob);
    link.click();
  }

  // === Startup ===
  document.getElementById("symbol").addEventListener("change", updateChart);
  document.getElementById("interval").addEventListener("change", updateChart);
  document.getElementById("range").addEventListener("change", updateChart);
  document.getElementById("indicator-controls").addEventListener("change", updateChart);
  document.getElementById("strategy-controls").addEventListener("change", updateChart);
  document.getElementById("concept-controls").addEventListener("change", updateChart);

  updateChart();
  setInterval(updateChart, 90000);

  window.addEventListener('resize', () => {
    if (!chart) return;
    chart.resize(document.getElementById('chart-container').offsetWidth, document.getElementById('chart-container').offsetHeight);
  });
</script>
</body>
</html>
