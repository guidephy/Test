---
name: physics-web-quiz
description: >
  給高中物理老師使用的「網頁版」素養題生成工具。當使用者想建立線上題庫、
  生成網頁試卷、出網頁版題目、更新題庫、或說「出互動實驗題」時，立即使用此 skill。
  流程：搜尋近期話題 → 撰寫素養文章 → 依 Bloom 設計題目 → 生成含互動實驗的 HTML 試卷
  （KaTeX 公式、SVG/圖表嵌入、Canvas 互動模擬）→ 儲存到本地資料夾
  → 自動更新題庫首頁 index.html（依單元篩選）→ 回傳可點擊的本地連結。
  請在使用者提到「網頁題目」、「線上題庫」、「出網頁版」、「互動實驗」、「出題目」時立即觸發。
---

# 高中物理網頁素養題生成 Skill

## 設定

```
OUTPUT_DIR     = /sessions/wonderful-awesome-gates/mnt/physicsWebTest
QUIZZES_SUBDIR = quizzes
INDEX_FILE     = index.html
```

> 所有輸出檔案都存到 `OUTPUT_DIR`，不需要 GitHub API。

---

## 完整流程

```
輸入：物理單元主題
 ↓
步驟一：web_search 搜尋近期話題
 ↓
步驟二：撰寫素養文章（600–900 字）
 ↓
步驟三：（若有數據）Python 生成圖表 SVG
 ↓
步驟四：依 Bloom 設計 8–12 題
 ↓
步驟五：設計互動實驗（Canvas + 滑桿）
 ↓
步驟六：生成完整 HTML 試卷（含互動實驗區）
 ↓
步驟七：儲存到本地資料夾，更新 index.html
 ↓
步驟八：回傳 computer:// 連結
```

---

## 步驟一：搜尋近期話題

使用 web_search：
- `{單元主題} 新聞 {當前年份}`
- `{英文主題} recent news {當前年份}`
- 優先：pansci.asia、phys.org、technews.tw
- 選近 6 個月、有具體數據、與課綱連結清楚的新聞

---

## 步驟二：撰寫素養文章

結構（600–900 字，繁體中文）：
1. 引言：從新聞切入
2. 背景知識：介紹相關物理概念
3. 數據呈現：若有數據則規劃圖表
4. 物理應用：原理解釋現象
5. 反思延伸：引導更廣泛思考

---

## 步驟三：生成圖表（若文章含數據）

### 方式 A：Python 生成 SVG（優先）
```python
import matplotlib
matplotlib.use('Agg')
matplotlib.rcParams['font.family'] = ['DejaVu Sans']
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(7, 4))
# ... 繪圖邏輯（標題/軸標籤用英文）...
plt.savefig('/tmp/chart.svg', format='svg', bbox_inches='tight', facecolor='white')
plt.close()

with open('/tmp/chart.svg', 'r') as f:
    svg_content = f.read()
# 移除 <?xml ...?> 宣告，只保留 <svg>...</svg>
svg_content = svg_content[svg_content.find('<svg'):]
```

### 方式 B：使用者上傳圖片
若使用者提供圖片，將其轉為 base64 嵌入：
```python
import base64
with open('uploaded_image.png', 'rb') as f:
    b64 = base64.b64encode(f.read()).decode()
img_tag = f'<img src="data:image/png;base64,{b64}" alt="圖表" style="max-width:100%">'
```

---

## 步驟四：依 Bloom 設計題目

| Bloom 層次 | 建議題數 | 題型 |
|------------|----------|------|
| Remember 記憶 | 1–2 | 選擇題 |
| Understand 理解 | 2–3 | 選擇題、填充 |
| Apply 應用 | 2–3 | 計算題、選擇 |
| Analyze 分析 | 1–2 | 計算、簡答 |
| Evaluate 評鑑 | 1 | 簡答 |
| Create 創造 | 1 | 開放題 |

- 選擇題四選一，分步計算題提供必要數據
- 題目中**不顯示** Bloom 標注（只在答案區標注）

---

## 步驟五：設計互動實驗

互動實驗的目的是讓學生透過**調整滑桿**觀察物理量的變化，直觀理解概念或定義。
每個試卷必須包含**一個**與本單元核心概念相關的互動實驗。

### 5-1 互動實驗框架（通用結構）

實驗由以下三部分組成：
1. **說明區**：簡短描述實驗目的與觀察重點（2–3 句）
2. **控制區**：2–4 個滑桿，每個對應一個物理參數
3. **視覺化區**：Canvas 動畫或即時計算圖表，顯示參數改變的效果

技術規格：
- 使用原生 HTML5 Canvas + JavaScript（無外部依賴）
- 所有動畫用 `requestAnimationFrame` 驅動
- 滑桿用 `<input type="range">` 實作
- 實驗結果即時更新（每次滑桿變動立即重繪）
- Canvas 尺寸：寬 100%（max 760px），高 280–360px

### 5-2 各單元互動實驗設計指南

#### 🔴 力學 / 運動學 / 牛頓運動定律
**實驗：拋體運動模擬**
- 滑桿：初速度 $v_0$（5–30 m/s）、發射角度 $\theta$（10°–80°）
- Canvas：畫出拋物線軌跡，標注最高點、落地點，小球沿路徑動畫移動
- 即時顯示：飛行時間、最大高度、水平射程
- 公式：$x = v_0\cos\theta \cdot t$，$y = v_0\sin\theta \cdot t - \frac{1}{2}gt^2$

#### 🔴 動量與衝量
**實驗：彈性/非彈性碰撞**
- 滑桿：$m_1$（1–5 kg）、$v_1$（1–10 m/s）、$m_2$（1–5 kg）
- Canvas：兩個方塊碰撞動畫，碰撞前後速度以箭頭顯示
- 切換按鈕：彈性碰撞 / 非彈性碰撞
- 即時顯示：碰撞前後動量、動能變化

#### 🟠 功與能量
**實驗：斜面能量轉換**
- 滑桿：斜面角度（10°–60°）、摩擦係數（0–0.5）、初始高度（1–5 m）
- Canvas：小球沿斜面滑下，動態顯示動能/位能/熱能的長條圖變化
- 即時顯示：各種能量數值

#### 🟠 波動 / 聲波
**實驗：波形疊加（干涉）**
- 滑桿：波一頻率（1–5 Hz）、波二頻率（1–5 Hz）、振幅比（0.5–2）
- Canvas：畫出兩個波形 + 疊加結果，用不同顏色區分
- 即時展現建設性/破壞性干涉

#### 🟡 光學 / 幾何光學
**實驗：薄透鏡成像**
- 滑桿：焦距 $f$（5–20 cm）、物距 $d_o$（3–40 cm）
- Canvas：畫出主光軸、透鏡、三條代表光線、成像位置
- 即時顯示：像距 $d_i$（透鏡公式 $\frac{1}{f}=\frac{1}{d_o}+\frac{1}{d_i}$）、放大率 $M$，並標注實像/虛像

#### 🔵 電學 / 電路學
**實驗：歐姆定律電路**
- 滑桿：電壓 $V$（1–12 V）、電阻 $R$（10–1000 Ω）
- Canvas：繪製簡單電路圖，燈泡亮度隨電流變化（黃色光暈大小代表亮度）
- 即時顯示：電流 $I = V/R$、功率 $P = V^2/R$

#### 🔵 電磁學 / 電磁感應
**實驗：法拉第感應**
- 滑桿：磁場變化率 $\Delta B/\Delta t$（0.1–5 T/s）、線圈面積（0.01–0.1 m²）、匝數（1–20）
- Canvas：動畫顯示磁通量變化，感應電流方向（右手定則箭頭）
- 即時顯示：感應電動勢 $\varepsilon = -N\frac{\Delta\Phi}{\Delta t}$

#### 🟣 熱學 / 熱力學
**實驗：理想氣體狀態方程**
- 滑桿：溫度 $T$（200–500 K）、體積 $V$（0.5–3 L）
- Canvas：活塞容器動畫，氣體分子速度隨溫度變化（粒子動畫）
- 即時顯示：壓力 $P = nRT/V$，PV 圖上的點

#### 🟣 近代物理 / 量子 / 原子物理
**實驗：光電效應**
- 滑桿：光的頻率（$3\times10^{14}$–$3\times10^{15}$ Hz）、功函數（1–5 eV）
- Canvas：金屬板受光照射，電子噴出動畫，速度與頻率正相關
- 即時顯示：最大動能 $E_k = hf - W$，截止電壓

### 5-3 通用 JavaScript 實驗框架

以下是每個實驗的標準程式結構，Claude 應根據步驟五的設計填入具體邏輯：

```javascript
// ===== 互動實驗核心框架 =====
(function() {
  const canvas = document.getElementById('expCanvas');
  const ctx = canvas.getContext('2d');

  // 響應式尺寸
  function resizeCanvas() {
    const w = canvas.parentElement.clientWidth - 32;
    canvas.width = Math.min(w, 760);
    canvas.height = 300; // 可依需求調整
  }
  resizeCanvas();
  window.addEventListener('resize', () => { resizeCanvas(); draw(); });

  // 取得所有滑桿值
  function getParams() {
    return {
      p1: parseFloat(document.getElementById('slider1').value),
      p2: parseFloat(document.getElementById('slider2').value),
      // 視需要增加 p3, p4
    };
  }

  // 更新滑桿旁的數值顯示
  function updateLabels(p) {
    document.getElementById('val1').textContent = p.p1.toFixed(1);
    document.getElementById('val2').textContent = p.p2.toFixed(1);
  }

  // 更新計算結果區
  function updateResults(p) {
    const result = document.getElementById('expResult');
    // 填入計算結果 HTML
    result.innerHTML = `計算結果：...`;
  }

  // 主繪圖函式（每次滑桿變動都呼叫）
  function draw() {
    const p = getParams();
    updateLabels(p);
    updateResults(p);
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // ---- 在此填入各單元的具體繪圖邏輯 ----
    // 使用 ctx.beginPath(), ctx.arc(), ctx.lineTo() 等 Canvas API
    // 如需動畫，用 requestAnimationFrame(animate) 驅動
  }

  // 若需要連續動畫（如拋體軌跡、碰撞動畫）
  let animFrame = null;
  let t = 0;
  function animate() {
    t += 0.016; // 模擬 ~60fps 時間推進
    draw();
    animFrame = requestAnimationFrame(animate);
  }
  // 用按鈕觸發：document.getElementById('startBtn').onclick = () => { t=0; animate(); };

  // 滑桿事件（即時更新）
  document.querySelectorAll('.exp-slider').forEach(s => {
    s.addEventListener('input', () => {
      if (animFrame) cancelAnimationFrame(animFrame);
      draw();
    });
  });

  // 初始繪製
  draw();
})();
```

---

## 步驟六：生成 HTML 試卷

HTML 檔案路徑：`{OUTPUT_DIR}/quizzes/{unit_slug}/{unit_slug}_{YYYYMMDD}.html`

取得 unit_slug：
```python
import re, hashlib

UNIT_SLUG_MAP = {
    "力學": "mechanics", "運動學": "kinematics",
    "牛頓運動定律": "newton_laws", "動量與衝量": "momentum",
    "功與能量": "energy", "萬有引力": "gravitation",
    "流體力學": "fluid", "熱學": "thermodynamics",
    "熱力學": "thermodynamics", "波動": "waves",
    "聲波": "sound", "光學": "optics",
    "幾何光學": "geometric_optics", "電學": "electricity",
    "靜電學": "electrostatics", "電路學": "circuits",
    "電流與電路": "circuits", "磁學": "magnetism",
    "電磁學": "electromagnetism", "電磁感應": "em_induction",
    "近代物理": "modern_physics", "量子物理": "quantum",
    "原子物理": "atomic", "核物理": "nuclear", "相對論": "relativity",
}

def get_unit_slug(unit_name):
    slug = UNIT_SLUG_MAP.get(unit_name)
    if slug: return slug
    safe = re.sub(r'[^\w]', '', unit_name)
    if safe and safe.isascii(): return safe.lower()
    return f"unit_{hashlib.md5(unit_name.encode()).hexdigest()[:8]}"
```

### ⚠️ 重要：Python 字串與 LaTeX 反斜線衝突（必讀！）

Python 的普通字串（非 raw string）會將特定反斜線序列解讀為控制字元，導致 LaTeX 公式損壞：

| 原始 LaTeX    | Python 解讀               | 檔案中的錯誤結果 |
|--------------|--------------------------|----------------|
| `\varepsilon` | `\v`（垂直 tab, chr 11）  | `arepsilon`    |
| `\theta`      | `\t`（tab, chr 9）        | `heta`         |
| `\frac{`      | `\f`（換頁, chr 12）      | `rac{`         |
| `\approx`     | `\a`（響鈴, chr 7）       | `pprox`        |
| `\times`      | `\t`（tab, chr 9）        | `imes`         |
| `\text{`      | `\t`（tab, chr 9）        | `ext{`         |

**解法：所有含 LaTeX 的 HTML 字串，一律使用 Python raw string `r'''...'''`**

```python
# ✅ 正確：raw string，反斜線原樣保留
html_article = r'''
<p>感應電動勢：$\varepsilon = -N\dfrac{\Delta\Phi}{\Delta t}$</p>
<p>磁通量：$\Phi = B \cdot A \cdot \cos\theta$</p>
'''

# ❌ 錯誤：普通字串，\v、\t、\f 被解讀為控制字元
html_article = '''
<p>感應電動勢：$\varepsilon = ...$</p>  ← \v 變成垂直tab，公式損壞！
'''
```

**若需在 raw string 中插入 Python 變數**，用 `%` 格式化或字串串接：
```python
# 用 % 格式化（raw string 支援 %s）
html = r'''<p>單元：%s，日期：%s，題數：%d 題</p>''' % (unit_name, date_str, total_questions)

# 或字串串接
html = r'''前半段 LaTeX $\varepsilon$ ''' + str(variable) + r''' 後半段 $\theta$'''
```

> 每次生成 HTML 前，先確認所有 Python 字串都是 `r'''...'''` 格式。

### HTML 完整模板結構

生成 HTML 時，按照以下結構組合：
1. `<head>` - KaTeX CDN、CSS 樣式（含 `.experiment-card` 互動實驗區 CSS）
2. `<body>` - 抬頭 → 文章區 → **互動實驗區** → 各題型區塊 → 頁尾
3. `<script>` - `toggleAnswer()` 函式 + 互動實驗 JavaScript

#### CSS 必包含（互動實驗相關）

```css
:root {
  --primary: #1a5276; --accent: #2e86c1;
  --bg: #f4f6f9; --card: #ffffff;
  --text: #2c3e50; --border: #d5d8dc;
  --exp: #1e8449;  /* 互動實驗專用綠色 */
}

/* 互動實驗區塊 */
.experiment-card {
  background: var(--card); border-left: 4px solid var(--exp);
  border-radius: 8px; padding: 1.5rem 2rem; margin-bottom: 2rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
}
.experiment-card h2 {
  color: var(--exp); font-size: 1.15rem; margin-bottom: 0.5rem;
}
.experiment-desc { font-size: 0.95rem; color: #555; margin-bottom: 1rem; }
.exp-canvas-wrap {
  background: #f8fffe; border: 1px solid #b2dfdb;
  border-radius: 8px; padding: 1rem; margin-bottom: 1rem; overflow: hidden;
}
canvas#expCanvas { display: block; width: 100%; border-radius: 4px; }
.exp-controls { display: flex; flex-direction: column; gap: 0.8rem; }
.exp-control-row { display: flex; align-items: center; gap: 1rem; flex-wrap: wrap; }
.exp-control-row label {
  min-width: 130px; font-size: 0.9rem; font-weight: bold; color: var(--exp);
}
.exp-slider { flex: 1; min-width: 160px; accent-color: var(--exp); }
.exp-val { min-width: 70px; text-align: right; font-size: 0.9rem; font-weight: bold; color: var(--primary); }
.exp-result {
  background: #e8f8f5; border-radius: 6px; padding: 0.8rem 1rem;
  margin-top: 1rem; font-size: 0.9rem; display: flex; gap: 2rem; flex-wrap: wrap;
}
```

#### HTML 互動實驗區塊（放在文章後、題目前）

```html
<div class="experiment-card">
  <h2>🧪 互動實驗：{EXPERIMENT_TITLE}</h2>
  <p class="experiment-desc">{EXPERIMENT_DESC}——請調整下方滑桿，觀察物理量的變化。</p>
  <div class="exp-canvas-wrap">
    <canvas id="expCanvas" height="300"></canvas>
  </div>
  <div class="exp-controls">
    <div class="exp-control-row">
      <label>{參數一}：</label>
      <input type="range" id="slider1" class="exp-slider"
        min="{min1}" max="{max1}" step="{step1}" value="{def1}">
      <span class="exp-val"><span id="val1">{def1}</span> {單位1}</span>
    </div>
    <div class="exp-control-row">
      <label>{參數二}：</label>
      <input type="range" id="slider2" class="exp-slider"
        min="{min2}" max="{max2}" step="{step2}" value="{def2}">
      <span class="exp-val"><span id="val2">{def2}</span> {單位2}</span>
    </div>
    <!-- 依需要加更多滑桿 -->
  </div>
  <div class="exp-result" id="expResult"></div>
</div>
```

#### 公式寫法（KaTeX）
| 類型 | 寫法 |
|------|------|
| 行內公式 | `$F = ma$` |
| 獨立公式 | `$$\vec{F} = m\vec{a}$$` |
| 分數 | `$\frac{v^2}{r}$` |
| 根號 | `$\sqrt{2gh}$` |
| 單位 | `$10\ \text{m/s}^2$` |

---

## 步驟七：儲存到本地資料夾，更新 index.html

### 7-1 儲存題目 HTML

```python
import os, json, re
from datetime import datetime

OUTPUT_DIR = "/sessions/wonderful-awesome-gates/mnt/physicsWebTest"
date_str = datetime.now().strftime('%Y%m%d')
unit_slug = get_unit_slug(unit_name)

# 建立子目錄
quiz_dir = os.path.join(OUTPUT_DIR, "quizzes", unit_slug)
os.makedirs(quiz_dir, exist_ok=True)

# 儲存題目 HTML
quiz_filename = f"{unit_slug}_{date_str}.html"
quiz_path = os.path.join(quiz_dir, quiz_filename)
with open(quiz_path, 'w', encoding='utf-8') as f:
    f.write(quiz_html_content)

print(f"✅ 題目已儲存：{quiz_path}")
```

### 7-2 讀取現有 index.html 的題目清單

```python
index_path = os.path.join(OUTPUT_DIR, "index.html")
quiz_list = []

if os.path.exists(index_path):
    with open(index_path, 'r', encoding='utf-8') as f:
        existing_html = f.read()
    m = re.search(r'<!-- QUIZ_DATA_START -->(.*?)<!-- QUIZ_DATA_END -->', existing_html, re.DOTALL)
    if m:
        try:
            # 取出 JSON 部分（去掉 JS 變數宣告）
            raw = m.group(1).strip()
            raw = re.sub(r'^const QUIZ_LIST\s*=\s*', '', raw).rstrip(';').strip()
            quiz_list = json.loads(raw)
        except Exception as e:
            print(f"⚠️ 無法解析舊清單，從空白開始：{e}")
            quiz_list = []
```

### 7-3 新增本次題目到清單

```python
quiz_relative_url = f"quizzes/{unit_slug}/{quiz_filename}"

quiz_list.append({
    "unit": unit_name,
    "title": article_title,
    "date": date_str,
    "url": quiz_relative_url,
    "has_experiment": True,
    "bloom_levels": ["Apply", "Analyze", "Evaluate"],
    "question_count": total_questions
})
```

### 7-4 生成並儲存新版 index.html

```python
quiz_json = json.dumps(quiz_list, ensure_ascii=False, indent=2)

new_index_html = '''<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>桃園觀音高中物理科 - 線上題庫</title>
  <style>
    :root { --primary:#1a5276; --accent:#2e86c1; --bg:#f4f6f9; --exp:#1e8449; }
    * { box-sizing:border-box; margin:0; padding:0; }
    body { font-family:"Microsoft JhengHei",sans-serif; background:var(--bg); color:#2c3e50; }
    .header { background:var(--primary); color:white; padding:2rem; text-align:center; }
    .header h1 { font-size:1.8rem; }
    .header p { margin-top:0.5rem; opacity:0.85; }
    .container { max-width:960px; margin:0 auto; padding:2rem 1.5rem; }
    .filter-bar { display:flex; gap:1rem; margin-bottom:2rem; flex-wrap:wrap; align-items:center; }
    .filter-bar select, .filter-bar input {
      padding:0.5rem 1rem; border:1px solid #ccc; border-radius:6px; font-size:0.95rem;
    }
    .quiz-grid { display:grid; gap:1.2rem; grid-template-columns:repeat(auto-fill,minmax(280px,1fr)); }
    .quiz-card {
      background:white; border-radius:10px; padding:1.2rem 1.5rem;
      box-shadow:0 2px 8px rgba(0,0,0,0.07);
      border-left:4px solid var(--accent);
      text-decoration:none; color:inherit; display:block; transition:transform 0.15s;
    }
    .quiz-card.has-exp { border-left-color:var(--exp); }
    .quiz-card:hover { transform:translateY(-2px); }
    .unit-badge {
      display:inline-block; background:var(--accent); color:white;
      font-size:0.75rem; padding:0.2rem 0.6rem; border-radius:10px; margin-bottom:0.5rem;
    }
    .exp-badge {
      display:inline-block; background:var(--exp); color:white;
      font-size:0.7rem; padding:0.15rem 0.5rem; border-radius:10px; margin-left:0.3rem;
    }
    .quiz-card h3 { font-size:1rem; margin-bottom:0.4rem; }
    .quiz-card .meta { font-size:0.8rem; color:#888; }
    .no-result { text-align:center; color:#aaa; padding:3rem 0; }
  </style>
</head>
<body>
<div class="header">
  <h1>🔬 桃園觀音高中物理科 線上題庫</h1>
  <p>素養導向試題 ／ 含互動實驗 ／ 依單元篩選</p>
</div>
<div class="container">
  <div class="filter-bar">
    <label>篩選單元：</label>
    <select id="unitFilter" onchange="filterQuizzes()">
      <option value="">全部單元</option>
    </select>
    <select id="expFilter" onchange="filterQuizzes()">
      <option value="">全部題型</option>
      <option value="exp">含互動實驗</option>
    </select>
    <input type="text" id="searchInput" placeholder="搜尋關鍵字..."
      oninput="filterQuizzes()" style="flex:1;min-width:150px">
    <span id="countDisplay" style="color:#888;font-size:0.9rem"></span>
  </div>
  <div class="quiz-grid" id="quizGrid"></div>
  <div class="no-result" id="noResult" style="display:none">找不到符合的題目</div>
</div>
<script>
<!-- QUIZ_DATA_START -->
const QUIZ_LIST = ''' + quiz_json + ''';
<!-- QUIZ_DATA_END -->
function initFilter() {
  const units = [...new Set(QUIZ_LIST.map(q => q.unit))].sort();
  const sel = document.getElementById("unitFilter");
  units.forEach(u => {
    const opt = document.createElement("option");
    opt.value = u; opt.textContent = u; sel.appendChild(opt);
  });
}
function filterQuizzes() {
  const unit = document.getElementById("unitFilter").value;
  const expOnly = document.getElementById("expFilter").value === "exp";
  const kw = document.getElementById("searchInput").value.toLowerCase();
  const filtered = QUIZ_LIST.filter(q => {
    return (!unit || q.unit === unit) &&
           (!expOnly || q.has_experiment) &&
           (!kw || q.title.toLowerCase().includes(kw) || q.unit.toLowerCase().includes(kw));
  }).sort((a,b) => b.date.localeCompare(a.date));
  const grid = document.getElementById("quizGrid");
  const noResult = document.getElementById("noResult");
  document.getElementById("countDisplay").textContent = `共 ${filtered.length} 份`;
  if (filtered.length === 0) { grid.innerHTML = ""; noResult.style.display = "block"; return; }
  noResult.style.display = "none";
  grid.innerHTML = filtered.map(q => `
    <a class="quiz-card ${q.has_experiment ? "has-exp" : ""}" href="${q.url}" target="_blank">
      <div>
        <span class="unit-badge">${q.unit}</span>
        ${q.has_experiment ? '<span class="exp-badge">🧪 互動實驗</span>' : ""}
      </div>
      <h3>${q.title}</h3>
      <div class="meta">${q.date.slice(0,4)}/${q.date.slice(4,6)}/${q.date.slice(6,8)} ／ ${q.question_count} 題</div>
    </a>
  `).join("");
}
initFilter();
filterQuizzes();
</script>
</body>
</html>'''

with open(index_path, 'w', encoding='utf-8') as f:
    f.write(new_index_html)
print(f"✅ 題庫首頁已更新：{index_path}")
```

---

## 步驟八：回傳結果

在對話中以 Markdown 連結格式回傳，讓使用者可直接點擊開啟：

```python
quiz_computer_url = f"computer://{quiz_path}"
index_computer_url = f"computer://{index_path}"
print(f"[📄 開啟試卷（含互動實驗）]({quiz_computer_url})")
print(f"[🏠 開啟題庫首頁]({index_computer_url})")
```

範例回應格式：
```
✅ 題目已成功生成！

[📄 開啟試卷：{UNIT_NAME}（{DATE}）](computer:///sessions/wonderful-awesome-gates/mnt/physicsWebTest/quizzes/{unit_slug}/{filename})
[🏠 開啟題庫首頁](computer:///sessions/wonderful-awesome-gates/mnt/physicsWebTest/index.html)
```

---

## 品質檢查清單

- [ ] HTML 在瀏覽器開啟後 KaTeX 公式正常顯示
- [ ] 圖表為 SVG 內嵌或 base64（無外部依賴）
- [ ] 答案折疊按鈕運作正常
- [ ] 題目涵蓋至少 4 個 Bloom 層次
- [ ] **互動實驗**：滑桿可拖動，Canvas 即時更新，有計算結果顯示
- [ ] 互動實驗主題與本單元核心概念直接相關
- [ ] index.html `<!-- QUIZ_DATA_START/END -->` 標記完整
- [ ] 新題目出現在題庫首頁，顯示「🧪 互動實驗」標籤
- [ ] quiz_path 不含中文字元（使用英文 slug）
- [ ] 回傳的 computer:// 連結可直接點擊開啟

---

## 錯誤排除

| 問題 | 原因 | 解法 |
|------|------|------|
| 資料夾無法建立 | OUTPUT_DIR 不存在或無寫入權限 | 確認掛載路徑正確，用 `os.makedirs(..., exist_ok=True)` |
| Canvas 不顯示 | JS 語法錯誤 | 在瀏覽器 DevTools Console 查看錯誤 |
| 滑桿沒有反應 | HTML id 與 JS id 不匹配 | 確認 `id="slider1"` 與 `getElementById('slider1')` 一致 |
| 公式不顯示 | KaTeX CDN 無法載入 | 確認網路，或改用 jsDelivr 備用 CDN |
| index.html 解析失敗 | QUIZ_DATA 標記被修改 | 重建 index.html，quiz_list 從空陣列開始 |
| 互動實驗畫面空白 | Canvas 尺寸未初始化 | 確認 `resizeCanvas()` 在頁面載入後立即執行 |
