local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local battlePass = player.PlayerGui:WaitForChild("BattlePass")
local battlePassFrame = battlePass:WaitForChild("Container"):WaitForChild("BattlePassFrame")
local preview = battlePassFrame:WaitForChild("Preview")
local pageList = battlePassFrame:WaitForChild("PageList")

local freeGamepass = preview:WaitForChild("FreeGamepass")
local premiumGamepass = preview:WaitForChild("PremiumGamepass")

local nextButton = battlePassFrame:WaitForChild("Next")
local backButton = battlePassFrame:WaitForChild("Back")

local MODULES = {
	"UIGradient",
	"UIStroke",
	"UIShadow",
}

local PAGE_STEP = 745
local MAX_PAGES = 6

local CANVAS_TWEEN_INFO = TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

local syncing = false
local currentPage = nil
local tweening = false

local function removeEffects(button)
	for _, className in ipairs(MODULES) do
		local existing = button:FindFirstChildOfClass(className)
		if existing then
			existing:Destroy()
		end
	end
end

local function applyEffects(button, sourceFolderName)
	local source = button:FindFirstChild(sourceFolderName)
	if not source then return end

	removeEffects(button)

	for _, className in ipairs(MODULES) do
		local template = source:FindFirstChildOfClass(className)
		if template then
			local clone = template:Clone()
			clone.Parent = button
		end
	end
end

local function getPageIndexFromCanvas()
	local x = freeGamepass.CanvasPosition.X
	local pageIndex = math.floor(x / PAGE_STEP) + 1
	pageIndex = math.clamp(pageIndex, 1, MAX_PAGES)
	return pageIndex
end

local function updatePage()
	local pageIndex = getPageIndexFromCanvas()
	local targetName = "Page" .. pageIndex
	if targetName == currentPage then return end
	currentPage = targetName

	for _, button in ipairs(pageList:GetChildren()) do
		if button:IsA("TextButton") or button:IsA("ImageButton") then
			if button.Name == targetName then
				applyEffects(button, "Selected")
			else
				applyEffects(button, "Unselected")
			end
		end
	end
end

local function setCanvasImmediate(position)
	syncing = true
	freeGamepass.CanvasPosition = position
	premiumGamepass.CanvasPosition = position
	syncing = false
end

local function tweenCanvas(targetX)
	if tweening then return end
	tweening = true

	local startX = freeGamepass.CanvasPosition.X
	local targetPosition = Vector2.new(targetX, freeGamepass.CanvasPosition.Y)

	local tween = TweenService:Create(
		freeGamepass,
		CANVAS_TWEEN_INFO,
		{ CanvasPosition = targetPosition }
	)

	tween.Completed:Connect(function()
		tweening = false
	end)

	tween:Play()
end

freeGamepass:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
	if syncing then
		updatePage()
		return
	end
	syncing = true
	premiumGamepass.CanvasPosition = freeGamepass.CanvasPosition
	syncing = false
	updatePage()
end)

premiumGamepass:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
	if syncing then
		updatePage()
		return
	end
	syncing = true
	freeGamepass.CanvasPosition = premiumGamepass.CanvasPosition
	syncing = false
	updatePage()
end)

nextButton.MouseButton1Click:Connect(function()
	local currentIndex = getPageIndexFromCanvas()
	if currentIndex >= MAX_PAGES then return end
	local targetX = currentIndex * PAGE_STEP
	tweenCanvas(targetX)
end)

backButton.MouseButton1Click:Connect(function()
	local currentIndex = getPageIndexFromCanvas()
	if currentIndex <= 1 then return end
	local targetX = (currentIndex - 2) * PAGE_STEP
	tweenCanvas(targetX)
end)

updatePage()
