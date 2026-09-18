local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local lp         = Players.LocalPlayer
local playerGui  = lp:WaitForChild("PlayerGui")

task.spawn(function()
	local weaponsScreen = playerGui:WaitForChild("WeaponsScreen")
	local container     = weaponsScreen:WaitForChild("Container")

	------------------------------------------------------------
	-- SECCIÓN 1: Stats / Overview / MasteryList
	------------------------------------------------------------
	local statsFrame    = container:WaitForChild("StatsFrame")
	local careerCont    = container:WaitForChild("CareerContainer")

	local stats         = statsFrame:WaitForChild("Stats")
	local overview      = statsFrame:WaitForChild("Overview")
	local masteryList   = statsFrame:WaitForChild("MasteryList")
	local closeButton   = statsFrame:WaitForChild("CloseButton")

	local weaponList    = container:WaitForChild("WeaponsList")


	local mainCloseButton = container:WaitForChild("CloseButton")

	local masteryButton = careerCont:WaitForChild("3WeaponDesc")
		:WaitForChild("GunStats")
		:WaitForChild("1Mastery")

	local TWEEN_INFO = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)


	local function refreshFromChildren()
		if stats.Visible or overview.Visible or masteryList.Visible then
			statsFrame.Visible = true
			careerCont.Visible = false
		else
			statsFrame.Visible = false
			careerCont.Visible = true
		end
	end

	stats:GetPropertyChangedSignal("Visible"):Connect(refreshFromChildren)
	overview:GetPropertyChangedSignal("Visible"):Connect(refreshFromChildren)
	masteryList:GetPropertyChangedSignal("Visible"):Connect(refreshFromChildren)

	refreshFromChildren()

	closeButton.Activated:Connect(function()
		stats.Visible = false
		overview.Visible = false
		masteryList.Visible = false
	end)

	masteryButton.Activated:Connect(function()
		stats.Visible = false
		overview.Visible = false
		masteryList.Visible = true
	end)

	------------------------------------------------------------
	-- SECCIÓN 2: Loadout (1Skin, 2Wraps, 3Charms, 4Finishers)
	------------------------------------------------------------
	local setupFrames       = container:WaitForChild("SetupFrames")
	local setup             = container:WaitForChild("Setup")
	local loadout           = setup:WaitForChild("Loadout")


	local setupCloseButton  = setupFrames:WaitForChild("CloseButton")


	local buttonToFrame = {
		["1Skin"]      = "Skins",
		["2Wraps"]     = "Wraps",
		["3Charms"]    = "Charms",
		["4Finishers"] = "Finishers",
	}


	local originalSizes = {}
	for _, frameName in pairs(buttonToFrame) do
		local frame = setupFrames:WaitForChild(frameName)
		originalSizes[frame] = frame.Size
	end


	local currentFrame = nil

	-- Animación al ABRIR
	local function animateEnter()
		TweenService:Create(careerCont, TWEEN_INFO, {
			Position = UDim2.new(-1.007, 0, 0.574, 0)
		}):Play()

		TweenService:Create(mainCloseButton, TWEEN_INFO, {
			Position = UDim2.new(0.5, 0, 1.946, 0)
		}):Play()

		TweenService:Create(weaponList, TWEEN_INFO, {
			Position = UDim2.new(1.995, 0, 0.5, 0)
		}):Play()


		TweenService:Create(setupCloseButton, TWEEN_INFO, {
			Position = UDim2.new(0.5, 0, 2.317, 0)
		}):Play()
	end


	local function animateExit()
		TweenService:Create(careerCont, TWEEN_INFO, {
			Position = UDim2.new(0.007, 0, 0.574, 0)
		}):Play()

		TweenService:Create(mainCloseButton, TWEEN_INFO, {
			Position = UDim2.new(0.5, 0, 0.946, 0)
		}):Play()

		TweenService:Create(weaponList, TWEEN_INFO, {
			Position = UDim2.new(0.995, 0, 0.5, 0)
		}):Play()

		TweenService:Create(setupCloseButton, TWEEN_INFO, {
			Position = UDim2.new(0.5, 0, 4, 0)
		}):Play()
	end


	for buttonName, frameName in pairs(buttonToFrame) do
		local button      = loadout:WaitForChild(buttonName)
		local targetFrame = setupFrames:WaitForChild(frameName)

		button.Activated:Connect(function()
			-- 1) Oculta los demás frames
			for _, frame in ipairs(setupFrames:GetChildren()) do
				if frame:IsA("GuiObject") and frame ~= setupCloseButton then
					frame.Visible = false
				end
			end


			local originalSize = originalSizes[targetFrame] or targetFrame.Size
			targetFrame.Size = UDim2.new(0, 0, 0, 0)
			targetFrame.Visible = true
			currentFrame = targetFrame

			TweenService:Create(targetFrame, TWEEN_INFO, { Size = originalSize }):Play()

			if parent then
				parent.Visible = false
			end


			animateEnter()
		end)
	end


	setupCloseButton.Activated:Connect(function()
		if not currentFrame then return end

		local originalSize = originalSizes[currentFrame] or currentFrame.Size

		local shrink = TweenService:Create(currentFrame, TWEEN_INFO, {
			Size = UDim2.new(0, 0, 0, 0)
		})
		shrink:Play()
		shrink.Completed:Wait()

		currentFrame.Visible = false
		currentFrame.Size = originalSize
		currentFrame = nil

		loadout.Visible = true
		animateExit()
	end)
end)
