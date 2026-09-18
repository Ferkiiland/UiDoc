local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

local settingsScreen = player.PlayerGui:WaitForChild("SettingsScreen")
local settingsFrame = settingsScreen:WaitForChild("Container"):WaitForChild("SettingsFrame")

local SettingsDataModule
do
	local uiFolder = ReplicatedStorage:FindFirstChild("UI")
	if uiFolder then
		local module = uiFolder:FindFirstChild("UIAnimationsModule")
		if module then
			local data = module:FindFirstChild("SettingsData")
			if data then
				local ok, result = pcall(require, data)
				if ok then
					SettingsDataModule = result
				else
					warn("Error al requerir SettingsData:", result)
				end
			else
				warn("No se encontró SettingsData dentro de UI.UIAnimationsModule")
			end
		else
			warn("No se encontró UIAnimationsModule dentro de UI")
		end
	else
		warn("No se encontró la carpeta UI en ReplicatedStorage")
	end
end

SettingsDataModule = SettingsDataModule or {}

local DEFAULT_SCROLL = SettingsDataModule.SettingScroll or {}
local DEFAULT_LIST = SettingsDataModule.SettingList or {}
local DEFAULT_TOGGLE = SettingsDataModule.SettingTogle or {}

local HOVER_TWEEN_INFO = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local UNHOVER_TWEEN_INFO = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local TOGGLE_TWEEN_INFO = TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

local activeSelector = nil

local function clamp(value, min, max)
	if value < min then return min end
	if value > max then return max end
	return value
end

local function applyHover(button)
	if button:GetAttribute("IsHovering") then return end
	button:SetAttribute("IsHovering", true)

	local tween = TweenService:Create(
		button,
		HOVER_TWEEN_INFO,
		{ BackgroundTransparency = 0.2 }
	)
	tween:Play()
end

local function removeHover(button)
	if not button:GetAttribute("IsHovering") then return end
	button:SetAttribute("IsHovering", false)

	local tween = TweenService:Create(
		button,
		UNHOVER_TWEEN_INFO,
		{ BackgroundTransparency = 0.5 }
	)
	tween:Play()
end

local function updateToggleVisual(button, activated)
	local toggler = button:FindFirstChild("Toggler")
	if not toggler then return end

	local innerBtn = toggler:FindFirstChild("InnerBtn")
	if not innerBtn then return end

	local onLabel = toggler:FindFirstChild("On")
	local offLabel = toggler:FindFirstChild("Off")

	local targetPos
	if activated then
		targetPos = UDim2.new(0.51, 0, 0.5, 0)
		if onLabel then onLabel.Enabled = true end
		if offLabel then offLabel.Enabled = false end
		innerBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	else
		targetPos = UDim2.new(0.05, 0, 0.5, 0)
		if onLabel then onLabel.Enabled = false end
		if offLabel then offLabel.Enabled = true end
		innerBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	end

	local posTween = TweenService:Create(
		innerBtn,
		TOGGLE_TWEEN_INFO,
		{ Position = targetPos }
	)
	posTween:Play()
end

local function setupToggle(button)
	if button:GetAttribute("Activated") == nil then
		local default = DEFAULT_TOGGLE[button.Name]
		if default == nil then
			default = false
		end
		button:SetAttribute("Activated", default)
	end

	updateToggleVisual(button, button:GetAttribute("Activated"))

	button:GetAttributeChangedSignal("Activated"):Connect(function()
		updateToggleVisual(button, button:GetAttribute("Activated"))
	end)

	button.MouseEnter:Connect(function()
		applyHover(button)
	end)

	button.MouseLeave:Connect(function()
		removeHover(button)
	end)

	button.MouseButton1Click:Connect(function()
		button:SetAttribute("Activated", not button:GetAttribute("Activated"))
	end)
end

local function setupList(button)
	local toggler = button:FindFirstChild("Toggler")
	local selector = button:FindFirstChild("Selector")

	if not toggler or not selector then return end

	local togglerTextLabel = toggler:FindFirstChild("TextLabel")

	if button:GetAttribute("Status") == nil then
		local default = DEFAULT_LIST[button.Name]
		if default == nil then
			default = ""
			if togglerTextLabel then
				default = togglerTextLabel.Text
			end
		end
		button:SetAttribute("Status", default)
	end

	selector.Visible = false

	local function updateTogglerText()
		if togglerTextLabel then
			togglerTextLabel.Text = button:GetAttribute("Status")
		end
	end

	updateTogglerText()

	button:GetAttributeChangedSignal("Status"):Connect(function()
		updateTogglerText()
	end)

	button.MouseEnter:Connect(function()
		applyHover(button)
	end)

	button.MouseLeave:Connect(function()
		removeHover(button)
	end)

	button.MouseButton1Click:Connect(function()
		if activeSelector and activeSelector ~= selector then
			activeSelector.Visible = false
		end

		selector.Visible = not selector.Visible
		activeSelector = selector.Visible and selector or nil
	end)

	for _, option in ipairs(selector:GetChildren()) do
		if option:IsA("TextButton") or option:IsA("ImageButton") then
			option.MouseButton1Click:Connect(function()
				local textLabel = option:FindFirstChild("TextLabel")
				if not textLabel then return end

				button:SetAttribute("Status", textLabel.Text)
				selector.Visible = false
				activeSelector = nil
			end)
		end
	end
end

local function setupScroll(button)
	local bar = button:FindFirstChild("Bar")
	local deco = bar and bar:FindFirstChild("Deco")
	local progressBar = bar and bar:FindFirstChild("ProgressBar")
	local percentage = button:FindFirstChild("Percentaje")
	local textLabel = percentage and percentage:FindFirstChild("TextLabel")
	local upDown = button:FindFirstChild("UpDown")
	local up = upDown and upDown:FindFirstChild("Up")
	local down = upDown and upDown:FindFirstChild("Down")

	if not bar or not deco or not progressBar then return end

	local currentLevel = button:GetAttribute("Level")
	if currentLevel == nil or currentLevel == "" then
		local default = DEFAULT_SCROLL[button.Name]
		if default == nil then
			default = "100"
		end
		button:SetAttribute("Level", tostring(default))
	end

	local dragging = false
	local STEP = 1

	local function applyVisual(level)
		level = clamp(level, 0, 100)
		local xPos = level / 100

		deco.Position = UDim2.new(xPos, 0, deco.Position.Y.Scale, deco.Position.Y.Offset)
		progressBar.Size = UDim2.new(xPos, 0, progressBar.Size.Y.Scale, progressBar.Size.Y.Offset)

		if textLabel then
			textLabel.Text = tostring(math.floor(level + 0.5)) .. "%"
		end
	end

	local function setLevel(level)
		level = clamp(level, 0, 100)
		local rounded = math.floor(level + 0.5)
		applyVisual(rounded)
		button:SetAttribute("Level", tostring(rounded))
	end

	applyVisual(tonumber(button:GetAttribute("Level")) or 0)

	button:GetAttributeChangedSignal("Level"):Connect(function()
		local lvl = tonumber(button:GetAttribute("Level")) or 0
		applyVisual(lvl)
	end)

	local function getXFromMouse()
		local barAbsX = bar.AbsolutePosition.X
		local barAbsWidth = bar.AbsoluteSize.X
		if barAbsWidth <= 0 then return 0 end
		return (mouse.X - barAbsX) / barAbsWidth
	end

	local function startDrag()
		dragging = true
		setLevel(getXFromMouse() * 100)
	end

	local function stopDrag()
		dragging = false
	end

	local function isPrimaryInput(input)
		return input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch
	end

	bar.InputBegan:Connect(function(input)
		if isPrimaryInput(input) then startDrag() end
	end)

	bar.InputEnded:Connect(function(input)
		if isPrimaryInput(input) then stopDrag() end
	end)

	deco.InputBegan:Connect(function(input)
		if isPrimaryInput(input) then startDrag() end
	end)

	deco.InputEnded:Connect(function(input)
		if isPrimaryInput(input) then stopDrag() end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if not dragging then return end
		if input.UserInputType == Enum.UserInputType.MouseMovement
			or input.UserInputType == Enum.UserInputType.Touch then
			setLevel(getXFromMouse() * 100)
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if isPrimaryInput(input) then stopDrag() end
	end)

	if up then
		up.MouseButton1Click:Connect(function()
			local level = tonumber(button:GetAttribute("Level")) or 0
			setLevel(level + STEP)
		end)
	end

	if down then
		down.MouseButton1Click:Connect(function()
			local level = tonumber(button:GetAttribute("Level")) or 0
			setLevel(level - STEP)
		end)
	end

	button.MouseEnter:Connect(function()
		applyHover(button)
	end)

	button.MouseLeave:Connect(function()
		removeHover(button)
	end)
end

local function setupAllButtons()
	for _, scrollingFrame in ipairs(settingsFrame:GetChildren()) do
		if scrollingFrame:IsA("ScrollingFrame") then
			for _, child in ipairs(scrollingFrame:GetDescendants()) do
				if child:IsA("TextButton") or child:IsA("ImageButton") then
					if child:HasTag("SettingToggle") then
						setupToggle(child)
					elseif child:HasTag("SettingList") then
						setupList(child)
					elseif child:HasTag("SettingScroll") then
						setupScroll(child)
					end
				end
			end
		end
	end
end

local function setupCategories()
	local categoryFrame = settingsScreen:WaitForChild("Container"):WaitForChild("Category")
	local hotKeysFrame = settingsScreen:WaitForChild("Container"):WaitForChild("HotKeys")
	local descPanel = settingsScreen:WaitForChild("Container"):WaitForChild("DescPanel")

	local function setCategorySelected(button, selected)
		local selectedFrame = button:FindFirstChild("Selected")
		if selectedFrame then
			selectedFrame.Visible = selected
		end

		local icon = button:FindFirstChild("CategoryIcon")
		if icon then
			if selected then
				icon.ImageColor3 = Color3.fromRGB(0, 0, 0)
			else
				icon.ImageColor3 = Color3.fromRGB(255, 255, 255)
			end
		end

		local textLabel = button:FindFirstChild("TextLabel")
		if textLabel then
			if selected then
				textLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
			else
				textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
			end

			local stroke = textLabel:FindFirstChildOfClass("UIStroke")
			if stroke then
				stroke.Enabled = selected
			end
		end
	end

	local function showCategory(button)
		local targetName = button.Name

		for _, other in ipairs(categoryFrame:GetChildren()) do
			if other:IsA("TextButton") or other:IsA("ImageButton") then
				setCategorySelected(other, other == button)
			end
		end

		for _, child in ipairs(settingsFrame:GetChildren()) do
			if child:IsA("ScrollingFrame") then
				child.Visible = false
			end
		end

		if targetName == "HotKeys" then
			hotKeysFrame.Visible = true
			descPanel.Visible = false
		else
			hotKeysFrame.Visible = false
			descPanel.Visible = true

			local target = settingsFrame:FindFirstChild(targetName)
			if target and target:IsA("ScrollingFrame") then
				target.Visible = true
			end
		end
	end

	for _, button in ipairs(categoryFrame:GetChildren()) do
		if button:IsA("TextButton") or button:IsA("ImageButton") then
			button.MouseButton1Click:Connect(function()
				showCategory(button)
			end)
		end
	end

	local defaultButton = categoryFrame:FindFirstChild("General")
	if defaultButton and (defaultButton:IsA("TextButton") or defaultButton:IsA("ImageButton")) then
		showCategory(defaultButton)
	end
end

setupAllButtons()
setupCategories()
