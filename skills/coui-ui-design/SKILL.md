---
name: coui-ui-design
description: Build 1:1 authentic ColorOS 16/17 (COUI) native Android user interfaces in Jetpack Compose or native com.coui.appcompat XML widgets. Use when asked to create or restyle COUI, ColorOS, ColorOS 17, Oplus, or OnePlus/OPPO UIs, Settings screens, COUI-Kit, suqi8/coui, ONextBox, AndroidLiquidGlass floating docks, Capsule squircle cards, or inject COUI Preference XML into system APKs.
---

# ColorOS 17 (COUI) UI Design & Engineering Skill

Build native **ColorOS 16/17 (COUI)** interfaces that feel indistinguishable from OPPO / OnePlus system apps—never generic Material 3 (`12dp` rounded cards, `M3` ripple rectangles, heavy outlined action pills, or white page backgrounds).

## 1. Choose Your Implementation Track

| Scenario | Primary Track | Reference |
|---|---|---|
| Standalone / module companion Android app in Jetpack Compose | `ItosEO/COUI-Kit` (`io.github.itoseo:coui`) or `suqi8/coui` + `Kyant0/Capsule` + `Kyant0/AndroidLiquidGlass` | [compose-components.md](references/compose-components.md) |
| Custom ColorOS Settings / System APK / Xposed / Magisk-or-KernelSU XML & smali UI injection | Native `com.coui.appcompat.*` Preference XML + `COUISectionSeekBar` + `Theme.COUI` attrs | [native-xml-widgets.md](references/native-xml-widgets.md) |
| Color palette, dark AMOLED `#000000` vs `#F5F6F8` canvas, typography, squircle radii, ROM haptics | COUI Light / Dark / Material You tokens & haptics (`302`, `307`, `308`) | [design-tokens.md](references/design-tokens.md) |
| App icons (Classic + ColorOS Flux `<monochrome>`), press springs, rubber-band overscroll, bilingual EN/ID | `1500×1500` viewport `250px` safe inset, `CouiPressMotion`, `couiBouncyOverscroll` | [icons-and-motion.md](references/icons-and-motion.md) |

## 2. Seven Non-Negotiable ColorOS 17 Rules

1. **Inverted Background Hierarchy ("Cool Off-White Canvas + Pure White Cards" in Light, "Pure Black + Elevated Dark Cards" in Dark):**
   - Page canvas (`background` / `surface` scaffold): **`#F5F6F8`** (Settings subpages; `#F0F1F2`–`#F0F1F3` system canvas) in Light; **`#000000`** AMOLED black in Dark.
   - Grouped & standalone preference cards (`card`): **`#FFFFFF`** in Light; **`#1C1C1E`** (or `#1AFFFFFF` 10% white over `#000000`) in Dark. Never tint active cards gray (`#EBEDF0`); reserve neutral `#E8E9EB` / `#2C2C2E` strictly for pressed highlights or footer suggestion blocks.
   - Dialogs & bottom sheet panels (`surface` / `background` dialog panel): **`#FFFFFF`** in Light; **`#1C1C1E` / `#1E1E1E`** (`#333333` for floating popups/menus) in Dark.
2. **Continuous Squircle Geometry & Compact Text-First Card Hierarchy (`Capsule` / `CouiShape`):**
   - Never use flat circular `RoundedCornerShape` on cards, sheets, or dialogs. Use `ContinuousRoundedRectangle(20.dp)` (`Kyant0/Capsule`) or `CouiShape(24.dp, cornerWeight = 3f)` (`COUI-Kit`) for cards; `28.dp` continuous corners for bottom sheets/dialogs; `Capsule()` for buttons and floating docks.
   - **No redundant left icon boxes** on standard Settings subpage rows: primary rows start cleanly at `16.dp` horizontal padding (`14.dp` vertical padding) inside `16.dp`-margined cards. Section headers (`13.sp` Medium `#7F7F7F`) align cleanly at `32.dp` total inset from the screen edge.
   - Use **borderless right-aligned action/value labels** (`#8C8C8C` for status values, `#0080FF` for inline actions like `"Copy"`) rather than heavy outlined pill boxes, and keep status pills (`"eSIM"`, `"Active"`) sleek and low-profile (`10.5.sp` Medium, subtle translucent tint).
3. **ColorOS 17 Switch & COUI Blue (`#0080FF` / `#1A8CFF`):**
   - ColorOS 17 switches are **`44.dp × 24.dp`** (`18.dp` white thumb, `3.dp` padding, `#0080FF` checked / `#D8DADC` unchecked in light, `1.3x` horizontal stretch at toggle peak `p ≈ 0.347`, `toggleResponse = 0.4f`, `toggleBounce = 0.3f`).
   - Always make the **entire row** `toggleable(role = Role.Switch, indication = null)` sharing a `MutableInteractionSource` with the `Switch` (`onCheckedChange = null`) and triggering ROM haptic **`302`**.
4. **Top Bars (`42–44dp` Frosted-Glass Back Button) & Liquid Glass Floating Docks:**
   - Detail subpages float a **`60.dp`** top bar below `WindowInsets.statusBars` with a **`42.dp`–`44.dp` circular frosted-glass back button** (`#E6FFFFFF` light / `#26FFFFFF` dark, `0.5.dp` specular hairline border, soft ambient shadow `elevation = 4.dp–6.dp`, and a larger `22.dp–24.dp` bold `2.dp`-stroke back arrow), `19.5.sp` `FontWeight.Medium` title, and **no right-bar clutter** on detail subpages (avoid redundant refresh/overflow icons).
   - Dark media overlays & floating bottom docks use `AndroidLiquidGlass` (`vibrancy()` + `blur()` + subtle `lens()`) and `ColorOsToolbarGlass` (`BlurRadiusPx = 25f`).
5. **ColorOS 17 Rubber-Band Bouncy Overscroll (Single-Screen & Multi-Screen):**
   - Always disable AOSP `StretchOverscroll` (`CompositionLocalProvider(LocalOverscrollFactory provides null)`) so dragging past top/bottom boundaries never distorts text vertically.
   - Wrap scrollable lists in **`Modifier.couiBouncyOverscroll()`** (`NestedScrollConnection` + `Animatable` `translationY` with rubber-band damping `maxOffset * (1 - exp(-|drag| / k))` and `spring(dampingRatio = 0.72f, stiffness = 380f)` rebound on fling or pointer release) so both single-screen and multi-screen lists bounce authentically.
6. **Authentic Motion & ROM Haptics (Zero Material Ripples):**
   - Disable rectangular Material ripples (`indication = null`) on cards and switches; use **Tint** feedback on list rows and `CouiPressMotion` scale springs on cards/buttons.
   - Trigger ColorOS ROM haptics: **`302`** (`CouiHapticFeedback.Switch`), **`307`** (`CouiHapticFeedback.ScrollEdge`), **`308`** (wheel/slider detents), with safe Android fallback (`CLOCK_TICK`).
7. **Inset Hairline Dividers & Edge-to-Edge Insets:**
   - Draw **`1.dp` `#14000000`** (light) / **`#24FFFFFF`** (dark) internal dividers inset **`16.dp`** horizontally (aligning with row text, never edge-to-edge) **strictly between** adjacent rows inside multi-row cards—never at card top/bottom edges.
   - Pad scrollable lists and floating bars cleanly for `WindowInsets.statusBars`, `navigationBars`, and `ime`.

## 3. Pre-Release Visual Parity Audit

Before considering any ColorOS 17 UI complete:
1. Check Light & Dark modes side-by-side (`#F5F6F8`/`#FFFFFF` vs `#000000`/`#1C1C1E`).
2. Verify `42–44dp` frosted-glass circular back button with soft ambient shadow, crisp `22–24dp` back arrow, and clean top bar with zero unnecessary right-side icons.
3. Verify zero redundant left icon boxes on standard preference rows, zero heavy outlined inline action pills, inset `1dp` `#14000000` dividers, and `20.dp` continuous squircle corners.
4. Verify smooth rubber-band top/bottom bouncy overscroll (even when all rows fit on one screen) with zero vertical text stretching.
5. Verify edge-to-edge clearance under top status bar cutout and bottom gesture pill, plus Flux monochrome launcher icon silhouette inside the `108dp` safe circle.
