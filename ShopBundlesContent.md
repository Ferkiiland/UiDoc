local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local lp        = Players.LocalPlayer
local playerGui = lp:WaitForChild("PlayerGui")

local function tw(inst, dur, goal, style, dir)
	local t = TweenService:Create(
		inst,
		TweenInfo.new(dur, style or Enum.EasingStyle.Quad, dir or Enum.EasingDirection.Out),
		goal
	)
	t:Play()
	return t
end

task.spawn(function()
	local shopScreen  = playerGui:WaitForChild("ShopScreen")
	local shopsFrame  = shopScreen:WaitForChild("Container").ShopsFrame
	local bundleFrame = shopsFrame:WaitForChild("Bundles")

	for _, detector in ipairs(bundleFrame:GetDescendants()) do
		if detector:IsA("TextButton") or detector:IsA("ImageButton") then
			if detector.Name == "HoverDetector" then
				local parent = detector.Parent
				local character     = parent:FindFirstChild("Character")
				local icon          = parent:FindFirstChild("Icon")
				local bundleContent = parent:FindFirstChild("BundleContent")

				detector.MouseEnter:Connect(function()
					if character and character:IsA("ImageLabel") then
						tw(character, 0.25, { ImageTransparency = 0.85 })
					end
					if icon and icon:IsA("ImageLabel") then
						tw(icon, 0.25, { ImageTransparency = 0.85 })
					end
					if bundleContent and bundleContent:IsA("GuiObject") then
						bundleContent.Visible = true
					end
				end)

				detector.MouseLeave:Connect(function()
					if character and character:IsA("ImageLabel") then
						tw(character, 0.25, { ImageTransparency = 0 })
					end
					if icon and icon:IsA("ImageLabel") then
						tw(icon, 0.25, { ImageTransparency = 0 })
					end
					if bundleContent and bundleContent:IsA("GuiObject") then
						bundleContent.Visible = false
					end
				end)
			end
		end
	end
end)
