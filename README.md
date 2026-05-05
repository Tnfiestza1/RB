-- [[ SERVICES ]]
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- [[ UI CREATION ]]
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LockSystemGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainButton = Instance.new("TextButton")
mainButton.Name = "LockButton"
mainButton.Size = UDim2.new(0, 120, 0, 50)
mainButton.Position = UDim2.new(0.5, -60, 0.8, 0)
mainButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
mainButton.Text = "LOCK: OFF"
mainButton.TextColor3 = Color3.new(1, 1, 1)
mainButton.Font = Enum.Font.GothamBold
mainButton.TextSize = 18
mainButton.Parent = screenGui

-- ทำให้ปุ่มมนขึ้น
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = mainButton

-- [[ DRAGGABLE LOGIC (ระบบลากปุ่มย้ายที่) ]]
local dragging, dragInput, dragStart, startPos

mainButton.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = mainButton.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

mainButton.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		mainButton.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

-- [[ LOCK SYSTEM LOGIC (ระบบล็อกตัวละคร) ]]
local isLocked = false
local attachment, antiKnockback

local function toggleLock()
	character = player.Character
	local rootPart = character:FindFirstChild("HumanoidRootPart")
	local humanoid = character:FindFirstChild("Humanoid")
	
	if not rootPart or not humanoid then return end

	isLocked = not isLocked

	if isLocked then
		-- สภาวะ LOCK: กันแรงผลัก + เดินไม่ได้
		mainButton.Text = "LOCK: ON"
		mainButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
		
		-- สร้างตัวกัน Knockback
		attachment = Instance.new("Attachment", rootPart)
		antiKnockback = Instance.new("LinearVelocity", rootPart)
		antiKnockback.MaxForce = math.huge
		antiKnockback.VectorVelocity = Vector3.new(0, 0, 0)
		antiKnockback.Attachment0 = attachment
		
		humanoid.WalkSpeed = 0
		humanoid.JumpPower = 0
		rootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
	else
		-- สภาวะ UNLOCK: กลับมาเป็นปกติ
		mainButton.Text = "LOCK: OFF"
		mainButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
		
		if antiKnockback then antiKnockback:Destroy() end
		if attachment then attachment:Destroy() end
		
		humanoid.WalkSpeed = 16
		humanoid.JumpPower = 50
	end
end

mainButton.MouseButton1Click:Connect(toggleLock)
