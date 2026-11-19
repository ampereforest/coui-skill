# ColorOS 17 (COUI) Design Tokens & Physical Constants

Calibrated against native ColorOS 17 Settings subpages (`Mobile network`, `Personal hotspot`, `Battery health`), `com.oplus.uxdesign` / `Theme.COUI`, `ItosEO/COUI-Kit`, `suqi8/coui`, `MiToverG422/ONextBox`, `ampereforest/dada_esim_compat`, and `Kyant0/Capsule`.

---

## 1. Surface & Semantic Color Roles (Light vs. Dark AMOLED)

ColorOS 17 uses an **inverted background hierarchy**: soft cool off-white page canvas + crisp pure white cards in Light Mode, and true `#000000` AMOLED canvas + `#1C1C1E` (or translucent white) cards in Dark Mode. Dialogs and bottom sheets always sit on elevated opaque surfaces (`#FFFFFF` / `#1C1C1E`), never gray canvas.

| Token Role | Light Mode | Dark Mode (AMOLED) | COUI / Android Usage |
|---|---|---|---|
| `background` (Page Canvas) | `#F5F6F8` (Settings subpages) / `#F0F1F3` (system canvas) | `#000000` | Root scrollable `Scaffold` / page background |
| `card` (`couiColorCardBackground`) | `#FFFFFF` | `#1C1C1E` (or `#1AFFFFFF` 10% white over `#000000`) | Grouped & standalone preference cards |
| `topBarGlassButton` | `#E6FFFFFF` (`0.5.dp` gradient rim `#FFFFFF -> #18000000`) | `#26FFFFFF` (`0.5.dp` gradient rim `#3DFFFFFF -> #14FFFFFF`) | `42–44dp` circular frosted back button (`elevation = 4–6dp`) |
| `surface` (Dialog / Bottom Sheet) | `#FFFFFF` | `#1C1C1E` / `#1E1E1E` | Bottom sheet panels, modal dialogs (`coui_color_background_elevated`) |
| `container` / `menuSurface` | `#FFFFFF` | `#333333` | Floating popups, dropdown menus (`coui_color_surface_top`) |
| `pressed` / `surfaceContainerHigh` | `#E8E9EB` | `#2C2C2E` / `#33FFFFFF` | List row press tint (`coui_color_card_pressed`) & `"You might be looking for"` footer |
| `accent` (COUI Blue — `CouiColor.Day/Night`) | `#0080FF` (Settings 17) / `#0066FF` (`coui_color_blue`) | `#1A8CFF` (Settings 17) / `#247CFF` (`_dark`) | Checked `Switch` track, primary CTA, borderless inline actions (`Copy`) |
| `accentOnSurface` | `#0080FF` / `#0066FF` | `#5C9DFF` | Theme-tinted links & icons inside dark cards |
| `switchTrackOff` (`secondary`) | `#D8DADC` | `#48484A` | Unchecked `44dp × 24dp` COUI switch track |
| `onSurface` (Primary Title) | `#E6000000` (90% black) | `#E6FFFFFF` (90% white) | Card row titles, top bar title, headlines |
| `onSurfaceSecondary` (Section Header) | `#7F7F7F` | `#8AFFFFFF` (54% white) | Section group titles (`eSIM Profiles`, `Tethering`, `Advanced`) |
| `onSurfaceVariantSummary` (Subtitle / Value) | `#8C8C8C` | `#73FFFFFF` (45% white) | Row summaries, trailing status values (`Off`, `80%`, `5240 mAh`) |
| `disabled` / `chevron` | `#4D000000` | `#4DFFFFFF` | Disabled titles & trailing `>` chevrons |
| `outlineVariant` / `dividerLine` | `#14000000` (8% black) | `#24FFFFFF` (14% white) | `1.dp` card-internal dividers inset `16.dp` horizontally |
| `windowDimming` | `#33000000` (20% black) | `#99000000` (60% black) | Modal dialog & bottom-sheet scrim (`coui_color_mask`) |
| `sliderBackground` | `#0D000000` | `#1AFFFFFF` | Unfilled seekbar/slider capsule track |
| `sliderKeyPoint` | `#F2F2F2` | `#595959` | Section seekbar tick marks (`coui_section_seekbar`) |

### Semantic Status Palette (`CouiColor`)

| Role | Light (`CouiColor.Day`) | Dark (`CouiColor.Night`) | Low-Profile Pill Container |
|---|---|---|---|
| Blue (Primary / Route) | `#0080FF` | `#5C9DFF` | `#180080FF` / `#331A8CFF` |
| Green (Healthy / Active) | `#1F9D55` (`#00BD13` system) | `#4ADE80` (`#24B232` system) | `#1A1F9D55` / `#294ADE80` |
| Yellow (Caution / Pending) | `#FFB200` | `#E5A100` | `#1FFFB200` / `#2EE5A100` |
| Orange (Alert) | `#FF7700` | `#F08222` | `#1FFF7700` / `#2EF08222` |
| Red (Error / Destructive) | `#EB3B2F` (`#DB382C` label) | `#EB493D` (`#FF6C61` label) | `#22EB3B2F` / `#38EB493D` |

### Material You Dynamic Seeding Compatibility (`CouiMaterial3Style`)

When integrating COUI components inside a Material 3 / Monet dynamic-color host app (instead of `Theme.COUI`), initialize `COUI-Kit` with `CouiMaterial3Style()`:

```kotlin
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        Coui.initialize(defaultStyle = CouiMaterial3Style())
    }
}
```

`CouiMaterial3Style` dynamically binds `MaterialTheme.colorScheme.primary`, `surface`, `surfaceBright`, `onSurface`, `outlineVariant`, and `background` into `CouiColorScheme` while preserving COUI continuous squircle geometry, spring physics, and `44.dp × 24.dp` switch proportions.

---

## 2. Continuous Squircle Radii & Spacing Scale

Never mix circular `RoundedCornerShape` on primary surfaces—always use `ContinuousRoundedRectangle` (`Kyant0/Capsule`) or `CouiShape` (`COUI-Kit`, $L_p$ zero-isoline with `CouiCornerWeight = 2.5f` general / `3.0f` page cards):

| Surface / Element | Recommended Radius | Notes |
|---|---|---|
| Preference Card / Standalone Card | `20.dp` (`Capsule`) – `24.dp` (`CouiShape(24.dp, 3f)`) | Legacy COUI `1.28x` extension makes `20.dp` `ContinuousRoundedRectangle` match `24.dp` `CouiShape` |
| Dual-column Hero Card (e.g., SIM 1 / SIM 2) | `20.dp` continuous | `12.dp` horizontal gap between paired cards |
| Dialogs & Floating Modals | `24.dp`–`28.dp` continuous | Floating bottom dialogs inset `16.dp` horizontally |
| Bottom Sheet Top Corners | `28.dp` continuous | `coui_panel_os_16_1_radius_28_dp` |
| Popup Menus / Dropdowns | `16.dp` continuous | `coui_popup_list_window_os_16_1_radius_16_dp` |
| Inline Status Pill (`eSIM`, `Active`) | `ContinuousCapsule` | Compact `horizontal = 6.dp, vertical = 1.5.dp`, `10.5.sp` Medium |
| Buttons, Chips, Floating Docks | `ContinuousCapsule` (`50%`) | `coui_btn_large_height_min = 44.dp–46.dp` |
| TopBar Frosted Back Button | `CircleShape` (`42.dp–44.dp`) | `#E6FFFFFF` fill, `0.5.dp` gradient rim, `4.dp–6.dp` ambient shadow, `22–24.dp` `2.dp`-stroke arrow |

### Layout Metrics (Calibrated to ColorOS 17 Settings Subpages)

- **Page outer screen margin:** `16.dp`
- **Card spacing (unlabelled):** `12.dp` between stacked standalone cards
- **Section header spacing:** `8.dp` top spacer + `padding(start = 16.dp, end = 16.dp, top = 10.dp, bottom = 6.dp)` inside `16.dp` list padding = **`32.dp` total inset from screen edge**
- **Settings row density:** `minHeight = 52.dp`, internal `PaddingValues(horizontal = 16.dp, vertical = 14.dp)` (no bulky left icon boxes on standard text/toggle rows)
- **Internal dividers:** `1.dp` `#14000000` (light) / `#24FFFFFF` (dark), inset `16.dp` horizontally to align cleanly with row text
- **Detail TopBar height:** `60.dp` below `WindowInsets.statusBars`, floated over content with a soft vertical surface scrim

---

## 3. Typography Scale

Always preserve font padding & clean single-line alignment on COUI rows:

```kotlin
TextStyle(
    fontFamily = FontFamily.SansSerif,
    platformStyle = PlatformTextStyle(includeFontPadding = true),
    lineHeightStyle = LineHeightStyle(
        alignment = LineHeightStyle.Alignment.Top,
        trim = LineHeightStyle.Trim.LastLineBottom,
        mode = LineHeightStyle.Mode.Minimum,
    )
)
```

| Role | Size | Weight | Color / Usage |
|---|---|---|---|
| Expanded Large Title | `32.sp`–`40.sp` | `Bold` (`700`) | `onSurface`, inside collapsible `LargeTopAppBar` |
| Detail TopBar Title | `19.5.sp` | `Medium` (`500`) | Single-line ellipsis, `12.dp` right of `44.dp` frosted back button |
| Section Category Header | `13.sp` | `Medium` (`500`) | `#7F7F7F` light / `#8AFFFFFF` dark, `32.dp` total screen inset |
| Preference Row Title | `16.sp` | `Medium` (`500`) | `onSurface` |
| Preference Summary | `13.sp` | `Normal` (`400`) | `#8C8C8C` light / `#73FFFFFF` dark, `2.dp` below title |
| Trailing Value / Action | `14.sp` | `Normal` (value) / `Medium` (action) | `#8C8C8C` for passive status (`Off`, `80%`), `#0080FF` for borderless inline actions (`Copy`) |
| Inline Status Badge | `10.5.sp` | `Medium` (`500`) | Sleek low-profile tinted `ContinuousCapsule` (`eSIM`, `Active`) |
| Pill Button Label | `16.sp` | `Medium` (`500`) | Centered in `ContinuousCapsule` (`minHeight = 46.dp`) |

---

## 4. Authentic ColorOS ROM Haptic Feedback Constants

Do **not** use generic heavy `LONG_PRESS` vibrations when Oplus ROM haptic codes are available. Always gate with `view.isAttachedToWindow`:

| Event | Constant Code | Enum / Usage | Fallback on Non-Oplus ROMs |
|---|---|---|---|
| Switch Toggle / Inline Action Tap | **`302`** | `CouiHapticFeedback.Switch` | `HapticFeedbackConstants.CLOCK_TICK` |
| List Overscroll / Edge | **`307`** | `CouiHapticFeedback.ScrollEdge` | Omit or subtle edge feedback |
| Picker / Slider Detent | **`308`** | Section seekbar step / wheel tick | `302` -> `SEGMENT_TICK` (`26`) -> `CLOCK_TICK` |
