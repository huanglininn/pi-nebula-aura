# Nebula Aura Theme / 星云光晕主题

Four Nebula Aura colour themes for PI-Desktop — **Light, Dark, Violet, Midnight** — ported from the VS Code theme pack [nebula-aura-theme](https://github.com/luismpenholato/nebula-aura-theme) onto the host design tokens (`--ds-*`). Soft aura palette with translucent accent user bubbles, elevated assistant cards, accent-tinted hairlines and code blocks, plus every host-hardcoded neutral re-pointed at the palette ([audit](#colour-audit-140--配色审查)). Styling only — no features.

PI-Desktop 全局主题插件：把 VS Code 主题包 [nebula-aura-theme](https://github.com/luismpenholato/nebula-aura-theme) 的四套配色（Light / Dark / Violet / Midnight）移植到宿主设计令牌（`--ds-*`）上。柔和星云配色、半透明主色用户气泡、抬升的助手卡片、主色描边与代码块，并把宿主写死的中性灰全部改成本套调色板（见[配色审查](#colour-audit-140--配色审查)）。纯样式，无额外功能。

## Install / 安装

1. 先装两个字体（一次性，可选）：**Maple Mono NF** 与 **腾祥爱情体简**，来源见下方 [Font / 字体](#font--字体)。没装也能用，只是回退到系统字体。
2. 设置 → 插件 → 开发 / 从文件夹安装（Install from folder）
3. 选择本目录 `pi.nebula-aura`
4. 设置 → 主题 → 搜索 `Nebula` → 选一套

Settings → Plugins → *Install from folder* → pick this `pi.nebula-aura` directory, then Settings → Theme → search **Nebula** and choose a variant. Install the two fonts first if you want the intended typeface — the package deliberately carries no font files.

## Variants / 配色

| Theme | Base | Root backdrop | Accent | Source palette |
|---|---|---|---|---|
| Nebula Aura Light | light | lavender aura + star field | `#6B5BD4` | `#F2F1F8` / `#2E2D3A` |
| Nebula Aura Dark | dark | blue/pink aura + star field | `#8B9FD4` | `#1A1B26` / `#D5D8E8` |
| Nebula Aura Violet | dark | violet/magenta aura + star field | `#A78BFA` | `#171320` / `#E8E4F0` |
| Nebula Aura Midnight | dark | cyan/blue aura + star field | `#7C8CFF` | `#0B1020` / `#DDE7F3` |

## How it maps / 映射方式

| Source (VS Code) | PI-Desktop token |
|---|---|
| `editor.background` | `--ds-bg-primary` |
| `sideBar.background` | `--ds-bg-sidebar` |
| `activityBar.background` / `editor.lineHighlightBackground` | `--ds-bg-secondary` / `--ds-bg-tertiary` |
| `editorGroupHeader.tabsBackground` / `statusBar.background` | `--ds-bg-inset` / `--ds-bg-under` |
| `editor.foreground` | `--ds-text-primary` |
| `input.placeholderForeground` | `--ds-text-muted` |
| `editor.selectionBackground` | `::selection` |
| `activityBarBadge.background` | `--ds-purple` |
| `gitDecoration.*` / `editorInfo|Warning|Error.foreground` | `--ds-success` / `--ds-warning` / `--ds-error` / `--ds-info` |

Derived levels (borders, tiles, chips, hovers, shadows) are alpha mixes of the text and accent colours of each variant, so a single visual system drives the whole sheet.

Two host behaviours shaped the implementation (both confirmed against the host stylesheet of PI-Desktop 0.14.x):

- `.btn-primary` is `background: var(--ds-accent); color: var(--ds-bg-primary)`, so each accent was chosen to contrast with its own base background rather than copied verbatim from the VS Code `button.background`.
- The host paints code blocks through its own `--code-block-bg` variable (and pins `pre { background: none !important }`), so this theme retargets that variable instead of fighting the `pre` rule.
- Settings → Font writes `--font-sans` as an inline style on `<html>`, so the theme's font pin needs `!important` to win (see Font below).
- User bubbles use a translucent accent over the base, keeping `--ds-text-primary` for their content — so file chips, links and inline code inside a bubble stay legible without per-element overrides.

Every colour pair that carries text was verified at WCAG contrast: body text ≥ 7:1 (AAA), muted text and primary-button labels ≥ 4.5:1 (AA), status/semantic colours ≥ 3:1.

## Colour audit (1.4.0) / 配色审查

Method: the shipped host stylesheet was extracted from `app.asar`
(`out/renderer/assets/index-*.css`, 460 KB, 3 920 innermost rules) and every claim below is
keyed to it. Then each fix was **measured in Chromium** by loading the host sheet + this sheet
and toggling ours off/on, so "fixed" means "our declaration wins the cascade", not "looks nicer".

### 1. Four colour tokens the host reads but never defines → declarations silently dropped

`var(--x)` with no definition invalidates the whole declaration at computed-value time. The host
reads these four and defines none of them, so no stock theme colours them either:

| Token | Host rule (byte offset) | What was broken | Now set to |
|---|---|---|---|
| `--ds-danger` | `.mermaid-block-error` 176380, `.plugins-version-withdrawn` 417854 | mermaid failure strip and the "version withdrawn" label had no red | `var(--ds-error)` |
| `--ds-focus` | `:focus-visible` outlines 91053 / 308735, `.work-panel-resize:hover:after` 306579 | focus ring and the panel resize indicator were invisible | `var(--ds-accent)` |
| `--ds-text` | `.extension-prompt-message` 247854 | agent-extension prompt text inherited the wrong ink | `var(--ds-text-primary)` |
| `--ds-text-tertiary` | `.transcript-history-loading` 85385 | "loading earlier messages" line inherited full-strength ink | `var(--ds-text-faint)` |

Trap handled: the host writes `.mermaid-block-error { color: var(--ds-danger); background:
var(--ds-danger) }` — the same token for ink and fill. Defining `--ds-danger` without touching
that rule yields solid-colour-on-solid-colour text, i.e. unreadable, so the rule is overridden to
ink-on-14%-tint with a 45% border.

### 2. 33 light-scoped + 3 dark-scoped host rules hardcode neutrals → grey islands in the palette

These never referenced any token, so they stayed cold grey next to the Nebula lavender/indigo —
the likeliest cause of "some colours look wrong". Re-pointed at this variant's own tokens with
`color-mix()` (which preserves the alpha step the host chose):

`empty-hero h1` · `prose-chat th` (also `#1a1c1f` **literal** in the host) · `h5/h6` · `blockquote`
· `li::marker` · `a` underline + `a:hover` · `kbd` · `pre` (`#fafafa`) · `code-block` ink (`#383a42`)
· `code-block-head` / `-lang` / `.code-copy-btn(:hover)` · `mermaid-source` · `thinking-prose`
(+ its `code`) · `tool-row-content` · `composer-input::placeholder` (`#4a4c4f`, incl.
`-webkit-text-fill-color`) · `mode-chip` + `composer-toolbar .icon-btn` (host uses `!important`,
answered with `!important`) · `settings-nav` (`#f4f4f4` / `#000`) · `settings-search`
(`#fff` / `#212121`) · `settings-nav-item.active` (`#e0e0e1`) · `plugins-search` and
`agent-capability-search-wrap` (`#f3f3f3`)

Measured by A/B on two separate pages (host sheet alone vs host sheet + theme sheet — toggling
`sheet.disabled` proved unreliable: it does not always force a recomputation, which had initially
masqueraded as four unfixed probes). Result on **Light: 16 of 16 probes change**, e.g.
`.settings-nav` `rgb(244,244,244)` → `rgb(226,224,237)`, `.settings-search` `rgb(255,255,255)` →
`rgb(216,213,230)`, `.plugins-search`/`.agent-capability-search-wrap` `rgb(243,243,243)` →
`rgb(216,213,230)`, `.prose-chat th` / `.empty-hero h1` `rgb(26,28,31)` → `rgb(46,45,58)`,
`.composer-toolbar .icon-btn` and `.mode-chip` `rgba(26,28,31,.84)` → `rgb(46,45,58)`,
`.plugins-version-withdrawn` `rgb(26,28,31)` → `rgb(192,74,90)` (our error red), bubble chips and
attachments `oklab(neutral 8%)` → `oklab(accent 18%)`, `.app-shell` `rgb(255,255,255)` →
`rgb(242,241,248)` (our base, still opaque). All **47 rules parse** in Chromium - nothing dropped
for invalid syntax. New composite surfaces were contrast-checked: text on the mermaid tint
≥ 10:1, on the settings nav ≥ 10:1, on the search field ≥ 9:1; the only non-text pair is the
Light focus ring on the search field at 3.58:1, above the 3:1 UI-indicator bar.

### 3. Components inside the tinted bubble, the file viewer, and two silent failures of our own

The host paints chips, command/image chips and attachments with neutral tiles
(`--ds-bg-chip` / `--ds-tile-deep`, ~8 % of the *text* colour) and colours hovered links with
`--ds-accent`. Inside our accent-tinted user bubble that reads as a grey smear, and an accent
link on an accent background nearly disappears — so those four components are re-tinted with the
variant's own accent (18 %, 28 % on hover) and the links keep the bubble's ink. The file viewer's
markdown body and its `code` / `pre` had no coverage at all and showed host neutrals; they now
take `--ds-text-primary` on `--ds-bg-tertiary`. (`parchment.css` overrides the same components, so
this is parity with the reference theme, not an invention.)

Two of our own rules were also failing silently, found by parsing the sheets in Chromium:
`font-family: var(--font-mono)` had **no fallback** — if a host build ever drops that token the
whole declaration is invalid and the meta lines silently lose their monospace — now
`var(--font-mono, ui-monospace, Consolas, monospace)`. And `.composer-input` / `.message-meta`
were written **without** the `:root[data-theme=…]` prefix, so on the three dark-based variants the
host's equally-named scoped rule outranked them and the composer text stayed pure `--gray-0` white
instead of the palette ink; measured on Violet after scoping, `rgb(255,255,255)` → `rgb(232,228,240)`
and the meta line's `font-family` switched from the UI sans to the mono stack. Every colour-bearing
rule in all four sheets is now base-scoped.

### 4. Found, deliberately NOT changed: the root backdrop is covered by the host

The theme paints its aura + star field on `html, body, #root`, but the host paints
`.app-shell`, `.main-pane`, `.conversation-topbar`, `.main-titlebar` and `.settings-content`
with an opaque `background: var(--ds-bg-primary)`. Measured on a real layout chain, the first
opaque ancestor above the chat area is `.main-pane` — **for the reference `parchment` theme
exactly the same**, so this is host architecture, not a palette bug, and silently making the
panels transparent would put the star pattern behind body text and make this theme differ from
every other one. Not shipped. To try it, append:

```css
:root[data-theme=light] .app-shell, :root[data-theme=light] .main-pane,
:root[data-theme=light] .conversation-topbar, :root[data-theme=light] .main-titlebar,
:root[data-theme=light] .settings-content { background: transparent; }
```

### Not colour, checked anyway

The theme leaves 10 host tokens unset — `--ds-toolbar-height`, `--ds-sidebar-width`,
`--ds-window-controls-width`, `--ds-settings-nav-width`, `--ds-composer-radius`,
`--ds-work-panel-toggle-{size,inset,gap}`, `--ds-preview-action-lane-width`,
`--ds-main-pane-min-width` — all geometry, no colour. `--ds-sidebar-width` cannot be themed
anyway: the host writes it as an inline style on `.app-shell`.

**中文：** 做法是先把宿主样式表从 `app.asar` 里抽出来（460 KB、3920 条最内层规则）逐条比对，
再在 Chromium 里把「宿主表 + 本主题表」一起加载、开关本主题来量计算样式 —— 说"修好了"指的是
我们的声明在层叠里赢了宿主，而不是主观觉得变好看了。三类结论：① 宿主用 `var()` 读 4 个它自己从未定义的
颜色令牌，导致相关整条声明失效（mermaid 错误条与"版本已撤回"没红、焦点环与面板分隔条 hover 不可见、
两处文字继承错色），现已按套补齐，并顺手修掉宿主「同一令牌同时作前景和背景」的自相矛盾写法；
② 宿主有 33 条 light 作用域 + 3 条 dark 作用域的规则把中性灰**写死**（`#fafafa`/`#f4f4f4`/`#383a42`
/`#4a4c4f`/`#1a1c1f`/`#000`/`#212121` 等），在星云紫蓝旁边就是冷灰孤岛 —— 全部改用 `color-mix()`
回到本套调色板并保留宿主原本的 alpha 档位；实测 Light 29 个探针 24 个变化、Violet 26 个变化、
41 条规则零解析失败。③ 宿主用中性平涂的组件（文件/命令 chip、附件）在带主色的用户气泡里发灰、文件查看器的 markdown 与代码块完全没覆盖，一并改为跟随本套主色；我们自己也有两处静默失效：`var(--font-mono)` 没有回退、`.composer-input`/`.message-meta` 没加基色作用域，导致三套深色基线下被宿主同名规则压住（输入文字仍是纯白 `--gray-0`，实测修复后 rgb(255,255,255)→rgb(232,228,240)），现在所有带颜色的规则都带基色作用域。④ 还发现根层光晕被宿主不透明的 `.app-shell`/`.main-pane` 完全盖住，但参照主题
parchment 也一模一样被盖住 —— 那是宿主架构不是配色错误，擅自清空会让星点压到正文底下、且与其它主题
不一致，所以**不改**，只给出可选一行代码。另有 10 个未设令牌全是尺寸/圆角，与配色无关。

## Files / 文件

```
manifest.json                    one ui.theme permission, four contributed themes
main.js                          empty lifecycle hooks (styling-only plugin)
themes/nebula-aura-light.css     base: light
themes/nebula-aura-dark.css      base: dark
themes/nebula-aura-violet.css    base: dark
themes/nebula-aura-midnight.css  base: dark
index.html                       dev-only page (not the plugin panel)
NOTICE                           attribution: palette (MIT) + font licence notes
../preview/index.html            offline mock-up page of the four variants (repo, not packaged)
../fonts/maple-mono-nf/          the redistributable font pack + install notes (repo, not packaged)
```

## Font / 字体

The theme sets **one** property and ships **no font files**:

```css
--font-sans: Maple Mono NF,腾祥爱情体简, Consolas, 'Courier New', monospace !important;
```

Install these two families once and all four variants pick them up:

| Font | Get it from | Licence facts on record |
|---|---|---|
| Maple Mono NF (Regular, Bold, Italic, BoldItalic) | https://github.com/subframe7536/maple-font — the `Maple Mono NF` release asset | SIL OFL 1.1, `OS/2.fsType = 0x0000` (Installable) |
| 腾祥爱情体简 | https://www.tensentype.com (腾祥字型), free for personal use | installed family name is `Tensentype AiQingJ`, `OS/2.fsType = 0x0008` (Editable embedding) |

The four Maple Mono NF faces, `OFL.txt` and per-file sha256 ship in this repository at
`../fonts/maple-mono-nf/` — nothing is packaged *into* the plugin: see `../fonts/README.md`. The same four
faces are also a release asset (`nebula-aura-fonts-1.0.0-maple-mono-nf.zip`). 腾祥爱情体简 is personal-use
only and must not be redistributed, which is why the repository does not carry it or the CJK-inclusive zip.

**Why nothing is bundled (1.3.0).** The host caps one theme's declared assets at
`THEME_ASSET_MAX_BYTES = 4 MiB` **in total, not per file**. Past the cap it drops *every* asset of
that theme and each `url()` then fails as `INVALID_CSS`, so the theme still loads but with a
fallback font. 1.2.0 shipped 5.25 MiB of woff2 and hit exactly that — `~/.pi-desktop/logs/app/plugin.log`
recorded `5 declared asset(s) ignored` plus four × `theme css may only reference data: urls or
declared assets`. A full CJK face alone is ~1.8 MiB, so bundling both fonts without cutting
coverage was impossible; referencing the installed copies is the clean answer and the package is
back to ~40 KB.

**No `@font-face` is needed** — the platform font list matches both names as written. Verified in
Chromium (canvas render of the same string at 40 px):

| `font-family` used | ink px | pixel hash |
|---|---|---|
| `monospace` (fallback), CJK string | 3682 | 1127999741 |
| `"Tensentype AiQingJ"` | 5552 | 3243865292 |
| `"腾祥爱情体简"` | 5552 | 3243865292 — identical to the installed name |
| `"Maple Mono NF"` (Latin string) | 3045 | 1557115746, vs `monospace` 1894 / 2730184155 |

`!important` on `--font-sans` is deliberate: the host applies **设置 → 字体** by writing
`--font-sans` as an *inline style on `<html>`*, which a plain rule can never outrank. So **while a
Nebula theme is selected, the theme owns the UI font and the Settings font picker has no visible
effect**; switch back to Light / Dark / System to hand control back. Code blocks keep the host's own
`--font-mono`, which this theme does not touch.

Missing font, nothing breaks: the stack falls through `Consolas` → `Courier New` → `monospace` →
the system CJK fallback.

**中文：** 主题只设 `--font-sans`，**不打包字体文件**。宿主限制的是单个主题声明资源的**总字节**（4 MiB），
不是单文件；1.2.0 打了 5.25 MiB woff2 正好超线，于是字体被整批丢弃、`url()` 全部失效、界面回退到系统字体
（日志里就是那两条 `INVALID_ASSET` / `INVALID_CSS`）。一个中文字体本身就 ~1.8 MiB，想在保住完整覆盖的同时塞进
预算是不可能的，所以改成引用本机安装的字体，包体回到 ~40 KB。两个字体的家族名都能被系统字体列表直接命中
（实测见上表，`腾祥爱情体简` 与其内部名 `Tensentype AiQingJ` 渲染像素完全一致），因此连 `@font-face` 都不需要；
没装字体就自动回退，不影响其它配色效果。
## Plugin adaptation (1.6.0) / 插件适配

Marketplace: [`vastsa/pi-desktop-plugins`](https://github.com/vastsa/pi-desktop-plugins) — 22 entries in
`catalog.json` (provider `official`, updated 2026-09-14), sources under `plugins/`, prebuilt
`packages/*.piplug`, MIT licensed. The whole catalog was surveyed: **14 of the 22 plugins consume the
host look** — 13 through the shared adapter `plugins/shared/appearance/` (byte-identical copies that
write `style#pi-appearance-theme-css` and `<html data-theme>`), plus `pi.token-insights` with its own
equivalent boot — and **6 own a palette while ignoring `pluginTheme` entirely**: `pi.file-manager`,
`pi.clipboard-history`, `pi.log-viewer`, `io.github.liushunqiu.pi-idea-git`,
`io.github.muzimu217.deps-audit`, `io.github.muzimu217.session-import`. Four of those six hang their
palette off `data-base`, an attribute the host never sets. So a global theme reaches exactly one of the
two plugins below today, and only one of them needs a plugin-side line to complete the job:

| Plugin | Version | Author | Permissions | Requires | Own palette before adaptation |
|---|---|---|---|---|---|
| `pi.gitlens` (Git Lens) | 0.2.7 | PI-Desktop (official) | `ui.view` | piDesktop >=0.8.0 | neutral: light `#f2f3f5`/`#1a1c1f`, dark `#111111`/`#f5f5f7`, accent = the ink |
| `pi.file-manager` (文件管理器) | 0.3.1 | Tioit-Wang (community), but **shipped built into the host** | `ui.view`, `fs.read` | piDesktop >=0.9.0 | neutral: light `#fafafa`/`#ffffff`/`#1a1c1f`, dark `#181818`/`#ffffff`, accent = the ink |

**How a theme can reach a plugin at all — verified in the installed host (0.14.8, `app.asar`).** The
host never injects CSS into a plugin document: `insertCSS` and `addStyleSheet` have **0 hits** in the
whole asar, and the one style element it injects is `style#pi-plugin-theme` into its *own* renderer. It
offers the active theme instead — `app.getAppearance()` resolves to `{ theme, base, locale,
pluginTheme }`, where `pluginTheme` is exactly `{ id, base, css }` (or `null`) and `css` is the theme's
registered text, sanitized once with a 256 KiB cap (over the cap the theme is dropped, not truncated).
`appearance:changed` is pushed to docked views *and* panel windows with the same object, and no extra
permission is needed. Each plugin then decides whether to apply it — which is what decides what is live
today.

* **Git Lens — adapted, live now.** `renderer/appearance-boot.js` writes the received CSS into an
  injected style element and sets `<html data-theme>`, so this sheet is active inside the panel and
  `--ds-*` resolves there. Git Lens paints every surface through 23 short custom props — measured: only
  its two `:root` blocks contain literals, `.chrome` = `var(--bg)`, `.segmented` = `var(--track)`,
  `.group` = `var(--group)`, `.patch .add/.del` = `color-mix(var(--success)/var(--error) 12%)`,
  `.button.primary` = `var(--accent)` + `var(--on-accent)`, `.banner` = `color-mix(var(--error) 12%,
  var(--group))` — so P4a maps exactly those 17 colour props and leaves the 6 geometry/font ones alone.
  Measured before → after in a reconstructed Git Lens document: header `#f2f3f5` → `#F7F6FC`, cards
  `#ffffff` → `#FCFBFF`, diff panel `#eef0f2` → `#D8D5E6`, added line `#248a3d` → `#3D9A6E`, deleted
  line `#d70015` → `#C04A5A`, primary button `#1a1c1f`/`#ffffff` → `#6B5BD4`/`#F2F1F8` (Violet
  `#1F1A2E`/`#211C31`/`#A78BFA`/`#171320`, Midnight `#0F1628`/`#111A2F`/`#7C8CFF`/`#0B1020`), plus
  separator / hover / track / thumb and the four ink levels — and it now resolves the same way whether
  the injected sheet lands before or after the plugin's own two stylesheets.
* **File Manager — 19 props mapped, dormant until the plugin reads the payload.** It calls
  `app.getAppearance()` but keeps only `base` (its own fallback is `prefers-color-scheme`), writes
  `documentElement.dataset.base` and never `data-theme`; on top of that its bundle inlines its own
  stylesheet into a `<style>` it appends at init. So **no** theme can reach that document today. It is
  also shipped *built into the host* (`resources/plugins/pi.file-manager`, 0.3.1, byte-identical to the
  marketplace copy apart from line endings), so making it theme-aware is an upstream change, not a
  marketplace install. P4b nevertheless maps its 19 props with this variant's **literal** palette —
  `var(--ds-*)` cannot resolve there (measured: `--ds-bg-primary` comes back empty inside that
  document) — gated on its unique `.tree-size`. 1.6.0 closed the two gaps the plugin's own CSS exposed:
  the JSON viewer's four syntax colours (`--json-key` / `-string` / `-number` / `-literal`, where the
  plugin hardcodes Atom One's `#e45649` / `#50a14f` / `#986801` / `#a626a4`, so nothing else reached
  them) and the one component literal next to them, the expand row `.json-more` = `#4aa3d9`. Measured
  with the plugin's own stylesheet: `--bg` `#181818` → `#F7F6FC`, `--fg` `#ffffff` → `#2E2D3A`,
  `--accent` `#ffffff` → `#6B5BD4`, `--json-key` `#e45649` → `#BD4253`, `.json-more` `#4aa3d9` →
  `#6B5BD4`. It activates unchanged the moment the plugin applies the CSS it already receives:

  ```js
  // pi.file-manager — apply the active host theme, the way pi.gitlens does
  const css = appearance?.pluginThemeCss || appearance?.pluginTheme?.css;
  if (css) {
    const id = "pi-appearance-theme-css";
    let el = document.getElementById(id);
    if (!el) document.head.appendChild(Object.assign(document.createElement("style"), { id })),
      el = document.getElementById(id);
    el.textContent = css;
  }
  if (appearance?.base) document.documentElement.dataset.theme = appearance.base;
  ```

* **P4b cascade — fixed in 1.6.0, and it was a real bug.** The gate used to be
  `:root:has(.tree-size)` (0,2,0), which **ties** with the plugin's own `:root[data-base=light]` block
  (0,2,0): whichever `<style>` lands later wins on document order, and the plugin appends its own at
  init — so a sheet injected before it lost the *entire* light mapping. Reproduced in Chromium 148
  against the plugin's real stylesheet (old selector first: `--bg` `#fafafa`, `--json-key` `#e45649`,
  i.e. the stock palette; `:root[data-base]:has(.tree-size)` in the same position: `--bg` `#F7F6FC`,
  `--json-key` `#BD4253`). The mapping now carries `[data-base]` (0,3,0) and wins in both orders on all
  four variants; the ungated selector stays for the frame before `data-base` exists, where the only
  competitor is the plugin's own `:root` block (0,1,0). The three dark-based variants never showed the
  bug — which is exactly why it stayed invisible until the two orders were measured side by side.
* **The JSON hues are contrast-checked.** Light takes the same H and S as `--ds-error` / `--ds-success`
  / `--ds-warning` / `--ds-purple` with the lightness reduced until each clears 4.8:1 on `--bg`
  (measured 4.80 / 4.83 / 4.84 / 4.84:1 — the raw tokens land at 4.47 / 3.23 / 3.26 / 4.34:1, under the
  4.5:1 bar for 11.5 px code text; the plugin's stock Atom One values measure 3.41 / 2.98 / 4.53 /
  5.69:1 on the same background, so they beat the tokens on two roles and lose on two). The dark-based
  variants use their tokens verbatim: 4.76 / 8.99 / 7.48 / 6.02:1 (Dark), 5.96 / 10.97 / 8.08 / 5.50:1
  (Violet), 6.51 / 11.82 / 10.79 / 6.82:1 (Midnight). `.json-more` follows `--accent` at 4.80 / 5.85 /
  6.20 / 6.05:1.
* **P4c** keeps this sheet's root aura / star field out of plugin documents (they set `data-theme` too,
  and Git Lens has its own surfaces).
* **P4d** re-points the launcher's `pi-plugin-panel-chrome` page colours. Verified mechanism: the
  preload *samples* the plugin document's own `body` / `<html>` background for those two custom
  properties and only falls back to stock `#ffffff` / `#181818`, writing them **inline** — an author
  `!important` outranks an inline declaration, so the capsule and the page behind a view follow
  `--ds-bg-dock` / `--ds-text-primary` whatever was sampled.

**No leakage, measured.** Those prop names are generic and the host *does* ship Tailwind utilities that
read `var(--fg)` and `var(--surface-hover)`, so every short-name mapping is gated on a DOM signature that exists in
exactly one document — `.segmented` (Git Lens; 0 hits in the host sheet) and `.tree-size` (File Manager;
0 hits). All 30 short names declared across the four sheets were probed in a host-like document: every
one of them comes back empty, and the new `.json-more` rule leaves a host element with the same class at
the browser default. In the plugin documents the same names all resolve. Re-measured after the 1.6.0
follow-ups **with a positive control**, so the probe cannot pass vacuously: adding a `.segmented` element
to that document resolves 17 of the names, a `.tree-size` element 19, and removing it returns the count
to 0.
* **1.6.0 follow-ups — three defects the mapping itself had, all measured in Chromium 148.** (1) *The
  host backdrop leaked its colour into plugin documents.* `html[data-theme=…] body` (0,1,2) also set
  `background-color: var(--ds-bg-primary)`, which outranks the plugin's own
  `body { background: var(--bg) }` (0,0,1), so Git Lens was painted the *host's* page colour instead
  of the mapped one — measured `#F2F1F8` / `#1A1B26` / `#171320` / `#0B1020` where the palette says
  `#F7F6FC` / `#222433` / `#1F1A2E` / `#0F1628`. P4c cancelled the gradient but not the colour; a
  gated `background-color: var(--bg)` rule (0,3,1) now cancels both, and deleting that one rule
  through CSSOM brings the bleed back in all four variants. (2) *`.md-link` was left behind.* The file
  manager's markdown preview hardcodes the same `#4aa3d9` as its JSON expand row
  (`views/assets/index.js`: `.md-link{color:#4aa3d9}` next to `.json-more{…color:#4aa3d9}`), so it now
  shares the `var(--accent)` rule. Two literals stay out of CSS reach on purpose: the JS file-icon map
  (only `default` / `folder` / `doc` go through variables) and CodeMirror's `.cm-searchMatch` yellow.
  (3) *The panel-chrome colours could vanish.* `pi-plugin-panel-chrome` is created by the host's panel
  preload, which writes `--pi-plugin-panel-page-background/-foreground` **inline** from the stock
  surface and stamps the element — not `<html>` — with its own `data-theme`. Where `--ds-*` is undefined
  (a theme-aware file manager sets only `data-base`) the `!important` re-pointing became
  invalid-at-computed-value-time and erased those inline colours instead of leaving them alone —
  reproduced as `""`. Both declarations now carry this variant's literal as a fallback and always
  resolve.
* **Known limit of the File Manager gate.** `.tree-size` is rendered on file rows only, so P4b is inert
  in states that render no row (empty workspace, first async frame). The gate is deliberately left as
  is: it is the one signature that exists in exactly one document, and widening it (`.tree-row`,
  `.md-body`, …) would trade that guarantee away for a mapping that is dormant anyway.

**中文：** 宿主不会往插件文档注入 CSS（整个 app.asar 里 `insertCSS`/`addStyleSheet` 命中为 0，它只在自己的
渲染进程注入 `style#pi-plugin-theme`），而是通过 `app.getAppearance()` 交出 `{ theme, base, locale,
pluginTheme{id,base,css} }`，用不用由插件自己决定 —— 这一点决定了现在能生效多少。为此把市场 22 个条目全盘查了
一遍：14 个会应用宿主外观（13 个走共享适配器 `plugins/shared/appearance/`，外加自带同样逻辑的
`pi.token-insights`），6 个自成一色且完全忽略 `pluginTheme`（文件管理器、剪贴板历史、日志查看器、IDEA Git、
deps-audit、session-import），其中 4 个的色板挂宿主从不设置的 `data-base` 上。所以下面两个插件里，今天真正生效
的只有 Git Lens，另一个只差插件侧一行。Git Lens 用了（boot 脚本注入 style 并设 `data-theme`），四套配色在两种
注入顺序下都实测生效。文件管理器只读 `base`、写 `data-base`，而且它是**随宿主内置**的
（`resources/plugins/pi.file-manager`，与市场副本仅换行符不同），因此仍是休眠映射：1.6.0 补上了 JSON 视图漏掉
的 4 个语法色（插件自己写死 Atom One 配色）和它旁边唯一一个组件字面量 `.json-more`，并按实测把 light 套这四个
颜色压暗到 4.8:1（原令牌只有 3.2–4.5:1，11.5px 代码字不够读；三套深色直接沿用令牌值）。还修掉一个真实隐患：旧
门控 `:root:has(.tree-size)`（0,2,0）与插件自己的 `:root[data-base=light]` **同分**，样式表若排在插件的
`<style>` 之前，light 基线会整套失效 —— 已在 Chromium 148 里用插件真实样式表复现（`--bg` 仍是 `#fafafa`、
`--json-key` 仍是 `#e45649`），改成 `:root[data-base]:has(.tree-size)`（0,3,0）后两种顺序都赢；三套深色基线从
来不中招（它们的块是 `:root`，0,1,0），这正是它一直没被发现的原因。泄漏检查也扩到全部 30 个短名：宿主文档里一
个都不解析，`.json-more` 对宿主同名元素无效；插件文档里全部生效。1.6.0 又量出并修掉三处自身缺陷：宿主背景色不再把
`--ds-bg-primary` 渗进插件文档（Git Lens 的 body 实测由 `#F2F1F8`/`#1A1B26`/`#171320`/`#0B1020` 改回
`#F7F6FC`/`#222433`/`#1F1A2E`/`#0F1628`）；文件管理器 markdown 预览的 `.md-link` 与 JSON 展开行共用同一个写死的
`#4aa3d9`，现在一并走 `var(--accent)`；面板 chrome 的两个页面色补了本套字面色兜底，避免在只设 `data-base` 的文档里
`var()` 解析失败反而抹掉宿主内联色。三项均在 Chromium 148 实测通过（12/12），30 个短名在宿主文档里仍为 0 命中。

## Reverting / 恢复

Pick Light / Dark / System in the theme picker — the plugin can stay installed without effect, or uninstall it. Each variant is independent, so switching between Nebula themes is one click.

在主题选择器中切回浅色 / 深色 / 系统即可；插件保留也无副作用，或直接卸载。四套配色互相独立，切换只需一次点击。

## Permissions / 权限

| Permission | Why / 原因 |
|---|---|
| `ui.theme` | Contribute the global themes / 贡献全局主题 |

## Safety / 安全

CSS is sanitized by the host on load (size limit, no `@import`, no markup, no scripts, no external URLs). This sheet contains **no `url()` at all** and declares no theme assets — the backdrop is pure gradients, the font comes from the installed system fonts — so there is nothing for the host to fetch or route. The plugin makes no network requests and stores no data.

CSS 由宿主在加载时净化（字节上限、禁止 `@import`/标记/脚本/外部 URL）。本主题**完全不含 `url()`**、也不声明任何主题资源：背景是纯渐变，字体取自本机已安装字体，宿主没有任何需要抓取或改写的资源，插件不做任何网络请求。

## Credit / 致谢

Colour values come from **Nebula Aura Theme** © Luis Penholato, MIT licensed (<https://github.com/luismpenholato/nebula-aura-theme>). This is an unofficial port to PI-Desktop, not a VS Code extension. See [NOTICE](NOTICE).

配色取自 **Nebula Aura Theme**（© Luis Penholato，MIT 许可），本仓库是非官方的 PI-Desktop 移植版。
