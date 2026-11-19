# ColorOS 17 Adaptive Icons, Spring Motion & Localization Rules

---

## 1. Adaptive + ColorOS Flux `<monochrome>` App Icons

ColorOS 17 launchers (such as `OplusLauncher` Flux / Art+ icon themes) tint the `<monochrome>` layer dynamically and apply squircle masks automatically. Never bake outer rounded rectangles into the foreground or monochrome drawables.

### Exact Safe-Zone Vector Geometry (`ONextBox` Pattern)

Use a **`108dp × 108dp`** vector with a **`1500 × 1500`** viewport and wrap all artwork inside a **`translate(250, 250)`** group targeting a `1000 × 1000` inner design canvas. This keeps the visible silhouette comfortably inside Android's `66%` (`72dp`) adaptive safe zone (`250..1250`, i.e., `16.7%..83.3%`):

```xml
<!-- res/mipmap-anydpi/ic_launcher.xml (and ic_launcher_round.xml) -->
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
    <monochrome android:drawable="@drawable/ic_launcher_monochrome" />
</adaptive-icon>
```

```xml
<!-- res/drawable/ic_launcher_monochrome.xml -->
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="1500"
    android:viewportHeight="1500">
    <group
        android:translateX="250"
        android:translateY="250">
        <!-- Artwork authored on 0..1000 x 0..1000 coordinate grid -->
        <path
            android:pathData="M500 128 ..."
            android:fillColor="#FFFFFF"
            android:fillType="evenOdd" />
    </group>
</vector>
```

**Checklist for Flux Icons:**
- Keep `<monochrome>` purely single-color (`#FFFFFF`) with `android:fillType="evenOdd"` cutouts rather than opaque black patches (since launchers apply `tint` uniformly to the entire path).
- Keep stroke/cutout thickness wide enough (`>= 11` units on the `1000×1000` grid, `~24`+ preferred) so downscaled home-screen and App Drawer icons remain crisp.

---

## 2. Olivelink Press Motion, Overscroll & Spring Physics (`CouiSpring` + `CouiPressMotion`)

COUI avoids sharp Material ripples (`indication = null`) and instead drives tactile scale + spotlight + rubber-band overscroll using smooth springs (`dampingRatio = 1f - bounce`, `stiffness = (2π / response)²`):

| Interaction | Physics / Constants | Behavior |
|---|---|---|
| **Standard Card / Button Press** (`CouiPressMotion`) | `response = 0.25f` (`0.3f` pills), `bounce = 0f` | Scales down toward `0.85f..0.97f` proportional to view area (`0.94f` with `dampingRatio = 0.70f, stiffness = 600f` for the `46.dp` `AndroidLiquidGlass` circular back button, paired with the wide-winged `24.dp`–`25.dp` `1.65.dp`-stroke back arrow) |
| **Card / Button Release** | `response = 0.5f`, `bounce = 0.6f` (`0.5f` pills) | Smooth elastic return to `1.0f` preserving interruptible velocity |
| **List Rubber-Band Overscroll** (`couiBouncyOverscroll`) | `maxOffset * (1 - exp(-|drag| / k))`, `dampingRatio = 0.72f`, `stiffness = 380f` | Translates list via `graphicsLayer.translationY` at both top & bottom edges without vertical text stretch; works on single-screen and multi-screen lists |
| **Switch Toggle** (`CouiSwitchMotion`) | `response = 0.4f`, `bounce = 0.3f` | Thumb moves across `44.dp` track while stretching to `1.3x` width at `p = 0.347` |
| **Switch Drag / Attach** | `response = 0.3f / 0.4f`, `bounce = 0f / 0.2f` | Instant finger tracking (`bounce = 0f`), snappy magnetic attach (`bounce = 0.2f`) |
| **Loading Switch** | `800ms` linear loop | Thumb shrinks to `0f`; `312°` rotating arc renders inside track while disabled |

Never cancel a running spring mid-gesture with a hard jump—redirect `animateTo(target)` / `animateToFinalPosition(target)` so existing velocity carries smoothly across rapid taps or reversals.

---

## 3. Dual English & Indonesian (EN / ID) Localization Rules

When shipping ColorOS companion UIs or system settings strings for bilingual (English + Indonesian) users:

1. **Default `values/strings.xml` = Concise Natural English**; provide complete parity in `values-in/strings.xml` (`b+id` / legacy Android `in` qualifier—Android resolves Indonesian system locale to `values-in` across all API levels!).
2. **Short Labels Inside Pill Buttons & Stacked Dialogs:** Indonesian strings are often 15–30% longer than English (e.g., `"Turn off"` -> `"Matikan"`, `"Custom charging limit"` -> `"Batas pengisian kustom"`). Ensure pill buttons (`ContinuousCapsule`, `minHeight = 46.dp`) allow 1-line `TextOverflow.Ellipsis` or automatic vertical button stacking in dialogs (`CouiDialogButton`).
3. **Clear Status Badges & Honest Wording:** Keep preference titles under ~3–4 words (`"Smart charging protection"`, `"Disable memory killer"`) and place technical caveats cleanly into summary/footer strings.
