# Light Mode Visual Bug Audit & Fix List

This document lists all visual bugs and contrast issues identified in Light Mode from the desktop screenshots, along with the planned fixes.

---

## 1. Performance Stats Card (image_518)
*   **Issue:** The stats numerical values ("0", "0s", "7") are rendering in white (`#FFFFFF`) on a white card background, making them completely invisible.
*   **Fix:** Locate the TextBlocks rendering the stats value inside the Performance stats panel or view and bind their `Foreground` property to `{DynamicResource TokenTextPrimaryBrush}`.

---

## 2. Global Combobox Styles (image_520, image_523, image_524)
*   **Issue:** Selection comboboxes (e.g., Macro Selection, Master Kill Switch Hotkey, App Switch Speed, Playback Feel) have dark backgrounds with white text. This breaks the cohesive light-theme aesthetic.
*   **Fix:** Locate the global Combobox style (or the comboboxes inside the views) and ensure that in Light Mode, they use light backgrounds (e.g., `{DynamicResource TokenInputBgBrush}`) and dark text (e.g., `{DynamicResource TokenTextPrimaryBrush}`).

---

## 3. Macro Editor Card Drag Handles & Delete Icons (image_519)
*   **Issue:** The three-line drag handle icon on the left of action cards and the `x` delete icon on the right are white/light-grey, making them almost invisible against the white card background.
*   **Fix:** Locate the drag handle and delete button elements inside the card templates (e.g. `MacroStepCard.xaml` or editor views) and change their Foreground/Opacity to `{DynamicResource TokenTextMutedBrush}` or `{DynamicResource TokenTextDimBrush}`.

---

## 4. Editor Toolbar Icons (image_519)
*   **Issue:** Toolbar buttons like `Undo`, `Redo`, `Filter`, and `Timer` have white icons inside them, which are barely visible on the light grey button background.
*   **Fix:** Change the icon foreground inside the toolbar buttons to `{DynamicResource TokenTextSecondaryBrush}`.

---

## 5. Text Snippets Settings Gear (image_521)
*   **Issue:** The settings gear icon inside each snippet card is white, rendering it invisible against the white card background.
*   **Fix:** Change the gear icon TextBlock Foreground to `{DynamicResource TokenTextMutedBrush}`.

---

## 6. Keyboard Settings Tab (image_522)
*   **Issue:**
    *   Modifier keys (SHIFT, CTRL, ALT, WIN) have black backgrounds with dark blue text (completely unreadable).
    *   Regular keys have light-grey backgrounds with white text labels (almost unreadable).
*   **Fix:** Locate the virtual keyboard style or control in `Views/SettingsDashboardView.xaml` (Keyboard section) or the custom Keyboard key styling and ensure that:
    *   Active/selected modifiers use readable text colors.
    *   Default keys use a dark label color (e.g., `{DynamicResource TokenTextPrimaryBrush}`).

---

## 7. App UI Zoom Buttons (image_525)
*   **Issue:** The unselected zoom preset values ("80%", "90%") are white on a light-grey card background, making them invisible.
*   **Fix:** Set the unselected button foregrounds to `{DynamicResource TokenTextSecondaryBrush}`.
