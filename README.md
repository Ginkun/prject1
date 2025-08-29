--[[ Dungeon Heroes - Auto Kill Testador de Remotes de Ataque
Ativa cada RemoteEvent candidato para cada mob encontrado.
Veja no console se algum deles causa dano nos mobs!
]]

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Lista dos possíveis remotes de ataque
local remotePaths = {
    "Systems.Combat.DealDamage",
    "Systems.Combat.SetDamageType",
    "Systems.Combat.PlayerAttack",
    "Systems.Combat._PlayerSkillAttack",
    "Systems.Combat.HitboxIndicator",
    "Systems.Combat.AddToHitList",
    "Systems.Mobs.PetAttackAnim",
    "Systems.Mobs.RunAttackModule",
    "Systems.Mobs.RunPetAttackModule"
}

-- Pega o remote pelo caminho
local function getRemote(path)
    local node = ReplicatedStorage
    for seg in string.gmatch(path, "[^%.]+") do
        node = node:FindFirstChild(seg)
        if not node then return nil end
    end
    return node
end

-- Encontra mobs
local function getMobs()
    local mobs = {}
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") and obj ~= LocalPlayer.Character then
            if obj.Humanoid.Health > 0 then
                table.insert(mobs, obj)
            end
        end
    end
    return mobs
end

-- GUI simples
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DH_AutoKillTest"
local btn = Instance.new("TextButton", gui)
btn.Size = UDim2.new(0, 200, 0, 40)
btn.Position = UDim2.new(0, 10, 0, 120)
btn.Text = "TESTAR AUTO KILL"
btn.BackgroundColor3 = Color3.fromRGB(60,60,90)
btn.TextColor3 = Color3.new(1,1,1)
btn.Font = Enum.Font.SourceSansBold
btn.TextSize = 22

btn.MouseButton1Click:Connect(function()
    print("==== INICIANDO TESTE DE REMOTES DE ATAQUE ====")
    local mobs = getMobs()
    for _, mob in ipairs(mobs) do
        print("Testando em mob:", mob.Name)
        for _, path in ipairs(remotePaths) do
            local remote = getRemote(path)
            if remote then
                print("Chamando remote:", remote:GetFullName())
                pcall(function()
                    -- Tenta argumentos comuns: mob, mob.Humanoid, mob.HumanoidRootPart
                    remote:FireServer(mob)
                    remote:FireServer(mob.Humanoid)
                    remote:FireServer(mob.HumanoidRootPart)
                end)
            else
                print("Remote não encontrado:", path)
            end
        end
    end
    print("==== TESTE FINALIZADO! Veja se algum mob tomou dano! ====")
end)
