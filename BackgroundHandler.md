BackgroundHandler

`LocalScript` that powers the animated background for the `UIBackground` screen. It combines a fading `Bg` frame, a global blur effect, a cycling gradient, and **3 kinematic balls** that bounce off each other and the screen edges. It also coordinates the visibility of `MainGui` with any screen tagged as `UsesBg`.

> ℹ️ The system is fully automatic. It starts when `UIBackground.Enabled = true` and stops when it becomes `false`.

---

## 📦 Requirements

| Path | Type | Description |
|------|------|-------------|
| `PlayerGui.UIBackground` | `ScreenGui` | Container screen for the background |
| `PlayerGui.UIBackground.Bg` | `Frame` (or `GuiObject`) | Background element that fades, gradients and holds the balls |
| `PlayerGui.UIBackground.Bg.Ball1` | `GuiObject` | First bouncing ball |
| `PlayerGui.UIBackground.Bg.Ball2` | `GuiObject` | Second bouncing ball |
| `PlayerGui.UIBackground.Bg.Ball3` | `GuiObject` | Third bouncing ball |
| `PlayerGui.UIBackground.Bg.UIGradient` | `UIGradient` | Gradient that cycles its offset |
| `PlayerGui.MainGui` | `ScreenGui` | Main GUI, auto-disabled when any `UsesBg` screen is enabled |
| `Lighting.UIBackgroundBlur` | `BlurEffect` | Auto-created if it doesn't exist |

### Expected structure

```
PlayerGui
├── UIBackground (ScreenGui)
│   └── Bg (Frame)
│       ├── UIGradient
│       ├── Ball1
│       ├── Ball2
│       └── Ball3
└── MainGui (ScreenGui)

Lighting
└── UIBackgroundBlur (BlurEffect) ◄── Auto-created
```

---

## 🎯 What this script does

The script performs **six main functions**:

### 1. Background fade in/out

When the background is active, `Bg.BackgroundTransparency` tweens to `0.1`. When inactive, it tweens back to `1`.

| Constant | Value | Description |
|----------|-------|-------------|
| `BG_FADE_IN_TIME` | `0.25` | Fade-in duration |
| `BG_FADE_OUT_TIME` | `0.20` | Fade-out duration |
| `BG_FADE_IN_TARGET` | `0.1` | Fade-in transparency |
| `BG_FADE_OUT_TARGET` | `1` | Fade-out transparency |

### 2. Blur effect

Creates (if missing) a `BlurEffect` in `Lighting` named `UIBackgroundBlur` and animates its `Size`:

| Constant | Value | Description |
|----------|-------|-------------|
| `BLUR_NAME` | `"UIBackgroundBlur"` | Blur instance name |
| `BLUR_IN_TIME` | `0.75` | Fade-in duration |
| `BLUR_OUT_TIME` | `0.50` | Fade-out duration |
| `BLUR_MAX` | `100` | Maximum blur size |

### 3. Gradient cycling

The `UIGradient` inside `Bg` continuously cycles its `Offset` in a loop, creating a sweeping animation.

| Constant | Value | Description |
|----------|-------|-------------|
| `GRADIENT_CYCLE` | `45` | Duration of each sweep (seconds) |
| `GRADIENT_ANGLE` | `45` | Fixed rotation of the gradient |

**Loop sequence:**

```
Offset (-1, -1) → (1, 1) → (1, -1) → (-1, 1) → repeat
```

### 4. Kinematic bouncing balls

Three balls (`Ball1`, `Ball2`, `Ball3`) move inside `Bg` using **normalized coordinates** (0–1 range) and:

- Bounce off the edges of the background.
- Collide with each other and reflect their velocity.
- Update every `RunService.Heartbeat`.

| Constant | Value | Description |
|----------|-------|-------------|
| `BALL_SPEED` | `0.08` | Global speed multiplier |
| `BALLS` | `{ "Ball1", "Ball2", "Ball3" }` | Names of the balls |

**Initial state per ball:**

- Random position within bounds.
- Random direction with an angle offset of ±45°.

**Collision handling:**

- Edge collision: clamps position and reflects velocity component.
- Ball-ball collision: resolves the smallest overlap axis (X or Y) and reflects velocities.

### 5. `UsesBg` tag coordination

Any `ScreenGui` tagged with `UsesBg` acts as a "focus" screen:

- While **any** `UsesBg` screen is `Enabled = true`:
  - `MainGui:SetAttribute("IsEnabled", false)` → `MainGui` is hidden.
  - `UIBackground.Enabled = true` → background becomes visible.
- When **no** `UsesBg` screen is enabled:
  - `MainGui:SetAttribute("IsEnabled", true)` → `MainGui` is shown.
  - `UIBackground.Enabled = false` → background is hidden.

| Constant | Value |
|----------|-------|
| `USES_BG_TAG` | `"UsesBg"` |
| `MAIN_GUI` | `"MainGui"` |

### 6. Automatic start/stop

The script listens to `UIBackground.Enabled` and starts/stops everything accordingly:

- `Enabled = true` → `startAll()`
- `Enabled = false` → `stopAll()`

---

## ⚙️ Configurable constants

| Constant | Default value | Description |
|----------|---------------|-------------|
| `SCREEN_NAME` | `"UIBackground"` | Background screen name |
| `BG_NAME` | `"Bg"` | Background frame name |
| `BALLS` | `{ "Ball1", "Ball2", "Ball3" }` | Ball instance names |
| `BALL_SPEED` | `0.08` | Global ball speed |
| `GRADIENT_CYCLE` | `45` | Gradient sweep duration |
| `GRADIENT_ANGLE` | `45` | Gradient rotation |
| `BLUR_NAME` | `"UIBackgroundBlur"` | Blur instance name |
| `BLUR_IN_TIME` | `0.75` | Blur fade-in |
| `BLUR_OUT_TIME` | `0.50` | Blur fade-out |
| `BLUR_MAX` | `100` | Maximum blur size |
| `BG_FADE_IN_TIME` | `0.25` | Bg fade-in |
| `BG_FADE_OUT_TIME` | `0.20` | Bg fade-out |
| `BG_FADE_IN_TARGET` | `0.1` | Bg target transparency (in) |
| `BG_FADE_OUT_TARGET` | `1` | Bg target transparency (out) |
| `USES_BG_TAG` | `"UsesBg"` | Tag for focus screens |
| `MAIN_GUI` | `"MainGui"` | Main GUI name |

---

## 🔄 Execution flow

```
[Script starts]
   │
   ├─► Waits for UIBackground and its Bg child
   │
   ├─► Sets Bg.BackgroundTransparency = 1
   │
   ├─► If UIBackground.Enabled → startAll()
   │
   ├─► Listens to Enabled changes → startAll() / stopAll()
   │
   └─► Starts UsesBg watcher:
         ├─► Binds existing ScreenGuis with UsesBg tag
         ├─► Listens for new tagged instances
         ├─► Listens for tag removal
         └─► refreshUsesBgState():
               ├─► If any UsesBg screen enabled:
               │     • MainGui.IsEnabled = false
               │     • UIBackground.Enabled = true
               └─► Else:
                     • MainGui.IsEnabled = true
                     • UIBackground.Enabled = false
```

---

## 🧩 Main functions

### `bgFadeIn()` / `bgFadeOut()`

Tweens `Bg.BackgroundTransparency` to the configured targets.

### `blurIn()` / `blurOut()`

Animates the blur effect. `blurOut()` automatically disables the blur when it reaches `0`.

### `startGradient()` / `stopGradient()`

Starts or stops the gradient offset cycling loop.

### `startBalls()` / `stopBalls()`

Initializes ball data (position, velocity, size) and connects/disconnects the `Heartbeat` step.

### `ballStep(dt)`

Advances all balls by `velocity * BALL_SPEED * dt`, resolves collisions and applies the final positions.

### `ballResolveCollisions()`

Detects overlapping pairs and reflects their velocities along the smallest overlap axis.

### `ballBounceBounds(d, bw, bh)`

Clamps ball position to the bounds of `Bg` and reflects the velocity when hitting an edge.

### `anyUsesBgScreenEnabled()`

Returns `true` if any `ScreenGui` tagged with `UsesBg` is currently enabled.

### `refreshUsesBgState()`

Synchronizes `MainGui.IsEnabled` and `UIBackground.Enabled` based on the `UsesBg` state.

### `startAll()` / `stopAll()`

Full state transitions: fades, blur, gradient and balls.

---

## 🎨 Customization

### Add a new ball

```lua
local BALLS = { "Ball1", "Ball2", "Ball3", "Ball4" }
```

Then add a `Ball4` instance inside `Bg`.

### Change the ball speed

```lua
local BALL_SPEED = 0.15  -- faster
```

### Change the gradient cycle duration

```lua
local GRADIENT_CYCLE = 20  -- faster sweep
```

### Change the blur strength

```lua
local BLUR_MAX = 50
```

### Rename the background screen

```lua
local SCREEN_NAME = "MyBackground"
```

### Add a new `UsesBg` screen

Tag any `ScreenGui` with `UsesBg`:

```lua
someScreenGui:AddTag("UsesBg")
```

The script will automatically disable `MainGui` and enable `UIBackground` while it's visible.

---

## ⚠️ Considerations

- The script uses **normalized coordinates** (0–1) to position the balls. That means the balls will always stay inside `Bg` regardless of screen resolution.
- `Bg.BackgroundTransparency` is forced to `1` at startup. If you want an initial visible background, remove that line.
- The blur effect is **shared** across the experience (`Lighting.UIBackgroundBlur`). Other scripts should not create a second one with the same name.
- The gradient loop uses `task.spawn` + `task.cancel`. Do not modify it while running.
- `MainGui` is controlled **only** through the `IsEnabled` attribute. Other systems (like Screen Fader) must respect that attribute.
- The `UsesBg` watcher scans existing `ScreenGui`s on start and reacts to new ones added later.
- If `UIBackground` doesn't exist within 30 seconds, the script silently exits.

---

## 📋 Quick reference

| Concept | Name | Required |
|---------|------|----------|
| Background screen | `UIBackground` | ✅ |
| Background frame | `Bg` | ✅ |
| Balls | `Ball1`, `Ball2`, `Ball3` | ✅ |
| Gradient | `UIGradient` inside `Bg` | ❌ (animation skipped) |
| Blur effect | `UIBackgroundBlur` in `Lighting` | ❌ (auto-created) |
| Main GUI | `MainGui` | ❌ (checked for attribute updates) |
| Focus tag | `UsesBg` | ❌ (required for focus-screen coordination) |

---

## 🧪 Usage example

1. In `StarterGui`, create a `ScreenGui` named **`UIBackground`**.
2. Inside it, add a `Frame` named **`Bg`** with:
   - A `UIGradient` child.
   - `Ball1`, `Ball2` and `Ball3` as children (`Frame` or `ImageLabel`).
3. Set `UIBackground.Enabled = true` when you want the background active.
4. (Optional) Tag any other `ScreenGui` with `UsesBg` to automatically hide `MainGui` while it's visible.
5. Run the game. The background will fade in, blur will appear, gradient will cycle and the balls will bounce around.

> ✅ Everything stops cleanly when `UIBackground.Enabled = false` — tweens are cancelled, the `Heartbeat` connection is disconnected, and the blur is disabled.
