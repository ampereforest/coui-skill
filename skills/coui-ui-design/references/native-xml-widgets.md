# ColorOS 17 Native Android View / XML & System APK Injection Guide

When modifying or extending ColorOS system screens (e.g., Settings, `com.oplus.battery`, `com.android.phone` / `TeleService`), **reuse ColorOS 17's native `com.coui.appcompat.*` XML widgets** directly inside the existing `PreferenceScreen` so patched screens inherit ROM dark mode, COUI curves, section spacing, and localization natively.

---

## 1. Native COUI `PreferenceScreen` Pattern (`com.coui.appcompat.preference.*`)

Extracted directly from ColorOS 17 system APKs (`battery_health_preference.xml`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.preference.PreferenceScreen
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:title="@string/screen_title">

    <!-- Native COUI informational instruction card -->
    <com.coui.appcompat.preference.COUIPreferenceCategory android:key="info_group">
        <com.coui.appcompat.card.COUICardInstructionPreference
            android:key="instruction_card"
            app:instructionCardType="description" />
    </com.coui.appcompat.preference.COUIPreferenceCategory>

    <!-- Native COUI grouped switch category -->
    <com.coui.appcompat.preference.COUIPreferenceCategory android:key="feature_category">
        <com.coui.appcompat.preference.COUISwitchPreference
            android:key="feature_switch"
            android:title="@string/feature_title"
            android:summary="@string/feature_summary" />
    </com.coui.appcompat.preference.COUIPreferenceCategory>

    <!-- Native COUI footer explanation text underneath the category card -->
    <com.coui.appcompat.preference.COUISpannablePreference
        android:key="feature_footer_note"
        android:layout="@layout/battery_health_content"
        android:selectable="false"
        android:summary="@string/feature_footer_description" />

    <!-- Native COUI navigation / jump row with trailing chevron -->
    <com.coui.appcompat.preference.COUIPreferenceCategory android:key="more_category">
        <com.coui.appcompat.preference.COUIJumpPreference
            android:key="more_jump_pref"
            android:title="@string/more_title"
            android:summary="@string/more_summary" />
    </com.coui.appcompat.preference.COUIPreferenceCategory>

</androidx.preference.PreferenceScreen>
```

### Widget Reference Table

| Class | Role in ColorOS 17 System Settings |
|---|---|
| `com.coui.appcompat.preference.COUIPreferenceCategory` | Renders the white/dark-translucent `24dp` continuous squircle card grouping its child preferences and auto-inserts `0.33dp` internal dividers |
| `com.coui.appcompat.preference.COUISwitchPreference` | Native `44dp × 24dp` COUI animated switch row with whole-row touch & `302` ROM haptic |
| `com.coui.appcompat.preference.COUISpannablePreference` | Non-selectable (`android:selectable="false"`) explanatory footer supporting clickable spans below a category card |
| `com.coui.appcompat.preference.COUIJumpPreference` | Navigation row with right chevron (`>`) and optional trailing status text |
| `com.coui.appcompat.card.COUICardInstructionPreference` | Elevated description/warning banner (`app:instructionCardType="description"`) |
| `com.coui.appcompat.preference.COUIRecommendedPreference` | Footer `"You might be looking for:"` neutral pill suggestions block |

---

## 2. Inline `COUISectionSeekBar` Slider Card Pattern

When toggling a COUI switch reveals an inline segmented slider directly beneath it (such as ColorOS Battery Health's `80% – 100%` Custom Charging Limit picker, `regular_charge_power_level_picker.xml`), bind a custom `Preference` using `CardListSelectedItemLayout` + `COUISectionSeekBar`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.oplus.powermanager.fuelgaue.basic.customized.CardListSelectedItemLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content">

    <LinearLayout
        android:id="@id/seekbar_layout"
        android:layout_gravity="center_horizontal"
        android:orientation="vertical"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingVertical="10.0dip">

        <RelativeLayout
            android:gravity="center_vertical"
            android:orientation="horizontal"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:paddingHorizontal="32.0dip">

            <TextView
                android:id="@id/slider_label"
                android:textAppearance="?android:textAppearanceMedium"
                android:textSize="16.0sp"
                android:textFontWeight="500"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentStart="true"
                android:layout_toStartOf="@id/slider_value"
                android:layout_centerVertical="true"
                android:textDirection="locale"
                android:text="@string/slider_title" />

            <TextView
                android:id="@id/slider_value"
                android:textAppearance="@style/couiTextAppearanceSmallButton"
                android:textSize="12.0sp"
                android:textFontWeight="500"
                android:textColor="?couiColorPrimaryNeutral"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentEnd="true"
                android:layout_centerVertical="true"
                android:textDirection="locale" />
        </RelativeLayout>

        <LinearLayout
            android:gravity="center_vertical"
            android:layout_gravity="center_horizontal"
            android:orientation="horizontal"
            android:clipChildren="false"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="16.0dip"
            android:layout_marginRight="16.0dip">

            <com.coui.appcompat.seekbar.COUISectionSeekBar
                android:id="@id/custom_section_seek_bar"
                android:layout_width="0.0dip"
                android:layout_height="36.0dip"
                android:layout_marginLeft="16.0dip"
                android:layout_marginRight="16.0dip"
                android:layout_weight="1.0" />
        </LinearLayout>
    </LinearLayout>
</com.oplus.powermanager.fuelgaue.basic.customized.CardListSelectedItemLayout>
```

---

## 3. System APK Theme & SELinux Engineering Rules

- **Use ColorOS Theme Tokens, Never Hardcoded Hex in System XMLs:** Reference `?couiColorPrimaryNeutral`, `?couiColorPrimary`, `@style/couiTextAppearanceSmallButton`, and `coui_section_seekbar` dimens so Light, Dark AMOLED, and Wallpaper tinting work automatically.
- **Preserving System APK Platform Signatures / Bind-Mounting:**
  - `system_ext` / `system` APKs like `Battery.apk` or `TeleService.apk` often share `android.uid.system` or `android.uid.phone`. If modifying DEX/XML inside an existing system APK during KernelSU prototyping without full framework test-key re-signing, keep original signatures aligned or test with proper platform signing / framework hook.
- **Avoid Widening `system_app` SELinux Rules Unnecessarily:**
  - When a patched Settings/Battery Preference needs to control a root daemon (such as `lmkd`, `displayfeature`, or `mtb`), persist user state via `Settings.Global.putInt(...)` (already permitted to `android.uid.system`) and observe it from a small root service/watcher rather than exposing restricted `sysfs` or `resetprop` sockets directly to UI domains.
- **Search Index Integration:** ColorOS `PowerSearchIndexablesProvider` / `BaseSearchIndexProvider` caches Settings search entries. Register any newly added preference key there (or provide stable preference keys matching `res/xml/`) so Settings search works cleanly.
