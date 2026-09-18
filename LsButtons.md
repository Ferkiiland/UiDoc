# ClientPlayerProfileController

`LocalScript` that manages the **player profile panel** inside `Leaderstats`. Handles hover effects on action buttons (AddFriend, Block, Compare, Spectate, ViewProfile), opens the profile screen, and connects the corresponding player actions.

> ℹ️ Uses the `IsEnabled` attribute on `ScreenGui`s to open/close panels, integrating with the Screen Fader / Inflater systems.

---

## 📦 Requirements

| Path | Type | Description |
|------|------|-------------|
| `ReplicatedStorage.UI.UIAnimationsModule` | `ModuleScript` | Required module |
| `ReplicatedStorage.UI.UIAnimationsModule.Animator` | `ModuleScript` | Animation helpers |
| `PlayerGui.MainGui` | `ScreenGui` | Main GUI |
| `MainGui.Leaderstats` | `Frame` / `CanvasGroup` | Leaderstats panel |
| `MainGui.Leaderstats.PlayerProfile` | `Frame` | Profile panel with action buttons |
| `MainGui.LeaderstatsButton` | `Button` | Button that opens leaderstats |
| `MainGui.MatchesContainer` | `Frame` | Container that moves when leaderstats opens |
| `PlayerGui.PlayerProfile` | `ScreenGui` | Full profile screen |
| `PlayerGui.PlayerProfile.Container` | `Frame` | Container with `PlayerFrame` and `CompareFrame` |
| `PlayerGui.PlayerProfile.Container.PlayerFrame` | `Frame` | Player info panel (with `CloseButton`) |
| `PlayerGui.PlayerProfile.Container.CompareFrame` | `Frame` | Comparison panel (with `CloseButton`) |

### Expected structure

```
ReplicatedStorage
└── UI
    └── UIAnimationsModule
        └── Animator

PlayerGui
├── MainGui (ScreenGui)
│   ├── LeaderstatsButton
│   ├── MatchesContainer
│   └── Leaderstats
│       └── PlayerProfile
│           ├── AddFriend
│           ├── Block
│           ├── Compare
│           ├── Spectate
│           └── ViewProfile
│
└── PlayerProfile (ScreenGui)
    └── Container
        ├── PlayerFrame
        │   └── CloseButton
        └── CompareFrame
            └── CloseButton
```

---

## 🎯 What this script does

The script performs **seven main functions**:

### 1. Controls the leaderstats panel

Opens/closes `Leaderstats` when `LeaderstatsButton` is clicked, and moves `MatchesContainer` accordingly.

| Constant | Value | Description |
|----------|-------|-------------|
| `MATCHES_DEFAULT_POS` | `UDim2.new(0.99, 0, 0.076, 0)` | Position when leaderstats is closed |
| `MATCHES_LOWERED_POS` | `UDim2.new(0.989, 0, 0.588, 0)` | Position when leaderstats is open |

A `leaderstatsBusy` flag prevents overlapping tweens. After 0.35s the flag resets.

```lua
local function doOpenLeaderstats()
	if leaderstatsBusy then return end
	leaderstatsBusy = true

	Animator.setVisible(leaderstats, true, 0.3)
	Animator.setVisibleFade(leaderstatsBtn, false, 0.3)
	tweenMatches(MATCHES_LOWERED_POS)

	task.delay(0.35, function()
		leaderstatsBusy = false
	end)
end
```

### 2. Sets up hover effects on profile buttons

For each button in `BUTTON_NAMES`, a hover effect scales the button to **80%** on `MouseEnter` and back to its original size on `MouseLeave`.

| Constant | Value |
|----------|-------|
| `HOVER_TWEEN` | `TweenInfo.new(0.15, Quad, Out)` |
| Hover scale | `originalSize * 0.8` |

The original size is saved in `originalSizes[btn]`.

```lua
local BUTTON_NAMES = {
	"AddFriend",
	"Block",
	"Compare",
	"Spectate",
	"ViewProfile",
}
```

### 3. Click cooldown

Each button has a **2-second cooldown** to avoid spam:

```lua
local clickCooldown = 2
local lastClick = {}

local function canClick(key)
	local now = os.clock()
	if lastClick[key] and now - lastClick[key] < clickCooldown then
		return false
	end
	lastClick[key] = now
	return true
end
```

### 4. AddFriend

Sends a friendship request to the target player.

```lua
addFriendBtn.Activated:Connect(function()
	if not canClick("AddFriend") then return end
	local target = getTargetPlayer()
	if not target then
		warn("[AddFriend] No player objective.")
		return
	end
	local success, err = pcall(function()
		lp:RequestFriendship(target)
	end)
	if success then
		print("[AddFriend] Friendship request sent to:", target.Name)
	else
		warn("[AddFriend] Error sending friendship request:", err)
	end
end)
```

### 5. Block

Blocks the target player.

```lua
blockBtn.Activated:Connect(function()
	if not canClick("Block") then return end
	local target = getTargetPlayer()
	if not target then
		warn("[Block] No player objective.")
		return
	end
	local success, err = pcall(function()
		lp:BlockUser(target)
	end)
	if success then
		print("[Block] User blocked:", target.Name)
	else
		warn("[Block] Error blocking user:", err)
	end
end)
```

### 6. ViewProfile

Opens the profile screen, hides `MainGui` and shows `PlayerFrame`.

```lua
viewProfileBtn.Activated:Connect(function()
	if not canClick("ViewProfile") then return end

	resetAllButtons()

	playerProfileScreen:SetAttribute("IsEnabled", true)
	mainGui:SetAttribute("IsEnabled", false)

	playerFrame.Visible = true
	compareFrame.Visible = false
end)
```

### 7. Compare

Opens the profile screen, hides `MainGui` and shows `CompareFrame`.

```lua
compareBtn.Activated:Connect(function()
	if not canClick("Compare") then return end

	resetAllButtons()

	playerProfileScreen:SetAttribute("IsEnabled", true)
	mainGui:SetAttribute("IsEnabled", false)

	compareFrame.Visible = true
	playerFrame.Visible = false
end)
```

### 8. Spectate

Placeholder — currently does nothing (empty body inside the connection).

### 9. Close buttons

Both `PlayerFrame.CloseButton` and `CompareFrame.CloseButton` close the profile screen, restore `MainGui` and force-close leaderstats.

```lua
local function onProfileClose()
	playerProfileScreen:SetAttribute("IsEnabled", false)
	mainGui:SetAttribute("IsEnabled", true)

	forceCloseLeaderstats()
end

playerFrameClose.Activated:Connect(onProfileClose)
compareFrameClose.Activated:Connect(onProfileClose)
```

### 10. Resets buttons automatically

`resetAllButtons()` is triggered when:

- `PlayerProfile.Visible` becomes `false`.
- `Leaderstats.Visible` becomes `false`.
- `MainGui.IsEnabled` becomes `false`.

```lua
playerProfile:GetPropertyChangedSignal("Visible"):Connect(function()
	if not playerProfile.Visible then
		resetAllButtons()
	end
end)

leaderstats:GetPropertyChangedSignal("Visible"):Connect(function()
	if not leaderstats.Visible then
		resetAllButtons()
	end
end)

mainGui:GetAttributeChangedSignal("IsEnabled"):Connect(function()
	if mainGui:GetAttribute("IsEnabled") == false then
		resetAllButtons()
	end
end)
```

### 11. Dynamic button binding

If new buttons matching `BUTTON_NAMES` are added to `PlayerProfile` later, they are automatically set up.

```lua
playerProfile.DescendantAdded:Connect(function(descendant)
	if table.find(BUTTON_NAMES, descendant.Name) and descendant:IsA("GuiObject") then
		local reset = setupButton(descendant)
		if reset then
			table.insert(resetFunctions, reset)
		end
	end
end)
```

---

## ⚙️ Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `MATCHES_DEFAULT_POS` | `UDim2.new(0.99, 0, 0.076, 0)` | Default position of `MatchesContainer` |
| `MATCHES_LOWERED_POS` | `UDim2.new(0.989, 0, 0.588, 0)` | Lowered position when leaderstats is open |
| `BUTTON_NAMES` | `AddFriend, Block, Compare, Spectate, ViewProfile` | Supported action buttons |
| `HOVER_TWEEN` | `TweenInfo.new(0.15, Quad, Out)` | Hover animation |
| `clickCooldown` | `2` | Seconds between clicks per button |
| Hover scale | `0.8` | Scale applied on hover |

---

## 🔄 Execution flow

```
[Script starts]
   │
   ├─► Requires Animator
   │
   ├─► Waits for MainGui, Leaderstats, PlayerProfile, PlayerProfile screen
   │
   ├─► Sets up leaderstats open/close logic
   │
   ├─► Registers hover effects on all profile buttons
   │
   ├─► Binds logic to each button:
   │     • AddFriend  → RequestFriendship
   │     • Block      → BlockUser
   │     • ViewProfile→ Open PlayerProfile screen (PlayerFrame)
   │     • Compare    → Open PlayerProfile screen (CompareFrame)
   │     • Spectate   → (placeholder)
   │
   ├─► Binds close buttons on PlayerFrame / CompareFrame
   │
   ├─► Listens for visibility changes → resetAllButtons()
   │
   └─► Listens for DescendantAdded → auto-setup new buttons
```

---

## 🧩 Main functions

### `cancelMatchesTween()` / `tweenMatches(targetPos)`

Cancel and start the `MatchesContainer` position tween.

### `isLeaderstatsVisible()`

Returns `true` if `Leaderstats` is visible and has `UIVisible = true`.

### `doOpenLeaderstats()` / `doCloseLeaderstats()`

Show/hide `Leaderstats` with animations and move `MatchesContainer`.

### `forceCloseLeaderstats()`

Wraps `doCloseLeaderstats()` for external calls.

### `getButton(name)`

Returns `PlayerProfile:FindFirstChild(name)`.

### `canClick(key)`

Cooldown gate per button (2 seconds).

### `getTargetPlayer()`

Reads `PlayerProfile:GetAttribute("TargetUserId")` and returns the corresponding `Player`.

### `setupButton(btn)`

Registers hover/leave tweens and stores the original size. Returns a reset function.

### `resetAllButtons()`

Calls every reset function to restore button sizes.

### `onProfileClose()`

Closes the profile screen, restores `MainGui` and force-closes leaderstats.

---

## 🎨 Customization

### Change the hover scale

```lua
tweenTo(UDim2.new(
	originalSize.X.Scale * 0.9, originalSize.X.Offset * 0.9,
	originalSize.Y.Scale * 0.9, originalSize.Y.Offset * 0.9
))
```

### Change the click cooldown

```lua
local clickCooldown = 0.5  -- faster
```

### Add a new action button

1. Create the button inside `PlayerProfile` with the desired name.
2. Add the name to `BUTTON_NAMES`:

```lua
local BUTTON_NAMES = {
	"AddFriend",
	"Block",
	"Compare",
	"Spectate",
	"ViewProfile",
	"Report",  -- new
}
```

3. Add the logic:

```lua
local reportBtn = getButton("Report")
if reportBtn then
	reportBtn.Activated:Connect(function()
		if not canClick("Report") then return end
		local target = getTargetPlayer()
		if not target then return end
		-- your logic
	end)
end
```

### Change the tween timing for leaderstats

```lua
Animator.setVisible(leaderstats, true, 0.5)  -- slower open
```

### Change `MatchesContainer` positions

```lua
local MATCHES_DEFAULT_POS = UDim2.new(1, 0, 0.1, 0)
local MATCHES_LOWERED_POS = UDim2.new(1, 0, 0.6, 0)
```

---

## ⚠️ Considerations

- The target player is resolved via `PlayerProfile:GetAttribute("TargetUserId")`. If that attribute is missing, `getTargetPlayer()` returns `nil`.
- `AddFriend` and `Block` use `Player:RequestFriendship` and `Player:BlockUser`, which may throw errors. Wrapped in `pcall`.
- `Spectate` is currently a placeholder — no logic attached.
- The `leaderstatsBusy` flag prevents overlapping open/close requests (0.35s cooldown).
- Hover effects are disabled while `PlayerProfile` is not visible, or when `MainGui.IsEnabled = false`.
- New buttons added at runtime matching `BUTTON_NAMES` are auto-configured via `DescendantAdded`.
- Both close buttons (`PlayerFrame.CloseButton` and `CompareFrame.CloseButton`) use the same handler — `onProfileClose()`.
- The script relies on `IsEnabled` attributes on both `MainGui` and `PlayerProfile` (the `ScreenGui`), consistent with the rest of the UI system.

---

## 📋 Quick reference

| Concept | Name | Required |
|---------|------|----------|
| Main GUI | `MainGui` | ✅ |
| Leaderstats panel | `Leaderstats` | ✅ |
| Leaderstats toggle | `LeaderstatsButton` | ✅ |
| Matches container | `MatchesContainer` | ❌ (skip tween if absent) |
| Profile panel | `PlayerProfile` (child of `Leaderstats`) | ✅ |
| Profile screen | `PlayerProfile` (in `PlayerGui`) | ✅ |
| Profile container | `PlayerProfile.Container` | ✅ |
| Player panel | `Container.PlayerFrame` + `CloseButton` | ✅ |
| Compare panel | `Container.CompareFrame` + `CloseButton` | ✅ |
| Target player attribute | `TargetUserId` on `PlayerProfile` | ❌ (needed for actions) |
| State attribute | `IsEnabled` | ✅ |
| Visibility attribute | `UIVisible` on `Leaderstats` | ✅ |

---

## 🧪 Usage example

1. In `StarterGui`, create a `ScreenGui` named **`MainGui`** with:
   - `LeaderstatsButton`
   - `MatchesContainer`
   - `Leaderstats` (with the attribute `UIVisible = true`)
     - `PlayerProfile` (with the action buttons)
2. Create a `ScreenGui` named **`PlayerProfile`** with:
   - `Container.PlayerFrame` (with a `CloseButton`)
   - `Container.CompareFrame` (with a `CloseButton`)
3. Set `PlayerProfile:SetAttribute("TargetUserId", someUserId)` when opening it.
4. Run the game. Clicking `LeaderstatsButton` opens leaderstats and lowers `MatchesContainer`.
5. Clicking **ViewProfile** opens the full profile screen; clicking **Compare** opens the compare view.
6. Clicking the close buttons restores `MainGui` and closes leaderstats.

> ✅ Hover effects work on all supported buttons and reset automatically when panels are hidden.
