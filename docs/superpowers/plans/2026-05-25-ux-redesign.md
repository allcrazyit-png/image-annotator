# UX 改版：圖片標注工具直覺化 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 讓標注工具在任何工具模式下都能直接點選已有標注並修改，消除「需要切換工具才能選取」的痛點，並讓文字可以二次編輯。

**Architecture:** 所有改動都在 `index.html` 這個單一檔案內，分為 HTML 結構、CSS、JavaScript 三個區段。核心改動是在 `onDown()` 最前端加入通用命中偵測，讓繪圖工具也能觸發選取；`render()` 負責同步所有 UI 狀態到正確位置。

**Tech Stack:** 純 HTML/CSS/JavaScript，無框架，無 npm，無建置步驟。直接在瀏覽器開啟 `index.html` 驗證。

---

## 檔案變動清單

| 操作 | 路徑 | 說明 |
|------|------|------|
| Modify | `index.html` | 所有 9 個 task 均改這一個檔案 |

---

### Task 1：State 新增欄位 + 懸浮操作列 HTML/CSS

**Files:**
- Modify: `index.html`

- [ ] **Step 1：在 `S` 物件中加入 `editingIdx` 欄位**

找到 `const S = {` 區塊（約第 609 行），在 `initBox: null` 後新增：

```javascript
  initBox: null,
  editingIdx: -1        // -1 = 新增文字；>= 0 = 編輯第 N 個標注
```

- [ ] **Step 2：在 `#canvas-wrapper` 內加入懸浮操作列 DOM**

找到：
```html
      <!-- Floating delete button for selected element -->
      <button id="floating-delete-btn" ...>
```
在它**之前**插入：

```html
      <!-- Floating action bar (shown when annotation is selected) -->
      <div id="floating-action-bar" style="
        display:none; position:absolute; z-index:150;
        transform:translateX(-50%); pointer-events:all;
      ">
        <div style="
          background:#1c1c1e; border-radius:8px; padding:5px 10px;
          display:flex; gap:8px; box-shadow:0 3px 12px rgba(0,0,0,0.35);
          align-items:center;
        ">
          <button id="fab-edit" onclick="editSelectedText()" style="
            display:none; font-size:12px; color:white;
            background:rgba(59,130,246,0.7); border:none;
            border-radius:5px; padding:5px 10px; cursor:pointer; font-weight:600;
          ">✏ 編輯</button>
          <button id="fab-delete" onclick="deleteSelected()" style="
            font-size:12px; color:white;
            background:rgba(239,68,68,0.7); border:none;
            border-radius:5px; padding:5px 10px; cursor:pointer; font-weight:600;
          ">✕ 刪除</button>
        </div>
      </div>
```

- [ ] **Step 3：移除舊的 `#floating-delete-btn`**

找到並刪除整個：
```html
      <!-- Floating Delete Button -->
      <button id="floating-delete-btn" ...>
        <svg ...></svg>
      </button>
```

- [ ] **Step 4：移除右側面板中的 `selected-opts-sect`**

找到並刪除整個 `<div class="sect" id="selected-opts-sect" ...>` 區塊（含其內的按鈕）。

- [ ] **Step 5：在右側面板顏色區塊上方加入情境標籤**

找到：
```html
  <aside id="opts">
    <div class="sect">
      <div class="sect-label">顏色</div>
```
改為：
```html
  <aside id="opts">
    <div class="sect">
      <div id="panel-context-label" class="sect-label" style="color:var(--muted); margin-bottom:4px; font-size:9px;">下一個標注的設定</div>
      <div class="sect-label">顏色</div>
```

- [ ] **Step 6：在右側面板文字大小區塊後面加入「編輯文字」和「刪除」按鈕區塊**

找到：
```html
    <div class="sect">
      <div class="sect-label">草稿</div>
```
在它**之前**插入：

```html
    <div class="sect" id="panel-edit-sect" style="display:none;">
      <div class="sect-label">選取操作</div>
      <div class="btn-row">
        <button id="panel-edit-btn" class="btn-s prime" onclick="editSelectedText()" style="display:none;">✏ 編輯文字</button>
        <button class="btn-s red" onclick="deleteSelected()">✕ 刪除</button>
      </div>
    </div>
```

- [ ] **Step 7：在手機色條（`#mobile-opts`）最左側加入情境標籤和操作按鈕**

找到：
```html
<div id="mobile-opts">
  <div id="m-color-row" style="display:flex;gap:4px;flex-shrink:0;"></div>
```
改為：
```html
<div id="mobile-opts">
  <span id="m-ctx-label" style="font-size:9px;font-weight:700;text-transform:uppercase;letter-spacing:0.06em;color:#bbb;flex-shrink:0;white-space:nowrap;">下一個</span>
  <div id="m-color-row" style="display:flex;gap:4px;flex-shrink:0;"></div>
```

在 `#mobile-opts` 的關閉 `</div>` 前插入：

```html
  <button id="m-edit-btn" onclick="editSelectedText()" style="display:none;flex-shrink:0;padding:4px 8px;font-size:11px;background:#eff6ff;color:#2f6feb;border:1px solid #bfdbfe;border-radius:6px;cursor:pointer;white-space:nowrap;font-weight:600;">✏ 編輯</button>
  <button id="m-delete-btn" onclick="deleteSelected()" style="display:none;flex-shrink:0;padding:4px 8px;font-size:11px;background:#fef2f2;color:#dc2626;border:1px solid #fca5a5;border-radius:6px;cursor:pointer;white-space:nowrap;font-weight:600;">✕ 刪除</button>
```

- [ ] **Step 8：在瀏覽器開啟 `index.html` 確認頁面沒有 JS 錯誤**

用瀏覽器開啟 `index.html`，開 DevTools Console，確認無紅色錯誤。上傳任意圖片確認畫布正常顯示。

- [ ] **Step 9：Commit**

```bash
git add index.html
git commit -m "feat: add floating action bar DOM and context label elements"
```

---

### Task 2：重構 `onDown()` — 任何工具下均可點選標注

**Files:**
- Modify: `index.html` (JavaScript 區段的 `onDown` function)

- [ ] **Step 1：替換整個 `onDown` function**

找到 `function onDown(e) {` 到其對應的結尾 `}`，整段替換為：

```javascript
function onDown(e) {
  if (!S.img) return;
  const p = pt(e);

  // ── 文字工具：點選位置，開啟輸入彈窗 ──
  if (S.tool === 'text') {
    // 如果點到現有文字標注，改為編輯它
    const hitIdx = findHitAnnotation(p);
    if (hitIdx !== -1 && S.annotations[hitIdx].type === 'text') {
      selectAnnotation(hitIdx);
      editSelectedText();
      return;
    }
    S.pendTx = p.x; S.pendTy = p.y;
    S.editingIdx = -1;
    txtFld.value = '';
    txtModal.classList.add('show');
    setTimeout(() => txtFld.focus(), 60);
    return;
  }

  // ── 數字編號工具 ──
  if (S.tool === 'number') {
    pushUndo();
    S.annotations.push({ type:'num', x:p.x, y:p.y, n:S.numCount++, color:S.color, r:Math.max(16, S.strokeW * 5) });
    S.selectedIdx = S.annotations.length - 1;
    syncControlsFromSelected();
    render();
    return;
  }

  // ── 通用命中偵測（適用所有工具，含 select 和繪圖工具）──
  // 先檢查縮放把手（如果目前有選取）
  if (S.selectedIdx !== -1) {
    const box = getBoundingBox(S.annotations[S.selectedIdx]);
    if (box) {
      const handle = findHitHandle(p, box);
      if (handle) {
        S.isResizingSelected = true;
        S.activeHandle = handle;
        S.initAnno = JSON.parse(JSON.stringify(S.annotations[S.selectedIdx]));
        S.initBox = box;
        pushUndo();
        return;
      }
    }
  }

  // 檢查是否點到既有標注
  const hitIdx = findHitAnnotation(p);
  if (hitIdx !== -1) {
    selectAnnotation(hitIdx);
    S.isDraggingSelected = true;
    S.dragStartX = p.x;
    S.dragStartY = p.y;
    S.hasMovedSelected = false;
    canvas.style.cursor = 'move';
    return;
  }

  // 點到空白處：解除選取
  if (S.selectedIdx !== -1) {
    S.selectedIdx = -1;
    render();
  }

  // select 工具點到空白處就結束
  if (S.tool === 'select') return;

  // ── 開始繪圖 ──
  S.drawing = true;
  S.x0 = p.x; S.y0 = p.y;
  S.pts = [{ x:p.x, y:p.y }];
}
```

- [ ] **Step 2：加入 `selectAnnotation()` 輔助函式**

在 `findHitAnnotation` 函式**之前**加入：

```javascript
function selectAnnotation(idx) {
  S.selectedIdx = idx;
  syncControlsFromSelected();
  render();
}
```

- [ ] **Step 3：在瀏覽器驗證**

開啟 `index.html`，上傳圖片，選箭頭工具畫一個箭頭，然後：
- 不切換工具，直接點剛畫的箭頭 → 應該出現藍色選取框和懸浮操作列的刪除按鈕
- 點空白處 → 選取框消失
- 切換到圓圈工具，點既有箭頭 → 應該選取箭頭而非開始畫圓

- [ ] **Step 4：Commit**

```bash
git add index.html
git commit -m "feat: universal hit detection - select annotation with any tool"
```

---

### Task 3：畫完後自動選取 + `onMove` / `onUp` 整理

**Files:**
- Modify: `index.html` (JavaScript 的 `onUp` function)

- [ ] **Step 1：修改 `onUp` function，畫完後自動選取新標注**

找到 `function onUp(e) {`，將其中 `switch (S.tool) {` 區塊每個 `push` 後加入自動選取。整段替換為：

```javascript
function onUp(e) {
  if (S.tool === 'select' || S.isDraggingSelected || S.isResizingSelected) {
    S.isDraggingSelected = false;
    S.isResizingSelected = false;
    canvas.style.cursor = S.tool === 'select' ? 'default' : 'crosshair';
    return;
  }
  if (!S.drawing) return;
  S.drawing = false;
  const p = pt(e);
  const dx = p.x - S.x0, dy = p.y - S.y0;
  if (Math.hypot(dx, dy) < 3 && S.tool !== 'freehand' && S.tool !== 'highlight') { render(); return; }
  pushUndo();
  switch (S.tool) {
    case 'arrow':
      S.annotations.push({ type:'arrow', x1:S.x0, y1:S.y0, x2:p.x, y2:p.y, color:S.color, w:S.strokeW }); break;
    case 'circle':
      S.annotations.push({ type:'circle', x1:S.x0, y1:S.y0, x2:p.x, y2:p.y, color:S.color, w:S.strokeW }); break;
    case 'rect':
      S.annotations.push({ type:'rect', x1:S.x0, y1:S.y0, x2:p.x, y2:p.y, color:S.color, w:S.strokeW }); break;
    case 'freehand':
      if (S.pts.length > 1) S.annotations.push({ type:'free', pts:[...S.pts], color:S.color, w:S.strokeW }); break;
    case 'highlight':
      if (S.pts.length > 1) S.annotations.push({ type:'hi', pts:[...S.pts], color:S.color, w:Math.max(S.strokeW*6,24) }); break;
    case 'mosaic':
      { const mx=Math.min(S.x0,p.x), my=Math.min(S.y0,p.y), mw=Math.abs(dx), mh=Math.abs(dy);
        if (mw > 4 && mh > 4) S.annotations.push({ type:'mosaic', x:mx, y:my, w:mw, h:mh }); } break;
  }
  // 自動選取剛畫好的標注
  if (S.annotations.length > 0) {
    S.selectedIdx = S.annotations.length - 1;
    syncControlsFromSelected();
  }
  render();
}
```

- [ ] **Step 2：在瀏覽器驗證**

畫一個圓圈，鬆手後圓圈應立即出現藍色選取框，懸浮列出現刪除按鈕。再直接拖移剛畫的圓圈，應該可以移動（不需切換工具）。

- [ ] **Step 3：同步更新 `onMove()` — 拿掉 `S.tool === 'select'` 的限制**

在現有的 `onMove` 中，找到以下兩段並修改：

原本：
```javascript
  if (S.tool === 'select' && S.isResizingSelected && S.selectedIdx !== -1) {
```
改為：
```javascript
  if (S.isResizingSelected && S.selectedIdx !== -1) {
```

原本：
```javascript
  if (S.tool === 'select' && S.isDraggingSelected && S.selectedIdx !== -1) {
```
改為：
```javascript
  if (S.isDraggingSelected && S.selectedIdx !== -1) {
```

原本：
```javascript
  if (S.tool === 'select' && !S.isResizingSelected && !S.isDraggingSelected && S.selectedIdx !== -1) {
```
改為：
```javascript
  if (!S.isResizingSelected && !S.isDraggingSelected && S.selectedIdx !== -1) {
```

同一段最後的 `canvas.style.cursor = 'default';` 改為：
```javascript
    canvas.style.cursor = S.tool === 'select' ? 'default' : 'crosshair';
```

- [ ] **Step 4：Commit**

```bash
git add index.html
git commit -m "feat: auto-select annotation after drawing"
```

---

### Task 4：懸浮操作列定位與顯示邏輯

**Files:**
- Modify: `index.html` (JavaScript 的 `render` function)

- [ ] **Step 1：取得 `floatingActionBar` DOM reference**

在 `const toastEl = document.getElementById('toast');` 那行下面加入：

```javascript
const fabBar    = document.getElementById('floating-action-bar');
const fabEdit   = document.getElementById('fab-edit');
const fabDelete = document.getElementById('fab-delete');
```

- [ ] **Step 2：更新 `render()` 函式，移除舊的 floating-delete-btn 邏輯，加入新懸浮列邏輯**

找到 `render()` 函式中這段：

```javascript
    // Sync delete buttons
    document.getElementById('selected-opts-sect').style.display = 'block';
    document.getElementById('floating-delete-btn').style.display = 'flex';
  } else {
    document.getElementById('selected-opts-sect').style.display = 'none';
    document.getElementById('floating-delete-btn').style.display = 'none';
  }
```

替換為：

```javascript
    updateSelectionUI(box);
  } else {
    fabBar.style.display = 'none';
    document.getElementById('panel-edit-sect').style.display = 'none';
    document.getElementById('panel-context-label').textContent = '下一個標注的設定';
    document.getElementById('m-ctx-label').textContent = '下一個';
    document.getElementById('m-ctx-label').style.color = '#bbb';
    document.getElementById('m-edit-btn').style.display = 'none';
    document.getElementById('m-delete-btn').style.display = 'none';
  }
```

- [ ] **Step 3：加入 `updateSelectionUI()` 函式**

在 `render()` 函式**之後**加入：

```javascript
function updateSelectionUI(box) {
  const a = S.annotations[S.selectedIdx];
  if (!a) return;

  // 懸浮操作列定位
  const rect = canvas.getBoundingClientRect();
  const scaleX = rect.width / canvas.width;
  const scaleY = rect.height / canvas.height;
  const midX = ((box.minX + box.maxX) / 2) * scaleX + rect.left - wrapper.getBoundingClientRect().left;
  const topY  = box.minY * scaleY + rect.top - wrapper.getBoundingClientRect().top - 44;

  fabBar.style.left = midX + 'px';
  fabBar.style.top  = Math.max(4, topY) + 'px';
  fabBar.style.display = 'block';

  // 「編輯文字」按鈕只對 text 類型顯示
  const isText = (a.type === 'text');
  fabEdit.style.display = isText ? 'inline-block' : 'none';

  // 右側面板
  const panelEditSect = document.getElementById('panel-edit-sect');
  panelEditSect.style.display = 'block';
  document.getElementById('panel-edit-btn').style.display = isText ? 'block' : 'none';

  const typeNames = { arrow:'箭頭', text:'文字', circle:'圓圈', rect:'矩形', free:'手繪', hi:'螢光', num:'編號', mosaic:'馬賽克' };
  const typeName = typeNames[a.type] || a.type;
  document.getElementById('panel-context-label').textContent = `選取中：${typeName}`;

  // 手機色條
  const mCtxLabel = document.getElementById('m-ctx-label');
  mCtxLabel.textContent = '選取中';
  mCtxLabel.style.color = '#2f6feb';
  document.getElementById('m-edit-btn').style.display = isText ? 'inline-flex' : 'none';
  document.getElementById('m-delete-btn').style.display = 'inline-flex';
}
```

- [ ] **Step 4：在瀏覽器驗證**

1. 選取一個箭頭 → 懸浮列只有「✕ 刪除」，右側面板標籤改為「選取中：箭頭」，行動版色條出現「✕ 刪除」
2. 選取一個文字標注 → 懸浮列有「✏ 編輯」和「✕ 刪除」，行動版色條有「✏ 編輯」和「✕ 刪除」
3. 點空白處 → 所有按鈕消失，標籤回到「下一個標注的設定」

- [ ] **Step 5：Commit**

```bash
git add index.html
git commit -m "feat: floating action bar positioning and context panel sync"
```

---

### Task 5：文字二次編輯功能

**Files:**
- Modify: `index.html` (JavaScript 的 `confirmText` 和新增 `editSelectedText`)

- [ ] **Step 1：加入 `editSelectedText()` 函式**

在 `cancelText()` 函式**之後**加入：

```javascript
function editSelectedText() {
  if (S.selectedIdx === -1) return;
  const a = S.annotations[S.selectedIdx];
  if (a.type !== 'text') return;
  S.editingIdx = S.selectedIdx;
  txtFld.value = a.text;
  txtModal.classList.add('show');
  setTimeout(() => { txtFld.focus(); txtFld.select(); }, 60);
}
```

- [ ] **Step 2：修改 `confirmText()` 以支援編輯模式**

找到：
```javascript
function confirmText() {
  const txt = txtFld.value.trim();
  txtModal.classList.remove('show');
  if (!txt) return;
  pushUndo();
  S.annotations.push({ type:'text', x:S.pendTx, y:S.pendTy, text:txt, color:S.color, fs:S.fontSize });
  render();
}
```

替換為：
```javascript
function confirmText() {
  const txt = txtFld.value.trim();
  txtModal.classList.remove('show');
  if (!txt) return;
  pushUndo();
  if (S.editingIdx >= 0) {
    // 編輯既有文字
    S.annotations[S.editingIdx].text = txt;
    S.selectedIdx = S.editingIdx;
    S.editingIdx = -1;
  } else {
    // 新增文字
    S.annotations.push({ type:'text', x:S.pendTx, y:S.pendTy, text:txt, color:S.color, fs:S.fontSize });
    S.selectedIdx = S.annotations.length - 1;
    syncControlsFromSelected();
  }
  render();
}
```

- [ ] **Step 3：在瀏覽器驗證**

1. 新增一個文字標注「測試」
2. 在文字工具模式下直接點擊「測試」文字 → 彈窗應預填「測試」
3. 把文字改成「修改後」→ 點確定 → 畫布上的文字應變為「修改後」
4. 選取文字後，點右側面板「✏ 編輯文字」→ 同樣效果
5. 選取文字後，點懸浮列「✏ 編輯」→ 同樣效果

- [ ] **Step 4：Commit**

```bash
git add index.html
git commit -m "feat: text annotation in-place editing"
```

---

### Task 6：桌機雙擊文字標注觸發編輯

**Files:**
- Modify: `index.html` (JavaScript 的 `wireCanvas` function)

- [ ] **Step 1：在 `wireCanvas()` 中加入 dblclick 事件**

找到 `wireCanvas()` 函式，在最後一個 `canvas.addEventListener('touchcancel', ...)` 之後加入：

```javascript
  canvas.addEventListener('dblclick', e => {
    if (!S.img) return;
    const p = pt(e);
    const hitIdx = findHitAnnotation(p);
    if (hitIdx !== -1 && S.annotations[hitIdx].type === 'text') {
      selectAnnotation(hitIdx);
      editSelectedText();
    }
  });
```

- [ ] **Step 2：在瀏覽器驗證（桌機）**

雙擊畫布上的文字標注 → 彈窗應預填現有文字。確認雙擊空白處不觸發任何動作。

- [ ] **Step 3：Commit**

```bash
git add index.html
git commit -m "feat: double-click text annotation to edit on desktop"
```

---

### Task 7：行動版工具列 Tab 分頁

**Files:**
- Modify: `index.html` (HTML 的 `#mobile-bar` 區段 + CSS + JavaScript)

- [ ] **Step 1：加入 Tab 列 CSS**

在 `/* ── Responsive ── */` 之前加入：

```css
/* ── Mobile tab bar ── */
#mobile-tab-bar {
  display: none;
  background: var(--surface);
  border-top: 1px solid var(--border);
  flex-shrink: 0;
}
#mobile-tab-bar button {
  flex: 1;
  padding: 6px 8px;
  font-size: 12px;
  font-weight: 600;
  border: none;
  background: transparent;
  color: var(--muted);
  cursor: pointer;
  border-bottom: 2px solid transparent;
  transition: color var(--fast), border-color var(--fast);
}
#mobile-tab-bar button.m-tab-active {
  color: var(--accent);
  border-bottom-color: var(--accent);
}
@media (max-width: 640px) {
  #mobile-tab-bar { display: flex; }
}
```

- [ ] **Step 2：在 `#mobile-bar` 之前插入 Tab DOM**

找到 `<!-- Mobile toolbar -->` 註解，在 `<div id="mobile-bar">` **之前**插入：

```html
<!-- Mobile tool tab bar -->
<div id="mobile-tab-bar">
  <button id="tab-btn-common" class="m-tab-active" onclick="switchMobileTab('common')">常用工具</button>
  <button id="tab-btn-all" onclick="switchMobileTab('all')">全部工具</button>
</div>
```

- [ ] **Step 3：將 `#mobile-bar` 改為「常用工具」內容**

找到整個 `<div id="mobile-bar">` 的內容，替換為：

```html
<!-- Mobile toolbar -->
<div id="mobile-bar">
  <!-- 常用工具（預設） -->
  <div id="tab-common-content" style="display:flex;align-items:center;padding:0 8px;gap:6px;width:100%;overflow-x:auto;scrollbar-width:none;">
    <button class="m-btn" onclick="triggerUpload()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21,15 16,10 5,21"/></svg>
      上傳
    </button>
    <div class="m-sep"></div>
    <button class="m-btn active" data-tool="arrow" onclick="setTool('arrow')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="19" x2="19" y2="5"/><polyline points="9,5 19,5 19,15"/></svg>
      箭頭
    </button>
    <button class="m-btn" data-tool="text" onclick="setTool('text')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="4,7 4,4 20,4 20,7"/><line x1="9" y1="20" x2="15" y2="20"/><line x1="12" y1="4" x2="12" y2="20"/></svg>
      文字
    </button>
    <button class="m-btn" data-tool="circle" onclick="setTool('circle')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><ellipse cx="12" cy="12" rx="10" ry="7"/></svg>
      圓圈
    </button>
    <button class="m-btn" data-tool="number" onclick="setTool('number')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/></svg>
      編號
    </button>
    <div class="m-sep"></div>
    <button class="m-btn" onclick="undo()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="9,14 4,9 9,4"/><path d="M20 20v-7a4 4 0 0 0-4-4H4"/></svg>
      復原
    </button>
    <button class="m-btn" onclick="redo()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="15,14 20,9 15,4"/><path d="M4 20v-7a4 4 0 0 1 4-4h12"/></svg>
      重做
    </button>
    <div class="m-sep"></div>
    <button class="m-btn" onclick="doDownload()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7,10 12,15 17,10"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
      下載
    </button>
    <button class="m-btn" onclick="doShare()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"/><polyline points="16,6 12,2 8,6"/><line x1="12" y1="2" x2="12" y2="15"/></svg>
      分享
    </button>
  </div>
  <!-- 全部工具 -->
  <div id="tab-all-content" style="display:none;align-items:center;padding:0 8px;gap:6px;width:100%;overflow-x:auto;scrollbar-width:none;">
    <button class="m-btn" onclick="triggerUpload()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21,15 16,10 5,21"/></svg>
      上傳
    </button>
    <div class="m-sep"></div>
    <button class="m-btn" data-tool="select" onclick="setTool('select')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 3l7.07 16.97 2.51-7.39 7.39-2.51L3 3z"/><path d="M13 13l6 6"/></svg>
      選擇
    </button>
    <button class="m-btn" data-tool="arrow" onclick="setTool('arrow')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="19" x2="19" y2="5"/><polyline points="9,5 19,5 19,15"/></svg>
      箭頭
    </button>
    <button class="m-btn" data-tool="text" onclick="setTool('text')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="4,7 4,4 20,4 20,7"/><line x1="9" y1="20" x2="15" y2="20"/><line x1="12" y1="4" x2="12" y2="20"/></svg>
      文字
    </button>
    <button class="m-btn" data-tool="circle" onclick="setTool('circle')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><ellipse cx="12" cy="12" rx="10" ry="7"/></svg>
      橢圓
    </button>
    <button class="m-btn" data-tool="rect" onclick="setTool('rect')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="3" y="5" width="18" height="14" rx="1"/></svg>
      矩形
    </button>
    <button class="m-btn" data-tool="freehand" onclick="setTool('freehand')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M4 20 C6 16 8 13 11 14 S16 17 19 14 S22 8 21 6"/></svg>
      手繪
    </button>
    <button class="m-btn" data-tool="highlight" onclick="setTool('highlight')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M15 5L19 9L9 19L4 20L5 15Z"/></svg>
      螢光
    </button>
    <button class="m-btn" data-tool="number" onclick="setTool('number')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/></svg>
      編號
    </button>
    <button class="m-btn" data-tool="mosaic" onclick="setTool('mosaic')">
      <svg viewBox="0 0 24 24" fill="currentColor"><rect x="3" y="3" width="4" height="4" opacity=".4"/><rect x="10" y="3" width="4" height="4" opacity=".8"/><rect x="17" y="3" width="4" height="4" opacity=".4"/><rect x="3" y="10" width="4" height="4" opacity=".8"/><rect x="10" y="10" width="4" height="4"/><rect x="17" y="10" width="4" height="4" opacity=".8"/><rect x="3" y="17" width="4" height="4" opacity=".4"/><rect x="10" y="17" width="4" height="4" opacity=".8"/><rect x="17" y="17" width="4" height="4" opacity=".4"/></svg>
      馬賽克
    </button>
    <div class="m-sep"></div>
    <button class="m-btn" onclick="undo()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="9,14 4,9 9,4"/><path d="M20 20v-7a4 4 0 0 0-4-4H4"/></svg>
      復原
    </button>
    <button class="m-btn" onclick="redo()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="15,14 20,9 15,4"/><path d="M4 20v-7a4 4 0 0 1 4-4h12"/></svg>
      重做
    </button>
    <div class="m-sep"></div>
    <button class="m-btn" onclick="doDownload()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7,10 12,15 17,10"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
      下載
    </button>
    <button class="m-btn" onclick="doShare()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M4 12v8a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-8"/><polyline points="16,6 12,2 8,6"/><line x1="12" y1="2" x2="12" y2="15"/></svg>
      分享
    </button>
    <button class="m-btn" onclick="saveDraft()">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/><polyline points="17,21 17,13 7,13 7,21"/><polyline points="7,3 7,8 15,8"/></svg>
      草稿
    </button>
    <button class="m-btn" onclick="clearAll()" style="color:var(--danger);">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><polyline points="3,6 5,6 21,6"/><path d="M19 6l-1 14H6L5 6"/><path d="M10 11v6m4-6v6"/><path d="M9 6V4h6v2"/></svg>
      清除
    </button>
  </div>
</div>
```

- [ ] **Step 4：加入 `switchMobileTab()` 函式**

在 `showToast` 函式**之後**加入：

```javascript
function switchMobileTab(tab) {
  const common = document.getElementById('tab-common-content');
  const all    = document.getElementById('tab-all-content');
  const btnCommon = document.getElementById('tab-btn-common');
  const btnAll    = document.getElementById('tab-btn-all');
  if (tab === 'common') {
    common.style.display = 'flex';
    all.style.display = 'none';
    btnCommon.classList.add('m-tab-active');
    btnAll.classList.remove('m-tab-active');
  } else {
    common.style.display = 'none';
    all.style.display = 'flex';
    btnCommon.classList.remove('m-tab-active');
    btnAll.classList.add('m-tab-active');
  }
  try { localStorage.setItem('annotator-tab', tab); } catch {}
}
```

- [ ] **Step 5：在 `init()` 中讀取 tab 記憶**

找到：
```javascript
(function init() {
  buildPalette();
  wireControls();
  wireCanvas();
  canvas.style.display = 'none';
  checkDraftBanner();
})();
```
改為：
```javascript
(function init() {
  buildPalette();
  wireControls();
  wireCanvas();
  canvas.style.display = 'none';
  checkDraftBanner();
  try {
    const savedTab = localStorage.getItem('annotator-tab');
    if (savedTab === 'all') switchMobileTab('all');
  } catch {}
})();
```

- [ ] **Step 6：在手機瀏覽器驗證**

縮小瀏覽器視窗到 ≤640px，確認：
- 頂部出現「常用工具 / 全部工具」兩個 tab
- 常用工具 tab 只有 8 個按鈕（上傳、箭頭、文字、圓圈、編號、復原、重做、下載、分享）
- 全部工具 tab 有所有工具
- 切換 tab 後重整頁面，仍停在原本的 tab

- [ ] **Step 7：Commit**

```bash
git add index.html
git commit -m "feat: mobile toolbar tab split into common and all tools"
```

---

### Task 8：最終整合驗收

**Files:**
- Modify: `index.html` (版本號)

- [ ] **Step 1：逐一跑驗收清單**

開啟 `index.html`，上傳一張圖片，依序測試：

| # | 測試步驟 | 預期結果 |
|---|---------|---------|
| 1 | 選箭頭工具，畫一個箭頭，鬆手 | 箭頭立即出現藍色選取框和懸浮列「✕ 刪除」 |
| 2 | 不切換工具，直接拖移剛畫的箭頭 | 箭頭可以移動 |
| 3 | 選圓圈工具，點步驟1的箭頭 | 選取箭頭（不開始畫圓） |
| 4 | 選文字工具，點畫布空白處，輸入「測試」確定 | 文字放下後自動選取，懸浮列有「✏ 編輯」和「✕ 刪除」 |
| 5 | 點懸浮列「✏ 編輯」 | 彈窗預填「測試」 |
| 6 | 改成「修改後」→ 確定 | 畫布文字改為「修改後」 |
| 7 | 桌機上雙擊文字標注 | 彈窗預填現有文字 |
| 8 | 點空白處 | 選取框和懸浮列消失，面板標籤回到「下一個標注的設定」 |
| 9 | 縮小視窗到手機寬度 | 出現「常用工具 / 全部工具」tab 列 |
| 10 | 選取任意標注 | 色條最左側顯示藍色「選取中」，出現「✕ 刪除」按鈕 |

- [ ] **Step 2：更新版本號**

找到：
```html
      <span>版本：v1.2.0 (穩定版)</span>
```
改為：
```html
      <span>版本：v1.3.0 (穩定版)</span>
```

- [ ] **Step 3：Final commit**

```bash
git add index.html
git commit -m "feat: v1.3.0 - intuitive annotation UX redesign

- Universal hit detection: select any annotation with any tool
- Auto-select after drawing for immediate drag
- Text annotations support in-place editing
- Floating action bar with edit/delete near selected annotation
- Context-aware panel shows selected annotation's properties
- Mobile toolbar tab split: common tools vs all tools"
```
