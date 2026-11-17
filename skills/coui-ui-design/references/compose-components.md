# ColorOS 17 Jetpack Compose Component Blueprints

Production-tested patterns synthesized from **`ItosEO/COUI-Kit`**, **`MiToverG422/ONextBox`**, **`suqi8/coui`**, **`Kyant0/Capsule`**, and **`Kyant0/AndroidLiquidGlass`**.

---

## 1. Gradle Dependencies

```kotlin
dependencies {
    // Option A: ItosEO/COUI-Kit (ColorOS 17 switches, haptics, springs, squircles, effects)
    implementation("io.github.itoseo:coui:0.0.1-alpha11")
    implementation("io.github.itoseo:coui-icons:0.0.1-alpha11")       // Optional system icons
    implementation("io.github.itoseo:coui-material3:0.0.1-alpha11")   // Optional Material 3 Monet bridge

    // Option B: Kyant0 continuous G2 squircle geometry & backdrop glass
    implementation("io.github.kyant0:capsule:2.1.0")
    implementation("io.github.kyant0:backdrop:1.0.0")
}
```

---

## 2. Continuous Squircle Cards & Grouped Settings Pattern

Never tint secondary clickable cards gray; **every active preference card** uses `card` (`#FFFFFF` light / `#1AFFFFFF` dark), and dividers (`0.33.dp` / `1px` minimum) appear **only between** rows inside multi-row cards.

```kotlin
@Composable
fun CouiSettingsGroup(
    title: String? = null,
    modifier: Modifier = Modifier,
    content: @Composable ColumnScope.() -> Unit,
) {
    Column(modifier = modifier.fillMaxWidth()) {
        if (!title.isNullOrEmpty()) {
            Text(
                text = title,
                color = CouiTheme.colorScheme.onSurfaceVariant,
                fontSize = 13.sp,
                fontWeight = FontWeight.Medium,
                modifier = Modifier.padding(horizontal = 32.dp, vertical = 8.dp),
            )
        }
        Surface(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp),
            shape = CouiShape(24.dp, cornerWeight = 3f), // or ContinuousRoundedRectangle(20.dp)
            color = CouiTheme.colorScheme.card,
        ) {
            Column(
                modifier = Modifier.padding(vertical = 2.dp),
                content = content,
            )
        }
    }
}

@Composable
fun CouiCardDivider() {
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp)
            .height(0.33.dp)
            .background(CouiTheme.colorScheme.outlineVariant),
    )
}
```

---

## 3. ColorOS 17 Whole-Row Switch Pattern (`44.dp × 24.dp`)

Key ColorOS 17 Switch constants (`CouiSwitchDefaults`):
- Track: `44.dp × 24.dp`, shape `CouiCapsuleShape`
- Thumb: `18.dp` white circle (`3.dp` inner inset)
- Motion: `toggleResponse = 0.4f`, `toggleBounce = 0.3f`, horizontal stretch `1.3x` at `p ≈ 0.347`
- Haptics: ROM constant `302` (`CouiHapticFeedback.Switch`)

Always attach `Modifier.toggleable(...)` to the **entire row** with `indication = null`, sharing `MutableInteractionSource` with the `Switch` (`onCheckedChange = null`):

```kotlin
@Composable
fun CouiSwitchPreferenceRow(
    title: String,
    checked: Boolean,
    onCheckedChange: (Boolean) -> Unit,
    summary: String? = null,
    enabled: Boolean = true,
) {
    val view = LocalView.current
    val interactionSource = remember { MutableInteractionSource() }

    Row(
        modifier = Modifier
            .fillMaxWidth()
            .defaultMinSize(minHeight = 48.dp)
            .toggleable(
                value = checked,
                enabled = enabled,
                role = Role.Switch,
                interactionSource = interactionSource,
                indication = null,
                onValueChange = { next ->
                    if (!view.performCouiHapticFeedback(CouiHapticFeedback.Switch)) {
                        view.performHapticFeedback(HapticFeedbackConstants.CLOCK_TICK)
                    }
                    onCheckedChange(next)
                },
            )
            .padding(horizontal = 16.dp, vertical = 12.dp),
        verticalAlignment = Alignment.CenterVertically,
    ) {
        Column(modifier = Modifier.weight(1f).padding(end = 16.dp)) {
            Text(
                text = title,
                fontSize = 16.sp,
                fontWeight = FontWeight.Medium,
                color = CouiTheme.colorScheme.onSurface,
            )
            if (!summary.isNullOrEmpty()) {
                Spacer(Modifier.height(2.dp))
                Text(
                    text = summary,
                    fontSize = 13.sp,
                    color = CouiTheme.colorScheme.onSurfaceVariant,
                )
            }
        }
        com.itos.coui.switch.Switch(
            checked = checked,
            onCheckedChange = null, // Row owns click & semantics
            onDragCheckedChange = onCheckedChange,
            interactionSource = interactionSource,
            enabled = enabled,
        )
    }
}
```

---

## 4. Detail TopBar (`52.dp`) & `40.dp` Circular Glass Button (`ONextBox`)

Detail screens render a **`52.dp`** toolbar below `WindowInsets.statusBars` with:
1. A **`40.dp` `CircleShape`** frosted back button (`ColorOsTopBarButton`) with `1.dp` border mask (`#1F000000` light / `#33FFFFFF` dark) and `ColorOsTopBarMotion` press scale.
2. **`18.dp.toSp()`** (`FontWeight(550)`) title with single-line ellipsis.
3. A **scroll-revealed bottom divider** (`0.33.dp`–`0.5.dp`) whose horizontal inset narrows from `16.dp -> 0.dp` and `alpha` animates `0f -> 1f` over `scrollDistancePx / ToolbarHeight`.

```kotlin
// Progressive hairline divider inside ColorOs17DetailTopBar:
Modifier.drawWithContent {
    drawContent()
    val fraction = (scrollDistancePx() / dividerRevealPx).coerceIn(0f, 1f)
    if (fraction > 0f) {
        val inset = expandedDividerInsetPx * (1f - fraction)
        val y = size.height - dividerThicknessPx / 2f
        drawLine(
            color = dividerColor,
            start = Offset(inset, y),
            end = Offset(size.width - inset, y),
            strokeWidth = dividerThicknessPx,
            alpha = fraction,
        )
    }
}
```

---

## 5. `AndroidLiquidGlass` & `ColorOsToolbarGlass` Floating Pill Docks

For media viewers, dark galleries, and floating bottom navigation bars, pair `Kyant0/Capsule`'s `ContinuousCapsule` / `Capsule()` with `Kyant0/AndroidLiquidGlass` `drawBackdrop(...)` and `ColorOsToolbarGlass` fallback material:

```kotlin
object ColorOsToolbarGlass {
    fun fallbackColor(container: Color, surface: Color): Color = container.compositeOver(surface)
    fun firstTint(darkTheme: Boolean): Int = if (darkTheme) 0x80666666.toInt() else 0xB3666666.toInt()
    fun luminosityTint(darkTheme: Boolean): Int = if (darkTheme) 0xB3262626.toInt() else 0x99F5F5F5.toInt()
    const val BlurRadiusPx: Float = 25f
}

// Modifier pipeline on a Capsule() floating dock:
Modifier
    .windowInsetsPadding(WindowInsets.navigationBars)
    .padding(horizontal = 20.dp, vertical = 12.dp)
    .drawBackdrop(
        backdrop = backdrop,
        shape = { ContinuousCapsule },
        effects = {
            vibrancy()
            blur(25f) // ColorOsToolbarGlass.BlurRadiusPx
            lens(refractionHeight = 16.dp.toPx(), refractionAmount = 32.dp.toPx(), depthEffect = true)
        },
        onDrawSurface = {
            drawRect(if (darkTheme) Color(0x991E1E1E) else Color(0xCCF5F5F5))
        },
    )
```

---

## 6. Dialogs, Loading Dialogs & Bottom Sheets

- **Dialogs:** `28.dp`–`33.dp` continuous squircle corners, `24.dp` content padding, two horizontal pill buttons (`Capsule()`, `minHeight = 44.dp`) that auto-stack vertically when 3 actions or long localized labels are present.
- **Loading Dialog:** Non-dismissible `24.dp` continuous squircle card with COUI indeterminate spinner and a short single-line status label.
- **Bottom Sheets:** `28.dp` continuous top corners, `#FFFFFF` (Light) / `#1E1E1E` (Dark) fill over `#33000000` / `#99000000` scrim, `navigationBars` + `ime` bottom padding.
