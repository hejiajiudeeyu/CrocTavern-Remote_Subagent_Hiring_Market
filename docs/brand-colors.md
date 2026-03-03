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

## 3. 辅助色（不随主题变化）

| 角色/语义 | HEX | 说明 |
|---|---|---|
| Buyer | `#3B82F6` (蓝) / `#0f766e` (青) | 买家标识 |
| Seller | `#2563eb` (蓝) | 卖家标识 |
| Platform | `#4A6521` (croc-600) | 平台标识（与品牌色同系） |
| OK / Success | `#15803d` / `#10B981` | 成功状态 |

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
  --buyer: #0f766e;
  --seller: #2563eb;
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
}
```
