---
name: coui-ui-design
description: Build 1:1 authentic ColorOS 16/17 (COUI) native Android user interfaces in Jetpack Compose or native com.coui.appcompat XML widgets. Use when asked to create or restyle COUI, ColorOS, ColorOS 17, Oplus, or OnePlus/OPPO UIs, Settings screens, COUI-Kit, suqi8/coui, ONextBox, AndroidLiquidGlass floating docks, Capsule squircle cards, or inject COUI Preference XML into system APKs.
---

# ColorOS 17 (COUI) UI Design & Engineering Skill

Build native **ColorOS 16/17 (COUI)** interfaces that feel indistinguishable from OPPO / OnePlus system apps—never generic Material 3 (`12dp` rounded cards, `M3` ripple rectangles, or white page backgrounds).

## 1. Choose Your Implementation Track

| Scenario | Primary Track | Reference |
|---|---|---|
| Standalone / module companion Android app in Jetpack Compose | `ItosEO/COUI-Kit` (`io.github.itoseo:coui`) or `suqi8/coui` + `Kyant0/Capsule` + `Kyant0/AndroidLiquidGlass` | [compose-components.md](skills/coui-ui-design/references/compose-components.md) |
| Custom ColorOS Settings / System APK / Xposed / Magisk-or-KernelSU XML & smali UI injection | Native `com.coui.appcompat.*` Preference XML + `COUISectionSeekBar` + `Theme.COUI` attrs | [native-xml-widgets.md](skills/coui-ui-design/references/native-xml-widgets.md) |
| Color palette, dark AMOLED `#000000` vs `#F0F1F2` canvas, typography, squircle radii, ROM haptics | COUI Light / Dark / Material You tokens & haptics (`302`, `307`, `308`) | [design-tokens.md](skills/coui-ui-design/references/design-tokens.md) |
| App icons (Classic + ColorOS Flux `<monochrome>`), press springs, bilingual EN/ID UI strings | `1500×1500` viewport `250px` safe inset, `CouiPressMotion`, `CouiSpring` | [icons-and-motion.md](skills/coui-ui-design/references/icons-and-motion.md) |

## 2. Six Non-Negotiable ColorOS 17 Rules

1. **Inverted Background Hierarchy ("Gray Canvas + White Cards" in Light, "Pure Black + Translucent White Cards" in Dark):**
   - Page canvas (`background` / `surface` scaffold): **`#F0F1F2`** (`#F7F8FA` in Settings subpages) in Light; **`#000000`** AMOLED black in Dark.
   - Grouped & standalone preference cards (`card`): **`#FFFFFF`** in Light; **`#1AFFFFFF`** (`10%` white over `#000000`) in Dark. Never tint active cards gray (`#EBEDF0`); reserve neutral `#E6E6E6` / `#33FFFFFF` strictly for pressed highlights or footer suggestion blocks.
   - Dialogs & bottom sheet panels (`surface` / `background` dialog panel): **`#FFFFFF`** in Light; **`#1E1E1E`** (`#333333` for floating popups/menus) in Dark.
2. **Continuous Squircle Geometry Everywhere (`Capsule` / `CouiShape`):**
   - Never use flat circular `RoundedCornerShape` on cards, sheets, or dialogs.
   - Use `CouiShape(24.dp, cornerWeight = 3f)` (`COUI-Kit`) or `ContinuousRoundedRectangle(20.dp)` (`Kyant0/Capsule` — visually equivalent to legacy `24.dp` COUI curves) for cards; `28.dp` continuous corners for bottom sheets/dialogs; `Capsule()` for buttons and floating docks; and `40.dp` `CircleShape` for top-bar back/action buttons.
3. **ColorOS 17 Switch & COUI Blue (`#0080FF` / `#1A8CFF`):**
   - ColorOS 17 switches are **`44.dp × 24.dp`** (`18.dp` white thumb, `3.dp` padding, `1.3x` horizontal stretch at toggle peak `p ≈ 0.347`, `toggleResponse = 0.4f`, `toggleBounce = 0.3f`), wider than ColorOS 16's `38.dp × 24.dp` (or `49×28` miuix compat).
   - Always make the **entire row** `toggleable(role = Role.Switch, indication = null)` sharing a `MutableInteractionSource` with the `Switch` (`onCheckedChange = null`).
4. **Top Bars & Liquid Glass Floating Docks:**
   - Detail subpages use a **`52.dp`** toolbar below status bar insets with a `40.dp` circular frosted back button (`ColorOsTopBarButton`), `18.dp.toSp()` `FontWeight(550)` title, and a `0.33.dp`–`0.5.dp` scroll-revealed hairline divider.
   - Dark media overlays & floating bottom docks use `AndroidLiquidGlass` (`vibrancy()` + `blur()` + subtle `lens()`) and `ColorOsToolbarGlass` (`BlurRadiusPx = 25f`).
5. **Authentic Motion & ROM Haptics (Zero Material Ripples):**
   - Disable rectangular Material ripples (`indication = null`) on cards and switches; use **Tint** feedback on list rows and `CouiPressMotion` scale springs on cards/buttons.
   - Trigger ColorOS ROM haptics: **`302`** (`CouiHapticFeedback.Switch`), **`307`** (`CouiHapticFeedback.ScrollEdge`), **`308`** (wheel/slider detents), with safe Android fallback.
6. **Clean Dividers & Edge-to-Edge Insets:**
   - Draw `0.33.dp` (`>= 1px`) hairline dividers (`#1F000000` light / `#33FFFFFF` dark), inset `16.dp` horizontally, **strictly between** adjacent rows inside multi-row cards—never at card top/bottom edges.
   - Pad scrollable lists and floating bars cleanly for `WindowInsets.statusBars`, `navigationBars`, and `ime`.
