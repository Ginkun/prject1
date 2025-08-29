local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- Re-get character on respawn
player.CharacterAdded:Connect(function(char)
	character = char
	humanoid = char:WaitForChild("Humanoid")
end)

-- UI creation
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "OneClickTeleportGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Exit Button
local exitButton = Instance.new("TextButton")
exitButton.Name = "ExitButton"
exitButton.Size = UDim2.new(0, 150, 0, 50)
exitButton.Position = UDim2.new(0, 10, 0, 10)
exitButton.Text = "Exit"
exitButton.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
exitButton.TextColor3 = Color3.new(1, 1, 1)
exitButton.Font = Enum.Font.SourceSansBold
exitButton.TextSize = 20
exitButton.Parent = screenGui

-- Teleport NPCs Button
local teleportButton = Instance.new("TextButton")
teleportButton.Name = "TeleportButton"
teleportButton.Size = UDim2.new(0, 150, 0, 50)
teleportButton.Position = UDim2.new(0, 10, 0, 70)
teleportButton.Text = "Bring Enemies (R)"
teleportButton.BackgroundColor3 = Color3.fromRGB(0, 180, 90)
teleportButton.TextColor3 = Color3.new(1, 1, 1)
teleportButton.Font = Enum.Font.SourceSansBold
teleportButton.TextSize = 20
teleportButton.Parent = screenGui

-- Heal Button
local healButton = Instance.new("TextButton")
healButton.Name = "HealButton"
healButton.Size = UDim2.new(0, 150, 0, 50)
healButton.Position = UDim2.new(0, 10, 0, 130)
healButton.Text = "Heal (F)"
healButton.BackgroundColor3 = Color3.fromRGB(200, 100, 100)
healButton.TextColor3 = Color3.new(1, 1, 1)
healButton.Font = Enum.Font.SourceSansBold
healButton.TextSize = 20
healButton.Parent = screenGui

-- Speed Toggle Button
local speedButton = Instance.new("TextButton")
speedButton.Name = "SpeedButton"
speedButton.Size = UDim2.new(0, 150, 0, 50)
speedButton.Position = UDim2.new(0, 10, 0, 190)
speedButton.Text = "Speed (G): OFF"
speedButton.BackgroundColor3 = Color3.fromRGB(255, 200, 50)
speedButton.TextColor3 = Color3.new(0, 0, 0)
speedButton.Font = Enum.Font.SourceSansBold
speedButton.TextSize = 20
speedButton.Parent = screenGui

-- Settings
local offset = Vector3.new(0, 0, -5)
local maxDistance = 80
local speedEnabled = false

-- Utility: Check if model is NPC
local function isNPC(model)
	if not model:IsA("Model") then return false end
	if Players:GetPlayerFromCharacter(model) then return false end
	return model:FindFirstChild("Humanoid") and model:FindFirstChild("HumanoidRootPart")
end

-- Teleport Nearby NPCs
local function teleportNearbyNPCs()
	local char = player.Character
	if not char then return end

	local myHRP = char:FindFirstChild("HumanoidRootPart")
	if not myHRP then return end

	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("Part") and obj.Name == "HumanoidRootPart" then
			local npc = obj.Parent
			if isNPC(npc) and (obj.Position - myHRP.Position).Magnitude <= maxDistance then
				obj.CFrame = myHRP.CFrame * CFrame.new(offset)
			end
		end
	end
end

-- Find nearest usable seat
local function findNearestSeat()
	local char = player.Character
	if not char then return nil end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return nil end

	local nearest = nil
	local shortestDist = math.huge

	for _, seat in ipairs(workspace:GetDescendants()) do
		if seat:IsA("Seat") and not seat.Disabled then
			local dist = (seat.Position - hrp.Position).Magnitude
			if dist < shortestDist then
				shortestDist = dist
				nearest = seat
			end
		end
	end
	return nearest
end

local function healPlayer()
	local char = player.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	local humanoid = char:FindFirstChild("Humanoid")
	if not hrp or not humanoid then return end

	local seat = findNearestSeat()
	if not seat then
		warn("No usable seat found!")
		return
	end

	local originalCFrame = hrp.CFrame

	-- Anchor the seat so it doesn't move with the player
	local wasAnchored = seat.Anchored
	seat.Anchored = true

	-- Teleport player to seat (position a bit above so player can sit)
	hrp.CFrame = seat.CFrame + Vector3.new(0, 2, 0)

	-- Wait until full HP
	while true do
		task.wait(1)
		local hpLabel = player:FindFirstChild("PlayerGui")
			and player.PlayerGui:FindFirstChild("Main")
			and player.PlayerGui.Main:FindFirstChild("HomePage")
			and player.PlayerGui.Main.HomePage:FindFirstChild("Bottom")
			and player.PlayerGui.Main.HomePage.Bottom:FindFirstChild("Main")
			and player.PlayerGui.Main.HomePage.Bottom.Main:FindFirstChild("Health")
			and player.PlayerGui.Main.HomePage.Bottom.Main.Health:FindFirstChild("Num")
			and player.PlayerGui.Main.HomePage.Bottom.Main.Health.Num:FindFirstChild("Health")

		if hpLabel and hpLabel:IsA("TextLabel") then
			local text = hpLabel.Text -- example: "120/200"
			local current, max = string.match(text, "(%d+)%s*/%s*(%d+)")
			if current and max and tonumber(current) == tonumber(max) then
				break
			end
		else
			warn("HP label not found!")
			break
		end
	end

	-- Unsit before teleporting back
	humanoid.Sit = false
	task.wait(0.1) -- short wait to ensure unsitting

	-- Teleport back
	hrp.CFrame = originalCFrame

	-- Restore seat anchor state
	seat.Anchored = wasAnchored
end

-- Infinite jump
UserInputService.JumpRequest:Connect(function()
	if humanoid then
		humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

-- LoopSpeed function
RunService.Stepped:Connect(function()
	if speedEnabled and humanoid then
		humanoid.WalkSpeed = 60
	end
end)

-- Toggle speed
local function toggleSpeed()
	speedEnabled = not speedEnabled
	speedButton.Text = "Speed (G): " .. (speedEnabled and "ON" or "OFF")
	if not speedEnabled and humanoid then
		humanoid.WalkSpeed = 16
	end
end

-- Connect buttons
teleportButton.MouseButton1Click:Connect(teleportNearbyNPCs)
healButton.MouseButton1Click:Connect(healPlayer)
speedButton.MouseButton1Click:Connect(toggleSpeed)

-- Hotkeys
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.R then
		teleportNearbyNPCs()
	elseif input.KeyCode == Enum.KeyCode.F then
		healPlayer()
	elseif input.KeyCode == Enum.KeyCode.G then
		toggleSpeed()
	end
end)

-- Exit button click: remove the GUI
exitButton.MouseButton1Click:Connect(function()
	screenGui:Destroy()
end)
