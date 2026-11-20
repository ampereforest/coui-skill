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

    // Option B: Kyant0 continuous G2 squircle geometry & Liquid Glass backdrop refraction
    implementation("io.github.kyant0:capsule:2.1.0")
    implementation("io.github.kyant0:backdrop:2.0.1")
}
```

> **Kotlin `2.3.x` vs `2.4.x` Vendoring Note for `com.kyant.backdrop`:**
> Upstream `io.github.kyant0:backdrop:2.0.1` is published with Kotlin `2.4.10` / Compose `1.12.0` metadata. For Android projects on Kotlin `2.2.x`–`2.3.x` / Compose `1.10.x`, vendor the 30 pure-Kotlin files from `Kyant0/AndroidLiquidGlass` (`backdrop/src/{commonMain,androidMain}/kotlin/com/kyant/backdrop/`) directly into `app/src/main/java/com/kyant/backdrop/` alongside `com.kyant.capsule`, merging the 4 `androidMain` `actual` files (`Platform.kt`, `RuntimeShader.kt`, `internal/Paint.kt`, `internal/RenderEffect.kt`) and removing the optional `com.kyant.shapes.RoundedRectangularShape` branch in `effects/Lens.kt` and `@Language("AGSL")` annotation in `internal/Shaders.kt`.

---

## 2. Continuous Squircle Cards, Text-First Rows (No Inline Status Pills) & Inset Dividers

Never tint secondary clickable cards gray, omit bulky left icon boxes on standard Settings subpage rows so all row titles align cleanly at `16.dp` internal padding (`32.dp` from screen edge), and **NEVER place inline colored status pills** (`"eSIM"`, `"Active"`, `"SIM 2"`, etc.) next to preference row titles. Use **`1.dp` `#14000000`** (light) / **`#24FFFFFF`** (dark) dividers inset **`16.dp`** horizontally **only between** adjacent rows inside multi-row cards.

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

## 4. Floating Detail TopBar & `46.dp` `Kyant0/AndroidLiquidGlass` Circular Back Button (`LayerBackdrop` Scroll-Under Refraction)

Detail subpages (`Mobile network`, `Personal hotspot`, `Battery health`, `Manage eSIM`) float a **`64.dp`** top bar below `WindowInsets.statusBars` as a **sibling overlaid above** a full-screen `Modifier.layerBackdrop(backdrop)` scrollable container so cards slide *under* the header and visibly refract inside the `46.dp` circular glass back button—with **no redundant right-bar icons**:

> **Critical Layer Architecture Rule:**
> Attach `Modifier.layerBackdrop(backdrop)` to the scrollable background container (`Box` wrapping `LazyColumn`), and place `CouiTopAppBar(backdrop = backdrop, ...)` as a **sibling** above it inside the root `Box`. Never place a `drawBackdrop` composable inside its own `layerBackdrop` subtree.

1. **`46.dp` `CircleShape` `Kyant0/AndroidLiquidGlass` (`com.kyant.backdrop`) back button:**
   - **Real-time effects (`API >= 33`):** `vibrancy()`, `blur(4f.dp.toPx())`, `lens(refractionHeight = 10f.dp.toPx(), refractionAmount = 20f.dp.toPx(), depthEffect = true)`
   - **Specular rim (`Highlight`):** `Highlight(width = 0.75.dp, alpha = if (isDark) 0.45f else 0.90f, style = HighlightStyle.Default)` — crisp bright `#FDFDFD`–`#FEFEFE` rim with zero harsh dark `1px` border ring
   - **Diffuse ambient drop shadow (`Shadow`):** `Shadow(radius = 18.dp, offset = DpOffset(0.dp, 5.dp), color = Color.Black.copy(alpha = if (isDark) 0.32f else 0.09f))` matching the wide soft downward shadow (`+19px` peak fading to `+49px`) measured on `hotspot-ui.jpg`
   - **Translucent surface fill (`onDrawSurface`):** `drawRect(if (isDark) Color(0x28FFFFFF) else Color(0xB8F6F7F9))` (`#F6F7F9` frosted glass in Light Mode, `#28FFFFFF` in Dark Mode)
   - **Icon geometry:** Sleek wide-winged **`24.dp`–`25.dp`** back arrow canvas (`~20.3dp × 16.6dp` glyph bounds) with refined **`1.65.dp`** rounded stroke (`startX = 0.14f`, `endX = 0.86f`, `headLen = 0.30f`, `headHeight = headLen * 0.92f`)
   - **Press motion:** `spring(dampingRatio = 0.70f, stiffness = 600f)` scaling to `0.94f`
2. **Translucent Top Scrim & Title:**
   - Top bar uses a soft translucent vertical gradient scrim (`colors.surface.copy(alpha = 0.90f)` → `colors.surface.copy(alpha = 0.65f)` → `Color.Transparent`) so scrolled cards remain visible as they pass behind the `46.dp` Liquid Glass button.
   - Title: `19.5.sp`–`20.sp` `FontWeight.Medium`, `12.dp` right of the `46.dp` circular button, single-line ellipsis.

```kotlin
@Composable
fun CouiBackButton(
    onClick: () -> Unit,
    backdrop: Backdrop? = null,
    modifier: Modifier = Modifier,
) {
    val colors = CouiTheme.colors
    val interactionSource = remember { MutableInteractionSource() }
    val isPressed by interactionSource.collectIsPressedAsState()
    val scale = remember { Animatable(1f) }
    LaunchedEffect(isPressed) {
        scale.animateTo(
            targetValue = if (isPressed) 0.94f else 1f,
            animationSpec = spring(dampingRatio = 0.70f, stiffness = 600f),
        )
    }

    val isDark = colors.isDark
    val surfaceColor = if (isDark) Color(0x28FFFFFF) else Color(0xB8F6F7F9)

    val glassModifier = if (backdrop != null && Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        Modifier.drawBackdrop(
            backdrop = backdrop,
            shape = { CircleShape },
            effects = {
                vibrancy()
                blur(4f.dp.toPx())
                lens(
                    refractionHeight = 10f.dp.toPx(),
                    refractionAmount = 20f.dp.toPx(),
                    depthEffect = true,
                )
            },
            highlight = {
                Highlight(
                    width = 0.75.dp,
                    alpha = if (isDark) 0.45f else 0.90f,
                    style = HighlightStyle.Default,
                )
            },
            shadow = {
                Shadow(
                    radius = 18.dp,
                    offset = DpOffset(0.dp, 5.dp),
                    color = Color.Black.copy(alpha = if (isDark) 0.32f else 0.09f),
                )
            },
            onDrawSurface = {
                drawRect(surfaceColor)
            },
        )
    } else {
        // API < 33 graceful fallback
        Modifier
            .shadow(
                elevation = if (isDark) 0.dp else 10.dp,
                shape = CircleShape,
                ambientColor = Color(0x24000000),
                spotColor = Color(0x1E000000),
            )
            .clip(CircleShape)
            .background(if (isDark) Color(0x28FFFFFF) else Color(0xCCF6F7F9))
    }

    Box(
        modifier = modifier
            .size(46.dp)
            .graphicsLayer {
                scaleX = scale.value
                scaleY = scale.value
            }
            .then(glassModifier)
            .clickable(
                interactionSource = interactionSource,
                indication = null,
                onClick = onClick,
            ),
        contentAlignment = Alignment.Center,
    ) {
        val arrowColor = colors.onSurface
        Canvas(modifier = Modifier.size(24.dp)) {
            val stroke = 1.65.dp.toPx()
            val cy = size.height * 0.50f
            val startX = size.width * 0.14f
            val endX = size.width * 0.86f
            val headLen = size.width * 0.30f
            val headHalfH = headLen * 0.92f

            drawLine(
                color = arrowColor,
                start = Offset(startX + stroke * 0.35f, cy),
                end = Offset(endX, cy),
                strokeWidth = stroke,
                cap = StrokeCap.Round,
            )
            val chevron = Path().apply {
                moveTo(startX + headLen, cy - headHalfH)
                lineTo(startX, cy)
                lineTo(startX + headLen, cy + headHalfH)
            }
            drawPath(
                path = chevron,
                color = arrowColor,
                style = Stroke(width = stroke, cap = StrokeCap.Round, join = StrokeJoin.Round),
            )
        }
    }
}
```

### Root Screen Layout with `rememberLayerBackdrop()` Scroll-Under Refraction

```kotlin
val backdrop = rememberLayerBackdrop()
val statusBarTop = WindowInsets.statusBars.asPaddingValues().calculateTopPadding()

Box(
    modifier = Modifier
        .fillMaxSize()
        .background(CouiTheme.colors.surface), // #F0F1F3 light / #000000 dark
) {
    // 1. Scrollable content layer captured by LayerBackdrop
    Box(
        modifier = Modifier
            .fillMaxSize()
            .layerBackdrop(backdrop)
            .background(CouiTheme.colors.surface),
    ) {
        CompositionLocalProvider(LocalOverscrollFactory provides CouiOverscrollFactory) {
            LazyColumn(
                state = listState,
                modifier = Modifier
                    .fillMaxSize()
                    .navigationBarsPadding(),
                contentPadding = PaddingValues(
                    start = 16.dp,
                    end = 16.dp,
                    top = statusBarTop + 68.dp,
                    bottom = 28.dp,
                ),
            ) {
                // CouiCard items...
            }
        }
    }

    // 2. Sibling floating ColorOS 17 Liquid Glass Top App Bar
    CouiTopAppBar(
        title = "Manage eSIM",
        onBack = onBack,
        backdrop = backdrop,
        modifier = Modifier.align(Alignment.TopCenter),
    )
}
```

---

## 5. ColorOS 17 Native `OverscrollEffect` (`SpringOverScroller` RK4 Physics)

Native ColorOS 17 lists smoothly translate vertically past top and bottom boundaries with progressive resistance (`0.4 * (1 - |offset| / screenSize)`) and RK4 spring rebound (`SPRING_BACK_FRICTION = 12.19f`, `SPRING_BACK_TENSION = 16f`, vertical tension multiple `2.15f`) — never stretching typography like AOSP `StretchOverscroll`.

> **Single-Screen vs Multi-Screen Rule (verified on live ColorOS 17 + `suqi8/coui`):**
> - **Multi-screen lists** (content taller than viewport): provide `CompositionLocalProvider(LocalOverscrollFactory provides CouiOverscrollFactory)`.
> - **Single-screen lists** (all cards fit inside viewport, `!canScrollForward && !canScrollBackward`): Compose's `Scrollable` skips `OverscrollEffect.applyToScroll`. Pair `LocalOverscrollFactory provides null` with `Modifier.couiOverScrollVertical()` (`DelegatingNode` + `nestedScrollModifierNode` + `LayoutModifierNode` `placeWithLayer` sharing the exact same `CouiReboundOverScroller` RK4 physics — zero `pointerInput` hacks!).
> Never attach `Modifier.nestedScroll().pointerInput().graphicsLayer { translationY = ... }` on the outside of a `LazyColumn`. Translating the same node that processes pointer input feeds offsets back into touch coordinates and races between `ACTION_UP` and `onPreFling`, causing inconsistent instant snap-backs.
> Always implement Jetpack Compose's native **`OverscrollEffect` + `OverscrollFactory`** (`LocalOverscrollFactory provides CouiOverscrollFactory`, as pioneered in `suqi8/coui`) with an internal `LayoutModifierNode` (`placeable.placeWithLayer(0, 0) { translationY = round(effect.offsetY); clip = true }`). Compose's `Scrollable` guarantees `applyToFling` runs on every release (even 0-velocity slow releases) and passes leftover fling velocity cleanly for over-fling overshoot (`37.31%` screen saturation) + edge haptic `307`.

```kotlin
object CouiOverscrollFactory : OverscrollFactory {
    override fun createOverscrollEffect(): OverscrollEffect = CouiOverscrollEffect()
    override fun equals(other: Any?): Boolean = this === other
    override fun hashCode(): Int = 17
}

// Usage (replaces AOSP StretchOverscroll cleanly across every LazyColumn / scrollable):
CompositionLocalProvider(LocalOverscrollFactory provides CouiOverscrollFactory) {
    LazyColumn(
        modifier = Modifier.fillMaxSize().navigationBarsPadding(),
        contentPadding = PaddingValues(start = 16.dp, end = 16.dp, top = statusBarTop + 68.dp, bottom = 28.dp),
    ) { /* CouiCard items */ }
}
```

Key physics constants (1:1 port of ColorOS 17 `COUIRecyclerView` / `SpringOverScroller` via `suqi8/coui`):
- **Drag resistance:** `delta * 0.4f * (1f - min(|offset| / screenHeight, 1f))`
- **Spring-back RK4:** Origami `friction = 12.19f`, `tension = 16f * 2.15f` (vertical) / `16f * 3.2f` (horizontal), stepped via `withFrameNanos`
- **Over-fling saturation:** `over * limit / sqrt(over² + limit²)` where `limit = screenHeight * 0.3731f`
- **Reverse fling threshold:** `6000 px/s` (flings back through edge and hands leftover velocity to `LazyColumn`)
- **Edge haptic:** `LocalView.current.performHapticFeedback(307)` on overscroll release (`dragTravel >= 10px`) or fling edge hit

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
