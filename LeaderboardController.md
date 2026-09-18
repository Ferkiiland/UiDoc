# LeaderboardController

`LocalScript` that manages **tab switching** inside leaderboard GUIs. It shows/hides panels based on the clicked button and updates the visual state (stroke, icon transparency, gradient) of the selected tab.

> ℹ️ Works with any `ScreenGui` following the expected structure: a `Buttons.Container` for tabs and `CanvasGroup` panels with matching names.

---

## 📦 Requirements

| Path | Type | Description |
|------|------|-------------|
| `PlayerGui.MapLeaderboard` | `ScreenGui` | First leaderboard screen |
| `PlayerGui.RankMapLeaderboard` | `ScreenGui` | Second leaderboard screen |
| `<Leaderboard>.Buttons.Container` | `Folder` / `Frame` | Container with tab buttons |
| `<Leaderboard>.Header` | `Frame` / `CanvasGroup` | Header with per-tab elements |
| `<Leaderboard>.<PanelName>` | `CanvasGroup` | Panels to show/hide, name must match the button name |

### Expected structure

```
PlayerGui
├── MapLeaderboard (ScreenGui)
│   ├── Header
│   │   ├── Bg             ◄── excluded
│   │   ├── ImageLabel     ◄── excluded
│   │   └── <PanelName>    ◄── shown/hidden per tab
│   ├── Buttons
│   │   └── Container
│   │       ├── Tab1       ◄── TextButton / ImageButton
│   │       │   ├── Icon
│   │       │   ├── UIStroke
│   │       │   └── TextLabel
│   │       └── Tab2
│   └── <PanelName>        ◄── CanvasGroup shown when Tab matches
│
└── RankMapLeaderboard (ScreenGui)
    └── (same structure)
```

---

## 🎯 What this script does

The script performs **four main functions**:

### 1. Defines excluded containers

Elements inside `Header` or top-level children that should **not** be treated as panels/headers.

```lua
local EXCLUDED_CANVAS = {
	Header  = true,
	Buttons = true,
}

local EXCLUDED_HEADER = {
	Bg         = true,
	ImageLabel = true,
}
```

- **`EXCLUDED_CANVAS`** — top-level children ignored by the panel logic.
- **`EXCLUDED_HEADER`** — header children ignored by the header toggle.

### 2. Updates button visual state

When a tab is selected, `setButtonSelected` adjusts:

| Element | Selected | Unselected |
|---------|----------|------------|
| `Icon.ImageTransparency` | `0` | `0.5` |
| `UIStroke.Color` | `SELECTED_STROKE_COLOR` (white) | `UNSELECTED_STROKE_COLOR` (grey) |
| `UIGradient.Enabled` | `true` | `false` |

The function checks for `UIGradient` inside `UIStroke`, `TextLabel` and `Icon`.

### 3. Shows the correct panel

On button click, `showPanel(buttonName)`:

1. Iterates over top-level children of the leaderboard.
2. Shows the `CanvasGroup` whose `Name` matches `buttonName`.
3. Hides the rest (excluding `EXCLUDED_CANVAS`).

```lua
for _, child in ipairs(leaderboard:GetChildren()) do
	if child:IsA("CanvasGroup") and not EXCLUDED_CANVAS[child.Name] then
		child.Visible = (child.Name == buttonName)
	end
end
```

### 4. Shows the correct header elements

Also toggles `Frame` / `CanvasGroup` children inside `Header` (excluding `EXCLUDED_HEADER`) so the header updates with the active tab.

```lua
for _, child in ipairs(header:GetChildren()) do
	if not EXCLUDED_HEADER[child.Name] then
		if child:IsA("Frame") or child:IsA("CanvasGroup") then
			child.Visible = (child.Name == buttonName)
		end
	end
end
```

### 5. Highlights the selected button

Finally, iterates over `Buttons.Container` children and updates each button's visual state.

```lua
for _, button in ipairs(buttonsContainer:GetChildren()) do
	if button:IsA("TextButton") or button:IsA("ImageButton") then
		setButtonSelected(button, button.Name == buttonName)
	end
end
```

---

## ⚙️ Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `EXCLUDED_CANVAS` | `{ Header, Buttons }` | Top-level children ignored as panels |
| `EXCLUDED_HEADER` | `{ Bg, ImageLabel }` | Header children ignored as toggleable |
| `SELECTED_STROKE_COLOR` | `Color3.fromRGB(255, 255, 255)` | White stroke for selected |
| `UNSELECTED_STROKE_COLOR` | `Color3.fromRGB(70, 70, 70)` | Grey stroke for unselected |

---

## 🔄 Execution flow

```
[Script starts]
   │
   ├─► Waits for PlayerGui
   │
   ├─► Waits for MapLeaderboard
   │     └─► setupLeaderboard():
   │           ├─► Gets Buttons.Container and Header
   │           ├─► Defines showPanel(buttonName):
   │           │     ├─► Toggles top-level CanvasGroups
   │           │     ├─► Toggles Header children
   │           │     └─► Updates button states
   │           └─► Connects MouseButton1Click on each button
   │
   ├─► Waits for RankMapLeaderboard
   │     └─► setupLeaderboard(): (same setup)
   │
   └─► Done
```

---

## 🧩 Main functions

### `setButtonSelected(button, selected)`

Updates the visual state of a tab button:

- `Icon.ImageTransparency` → `0` (selected) / `0.5` (unselected).
- `UIStroke.Color` → white / grey.
- `UIGradient.Enabled` → `true` / `false` (on `UIStroke`, `TextLabel`, `Icon`).

### `setupLeaderboard(leaderboard)`

Full setup for one leaderboard screen:

- Resolves `Buttons.Container` and `Header`.
- Defines `showPanel(buttonName)`.
- Connects every button in `Buttons.Container` to `showPanel(button.Name)`.
- Returns `showPanel` (currently unused by the caller).

### `showPanel(buttonName)`

Shows the panel and header elements whose name matches `buttonName`, hides the rest, and updates button visual states.

---

## 🎨 Customization

### Add a new leaderboard screen

```lua
local newLeaderboard = playerGui:WaitForChild("NewLeaderboard")
setupLeaderboard(newLeaderboard)
```

### Exclude more top-level containers

```lua
local EXCLUDED_CANVAS = {
	Header  = true,
	Buttons = true,
	Footer  = true,  -- new
}
```

### Exclude more header children

```lua
local EXCLUDED_HEADER = {
	Bg         = true,
	ImageLabel = true,
	Divider    = true,  -- new
}
```

### Change the selected stroke color

```lua
local SELECTED_STROKE_COLOR = Color3.fromRGB(0, 170, 255)  -- blue
```

### Change the unselected icon transparency

```lua
icon.ImageTransparency = selected and 0 or 0.75  -- more faded
```

---

## ⚠️ Considerations

- **Button names must match panel names** exactly. The comparison uses `child.Name == buttonName`.
- Only top-level children of the leaderboard that are `CanvasGroup` are treated as panels.
- Only `Frame` or `CanvasGroup` children of `Header` are toggled.
- Excluded names (`Header`, `Buttons`, `Bg`, `ImageLabel`) are skipped to avoid accidentally hiding structural elements.
- `UIGradient.Enabled` is only touched if a `UIGradient` exists inside `UIStroke`, `TextLabel` or `Icon`.
- If `Buttons.Container` or `Header` are missing, `WaitForChild` will yield indefinitely.
- The script runs **once per screen**; it does not react to dynamically added buttons or panels after startup.

---

## 📋 Quick reference

| Concept | Name | Required |
|---------|------|----------|
| First leaderboard | `MapLeaderboard` | ✅ |
| Second leaderboard | `RankMapLeaderboard` | ✅ |
| Tabs container | `Buttons.Container` | ✅ |
| Header | `Header` | ✅ |
| Excluded top-level | `Header`, `Buttons` | — |
| Excluded header | `Bg`, `ImageLabel` | — |
| Selected color | White (`255, 255, 255`) | — |
| Unselected color | Grey (`70, 70, 70`) | — |

---

## 🧪 Usage example

1. In `StarterGui`, create a `ScreenGui` named **`MapLeaderboard`**.
2. Inside, add:
   - A `Frame` named **`Header`** with a `Bg`, an `ImageLabel`, and one child per tab (e.g. `Kills`, `Wins`).
   - A `Frame` named **`Buttons`** containing a `Container` folder with `TextButton`s named exactly like the panels (e.g. `Kills`, `Wins`).
   - One `CanvasGroup` per tab, named exactly like the button (e.g. `Kills`, `Wins`).
3. Repeat the structure for `RankMapLeaderboard`.
4. Run the game. Clicking a tab button will:
   - Show its matching panel.
   - Hide the other panels.
   - Toggle the matching header element.
   - Update the selected button's icon, stroke and gradient.

> ✅ Both `MapLeaderboard` and `RankMapLeaderboard` are set up independently, so their tab state does not interfere.
