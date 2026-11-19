# ColorOS 17 Jetpack Compose Component Blueprints

Production-tested patterns synthesized from native ColorOS 17 Settings (`Mobile network`, `Personal hotspot`, `Battery health`), **`ItosEO/COUI-Kit`**, **`MiToverG422/ONextBox`**, **`ampereforest/dada_esim_compat`**, **`suqi8/coui`**, **`Kyant0/Capsule`**, and **`Kyant0/AndroidLiquidGlass`**.

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

## 2. Continuous Squircle Cards, Text-First Rows & Inset Dividers

Never tint secondary clickable cards gray, and omit bulky left icon boxes on standard Settings subpage rows so all row titles align cleanly at `16.dp` internal padding (`32.dp` from screen edge). Use **`1.dp` `#14000000`** (light) / **`#24FFFFFF`** (dark) dividers inset **`16.dp`** horizontally **only between** adjacent rows inside multi-row cards.

```kotlin
@Composable
fun CouiCard(
    modifier: Modifier = Modifier,
    cornerRadius: Dp = 20.dp,
    backgroundColor: Color = CouiTheme.colors.surfaceContainer, // #FFFFFF light / #1C1C1E dark
    content: @Composable ColumnScope.() -> Unit,
) {
    val shape = remember(cornerRadius) { ContinuousRoundedRectangle(cornerRadius) }
    Column(
        modifier = modifier
            .fillMaxWidth()
            .clip(shape)
            .background(backgroundColor, shape),
        content = content,
    )
}

@Composable
fun CouiSmallTitle(text: String, modifier: Modifier = Modifier) {
    Text(
        text = text,
        fontSize = 13.sp,
        fontWeight = FontWeight.Medium,
        color = CouiTheme.colors.onSurfaceSecondary, // #7F7F7F light / #8AFFFFFF dark
        modifier = modifier.padding(start = 16.dp, end = 16.dp, top = 10.dp, bottom = 6.dp),
    )
}

@Composable
fun CouiDivider(
    horizontalInset: Dp = 16.dp,
    thickness: Dp = 1.dp,
    color: Color = CouiTheme.colors.dividerLine, // #14000000 light / #24FFFFFF dark
) {
    Canvas(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = horizontalInset)
            .height(thickness),
    ) {
        val px = thickness.toPx().coerceAtLeast(1f)
        drawLine(color = color, strokeWidth = px, start = Offset(0f, px / 2f), end = Offset(size.width, px / 2f))
    }
}
```

For inline actions (such as copying an EID or displaying status), prefer **borderless right-aligned text** (`14.sp` Medium `#0080FF` for actions, `14.sp` Normal `#8C8C8C` for passive values) + haptic `302` on tap, rather than outlined pill buttons.

---

## 3. ColorOS 17 Whole-Row Switch Pattern (`44.dp × 24.dp`)

Key ColorOS 17 Switch constants:
- Track: `44.dp × 24.dp`, `CircleShape` / `ContinuousCapsule`, `#0080FF` checked / `#D8DADC` unchecked (light)
- Thumb: `18.dp` white circle (`3.dp` inner inset)
- Motion: `toggleResponse = 0.4f`, `toggleBounce = 0.3f`, horizontal stretch `1.3x` at `p ≈ 0.347`
- Haptics: ROM constant **`302`** (`CouiHapticFeedback.Switch`) with `CLOCK_TICK` fallback

Always attach `Modifier.toggleable(...)` to the **entire row** with `indication = null`, sharing `MutableInteractionSource` with the `Switch` (`onCheckedChange = null`):

```kotlin
fun performCouiSwitchHaptic(view: android.view.View) {
    if (view.isAttachedToWindow && !view.performHapticFeedback(302)) {
        view.performHapticFeedback(HapticFeedbackConstants.CLOCK_TICK)
    }
}

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
            .heightIn(min = 52.dp)
            .toggleable(
                value = checked,
                enabled = enabled,
                role = Role.Switch,
                interactionSource = interactionSource,
                indication = null,
                onValueChange = { next ->
                    performCouiSwitchHaptic(view)
                    onCheckedChange(next)
                },
            )
            .padding(horizontal = 16.dp, vertical = 14.dp),
        verticalAlignment = Alignment.CenterVertically,
    ) {
        Column(modifier = Modifier.weight(1f).padding(end = 12.dp)) {
            Text(
                text = title,
                fontSize = 16.sp,
                fontWeight = FontWeight.Medium,
                color = CouiTheme.colors.onSurface,
            )
            if (!summary.isNullOrEmpty()) {
                Text(
                    text = summary,
                    fontSize = 13.sp,
                    color = CouiTheme.colors.onSurfaceVariantSummary,
                    modifier = Modifier.padding(top = 2.dp),
                )
            }
        }
        CouiSwitch(
            checked = checked,
            onCheckedChange = null, // Row owns click & semantics
            enabled = enabled,
        )
    }
}
```

---

## 4. Floating Detail TopBar & `42–44dp` Circular Frosted-Glass Back Button

Detail subpages (`Mobile network`, `Personal hotspot`, `Battery health`, `Manage eSIM`) float a **`60.dp`** top bar below `WindowInsets.statusBars` over the top of the `LazyColumn` using a subtle vertical surface scrim—with **no redundant right-bar icons**:

1. **`42.dp`–`44.dp` `CircleShape` frosted-glass back button:**
   - Fill: `#E6FFFFFF` (light) / `#26FFFFFF` (dark)
   - Border: `0.5.dp` vertical specular rim (`#FFFFFF -> #18000000` light, `#3DFFFFFF -> #14FFFFFF` dark)
   - Elevation: `4.dp`–`6.dp` soft ambient shadow (`ambientColor = #26000000`, `spotColor = #1C000000` in light; `0.dp` in dark)
   - Icon: Crisp `22.dp`–`24.dp` wide back arrow (`2.dp` rounded stroke)
   - Press motion: `spring(dampingRatio = 0.75f, stiffness = 500f)` scaling to `0.93f`
2. **Title:** `19.5.sp` `FontWeight.Medium`, `12.dp` right of the circular button, single-line ellipsis.

```kotlin
@Composable
fun CouiGlassBackButton(onBack: () -> Unit) {
    val colors = CouiTheme.colors
    val interactionSource = remember { MutableInteractionSource() }
    val isPressed by interactionSource.collectIsPressedAsState()
    val scale = remember { Animatable(1f) }
    LaunchedEffect(isPressed) {
        scale.animateTo(if (isPressed) 0.93f else 1f, spring(dampingRatio = 0.75f, stiffness = 500f))
    }

    val glassFill = if (colors.isDark) Color(0x26FFFFFF) else Color(0xE6FFFFFF)
    val glassBorder = if (colors.isDark) {
        Brush.verticalGradient(listOf(Color(0x3DFFFFFF), Color(0x14FFFFFF)))
    } else {
        Brush.verticalGradient(listOf(Color(0xFFFFFFFF), Color(0x18000000)))
    }

    Box(
        modifier = Modifier
            .size(44.dp)
            .graphicsLayer { scaleX = scale.value; scaleY = scale.value }
            .shadow(
                elevation = if (colors.isDark) 0.dp else 6.dp,
                shape = CircleShape,
                ambientColor = Color(0x26000000),
                spotColor = Color(0x1C000000),
            )
            .clip(CircleShape)
            .background(glassFill)
            .border(width = 0.5.dp, brush = glassBorder, shape = CircleShape)
            .clickable(interactionSource = interactionSource, indication = null, onClick = onBack),
        contentAlignment = Alignment.Center,
    ) {
        val arrowColor = colors.onSurface
        Canvas(modifier = Modifier.size(23.dp)) {
            val stroke = 2.dp.toPx()
            val cy = size.height / 2f
            val startX = size.width * 0.16f
            val endX = size.width * 0.84f
            val headLen = size.width * 0.30f
            drawLine(arrowColor, Offset(startX + stroke * 0.35f, cy), Offset(endX, cy), stroke, StrokeCap.Round)
            val chevron = Path().apply {
                moveTo(startX + headLen, cy - headLen)
                lineTo(startX, cy)
                lineTo(startX + headLen, cy + headLen)
            }
            drawPath(chevron, arrowColor, style = Stroke(stroke, cap = StrokeCap.Round, join = StrokeJoin.Round))
        }
    }
}
```

---

## 5. ColorOS 17 Rubber-Band Bouncy Overscroll (Single-Screen & Multi-Screen)

Native ColorOS 17 lists smoothly translate vertically past top and bottom boundaries with progressive exponential resistance and elastic spring rebound—never stretching typography like AOSP `StretchOverscroll`.

Always disable default Compose overscroll (`LocalOverscrollFactory provides null`) and attach `Modifier.couiBouncyOverscroll()` directly to the `LazyColumn` (works even when all cards fit on one screen):

```kotlin
fun Modifier.couiBouncyOverscroll(
    maxOffset: Dp = 148.dp,
    dampingFactor: Dp = 320.dp,
): Modifier = composed {
    val density = LocalDensity.current
    val scope = rememberCoroutineScope()
    val maxOffsetPx = remember(density, maxOffset) { with(density) { maxOffset.toPx() } }
    val kPx = remember(density, dampingFactor) { with(density) { dampingFactor.toPx() } }
    val overscrollOffset = remember { Animatable(0f) }
    var rawDrag by remember { mutableFloatStateOf(0f) }

    fun rubberBand(raw: Float): Float =
        if (raw == 0f) 0f else sign(raw) * maxOffsetPx * (1f - exp(-abs(raw) / kPx))

    fun releaseSpring() {
        rawDrag = 0f
        if (abs(overscrollOffset.value) > 0.5f) {
            scope.launch {
                overscrollOffset.animateTo(0f, spring(dampingRatio = 0.72f, stiffness = 380f))
            }
        }
    }

    val connection = remember(maxOffsetPx, kPx) {
        object : NestedScrollConnection {
            override fun onPreScroll(available: Offset, source: NestedScrollSource): Offset {
                if (source != NestedScrollSource.UserInput || available.y == 0f) return Offset.Zero
                if (rawDrag != 0f && sign(available.y) != sign(rawDrag)) {
                    val prev = rawDrag
                    val next = prev + available.y
                    val consumedY = if (sign(next) != sign(prev)) { rawDrag = 0f; -prev } else { rawDrag = next; available.y }
                    scope.launch { overscrollOffset.snapTo(rubberBand(rawDrag)) }
                    return Offset(0f, consumedY)
                }
                return Offset.Zero
            }
            override fun onPostScroll(consumed: Offset, available: Offset, source: NestedScrollSource): Offset {
                if (source != NestedScrollSource.UserInput || available.y == 0f) return Offset.Zero
                rawDrag += available.y
                scope.launch { overscrollOffset.snapTo(rubberBand(rawDrag)) }
                return Offset(0f, available.y)
            }
            override suspend fun onPreFling(available: Velocity): Velocity {
                if (abs(overscrollOffset.value) > 0.5f || rawDrag != 0f) {
                    releaseSpring()
                    return Velocity(0f, available.y)
                }
                return Velocity.Zero
            }
            override suspend fun onPostFling(consumed: Velocity, available: Velocity): Velocity {
                if (abs(overscrollOffset.value) > 0.5f || rawDrag != 0f) releaseSpring()
                return Velocity.Zero
            }
        }
    }

    this
        .nestedScroll(connection)
        .pointerInput(Unit) {
            awaitPointerEventScope {
                while (true) {
                    val event = awaitPointerEvent(PointerEventPass.Final)
                    if (event.changes.none { it.pressed } && (abs(overscrollOffset.value) > 0.5f || rawDrag != 0f)) {
                        releaseSpring()
                    }
                }
            }
        }
        .graphicsLayer { translationY = overscrollOffset.value }
}
```

Usage:
```kotlin
CompositionLocalProvider(LocalOverscrollFactory provides null) {
    LazyColumn(
        modifier = Modifier.fillMaxSize().couiBouncyOverscroll().navigationBarsPadding(),
        contentPadding = PaddingValues(start = 16.dp, end = 16.dp, top = statusBarTop + 68.dp, bottom = 24.dp),
    ) { /* CouiCard items */ }
}
```

---

## 6. `AndroidLiquidGlass` & `ColorOsToolbarGlass` Floating Pill Docks

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

## 7. Dialogs, Loading Dialogs & Bottom Sheets

- **Dialogs:** `24.dp`–`28.dp` continuous squircle corners, `20.dp`–`24.dp` content padding, two horizontal pill buttons (`ContinuousCapsule`, `minHeight = 46.dp`) that auto-stack vertically when 3 actions or long localized labels are present.
- **Loading Dialog:** Non-dismissible `24.dp` continuous squircle card with COUI indeterminate spinner and a concise title + subtitle.
- **Bottom Sheets:** `28.dp` continuous top corners, `#FFFFFF` (Light) / `#1C1C1E` (Dark) fill over `#33000000` / `#99000000` scrim, `navigationBars` + `ime` bottom padding.
