# 黑名单功能 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add restaurant blacklist — users can ban specific restaurants from appearing in search results, with a management modal for batch edits.

**Architecture:** Single `index.html` file modified. New data stored in localStorage (`food_picker_blacklist`). New modal UI added to HTML. Filter logic inserted between search and budget filter. All changes additive, no existing behavior altered.

**Tech Stack:** HTML5, CSS3, vanilla JS, localStorage

---

### Task 1: Blacklist modal HTML markup

**Files:**
- Modify: `index.html` — add modal after result card section

- [ ] **Step 1: Add blacklist modal HTML**

Insert this block right before the closing `</main>` tag (after the `#message` div, before history section):

```html
    <!-- Blacklist modal -->
    <div id="blacklist-modal" class="hidden">
      <div class="bl-overlay" id="bl-overlay"></div>
      <div class="bl-panel">
        <div class="bl-header">
          <span>🚫 黑名单</span>
          <button id="bl-edit-toggle" class="bl-edit-btn">编辑</button>
        </div>
        <div class="bl-list" id="bl-list">
          <span class="bl-empty">还没有拉黑任何店铺</span>
        </div>
        <div class="bl-footer">
          <button id="bl-delete-selected" class="hidden">🗑 删除选中</button>
          <button id="bl-clear-all">清除全部</button>
          <button id="bl-close">关闭</button>
        </div>
      </div>
    </div>
```

- [ ] **Step 2: Verify HTML structure**

Open `index.html` in browser, inspect DevTools → Elements. The `#blacklist-modal` should exist but be hidden (class `hidden`).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add blacklist modal HTML structure"
```

---

### Task 2: Add "黑名单" button and "👎 拉黑" button to HTML

**Files:**
- Modify: `index.html` — add buttons to history section and result card

- [ ] **Step 1: Add "黑名单" button next to history heading**

Change the history section heading from:

```html
      <h4>📋 最近选择
        <button id="btn-clear-history" class="btn-clear-history">🗑 清空</button>
      </h4>
```

To:

```html
      <h4>📋 最近选择
        <button id="btn-clear-history" class="btn-clear-history">🗑 清空</button>
        <button id="btn-open-blacklist" class="btn-blacklist-open">🚫 黑名单</button>
      </h4>
```

- [ ] **Step 2: Add "👎 拉黑" button to result card actions**

Change the result-actions div from:

```html
        <div class="result-actions">
          <button id="btn-reroll" class="btn-secondary">🔄 换一个</button>
          <button id="btn-nav" class="btn-secondary">🗺️ 导航去</button>
          <button id="btn-confirm" class="btn-confirm">✅ 就它了</button>
        </div>
```

To:

```html
        <div class="result-actions">
          <button id="btn-reroll" class="btn-secondary">🔄 换一个</button>
          <button id="btn-nav" class="btn-secondary">🗺️ 导航去</button>
          <button id="btn-ban" class="btn-ban">👎 拉黑</button>
          <button id="btn-confirm" class="btn-confirm">✅ 就它了</button>
        </div>
```

Now there are 4 buttons. Update the `.result-actions button` CSS to adjust for 4 items.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add blacklist open button and ban button to result card"
```

---

### Task 3: CSS for blacklist modal, buttons, and edit mode

**Files:**
- Modify: `index.html` — add styles to `<style>` block

- [ ] **Step 1: Add all new CSS**

Insert these styles before the `/* ====== Utility ====== */` comment:

```css
    /* ====== Blacklist ====== */
    .btn-blacklist-open {
      font-size: 11px; padding: 2px 10px;
      border: 1px solid #d0c0c0; border-radius: 12px;
      background: transparent; color: #999; cursor: pointer;
      font-family: inherit; line-height: 1.6;
    }
    .btn-blacklist-open:active { background: #fff0f0; border-color: var(--red); color: var(--red); }

    #btn-ban {
      background: #F5F5F5; color: #999; flex: 1; font-size: 12px;
    }
    #btn-ban:active { background: #FFE0E0; color: var(--red); }

    /* Blacklist modal */
    #blacklist-modal {
      position: fixed; top: 0; left: 0; right: 0; bottom: 0;
      z-index: 200; display: flex; align-items: center; justify-content: center;
    }
    .bl-overlay {
      position: absolute; top: 0; left: 0; right: 0; bottom: 0;
      background: rgba(0,0,0,0.4);
    }
    .bl-panel {
      position: relative; background: white; border-radius: var(--radius);
      width: 90%; max-width: 380px; max-height: 70vh; display: flex;
      flex-direction: column; overflow: hidden;
    }
    .bl-header {
      display: flex; align-items: center; justify-content: space-between;
      padding: 16px 18px 12px; font-size: 17px; font-weight: 700;
      border-bottom: 1px solid #eee;
    }
    .bl-edit-btn {
      font-size: 13px; padding: 4px 14px; border: 1px solid #ddd;
      border-radius: 14px; background: white; cursor: pointer;
      color: var(--primary); font-weight: 600;
    }
    .bl-edit-btn.active { background: var(--primary); color: white; border-color: var(--primary); }
    .bl-list {
      flex: 1; overflow-y: auto; padding: 8px 14px;
    }
    .bl-empty {
      display: block; text-align: center; color: var(--text-light);
      font-size: 14px; padding: 32px 0;
    }
    .bl-item {
      display: flex; align-items: center; gap: 10px;
      padding: 10px 4px; border-bottom: 1px solid #f5f5f5;
      font-size: 14px;
    }
    .bl-item .bl-checkbox {
      display: none; width: 18px; height: 18px; accent-color: var(--red); cursor: pointer; flex-shrink: 0;
    }
    .bl-item.edit-mode .bl-checkbox { display: block; }
    .bl-item .bl-name { flex: 1; }
    .bl-item .bl-date {
      font-size: 11px; color: var(--text-light); white-space: nowrap;
    }
    .bl-footer {
      display: flex; gap: 8px; padding: 12px 18px; border-top: 1px solid #eee;
      flex-wrap: wrap;
    }
    .bl-footer button {
      flex: 1; padding: 10px 0; border-radius: 8px;
      border: none; font-size: 14px; cursor: pointer;
    }
    #bl-delete-selected { background: #FFF0F0; color: var(--red); }
    #bl-clear-all { background: #FFE0E0; color: var(--red); }
    #bl-close { background: #eee; color: var(--text); }
```

- [ ] **Step 2: Update `.result-actions button` for 4 buttons**

Change the existing `.result-actions button` to reduce padding for 4-buttons layout:

```css
    .result-actions button {
      flex: 1; padding: 10px 0; border: none; border-radius: 8px;
      font-size: 13px; cursor: pointer; font-weight: 600;
    }
```

- [ ] **Step 3: Verify styles**

Open `index.html`. Inspect the result card action buttons — 4 buttons should fit in one row evenly. The blacklist modal is hidden initially, but inspectable in DevTools.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "style: add blacklist modal, ban button, and blacklist-open button styles"
```

---

### Task 4: Blacklist data layer (localStorage)

**Files:**
- Modify: `index.html` — add JS functions before the search section

- [ ] **Step 1: Add blacklist data functions**

Insert after the existing history functions, before `// ====== Restaurant Search ======`:

```javascript
    // ====== Blacklist ======
    var BLACKLIST_KEY = 'food_picker_blacklist';
    var MAX_BLACKLIST = 50;

    function loadBlacklist() {
      try { return JSON.parse(localStorage.getItem(BLACKLIST_KEY)) || []; }
      catch (e) { return []; }
    }

    function saveBlacklist(items) {
      localStorage.setItem(BLACKLIST_KEY, JSON.stringify(items));
    }

    function isBlacklisted(name) {
      var items = loadBlacklist();
      for (var i = 0; i < items.length; i++) {
        if (items[i].name === name) return true;
      }
      return false;
    }

    function addToBlacklist(name) {
      if (isBlacklisted(name)) return false;
      var items = loadBlacklist();
      if (items.length >= MAX_BLACKLIST) {
        alert('黑名单已满（最多50条），请先清理');
        return false;
      }
      items.unshift({ name: name, date: new Date().toISOString() });
      saveBlacklist(items);
      return true;
    }

    function removeFromBlacklist(names) {
      var items = loadBlacklist();
      items = items.filter(function(item) {
        return names.indexOf(item.name) === -1;
      });
      saveBlacklist(items);
    }

    function clearBlacklist() {
      localStorage.removeItem(BLACKLIST_KEY);
    }

    function excludeBlacklisted(restaurants) {
      var blacklist = loadBlacklist();
      if (blacklist.length === 0) return restaurants;
      var names = blacklist.map(function(item) { return item.name; });
      return restaurants.filter(function(r) {
        return names.indexOf(r.name) === -1;
      });
    }
```

- [ ] **Step 2: Verify in console**

Open `index.html` → F12 Console. Run:
```javascript
addToBlacklist('测试餐厅')
console.log(loadBlacklist())
// Expected: [{ name: '测试餐厅', date: '...' }]

clearBlacklist()
console.log(loadBlacklist())
// Expected: []
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add blacklist data layer (load/save/add/remove/clear/exclude)"
```

---

### Task 5: Integrate blacklist filter into search flow

**Files:**
- Modify: `index.html` — add filter call after `enrichWithCostData`

- [ ] **Step 1: Insert `excludeBlacklisted` into the pick handler**

Find this line:
```javascript
      searchResults = allResults;
```

And change to:
```javascript
      // Exclude blacklisted restaurants
      allResults = excludeBlacklisted(allResults);

      searchResults = allResults;
```

- [ ] **Step 2: Verify filter works**

1. Add a restaurant name to blacklist via console: `addToBlacklist('海底捞火锅')`
2. Search with keyword '火锅' — '海底捞火锅' should never appear in results
3. Clear: `clearBlacklist()`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: integrate blacklist filter into search flow"
```

---

### Task 6: Blacklist modal logic (open/close/edit/delete)

**Files:**
- Modify: `index.html` — add modal interaction functions

- [ ] **Step 1: Add modal management functions**

Insert after the blacklist data functions:

```javascript
    // ====== Blacklist Modal ======
    var blEditMode = false;

    function renderBlacklistModal() {
      var container = document.getElementById('bl-list');
      var items = loadBlacklist();
      var editToggle = document.getElementById('bl-edit-toggle');
      var delBtn = document.getElementById('bl-delete-selected');

      if (items.length === 0) {
        container.innerHTML = '<span class="bl-empty">还没有拉黑任何店铺</span>';
        editToggle.classList.add('hidden');
        delBtn.classList.add('hidden');
        return;
      }

      editToggle.classList.remove('hidden');

      var html = '';
      for (var i = 0; i < items.length; i++) {
        var d = new Date(items[i].date);
        var dateStr = (d.getMonth() + 1) + '/' + d.getDate();
        html += '<div class="bl-item' + (blEditMode ? ' edit-mode' : '') + '">';
        html += '<input type="checkbox" class="bl-checkbox" value="' + i + '">';
        html += '<span class="bl-name">' + items[i].name + '</span>';
        html += '<span class="bl-date">' + dateStr + '</span>';
        html += '</div>';
      }
      container.innerHTML = html;

      if (blEditMode) {
        delBtn.classList.remove('hidden');
        editToggle.textContent = '完成';
        editToggle.classList.add('active');
      } else {
        delBtn.classList.add('hidden');
        editToggle.textContent = '编辑';
        editToggle.classList.remove('active');
      }
    }

    function openBlacklistModal() {
      blEditMode = false;
      renderBlacklistModal();
      document.getElementById('blacklist-modal').classList.remove('hidden');
    }

    function closeBlacklistModal() {
      document.getElementById('blacklist-modal').classList.add('hidden');
    }

    document.getElementById('btn-open-blacklist').addEventListener('click', openBlacklistModal);
    document.getElementById('bl-close').addEventListener('click', closeBlacklistModal);
    document.getElementById('bl-overlay').addEventListener('click', closeBlacklistModal);

    // Edit mode toggle
    document.getElementById('bl-edit-toggle').addEventListener('click', function() {
      blEditMode = !blEditMode;
      renderBlacklistModal();
    });

    // Delete selected
    document.getElementById('bl-delete-selected').addEventListener('click', function() {
      var checks = document.querySelectorAll('.bl-checkbox:checked');
      if (checks.length === 0) {
        alert('请先勾选要删除的项');
        return;
      }
      var names = [];
      var items = loadBlacklist();
      for (var i = 0; i < checks.length; i++) {
        var idx = parseInt(checks[i].value);
        names.push(items[idx].name);
      }
      if (names.length === items.length) {
        if (!confirm('确定要清除全部黑名单吗？')) return;
      }
      removeFromBlacklist(names);
      renderBlacklistModal();
    });

    // Clear all
    document.getElementById('bl-clear-all').addEventListener('click', function() {
      if (!confirm('确定要清除全部黑名单记录吗？')) return;
      clearBlacklist();
      renderBlacklistModal();
    });
```

- [ ] **Step 2: Verify modal interactions**

1. Click "🚫 黑名单" → modal opens, shows empty state
2. Add items via console: `addToBlacklist('测试A'); addToBlacklist('测试B')`
3. Reopen modal → 2 items shown
4. Click "编辑" → checkboxes appear, "删除选中" button visible
5. Check one, click "删除选中" → that item removed
6. Click "清除全部" → confirm → all cleared
7. Click backdrop or "关闭" → modal closes

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add blacklist modal with edit mode and batch operations"
```

---

### Task 7: Ban button handler — add to blacklist + auto-reroll

**Files:**
- Modify: `index.html` — add event listener for btn-ban

- [ ] **Step 1: Add ban button handler**

Insert after the existing button handlers, before the history section:

```javascript
    // ====== "拉黑" Button ======
    document.getElementById('btn-ban').addEventListener('click', function() {
      var name = document.getElementById('result-name').textContent;
      if (isBlacklisted(name)) {
        alert('该店铺已在黑名单中');
        return;
      }
      if (!addToBlacklist(name)) return; // Failed (full)
      // Visual feedback
      var banBtn = document.getElementById('btn-ban');
      banBtn.textContent = '🚫 已拉黑';
      banBtn.style.background = '#FFE0E0';
      banBtn.style.color = 'var(--red)';
      setTimeout(function() {
        banBtn.textContent = '👎 拉黑';
        banBtn.style.background = '#F5F5F5';
        banBtn.style.color = '#999';
      }, 1500);
      // Auto-switch to next
      document.getElementById('btn-reroll').click();
    });
```

- [ ] **Step 2: Verify ban flow**

1. Search and get a result
2. Click "👎 拉黑" → button briefly shows "🚫 已拉黑" → auto-switches to next restaurant
3. Click "🚫 黑名单" button → modal shows the blacklisted restaurant
4. Search again → the blacklisted restaurant never appears

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add ban button handler with auto-reroll and visual feedback"
```

---

### Task 8: End-to-end verification

**Files:**
- Verify: `index.html` — full manual test

- [ ] **Step 1: Run full E2E test**

1. Configure API Key, allow GPS
2. Search with any radius/budget
3. Get a result → click "👎 拉黑" → auto-rerolls to new restaurant
4. Click "🚫 黑名单" → modal shows the banned restaurant with today's date
5. Click "编辑" → checkboxes appear → select one → "删除选中" → removed
6. Ban another restaurant → verify it's in modal
7. Click "清除全部" → confirm → modal shows empty state
8. Close modal → search again → previously banned restaurants don't appear
9. Fill blacklist to 50 items → 51st should trigger alert "黑名单已满"
10. Ban already-banned restaurant → alert "该店铺已在黑名单中"

- [ ] **Step 2: Final commit**

```bash
git add -A
git commit -m "chore: finalize blacklist feature, all spec requirements met"
```
