# 您吃了吗？ Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-page mobile web app that randomly picks nearby restaurants within a budget range using Amap POI search.

**Architecture:** Single `index.html` file with embedded `<style>` and `<script>`. No build tools, no frameworks, no backend. Amap JS API v2 loaded via CDN `<script>` tag. Data persisted in `localStorage`.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript (ES6+), Amap Web JS API v2, Browser Geolocation API, localStorage

---

### Task 1: Project scaffold — HTML skeleton + CSS layout

**Files:**
- Create: `index.html`

- [ ] **Step 1: Write the HTML structure with all UI sections**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>您吃了吗？</title>
  <style>
    /* CSS goes here in Task 2 */
  </style>
</head>
<body>
  <header id="app-header">
    <h1 class="title">🍜 您吃了吗？</h1>
    <p class="subtitle">Do You Eat Yet?</p>
  </header>

  <main id="app-main">
    <!-- API Key banner -->
    <div id="key-banner" class="warning-banner hidden">
      ⚠️ 请先配置高德地图 API Key（<a href="#" id="key-link">点击设置</a>）
    </div>

    <!-- Key input modal (hidden by default) -->
    <div id="key-modal" class="modal hidden">
      <div class="modal-content">
        <h3>设置 API Key</h3>
        <p>前往 <a href="https://lbs.amap.com" target="_blank">lbs.amap.com</a> 注册并创建应用，获取 Web 服务 Key</p>
        <input type="text" id="key-input" placeholder="粘贴你的 Key">
        <button id="key-save">保存</button>
        <button id="key-cancel">取消</button>
      </div>
    </div>

    <!-- Location section -->
    <section id="location-section">
      <div class="loc-row">
        <span class="loc-icon">📍</span>
        <span id="loc-text">正在获取位置...</span>
        <button id="loc-retry" class="btn-small hidden">重定位</button>
      </div>
      <!-- Manual location fallback -->
      <div id="manual-loc" class="hidden">
        <input type="text" id="manual-address" placeholder="手动输入地址">
        <button id="manual-search">搜索</button>
      </div>
    </section>

    <!-- Radius selector -->
    <section id="radius-section">
      <label class="param-label">搜索范围</label>
      <select id="radius-select">
        <option value="500">500m</option>
        <option value="1000" selected>1000m</option>
        <option value="2000">2000m</option>
        <option value="3000">3000m</option>
        <option value="5000">5000m</option>
      </select>
    </section>

    <!-- Budget range -->
    <section id="budget-section">
      <label class="param-label">人均预算</label>
      <div class="budget-row">
        <span>¥</span>
        <input type="number" id="budget-min" value="10" min="0" max="500" placeholder="最低">
        <span>～</span>
        <input type="number" id="budget-max" value="50" min="0" max="500" placeholder="最高">
      </div>
    </section>

    <!-- CTA Button -->
    <section id="action-section">
      <button id="btn-pick" disabled>🎲 帮我决定</button>
    </section>

    <!-- Result card (hidden initially) -->
    <section id="result-section" class="hidden">
      <div id="result-card">
        <div class="result-top">
          <span id="result-icon"></span>
          <span id="result-name"></span>
          <span id="result-rating"></span>
        </div>
        <div class="result-mid">
          <span id="result-cost"></span>
          <span id="result-distance"></span>
        </div>
        <div class="result-addr" id="result-address"></div>
        <div class="result-actions">
          <button id="btn-reroll" class="btn-secondary">🔄 换一个</button>
          <button id="btn-confirm" class="btn-confirm">✅ 就它了</button>
        </div>
      </div>
    </section>

    <!-- Loading state -->
    <div id="loading" class="hidden">
      <div class="spinner"></div>
      <p>正在搜索周边的美食...</p>
    </div>

    <!-- Error / empty state -->
    <div id="message" class="hidden"></div>

    <!-- History -->
    <section id="history-section">
      <h4>📋 最近选择</h4>
      <div id="history-list"></div>
    </section>
  </main>

  <!-- Amap JS API SDK -->
  <script src="https://webapi.amap.com/maps?v=2.0&key=PLACEHOLDER_KEY&plugin=AMap.PlaceSearch,AMap.Geolocation"></script>
  <script>
    // App logic starts here in Task 3
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify the scaffold renders**

Open `index.html` in browser. You should see the header, dropdown, budget inputs, a disabled button, and an empty history section. The result card should be hidden.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML scaffold for food picker app"
```

---

### Task 2: CSS styling — warm food theme, mobile-first layout

**Files:**
- Modify: `index.html` — replace the empty `<style>` block

- [ ] **Step 1: Write the complete CSS**

Replace the empty `<style></style>` block with:

```css
/* ====== Reset & Base ====== */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --primary: #FF6B35;
  --primary-dark: #E55A2B;
  --bg: #FFF8F0;
  --card-bg: #FFFFFF;
  --text: #333333;
  --text-light: #888888;
  --green: #4CAF50;
  --red: #F44336;
  --radius: 12px;
  --shadow: 0 2px 12px rgba(0,0,0,0.08);
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
               "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
  background: var(--bg);
  color: var(--text);
  max-width: 420px;
  margin: 0 auto;
  min-height: 100vh;
  padding: 16px;
  -webkit-tap-highlight-color: transparent;
}

/* ====== Header ====== */
#app-header {
  text-align: center;
  padding: 24px 0 8px;
}
.title { font-size: 28px; font-weight: 700; }
.subtitle { font-size: 13px; color: var(--text-light); margin-top: 2px; }

/* ====== Warning Banner ====== */
.warning-banner {
  background: #FFF3CD;
  border: 1px solid #FFC107;
  border-radius: var(--radius);
  padding: 10px 14px;
  font-size: 13px;
  margin-bottom: 12px;
}
.warning-banner a { color: var(--primary); }

/* ====== Modal ====== */
.modal {
  position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex; align-items: center; justify-content: center;
  z-index: 100;
}
.modal-content {
  background: white; border-radius: var(--radius);
  padding: 20px; width: 90%; max-width: 360px;
}
.modal-content h3 { margin-bottom: 8px; }
.modal-content p { font-size: 13px; color: var(--text-light); margin-bottom: 12px; }
.modal-content input {
  width: 100%; padding: 10px; border: 1px solid #ddd;
  border-radius: 8px; font-size: 14px; margin-bottom: 12px;
}
.modal-content button {
  padding: 8px 16px; border: none; border-radius: 8px;
  font-size: 14px; cursor: pointer; margin-right: 8px;
}
#key-save { background: var(--primary); color: white; }
#key-cancel { background: #eee; color: var(--text); }

/* ====== Sections ====== */
section { margin-bottom: 16px; }

.param-label {
  display: block; font-size: 13px; color: var(--text-light);
  margin-bottom: 6px; font-weight: 600;
}

/* ====== Location ====== */
#location-section {
  background: var(--card-bg); border-radius: var(--radius);
  padding: 14px; box-shadow: var(--shadow);
}
.loc-row { display: flex; align-items: center; gap: 8px; }
.loc-icon { font-size: 18px; }
#loc-text { font-size: 14px; flex: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.btn-small {
  font-size: 12px; padding: 4px 10px; border: 1px solid #ddd;
  border-radius: 14px; background: white; cursor: pointer;
}
#manual-loc { margin-top: 10px; display: flex; gap: 8px; }
#manual-loc input { flex: 1; padding: 8px; border: 1px solid #ddd; border-radius: 8px; font-size: 14px; }
#manual-loc button {
  padding: 8px 14px; border: none; border-radius: 8px;
  background: var(--primary); color: white; cursor: pointer; font-size: 14px;
}

/* ====== Radius Select ====== */
#radius-section {
  background: var(--card-bg); border-radius: var(--radius);
  padding: 14px; box-shadow: var(--shadow);
}
#radius-select {
  width: 100%; padding: 10px; border: 1px solid #ddd;
  border-radius: 8px; font-size: 16px; background: white;
}

/* ====== Budget ====== */
#budget-section {
  background: var(--card-bg); border-radius: var(--radius);
  padding: 14px; box-shadow: var(--shadow);
}
.budget-row { display: flex; align-items: center; gap: 8px; }
.budget-row span { font-size: 15px; color: var(--text-light); }
.budget-row input {
  flex: 1; padding: 10px; border: 1px solid #ddd;
  border-radius: 8px; font-size: 16px; min-width: 0;
}

/* ====== CTA Button ====== */
#btn-pick {
  width: 100%; padding: 16px; font-size: 20px; font-weight: 700;
  border: none; border-radius: var(--radius); cursor: pointer;
  background: var(--primary); color: white;
  box-shadow: 0 4px 16px rgba(255,107,53,0.35);
  transition: transform 0.1s, box-shadow 0.1s;
}
#btn-pick:active { transform: scale(0.97); }
#btn-pick:disabled {
  background: #ccc; cursor: not-allowed;
  box-shadow: none;
}

/* ====== Result Card ====== */
#result-section { margin-top: 4px; }
#result-card {
  background: var(--card-bg); border-radius: var(--radius);
  padding: 18px; box-shadow: var(--shadow);
  border-left: 4px solid var(--primary);
  animation: slideUp 0.3s ease;
}
@keyframes slideUp {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}
.result-top {
  display: flex; align-items: center; gap: 8px;
  font-size: 18px; font-weight: 700; margin-bottom: 8px;
}
#result-icon { font-size: 22px; }
#result-rating { font-size: 13px; color: #F5A623; margin-left: auto; }
.result-mid {
  display: flex; gap: 16px; font-size: 14px;
  color: var(--text-light); margin-bottom: 8px;
}
#result-cost { color: var(--primary); font-weight: 600; }
.result-addr {
  font-size: 12px; color: var(--text-light);
  margin-bottom: 14px;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.result-actions { display: flex; gap: 10px; }
.result-actions button {
  flex: 1; padding: 12px; border: none; border-radius: 8px;
  font-size: 15px; cursor: pointer; font-weight: 600;
}
#btn-reroll { background: #FFF0E8; color: var(--primary); }
#btn-confirm { background: var(--green); color: white; }

/* ====== Loading ====== */
#loading { text-align: center; padding: 24px; }
.spinner {
  width: 36px; height: 36px; border: 3px solid #eee;
  border-top-color: var(--primary); border-radius: 50%;
  animation: spin 0.8s linear infinite; margin: 0 auto 12px;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ====== Message ====== */
#message {
  background: var(--card-bg); border-radius: var(--radius);
  padding: 16px; text-align: center; font-size: 14px;
  box-shadow: var(--shadow);
}
#message.error { border-left: 4px solid var(--red); }
#message.info { border-left: 4px solid var(--primary); }

/* ====== History ====== */
#history-section { margin-top: 24px; }
#history-section h4 {
  font-size: 14px; color: var(--text-light); margin-bottom: 8px;
}
#history-list { display: flex; flex-wrap: wrap; gap: 8px; }
.history-item {
  background: var(--card-bg); border-radius: 20px;
  padding: 6px 14px; font-size: 13px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.06);
}

/* ====== Utility ====== */
.hidden { display: none !important; }
```

- [ ] **Step 2: Verify the styling**

Open `index.html` in browser. Confirm: warm cream background, orange header, white cards with rounded corners, max-width 420px centered on desktop. Check that the disabled "帮我决定" button is grayed out.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: add warm food-themed CSS with mobile-first layout"
```

---

### Task 3: API Key management — save, load, validate

**Files:**
- Modify: `index.html` — replace the empty `<script>` block

- [ ] **Step 1: Write the API Key management logic**

Replace the empty `<script></script>` block (the one AFTER the Amap SDK script tag) with:

```javascript
<script>
  // ====== API Key Management ======
  const KEY_STORAGE = 'amap_key';

  function getKey() {
    return localStorage.getItem(KEY_STORAGE) || '';
  }

  function saveKey(key) {
    localStorage.setItem(KEY_STORAGE, key.trim());
  }

  function updateSDKKey() {
    // Reload the page with the new key by reloading Amap SDK script
    // For simplicity: store key and reload
    const scripts = document.querySelectorAll('script');
    scripts.forEach(s => {
      if (s.src && s.src.includes('webapi.amap.com')) {
        const newSrc = s.src.replace(/key=[^&]*/, 'key=' + getKey());
        s.src = newSrc;
      }
    });
  }
</script>
```

Wait — the SDK loads at page start. We need a different approach. The Amap SDK script with `?key=PLACEHOLDER_KEY` is in the HTML. The user must paste their key BEFORE any API calls. Let's do it properly:

```javascript
<script>
  // ====== API Key Management ======
  const KEY_STORAGE = 'amap_key';

  function getKey() {
    return localStorage.getItem(KEY_STORAGE) || '';
  }

  function saveKey(key) {
    localStorage.setItem(KEY_STORAGE, key.trim());
  }

  // Show/hide key banner and modal
  function checkKey() {
    const key = getKey();
    const banner = document.getElementById('key-banner');
    if (!key) {
      banner.classList.remove('hidden');
      document.getElementById('btn-pick').disabled = true;
    } else {
      banner.classList.add('hidden');
    }
  }

  // Modal controls
  document.getElementById('key-link').addEventListener('click', (e) => {
    e.preventDefault();
    document.getElementById('key-modal').classList.remove('hidden');
    document.getElementById('key-input').value = getKey();
  });

  document.getElementById('key-cancel').addEventListener('click', () => {
    document.getElementById('key-modal').classList.add('hidden');
  });

  document.getElementById('key-save').addEventListener('click', () => {
    const input = document.getElementById('key-input').value.trim();
    if (!input) {
      alert('请输入有效的 API Key');
      return;
    }
    saveKey(input);
    document.getElementById('key-modal').classList.add('hidden');
    // Reload to apply new key to Amap SDK
    window.location.reload();
  });

  // Run on load
  checkKey();
</script>
```

Note: This script goes BEFORE the Amap SDK `<script>` tag. Move the SDK tag to after this script, so we can dynamically set the key. Actually, to keep it simple — keep the SDK script where it is, but we'll construct its URL dynamically in the JS. Let's restructure:

**Revised approach for `index.html`:** Remove the hardcoded Amap SDK `<script>` tag from the HTML. Instead, dynamically inject it via JS after reading the key from localStorage. This way the key is always correct.

In the `<script>` block (which should be the ONLY script block, placed at end of `<body>`):

```javascript
  // ====== API Key ======
  const KEY_STORAGE = 'amap_key';

  function getKey() { return localStorage.getItem(KEY_STORAGE) || ''; }
  function saveKey(k) { localStorage.setItem(KEY_STORAGE, k.trim()); }

  function checkKey() {
    const key = getKey();
    const banner = document.getElementById('key-banner');
    if (!key) {
      banner.classList.remove('hidden');
      document.getElementById('btn-pick').disabled = true;
      return false;
    }
    banner.classList.add('hidden');
    return true;
  }

  // Modal
  document.getElementById('key-link').addEventListener('click', (e) => {
    e.preventDefault();
    document.getElementById('key-modal').classList.remove('hidden');
    document.getElementById('key-input').value = getKey();
  });
  document.getElementById('key-cancel').addEventListener('click', () => {
    document.getElementById('key-modal').classList.add('hidden');
  });
  document.getElementById('key-save').addEventListener('click', () => {
    const input = document.getElementById('key-input').value.trim();
    if (!input) { alert('请输入有效的 API Key'); return; }
    saveKey(input);
    window.location.reload();
  });

  checkKey();

  // ====== Load Amap SDK dynamically ======
  function loadAmapSDK(callback) {
    const key = getKey();
    if (!key) {
      document.getElementById('key-banner').classList.remove('hidden');
      return;
    }
    const script = document.createElement('script');
    script.src = `https://webapi.amap.com/maps?v=2.0&key=${key}&plugin=AMap.PlaceSearch,AMap.Geolocation`;
    script.onload = callback;
    script.onerror = () => {
      showMessage('高德地图加载失败，请检查 Key 是否正确', 'error');
    };
    document.head.appendChild(script);
  }
</script>
```

Also: **remove** the hardcoded Amap SDK `<script>` tag from the `<body>` (the one with `PLACEHOLDER_KEY`).

- [ ] **Step 2: Verify key flow**

Open `index.html` in browser:
1. Red banner should show: "⚠️ 请先配置高德地图 API Key（点击设置）"
2. Click "点击设置" → modal opens
3. Enter a wrong key, save → page reloads, Amap SDK loads
4. Enter valid key, save → page reloads, SDK loads successfully (check DevTools Network tab for 200 on the SDK)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add API Key management with modal and localStorage"
```

---

### Task 4: Geolocation — browser GPS + Amap fallback

**Files:**
- Modify: `index.html` — add to `<script>` block

- [ ] **Step 1: Write the geolocation logic**

Add this code after the `loadAmapSDK` function:

```javascript
  // ====== Geolocation ======
  let currentPosition = null;    // {lng, lat}
  let currentAddress = '';       // "XX区XX路"

  function updateLocationUI() {
    const el = document.getElementById('loc-text');
    if (currentAddress) {
      el.textContent = '当前位置: ' + currentAddress;
    } else if (currentPosition) {
      el.textContent = `已定位 (${currentPosition.lng.toFixed(4)}, ${currentPosition.lat.toFixed(4)})`;
    }
    // Enable pick button once we have position
    if (currentPosition) {
      document.getElementById('btn-pick').disabled = false;
    }
  }

  function getBrowserLocation() {
    return new Promise((resolve, reject) => {
      if (!navigator.geolocation) {
        reject(new Error('浏览器不支持定位'));
        return;
      }
      navigator.geolocation.getCurrentPosition(
        (pos) => {
          resolve({
            lng: pos.coords.longitude,
            lat: pos.coords.latitude,
          });
        },
        (err) => {
          reject(new Error('定位失败: ' + err.message));
        },
        { enableHighAccuracy: true, timeout: 10000, maximumAge: 300000 }
      );
    });
  }

  function getAmapLocation() {
    return new Promise((resolve, reject) => {
      if (!window.AMap) {
        reject(new Error('高德地图未加载'));
        return;
      }
      AMap.plugin('AMap.Geolocation', () => {
        const geolocation = new AMap.Geolocation({ enableHighAccuracy: true, timeout: 10000 });
        geolocation.getCurrentPosition((status, result) => {
          if (status === 'complete' && result.position) {
            resolve({
              lng: result.position.lng,
              lat: result.position.lat,
              address: result.formattedAddress || '',
            });
          } else {
            reject(new Error('高德定位失败'));
          }
        });
      });
    });
  }

  async function locate() {
    document.getElementById('loc-text').textContent = '正在获取位置...';
    document.getElementById('loc-retry').classList.add('hidden');

    try {
      // Try browser GPS first
      const pos = await getBrowserLocation();
      currentPosition = { lng: pos.lng, lat: pos.lat };
      currentAddress = '';
      updateLocationUI();
      // Then try Amap for reverse geocode (address)
      if (window.AMap) {
        try {
          const amapResult = await getAmapLocation();
          currentAddress = amapResult.address || '';
        } catch (e) {
          // Amap geocode failed, that's ok - we still have coordinates
          console.warn('Amap reverse geocode failed:', e);
        }
      }
      updateLocationUI();
      document.getElementById('manual-loc').classList.add('hidden');
    } catch (e) {
      // GPS failed, try Amap IP-based
      document.getElementById('loc-text').textContent = 'GPS 定位失败，试试手动输入...';
      document.getElementById('manual-loc').classList.remove('hidden');
    }
  }

  // Retry button
  document.getElementById('loc-retry').addEventListener('click', () => {
    locate();
  });

  // Manual address search via Amap geocoding
  document.getElementById('manual-search').addEventListener('click', () => {
    const addr = document.getElementById('manual-address').value.trim();
    if (!addr) return;
    if (!window.AMap) {
      showMessage('高德地图未加载，请先配置 API Key', 'error');
      return;
    }
    AMap.plugin('AMap.Geocoder', () => {
      const geocoder = new AMap.Geocoder();
      geocoder.getLocation(addr, (status, result) => {
        if (status === 'complete' && result.info === 'OK') {
          const geocodes = result.geocodes;
          if (geocodes.length > 0) {
            currentPosition = {
              lng: geocodes[0].location.lng,
              lat: geocodes[0].location.lat,
            };
            currentAddress = geocodes[0].formattedAddress || addr;
            updateLocationUI();
            document.getElementById('manual-loc').classList.add('hidden');
            document.getElementById('loc-retry').classList.remove('hidden');
          } else {
            showMessage('找不到这个地址，请换个关键词试试', 'error');
          }
        } else {
          showMessage('地址搜索失败，请重试', 'error');
        }
      });
    });
  });
```

- [ ] **Step 2: Update `loadAmapSDK` callback**

Replace the `loadAmapSDK` function to call `locate()` when SDK loads:

```javascript
  function loadAmapSDK(callback) {
    const key = getKey();
    if (!key) {
      document.getElementById('key-banner').classList.remove('hidden');
      return;
    }
    const script = document.createElement('script');
    script.src = `https://webapi.amap.com/maps?v=2.0&key=${key}&plugin=AMap.PlaceSearch,AMap.Geolocation,AMap.Geocoder`;
    script.onload = () => {
      checkKey();
      locate();  // <-- auto-locate when SDK is ready
      if (callback) callback();
    };
    script.onerror = () => {
      showMessage('高德地图加载失败，请检查 Key 是否正确', 'error');
    };
    document.head.appendChild(script);
  }

  // Bootstrap
  loadAmapSDK(() => {
    console.log('Amap SDK ready');
  });
```

Note: Also add `AMap.Geocoder` to the plugin list in `script.src` (already added above).

- [ ] **Step 3: Verify geolocation**

Open `index.html` in browser:
1. Browser should request GPS permission → allow
2. Location text should update to show coordinates or address
3. "帮我决定" button should enable (turn orange from gray)
4. Test with GPS denied: manual address input should appear
5. Enter "北京市朝阳区" in manual input, click "搜索" → location should update

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add geolocation with browser GPS and Amap fallback"
```

---

### Task 5: PlaceSearch — search nearby restaurants

**Files:**
- Modify: `index.html` — add search logic to `<script>` block

- [ ] **Step 1: Write the search function**

Add after geolocation code:

```javascript
  // ====== Restaurant Search ======
  let searchResults = [];   // Full results from Amap
  let filteredResults = []; // After budget filter
  let seenIndices = [];     // Track which ones user has seen this session

  function searchNearby(params) {
    return new Promise((resolve, reject) => {
      if (!window.AMap) {
        reject(new Error('高德地图未加载'));
        return;
      }
      AMap.plugin('AMap.PlaceSearch', () => {
        const placeSearch = new AMap.PlaceSearch({
          pageSize: 25,
          pageIndex: 1,
          citylimit: false,
          extensions: 'all', // Request extended info including deep_info for avg cost
        });

        placeSearch.searchNearBy(
          '餐饮',                    // keywords
          [params.center.lng, params.center.lat],
          params.radius,
          (status, result) => {
            if (status === 'complete' && result.info === 'OK') {
              const pois = result.poiList.pois;
              const restaurants = pois.map(poi => ({
                id: poi.id,
                name: poi.name,
                address: poi.paddress || poi.address || '',
                location: { lng: poi.location.lng, lat: poi.location.lat },
                distance: poi.distance || 0,
                rating: poi.biz_ext?.rating ? parseFloat(poi.biz_ext.rating) : null,
                avgCost: parseAvgCost(poi),
                type: poi.type || '',
              }));
              resolve(restaurants);
            } else {
              reject(new Error('搜索失败: ' + (result.info || '未知错误')));
            }
          }
        );
      });
    });
  }

  function parseAvgCost(poi) {
    // Try multiple fields where avg cost might live
    if (poi.biz_ext && poi.biz_ext.cost) {
      const cost = parseFloat(poi.biz_ext.cost);
      if (cost > 0) return cost;
    }
    if (poi.deep_info && poi.deep_info.avg_cost) {
      const cost = parseFloat(poi.deep_info.avg_cost);
      if (cost > 0) return cost;
    }
    // If we have a price range string like "¥30-50"
    if (poi.biz_ext && poi.biz_ext.cost_range) {
      const m = poi.biz_ext.cost_range.match(/(\d+)/);
      if (m) return parseFloat(m[1]);
    }
    return null; // Unknown
  }

  function filterByBudget(restaurants, minBudget, maxBudget) {
    return restaurants.filter(r => {
      if (r.avgCost === null) return true; // Include unknown-cost restaurants
      return r.avgCost >= minBudget && r.avgCost <= maxBudget;
    });
  }
```

- [ ] **Step 2: Wire up the "帮我决定" button**

Add button click handler:

```javascript
  // ====== Main Action: Pick Restaurant ======
  document.getElementById('btn-pick').addEventListener('click', async () => {
    if (!currentPosition) {
      showMessage('请先获取位置', 'error');
      return;
    }

    const radius = parseInt(document.getElementById('radius-select').value);
    const minBudget = parseInt(document.getElementById('budget-min').value) || 0;
    const maxBudget = parseInt(document.getElementById('budget-max').value) || 9999;

    // Show loading
    showLoading(true);
    hideResult();
    hideMessage();

    try {
      const results = await searchNearby({
        center: currentPosition,
        radius: radius,
      });

      searchResults = results;

      if (results.length === 0) {
        showMessage('这个范围内没有搜到餐厅，试试扩大搜索范围', 'info');
        showLoading(false);
        return;
      }

      // Filter by budget
      filteredResults = filterByBudget(results, minBudget, maxBudget);

      if (filteredResults.length === 0) {
        const withCost = results.filter(r => r.avgCost !== null);
        if (withCost.length > 0) {
          const minCost = Math.min(...withCost.map(r => r.avgCost));
          const maxCost = Math.max(...withCost.map(r => r.avgCost));
          showMessage(
            `预算区间内没有餐厅。附近均价范围约 ¥${minCost}-${maxCost}，试试调整预算`,
            'info'
          );
        } else {
          showMessage('搜到了 ' + results.length + ' 家餐厅，但价格信息不足，试试放宽条件', 'info');
        }
        showLoading(false);
        return;
      }

      // Random pick
      seenIndices = [];
      const idx = Math.floor(Math.random() * filteredResults.length);
      seenIndices.push(idx);
      showResult(filteredResults[idx]);
      showLoading(false);

    } catch (err) {
      showMessage('搜索出错了: ' + err.message, 'error');
      showLoading(false);
    }
  });
```

- [ ] **Step 3: Add helper functions for UI state**

```javascript
  function showLoading(show) {
    document.getElementById('loading').classList.toggle('hidden', !show);
  }

  function hideResult() {
    document.getElementById('result-section').classList.add('hidden');
  }

  function hideMessage() {
    document.getElementById('message').classList.add('hidden');
  }

  function showMessage(msg, type) {
    const el = document.getElementById('message');
    el.textContent = msg;
    el.className = type; // 'error' or 'info'
    el.classList.remove('hidden');
  }
```

- [ ] **Step 4: Verify search**

1. Ensure Amap Key is configured
2. Allow GPS, set radius to 1000m, budget ¥10-¥50
3. Click "帮我决定"
4. Should see loading spinner briefly, then result card
5. Check DevTools Console for the search results array

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Amap PlaceSearch and budget filtering"
```

---

### Task 6: Result card — display + interactions

**Files:**
- Modify: `index.html` — add `showResult()` and button handlers

- [ ] **Step 1: Write the result display function**

```javascript
  // ====== Result Display ======
  const FOOD_ICONS = ['🍜', '🍕', '🍔', '🍣', '🥘', '🍲', '🫕', '🥟', '🍱', '🥗', '🌮', '🍗', '🥩', '🍝'];

  function getRandomIcon() {
    return FOOD_ICONS[Math.floor(Math.random() * FOOD_ICONS.length)];
  }

  function showResult(restaurant) {
    document.getElementById('result-icon').textContent = getRandomIcon();
    document.getElementById('result-name').textContent = restaurant.name;

    if (restaurant.rating) {
      const stars = '★'.repeat(Math.round(restaurant.rating)) + '☆'.repeat(5 - Math.round(restaurant.rating));
      document.getElementById('result-rating').textContent = stars + ' ' + restaurant.rating.toFixed(1);
    } else {
      document.getElementById('result-rating').textContent = '';
    }

    if (restaurant.avgCost) {
      document.getElementById('result-cost').textContent = '人均 ¥' + restaurant.avgCost + '/人';
    } else {
      document.getElementById('result-cost').textContent = '人均 -';
    }

    document.getElementById('result-distance').textContent =
      restaurant.distance > 1000
        ? (restaurant.distance / 1000).toFixed(1) + 'km'
        : restaurant.distance + 'm';

    document.getElementById('result-address').textContent = restaurant.address || '地址未提供';

    // Store current shown restaurant
    document.getElementById('result-card').dataset.restaurantId = restaurant.id;
    document.getElementById('result-card').dataset.restaurantLng = restaurant.location.lng;
    document.getElementById('result-card').dataset.restaurantLat = restaurant.location.lat;

    document.getElementById('result-section').classList.remove('hidden');
    // Scroll to result
    document.getElementById('result-section').scrollIntoView({ behavior: 'smooth' });
  }

  // ====== "换一个" Button ======
  document.getElementById('btn-reroll').addEventListener('click', () => {
    if (filteredResults.length === 0) return;

    // If all have been seen, reset
    if (seenIndices.length >= filteredResults.length) {
      seenIndices = [];
      // Brief flash message
      const btn = document.getElementById('btn-reroll');
      btn.textContent = '🔄 已经看过一轮，重新来';
      setTimeout(() => { btn.textContent = '🔄 换一个'; }, 1500);
      return;
    }

    // Pick a random one not yet seen
    const available = [];
    for (let i = 0; i < filteredResults.length; i++) {
      if (!seenIndices.includes(i)) available.push(i);
    }

    const idx = available[Math.floor(Math.random() * available.length)];
    seenIndices.push(idx);
    showResult(filteredResults[idx]);
  });

  // ====== "就它了" Button ======
  document.getElementById('btn-confirm').addEventListener('click', () => {
    const card = document.getElementById('result-card');
    const name = document.getElementById('result-name').textContent;
    const costText = document.getElementById('result-cost').textContent;

    // Save to history
    addHistory({
      name: name,
      date: new Date().toISOString(),
      avgCost: costText,
    });

    // Visual feedback
    document.getElementById('btn-confirm').textContent = '✅ 已记录！';
    document.getElementById('btn-confirm').style.background = '#388E3C';
    document.getElementById('result-card').style.borderLeftColor = 'var(--green)';

    setTimeout(() => {
      document.getElementById('btn-confirm').textContent = '✅ 就它了';
      document.getElementById('btn-confirm').style.background = 'var(--green)';
    }, 2000);
  });
```

- [ ] **Step 2: Verify interactions**

1. Click "帮我决定" → result card appears with restaurant name, rating, cost, distance
2. Click "换一个" → new random restaurant shows, previous one not repeated
3. Click "换一个" repeatedly → cycle through all, get "已经看过一轮" message
4. Click "就它了" → button turns dark green briefly, border turns green

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add result card display with reroll and confirm buttons"
```

---

### Task 7: History — localStorage persistence

**Files:**
- Modify: `index.html` — add history functions

- [ ] **Step 1: Write history logic**

```javascript
  // ====== History ======
  const HISTORY_KEY = 'food_picker_history';
  const MAX_HISTORY = 10;

  function loadHistory() {
    try {
      return JSON.parse(localStorage.getItem(HISTORY_KEY)) || [];
    } catch {
      return [];
    }
  }

  function saveHistory(items) {
    localStorage.setItem(HISTORY_KEY, JSON.stringify(items));
  }

  function addHistory(item) {
    const items = loadHistory();
    items.unshift(item);
    if (items.length > MAX_HISTORY) items.pop();
    saveHistory(items);
    renderHistory();
  }

  function renderHistory() {
    const container = document.getElementById('history-list');
    const items = loadHistory();
    if (items.length === 0) {
      container.innerHTML = '<span style="color:#aaa;font-size:12px;">还没有记录，去选一家吧</span>';
      return;
    }
    container.innerHTML = items
      .map(item => {
        const date = new Date(item.date);
        const dateStr = `${date.getMonth() + 1}/${date.getDate()}`;
        return `<span class="history-item" title="${dateStr}">🍜 ${item.name}</span>`;
      })
      .join('');
  }

  // Call on page load
  renderHistory();
```

- [ ] **Step 2: Verify history**

1. Click "帮我决定" → tap "就它了" → history section at bottom shows the restaurant
2. Refresh page → history persists
3. Add 11 items → only 10 show (oldest removed)
4. Clear localStorage via DevTools → history shows empty message

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add localStorage history for confirmed picks"
```

---

### Task 8: Edge cases & error handling cleanup

**Files:**
- Modify: `index.html` — consolidate, ensure all edge cases covered

- [ ] **Step 1: Review and fill gaps**

Walk through the edge case table from the spec and verify each one is handled. Check the full `<script>` block is complete and consistent.

Key checks:
1. **GPS denied** → `locate()` catch block shows manual address input ✅
2. **No restaurants in radius** → `searchNearby` returns empty → message shown ✅
3. **No results in budget** → filter empty → message with price range hint ✅
4. **Amap API fails** → `searchNearby` rejects → catch block shows error ✅
5. **No API Key** → `checkKey()` shows banner, `loadAmapSDK` early returns ✅
6. **Browser lacks Geolocation** → `navigator.geolocation` check → reject → falls through to manual ✅
7. **Desktop layout** → CSS `max-width: 420px; margin: 0 auto` centers content ✅

- [ ] **Step 2: Add the "导航去" button to the result card HTML**

Update the result card HTML (add a 3rd button):

```html
<div class="result-actions">
  <button id="btn-reroll" class="btn-secondary">🔄 换一个</button>
  <button id="btn-nav" class="btn-secondary">🗺️ 导航去</button>
  <button id="btn-confirm" class="btn-confirm">✅ 就它了</button>
</div>
```

And add its JS handler:

```javascript
  // ====== "导航去" Button ======
  document.getElementById('btn-nav').addEventListener('click', () => {
    const card = document.getElementById('result-card');
    const lng = card.dataset.restaurantLng;
    const lat = card.dataset.restaurantLat;
    if (!lng || !lat) return;

    // Open in Amap app or web
    const name = document.getElementById('result-name').textContent;
    const url = `https://uri.amap.com/navigation?to=${lng},${lat},${encodeURIComponent(name)}&mode=walk&callnative=1`;
    window.open(url, '_blank');
  });
```

- [ ] **Step 3: Full end-to-end manual test**

1. Open HTML in browser, no key → banner shows
2. Enter valid Amap key → page reloads, banner gone, auto-locates
3. Allow GPS → location shows
4. Set radius 1000m, budget ¥10-¥50
5. Tap "帮我决定" → spinner → result appears
6. Tap "换一个" → different restaurant
7. Tap "导航去" → Amap navigation opens in new tab
8. Tap "就它了" → saved to history
9. Deny GPS on next load → manual address input appears → enter address → location works
10. Set budget ¥1000-¥2000 (unlikely range) → appropriate message

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add navigation button and edge case polish"
```

---

### Task 9: PWA setup — manifest + service worker (optional, for "Add to Home Screen")

**Files:**
- Create: `manifest.json`

- [ ] **Step 1: Add web app manifest**

Create `manifest.json`:

```json
{
  "name": "您吃了吗？",
  "short_name": "吃了吗",
  "description": "随机帮你决定今天吃什么",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#FFF8F0",
  "theme_color": "#FF6B35",
  "icons": [
    {
      "src": "data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🍜</text></svg>",
      "sizes": "100x100",
      "type": "image/svg+xml"
    }
  ]
}
```

Add to `<head>`:

```html
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#FF6B35">
```

- [ ] **Step 2: Verify PWA**

1. Open in Chrome on Android
2. Chrome should show "Add to Home Screen" banner (or find in menu)
3. Add → opens as standalone app with orange theme color

- [ ] **Step 3: Commit**

```bash
git add manifest.json index.html
git commit -m "feat: add PWA manifest for Add to Home Screen"
```

---

### Task 10: Final review — self-check against spec

- [ ] **Step 1: Spec coverage check**

| Spec Requirement | Covered By |
|-----------------|------------|
| Single HTML file | Task 1 |
| Mobile-first layout, max 420px | Task 2 |
| Warm orange theme | Task 2 |
| GPS location | Task 4 |
| Amap PlaceSearch | Task 5 |
| Budget filtering | Task 5 |
| Random pick | Task 5 |
| Result card display | Task 6 |
| "换一个" reroll | Task 6 |
| "就它了" confirm | Task 6 |
| "导航去" navigation | Task 8 |
| localStorage history | Task 7 |
| API Key management | Task 3 |
| GPS denied fallback | Task 4 |
| No results handling | Task 5 |
| PWA home screen | Task 9 |

All spec points covered. No placeholders, no TODOs.

- [ ] **Step 2: Final commit**

```bash
git add -A
git commit -m "chore: finalize implementation, all spec requirements met"
```
