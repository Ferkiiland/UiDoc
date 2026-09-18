local Players     = game:GetService("Players")
local lp          = Players.LocalPlayer
local playerGui   = lp:WaitForChild("PlayerGui")

local shopScreen  = playerGui:WaitForChild("ShopScreen")
local gifting     = playerGui:WaitForChild("Gifting")

local container   = shopScreen:WaitForChild("Container")
local shopsFrame  = container:WaitForChild("ShopsFrame")
local gift        = shopsFrame:WaitForChild("Gift")

local giftingContainer = gifting:WaitForChild("Container")
local giftFrame        = giftingContainer:WaitForChild("GiftFrame")
local giftingCloseBtn  = giftFrame:WaitForChild("CloseButton")

shopScreen:SetAttribute("IsEnabled", true)
gifting:SetAttribute("IsEnabled", false)

------------------------------------------------------------
-- BuyButton: ShopScreen OFF, Gifting ON
------------------------------------------------------------
local function connectBuyButton(buyButton)
	buyButton.Activated:Connect(function()
		shopScreen:SetAttribute("IsEnabled", false)
		gifting:SetAttribute("IsEnabled", true)
	end)
end

local function connectAllBuyButtons(parent)
	for _, child in ipairs(parent:GetDescendants()) do
		if child.Name == "BuyButton" and (child:IsA("TextButton") or child:IsA("ImageButton")) then
			connectBuyButton(child)
		end
	end
end

connectAllBuyButtons(gift)

gift.DescendantAdded:Connect(function(descendant)
	if descendant.Name == "BuyButton" and (descendant:IsA("TextButton") or descendant:IsA("ImageButton")) then
		connectBuyButton(descendant)
	end
end)

------------------------------------------------------------
-- CloseButton Gifting
------------------------------------------------------------
giftingCloseBtn.Activated:Connect(function()
	gifting:SetAttribute("IsEnabled", false)
	shopScreen:SetAttribute("IsEnabled", true)
end)
