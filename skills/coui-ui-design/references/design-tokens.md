# ColorOS 17 (COUI) Design Tokens & Physical Constants

Calibrated against ColorOS 16/17 (`com.oplus.uxdesign` / `Theme.COUI`), `ItosEO/COUI-Kit`, `suqi8/coui`, `MiToverG422/ONextBox`, and `Kyant0/Capsule`.

---

## 1. Surface & Semantic Color Roles (Light vs. Dark AMOLED)

ColorOS uses an **inverted background hierarchy**: gray page canvas + crisp white cards in Light Mode, and true `#000000` AMOLED canvas + translucent white cards in Dark Mode. Dialogs and bottom sheets always sit on elevated opaque surfaces (`#FFFFFF` / `#1E1E1E`), never gray canvas.

| Token Role | Light Mode | Dark Mode (AMOLED) | COUI / Android Usage |
|---|---|---|---|
| `background` (Page Canvas) | `#F0F1F2` (COUI native) / `#F7F8FA` (Settings subpages) | `#000000` | Root scrollable `Scaffold` / page background |
| `card` (`couiColorCardBackground`) | `#FFFFFF` | `#1AFFFFFF` (10% white over `#000000`) | Grouped & standalone preference cards |
| `surface` (Dialog / Bottom Sheet) | `#FFFFFF` | `#1E1E1E` | Bottom sheet panels, modal dialogs (`coui_color_background_elevated`) |
| `container` / `menuSurface` | `#FFFFFF` | `#333333` | Floating popups, dropdown menus (`coui_color_surface_top`) |
| `pressed` / `surfaceContainerHigh` | `#E6E6E6` | `#33FFFFFF` | List row press tint (`coui_color_card_pressed`) & `"You might be looking for"` footer |
| `accent` (COUI Blue — `CouiColor.Day/Night`) | `#0080FF` (Settings 17) / `#0066FF` (`coui_color_blue`) | `#1A8CFF` (Settings 17) / `#247CFF` (`_dark`) | Checked `Switch` track, primary CTA, active sliders |
| `accentOnSurface` | `#0080FF` / `#0066FF` | `#5C9DFF` | Theme-tinted links & icons inside dark cards |
| `onSurface` (Primary Title) | `#E6000000` (90% black) | `#E6FFFFFF` (90% white) | Card row titles, headlines |
| `onSurfaceVariant` (Summary / Header) | `#8A000000` (54% black) | `#8AFFFFFF` (54% white) | Group titles, trailing assignments, subtitles |
| `onSurfaceSummary` (Tertiary Caption) | `#66000000` (40% black) | `#66FFFFFF` (40% white) | Footers & explanatory captions |
| `disabled` (Neutral Text/Icons) | `#42000000`–`#4D000000` | `#4DFFFFFF` | Disabled titles & chevrons |
| `outlineVariant` / `dividerLine` | `#1F000000` (12% black) | `#33FFFFFF` (20% white) | `0.33.dp`–`0.5.dp` (`>= 1px`) card-internal dividers |
| `windowDimming` | `#33000000` (20% black) | `#99000000` (60% black) | Modal dialog & bottom-sheet scrim (`coui_color_mask`) |
| `sliderBackground` | `#0D000000` | `#1AFFFFFF` | Unfilled seekbar/slider capsule track |
| `sliderKeyPoint` | `#F2F2F2` | `#595959` | Section seekbar tick marks (`coui_section_seekbar`) |

### Semantic Status Palette (`CouiColor`)

| Role | Light (`CouiColor.Day`) | Dark (`CouiColor.Night`) |
|---|---|---|
| Green (Healthy / Active) | `#00BD13` | `#24B232` |
| Yellow (Caution / Pending) | `#FFB200` | `#E5A100` |
| Orange (Alert) | `#FF7700` | `#F08222` |
| Red (Error / Destructive) | `#EB3B2F` (`#DB382C` label) | `#EB493D` (`#FF6C61` label) |

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
| Dialogs & Floating Modals | `28.dp`–`33.dp` continuous | `coui_dialog_os_16_1_radius_33_dp`; floating bottom dialogs inset `16.dp` horizontally |
| Bottom Sheet Top Corners | `28.dp` continuous | `coui_panel_os_16_1_radius_28_dp` |
| Popup Menus / Dropdowns | `16.dp` continuous | `coui_popup_list_window_os_16_1_radius_16_dp` |
| Inner Icon Badge / Tile | `12.dp`–`16.dp` continuous | `12.dp` matches `couiRoundCornerM` |
| Buttons, Chips, Floating Docks | `Capsule()` (`50%`) | `coui_btn_large_height_min = 44.dp` |
| TopBar Back / Action Buttons | `CircleShape` (`40.dp × 40.dp`) | `1.dp` border (`#1F000000` light / `#33FFFFFF` dark) |

### Layout Metrics

- **Page horizontal card margin:** `16.dp`
- **Card spacing (unlabelled):** `12.dp` (stacked single-row cards) – `16.dp` (major sections)
- **Section header spacing:** `12.dp` top gap; header text padded `horizontal = 32.dp, vertical = 8.dp`
- **Settings row height:** `minHeight = 48.dp` (`+2.dp` top padding on first item, `+2.dp` bottom padding on last item in multi-row cards)
- **Settings row internal padding:** `horizontal = 16.dp, vertical = 10.dp–14.dp`
- **Detail TopBar height:** `52.dp` below `WindowInsets.statusBars`

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
| Detail TopBar Title | `18.dp.toSp()` | `FontWeight(550)` | Single-line ellipsis next to `40.dp` back button |
| Section Category Header | `12.sp`–`13.sp` | `Medium` (`500`) | `onSurfaceVariant`, `padding(horizontal = 32.dp, vertical = 8.dp)` |
| Preference Row Title | `16.sp` | `Medium` (`500`) | `onSurface` |
| Preference Summary | `13.sp`–`14.sp` | `Normal` (`400`) | `onSurfaceVariant` / `onSurfaceSummary`, `2.dp` below title |
| Trailing Value | `14.sp` | `Normal` (`400`) | `onSurfaceVariant`, `4.dp`–`8.dp` before chevron |
| Pill Button Label | `15.sp`–`16.sp` | `FontWeight(550)` | Centered in `Capsule()` (`minHeight = 44.dp`) |

---

## 4. Authentic ColorOS ROM Haptic Feedback Constants

Do **not** use generic heavy `LONG_PRESS` vibrations when Oplus ROM haptic codes are available. Always gate with `view.isAttachedToWindow`:

| Event | Constant Code | Enum / Usage | Fallback on Non-Oplus ROMs |
|---|---|---|---|
| Switch Toggle | **`302`** | `CouiHapticFeedback.Switch` | `HapticFeedbackConstants.CLOCK_TICK` / `TextHandleMove` |
| List Overscroll / Edge | **`307`** | `CouiHapticFeedback.ScrollEdge` | Omit or subtle edge feedback |
| Picker / Slider Detent | **`308`** | Section seekbar step / wheel tick | `302` -> `SEGMENT_TICK` (`26`) -> `CLOCK_TICK` |
