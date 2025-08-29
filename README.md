--[[
Dungeon Heroes Utility Script
Features:
- GUI menu with toggles for:
  - Auto Kill (auto attack mobs in dungeons)
  - Player Height (set Y/XYZ to float above mobs)
Instructions:
- Execute in Roblox executor (for educational purposes only).
- Works for "Dungeon Heroes" (may require updates if game changes).
- GUI will appear on the left; toggle features as needed.
]]

-- Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- UI Library (Basic)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DungeonHeroesGUI"
ScreenGui.Parent = game.CoreGui

local function createButton(text, position)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 180, 0, 36)
    btn.Position = UDim2.new(0, 10, 0, position)
    btn.BackgroundColor3 = Color3.fromRGB(44,44,60)
    btn.BorderSizePixel = 0
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 20
    btn.Parent = ScreenGui
    return btn
end

local function createLabel(text, position)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0, 180, 0, 36)
    lbl.Position = UDim2.new(0, 10, 0, position)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.new(1,1,1)
    lbl.Font = Enum.Font.SourceSans
    lbl.TextSize = 18
    lbl.Parent = ScreenGui
    return lbl
end

-- Variables
local autoKill = false
local floatEnabled = false
local floatY = 50 -- Default height above ground
local autoKillConnection
local floatConnection

-- UI
local yLabel = createLabel("Altura (Y): "..tostring(floatY), 98)
local upBtn = createButton("Aumentar Altura (+5)", 134)
local downBtn = createButton("Diminuir Altura (-5)", 174)
local autoKillBtn = createButton("Auto Kill: OFF", 10)
local floatBtn = createButton("Flutuar: OFF", 54)

-- Toggle Auto Kill
autoKillBtn.MouseButton1Click:Connect(function()
    autoKill = not autoKill
    autoKillBtn.Text = "Auto Kill: " .. (autoKill and "ON" or "OFF")
    if autoKill then
        -- Start attacking
        if not autoKillConnection then
            autoKillConnection = RunService.RenderStepped:Connect(function()
                -- Find nearest mobs (customize for game's mob model)
                local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if not root then return end

                -- Find mobs (may need to adjust for game's structure)
                for _, mob in pairs(workspace:GetDescendants()) do
                    if mob:IsA("Model") and mob:FindFirstChild("Humanoid") and mob ~= LocalPlayer.Character then
                        local mobRoot = mob:FindFirstChild("HumanoidRootPart") or mob.PrimaryPart
                        if mobRoot and (mob.Humanoid.Health > 0) then
                            -- Simulate attack (touch or remote event)
                            -- Method 1: Move close to mob and simulate attack
                            root.CFrame = mobRoot.CFrame + Vector3.new(0, 2, 0)
                            -- Method 2: Fire remote event if found:
                            -- for _,v in pairs(getgc(true)) do
                            --   if typeof(v)=="table" and v.Attack then v:Attack() end
                            -- end
                            -- Wait a short time to avoid rapid teleports
                            wait(0.1)
                        end
                    end
                end
            end)
        end
    else
        if autoKillConnection then
            autoKillConnection:Disconnect()
            autoKillConnection = nil
        end
    end
end)

-- Toggle Float
floatBtn.MouseButton1Click:Connect(function()
    floatEnabled = not floatEnabled
    floatBtn.Text = "Flutuar: " .. (floatEnabled and "ON" or "OFF")
    if floatEnabled then
        if not floatConnection then
            floatConnection = RunService.RenderStepped:Connect(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.Velocity = Vector3.new(0,0,0)
                    hrp.CFrame = CFrame.new(hrp.Position.X, floatY, hrp.Position.Z)
                end
            end)
        end
    else
        if floatConnection then
            floatConnection:Disconnect()
            floatConnection = nil
        end
    end
end)

-- Increase Y
upBtn.MouseButton1Click:Connect(function()
    floatY = floatY + 5
    yLabel.Text = "Altura (Y): "..tostring(floatY)
end)

-- Decrease Y
downBtn.MouseButton1Click:Connect(function()
    floatY = floatY - 5
    yLabel.Text = "Altura (Y): "..tostring(floatY)
end)

-- Draggable GUI (optional)
local dragToggle, dragInput, dragStart, startPos
ScreenGui.Enabled = true
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 200
ScreenGui.IgnoreGuiInset = true

local function dragify(gui)
    gui.Active = true
    gui.Draggable = true
end

for _, v in ipairs(ScreenGui:GetChildren()) do
    if v:IsA("TextButton") or v:IsA("TextLabel") then
        dragify(v)
    end
end

-- Notice
print("Dungeon Heroes Script Loaded! GUI on left of screen.")

--[[
Obs:
- O ataque automático depende da estrutura do jogo. Se os mobs tiverem um RemoteEvent para atacar, substitua o método de ataque.
- A função de flutuar pode ser ajustada para X/Z se quiser movimentos laterais (adicione sliders/inputs).
- Para evitar detecção, use moderadamente.
]]
