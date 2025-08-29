-- Script de Dungeon para Roblox
-- Baseado no script open source by tsuo
-- Foco em sistema de HIT para NPCs/Mobs

-- Configurações Globais
_G.AutoFarm = false
_G.FastAttack = false
_G.FastAttackDelay = 0.1
_G.SelectWeapon = ""
_G.AutoHaki = true
_G.TeleportSpeed = 300

-- Detecção do Jogo (PlaceIds comuns de dungeons)
local DungeonGames = {
    [2753915549] = "Blox Fruits World 1",
    [4442272183] = "Blox Fruits World 2", 
    [7449423635] = "Blox Fruits World 3",
    [537413528] = "Build A Boat",
    [286090429] = "Arsenal",
    [1537690962] = "Bee Swarm Simulator"
}

local CurrentGame = DungeonGames[game.PlaceId]
if not CurrentGame then
    game:GetService("Players").LocalPlayer:Kick("Jogo não suportado para dungeon farming!")
    return
end

print("Dungeon Script carregado para: " .. CurrentGame)

-- Função para ativar Haki automaticamente
function AutoHaki()
    if not game:GetService("Players").LocalPlayer.Character:FindFirstChild("HasBuso") then
        game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Buso")
    end
end

-- Função para equipar arma
function EquipWeapon(ToolSe)
    if not _G.NotAutoEquip then
        if game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe) then
            Tool = game.Players.LocalPlayer.Backpack:FindFirstChild(ToolSe)
            wait(.1)
            game.Players.LocalPlayer.Character.Humanoid:EquipTool(Tool)
        end
    end
end

-- Função de teleporte otimizada
function topos(Pos)
    local Distance = (Pos.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
    local Speed
    
    if Distance < 25 then
        Speed = 5000
    elseif Distance < 50 then
        Speed = 2000
    elseif Distance < 150 then
        Speed = 800
    elseif Distance < 250 then
        Speed = 600
    elseif Distance < 500 then
        Speed = 300
    elseif Distance < 750 then
        Speed = 250
    elseif Distance >= 1000 then
        Speed = 200
    end
    
    game:GetService("TweenService"):Create(
        game:GetService("Players").LocalPlayer.Character.HumanoidRootPart,
        TweenInfo.new(Distance/Speed, Enum.EasingStyle.Linear),
        {CFrame = Pos}
    ):Play()
end

-- Função para obter a arma equipada
function GetBladeHit()
    local CombatFrameworkLib = debug.getupvalues(require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework))
    local CmrFwLib = CombatFrameworkLib[2]
    local p13 = CmrFwLib.activeController
    local weapon = p13.blades[1]
    if not weapon then 
        return weapon
    end
    while weapon.Parent ~= game.Players.LocalPlayer.Character do
        weapon = weapon.Parent 
    end
    return weapon
end

-- Função principal de ataque/hit
function AttackHit()
    local CombatFrameworkLib = debug.getupvalues(require(game:GetService("Players").LocalPlayer.PlayerScripts.CombatFramework))
    local CmrFwLib = CombatFrameworkLib[2]
    local plr = game.Players.LocalPlayer
    
    for i = 1, 1 do
        local bladehit = require(game.ReplicatedStorage.CombatFramework.RigLib).getBladeHits(plr.Character,{plr.Character.HumanoidRootPart},60)
        local cac = {}
        local hash = {}
        
        for k, v in pairs(bladehit) do
            if v.Parent:FindFirstChild("HumanoidRootPart") and not hash[v.Parent] then
                table.insert(cac, v.Parent.HumanoidRootPart)
                hash[v.Parent] = true
            end
        end
        
        bladehit = cac
        if #bladehit > 0 then
            pcall(function()
                CmrFwLib.activeController.timeToNextAttack = 1
                CmrFwLib.activeController.attacking = false
                CmrFwLib.activeController.blocking = false
                CmrFwLib.activeController.timeToNextBlock = 0
                CmrFwLib.activeController.increment = 3
                CmrFwLib.activeController.hitboxMagnitude = 120
                CmrFwLib.activeController.focusStart = 0
                game:GetService("ReplicatedStorage").RigControllerEvent:FireServer("weaponChange",tostring(GetBladeHit()))
                game:GetService("ReplicatedStorage").RigControllerEvent:FireServer("hit", bladehit, i, "")
            end)
        end
    end
end

-- Loop de ataque rápido
spawn(function()
    while wait(.1) do
        if _G.FastAttack then
            pcall(function()
                repeat task.wait(_G.FastAttackDelay)
                    AttackHit()
                until not _G.FastAttack
            end)
        end
    end
end)

-- Função para encontrar NPCs/Mobs próximos
function FindNearestEnemy()
    local nearestEnemy = nil
    local shortestDistance = math.huge
    
    for i,v in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
        if v.Name and v:FindFirstChild("Humanoid") and v:FindFirstChild("HumanoidRootPart") then
            if v.Humanoid.Health > 0 then
                local distance = (v.HumanoidRootPart.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                if distance < shortestDistance then
                    shortestDistance = distance
                    nearestEnemy = v
                end
            end
        end
    end
    
    return nearestEnemy
end

-- Sistema de Farm Automático para Dungeons
function AutoFarmDungeon()
    local enemy = FindNearestEnemy()
    
    if enemy then
        repeat wait()
            -- Ativar Haki se necessário
            if _G.AutoHaki then
                AutoHaki()
            end
            
            -- Equipar arma se especificada
            if _G.SelectWeapon ~= "" then
                EquipWeapon(_G.SelectWeapon)
            end
            
            -- Teleportar para o inimigo
            topos(enemy.HumanoidRootPart.CFrame * CFrame.new(0, 20, 0))
            
            -- Configurar inimigo para facilitar o hit
            enemy.HumanoidRootPart.CanCollide = false
            enemy.HumanoidRootPart.Size = Vector3.new(60, 60, 60)
            enemy.Humanoid.WalkSpeed = 0
            
            -- Ativar ataque rápido
            _G.FastAttack = true
            
            -- Simular clique para atacar
            game:GetService("VirtualUser"):CaptureController()
            game:GetService("VirtualUser"):Button1Down(Vector2.new(1280, 672), game.Workspace.CurrentCamera.CFrame)
            
        until not _G.AutoFarm or not enemy.Parent or enemy.Humanoid.Health <= 0
        
        _G.FastAttack = false
    end
end

-- Loop principal de farm
spawn(function()
    while wait(1) do
        if _G.AutoFarm then
            pcall(function()
                AutoFarmDungeon()
            end)
        end
    end
end)

-- Remover efeitos visuais para melhor performance
spawn(function()
    while wait() do
        for i,v in pairs(game:GetService("Workspace")["_WorldOrigin"]:GetChildren()) do
            pcall(function()
                if v.Name == ("CurvedRing") or v.Name == ("SlashHit") or v.Name == ("SwordSlash") or v.Name == ("SlashTail") or v.Name == ("Sounds") then
                    v:Destroy()
                end
            end)
        end
    end
end)

-- Interface simples para controle
print("=== DUNGEON SCRIPT CARREGADO ===")
print("Comandos disponíveis:")
print("_G.AutoFarm = true/false -- Ativar/Desativar farm automático")
print("_G.FastAttack = true/false -- Ativar/Desativar ataque rápido")
print("_G.SelectWeapon = 'NomeArma' -- Definir arma para equipar")
print("_G.AutoHaki = true/false -- Ativar/Desativar haki automático")
print("================================")

-- Exemplo de uso:
-- _G.AutoFarm = true
-- _G.SelectWeapon = "Katana"
-- _G.FastAttack = true
