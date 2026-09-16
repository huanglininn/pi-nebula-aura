# Fonts / 字体

`pi.nebula-aura` pins the UI font to two families:

```css
--font-sans: Maple Mono NF,腾祥爱情体简, Consolas, 'Courier New', monospace !important;
```

This folder ships **one** of them — the one whose licence allows redistribution — plus the install
notes for the other.

| Family | In this repo | Licence | Get it from |
|---|---|---|---|
| **Maple Mono NF** — Regular / Bold / Italic / BoldItalic, v7.000 | ✅ [`maple-mono-nf/`](maple-mono-nf), 4 faces ≈ 2.09 MB each | **SIL OFL 1.1** — redistributable (keep [`OFL.txt`](maple-mono-nf/OFL.txt) and the copyright line) | upstream: [subframe7536/maple-font](https://github.com/subframe7536/maple-font), release asset `Maple Mono NF` |
| **腾祥爱情体简** — internal family name `Tensentype AiQingJ`, v1.00 | ❌ **not included** | free for **personal use**, **no redistribution grant** — the file embeds no licence text, copyright `Beijing Tensentype Technology Co.,Ltd. 2017`, `OS/2.fsType = 0x0008` (Editable embedding) | vendor: <https://www.tensentype.com> |

The CJK family is deliberately absent. The theme only *references* it by family name, and redistributing a
font whose grant is "personal use" would be over-distribution — so install it from the vendor if you want
the intended CJK face. The themes work without it.

`OS/2.fsType` is often mistaken for a redistribution flag: it only says whether a *document* may embed the
font. Redistribution is governed by the licence, and the two are independent.

## Files / 文件与校验

| File | Bytes | sha256 |
|---|---|---|
| `maple-mono-nf/MapleMono-NF-Regular.ttf` | 2,087,456 | `9f2fb964a22de7a97defa50b61a69ced3e94e6d078dedbfb397b68f511d3cbaf` |
| `maple-mono-nf/MapleMono-NF-Bold.ttf` | 2,087,748 | `6ea26958de57b656fc78e35a410522526e5623374669be9ae6cf065bf4a18fc6` |
| `maple-mono-nf/MapleMono-NF-Italic.ttf` | 2,107,208 | `c0575bc9cd7f2ccba2634cb53d91a3238438e7e731164af8691fda958fba70ec` |
| `maple-mono-nf/MapleMono-NF-BoldItalic.ttf` | 2,107,516 | `d29084aec14c97510ac80bfa4e1eb125e38d79cba819421cc241daec3b401d69` |
| `maple-mono-nf/OFL.txt` | 4,406 | — (SIL OFL 1.1 text as published upstream, incl. Reserved Font Name `Maple Mono`) |

The four faces above are the ones the theme's README names; the upstream release also ships the other 12
weights (Thin … ExtraBold, plus italics) if you want them.

## Install / 安装（Windows）

1. Select the `.ttf` files → right click → **Install** (current user) or **Install for all users**.
2. Restart PI-Desktop afterwards: the renderer enumerates system fonts at start-up, so a hot install is
   not picked up.
3. Missing fonts are harmless — the stack falls through to `Consolas → Courier New → monospace` and the
   system CJK face.

Manual per-user install (no admin rights) is just "copy into the user font directory + register the file
name". Add the CJK face you downloaded from the vendor the same way on the last line:

```powershell
$dst = "$env:LOCALAPPDATA\Microsoft\Windows\Fonts"
Copy-Item .\maple-mono-nf\*.ttf $dst -Force
$key = "HKCU:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts"
New-ItemProperty $key -Name "Maple Mono NF Regular (TrueType)"     -Value "$dst\MapleMono-NF-Regular.ttf"    -PropertyType String -Force
New-ItemProperty $key -Name "Maple Mono NF Bold (TrueType)"        -Value "$dst\MapleMono-NF-Bold.ttf"       -PropertyType String -Force
New-ItemProperty $key -Name "Maple Mono NF Italic (TrueType)"      -Value "$dst\MapleMono-NF-Italic.ttf"     -PropertyType String -Force
New-ItemProperty $key -Name "Maple Mono NF Bold Italic (TrueType)" -Value "$dst\MapleMono-NF-BoldItalic.ttf" -PropertyType String -Force
# 腾祥爱情体简 / Tensentype AiQingJ (download from tensentype.com first):
# New-ItemProperty $key -Name "腾祥爱情体简 (TrueType)" -Value "$dst\腾祥爱情体简.ttf" -PropertyType String -Force
```

## Why the fonts are not bundled *into* the plugin / 为什么不随插件打包

The host caps one theme's **declared assets** at `THEME_ASSET_MAX_BYTES = 4 MiB` *in total, not per file*.
Past the cap it drops every asset of that theme and each `url()` then fails as `INVALID_CSS` — the theme
still loads, but silently with a fallback font (release 1.2.0 shipped 5.25 MiB of woff2 and hit exactly
that). Measured subsets do not rescue it either:

| Option | Measured |
|---|---|
| both families in full | 12.3 MB — impossible |
| Maple Mono NF, Latin + icons only, one face as woff2 | 0.90 MB → 3.58 MB for four faces |
| 腾祥爱情体简, GB2312 common set (7,445 chars) as woff2 | 1.86 MB (3.88 MB in full) |

So the theme references the *installed* families by name and ships nothing — the only combination that
keeps full coverage without touching the cap.

## Licence / 许可

* `maple-mono-nf/` — **SIL OFL 1.1** ([`OFL.txt`](maple-mono-nf/OFL.txt)). Keep the licence and the
  copyright line (`Copyright 2022 The Maple Mono Project Authors`) when redistributing; renaming or
  modifying the files invokes the Reserved Font Name clause for `Maple Mono`.
* Everything else in this repository — **MIT**, see [../LICENSE](../LICENSE). The CJK family above is
  governed by its own vendor terms, not by either licence here.
