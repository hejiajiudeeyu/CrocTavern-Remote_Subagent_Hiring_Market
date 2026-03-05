# Brand Colors（品牌配色规范）

本文档定义 Croc Tavern 项目的统一配色方案，所有前端页面（ecosystem.html、playground.html 及后续页面）必须遵循。

## 1. 品牌主色（Croc Green）

来源：logo 中鳄鱼的橄榄绿色系。色阶按 Tailwind 惯例从 50（最浅）到 700（最深）。

| 色阶 | HEX | 用途 |
|---|---|---|
| croc-50 | `#F4F7F0` | 最浅背景、hover 态、alt 区域背景 |
| croc-100 | `#E4ECD8` | 浅色背景、badge、toggle track、accent 背景 |
| croc-200 | `#C9D9B1` | 边框高亮、箭头、浅色边框、光晕 |
| croc-500 | `#5C7A29` | 主品牌色 — 按钮、导航下划线、图标、primary 动作 |
| croc-600 | `#4A6521` | hover/active 态、导航高亮文字、platform 角色色 |
| croc-700 | `#3A5118` | 深色文字、渐变终点、表头文字、badge 文字 |

### 渐变

品牌渐变统一为：

```css
linear-gradient(135deg, #5C7A29, #3A5118)
```

用于 hero 标题高亮、CTA banner 背景等。

## 2. 语义映射

| 语义 | 变量名 | 色值 | 说明 |
|---|---|---|---|
| 品牌主色 | `--brand` / `--croc-500` | `#5C7A29` | 按钮、链接、图标默认色 |
| 品牌深色 | `--brand-deep` / `--croc-700` | `#3A5118` | 强调文字、深色背景文字 |
| 交替背景 | `--bg-alt` / `--croc-50` | `#F4F7F0` | section 交替背景 |
| 浅色边框 | `--croc-200` | `#C9D9B1` | 高亮边框、分隔线 |
| 基础边框 | `--line` | `#D4DFC6` | 卡片/面板边框 |

## 3. 角色色（方案 G：靛蓝 + 酒红 + Croc Green）

三方角色色完全独立，互不撞色。

| 角色 | 主色 HEX | 浅底 HEX | CSS 变量 | 用途 |
|---|---|---|---|---|
| **Buyer（买家）** | `#4F46B0` 靛蓝 | `#EEEDF7` | `--buyer` / `--buyer-light` | 标题、图标、步骤数字、大色块背景、泳道标签 |
| **Seller（卖家）** | `#8B2E45` 酒红 | `#F5ECF0` | `--seller` / `--seller-light` | 标题、图标、步骤数字、大色块背景、泳道标签 |
| **Platform（平台）** | `#5C7A29` Croc Green | — | `--croc-500` | 专属，不与买卖双方共用 |

### 用法规则

**深色大色块模式（买家/卖家双栏）**：
- 买家卡片整体背景 `--buyer`（靛蓝），卖家卡片整体背景 `--seller`（酒红）
- 卡片内标题、步骤标题：`#FFFFFF`
- 卡片内正文、副标题：`rgba(255,255,255,0.85)`
- 卡片内弱化文字（tagline、底部说明）：`rgba(255,255,255,0.8)`
- 步骤数字圆圈：`rgba(255,255,255,0.2)` 背景 + `#FFFFFF` 文字
- 受众标签（persona-tag）：`rgba(255,255,255,0.15)` 背景 + `#FFFFFF` 文字
- 分割线：`rgba(255,255,255,0.2)`
- 技术版代码高亮：`rgba(255,255,255,0.15)` 背景 + `#FFFFFF` 文字
- 技术版图标：`rgba(255,255,255,0.5)`

**浅色模式（场景流程图 flow-side 等）**：
- 买家侧浅底 `--buyer-light`（`#EEF2FF`）+ 标签用 `--buyer` 色文字
- 卖家侧浅底 `--seller-light`（`#FFF1F2`）+ 标签用 `--seller` 色文字
- Platform 相关元素（生态三角 platform 节点、四大支柱图标等）统一使用 Croc Green

## 4. 背景色

| 场景 | HEX | 说明 |
|---|---|---|
| 页面底色 | `#FAFAFA` / `#F6F8F3` | ecosystem / playground |
| 卡片底色 | `#FFFFFF` / `#FDFEFB` | 白底卡片 |
| Alt 区域 | `#F4F7F0` | 交替 section 背景 |
| 代码/表头背景 | `#F4F7F0` | 浅绿背景 |

## 5. 文字色（不变）

| 场景 | HEX |
|---|---|
| 主文字 | `#18181B` / `#1f2937` |
| 次文字 | `#52525B` / `#5b6472` |
| 弱化文字 | `#A1A1AA` |

## 6. playground.html 变量映射

```css
:root {
  --bg: #F6F8F3;
  --bg-card: #FDFEFB;
  --bg-accent: #E4ECD8;
  --ink: #1f2937;
  --muted: #5b6472;
  --line: #D4DFC6;
  --brand: #5C7A29;
  --brand-deep: #3A5118;
  --buyer: #4F46B0;
  --seller: #8B2E45;
  --platform: #4A6521;
  --ok: #15803d;
}
```

## 7. ecosystem.html 变量映射

```css
:root {
  --bg-page: #FAFAFA;
  --bg-card: #FFFFFF;
  --bg-alt: #F4F7F0;
  --croc-50: #F4F7F0;
  --croc-100: #E4ECD8;
  --croc-200: #C9D9B1;
  --croc-500: #5C7A29;
  --croc-600: #4A6521;
  --croc-700: #3A5118;
  --buyer: #4F46B0;
  --buyer-light: #EEEDF7;
  --seller: #8B2E45;
  --seller-light: #F5ECF0;
}
```
