# UI Animations Module

Modular animation and UI controller system for Roblox. Uses `CollectionService` for tagging instances and `Attributes` for configuring behaviors without additional scripts.

> ℹ️ All systems initialize automatically at the end of the module. You only need to tag and configure attributes on your instances.

---

## 📦 Installation

Place the module in the following structure:

```
ReplicatedStorage
└── UI
    └── UIAnimationsModule
        ├── Animator
        └── Config
```

Require it from a `LocalScript`:

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UIAnimationsModule = ReplicatedStorage:WaitForChild("UI"):WaitForChild("UIAnimationsModule")
local Animator = require(UIAnimationsModule:WaitForChild("Animator"))
local Config   = require(UIAnimationsModule:WaitForChild("Config"))
```

---

## 🎬 Screen Fader

Animates the **position** of GUI elements when a screen opens or closes.

### Tag

```
Fadeable
```

| Applies to | Does NOT apply to |
|------------|-------------------|
| `GuiObject` (`Frame`, `TextLabel`, `ImageLabel`, etc.) | `ScreenGui`, `BillboardGui`, `SurfaceGui` |

### Required Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `DFade` | `UDim2` | **Destination** position (visible / open) |
| `OFade` | `UDim2` | **Origin** position (hidden / closed) |

### Optional Attributes

| Attribute | Type | Default | Description |
|-----------|------|---------|-------------|
| `FDDuration` | `string` | `"Slow"` | Fade **in** duration |
| `FODuration` | `string` | `"Slow"` | Fade **out** duration |

> ℹ️ Accepted duration values: `"Fast"`, `"Normal"`, `"Slow"`.

### Screen Control

The parent `ScreenGui` must have:

| Attribute | Type | Description |
|-----------|------|-------------|
| `IsEnabled` | `boolean` | `true` = open, `false` = close |

### Example

```lua
local frame = script.Parent.Frame

frame:SetAttribute("DFade", UDim2.new(0.5, 0, 0.5, 0))
frame:SetAttribute("OFade", UDim2.new(0.5, -0.5, 0.5, 0))
frame:SetAttribute("FDDuration", "Normal")
frame:SetAttribute("FODuration", "Fast")
frame:AddTag("Fadeable")

screenGui:SetAttribute("IsEnabled", true)
```

---

## 🎈 Screen Inflater

Animates the **size** of GUI elements with an inflate/deflate effect.

### Tag

```
Inflate
```

### Required Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `DInflate` | `UDim2` | **Destination** size (open) |
| `OInflate` | `UDim2` | **Origin** size (closed) |

### Example

```lua
local frame = script.Parent.Frame

frame:SetAttribute("DInflate", UDim2.new(0, 200, 0, 100))
frame:SetAttribute("OInflate", UDim2.new(0, 0, 0, 0))
frame:AddTag("Inflate")

screenGui:SetAttribute("IsEnabled", true)
```

---

## 🖼️ Load Player Thumbnail

Loads the player's headshot thumbnail into an `ImageLabel`.

### Tag

```
LoadPlayer
```

### Required Attribute

| Attribute | Type | Description |
|-----------|------|-------------|
| `PlayerID` | `string` or `number` | UserId. Accepts `"Self"` for the local player |

### Visual Behavior

| State | Image | BackgroundColor3 | UIGradient |
|-------|-------|------------------|------------|
| No valid ID | `""` | White | Enabled |
| Valid ID | Thumbnail | Black | Disabled |

### Example

```lua
local imageLabel = script.Parent.ImageLabel

imageLabel:SetAttribute("PlayerID", "Self")
imageLabel:AddTag("LoadPlayer")
```

---

## 💥 Bloat

Scale effect on button hover/click.

### Tags

| Tag | Applies to | Description |
|-----|------------|-------------|
| `Bloat` | `TextButton` / `ImageButton` | Scales only the button |
| `BloatImg` | `TextButton` / `ImageButton` | Scales the button + moves its child `ImageLabel` |

### Behavior

| Event | Scale | Duration |
|-------|-------|----------|
| Hover | `base * 1.2` | 0.18s |
| Click down | `base * 0.8` | 0.08s |
| Release / Leave | `base` | 0.18s |

### Example

```lua
button:AddTag("Bloat")
-- or
button:AddTag("BloatImg")
```

---

## 🧭 Scroll Route

Tab-style navigation system: shows/hides containers on click.

### Main Attribute

| Attribute | Type | Description |
|-----------|------|-------------|
| `ScrollRoute` | `string` or `Instance` | Path to the target container |

### Accepted path formats

```
"Player.PlayerGui.MainGui.Route1"
"PlayerGui.MainGui.Route1"
"game.Players.LocalPlayer.PlayerGui.MainGui.Route1"
"MainGui.Route1"
```

### Style Tags

| Tag | Effect |
|-----|--------|
| `GradiantTransition` | Toggles child `UIGradient` `Selected` / `Unselected` |
| `ImgTransition` | Toggles `UIShadow` and adjusts `ImageTransparency` |
| `AppearAnim` | Target container children appear with 0.05s stagger |

### Full Example

```lua
local tabButton = script.Parent

tabButton:SetAttribute("ScrollRoute", "MainGui.Page1")
tabButton:AddTag("GradiantTransition")
tabButton:AddTag("AppearAnim")

local hideFolder = Instance.new("Folder")
hideFolder.Name = "ScrollHide"
hideFolder.Parent = tabButton
```

---

## 🎥 Uses Camera

Cinematic camera animation.

### Tags

| Tag | Applies to | Description |
|-----|------------|-------------|
| `UsesCamera` | `GuiButton` | Triggers the animation |
| `ToMain` | `GuiButton` | Returns the camera and re-enables `MainGui` |

### Required Child

Inside the button, a `StringValue` named **`CameraRoute`**:

| Value | Example |
|-------|---------|
| String path | `"workspace.CameraPoints.ShopView"` |

### Required Instances

| Name | Location | Type |
|------|----------|------|
| `UIBackground` | `PlayerGui` | `ScreenGui` |
| `BlackBg` | `UIBackground` | `Frame` |
| `MainGui` | `PlayerGui` | `ScreenGui` |
| `CameraPoints` | `workspace` | `Folder` |

### Example

```lua
button:AddTag("UsesCamera")

local route = Instance.new("StringValue")
route.Name = "CameraRoute"
route.Value = "workspace.CameraPoints.ShopView"
route.Parent = button

backButton:AddTag("UsesCamera")
backButton:AddTag("ToMain")
```

---

## 🌫️ Intro Blur

Blur effect on experience startup.

> ✅ No tags or attributes required. Runs automatically.

**Behavior:**

- Creates `IntroBlur` (`BlurEffect`) in `Lighting` if it doesn't exist
- Animates `Size` from `100 → 0` in 2.25s
- Disables it when finished

---

## 📊 Leaderstats Controller

Controls `MainGui.Leaderstats` through direct name references.

### Required Paths

| Element | Path |
|---------|------|
| `LeaderstatsButton` | `MainGui.LeaderstatsButton` |
| `Leaderstats` | `MainGui.Leaderstats` |
| `CloseButton` | `MainGui.CloseButton` |
| `PlayerTemplate` | `MainGui.PlayerTemplate` |
| `PlayerProfile` | `MainGui.PlayerProfile` |
| `ProfileCloseButton` | `MainGui.ProfileCloseButton` |
| `MatchesContainer` | `MainGui.MatchesContainer` |

### Attributes Used

| Attribute | Where | Type | Description |
|-----------|-------|------|-------------|
| `UIVisible` | `PlayerProfile` | `boolean` | Profile visibility |
| `UIOriginalSize` | `PlayerProfile` | `UDim2` | Saved original size |
| `UISlideOrigSize` | `Guide` | `UDim2` | Guide original size |

---

## 📋 Quick Reference

### Tags

| Tag | Instance type | Requires attribute | Effect |
|-----|---------------|--------------------|--------|
| `Fadeable` | `GuiObject` | `DFade`, `OFade` | Animates position |
| `Inflate` | `GuiObject` | `DInflate`, `OInflate` | Animates size |
| `LoadPlayer` | `ImageLabel` | `PlayerID` | Loads thumbnail |
| `Bloat` | `Button` | — | Scales on hover/click |
| `BloatImg` | `Button` | — | Scales + moves ImageLabel |
| `GradiantTransition` | `Button` | `ScrollRoute` | Changes gradients |
| `ImgTransition` | `Button` | `ScrollRoute` | Changes shadow/transparency |
| `AppearAnim` | `Button` | `ScrollRoute` | Appear stagger |
| `UsesCamera` | `Button` | `CameraRoute` (child) | Camera animation |
| `ToMain` | `Button` | `UsesCamera` | Returns camera to origin |

### Attributes

| Attribute | Type | Used in |
|-----------|------|---------|
| `DFade` / `OFade` | `UDim2` | ScreenFader |
| `FDDuration` / `FODuration` | `string` | ScreenFader |
| `DInflate` / `OInflate` | `UDim2` | Inflater |
| `IsEnabled` | `boolean` | ScreenFader / Inflater |
| `PlayerID` | `string` / `number` | LoadPlayer |
| `ScrollRoute` | `string` / `Instance` | ScrollRoute |
| `CameraRoute` | `string` | UsesCamera |
| `UIVisible` / `UIOriginalSize` | `boolean` / `UDim2` | Leaderstats |
| `UISlideOrigSize` | `UDim2` | Leaderstats |
