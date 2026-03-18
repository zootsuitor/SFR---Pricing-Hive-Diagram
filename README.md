<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fence Franchise Price Map</title>
<style>
*{box-sizing:border-box;margin:0;padding:0}
:root{
  --bg:#0d1117;--surface:#161b22;--surface2:#21262d;--border:#30363d;
  --amber:#f59e0b;--amber2:#fbbf24;--amber-dim:#78350f;
  --blue:#58a6ff;--blue-dim:#1f3d6e;
  --text:#e6edf3;--muted:#8b949e;--muted2:#484f58;
  --green:#3fb950;--red:#f85149;
  --r:#f59e0b;--g:#58a6ff;--a:#22c55e;
}
body{background:var(--bg);color:var(--text);font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;min-height:100vh}
#app{max-width:1400px;margin:0 auto;padding:20px}

header{margin-bottom:20px}
header h1{font-size:1.4rem;font-weight:700;color:var(--text)}
header p{color:var(--muted);font-size:.82rem;margin-top:4px}

.legend-row{display:flex;gap:20px;margin-top:10px;align-items:center;flex-wrap:wrap}
.legend-item{display:flex;align-items:center;gap:6px;font-size:.78rem;color:var(--muted)}
.legend-dot{width:12px;height:12px;border-radius:50%;flex-shrink:0}
.legend-dot.my{background:var(--amber);box-shadow:0 0 6px var(--amber)}
.legend-dot.other{background:var(--blue);opacity:.7}
.legend-dot.near{background:#a78bfa;opacity:.8}
.legend-tag{background:var(--amber-dim);color:var(--amber);padding:2px 8px;border-radius:4px;font-size:.72rem;font-weight:600;margin-left:4px}

.tabs-row{display:flex;gap:0;margin-bottom:0;border-bottom:2px solid var(--border)}
.tab{padding:8px 14px;cursor:pointer;font-size:.78rem;font-weight:500;color:var(--muted);border-bottom:2px solid transparent;margin-bottom:-2px;transition:all .15s;white-space:nowrap;user-select:none}
.tab:hover{color:var(--text)}
.tab.active{color:var(--amber);border-bottom-color:var(--amber)}

.view-row{display:flex;gap:8px;margin:14px 0;flex-wrap:wrap}
.view-btn{padding:6px 14px;cursor:pointer;font-size:.78rem;font-weight:500;background:var(--surface2);border:1px solid var(--border);border-radius:6px;color:var(--muted);transition:all .15s;user-select:none}
.view-btn:hover{border-color:var(--muted);color:var(--text)}
.view-btn.active{background:var(--amber-dim);border-color:var(--amber);color:var(--amber)}
.view-btn.all.active{background:#1f3d6e;border-color:var(--blue);color:var(--blue)}

.stats-bar{display:flex;gap:16px;flex-wrap:wrap;margin:10px 0;padding:10px 14px;background:var(--surface);border-radius:8px;border:1px solid var(--border)}
.stat{font-size:.76rem}
.stat-label{color:var(--muted);margin-right:5px}
.stat-value{font-weight:600;color:var(--text)}
.stat-my{color:var(--amber);font-weight:700}

.chart-wrap{background:var(--surface);border-radius:10px;border:1px solid var(--border);padding:16px;overflow-x:auto;position:relative;margin-bottom:8px}
#chart-svg{display:block;min-width:600px}

.note{font-size:.72rem;color:var(--muted2);margin-top:6px;font-style:italic}

#tooltip{position:fixed;background:var(--surface2);border:1px solid var(--amber);border-radius:6px;padding:8px 12px;font-size:.78rem;pointer-events:none;z-index:100;display:none;box-shadow:0 4px 20px rgba(0,0,0,.5)}
#tooltip .tt-name{font-weight:700;color:var(--text);margin-bottom:3px}
#tooltip .tt-price{color:var(--amber)}
#tooltip .tt-dist{color:var(--muted);font-size:.72rem;margin-top:2px}

.no-data{text-align:center;padding:60px;color:var(--muted);font-size:.9rem}
</style>
</head>
<body>
<div id="app">
<header>
  <h1>Fence Franchise Price Map</h1>
  <p>Per-linear-foot pricing by franchise and fence type &nbsp;·&nbsp; Anomalous entries (&gt;$200) excluded &nbsp;·&nbsp; $0.00 entries excluded</p>
  <div class="legend-row">
    <div class="legend-item"><div class="legend-dot my"></div>Your locations (Raleigh, Greensboro, Atlanta)<span class="legend-tag">CAM</span></div>
    <div class="legend-item"><div class="legend-dot other"></div>Other franchises — hover for details</div>
    <div class="legend-item"><div class="legend-dot near"></div>15 nearest to selected location</div>
  </div>
</header>

<div class="tabs-row" id="fence-tabs"></div>
<div class="view-row" id="view-btns"></div>
<div class="stats-bar" id="stats-bar"></div>
<div class="chart-wrap"><svg id="chart-svg"></svg></div>
<div class="note" id="chart-note"></div>
</div>

<div id="tooltip"><div class="tt-name" id="tt-name"></div><div class="tt-price" id="tt-price"></div><div class="tt-dist" id="tt-dist"></div></div>

<script>
const RAW = [{"f":"Akron","t":"4'H Black 300 Sterling (R)","p":42.79},{"f":"Akron","t":"5'H Black 300 Sterling (R)","p":41.17},{"f":"Akron","t":"54H Black 300 Sterling","p":39.35},{"f":"Akron","t":"6'H Cap and Trim Board on Board","p":55.62},{"f":"Akron","t":"6'H Residential Galvanized Chain-Link","p":25.27},{"f":"Akron","t":"6'H Stockade","p":30.11},{"f":"Akron","t":"6'H White Hamilton","p":1.08},{"f":"Albany","t":"4'H Black 300 Sterling (R)","p":39.0},{"f":"Albany","t":"5'H Black 300 Sterling (R)","p":55.9},{"f":"Albany","t":"54H Black 300 Sterling","p":54.08},{"f":"Albany","t":"6'H Cap and Trim Board on Board","p":73.84},{"f":"Albany","t":"6'H Residential Galvanized Chain-Link","p":33.28},{"f":"Albany","t":"6'H Stockade","p":46.28},{"f":"Albany","t":"6'H White Hamilton","p":53.3},{"f":"Arkansas","t":"4'H Black 300 Sterling (R)","p":51.99},{"f":"Arkansas","t":"5'H Black 300 Sterling (R)","p":61.52},{"f":"Arkansas","t":"54H Black 300 Sterling","p":33.1},{"f":"Arkansas","t":"6'H Cap and Trim Board on Board","p":47.39},{"f":"Arkansas","t":"6'H Residential Galvanized Chain-Link","p":30.74},{"f":"Arkansas","t":"6'H Stockade","p":32.97},{"f":"Arkansas","t":"6'H White Hamilton","p":77.35},{"f":"Athens","t":"4'H Black 300 Sterling (R)","p":27.68},{"f":"Athens","t":"5'H Black 300 Sterling (R)","p":29.05},{"f":"Athens","t":"54H Black 300 Sterling","p":22.9},{"f":"Athens","t":"6'H Residential Galvanized Chain-Link","p":12.35},{"f":"Athens","t":"6'H Stockade","p":17.88},{"f":"Athens","t":"6'H White Hamilton","p":35.7},{"f":"Atlanta","t":"4'H Black 300 Sterling (R)","p":34.44},{"f":"Atlanta","t":"5'H Black 300 Sterling (R)","p":38.19},{"f":"Atlanta","t":"54H Black 300 Sterling","p":34.99},{"f":"Atlanta","t":"6'H Cap and Trim Board on Board","p":26.0},{"f":"Atlanta","t":"6'H Residential Galvanized Chain-Link","p":24.6},{"f":"Atlanta","t":"6'H Stockade","p":21.68},{"f":"Atlanta","t":"6'H White Hamilton","p":31.2},{"f":"Augusta","t":"4'H Black 300 Sterling (R)","p":33.48},{"f":"Augusta","t":"5'H Black 300 Sterling (R)","p":46.33},{"f":"Augusta","t":"54H Black 300 Sterling","p":39.55},{"f":"Augusta","t":"6'H Cap and Trim Board on Board","p":43.53},{"f":"Augusta","t":"6'H Residential Galvanized Chain-Link","p":24.84},{"f":"Augusta","t":"6'H Stockade","p":22.4},{"f":"Augusta","t":"6'H White Hamilton","p":35.85},{"f":"Baltimore","t":"4'H Black 300 Sterling (R)","p":41.3},{"f":"Baltimore","t":"5'H Black 300 Sterling (R)","p":50.02},{"f":"Baltimore","t":"54H Black 300 Sterling","p":43.27},{"f":"Baltimore","t":"6'H Cap and Trim Board on Board","p":39.63},{"f":"Baltimore","t":"6'H Residential Galvanized Chain-Link","p":35.4},{"f":"Baltimore","t":"6'H Stockade","p":33.98},{"f":"Baltimore","t":"6'H White Hamilton","p":35.38},{"f":"Birmingham","t":"4'H Black 300 Sterling (R)","p":24.78},{"f":"Birmingham","t":"5'H Black 300 Sterling (R)","p":43.5},{"f":"Birmingham","t":"54H Black 300 Sterling","p":34.11},{"f":"Birmingham","t":"6'H Cap and Trim Board on Board","p":62.0},{"f":"Birmingham","t":"6'H Residential Galvanized Chain-Link","p":24.08},{"f":"Birmingham","t":"6'H Stockade","p":22.5},{"f":"Birmingham","t":"6'H White Hamilton","p":39.75},{"f":"Brevard County","t":"4'H Black 300 Sterling (R)","p":28.54},{"f":"Brevard County","t":"5'H Black 300 Sterling (R)","p":37.95},{"f":"Brevard County","t":"54H Black 300 Sterling","p":35.19},{"f":"Brevard County","t":"6'H Cap and Trim Board on Board","p":93.15},{"f":"Brevard County","t":"6'H Residential Galvanized Chain-Link","p":25.76},{"f":"Brevard County","t":"6'H Stockade","p":30.24},{"f":"Brevard County","t":"6'H White Hamilton","p":33.22},{"f":"Broward","t":"4'H Black 300 Sterling (R)","p":33.99},{"f":"Broward","t":"5'H Black 300 Sterling (R)","p":42.49},{"f":"Broward","t":"6'H Cap and Trim Board on Board","p":78.8},{"f":"Broward","t":"6'H Residential Galvanized Chain-Link","p":26.27},{"f":"Broward","t":"6'H Stockade","p":34.51},{"f":"Broward","t":"6'H White Hamilton","p":39.14},{"f":"Buffalo","t":"4'H Black 300 Sterling (R)","p":46.5},{"f":"Buffalo","t":"5'H Black 300 Sterling (R)","p":55.88},{"f":"Buffalo","t":"54H Black 300 Sterling","p":51.01},{"f":"Buffalo","t":"6'H Cap and Trim Board on Board","p":71.26},{"f":"Buffalo","t":"6'H Residential Galvanized Chain-Link","p":37.6},{"f":"Buffalo","t":"6'H Stockade","p":39.14},{"f":"Buffalo","t":"6'H White Hamilton","p":43.93},{"f":"Central & South Jersey","t":"4'H Black 300 Sterling (R)","p":39.16},{"f":"Central & South Jersey","t":"5'H Black 300 Sterling (R)","p":43.04},{"f":"Central & South Jersey","t":"54H Black 300 Sterling","p":31.46},{"f":"Central & South Jersey","t":"6'H Cap and Trim Board on Board","p":49.2},{"f":"Central & South Jersey","t":"6'H Residential Galvanized Chain-Link","p":24.26},{"f":"Central & South Jersey","t":"6'H Stockade","p":29.03},{"f":"Central & South Jersey","t":"6'H White Hamilton","p":31.52},{"f":"Central Iowa","t":"4'H Black 300 Sterling (R)","p":30.05},{"f":"Central Iowa","t":"5'H Black 300 Sterling (R)","p":46.96},{"f":"Central Iowa","t":"54H Black 300 Sterling","p":39.3},{"f":"Central Iowa","t":"6'H Cap and Trim Board on Board","p":51.53},{"f":"Central Iowa","t":"6'H Residential Galvanized Chain-Link","p":24.36},{"f":"Central Iowa","t":"6'H Stockade","p":27.38},{"f":"Central Iowa","t":"6'H White Hamilton","p":43.42},{"f":"Central Jersey","t":"4'H Black 300 Sterling (R)","p":47.55},{"f":"Central Jersey","t":"5'H Black 300 Sterling (R)","p":39.28},{"f":"Central Jersey","t":"54H Black 300 Sterling","p":36.84},{"f":"Central Jersey","t":"6'H Cap and Trim Board on Board","p":38.68},{"f":"Central Jersey","t":"6'H Residential Galvanized Chain-Link","p":22.52},{"f":"Central Jersey","t":"6'H Stockade","p":29.52},{"f":"Central Jersey","t":"6'H White Hamilton","p":31.66},{"f":"Central Oklahoma","t":"4'H Black 300 Sterling (R)","p":50.48},{"f":"Central Oklahoma","t":"5'H Black 300 Sterling (R)","p":49.51},{"f":"Central Oklahoma","t":"54H Black 300 Sterling","p":44.66},{"f":"Central Oklahoma","t":"6'H Cap and Trim Board on Board","p":37.86},{"f":"Central Oklahoma","t":"6'H Residential Galvanized Chain-Link","p":30.1},{"f":"Central Oklahoma","t":"6'H Stockade","p":20.38},{"f":"Central Oklahoma","t":"6'H White Hamilton","p":49.51},{"f":"Central Texas","t":"4'H Black 300 Sterling (R)","p":23.77},{"f":"Central Texas","t":"5'H Black 300 Sterling (R)","p":27.22},{"f":"Central Texas","t":"54H Black 300 Sterling","p":24.84},{"f":"Central Texas","t":"6'H Cap and Trim Board on Board","p":39.91},{"f":"Central Texas","t":"6'H Residential Galvanized Chain-Link","p":23.35},{"f":"Central Texas","t":"6'H Stockade","p":29.43},{"f":"Central Texas","t":"6'H White Hamilton","p":55.78},{"f":"Charleston","t":"4'H Black 300 Sterling (R)","p":27.0},{"f":"Charleston","t":"5'H Black 300 Sterling (R)","p":41.33},{"f":"Charleston","t":"54H Black 300 Sterling","p":32.66},{"f":"Charleston","t":"6'H Cap and Trim Board on Board","p":48.03},{"f":"Charleston","t":"6'H Residential Galvanized Chain-Link","p":26.01},{"f":"Charleston","t":"6'H Stockade","p":25.65},{"f":"Charleston","t":"6'H White Hamilton","p":32.22},{"f":"Charlotte","t":"4'H Black 300 Sterling (R)","p":28.69},{"f":"Charlotte","t":"5'H Black 300 Sterling (R)","p":31.54},{"f":"Charlotte","t":"54H Black 300 Sterling","p":30.2},{"f":"Charlotte","t":"6'H Residential Galvanized Chain-Link","p":28.63},{"f":"Charlotte","t":"6'H Stockade","p":25.73},{"f":"Charlotte","t":"6'H White Hamilton","p":36.0},{"f":"Chattanooga","t":"4'H Black 300 Sterling (R)","p":30.68},{"f":"Chattanooga","t":"5'H Black 300 Sterling (R)","p":36.9},{"f":"Chattanooga","t":"54H Black 300 Sterling","p":36.9},{"f":"Chattanooga","t":"6'H Cap and Trim Board on Board","p":35.02},{"f":"Chattanooga","t":"6'H Residential Galvanized Chain-Link","p":23.6},{"f":"Chattanooga","t":"6'H Stockade","p":21.63},{"f":"Chattanooga","t":"6'H White Hamilton","p":33.99},{"f":"Chicagoland","t":"4'H Black 300 Sterling (R)","p":38.42},{"f":"Chicagoland","t":"5'H Black 300 Sterling (R)","p":37.8},{"f":"Chicagoland","t":"54H Black 300 Sterling","p":23.88},{"f":"Chicagoland","t":"6'H Cap and Trim Board on Board","p":36.75},{"f":"Chicagoland","t":"6'H Residential Galvanized Chain-Link","p":34.96},{"f":"Chicagoland","t":"6'H Stockade","p":32.01},{"f":"Chicagoland","t":"6'H White Hamilton","p":38.7},{"f":"Cincinnati","t":"4'H Black 300 Sterling (R)","p":39.14},{"f":"Cincinnati","t":"5'H Black 300 Sterling (R)","p":44.29},{"f":"Cincinnati","t":"54H Black 300 Sterling","p":23.42},{"f":"Cincinnati","t":"6'H Cap and Trim Board on Board","p":46.44},{"f":"Cincinnati","t":"6'H Residential Galvanized Chain-Link","p":35.02},{"f":"Cincinnati","t":"6'H Stockade","p":32.06},{"f":"Cincinnati","t":"6'H White Hamilton","p":34.73},{"f":"Cleveland","t":"4'H Black 300 Sterling (R)","p":39.14},{"f":"Cleveland","t":"5'H Black 300 Sterling (R)","p":44.29},{"f":"Cleveland","t":"54H Black 300 Sterling","p":47.15},{"f":"Cleveland","t":"6'H Cap and Trim Board on Board","p":46.44},{"f":"Cleveland","t":"6'H Residential Galvanized Chain-Link","p":35.03},{"f":"Cleveland","t":"6'H Stockade","p":32.06},{"f":"Cleveland","t":"6'H White Hamilton","p":34.73},{"f":"Colorado Springs","t":"4'H Black 300 Sterling (R)","p":38.52},{"f":"Colorado Springs","t":"5'H Black 300 Sterling (R)","p":40.99},{"f":"Colorado Springs","t":"54H Black 300 Sterling","p":25.86},{"f":"Colorado Springs","t":"6'H Residential Galvanized Chain-Link","p":23.07},{"f":"Columbia","t":"4'H Black 300 Sterling (R)","p":38.88},{"f":"Columbia","t":"5'H Black 300 Sterling (R)","p":59.0},{"f":"Columbia","t":"54H Black 300 Sterling","p":53.1},{"f":"Columbia","t":"6'H Cap and Trim Board on Board","p":44.81},{"f":"Columbia","t":"6'H Residential Galvanized Chain-Link","p":20.55},{"f":"Columbia","t":"6'H Stockade","p":22.4},{"f":"Columbia","t":"6'H White Hamilton","p":35.85},{"f":"Columbus","t":"4'H Black 300 Sterling (R)","p":39.52},{"f":"Columbus","t":"5'H Black 300 Sterling (R)","p":44.72},{"f":"Columbus","t":"54H Black 300 Sterling","p":23.65},{"f":"Columbus","t":"6'H Cap and Trim Board on Board","p":46.89},{"f":"Columbus","t":"6'H Residential Galvanized Chain-Link","p":35.37},{"f":"Columbus","t":"6'H Stockade","p":32.38},{"f":"Columbus","t":"6'H White Hamilton","p":35.07},{"f":"Connecticut","t":"4'H Black 300 Sterling (R)","p":37.44},{"f":"Connecticut","t":"5'H Black 300 Sterling (R)","p":53.04},{"f":"Connecticut","t":"54H Black 300 Sterling","p":51.22},{"f":"Connecticut","t":"6'H Cap and Trim Board on Board","p":69.94},{"f":"Connecticut","t":"6'H Residential Galvanized Chain-Link","p":31.98},{"f":"Connecticut","t":"6'H Stockade","p":44.2},{"f":"Connecticut","t":"6'H White Hamilton","p":50.96},{"f":"Delaware Valley","t":"4'H Black 300 Sterling (R)","p":39.04},{"f":"Delaware Valley","t":"5'H Black 300 Sterling (R)","p":50.82},{"f":"Delaware Valley","t":"54H Black 300 Sterling","p":43.02},{"f":"Delaware Valley","t":"6'H Cap and Trim Board on Board","p":44.38},{"f":"Delaware Valley","t":"6'H Residential Galvanized Chain-Link","p":25.2},{"f":"Delaware Valley","t":"6'H Stockade","p":32.4},{"f":"Delaware Valley","t":"6'H White Hamilton","p":36.62},{"f":"Delmarva","t":"4'H Black 300 Sterling (R)","p":38.3},{"f":"Delmarva","t":"5'H Black 300 Sterling (R)","p":49.85},{"f":"Delmarva","t":"54H Black 300 Sterling","p":42.2},{"f":"Delmarva","t":"6'H Cap and Trim Board on Board","p":43.54},{"f":"Delmarva","t":"6'H Residential Galvanized Chain-Link","p":32.86},{"f":"Delmarva","t":"6'H Stockade","p":32.62},{"f":"Delmarva","t":"6'H White Hamilton","p":35.71},{"f":"Detroit","t":"4'H Black 300 Sterling (R)","p":54.08},{"f":"Detroit","t":"5'H Black 300 Sterling (R)","p":55.12},{"f":"Detroit","t":"54H Black 300 Sterling","p":70.72},{"f":"Detroit","t":"6'H Cap and Trim Board on Board","p":34.66},{"f":"Detroit","t":"6'H Residential Galvanized Chain-Link","p":38.48},{"f":"Detroit","t":"6'H Stockade","p":37.44},{"f":"Detroit","t":"6'H White Hamilton","p":48.88},{"f":"East Bay","t":"6'H Residential Galvanized Chain-Link","p":25.79},{"f":"East Bay","t":"6'H White Hamilton","p":51.5},{"f":"East Texas","t":"4'H Black 300 Sterling (R)","p":23.09},{"f":"East Texas","t":"5'H Black 300 Sterling (R)","p":26.7},{"f":"East Texas","t":"54H Black 300 Sterling","p":24.37},{"f":"East Texas","t":"6'H Cap and Trim Board on Board","p":48.47},{"f":"East Texas","t":"6'H Residential Galvanized Chain-Link","p":23.76},{"f":"East Texas","t":"6'H Stockade","p":28.87},{"f":"East Texas","t":"6'H White Hamilton","p":54.48},{"f":"Eastern NC","t":"4'H Black 300 Sterling (R)","p":32.24},{"f":"Eastern NC","t":"5'H Black 300 Sterling (R)","p":55.62},{"f":"Eastern NC","t":"54H Black 300 Sterling","p":40.94},{"f":"Eastern NC","t":"6'H Cap and Trim Board on Board","p":61.16},{"f":"Eastern NC","t":"6'H Residential Galvanized Chain-Link","p":29.85},{"f":"Eastern NC","t":"6'H Stockade","p":27.02},{"f":"Eastern NC","t":"6'H White Hamilton","p":36.51},{"f":"Fairfax & Montgomery","t":"4'H Black 300 Sterling (R)","p":35.19},{"f":"Fairfax & Montgomery","t":"5'H Black 300 Sterling (R)","p":46.16},{"f":"Fairfax & Montgomery","t":"54H Black 300 Sterling","p":44.57},{"f":"Fairfax & Montgomery","t":"6'H Cap and Trim Board on Board","p":41.4},{"f":"Fairfax & Montgomery","t":"6'H Residential Galvanized Chain-Link","p":23.29},{"f":"Fairfax & Montgomery","t":"6'H Stockade","p":26.91},{"f":"Fairfax & Montgomery","t":"6'H White Hamilton","p":32.86},{"f":"Fayetteville","t":"4'H Black 300 Sterling (R)","p":32.76},{"f":"Fayetteville","t":"5'H Black 300 Sterling (R)","p":44.59},{"f":"Fayetteville","t":"54H Black 300 Sterling","p":40.94},{"f":"Fayetteville","t":"6'H Cap and Trim Board on Board","p":61.16},{"f":"Fayetteville","t":"6'H Residential Galvanized Chain-Link","p":29.85},{"f":"Fayetteville","t":"6'H Stockade","p":27.02},{"f":"Fayetteville","t":"6'H White Hamilton","p":36.51},{"f":"Fort Collins","t":"4'H Black 300 Sterling (R)","p":48.72},{"f":"Fort Collins","t":"5'H Black 300 Sterling (R)","p":60.15},{"f":"Fort Collins","t":"54H Black 300 Sterling","p":25.86},{"f":"Fort Collins","t":"6'H Residential Galvanized Chain-Link","p":23.52},{"f":"Gainesville","t":"4'H Black 300 Sterling (R)","p":34.03},{"f":"Gainesville","t":"5'H Black 300 Sterling (R)","p":37.09},{"f":"Gainesville","t":"54H Black 300 Sterling","p":37.63},{"f":"Gainesville","t":"6'H Cap and Trim Board on Board","p":40.66},{"f":"Gainesville","t":"6'H Residential Galvanized Chain-Link","p":24.46},{"f":"Gainesville","t":"6'H Stockade","p":24.22},{"f":"Gainesville","t":"6'H White Hamilton","p":31.93},{"f":"Grand Rapids","t":"4'H Black 300 Sterling (R)","p":36.05},{"f":"Grand Rapids","t":"5'H Black 300 Sterling (R)","p":53.11},{"f":"Grand Rapids","t":"54H Black 300 Sterling","p":40.68},{"f":"Grand Rapids","t":"6'H Cap and Trim Board on Board","p":56.98},{"f":"Grand Rapids","t":"6'H Residential Galvanized Chain-Link","p":27.04},{"f":"Grand Rapids","t":"6'H Stockade","p":26.48},{"f":"Grand Rapids","t":"6'H White Hamilton","p":42.12},{"f":"Great Lakes Bay Region","t":"4'H Black 300 Sterling (R)","p":36.05},{"f":"Great Lakes Bay Region","t":"5'H Black 300 Sterling (R)","p":48.41},{"f":"Great Lakes Bay Region","t":"54H Black 300 Sterling","p":37.08},{"f":"Great Lakes Bay Region","t":"6'H Cap and Trim Board on Board","p":54.85},{"f":"Great Lakes Bay Region","t":"6'H Residential Galvanized Chain-Link","p":27.04},{"f":"Great Lakes Bay Region","t":"6'H Stockade","p":25.49},{"f":"Great Lakes Bay Region","t":"6'H White Hamilton","p":54.08},{"f":"Greater Boston","t":"4'H Black 300 Sterling (R)","p":54.61},{"f":"Greater Boston","t":"5'H Black 300 Sterling (R)","p":52.17},{"f":"Greater Boston","t":"54H Black 300 Sterling","p":50.26},{"f":"Greater Boston","t":"6'H Residential Galvanized Chain-Link","p":41.91},{"f":"Greater Boston","t":"6'H Stockade","p":25.88},{"f":"Greater Boston","t":"6'H White Hamilton","p":41.51},{"f":"Greater Lexington","t":"4'H Black 300 Sterling (R)","p":32.97},{"f":"Greater Lexington","t":"5'H Black 300 Sterling (R)","p":43.09},{"f":"Greater Lexington","t":"54H Black 300 Sterling","p":39.34},{"f":"Greater Lexington","t":"6'H Cap and Trim Board on Board","p":64.74},{"f":"Greater Lexington","t":"6'H Residential Galvanized Chain-Link","p":32.04},{"f":"Greater Lexington","t":"6'H Stockade","p":28.86},{"f":"Greater Lexington","t":"6'H White Hamilton","p":38.67},{"f":"Greater Louisville","t":"4'H Black 300 Sterling (R)","p":32.49},{"f":"Greater Louisville","t":"5'H Black 300 Sterling (R)","p":40.69},{"f":"Greater Louisville","t":"54H Black 300 Sterling","p":36.43},{"f":"Greater Louisville","t":"6'H Cap and Trim Board on Board","p":52.97},{"f":"Greater Louisville","t":"6'H Residential Galvanized Chain-Link","p":26.84},{"f":"Greater Louisville","t":"6'H Stockade","p":26.78},{"f":"Greater Louisville","t":"6'H White Hamilton","p":33.66},{"f":"Greater Seattle","t":"6'H Residential Galvanized Chain-Link","p":31.5},{"f":"Greater Seattle","t":"6'H White Hamilton","p":62.0},{"f":"Greater Toledo","t":"4'H Black 300 Sterling (R)","p":50.47},{"f":"Greater Toledo","t":"5'H Black 300 Sterling (R)","p":55.42},{"f":"Greater Toledo","t":"54H Black 300 Sterling","p":23.42},{"f":"Greater Toledo","t":"6'H Cap and Trim Board on Board","p":58.2},{"f":"Greater Toledo","t":"6'H Residential Galvanized Chain-Link","p":27.81},{"f":"Greater Toledo","t":"6'H Stockade","p":34.51},{"f":"Greater Toledo","t":"6'H White Hamilton","p":45.32},{"f":"Greensboro","t":"4'H Black 300 Sterling (R)","p":33.79},{"f":"Greensboro","t":"5'H Black 300 Sterling (R)","p":44.69},{"f":"Greensboro","t":"54H Black 300 Sterling","p":38.27},{"f":"Greensboro","t":"6'H Cap and Trim Board on Board","p":35.23},{"f":"Greensboro","t":"6'H Residential Galvanized Chain-Link","p":25.53},{"f":"Greensboro","t":"6'H Stockade","p":29.32},{"f":"Greensboro","t":"6'H White Hamilton","p":38.9},{"f":"Greenville","t":"4'H Black 300 Sterling (R)","p":31.59},{"f":"Greenville","t":"5'H Black 300 Sterling (R)","p":35.38},{"f":"Greenville","t":"54H Black 300 Sterling","p":34.16},{"f":"Greenville","t":"6'H Cap and Trim Board on Board","p":33.66},{"f":"Greenville","t":"6'H Residential Galvanized Chain-Link","p":28.12},{"f":"Greenville","t":"6'H Stockade","p":20.79},{"f":"Greenville","t":"6'H White Hamilton","p":29.14},{"f":"Hartford","t":"4'H Black 300 Sterling (R)","p":37.44},{"f":"Hartford","t":"5'H Black 300 Sterling (R)","p":53.04},{"f":"Hartford","t":"54H Black 300 Sterling","p":51.22},{"f":"Hartford","t":"6'H Cap and Trim Board on Board","p":69.94},{"f":"Hartford","t":"6'H Residential Galvanized Chain-Link","p":31.98},{"f":"Hartford","t":"6'H Stockade","p":44.2},{"f":"Hartford","t":"6'H White Hamilton","p":50.96},{"f":"Hudson Valley","t":"4'H Black 300 Sterling (R)","p":52.29},{"f":"Hudson Valley","t":"5'H Black 300 Sterling (R)","p":62.08},{"f":"Hudson Valley","t":"54H Black 300 Sterling","p":61.58},{"f":"Hudson Valley","t":"6'H Cap and Trim Board on Board","p":54.79},{"f":"Hudson Valley","t":"6'H Residential Galvanized Chain-Link","p":36.09},{"f":"Hudson Valley","t":"6'H Stockade","p":39.29},{"f":"Hudson Valley","t":"6'H White Hamilton","p":47.11},{"f":"Huntsville","t":"4'H Black 300 Sterling (R)","p":31.93},{"f":"Huntsville","t":"5'H Black 300 Sterling (R)","p":35.02},{"f":"Huntsville","t":"54H Black 300 Sterling","p":23.42},{"f":"Huntsville","t":"6'H Cap and Trim Board on Board","p":38.76},{"f":"Huntsville","t":"6'H Residential Galvanized Chain-Link","p":19.57},{"f":"Huntsville","t":"6'H Stockade","p":21.93},{"f":"Huntsville","t":"6'H White Hamilton","p":38.63},{"f":"Indianapolis","t":"4'H Black 300 Sterling (R)","p":42.81},{"f":"Indianapolis","t":"5'H Black 300 Sterling (R)","p":46.76},{"f":"Indianapolis","t":"54H Black 300 Sterling","p":50.66},{"f":"Indianapolis","t":"6'H Cap and Trim Board on Board","p":53.77},{"f":"Indianapolis","t":"6'H Residential Galvanized Chain-Link","p":29.57},{"f":"Indianapolis","t":"6'H Stockade","p":29.02},{"f":"Indianapolis","t":"6'H White Hamilton","p":36.4},{"f":"Inland Empire","t":"6'H Residential Galvanized Chain-Link","p":27.81},{"f":"Inland Empire","t":"6'H Stockade","p":54.6},{"f":"Inland Empire","t":"6'H White Hamilton","p":57.68},{"f":"Jackson","t":"4'H Black 300 Sterling (R)","p":57.75},{"f":"Jackson","t":"5'H Black 300 Sterling (R)","p":44.1},{"f":"Jackson","t":"54H Black 300 Sterling","p":23.88},{"f":"Jackson","t":"6'H Cap and Trim Board on Board","p":25.2},{"f":"Jackson","t":"6'H Residential Galvanized Chain-Link","p":19.95},{"f":"Jackson","t":"6'H Stockade","p":18.9},{"f":"Jackson","t":"6'H White Hamilton","p":46.2},{"f":"Kansas City","t":"4'H Black 300 Sterling (R)","p":31.01},{"f":"Kansas City","t":"5'H Black 300 Sterling (R)","p":43.24},{"f":"Kansas City","t":"6'H Residential Galvanized Chain-Link","p":26.64},{"f":"Kansas City","t":"6'H Stockade","p":25.38},{"f":"Kansas City","t":"6'H White Hamilton","p":43.28},{"f":"Knoxville","t":"4'H Black 300 Sterling (R)","p":31.05},{"f":"Knoxville","t":"5'H Black 300 Sterling (R)","p":33.6},{"f":"Knoxville","t":"54H Black 300 Sterling","p":34.2},{"f":"Knoxville","t":"6'H Cap and Trim Board on Board","p":31.83},{"f":"Knoxville","t":"6'H Residential Galvanized Chain-Link","p":21.85},{"f":"Knoxville","t":"6'H Stockade","p":19.95},{"f":"Knoxville","t":"6'H White Hamilton","p":28.5},{"f":"Lake County","t":"4'H Black 300 Sterling (R)","p":29.23},{"f":"Lake County","t":"5'H Black 300 Sterling (R)","p":39.65},{"f":"Lake County","t":"54H Black 300 Sterling","p":38.58},{"f":"Lake County","t":"6'H Cap and Trim Board on Board","p":39.53},{"f":"Lake County","t":"6'H Residential Galvanized Chain-Link","p":20.64},{"f":"Lake County","t":"6'H Stockade","p":22.42},{"f":"Lake County","t":"6'H White Hamilton","p":34.89},{"f":"Lansing","t":"4'H Black 300 Sterling (R)","p":36.05},{"f":"Lansing","t":"5'H Black 300 Sterling (R)","p":52.64},{"f":"Lansing","t":"54H Black 300 Sterling","p":40.32},{"f":"Lansing","t":"6'H Cap and Trim Board on Board","p":56.45},{"f":"Lansing","t":"6'H Residential Galvanized Chain-Link","p":27.04},{"f":"Lansing","t":"6'H Stockade","p":26.24},{"f":"Lansing","t":"6'H White Hamilton","p":41.72},{"f":"Little Rock","t":"4'H Black 300 Sterling (R)","p":44.34},{"f":"Little Rock","t":"5'H Black 300 Sterling (R)","p":54.15},{"f":"Little Rock","t":"54H Black 300 Sterling","p":30.51},{"f":"Little Rock","t":"6'H Cap and Trim Board on Board","p":50.57},{"f":"Little Rock","t":"6'H Residential Galvanized Chain-Link","p":24.07},{"f":"Little Rock","t":"6'H Stockade","p":30.12},{"f":"Little Rock","t":"6'H White Hamilton","p":67.57},{"f":"Madison & Rockford","t":"4'H Black 300 Sterling (R)","p":49.3},{"f":"Madison & Rockford","t":"5'H Black 300 Sterling (R)","p":53.55},{"f":"Madison & Rockford","t":"54H Black 300 Sterling","p":23.88},{"f":"Madison & Rockford","t":"6'H Cap and Trim Board on Board","p":45.6},{"f":"Madison & Rockford","t":"6'H Residential Galvanized Chain-Link","p":33.21},{"f":"Madison & Rockford","t":"6'H Stockade","p":35.36},{"f":"Madison & Rockford","t":"6'H White Hamilton","p":45.76},{"f":"Memphis","t":"4'H Black 300 Sterling (R)","p":30.64},{"f":"Memphis","t":"5'H Black 300 Sterling (R)","p":35.54},{"f":"Memphis","t":"54H Black 300 Sterling","p":23.42},{"f":"Memphis","t":"6'H Cap and Trim Board on Board","p":48.41},{"f":"Memphis","t":"6'H Residential Galvanized Chain-Link","p":22.66},{"f":"Memphis","t":"6'H Stockade","p":23.69},{"f":"Memphis","t":"6'H White Hamilton","p":38.63},{"f":"Middle Georgia","t":"4'H Black 300 Sterling (R)","p":28.64},{"f":"Middle Georgia","t":"5'H Black 300 Sterling (R)","p":35.37},{"f":"Middle Georgia","t":"54H Black 300 Sterling","p":26.15},{"f":"Middle Georgia","t":"6'H Cap and Trim Board on Board","p":49.25},{"f":"Middle Georgia","t":"6'H Residential Galvanized Chain-Link","p":14.66},{"f":"Middle Georgia","t":"6'H Stockade","p":21.34},{"f":"Middle Georgia","t":"6'H White Hamilton","p":36.6},{"f":"Milwaukee","t":"4'H Black 300 Sterling (R)","p":42.85},{"f":"Milwaukee","t":"5'H Black 300 Sterling (R)","p":57.48},{"f":"Milwaukee","t":"6'H Residential Galvanized Chain-Link","p":32.4},{"f":"Milwaukee","t":"6'H Stockade","p":43.89},{"f":"Minneapolis","t":"4'H Black 300 Sterling (R)","p":37.15},{"f":"Minneapolis","t":"5'H Black 300 Sterling (R)","p":26.16},{"f":"Minneapolis","t":"54H Black 300 Sterling","p":23.88},{"f":"Minneapolis","t":"6'H Cap and Trim Board on Board","p":35.0},{"f":"Minneapolis","t":"6'H Residential Galvanized Chain-Link","p":41.2},{"f":"Minneapolis","t":"6'H Stockade","p":38.88},{"f":"Minneapolis","t":"6'H White Hamilton","p":44.53},{"f":"Myrtle Beach","t":"4'H Black 300 Sterling (R)","p":32.5},{"f":"Myrtle Beach","t":"5'H Black 300 Sterling (R)","p":43.5},{"f":"Myrtle Beach","t":"54H Black 300 Sterling","p":34.0},{"f":"Myrtle Beach","t":"6'H Cap and Trim Board on Board","p":46.77},{"f":"Myrtle Beach","t":"6'H Residential Galvanized Chain-Link","p":29.04},{"f":"Myrtle Beach","t":"6'H Stockade","p":26.5},{"f":"Myrtle Beach","t":"6'H White Hamilton","p":27.0},{"f":"Nashville","t":"4'H Black 300 Sterling (R)","p":31.86},{"f":"Nashville","t":"5'H Black 300 Sterling (R)","p":34.36},{"f":"Nashville","t":"54H Black 300 Sterling","p":37.35},{"f":"Nashville","t":"6'H Cap and Trim Board on Board","p":40.03},{"f":"Nashville","t":"6'H Residential Galvanized Chain-Link","p":24.66},{"f":"Nashville","t":"6'H Stockade","p":26.64},{"f":"Nashville","t":"6'H White Hamilton","p":34.59},{"f":"Nassau County","t":"4'H Black 300 Sterling (R)","p":45.24},{"f":"Nassau County","t":"5'H Black 300 Sterling (R)","p":55.12},{"f":"Nassau County","t":"54H Black 300 Sterling","p":62.4},{"f":"Nassau County","t":"6'H Cap and Trim Board on Board","p":48.36},{"f":"Nassau County","t":"6'H Residential Galvanized Chain-Link","p":33.8},{"f":"Nassau County","t":"6'H Stockade","p":67.6},{"f":"Nassau County","t":"6'H White Hamilton","p":34.32},{"f":"Norfolk","t":"4'H Black 300 Sterling (R)","p":34.91},{"f":"Norfolk","t":"5'H Black 300 Sterling (R)","p":59.25},{"f":"Norfolk","t":"54H Black 300 Sterling","p":46.22},{"f":"Norfolk","t":"6'H Cap and Trim Board on Board","p":59.25},{"f":"Norfolk","t":"6'H Residential Galvanized Chain-Link","p":28.18},{"f":"Norfolk","t":"6'H Stockade","p":28.44},{"f":"Norfolk","t":"6'H White Hamilton","p":37.79},{"f":"North Dallas","t":"4'H Black 300 Sterling (R)","p":51.95},{"f":"North Dallas","t":"5'H Black 300 Sterling (R)","p":50.95},{"f":"North Dallas","t":"54H Black 300 Sterling","p":45.96},{"f":"North Dallas","t":"6'H Cap and Trim Board on Board","p":38.97},{"f":"North Dallas","t":"6'H Residential Galvanized Chain-Link","p":34.98},{"f":"North Dallas","t":"6'H Stockade","p":20.98},{"f":"North Dallas","t":"6'H White Hamilton","p":50.95},{"f":"North Florida","t":"4'H Black 300 Sterling (R)","p":25.0},{"f":"North Florida","t":"5'H Black 300 Sterling (R)","p":37.49},{"f":"North Florida","t":"54H Black 300 Sterling","p":26.8},{"f":"North Florida","t":"6'H Cap and Trim Board on Board","p":42.5},{"f":"North Florida","t":"6'H Residential Galvanized Chain-Link","p":26.64},{"f":"North Florida","t":"6'H Stockade","p":23.5},{"f":"North Florida","t":"6'H White Hamilton","p":27.97},{"f":"North Houston","t":"4'H Black 300 Sterling (R)","p":37.6},{"f":"North Houston","t":"5'H Black 300 Sterling (R)","p":55.04},{"f":"North Houston","t":"6'H Cap and Trim Board on Board","p":36.57},{"f":"North Houston","t":"6'H Residential Galvanized Chain-Link","p":23.18},{"f":"North Houston","t":"6'H Stockade","p":23.69},{"f":"North Houston","t":"6'H White Hamilton","p":33.99},{"f":"North San Diego","t":"4'H Black 300 Sterling (R)","p":22.2},{"f":"North San Diego","t":"5'H Black 300 Sterling (R)","p":25.66},{"f":"North San Diego","t":"54H Black 300 Sterling","p":23.42},{"f":"North San Diego","t":"6'H Cap and Trim Board on Board","p":34.33},{"f":"North San Diego","t":"6'H Residential Galvanized Chain-Link","p":17.32},{"f":"North San Diego","t":"6'H White Hamilton","p":45.06},{"f":"North Shore","t":"4'H Black 300 Sterling (R)","p":39.72},{"f":"North Shore","t":"5'H Black 300 Sterling (R)","p":48.86},{"f":"North Shore","t":"54H Black 300 Sterling","p":44.81},{"f":"North Shore","t":"6'H Cap and Trim Board on Board","p":38.67},{"f":"North Shore","t":"6'H Residential Galvanized Chain-Link","p":20.59},{"f":"North Shore","t":"6'H Stockade","p":23.2},{"f":"North Shore","t":"6'H White Hamilton","p":33.55},{"f":"North Texas","t":"4'H Black 300 Sterling (R)","p":38.92},{"f":"North Texas","t":"5'H Black 300 Sterling (R)","p":30.59},{"f":"North Texas","t":"54H Black 300 Sterling","p":27.92},{"f":"North Texas","t":"6'H Cap and Trim Board on Board","p":36.57},{"f":"North Texas","t":"6'H Residential Galvanized Chain-Link","p":24.84},{"f":"North Texas","t":"6'H Stockade","p":19.57},{"f":"North Texas","t":"6'H White Hamilton","p":64.61},{"f":"Northeastern Los Angeles","t":"4'H Black 300 Sterling (R)","p":26.08},{"f":"Northeastern Los Angeles","t":"5'H Black 300 Sterling (R)","p":25.16},{"f":"Northeastern Los Angeles","t":"54H Black 300 Sterling","p":22.97},{"f":"Northeastern Los Angeles","t":"6'H Cap and Trim Board on Board","p":33.66},{"f":"Northeastern Los Angeles","t":"6'H Residential Galvanized Chain-Link","p":55.38},{"f":"Northeastern Los Angeles","t":"6'H Stockade","p":52.56},{"f":"Northeastern Los Angeles","t":"6'H White Hamilton","p":70.82},{"f":"Northern Kentucky","t":"4'H Black 300 Sterling (R)","p":39.9},{"f":"Northern Kentucky","t":"5'H Black 300 Sterling (R)","p":43.0},{"f":"Northern Kentucky","t":"54H Black 300 Sterling","p":32.12},{"f":"Northern Kentucky","t":"6'H Cap and Trim Board on Board","p":45.09},{"f":"Northern Kentucky","t":"6'H Residential Galvanized Chain-Link","p":35.71},{"f":"Northern Kentucky","t":"6'H Stockade","p":31.13},{"f":"Northern Kentucky","t":"6'H White Hamilton","p":33.72},{"f":"Northern NJ","t":"4'H Black 300 Sterling (R)","p":31.25},{"f":"Northern NJ","t":"5'H Black 300 Sterling (R)","p":36.29},{"f":"Northern NJ","t":"54H Black 300 Sterling","p":35.2},{"f":"Northern NJ","t":"6'H Cap and Trim Board on Board","p":41.62},{"f":"Northern NJ","t":"6'H Residential Galvanized Chain-Link","p":29.0},{"f":"Northern NJ","t":"6'H Stockade","p":36.7},{"f":"Northern NJ","t":"6'H White Hamilton","p":29.15},{"f":"Northern Virginia","t":"4'H Black 300 Sterling (R)","p":34.91},{"f":"Northern Virginia","t":"5'H Black 300 Sterling (R)","p":59.25},{"f":"Northern Virginia","t":"54H Black 300 Sterling","p":46.22},{"f":"Northern Virginia","t":"6'H Cap and Trim Board on Board","p":59.25},{"f":"Northern Virginia","t":"6'H Residential Galvanized Chain-Link","p":28.18},{"f":"Northern Virginia","t":"6'H Stockade","p":33.5},{"f":"Northern Virginia","t":"6'H White Hamilton","p":37.79},{"f":"NW Georgia","t":"4'H Black 300 Sterling (R)","p":42.3},{"f":"NW Georgia","t":"5'H Black 300 Sterling (R)","p":48.3},{"f":"NW Georgia","t":"54H Black 300 Sterling","p":25.32},{"f":"NW Georgia","t":"6'H Cap and Trim Board on Board","p":35.44},{"f":"NW Georgia","t":"6'H Residential Galvanized Chain-Link","p":20.89},{"f":"NW Georgia","t":"6'H Stockade","p":24.68},{"f":"NW Georgia","t":"6'H White Hamilton","p":45.15},{"f":"Omaha","t":"4'H Black 300 Sterling (R)","p":34.62},{"f":"Omaha","t":"5'H Black 300 Sterling (R)","p":37.37},{"f":"Omaha","t":"54H Black 300 Sterling","p":34.14},{"f":"Omaha","t":"6'H Cap and Trim Board on Board","p":60.18},{"f":"Omaha","t":"6'H Residential Galvanized Chain-Link","p":19.5},{"f":"Omaha","t":"6'H Stockade","p":29.26},{"f":"Omaha","t":"6'H White Hamilton","p":30.81},{"f":"Orange County CA","t":"4'H Black 300 Sterling (R)","p":26.4},{"f":"Orange County CA","t":"5'H Black 300 Sterling (R)","p":27.4},{"f":"Orange County CA","t":"54H Black 300 Sterling","p":25.01},{"f":"Orange County CA","t":"6'H Cap and Trim Board on Board","p":36.66},{"f":"Orange County CA","t":"6'H Residential Galvanized Chain-Link","p":39.45},{"f":"Orange County CA","t":"6'H Stockade","p":51.73},{"f":"Orange County CA","t":"6'H White Hamilton","p":52.71},{"f":"Oviedo","t":"4'H Black 300 Sterling (R)","p":25.0},{"f":"Oviedo","t":"5'H Black 300 Sterling (R)","p":28.0},{"f":"Oviedo","t":"54H Black 300 Sterling","p":27.0},{"f":"Oviedo","t":"6'H Cap and Trim Board on Board","p":44.0},{"f":"Oviedo","t":"6'H Residential Galvanized Chain-Link","p":18.0},{"f":"Oviedo","t":"6'H Stockade","p":23.0},{"f":"Oviedo","t":"6'H White Hamilton","p":24.48},{"f":"Owensboro","t":"4'H Black 300 Sterling (R)","p":22.37},{"f":"Owensboro","t":"5'H Black 300 Sterling (R)","p":21.6},{"f":"Owensboro","t":"54H Black 300 Sterling","p":24.56},{"f":"Owensboro","t":"6'H Cap and Trim Board on Board","p":43.54},{"f":"Owensboro","t":"6'H Residential Galvanized Chain-Link","p":20.06},{"f":"Owensboro","t":"6'H Stockade","p":10.3},{"f":"Owensboro","t":"6'H White Hamilton","p":27.85},{"f":"Palm Beach","t":"4'H Black 300 Sterling (R)","p":39.85},{"f":"Palm Beach","t":"5'H Black 300 Sterling (R)","p":45.15},{"f":"Palm Beach","t":"54H Black 300 Sterling","p":44.01},{"f":"Palm Beach","t":"6'H Cap and Trim Board on Board","p":74.34},{"f":"Palm Beach","t":"6'H Residential Galvanized Chain-Link","p":28.57},{"f":"Palm Beach","t":"6'H Stockade","p":34.02},{"f":"Palm Beach","t":"6'H White Hamilton","p":33.86},{"f":"Pasco County","t":"4'H Black 300 Sterling (R)","p":18.31},{"f":"Pasco County","t":"5'H Black 300 Sterling (R)","p":29.39},{"f":"Pasco County","t":"54H Black 300 Sterling","p":38.09},{"f":"Pasco County","t":"6'H Cap and Trim Board on Board","p":44.42},{"f":"Pasco County","t":"6'H Residential Galvanized Chain-Link","p":19.0},{"f":"Pasco County","t":"6'H Stockade","p":23.19},{"f":"Pasco County","t":"6'H White Hamilton","p":23.94},{"f":"Pensacola","t":"4'H Black 300 Sterling (R)","p":41.92},{"f":"Pensacola","t":"5'H Black 300 Sterling (R)","p":48.86},{"f":"Pensacola","t":"54H Black 300 Sterling","p":44.81},{"f":"Pensacola","t":"6'H Cap and Trim Board on Board","p":41.03},{"f":"Pensacola","t":"6'H Residential Galvanized Chain-Link","p":21.73},{"f":"Pensacola","t":"6'H Stockade","p":24.61},{"f":"Pensacola","t":"6'H White Hamilton","p":34.53},{"f":"Philadelphia","t":"4'H Black 300 Sterling (R)","p":35.02},{"f":"Philadelphia","t":"5'H Black 300 Sterling (R)","p":47.57},{"f":"Philadelphia","t":"54H Black 300 Sterling","p":36.57},{"f":"Philadelphia","t":"6'H Cap and Trim Board on Board","p":43.54},{"f":"Philadelphia","t":"6'H Residential Galvanized Chain-Link","p":30.39},{"f":"Philadelphia","t":"6'H Stockade","p":38.38},{"f":"Philadelphia","t":"6'H White Hamilton","p":39.4},{"f":"Pinellas County","t":"4'H Black 300 Sterling (R)","p":29.3},{"f":"Pinellas County","t":"5'H Black 300 Sterling (R)","p":45.46},{"f":"Pinellas County","t":"6'H Cap and Trim Board on Board","p":46.3},{"f":"Pinellas County","t":"6'H Residential Galvanized Chain-Link","p":26.64},{"f":"Pinellas County","t":"6'H Stockade","p":22.86},{"f":"Pinellas County","t":"6'H White Hamilton","p":26.11},{"f":"Pittsburgh","t":"4'H Black 300 Sterling (R)","p":41.8},{"f":"Pittsburgh","t":"5'H Black 300 Sterling (R)","p":56.43},{"f":"Pittsburgh","t":"6'H Residential Galvanized Chain-Link","p":37.62},{"f":"Portland Metro","t":"4'H Black 300 Sterling (R)","p":86.98},{"f":"Portland Metro","t":"5'H Black 300 Sterling (R)","p":84.07},{"f":"Portland Metro","t":"6'H Residential Galvanized Chain-Link","p":38.59},{"f":"Portland Metro","t":"6'H White Hamilton","p":61.02},{"f":"Raleigh","t":"4'H Black 300 Sterling (R)","p":32.82},{"f":"Raleigh","t":"5'H Black 300 Sterling (R)","p":35.56},{"f":"Raleigh","t":"54H Black 300 Sterling","p":34.99},{"f":"Raleigh","t":"6'H Cap and Trim Board on Board","p":34.24},{"f":"Raleigh","t":"6'H Residential Galvanized Chain-Link","p":24.27},{"f":"Raleigh","t":"6'H Stockade","p":27.36},{"f":"Raleigh","t":"6'H White Hamilton","p":34.76},{"f":"Rhode Island","t":"5'H Black 300 Sterling (R)","p":53.04},{"f":"Rhode Island","t":"54H Black 300 Sterling","p":51.22},{"f":"Rhode Island","t":"6'H Cap and Trim Board on Board","p":71.76},{"f":"Rhode Island","t":"6'H Residential Galvanized Chain-Link","p":31.98},{"f":"Rhode Island","t":"6'H Stockade","p":22.81},{"f":"Rhode Island","t":"6'H White Hamilton","p":32.01},{"f":"Richmond","t":"5'H Black 300 Sterling (R)","p":59.25},{"f":"Richmond","t":"54H Black 300 Sterling","p":46.22},{"f":"Richmond","t":"6'H Cap and Trim Board on Board","p":59.25},{"f":"Richmond","t":"6'H Residential Galvanized Chain-Link","p":28.18},{"f":"Richmond","t":"6'H Stockade","p":37.33},{"f":"Richmond","t":"6'H White Hamilton","p":37.79},{"f":"Riverside County","t":"6'H Residential Galvanized Chain-Link","p":33.61},{"f":"Riverside County","t":"6'H Stockade","p":47.33},{"f":"Riverside County","t":"6'H White Hamilton","p":49.58},{"f":"Roanoke","t":"5'H Black 300 Sterling (R)","p":51.33},{"f":"Roanoke","t":"54H Black 300 Sterling","p":50.39},{"f":"Roanoke","t":"6'H Cap and Trim Board on Board","p":57.05},{"f":"Roanoke","t":"6'H Residential Galvanized Chain-Link","p":25.1},{"f":"Roanoke","t":"6'H Stockade","p":29.05},{"f":"Roanoke","t":"6'H White Hamilton","p":41.7},{"f":"Rochester","t":"5'H Black 300 Sterling (R)","p":57.84},{"f":"Rochester","t":"54H Black 300 Sterling","p":56.82},{"f":"Rochester","t":"6'H Cap and Trim Board on Board","p":48.42},{"f":"Rochester","t":"6'H Residential Galvanized Chain-Link","p":31.52},{"f":"Rochester","t":"6'H Stockade","p":36.76},{"f":"Rochester","t":"6'H White Hamilton","p":40.01},{"f":"Sacramento","t":"5'H Black 300 Sterling (R)","p":25.72},{"f":"Sacramento","t":"6'H Residential Galvanized Chain-Link","p":30.46},{"f":"Sacramento","t":"6'H White Hamilton","p":51.63},{"f":"Salt Lake City","t":"5'H Black 300 Sterling (R)","p":27.07},{"f":"Salt Lake City","t":"54H Black 300 Sterling","p":26.37},{"f":"Salt Lake City","t":"6'H Residential Galvanized Chain-Link","p":26.51},{"f":"San Jose","t":"6'H Residential Galvanized Chain-Link","p":43.78},{"f":"San Jose","t":"6'H White Hamilton","p":55.11},{"f":"Sarasota","t":"5'H Black 300 Sterling (R)","p":44.89},{"f":"Sarasota","t":"54H Black 300 Sterling","p":55.05},{"f":"Sarasota","t":"6'H Cap and Trim Board on Board","p":65.9},{"f":"Sarasota","t":"6'H Residential Galvanized Chain-Link","p":21.42},{"f":"Sarasota","t":"6'H Stockade","p":27.95},{"f":"Sarasota","t":"6'H White Hamilton","p":26.23},{"f":"Savannah","t":"5'H Black 300 Sterling (R)","p":26.53},{"f":"Savannah","t":"54H Black 300 Sterling","p":24.22},{"f":"Savannah","t":"6'H Cap and Trim Board on Board","p":55.29},{"f":"Savannah","t":"6'H Residential Galvanized Chain-Link","p":26.08},{"f":"Savannah","t":"6'H Stockade","p":24.92},{"f":"Savannah","t":"6'H White Hamilton","p":35.83},{"f":"SE Houston","t":"5'H Black 300 Sterling (R)","p":27.22},{"f":"SE Houston","t":"54H Black 300 Sterling","p":24.84},{"f":"SE Houston","t":"6'H Cap and Trim Board on Board","p":31.69},{"f":"SE Houston","t":"6'H Residential Galvanized Chain-Link","p":26.25},{"f":"SE Houston","t":"6'H Stockade","p":24.57},{"f":"SE Houston","t":"6'H White Hamilton","p":35.07},{"f":"Shenandoah","t":"5'H Black 300 Sterling (R)","p":67.62},{"f":"Shenandoah","t":"54H Black 300 Sterling","p":70.38},{"f":"Shenandoah","t":"6'H Cap and Trim Board on Board","p":61.5},{"f":"Shenandoah","t":"6'H Residential Galvanized Chain-Link","p":28.29},{"f":"Shenandoah","t":"6'H Stockade","p":36.9},{"f":"Shenandoah","t":"6'H White Hamilton","p":52.79},{"f":"Sioux Falls","t":"5'H Black 300 Sterling (R)","p":36.66},{"f":"Sioux Falls","t":"54H Black 300 Sterling","p":33.49},{"f":"Sioux Falls","t":"6'H Cap and Trim Board on Board","p":59.03},{"f":"Sioux Falls","t":"6'H Residential Galvanized Chain-Link","p":19.13},{"f":"Sioux Falls","t":"6'H Stockade","p":28.71},{"f":"Sioux Falls","t":"6'H White Hamilton","p":41.2},{"f":"South Atlanta","t":"5'H Black 300 Sterling (R)","p":31.5},{"f":"South Atlanta","t":"54H Black 300 Sterling","p":41.42},{"f":"South Atlanta","t":"6'H Cap and Trim Board on Board","p":44.77},{"f":"South Atlanta","t":"6'H Residential Galvanized Chain-Link","p":15.19},{"f":"South Atlanta","t":"6'H Stockade","p":20.18},{"f":"South Atlanta","t":"6'H White Hamilton","p":27.04},{"f":"South Bay","t":"5'H Black 300 Sterling (R)","p":24.91},{"f":"South Bay","t":"54H Black 300 Sterling","p":22.74},{"f":"South Bay","t":"6'H Cap and Trim Board on Board","p":33.33},{"f":"South Bay","t":"6'H Residential Galvanized Chain-Link","p":40.74},{"f":"South Bay","t":"6'H Stockade","p":52.04},{"f":"South Bay","t":"6'H White Hamilton","p":52.95},{"f":"South Bend","t":"5'H Black 300 Sterling (R)","p":53.81},{"f":"South Bend","t":"54H Black 300 Sterling","p":22.74},{"f":"South Bend","t":"6'H Cap and Trim Board on Board","p":56.5},{"f":"South Bend","t":"6'H Residential Galvanized Chain-Link","p":27.0},{"f":"South Bend","t":"6'H Stockade","p":33.5},{"f":"South Bend","t":"6'H White Hamilton","p":42.0},{"f":"South Georgia","t":"5'H Black 300 Sterling (R)","p":39.0},{"f":"South Georgia","t":"54H Black 300 Sterling","p":38.29},{"f":"South Georgia","t":"6'H Cap and Trim Board on Board","p":40.0},{"f":"South Georgia","t":"6'H Residential Galvanized Chain-Link","p":25.0},{"f":"South Georgia","t":"6'H Stockade","p":24.0},{"f":"South Georgia","t":"6'H White Hamilton","p":33.0},{"f":"Southern Louisiana","t":"5'H Black 300 Sterling (R)","p":37.72},{"f":"Southern Louisiana","t":"54H Black 300 Sterling","p":35.34},{"f":"Southern Louisiana","t":"6'H Cap and Trim Board on Board","p":31.69},{"f":"Southern Louisiana","t":"6'H Residential Galvanized Chain-Link","p":26.25},{"f":"Southern Louisiana","t":"6'H Stockade","p":24.15},{"f":"Southern Louisiana","t":"6'H White Hamilton","p":54.24},{"f":"Southern Maryland","t":"5'H Black 300 Sterling (R)","p":48.76},{"f":"Southern Maryland","t":"54H Black 300 Sterling","p":47.0},{"f":"Southern Maryland","t":"6'H Cap and Trim Board on Board","p":41.0},{"f":"Southern Maryland","t":"6'H Residential Galvanized Chain-Link","p":22.55},{"f":"Southern Maryland","t":"6'H Stockade","p":25.88},{"f":"Southern Maryland","t":"6'H White Hamilton","p":32.54},{"f":"Southern Pennsylvania","t":"5'H Black 300 Sterling (R)","p":40.02},{"f":"Southern Pennsylvania","t":"54H Black 300 Sterling","p":36.11},{"f":"Southern Pennsylvania","t":"6'H Cap and Trim Board on Board","p":45.23},{"f":"Southern Pennsylvania","t":"6'H Residential Galvanized Chain-Link","p":36.25},{"f":"Southern Pennsylvania","t":"6'H Stockade","p":36.92},{"f":"Southern Pennsylvania","t":"6'H White Hamilton","p":35.31},{"f":"Southwest Florida","t":"5'H Black 300 Sterling (R)","p":39.38},{"f":"Southwest Florida","t":"54H Black 300 Sterling","p":38.85},{"f":"Southwest Florida","t":"6'H Cap and Trim Board on Board","p":53.0},{"f":"Southwest Florida","t":"6'H Residential Galvanized Chain-Link","p":22.08},{"f":"Southwest Florida","t":"6'H Stockade","p":27.56},{"f":"Southwest Florida","t":"6'H White Hamilton","p":31.82},{"f":"Spring","t":"5'H Black 300 Sterling (R)","p":25.92},{"f":"Spring","t":"54H Black 300 Sterling","p":23.66},{"f":"Spring","t":"6'H Cap and Trim Board on Board","p":30.18},{"f":"Spring","t":"6'H Residential Galvanized Chain-Link","p":30.0},{"f":"Spring","t":"6'H Stockade","p":28.0},{"f":"Spring","t":"6'H White Hamilton","p":33.4},{"f":"Springfield","t":"5'H Black 300 Sterling (R)","p":42.48},{"f":"Springfield","t":"54H Black 300 Sterling","p":41.38},{"f":"Springfield","t":"6'H Cap and Trim Board on Board","p":28.63},{"f":"Springfield","t":"6'H Residential Galvanized Chain-Link","p":29.97},{"f":"Springfield","t":"6'H Stockade","p":28.61},{"f":"Springfield","t":"6'H White Hamilton","p":38.89},{"f":"St Louis","t":"5'H Black 300 Sterling (R)","p":44.4},{"f":"St Louis","t":"54H Black 300 Sterling","p":41.94},{"f":"St Louis","t":"6'H Residential Galvanized Chain-Link","p":36.99},{"f":"St Louis","t":"6'H Stockade","p":30.85},{"f":"St Louis","t":"6'H White Hamilton","p":39.15},{"f":"St. Paul","t":"5'H Black 300 Sterling (R)","p":44.1},{"f":"St. Paul","t":"54H Black 300 Sterling","p":23.88},{"f":"St. Paul","t":"6'H Cap and Trim Board on Board","p":47.73},{"f":"St. Paul","t":"6'H Residential Galvanized Chain-Link","p":42.0},{"f":"St. Paul","t":"6'H Stockade","p":38.28},{"f":"St. Paul","t":"6'H White Hamilton","p":46.23},{"f":"Suffolk County","t":"5'H Black 300 Sterling (R)","p":50.99},{"f":"Suffolk County","t":"54H Black 300 Sterling","p":48.41},{"f":"Suffolk County","t":"6'H Cap and Trim Board on Board","p":43.54},{"f":"Suffolk County","t":"6'H Residential Galvanized Chain-Link","p":26.78},{"f":"Suffolk County","t":"6'H Stockade","p":33.07},{"f":"Suffolk County","t":"6'H White Hamilton","p":29.36},{"f":"SW Georgia","t":"5'H Black 300 Sterling (R)","p":43.47},{"f":"SW Georgia","t":"54H Black 300 Sterling","p":57.16},{"f":"SW Georgia","t":"6'H Cap and Trim Board on Board","p":61.78},{"f":"SW Georgia","t":"6'H Residential Galvanized Chain-Link","p":19.62},{"f":"SW Georgia","t":"6'H Stockade","p":27.85},{"f":"SW Georgia","t":"6'H White Hamilton","p":41.37},{"f":"SW Houston","t":"5'H Black 300 Sterling (R)","p":44.86},{"f":"SW Houston","t":"6'H Residential Galvanized Chain-Link","p":24.72},{"f":"SW Houston","t":"6'H Stockade","p":30.24},{"f":"SW Houston","t":"6'H White Hamilton","p":40.19},{"f":"SW Michigan","t":"5'H Black 300 Sterling (R)","p":52.64},{"f":"SW Michigan","t":"54H Black 300 Sterling","p":40.32},{"f":"SW Michigan","t":"6'H Residential Galvanized Chain-Link","p":27.04},{"f":"SW Michigan","t":"6'H Stockade","p":27.72},{"f":"SW Michigan","t":"6'H White Hamilton","p":41.72},{"f":"Tacoma","t":"6'H Residential Galvanized Chain-Link","p":41.29},{"f":"Tacoma","t":"6'H Stockade","p":108.0},{"f":"Tacoma","t":"6'H White Hamilton","p":51.59},{"f":"Tallahassee","t":"5'H Black 300 Sterling (R)","p":26.16},{"f":"Tallahassee","t":"54H Black 300 Sterling","p":34.65},{"f":"Tallahassee","t":"6'H Cap and Trim Board on Board","p":47.01},{"f":"Tallahassee","t":"6'H Residential Galvanized Chain-Link","p":22.31},{"f":"Tallahassee","t":"6'H Stockade","p":23.16},{"f":"Tallahassee","t":"6'H White Hamilton","p":32.55},{"f":"Treasure Coast","t":"5'H Black 300 Sterling (R)","p":37.95},{"f":"Treasure Coast","t":"54H Black 300 Sterling","p":35.19},{"f":"Treasure Coast","t":"6'H Cap and Trim Board on Board","p":88.84},{"f":"Treasure Coast","t":"6'H Residential Galvanized Chain-Link","p":25.76},{"f":"Treasure Coast","t":"6'H Stockade","p":28.84},{"f":"Treasure Coast","t":"6'H White Hamilton","p":33.22},{"f":"Treasure Valley","t":"5'H Black 300 Sterling (R)","p":26.55},{"f":"Treasure Valley","t":"54H Black 300 Sterling","p":25.86},{"f":"Treasure Valley","t":"6'H Residential Galvanized Chain-Link","p":46.35},{"f":"Treasure Valley","t":"6'H White Hamilton","p":28.63},{"f":"Tulsa","t":"5'H Black 300 Sterling (R)","p":42.48},{"f":"Tulsa","t":"54H Black 300 Sterling","p":41.38},{"f":"Tulsa","t":"6'H Cap and Trim Board on Board","p":37.23},{"f":"Tulsa","t":"6'H Residential Galvanized Chain-Link","p":29.97},{"f":"Tulsa","t":"6'H Stockade","p":27.31},{"f":"Tulsa","t":"6'H White Hamilton","p":38.89},{"f":"Ventura","t":"5'H Black 300 Sterling (R)","p":25.66},{"f":"Ventura","t":"54H Black 300 Sterling","p":23.42},{"f":"Ventura","t":"6'H Cap and Trim Board on Board","p":34.33},{"f":"Ventura","t":"6'H Residential Galvanized Chain-Link","p":55.82},{"f":"Ventura","t":"6'H Stockade","p":58.96},{"f":"Ventura","t":"6'H White Hamilton","p":51.5},{"f":"Volusia","t":"5'H Black 300 Sterling (R)","p":39.6},{"f":"Volusia","t":"6'H Residential Galvanized Chain-Link","p":19.87},{"f":"Volusia","t":"6'H Stockade","p":23.69},{"f":"West Houston","t":"5'H Black 300 Sterling (R)","p":43.55},{"f":"West Houston","t":"54H Black 300 Sterling","p":39.75},{"f":"West Houston","t":"6'H Cap and Trim Board on Board","p":50.77},{"f":"West Houston","t":"6'H Residential Galvanized Chain-Link","p":32.32},{"f":"West Houston","t":"6'H Stockade","p":25.03},{"f":"West Houston","t":"6'H White Hamilton","p":34.7},{"f":"West Los Angeles","t":"5'H Black 300 Sterling (R)","p":25.66},{"f":"West Los Angeles","t":"54H Black 300 Sterling","p":23.42},{"f":"West Los Angeles","t":"6'H Cap and Trim Board on Board","p":34.33},{"f":"West Los Angeles","t":"6'H Residential Galvanized Chain-Link","p":46.35},{"f":"West Los Angeles","t":"6'H Stockade","p":53.6},{"f":"West Los Angeles","t":"6'H White Hamilton","p":53.12},{"f":"Wilmington","t":"5'H Black 300 Sterling (R)","p":41.2},{"f":"Wilmington","t":"54H Black 300 Sterling","p":37.08},{"f":"Wilmington","t":"6'H Cap and Trim Board on Board","p":46.35},{"f":"Wilmington","t":"6'H Residential Galvanized Chain-Link","p":19.6},{"f":"Wilmington","t":"6'H Stockade","p":29.87},{"f":"Wilmington","t":"6'H White Hamilton","p":36.05},{"f":"Winter Haven","t":"5'H Black 300 Sterling (R)","p":36.5},{"f":"Winter Haven","t":"54H Black 300 Sterling","p":34.5},{"f":"Winter Haven","t":"6'H Cap and Trim Board on Board","p":99.0},{"f":"Winter Haven","t":"6'H Residential Galvanized Chain-Link","p":21.5},{"f":"Winter Haven","t":"6'H Stockade","p":30.0},{"f":"Winter Haven","t":"6'H White Hamilton","p":23.0}];

const COORDS = {"Akron":[41.08,-81.52],"Albany":[42.65,-73.75],"Arkansas":[34.75,-92.29],"Athens":[33.96,-83.38],"Atlanta":[33.75,-84.39],"Augusta":[33.47,-82.01],"Baltimore":[39.29,-76.61],"Birmingham":[33.52,-86.80],"Brevard County":[28.25,-80.73],"Broward":[26.15,-80.25],"Buffalo":[42.89,-78.87],"Central & South Jersey":[39.95,-74.50],"Central Iowa":[41.6,-93.6],"Central Jersey":[40.3,-74.5],"Central Oklahoma":[35.47,-97.52],"Central Texas":[30.97,-97.0],"Charleston":[32.78,-79.94],"Charlotte":[35.23,-80.84],"Chattanooga":[35.05,-85.31],"Chicagoland":[41.85,-87.65],"Cincinnati":[39.10,-84.51],"Cleveland":[41.50,-81.69],"Colorado Springs":[38.83,-104.82],"Columbia":[34.0,-81.03],"Columbus":[39.96,-82.99],"Connecticut":[41.6,-72.7],"Delaware Valley":[39.95,-75.17],"Delmarva":[38.7,-75.6],"Detroit":[42.33,-83.05],"East Bay":[37.8,-122.25],"East Texas":[31.9,-95.5],"Eastern NC":[35.6,-77.4],"Fairfax & Montgomery":[38.9,-77.2],"Fayetteville":[35.05,-78.88],"Fort Collins":[40.59,-105.08],"Gainesville":[29.65,-82.33],"Grand Rapids":[42.96,-85.67],"Great Lakes Bay Region":[43.6,-84.0],"Greater Boston":[42.36,-71.06],"Greater Lexington":[38.05,-84.5],"Greater Louisville":[38.25,-85.76],"Greater Seattle":[47.6,-122.33],"Greater Toledo":[41.66,-83.56],"Greensboro":[36.07,-79.79],"Greenville":[34.85,-82.39],"Hartford":[41.76,-72.68],"Hudson Valley":[41.7,-74.0],"Huntsville":[34.73,-86.59],"Indianapolis":[39.77,-86.16],"Inland Empire":[33.95,-117.4],"Jackson":[32.30,-90.18],"Kansas City":[39.10,-94.58],"Knoxville":[35.97,-83.92],"Lake County":[28.75,-81.75],"Lansing":[42.73,-84.56],"Little Rock":[34.75,-92.29],"Madison & Rockford":[43.07,-89.4],"Memphis":[35.15,-90.05],"Middle Georgia":[32.5,-83.5],"Milwaukee":[43.04,-87.91],"Minneapolis":[44.98,-93.27],"Myrtle Beach":[33.69,-78.89],"NW Georgia":[34.3,-85.2],"Nashville":[36.16,-86.78],"Nassau County":[40.73,-73.6],"Norfolk":[36.85,-76.29],"North Dallas":[33.2,-96.8],"North Florida":[30.33,-81.66],"North Houston":[30.1,-95.4],"North San Diego":[33.1,-117.1],"North Shore":[42.5,-71.1],"North Texas":[34.0,-97.5],"Northeastern Los Angeles":[34.1,-117.9],"Northern Kentucky":[39.05,-84.6],"Northern NJ":[40.9,-74.2],"Northern Virginia":[38.9,-77.2],"Omaha":[41.26,-96.0],"Orange County CA":[33.7,-117.8],"Oviedo":[28.67,-81.2],"Owensboro":[37.77,-87.11],"Palm Beach":[26.71,-80.06],"Pasco County":[28.3,-82.4],"Pensacola":[30.42,-87.22],"Philadelphia":[39.95,-75.17],"Pinellas County":[27.9,-82.7],"Pittsburgh":[40.44,-79.99],"Portland Metro":[45.52,-122.68],"Raleigh":[35.77,-78.64],"Rhode Island":[41.7,-71.5],"Richmond":[37.54,-77.44],"Riverside County":[33.7,-116.5],"Roanoke":[37.27,-79.94],"Rochester":[43.16,-77.61],"SE Houston":[29.6,-95.1],"SW Georgia":[31.2,-84.4],"SW Houston":[29.6,-95.6],"SW Michigan":[42.0,-86.5],"Sacramento":[38.58,-121.49],"Salt Lake City":[40.77,-111.89],"San Jose":[37.34,-121.89],"Sarasota":[27.34,-82.53],"Savannah":[32.08,-81.1],"Shenandoah":[38.8,-78.5],"Sioux Falls":[43.54,-96.73],"South Atlanta":[33.5,-84.3],"South Bay":[33.8,-118.3],"South Bend":[41.68,-86.25],"South Georgia":[31.0,-83.5],"Southern Louisiana":[30.0,-91.0],"Southern Maryland":[38.5,-76.7],"Southern Pennsylvania":[40.0,-77.5],"Southwest Florida":[26.5,-81.8],"Spring":[30.07,-95.42],"Springfield":[39.8,-89.64],"St Louis":[38.63,-90.2],"St. Paul":[44.95,-93.09],"Suffolk County":[40.8,-73.2],"Tacoma":[47.25,-122.44],"Tallahassee":[30.44,-84.28],"Treasure Coast":[27.2,-80.3],"Treasure Valley":[43.6,-116.2],"Tulsa":[36.15,-95.99],"Ventura":[34.27,-119.25],"Volusia":[29.1,-81.0],"West Houston":[29.7,-95.7],"West Los Angeles":[34.05,-118.45],"Wilmington":[34.23,-77.94],"Winter Haven":[28.02,-81.73]};

const MY = ['Raleigh','Greensboro','Atlanta'];
const FENCE_TYPES = ["4'H Black 300 Sterling (R)","5'H Black 300 Sterling (R)","54H Black 300 Sterling","6'H Cap and Trim Board on Board","6'H Residential Galvanized Chain-Link","6'H Stockade","6'H White Hamilton"];
const FENCE_SHORT = ["4'H Sterling","5'H Sterling","54H Sterling","Cap & Trim","Chain-Link","Stockade","White Hamilton"];

let state = { fence: FENCE_TYPES[0], view: 'all' };

// --- Geo ---
function haversine(lat1,lon1,lat2,lon2){
  const R=3958.8, dLat=(lat2-lat1)*Math.PI/180, dLon=(lon2-lon1)*Math.PI/180;
  const a=Math.sin(dLat/2)**2+Math.cos(lat1*Math.PI/180)*Math.cos(lat2*Math.PI/180)*Math.sin(dLon/2)**2;
  return R*2*Math.asin(Math.sqrt(a));
}
function nearest15(from){
  const [la,lo]=COORDS[from]||[0,0];
  const all=Object.keys(COORDS).filter(f=>f!==from);
  return all.sort((a,b)=>{
    const [la2,lo2]=COORDS[a], [la3,lo3]=COORDS[b];
    return haversine(la,lo,la2,lo2)-haversine(la,lo,la3,lo3);
  }).slice(0,15);
}
function distMiles(a,b){
  if(!COORDS[a]||!COORDS[b]) return null;
  const [la,lo]=COORDS[a],[la2,lo2]=COORDS[b];
  return Math.round(haversine(la,lo,la2,lo2));
}

// --- Beeswarm ---
function beeswarm(pts, r, xFn){
  const sorted=[...pts].sort((a,b)=>a.p-b.p);
  const placed=[];
  for(const pt of sorted){
    const cx=xFn(pt.p);
    let cy=0, step=0, found=false;
    while(!found){
      found=true;
      for(const pp of placed){
        const dx=cx-pp.cx, dy=cy-pp.cy;
        if(dx*dx+dy*dy<(r*2.2)**2){found=false;break;}
      }
      if(!found){
        step++;
        const dir=step%2===0?1:-1;
        cy=dir*Math.ceil(step/2)*(r*2.2);
        if(Math.abs(cy)>r*80){cy=r*(step%2===0?1:-1)*2*(step/2);break;}
      }
    }
    placed.push({...pt,cx,cy});
  }
  return placed;
}

// --- Render ---
function getPoints(){
  const pts=RAW.filter(d=>d.t===state.fence);
  if(state.view==='all') return {pts, nearSet:new Set(), refLoc:null};
  const refLoc=state.view;
  const near=new Set(nearest15(refLoc));
  const showSet=new Set([...MY,...near]);
  return {pts:pts.filter(d=>showSet.has(d.f)), nearSet:near, refLoc};
}

function render(){
  const svg=document.getElementById('chart-svg');
  const W=Math.max(svg.parentElement.clientWidth-32,600);
  const PAD_L=12, PAD_R=12, PAD_TOP=110, PAD_BOT=60;
  const R=7;

  const {pts, nearSet, refLoc}=getPoints();
  if(!pts.length){
    svg.setAttribute('viewBox',`0 0 ${W} 200`);
    svg.innerHTML=`<text x="${W/2}" y="100" text-anchor="middle" fill="#484f58" font-size="14">No data for this fence type in selected view</text>`;
    document.getElementById('stats-bar').innerHTML='<span class="stat"><span class="stat-label">No data</span></span>';
    return;
  }

  const prices=pts.map(d=>d.p);
  const pMin=Math.min(...prices), pMax=Math.max(...prices);
  const pRange=pMax-pMin||1;
  const pad=pRange*0.08;
  const domMin=Math.max(0,pMin-pad), domMax=pMax+pad;

  const xFn=p=>(p-domMin)/(domMax-domMin)*(W-PAD_L-PAD_R)+PAD_L;

  const placed=beeswarm(pts,R,xFn);
  const maxCY=Math.max(...placed.map(d=>Math.abs(d.cy)),1);
  const chartH=Math.max(maxCY*2+PAD_TOP+PAD_BOT, 280);
  const midY=PAD_TOP+maxCY+10;

  svg.setAttribute('width',W);
  svg.setAttribute('height',chartH);
  svg.setAttribute('viewBox',`0 0 ${W} ${chartH}`);

  // Stats
  const myPts=pts.filter(d=>MY.includes(d.f));
  const avg=prices.reduce((a,b)=>a+b,0)/prices.length;
  const sortedP=[...prices].sort((a,b)=>a-b);
  const statsEl=document.getElementById('stats-bar');
  let statsHTML=`<div class="stat"><span class="stat-label">Showing</span><span class="stat-value">${pts.length} franchises</span></div>`;
  statsHTML+=`<div class="stat"><span class="stat-label">Min</span><span class="stat-value">$${pMin.toFixed(2)}</span></div>`;
  statsHTML+=`<div class="stat"><span class="stat-label">Max</span><span class="stat-value">$${pMax.toFixed(2)}</span></div>`;
  statsHTML+=`<div class="stat"><span class="stat-label">Avg</span><span class="stat-value">$${avg.toFixed(2)}</span></div>`;
  for(const loc of MY){
    const pt=pts.find(d=>d.f===loc);
    if(pt){
      const rank=sortedP.filter(p=>p<=pt.p).length;
      const pct=Math.round(rank/sortedP.length*100);
      statsHTML+=`<div class="stat"><span class="stat-label">${loc}</span><span class="stat-my">$${pt.p.toFixed(2)} (${pct}th pctile)</span></div>`;
    }
  }
  statsEl.innerHTML=statsHTML;

  // Build SVG
  let html='';

  // Grid lines
  const ticks=[];
  const step=pRange<10?1:pRange<30?5:pRange<80?10:20;
  let t=Math.ceil(domMin/step)*step;
  while(t<=domMax){ticks.push(t);t+=step;}
  for(const tk of ticks){
    const x=xFn(tk);
    html+=`<line x1="${x}" y1="${PAD_TOP-10}" x2="${x}" y2="${chartH-PAD_BOT+5}" stroke="#21262d" stroke-width="1"/>`;
    html+=`<text x="${x}" y="${chartH-PAD_BOT+18}" text-anchor="middle" fill="#484f58" font-size="10">$${tk}</text>`;
  }

  // Axis line
  html+=`<line x1="${PAD_L}" y1="${midY}" x2="${W-PAD_R}" y2="${midY}" stroke="#30363d" stroke-width="1.5"/>`;

  // Dots — draw non-my first, then my on top
  const nonMy=placed.filter(d=>!MY.includes(d.f));
  const myPlaced=placed.filter(d=>MY.includes(d.f));

  for(const d of nonMy){
    const isNear=nearSet.has(d.f);
    const fill=isNear?'#a78bfa':'#58a6ff';
    const opacity=isNear?0.85:0.55;
    const r2=isNear?R+1:R;
    html+=`<circle class="dot" cx="${d.cx}" cy="${midY+d.cy}" r="${r2}" fill="${fill}" opacity="${opacity}" stroke="none" data-f="${d.f}" data-p="${d.p}" data-loc="${refLoc||''}" style="cursor:pointer"/>`;
  }

  // MY dots
  for(const d of myPlaced){
    html+=`<circle class="dot my-dot" cx="${d.cx}" cy="${midY+d.cy}" r="${R+3}" fill="#f59e0b" opacity="1" stroke="#fbbf24" stroke-width="1.5" data-f="${d.f}" data-p="${d.p}" data-loc="" style="cursor:pointer;filter:drop-shadow(0 0 5px #f59e0b88)"/>`;
  }

  // Labels for MY locations — alternating above/below
  const myLabeled=myPlaced.sort((a,b)=>a.cx-b.cx);
  const labelPositions=[];
  myLabeled.forEach((d,i)=>{
    // Alternate above/below
    const above=i%2===0;
    const lx=d.cx;
    const dotY=midY+d.cy;
    const lineY1=above?dotY-(R+4):dotY+(R+4);
    const lineY2=above?dotY-32:dotY+32;
    const labelY=above?lineY2-6:lineY2+14;
    html+=`<line x1="${lx}" y1="${lineY1}" x2="${lx}" y2="${lineY2}" stroke="#f59e0b" stroke-width="1" stroke-dasharray="3,2" opacity="0.7"/>`;
    html+=`<rect x="${lx-38}" y="${labelY-13}" width="76" height="17" rx="3" fill="#78350f" opacity="0.9"/>`;
    html+=`<text x="${lx}" y="${labelY}" text-anchor="middle" fill="#fbbf24" font-size="11" font-weight="700">${d.f}  $${d.p.toFixed(2)}</text>`;
  });

  html+=`</svg>`;

  // note
  let noteText=`Fence type: ${state.fence}`;
  if(state.view!=='all') noteText+=` · Showing your 3 locations + 15 nearest to ${state.view} (by air distance)`;
  document.getElementById('chart-note').textContent=noteText;

  svg.innerHTML=html;

  // Tooltip events
  const tooltip=document.getElementById('tooltip');
  svg.querySelectorAll('.dot').forEach(el=>{
    el.addEventListener('mouseenter',e=>{
      const fname=el.getAttribute('data-f');
      const price=parseFloat(el.getAttribute('data-p'));
      const loc=el.getAttribute('data-loc');
      document.getElementById('tt-name').textContent=fname;
      document.getElementById('tt-price').textContent=`$${price.toFixed(2)} / lin ft`;
      let distText='';
      if(loc && COORDS[fname] && COORDS[loc]){
        const d=distMiles(loc,fname);
        distText=`~${d} mi from ${loc}`;
      }
      document.getElementById('tt-dist').textContent=distText;
      tooltip.style.display='block';
    });
    el.addEventListener('mousemove',e=>{
      const r=tooltip.getBoundingClientRect();
      let lx=e.clientX+14, ly=e.clientY-10;
      if(lx+r.width>window.innerWidth) lx=e.clientX-r.width-10;
      if(ly+r.height>window.innerHeight) ly=e.clientY-r.height-10;
      tooltip.style.left=lx+'px';
      tooltip.style.top=ly+'px';
    });
    el.addEventListener('mouseleave',()=>{tooltip.style.display='none';});
  });
}

// --- Build UI ---
function buildTabs(){
  const el=document.getElementById('fence-tabs');
  FENCE_TYPES.forEach((ft,i)=>{
    const t=document.createElement('div');
    t.className='tab'+(ft===state.fence?' active':'');
    t.textContent=FENCE_SHORT[i];
    t.title=ft;
    t.addEventListener('click',()=>{
      state.fence=ft;
      el.querySelectorAll('.tab').forEach(x=>x.classList.remove('active'));
      t.classList.add('active');
      render();
    });
    el.appendChild(t);
  });
}

function buildViewBtns(){
  const el=document.getElementById('view-btns');
  const views=[
    {id:'all',label:'All Locations',cls:'all'},
    {id:'Raleigh',label:'Near Raleigh ★',cls:''},
    {id:'Greensboro',label:'Near Greensboro ★',cls:''},
    {id:'Atlanta',label:'Near Atlanta ★',cls:''},
  ];
  views.forEach(v=>{
    const b=document.createElement('div');
    b.className='view-btn '+(v.cls)+' '+(state.view===v.id?'active':'');
    b.textContent=v.label;
    b.addEventListener('click',()=>{
      state.view=v.id;
      el.querySelectorAll('.view-btn').forEach(x=>x.classList.remove('active'));
      b.classList.add('active');
      render();
    });
    el.appendChild(b);
  });
}

buildTabs();
buildViewBtns();
render();
window.addEventListener('resize',()=>render());
</script>
</body>
</html>
