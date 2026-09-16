# Nebula Aura Theme for PI-Desktop / 星云光晕主题

Four **Nebula Aura** colour themes — **Light · Dark · Violet · Midnight** — for
[PI-Desktop](https://github.com/vastsa/PI-Desktop), ported from the VS Code theme pack
[nebula-aura-theme](https://github.com/luismpenholato/nebula-aura-theme) onto PI-Desktop's own
`--ds-*` design tokens. Styling only: no features, no network, no file or clipboard access.

为 [PI-Desktop](https://github.com/vastsa/PI-Desktop) 做的四套 **Nebula Aura** 配色，把 VS Code 主题包
[nebula-aura-theme](https://github.com/luismpenholato/nebula-aura-theme) 的调色板移植到 PI-Desktop 的设计令牌
（`--ds-*`）上。纯样式：无额外功能、无网络、无文件/剪贴板访问。

| | |
|---|---|
| Plugin id / version | `pi.nebula-aura` **1.6.0** |
| Permission | `ui.theme` — contributes four global themes, CSS only |
| Requires | PI-Desktop `>= 0.4.3` |
| Package | [`pi.nebula-aura/dist/pi.nebula-aura-1.6.0.piplug`](pi.nebula-aura/dist) |
| Licence | MIT for the theme; palette © Luis Penholato (MIT); bundled font under **SIL OFL 1.1** — see [NOTICE](pi.nebula-aura/NOTICE) |

## Install / 安装

1. **Fonts (optional, but the themes are designed around them).** The stack is
   `Maple Mono NF, 腾祥爱情体简, Consolas, 'Courier New', monospace`:
   * **Maple Mono NF** (Regular / Bold / Italic / BoldItalic, v7.000) ships in
     both [`fonts/maple-mono-nf/`](fonts/maple-mono-nf) and the release asset
     `nebula-aura-fonts-1.0.0-maple-mono-nf.zip` — SIL OFL 1.1, redistributable.
   * **腾祥爱情体简** is **not** included: it is free for personal use but carries no redistribution
     grant. Get it from <https://www.tensentype.com>.
   * Fonts are referenced by family name only (no `@font-face`, nothing bundled *into* the plugin):
     the host caps a theme's declared assets at 4 MiB in total, which two full families cannot fit.
     See [fonts/README.md](fonts/README.md) and the plugin README.
   * Missing fonts are harmless — the stack falls through to `Consolas → Courier New → monospace`
     plus the system CJK face.
2. **Install the plugin** — Settings → Plugins → *Install from folder* → pick `pi.nebula-aura`, or
   install the packaged `.piplug`: `pi.nebula-aura/dist/pi.nebula-aura-1.6.0.piplug`, the same build
   as the [latest release](https://github.com/huanglininn/pi-nebula-aura/releases/latest).
3. **Choose a theme** — Settings → Theme → search `Nebula`.

## Variants / 四套配色

| Theme | Base | Backdrop | Accent |
|---|---|---|---|
| Nebula Aura Light | light | lavender aura + star field | `#6B5BD4` |
| Nebula Aura Dark | dark | blue/pink aura + star field | `#8B9FD4` |
| Nebula Aura Violet | dark | violet/magenta aura + star field | `#A78BFA` |
| Nebula Aura Midnight | dark | cyan/blue aura + star field | `#7C8CFF` |

Offline mock-up page — all four variants in one HTML file, no network, no webfont:
[`preview/index.html`](preview/index.html).

## What it covers / 覆盖范围

* Every host design token (`--ds-*`) **plus** the ~36 hardcoded neutrals the host stylesheet leaves
  behind — audited rule by rule against the shipped `app.asar` sheet and measured in Chromium.
* Four colour tokens the host reads but never defines (`--ds-danger`, `--ds-focus`, `--ds-text`,
  `--ds-text-tertiary`) — without them, mermaid error strips, focus rings and two labels silently
  lose their colour.
* Two marketplace plugin panels: **pi.gitlens** (live — it applies the active theme inside its own
  document) and **pi.file-manager** (mapping complete and gated on its own DOM signature, but dormant
  until the plugin reads the `pluginTheme` payload the host offers; the plugin README carries the
  patch).
* Nothing else: every plugin-facing rule is gated on a DOM signature that exists in exactly one
  document, so none of the 30 short custom-property names resolve in the host document.

## Repository layout / 仓库结构

```text
pi.nebula-aura/     the plugin — manifest.json, main.js, themes/*.css, README, NOTICE, dist/*.piplug
fonts/              Maple Mono NF (OFL) + install notes; 腾祥爱情体简 deliberately not included
preview/            offline mock-ups, one HTML page per variant
```

## Credits / 致谢

* **Palette** — [Nebula Aura Theme](https://github.com/luismpenholato/nebula-aura-theme) © Luis
  Penholato, MIT. This is an unofficial PI-Desktop port; no VS Code sources are redistributed
  (the port re-targets the palette onto PI-Desktop's tokens and writes new CSS) — see
  [NOTICE](pi.nebula-aura/NOTICE).
* **Fonts** — [Maple Mono](https://github.com/subframe7536/maple-font) © The Maple Mono Project
  Authors (subframe7536), SIL OFL 1.1.
* **Plugin interop** — the two adapted panels belong to
  [vastsa/pi-desktop-plugins](https://github.com/vastsa/pi-desktop-plugins) (MIT). Only their public
  class and custom-property names are referenced; no plugin code, CSS or asset is copied.

## Licence / 许可

MIT — see [LICENSE](LICENSE). The font under `fonts/` is licensed under **SIL OFL 1.1**, not MIT.
