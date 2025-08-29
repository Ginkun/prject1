local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local runService = game:GetService("RunService")
local userInput = game:GetService("UserInputService")

local autoFarm = true
local HIT_RANGE = 10

local function isArcher(mob)
    return mob.Name:lower():find("archer") ~= nil 
end

local function getClosestMob()
    local closestMob = nil
    local closestDistance = math.huge
    for _, mob in ipairs(workspace.Characters:GetChildren()) do
        if mob:IsA("Model") and mob ~= character and mob:FindFirstChild("HumanoidRootPart") then
            local dist = (mob.HumanoidRootPart.Position - humanoidRootPart.Position).Magnitude
            if dist < closestDistance then
                closestDistance = dist
                closestMob = mob
            end
        end
    end
    return closestMob
end

local function teleportUnderEnemy(mob)
    if humanoidRootPart and mob:FindFirstChild("HumanoidRootPart") then
        local yOffset = -7
        local targetCFrame = mob.HumanoidRootPart.CFrame 
            * CFrame.new(0, yOffset, 0)
            * CFrame.Angles(math.rad(90), 0, 0)
        humanoidRootPart.CFrame = targetCFrame
    end
end

local function isInRange(mob, range)
    local mobRootPart = mob:FindFirstChild("HumanoidRootPart")
    if not mobRootPart or not humanoidRootPart then return false end
    return (mobRootPart.Position - humanoidRootPart.Position).Magnitude <= range
end

local function equipWeapon()
    local args = {
        [1] = "Equip/UnEquip"
    }
    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("WeaponsEvent"):FireServer(unpack(args))
end

equipWeapon()

local function useSkill(skillName)
    local args = {
        [1] = { ["Skill"] = skillName, ["Function"] = "Activate" },
        [2] = humanoidRootPart.Position,
        [3] = true
    }
    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Skill"):FireServer(unpack(args))
end

local function attack()
    while autoFarm do
        local mob = getClosestMob()
        if mob then
            if not isInRange(mob, HIT_RANGE) then
                teleportUnderEnemy(mob)
            end
            humanoidRootPart.Anchored = true
            game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Combat")
                :FireServer("Attack")
            useSkill("FlurryOfBlows")
            useSkill("BloodyCut")
            useSkill("Invisibility")
        end
        task.wait(0.5)
    end
end

local function stopAutoFarm()
    autoFarm = false
    humanoidRootPart.Anchored = false
end

local function enableNoFall()
    runService.Stepped:Connect(function()
        if autoFarm and humanoidRootPart then
            humanoidRootPart.Velocity = Vector3.new(0, 0, 0)
        end
    end)
end

enableNoFall()

userInput.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Zero then
        stopAutoFarm()
    end
end)

attack()
