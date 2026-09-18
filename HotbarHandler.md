# ClientOverlayController

`LocalScript` that manages **overlay screens** (inventory, tasks, shop, settings, etc.) from the hotbar. It handles opening, closing, mutual exclusivity, close buttons and camera-aware buttons.

> ℹ️ Only one overlay can be active at a time. Opening a new one automatically closes the previous one.

---

## 📦 Requirements

| Path | Type | Description |
|------|------|-------------|
| `ReplicatedStorage.UI.UIAnimationsModule` | `ModuleScript` | Required module |
| `ReplicatedStorage.UI.UIAnimationsModule.Animator` | `ModuleScript` | Animator (for future/extended use) |
| `PlayerGui.MainGui` | `ScreenGui` | Main GUI that contains the hotbar and buttons |
| `PlayerGui.<OverlayScreen>` | `ScreenGui` | Target overlay screens defined in `overlayList` |
| `MainGui.Hotbar.Buttons.<Button>` | `TextButton` / `ImageButton` | Default button container |
| `MainGui.<ContainerPath>.<Button>` | `TextButton` / `ImageButton` | Custom path container (optional) |
| `<OverlayScreen>.CloseButton` | `TextButton` / `ImageButton` | Close button, must be tagged with `ToMain` |

### Expected structure

```
ReplicatedStorage
└── UI
    └── UIAnimationsModule
        └── Animator

PlayerGui
├── MainGui (ScreenGui)
│   ├── Hotbar
│   │   └── Buttons
│   │       ├── Inventory
│   │       ├── Tasks
│   │       ├── Play
│   │       ├── Shop
│   │       ├── Weapons
│   │       ├── Pass
│   │       └── Settings
│   ├── Party
│   │   └── 0InviteButton
│   └── TaskFrame
│       └── TaskButton
│
├── Backpack (ScreenGui)
│   └── CloseButton   ◄── tag: ToMain
├── TasksScreen (ScreenGui)
├── GamemodeSelector (ScreenGui)
├── ShopScreen (ScreenGui)
├── WeaponsScreen (ScreenGui)
├── BattlePass (ScreenGui)
├── Invite (ScreenGui)
└── SettingsScreen (ScreenGui)
```

---

## 🎯 What this script does

The script performs **five main functions**:

### 1. Defines overlays

Each entry in `overlayList` describes an overlay screen, its trigger button, and how to find it.

```lua
local overlayList = {
	{ Button = "Inventory",  guiName = "Backpack",          overlay = true, displayOrder = 12 },
	{ Button = "Tasks",      guiName = "TasksScreen",       overlay = true, displayOrder = 12 },
	{ Button = "Play",       guiName = "GamemodeSelector",  overlay = true, displayOrder = 12 },
	{ Button = "Shop",       guiName = "ShopScreen",        overlay = true, displayOrder = 12 },
	{ Button = "Weapons",    guiName = "WeaponsScreen",     overlay = true, displayOrder = 12 },
	{ Button = "Pass",       guiName = "BattlePass",        overlay = true, displayOrder = 12 },
	{ Button = "0InviteButton", guiName = "Invite",         overlay = true, displayOrder = 12, containerPath = { "Party" } },
	{ Button = "TaskButton",    guiName = "TasksScreen",    overlay = true, displayOrder = 12, containerPath = { "TaskFrame" } },
	{ Button = "Settings",   guiName = "SettingsScreen",    overlay = true, displayOrder = 12 },
}
```

**Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Button` | `string` | ✅ | Name of the trigger button |
| `guiName` | `string` | ✅ | Name of the `ScreenGui` to toggle |
| `overlay` | `boolean` | ✅ | Marks this as an overlay |
| `displayOrder` | `number` | ❌ | Sets `gui.DisplayOrder` |
| `containerPath` | `table<string>` | ❌ | Custom path inside `MainGui` to find the button |

### 2. Resolves buttons

Two lookup modes:

- **Default:** `MainGui.Hotbar.Buttons.<Button>`
- **Custom:** `MainGui.<containerPath...>.<Button>`

```lua
local function getButtonFor(def)
	if def.containerPath then
		local container = getContainer(def.containerPath)
		if not container then return nil end
		return container:FindFirstChild(def.Button)
	end

	local hotbar = mainGui:FindFirstChild("Hotbar")
	...
end
```

### 3. Toggles overlay state

Uses the `IsEnabled` attribute on the target `ScreenGui` to open/close it.

```lua
local function setOverlayState(gui, enabled)
	if enabled then
		if gui:GetAttribute("DisplayOrderOverride") == nil then
			gui:SetAttribute("DisplayOrderOverride", gui.DisplayOrder)
		end
	else
		if gui:GetAttribute("DisplayOrderOverride") ~= nil then
			gui.DisplayOrder = gui:GetAttribute("DisplayOrderOverride")
		end
	end

	gui:SetAttribute("IsEnabled", enabled)
end
```

When opening, it:
- Saves the original `DisplayOrder` in the attribute `DisplayOrderOverride`.
- Sets `IsEnabled = true`.

When closing, it:
- Restores the saved `DisplayOrder`.
- Sets `IsEnabled = false`.

### 4. Enforces mutual exclusivity

Only one overlay can be active at a time.

- Opening a new overlay closes the previous one.
- Clicking the same button toggles it off.
- Clicking a different button closes the previous and opens the new one.

### 5. Binds close buttons

Finds a `CloseButton` inside each overlay (with `FindFirstChild("CloseButton", true)`) that is tagged with `ToMain` and connects its `MouseButton1Click`.

```lua
closeButton.MouseButton1Click:Connect(function()
	if activeOverlay then
		local activeGui = getScreen(activeOverlay.guiName)
		if activeGui == gui then
			activeOverlay = nil
		end
	end
	setOverlayState(gui, false)
end)
```

---

## 🚫 Skipped buttons

If a button has **both** `ToMain` and `UsesCamera` tags, it is skipped — those are handled by the camera system.

```lua
if hasToMain and hasUsesCamera then
	return
end
```

---

## ⚙️ Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `TO_MAIN_TAG` | `"ToMain"` | Tag for close buttons |
| `USES_CAMERA_TAG` | `"UsesCamera"` | Tag that marks camera buttons (skipped) |

---

## 🔄 Execution flow

```
[Script starts]
   │
   ├─► Requires Animator from UIAnimationsModule
   │
   ├─► Waits for PlayerGui and MainGui
   │
   └─► For each overlay in overlayList:
         │
         ├─► getButtonFor(def) → finds the trigger button
         │
         ├─► getScreen(def.guiName) → finds the ScreenGui
         │
         ├─► Sets gui.DisplayOrder = def.displayOrder (if defined)
         │
         ├─► Sets gui:SetAttribute("IsEnabled", false)
         │
         ├─► Connects button.MouseButton1Click → toggleOverlay(def, gui)
         │
         └─► bindCloseButton(gui):
               ├─► Finds child named "CloseButton"
               ├─► Requires tag "ToMain"
               └─► Connects MouseButton1Click → close overlay
```

---

## 🧩 Main functions

### `getContainer(pathParts)`

Traverses `mainGui` following a path array and returns the final instance.

### `getButtonFor(def)`

Returns the trigger button for an overlay, either from `Hotbar.Buttons` or from a custom `containerPath`.

### `getScreen(guiName)`

Returns the `ScreenGui` in `PlayerGui` matching `guiName`, or `nil` if not a `ScreenGui`.

### `setOverlayState(gui, enabled)`

Toggles the overlay using the `IsEnabled` attribute. Preserves the original `DisplayOrder`.

### `closeActiveOverlay()`

Closes the currently active overlay, if any.

### `toggleOverlay(def, gui)`

Main toggle logic:

- If the target overlay is open → close it.
- If a different overlay is open → close it and open the new one.
- If none is open → open the target.

### `bindCloseButton(gui)`

Finds a `CloseButton` inside the overlay (with `ToMain` tag) and connects its click to close the overlay.

### `bindEntry(def)`

Full setup for one overlay entry: resolves the button, sets `DisplayOrder`, initializes `IsEnabled = false`, connects the toggle and binds the close button.

### `init()`

Iterates over `overlayList` and calls `bindEntry` for each entry.

---

## 🎨 Customization

### Add a new overlay

```lua
table.insert(overlayList, {
	Button       = "Ranked",
	guiName      = "RankedScreen",
	overlay      = true,
	displayOrder = 12,
})
```

Then make sure:
- `MainGui.Hotbar.Buttons.Ranked` exists.
- `PlayerGui.RankedScreen` exists.

### Add an overlay from a custom container

```lua
table.insert(overlayList, {
	Button        = "MyButton",
	guiName       = "MyScreen",
	overlay       = true,
	displayOrder  = 12,
	containerPath = { "CustomFolder", "SubFolder" },
})
```

Path resolves to: `MainGui.CustomFolder.SubFolder.MyButton`.

### Change `DisplayOrder` per overlay

```lua
{ Button = "Shop", guiName = "ShopScreen", overlay = true, displayOrder = 20 },
```

### Skip auto-close behavior

Currently, opening any overlay closes the previously active one. To allow multiple overlays at once, remove the `closeActiveOverlay()` call inside `toggleOverlay`.

---

## ⚠️ Considerations

- Only **one overlay can be active at a time**. The state is tracked in `activeOverlay`.
- Buttons with **both** `ToMain` and `UsesCamera` tags are **ignored** by this controller (handled by the camera system).
- The close button must:
  - Be found via `gui:FindFirstChild("CloseButton", true)`.
  - Be a `TextButton` or `ImageButton`.
  - Have the `ToMain` tag.
- The overlay screens use the `IsEnabled` attribute — this integrates with **Screen Fader** and **Screen Inflater** systems from `UIAnimationsModule`.
- `gui.DisplayOrder` is preserved via the `DisplayOrderOverride` attribute. Do not modify it manually.
- If the trigger button or the screen is missing, the entry is silently skipped.
- The script relies on `MainGui.Hotbar.Buttons` by default. If your hierarchy differs, use `containerPath`.

---

## 📋 Quick reference

| Concept | Name | Required |
|---------|------|----------|
| Main GUI | `MainGui` | ✅ |
| Default button container | `MainGui.Hotbar.Buttons` | ✅ (unless using `containerPath`) |
| Overlay screens | Any `ScreenGui` in `overlayList` | ✅ |
| Close button | `CloseButton` inside the overlay | ❌ (optional) |
| Close button tag | `ToMain` | ✅ (for close to work) |
| Skipped button tag | `UsesCamera` | ❌ |
| State attribute | `IsEnabled` | ✅ (on the `ScreenGui`) |
| Display order backup | `DisplayOrderOverride` | Auto-managed |

---

## 🧪 Usage example

1. Create a `ScreenGui` named **`Backpack`** in `StarterGui` with the attribute `IsEnabled = false`.
2. Inside, add a `CloseButton` (`TextButton`) and tag it with `ToMain`:

```lua
closeButton:AddTag("ToMain")
```

3. In `MainGui.Hotbar.Buttons`, create a button named **`Inventory`**.
4. Run the game. Clicking **Inventory** will open the `Backpack` overlay.
5. Clicking the close button (or clicking **Inventory** again) will close it.
6. Clicking a different overlay button will close the current one and open the new one.

> ✅ The `IsEnabled` attribute integrates with the Screen Fader / Inflater systems, so the overlay opens and closes with its configured animations.
