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
| Color palette, dark AMOLED `#000000` vs `#F0F1F3` canvas, typography, squircle radii, ROM haptics | COUI Light / Dark / Material You tokens & haptics (`302`, `307`, `308`) | [design-tokens.md](references/design-tokens.md) |
| App icons (Classic + ColorOS Flux `<monochrome>`), press springs, rubber-band overscroll, bilingual EN/ID | `1500×1500` viewport `250px` safe inset, `CouiPressMotion`, `couiBouncyOverscroll` | [icons-and-motion.md](references/icons-and-motion.md) |

## 2. Seven Non-Negotiable ColorOS 17 Rules

1. **Inverted Background Hierarchy ("Cool `#F0F1F3` Off-White Canvas + Pure White Cards" in Light, "Pure Black + Elevated Dark Cards" in Dark):**
   - Page canvas (`background` / `surface` scaffold): **`#F0F1F3`** (calibrated 1:1 against ColorOS 17 V6 Settings subpages; never `#F5F6F8` or `#F7F7F7` which wash out card contrast) in Light; **`#000000`** AMOLED black in Dark.
   - Grouped & standalone preference cards (`card`): **`#FFFFFF`** in Light; **`#1C1C1E`** (or `#1AFFFFFF` 10% white over `#000000`) in Dark. Never tint active cards gray (`#EBEDF0`); reserve neutral `#E8E9EB` / `#2C2C2E` strictly for pressed highlights or footer suggestion blocks.
   - Dialogs & bottom sheet panels (`surface` / `background` dialog panel): **`#FFFFFF`** in Light; **`#1C1C1E` / `#1E1E1E`** (`#333333` for floating popups/menus) in Dark.
2. **Continuous Squircle Geometry & Compact Text-First Card Hierarchy (`Capsule` / `CouiShape` — Zero Inline Status Pills):**
   - Never use flat circular `RoundedCornerShape` on cards, sheets, or dialogs. Use `ContinuousRoundedRectangle(20.dp)` (`Kyant0/Capsule`) or `CouiShape(24.dp, cornerWeight = 3f)` (`COUI-Kit`) for cards; `28.dp` continuous corners for bottom sheets/dialogs; `Capsule()` for buttons and floating docks.
   - **No redundant left icon boxes** on standard Settings subpage rows: primary rows start cleanly at `16.dp` horizontal padding (`14.dp` vertical padding) inside `16.dp`-margined cards. Section headers (`13.sp` Medium `#7F7F7F`) align cleanly at `32.dp` total inset from the screen edge.
   - **Never place inline colored status pills** (`"eSIM"`, `"Active"`, `"SIM 2"`, `"Checking"`) next to preference row titles. Express state strictly through the clean row title, `13.sp` `#8C8C8C` summary subtitle, `44.dp × 24.dp` `CouiSwitch`, or borderless right-aligned action/value labels (`#8C8C8C` for status values, `#0080FF` for inline actions like `"Copy"`).
3. **ColorOS 17 Switch & COUI Blue (`#0080FF` / `#1A8CFF`):**
   - ColorOS 17 switches are **`44.dp × 24.dp`** (`18.dp` white thumb, `3.dp` padding, `#0080FF` checked / `#D8DADC` unchecked in light, `1.3x` horizontal stretch at toggle peak `p ≈ 0.347`, `toggleResponse = 0.4f`, `toggleBounce = 0.3f`).
   - Always make the **entire row** `toggleable(role = Role.Switch, indication = null)` sharing a `MutableInteractionSource` with the `Switch` (`onCheckedChange = null`) and triggering ROM haptic **`302`**.
4. **Top Bars (Frosted-Glass Scroll Blur + `40.dp` `Kyant0/AndroidLiquidGlass` Circular Back Button with Stock COUI Arrow) & Liquid Glass Floating Docks:**
   - Detail subpages overlay a **`64.dp`** (+ `14.dp` soft fade apron) true variable-radius frosted-glass top bar below `WindowInsets.statusBars` as a **sibling** above a `Modifier.layerBackdrop(backdrop)` scrollable container. **NEVER alpha-mask (`BlendMode.DstIn`) a uniform `blur()` layer** — blending `alpha * blurred + (1 - alpha) * sharp` produces a bloom/Orton halo with sharp text bleeding through instead of frosted glass! Instead, chain a **2-pass separable AGSL `RuntimeShader` progressive Gaussian blur** via `com.kyant.backdrop.effects.runtimeShaderEffect` (`dir=(1,0)` then `dir=(0,1)`, 13 taps, **100% opaque replacement**) where the blur radius itself ramps smoothly via `smoothstep(0.95, 0.35, t)` from **`0.dp`** at the bottom apron edge up to **`6.5.dp`** at the header/top (giving crisp readable `~4.5–5.5.dp` frosted glass right below the title with zero sharp ghost and zero bottom seam), paired with the bottom-to-top canvas fade (`#F0F1F3` light / `#000000` dark: `0.98f@0.00 → 0.94f@0.28 → 0.72f@0.52 → 0.36f@0.74 → 0.12f@0.88 → 0.0f@1.00`).
   - The circular back button is **`40.dp`** (`124 px` including specular rim on `1200×2670` 3x screens, matching stock ColorOS 17 1:1) using `Kyant0/AndroidLiquidGlass` (`vibrancy()`, `blur(4.dp)`, `lens(8.dp, 18.dp, depthEffect = true)`, `HighlightStyle.Default` `0.75.dp` rim, `Shadow(radius = 16.dp, offset = (0.dp, 4.dp))`, `Color(0xB8F6F7F9)` light / `Color(0x28FFFFFF)` dark tint) and the **exact stock ColorOS 17 `coui_back_arrow_normal.xml` vector** (`24×24` viewport rendered at `22.dp`: `M10.1,4.4L2.5,12M2.5,12L10.1,19.6M2.5,12L21.5,12.001`, `strokeWidth = 1.4.dp`, round cap/join), with `19.5.sp`–`20.sp` `FontWeight.Medium` title and **no right-bar clutter**.
   - Dark media overlays & floating bottom docks use `AndroidLiquidGlass` (`vibrancy()` + `blur()` + subtle `lens()`) and `ColorOsToolbarGlass` (`BlurRadiusPx = 25f`).
5. **ColorOS 17 Rubber-Band Bouncy Overscroll (Zero-Lag 120Hz Tracking):**
   - Always disable AOSP `StretchOverscroll` (`CompositionLocalProvider(LocalOverscrollFactory provides null)`) so dragging past top/bottom boundaries never distorts text vertically.
   - Wrap scrollable lists in **`Modifier.couiBouncyOverscroll()`** (`NestedScrollConnection` tracking finger drag synchronously in `mutableFloatStateOf` with rubber-band damping `maxOffset=180.dp`, `dampingFactor=360.dp`, `maxOffset * (1 - exp(-|drag| / k))` — zero per-frame coroutine launch lag — and handing off to `Animatable.animateTo(0f, initialVelocity = (vel * 0.12f).coerceIn(-1200f, 1200f), animationSpec = spring(dampingRatio = 1.20f, stiffness = 200f))` on pointer release or fling, matched to stock ColorOS 17 `COUISpringOverScroller` `tension=209.93, friction=37.57` overdamped smooth closing glide — never tight `stiffness >= 320f` snap-back).
6. **Authentic Motion & ROM Haptics (Zero Material Ripples):**
   - Disable rectangular Material ripples (`indication = null`) on cards and switches; use **Tint** feedback on list rows and `CouiPressMotion` scale springs on cards/buttons.
   - Trigger ColorOS ROM haptics: **`302`** (`CouiHapticFeedback.Switch`), **`307`** (`CouiHapticFeedback.ScrollEdge`), **`308`** (wheel/slider detents), with safe Android fallback (`CLOCK_TICK`).
7. **Inset Hairline Dividers & Edge-to-Edge Insets:**
   - Draw **`1.dp` `#14000000`** (light) / **`#24FFFFFF`** (dark) internal dividers inset **`16.dp`** horizontally (aligning with row text, never edge-to-edge) **strictly between** adjacent rows inside multi-row cards—never at card top/bottom edges.
   - Pad scrollable lists and floating bars cleanly for `WindowInsets.statusBars`, `navigationBars`, and `ime`.

## 3. Pre-Release Visual Parity Audit (Full Native `1:1` Resolution Only — Never Downscale `<= 800px`)

Before considering any ColorOS 17 UI complete:
1. Inspect all verification screenshots at **full native `1:1` device resolution** (e.g., `1200×2670`, never downscaled to `<= 800px`) in both Light & Dark modes (`#F0F1F3`/`#FFFFFF` vs `#000000`/`#1C1C1E`).
2. Verify `46.dp` `Kyant0/AndroidLiquidGlass` circular back button with `LayerBackdrop` scroll-under refraction (tested both at top rest and mid-scroll with card content refracting behind the glass disc), wide soft diffuse ambient shadow (`radius = 18.dp, offset = (0.dp, 5.dp)`), bright `0.75.dp` specular rim, sleek wide-winged `24.dp`–`25.dp` `1.65.dp`-stroke back arrow, and clean top bar with zero unnecessary right-side icons.
3. Verify **zero non-COUI inline colored status pills** (`"eSIM"`, `"Active"`, `"SIM 2"`) beside preference row titles, zero redundant left icon boxes on standard preference rows, zero heavy outlined inline action pills, inset `1dp` `#14000000` dividers, and `20.dp` continuous squircle corners.
4. Verify smooth rubber-band top/bottom bouncy overscroll (even when all rows fit on one screen) with zero vertical text stretching.
5. Verify edge-to-edge clearance under top status bar cutout and bottom gesture pill, plus Flux monochrome launcher icon silhouette inside the `108dp` safe circle.
