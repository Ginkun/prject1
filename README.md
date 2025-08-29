--[[
Dungeon Heroes GUI Script - Auto Kill (Testando várias funções)
Este script tenta atacar mobs automaticamente usando diferentes métodos,
para aumentar as chances de funcionar mesmo sem saber exatamente o RemoteEvent do jogo.
]]

-- Serviços básicos
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- GUI simples
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "DH_GUI"


local function printRemotes(folder)
    for _, v in pairs(folder:GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
            print("Remote encontrado:", v:GetFullName())
        end
    end
end

local mobsFolder = ReplicatedStorage:FindFirstChild("Systems")
if mobsFolder and mobsFolder:FindFirstChild("Mobs") then
    printRemotes(mobsFolder.Mobs)
end

local function createButton(text, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 180, 0, 36)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(44,44,60)
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 20
    btn.Parent = ScreenGui
    return btn
end

local autoKillBtn = createButton("Auto Kill: OFF", 10)
local floatBtn    = createButton("Flutuar: OFF", 54)

local floatY = 50
local floatEnabled = false
local autoKill = false
local autoKillConnection, floatConnection

-- TEST AUTO ATTACK FUNCTIONS
local function tryAttackMethods(mob)
    -- 1: Tentar tocar fisicamente o mob (alguns jogos aceitam)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = mob.HumanoidRootPart.CFrame + Vector3.new(0,2,0)
    end

    -- 2: Tentar RemoteEvents comuns em ReplicatedStorage
    for _,v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and v.Name:lower():find("attack") then
            pcall(function()
                v:FireServer(mob)
            end)
        elseif v:IsA("RemoteFunction") and v.Name:lower():find("attack") then
            pcall(function()
                v:InvokeServer(mob)
            end)
        end
    end

    -- 3: Tentar eventos genéricos ("Damage", "Hit", "Skill", etc)
    for _,v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name:lower():find("damage") or v.Name:lower():find("hit") or v.Name:lower():find("skill")) then
            pcall(function()
                v:FireServer(mob)
            end)
        end
    end

    -- 4: Procurar por eventos em StarterGui ou outras pastas comuns
    for _,service in pairs({game:GetService("StarterGui"), game:GetService("StarterPack")}) do
        for _,v in pairs(service:GetDescendants()) do
            if v:IsA("RemoteEvent") and v.Name:lower():find("attack") then
                pcall(function()
                    v:FireServer(mob)
                end)
            end
        end
    end

    -- 5: Ativar ferramentas equipadas (para jogos que usam Tool:Activate)
    local tool = char and char:FindFirstChildOfClass("Tool")
    if tool then
        pcall(function()
            tool:Activate()
        end)
    end
end

-- Função de Auto Kill
local function autoAttack()
    while autoKill do
        -- Buscar mobs (ajuste o filtro se necessário)
        for _, mob in pairs(workspace:GetChildren()) do
            if mob:IsA("Model") and mob:FindFirstChild("Humanoid") and mob:FindFirstChild("HumanoidRootPart") and mob ~= LocalPlayer.Character then
                if mob.Humanoid.Health > 0 then
                    tryAttackMethods(mob)
                    wait(0.1)
                end
            end
        end
        wait(0.3)
    end
end

-- Botão do Auto Kill
autoKillBtn.MouseButton1Click:Connect(function()
    autoKill = not autoKill
    autoKillBtn.Text = "Auto Kill: " .. (autoKill and "ON" or "OFF")
    if autoKill then
        spawn(autoAttack)
    end
end)

-- Botão de Flutuar
floatBtn.MouseButton1Click:Connect(function()
    floatEnabled = not floatEnabled
    floatBtn.Text = "Flutuar: " .. (floatEnabled and "ON" or "OFF")
    if floatEnabled then
        if not floatConnection then
            floatConnection = RunService.RenderStepped:Connect(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    hrp.CFrame = CFrame.new(hrp.Position.X, floatY, hrp.Position.Z)
                    hrp.Velocity = Vector3.new(0,0,0)
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

print("Dungeon Heroes Auto Kill Script carregado!\nSe algum ataque funcionar, avise qual RemoteEvent aparece no console F9.")

--[[
Dicas:
- Se algum mob morrer/receber dano, veja se aparece erro ou informação no console F9 (pode aparecer o nome do remote).
- Se o personagem for teleportado para cima do mob, mas não causar dano, geralmente é questão de descobrir o RemoteEvent correto.
- Se precisar de mais métodos de ataque, envie prints do console F9 ao atacar manualmente.
]]
