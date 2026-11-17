# coui-skill — ColorOS 17 (COUI) UI/UX Design & Engineering Skill

A comprehensive, `npx skills`-compatible Agent Skill for building **1:1 authentic ColorOS 17 (COUI) native Android user interfaces** across both **Jetpack Compose** and **native ColorOS `com.coui.appcompat.*` XML / Preference layouts**.

---

## Installation

### Using `npx skills` (Recommended — [skills.sh](https://skills.sh))

```bash
npx skills add ampereforest/coui-skill
```

Or using the full repository URL:

```bash
npx skills add https://github.com/ampereforest/coui-skill
```

To inspect available skills before installing:

```bash
npx skills add ampereforest/coui-skill --list
```

### Manual Installation

Copy `skills/coui-ui-design/` into your agent's local skills directory:

- **Antigravity / Gemini CLI:** `.agent/skills/coui-ui-design/` or `~/.gemini/antigravity/skills/coui-ui-design/`
- **Claude Code:** `.claude/skills/coui-ui-design/` or `~/.claude/skills/coui-ui-design/`
- **OpenCode / Codex / Generic Agent Skills:** `.agents/skills/coui-ui-design/`

---

## What's Inside

- [`skills/coui-ui-design/SKILL.md`](skills/coui-ui-design/SKILL.md) — Core Agent Skill entrypoint, workflow rules, decision matrix, and parity checklist.
- [`skills/coui-ui-design/references/design-tokens.md`](skills/coui-ui-design/references/design-tokens.md) — Complete COUI Light/Dark/AMOLED/Dynamic Material You color tokens, inverted surface/card hierarchy, typography, spacing, squircle radii, and ColorOS ROM haptic constants.
- [`skills/coui-ui-design/references/compose-components.md`](skills/coui-ui-design/references/compose-components.md) — Production Jetpack Compose blueprints covering `ItosEO/COUI-Kit`, `suqi8/coui`, `MiToverG422/ONextBox`, `Kyant0/Capsule`, and `Kyant0/AndroidLiquidGlass` (top bars, grouped preference cards, `44.dp × 24.dp` switches, sliders, dialogs/sheets, and liquid glass docks).
- [`skills/coui-ui-design/references/native-xml-widgets.md`](skills/coui-ui-design/references/native-xml-widgets.md) — ColorOS 17 system APK XML & `PreferenceFragment` / smali injection patterns (`COUIPreferenceCategory`, `COUISwitchPreference`, `COUISpannablePreference`, `COUISectionSeekBar`, `CardListSelectedItemLayout`, system COUI attributes).
- [`skills/coui-ui-design/references/icons-and-motion.md`](skills/coui-ui-design/references/icons-and-motion.md) — ColorOS 17 Classic & Flux `<monochrome>` adaptive icon safe-zone geometry, Olivelink press motion (`CouiPressMotion`), spring curves (`CouiSpring`), and bilingual English/Indonesian localization rules.

---

## Upstream Open-Source Credits

This skill synthesizes empirical ColorOS 16/17 design calibration together with patterns from outstanding open-source projects:

- **[ItosEO/COUI-Kit](https://github.com/ItosEO/COUI-Kit)** (`io.github.itoseo:coui`) — COUI theme tokens, `CouiColor`, `CouiShape`, `CouiSwitchDefaults` (`44.dp × 24.dp`), `CouiPressMotion`, `CouiSpring`, and `CouiHapticFeedback`.
- **[MiToverG422/ONextBox](https://github.com/MiToverG422/ONextBox)** — `ColorOs17DetailTopBar`, `ColorOsTopBarButton`, `ColorOsToolbarGlass`, `LiquidGlassBottomNavigationBar`, and adaptive/Flux launcher icon geometry.
- **[suqi8/coui](https://github.com/suqi8/coui)** (`io.github.suqi8.coui.kmp`) — Compose Multiplatform COUI component library (`BasicComponent`, `Card`, `Switch`, `Scaffold`) and ColorOS retheme specification.
- **[Kyant0/Capsule](https://github.com/Kyant0/Capsule)** (`io.github.kyant0:capsule`) — G2 continuous rounded rectangles (`ContinuousRoundedRectangle`, `ContinuousCapsule`).
- **[Kyant0/AndroidLiquidGlass](https://github.com/Kyant0/AndroidLiquidGlass)** (`io.github.kyant0:backdrop`) — Realtime backdrop blur, vibrancy, and lens refraction pipelines.

## License

MIT License — see [LICENSE](LICENSE). ColorOS and COUI are trademarks of OPPO / OnePlus; this repository is an independent community design & engineering reference.
