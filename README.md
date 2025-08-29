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


-- Função melhorada para encontrar e listar todos os remotes
local function printAllRemotes()
    print("=== BUSCANDO TODOS OS REMOTES ===")
    
    -- Buscar em ReplicatedStorage
    print("--- ReplicatedStorage ---")
    for _, v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
            print("Remote encontrado:", v:GetFullName(), "Tipo:", v.ClassName)
        end
    end
    
    -- Buscar em StarterGui
    print("--- StarterGui ---")
    for _, v in pairs(game:GetService("StarterGui"):GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
            print("Remote encontrado:", v:GetFullName(), "Tipo:", v.ClassName)
        end
    end
    
    -- Buscar em StarterPack
    print("--- StarterPack ---")
    for _, v in pairs(game:GetService("StarterPack"):GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
            print("Remote encontrado:", v:GetFullName(), "Tipo:", v.ClassName)
        end
    end
    
    -- Buscar em PlayerScripts
    print("--- PlayerScripts ---")
    local playerScripts = LocalPlayer:FindFirstChild("PlayerScripts")
    if playerScripts then
        for _, v in pairs(playerScripts:GetDescendants()) do
            if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
                print("Remote encontrado:", v:GetFullName(), "Tipo:", v.ClassName)
            end
        end
    end
    
    print("=== FIM DA BUSCA ===")
end

-- Executar busca de remotes
printAllRemotes()

-- Função para monitorar todos os eventos de rede
local function monitorNetworkEvents()
    print("=== INICIANDO MONITORAMENTO DE REDE ===")
    
    -- Monitorar RemoteEvents
    for _, v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") then
            pcall(function()
                v.OnClientEvent:Connect(function(...)
                    local args = {...}
                    local argsStr = ""
                    for i, arg in pairs(args) do
                        if type(arg) == "string" or type(arg) == "number" then
                            argsStr = argsStr .. tostring(arg)
                        else
                            argsStr = argsStr .. type(arg)
                        end
                        if i < #args then argsStr = argsStr .. ", " end
                    end
                    print("[REMOTE EVENT RECEBIDO]", v:GetFullName(), "Args:", argsStr)
                end)
            end)
        end
    end
    
    print("Monitoramento de rede ativo!")
end

-- Iniciar monitoramento
spawn(function()
    wait(2) -- Aguardar o jogo carregar
    pcall(monitorNetworkEvents)
end)

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

local autoHitBtn = createButton("Auto Hit: OFF", 10)
local floatBtn    = createButton("Flutuar: OFF", 54)
local debugBtn    = createButton("Listar Remotes", 98)
local mobBtn      = createButton("Buscar Mobs", 142)

-- VARIÁVEIS GLOBAIS
local floatY = 50
local floatEnabled = false
local autoHitActive = false
local autoHitConnection, floatConnection
local hitDelay = 0.05 -- Delay entre hits em segundos (mais agressivo baseado no console observado)

-- Função para equipar automaticamente uma arma
local function equipWeapon()
    local char = LocalPlayer.Character
    if not char then return end
    
    local backpack = LocalPlayer.Backpack
    if not backpack then return end
    
    -- Verificar se já tem uma ferramenta equipada
    local equippedTool = char:FindFirstChildOfClass("Tool")
    if equippedTool then
        return equippedTool -- Já tem uma arma equipada
    end
    
    -- Procurar por ferramentas na mochila
    for _, tool in pairs(backpack:GetChildren()) do
        if tool:IsA("Tool") then
            pcall(function()
                tool.Parent = char
                print("🗡️ Arma equipada:", tool.Name)
                return tool
            end)
            break
        end
    end
end

-- SISTEMA AUTO HIT - Simula cliques automáticos para 100% de acerto
local function autoHit()
    local char = LocalPlayer.Character
    if not char then 
        print("🚫 Auto Hit: Character não encontrado")
        return 
    end
    
    local humanoid = char:FindFirstChild("Humanoid")
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    if not humanoid or not rootPart then 
        print("🚫 Auto Hit: Humanoid ou HumanoidRootPart não encontrado")
        return 
    end
    
    print("🔍 Auto Hit: Procurando mobs próximos...")
    
    -- Ativar flutuação automaticamente quando Auto Hit estiver ON
    if not floatEnabled then
        floatEnabled = true
        floatBtn.Text = "Flutuar: ON"
        floatBtn.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        
        -- Iniciar flutuação com BodyPosition (posição fixa)
        if floatConnection then floatConnection:Disconnect() end
        floatConnection = RunService.Heartbeat:Connect(function()
            if char and char:FindFirstChild("HumanoidRootPart") then
                local bodyPosition = char.HumanoidRootPart:FindFirstChild("BodyPosition")
                if not bodyPosition then
                    bodyPosition = Instance.new("BodyPosition")
                    bodyPosition.MaxForce = Vector3.new(4000, 4000, 4000)
                    bodyPosition.Position = char.HumanoidRootPart.Position + Vector3.new(0, 5, 0)
                    bodyPosition.Parent = char.HumanoidRootPart
                else
                    -- Atualizar posição para manter flutuando 5 studs acima
                    local currentPos = char.HumanoidRootPart.Position
                    bodyPosition.Position = Vector3.new(currentPos.X, currentPos.Y + 5, currentPos.Z)
                end
            end
        end)
        
        print("🌟 Flutuação ativada automaticamente com Auto Hit")
    end
    
    -- Equipar arma automaticamente
    equipWeapon()
    
    -- Encontrar o mob mais próximo (apenas monstros/NPCs, não players)
    local nearestMob = nil
    local shortestDistance = math.huge
    local totalEntitiesChecked = 0
    local validTargetsFound = 0
    
    print("🔍 Iniciando busca por mobs em workspace...")
    
    for _, obj in pairs(workspace:GetDescendants()) do
        totalEntitiesChecked = totalEntitiesChecked + 1
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
            -- Verificar se não é o próprio player
            if obj ~= char and obj.Humanoid.Health > 0 then
                -- Filtros para detectar apenas monstros/NPCs:
                local isValidTarget = false
                
                -- 1. Não deve ser um player (Players service)
                local isPlayer = false
                for _, player in pairs(game.Players:GetPlayers()) do
                    if player.Character == obj then
                        isPlayer = true
                        break
                    end
                end
                
                if not isPlayer then
                    -- 2. Verificar se tem características de mob/NPC
                    local humanoid = obj.Humanoid
                    
                    -- NPCs geralmente têm DisplayName diferente ou vazio
                    -- Players têm DisplayName igual ao nome do usuário
                    if humanoid.DisplayName == "" or humanoid.DisplayName ~= obj.Name then
                        isValidTarget = true
                    end
                    
                    -- 3. Verificar se está em pastas típicas de NPCs
                    local parent = obj.Parent
                    if parent and (parent.Name:lower():find("npc") or parent.Name:lower():find("mob") or 
                                  parent.Name:lower():find("enemy") or parent.Name:lower():find("monster")) then
                        isValidTarget = true
                    end
                    
                    -- 4. Verificar se tem scripts típicos de NPCs
                    if obj:FindFirstChild("NPC") or obj:FindFirstChild("Monster") or obj:FindFirstChild("Enemy") then
                        isValidTarget = true
                    end
                    
                    -- 5. Verificar se não tem Player como owner
                    if not obj:FindFirstChild("Player") and not obj:GetAttribute("PlayerId") then
                        isValidTarget = true
                    end
                    
                    -- 6. Verificar nomes típicos de monstros (adicione mais conforme necessário)
                    local mobNames = {"bandit", "pirate", "marine", "boss", "guard", "soldier", "thug", "enemy", "monster", "npc"}
                    for _, mobName in pairs(mobNames) do
                        if obj.Name:lower():find(mobName) then
                            isValidTarget = true
                            break
                        end
                    end
                end
                
                -- Só considerar como alvo se passou nos filtros
                if isValidTarget then
                    validTargetsFound = validTargetsFound + 1
                    local distance = (rootPart.Position - obj.HumanoidRootPart.Position).Magnitude
                    print("✅ Alvo válido encontrado:", obj.Name, "Distância:", math.floor(distance), "studs")
                    if distance < shortestDistance and distance <= 50 then -- Alcance de 50 studs
                        shortestDistance = distance
                        nearestMob = obj
                        print("🎯 Novo alvo mais próximo:", obj.Name, "Distância:", math.floor(distance))
                    end
                end
            end
        end
    end
    
    print("📊 Busca concluída - Entidades verificadas:", totalEntitiesChecked, "Alvos válidos:", validTargetsFound)
    
    if nearestMob then
        print("🎯 Auto Hit ativo - Atacando MOB:", nearestMob.Name, "Distância:", math.floor(shortestDistance))
        print("📍 Tipo de alvo confirmado: NPC/Monster (não é player)")
        
        -- Teleporte automático mais agressivo (baseado no console observado)
        local targetPosition = nearestMob.HumanoidRootPart.Position
        local safeDistance = 5 -- Distância mais próxima para atacar
        local direction = (rootPart.Position - targetPosition).Unit
        local teleportPosition = targetPosition + (direction * safeDistance)
        
        pcall(function()
            -- Teleportar flutuando até o mob (mais agressivo) com posição específica
            local floatPosition = teleportPosition + Vector3.new(0, 8, 0) -- +8 para manter flutuando mais alto
            rootPart.CFrame = CFrame.new(floatPosition)
            
            -- Atualizar BodyPosition se existir para manter a posição específica
            local bodyPosition = rootPart:FindFirstChild("BodyPosition")
            if bodyPosition then
                bodyPosition.Position = floatPosition
            end
            
            print("🌀 Teleportado para mob:", nearestMob.Name)
            print("📍 Posição específica (X,Y,Z):", math.floor(floatPosition.X), math.floor(floatPosition.Y), math.floor(floatPosition.Z))
        end)
        
        -- Testar teleporte via RemoteEvent também
        pcall(function()
            local teleportEvent = game.ReplicatedStorage:FindFirstChild("Systems")
            if teleportEvent then
                teleportEvent = teleportEvent:FindFirstChild("Teleport")
                if teleportEvent then
                    teleportEvent = teleportEvent:FindFirstChild("TeleportPlayer")
                    if teleportEvent and teleportEvent:IsA("RemoteEvent") then
                        teleportEvent:FireServer(nearestMob)
                        print("🌀 Teleporte via RemoteEvent testado")
                    end
                end
            end
        end)
        
        -- Aguardar menos tempo para ser mais agressivo
        wait(0.05)
        
        -- Chamar função de RemoteEvents específicos
        trySpecificRemotes(nearestMob)
        
        -- Simular clique do mouse (método 1)
        pcall(function()
            local mouse = LocalPlayer:GetMouse()
            if mouse then
                mouse.Button1Down:Fire()
                wait(0.05)
                mouse.Button1Up:Fire()
            end
        end)
        
        -- Simular UserInputService (método 2)
        pcall(function()
            local UserInputService = game:GetService("UserInputService")
            UserInputService.InputBegan:Fire({
                UserInputType = Enum.UserInputType.MouseButton1,
                UserInputState = Enum.UserInputState.Begin
            })
            wait(0.05)
            UserInputService.InputEnded:Fire({
                UserInputType = Enum.UserInputType.MouseButton1,
                UserInputState = Enum.UserInputState.End
            })
        end)
        
        -- Ativar ferramenta equipada (método 3)
        pcall(function()
            local tool = char:FindFirstChildOfClass("Tool")
            if tool then
                tool:Activate()
                print("🔧 Ferramenta ativada:", tool.Name)
            end
        end)
        
        -- Simular tecla de ataque (método 4)
        pcall(function()
            local VirtualInputManager = game:GetService("VirtualInputManager")
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
            wait(0.05)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
        end)
        
        -- Tentar RemoteEvents de ataque direto (método 5)
        pcall(function()
            for _, remote in pairs(game.ReplicatedStorage:GetDescendants()) do
                if remote:IsA("RemoteEvent") then
                    local name = remote.Name:lower()
                    if name:find("attack") or name:find("hit") or name:find("damage") or name:find("combat") then
                        remote:FireServer(nearestMob)
                        remote:FireServer(nearestMob.HumanoidRootPart)
                        remote:FireServer("attack", nearestMob)
                        print("⚔️ RemoteEvent testado:", remote.Name)
                    end
                end
            end
        end)
        
        return true -- Hit realizado
    else
        print("❌ Nenhum mob encontrado para atacar")
        -- Debug: mostrar por que nenhum mob foi encontrado
        local playersNearby = 0
        local npcsNearby = 0
        local totalEntities = 0
        
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                if obj ~= char and obj.Humanoid.Health > 0 then
                    totalEntities = totalEntities + 1
                    local distance = (rootPart.Position - obj.HumanoidRootPart.Position).Magnitude
                    if distance <= 50 then
                        -- Verificar se é player
                        local isPlayer = false
                        for _, player in pairs(game.Players:GetPlayers()) do
                            if player.Character == obj then
                                isPlayer = true
                                playersNearby = playersNearby + 1
                                break
                            end
                        end
                        if not isPlayer then
                            npcsNearby = npcsNearby + 1
                        end
                    end
                end
            end
        end
        
        if totalEntities > 0 then
            print("🔍 Debug Auto Hit - Entidades próximas (50 studs): Players:", playersNearby, "NPCs:", npcsNearby)
        end
    end
    
    return false -- Nenhum mob encontrado
end

-- Função para testar RemoteEvents específicos encontrados no jogo
local function trySpecificRemotes(mob)
    if not mob or not mob.Parent then return end
    
    print("🎯 Testando RemoteEvents específicos do jogo...")
    
    -- Eventos de Combat baseados no console observado (PRIORIDADE ALTA)
    local combatEvents = {
        "ReplicatedStorage.Systems.Combat.DamageNumber",
        "ReplicatedStorage.Systems.Effects.HitEffect",
        "ReplicatedStorage.Systems.Effects.DoEffect",
        "ReplicatedStorage.Systems.SFX.PlaySFX",
        "ReplicatedStorage.Systems.Combat.HitboxIndicator",
        "ReplicatedStorage.Systems.Combat.AddToHitList",
        "ReplicatedStorage.Systems.Combat.KeyReward",
        "ReplicatedStorage.Systems.Combat.ChestReward", 
        "ReplicatedStorage.Systems.Combat.HitDestructibles",
        "ReplicatedStorage.Systems.Combat.ItemReward",
        "ReplicatedStorage.Systems.Combat.CharacterImpulse"
    }
    
    for _, eventPath in pairs(combatEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("⚔️ Testando evento de combate:", event.Name)
            
            -- Padrões baseados no console observado
            if event.Name == "DamageNumber" then
                pcall(function() event:FireServer(mob, 773, Vector3.new(0,0,0), true) end)
                pcall(function() event:FireServer(mob.HumanoidRootPart, 773, Vector3.new(0,0,0), true) end)
            elseif event.Name == "HitEffect" then
                pcall(function() event:FireServer(mob, Vector3.new(0,0,0), 773) end)
                pcall(function() event:FireServer(mob.HumanoidRootPart, Vector3.new(0,0,0), 773) end)
            elseif event.Name == "DoEffect" then
                pcall(function() event:FireServer(mob, "MagicBoltProjectile", Vector3.new(0,0,0)) end)
                pcall(function() event:FireServer(mob.HumanoidRootPart, "MagicBoltProjectile", Vector3.new(0,0,0)) end)
            elseif event.Name == "PlaySFX" then
                pcall(function() event:FireServer("Scepter_Impact", Vector3.new(0,0,0), 0.1) end)
            else
                -- Métodos padrão para outros eventos
                pcall(function() event:FireServer(mob) end)
                pcall(function() event:FireServer(mob.HumanoidRootPart) end)
                pcall(function() event:FireServer(mob.Name) end)
                pcall(function() event:FireServer("attack", mob) end)
                pcall(function() event:FireServer("damage", mob, 100) end)
            end
        end
    end
    
    -- Eventos de Dungeons
    local dungeonEvents = {
        "ReplicatedStorage.Systems.Dungeons.SpecialDungeons.BigBananaDungeon.ShowRewards",
        "ReplicatedStorage.Systems.Dungeons.SpecialDungeons.FireCultDungeon.ShowRewards",
        "ReplicatedStorage.Systems.Dungeons.SpecialDungeons.BossRushDungeon.ShowRewards",
        "ReplicatedStorage.Systems.Dungeons.ExitDungeon",
        "ReplicatedStorage.Systems.Dungeons.InstantChoice"
    }
    
    for _, eventPath in pairs(dungeonEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("🏰 Testando evento de dungeon:", event.Name)
            pcall(function() event:FireServer(mob) end)
            pcall(function() event:FireServer("target", mob) end)
        end
    end
    
    -- Eventos de Items/Armas
    local itemEvents = {
        "ReplicatedStorage.Systems.Items.InventoryChanged",
        "ReplicatedStorage.Systems.Items.ChestDropped",
        "ReplicatedStorage.Systems.Items.ItemSold",
        "ReplicatedStorage.Systems.Items.InfuseRaidItem"
    }
    
    for _, eventPath in pairs(itemEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("🎒 Testando evento de item:", event.Name)
            pcall(function() event:FireServer("use", mob) end)
            pcall(function() event:FireServer("activate", mob) end)
        end
    end
    
    -- Eventos de PvP (podem funcionar em mobs)
    local pvpEvents = {
        "ReplicatedStorage.Systems.PVP.TogglePVP",
        "ReplicatedStorage.Systems.PVP.PKTimeLeft"
    }
    
    for _, eventPath in pairs(pvpEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("⚔️ Testando evento PvP:", event.Name)
            pcall(function() event:FireServer(mob) end)
            pcall(function() event:FireServer(true, mob) end)
        end
    end
    
    -- Eventos de Party (podem ter ataques em grupo)
    local partyEvents = {
        "ReplicatedStorage.Systems.Party.Invited",
        "ReplicatedStorage.Systems.Party.KickPlayer",
        "ReplicatedStorage.Systems.Party.ListenToParty",
        "ReplicatedStorage.Systems.Party.AcceptInvite"
    }
    
    for _, eventPath in pairs(partyEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("👥 Testando evento de party:", event.Name)
            pcall(function() event:FireServer("attack", mob) end)
            pcall(function() event:FireServer(mob) end)
        end
    end
    
    -- Eventos de Pets (pets podem atacar)
    local petEvents = {
        "ReplicatedStorage.Systems.Pets.SetNickname",
        "ReplicatedStorage.Systems.Pets.AcceptRarity",
        "ReplicatedStorage.Systems.Pets.PetEquipped",
        "ReplicatedStorage.Systems.Pets.PetAction"
    }
    
    for _, eventPath in pairs(petEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("🐾 Testando evento de pet:", event.Name)
            pcall(function() event:FireServer("attack", mob) end)
            pcall(function() event:FireServer("target", mob) end)
            pcall(function() event:FireServer(mob) end)
        end
    end
    
    -- Eventos de Teleport baseados no console observado
    local teleportEvents = {
        "ReplicatedStorage.Systems.Teleport.TeleportPlayer",
        "ReplicatedStorage.Systems.Teleport.TowerTeleport",
        "ReplicatedStorage.Systems.Teleport.DungeonTeleport"
    }
    
    for _, eventPath in pairs(teleportEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("🌀 Testando evento de teleport:", event.Name)
            -- Usar padrão observado no console: apenas userdata
            pcall(function() event:FireServer(mob) end)
            pcall(function() event:FireServer(mob.HumanoidRootPart) end)
            pcall(function() event:FireServer(mob.Position) end)
            pcall(function() event:FireServer(mob.HumanoidRootPart.Position) end)
        end
    end
    
    -- Eventos genéricos que podem existir
    local genericEvents = {
        "ReplicatedStorage.Systems.Action",
        "ReplicatedStorage.Systems.Skill",
        "ReplicatedStorage.Systems.Attack",
        "ReplicatedStorage.Systems.Damage",
        "ReplicatedStorage.Systems.Hit",
        "ReplicatedStorage.Systems.Target",
        "ReplicatedStorage.Systems.Use",
        "ReplicatedStorage.Systems.Activate"
    }
    
    for _, eventPath in pairs(genericEvents) do
        local event = game
        for part in eventPath:gmatch("[^.]+") do
            event = event:FindFirstChild(part)
            if not event then break end
        end
        
        if event and event:IsA("RemoteEvent") then
            print("🎮 Testando evento genérico:", event.Name)
            pcall(function() event:FireServer(mob) end)
            pcall(function() event:FireServer(mob.HumanoidRootPart) end)
            pcall(function() event:FireServer(mob.Name) end)
        end
    end
end

-- TEST AUTO ATTACK FUNCTIONS COM DEBUGGING
local function tryAttackMethods(mob)
    if not mob or not mob.Parent then
        print("Mob inválido ou foi removido")
        return
    end
    
    print("Tentando atacar mob:", mob.Name)
    local char = LocalPlayer.Character
    
    -- Primeiro, testar os RemoteEvents específicos encontrados
    trySpecificRemotes(mob)
    
    -- 1: Tentar tocar fisicamente o mob (alguns jogos aceitam)
    if char and char:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("HumanoidRootPart") then
        print("Teleportando para o mob...")
        pcall(function()
            char.HumanoidRootPart.CFrame = mob.HumanoidRootPart.CFrame + Vector3.new(0,2,0)
        end)
    end

    -- 2: Tentar RemoteEvents comuns em ReplicatedStorage
    print("Testando remotes com 'attack'...")
    for _,v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and v.Name:lower():find("attack") then
            print("Tentando FireServer em:", v:GetFullName())
            pcall(function()
                v:FireServer(mob)
            end)
            pcall(function()
                v:FireServer(mob.HumanoidRootPart)
            end)
            pcall(function()
                v:FireServer(mob.Name)
            end)
            pcall(function()
                v:FireServer()
            end)
        elseif v:IsA("RemoteFunction") and v.Name:lower():find("attack") then
            print("Tentando InvokeServer em:", v:GetFullName())
            pcall(function()
                v:InvokeServer(mob)
            end)
            pcall(function()
                v:InvokeServer(mob.HumanoidRootPart)
            end)
            pcall(function()
                v:InvokeServer(mob.Name)
            end)
            pcall(function()
                v:InvokeServer()
            end)
        end
    end

    -- 3: Tentar eventos genéricos ("Damage", "Hit", "Skill", "Combat", "Fight", "Kill")
    print("Testando remotes genéricos...")
    local keywords = {"damage", "hit", "skill", "combat", "fight", "kill", "mob", "enemy", "monster"}
    for _,v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") then
            for _, keyword in pairs(keywords) do
                if v.Name:lower():find(keyword) then
                    print("Tentando remote genérico:", v:GetFullName())
                    pcall(function()
                        v:FireServer(mob)
                    end)
                    pcall(function()
                        v:FireServer(mob.HumanoidRootPart)
                    end)
                    pcall(function()
                        v:FireServer(mob.Name)
                    end)
                    pcall(function()
                        v:FireServer()
                    end)
                    break
                end
            end
        end
    end

    -- 4: Procurar por eventos em StarterGui ou outras pastas comuns
    print("Testando remotes em outras pastas...")
    for _,service in pairs({game:GetService("StarterGui"), game:GetService("StarterPack")}) do
        for _,v in pairs(service:GetDescendants()) do
            if v:IsA("RemoteEvent") and v.Name:lower():find("attack") then
                print("Tentando remote em", service.Name, ":", v:GetFullName())
                pcall(function()
                    v:FireServer(mob)
                end)
                pcall(function()
                    v:FireServer(mob.HumanoidRootPart)
                end)
                pcall(function()
                    v:FireServer(mob.Name)
                end)
                pcall(function()
                    v:FireServer()
                end)
            end
        end
    end

    -- 5: Ativar ferramentas equipadas (para jogos que usam Tool:Activate)
    local tool = char and char:FindFirstChildOfClass("Tool")
    if tool then
        print("Ativando ferramenta:", tool.Name)
        pcall(function()
            tool:Activate()
        end)
    else
        print("Nenhuma ferramenta equipada")
    end
    
    -- 6: Tentar simular clique do mouse
    print("Simulando clique do mouse...")
    pcall(function()
        game:GetService("VirtualUser"):CaptureController()
        game:GetService("VirtualUser"):Button1Down(Vector2.new(1280, 672))
        wait(0.1)
        game:GetService("VirtualUser"):Button1Up(Vector2.new(1280, 672))
    end)
    
    -- 7: Tentar usar UserInputService
    print("Tentando UserInputService...")
    pcall(function()
        local UserInputService = game:GetService("UserInputService")
        UserInputService:GetFocusedTextBox()
    end)
end

-- Função melhorada para encontrar mobs
local function findMobs()
    local mobs = {}
    print("=== BUSCANDO MOBS ===")
    
    -- Buscar em workspace
    print("Buscando em workspace...")
    for _, obj in pairs(workspace:GetChildren()) do
        pcall(function()
            if obj and obj.Parent and obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") and obj ~= LocalPlayer.Character then
                if obj.Humanoid.Health > 0 then
                    table.insert(mobs, obj)
                    print("Mob encontrado:", obj.Name, "Health:", obj.Humanoid.Health)
                end
            end
        end)
    end
    
    -- Buscar em pastas específicas comuns
    local commonFolders = {"DungeonMobs", "Enemies", "Mobs", "NPCs", "Monsters", "Characters"}
    for _, folderName in pairs(commonFolders) do
        local folder = workspace:FindFirstChild(folderName)
        if folder then
            print("Buscando em", folderName, "...")
            for _, obj in pairs(folder:GetChildren()) do
                pcall(function()
                    if obj and obj.Parent and obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                        if obj.Humanoid.Health > 0 then
                            table.insert(mobs, obj)
                            print("Mob encontrado em", folderName, ":", obj.Name, "Health:", obj.Humanoid.Health)
                        end
                    end
                end)
            end
        end
    end
    
    print("Total de mobs encontrados:", #mobs)
    return mobs
end

-- Função de Auto Kill melhorada
local function autoAttack()
    while autoKill do
        pcall(function()
            local mobs = findMobs()
            
            if #mobs > 0 then
                for _, mob in pairs(mobs) do
                    if mob and mob.Parent and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                        print("\n--- ATACANDO:", mob.Name, "---")
                        tryAttackMethods(mob)
                        wait(0.5) -- Dar tempo para o ataque processar
                        break -- Atacar um mob por vez
                    end
                end
            else
                print("Nenhum mob encontrado, aguardando...")
            end
        end)
        
        wait(1) -- Aguardar antes da próxima busca
    end
end

-- Botão do Auto Hit
autoHitBtn.MouseButton1Click:Connect(function()
    autoHitActive = not autoHitActive
    autoHitBtn.Text = "Auto Hit: " .. (autoHitActive and "ON" or "OFF")
    autoHitBtn.BackgroundColor3 = autoHitActive and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    
    if autoHitActive then
        print("🎯 Auto Hit ativado! Sistema de cliques automáticos iniciado.")
        print("⏱️ Delay entre hits configurado para:", hitDelay, "segundos")
        autoHitConnection = game:GetService("RunService").Heartbeat:Connect(function()
            pcall(function()
                print("🔄 Loop Auto Hit executando...")
                if autoHit() then
                    print("✅ Hit realizado, aguardando", hitDelay, "segundos")
                    wait(hitDelay) -- Delay entre hits
                else
                    print("❌ Nenhum hit realizado neste ciclo")
                end
            end)
        end)
    else
        print("❌ Auto Hit desativado!")
        if autoHitConnection then
            autoHitConnection:Disconnect()
            autoHitConnection = nil
        end
        
        -- Desativar flutuação automaticamente quando Auto Hit for desligado
        if floatEnabled then
            floatEnabled = false
            floatBtn.Text = "Flutuar: OFF"
            floatBtn.BackgroundColor3 = Color3.fromRGB(44,44,60)
            
            -- Parar flutuação
            if floatConnection then
                floatConnection:Disconnect()
                floatConnection = nil
            end
            
            -- Remover BodyPosition
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local bodyPosition = char.HumanoidRootPart:FindFirstChild("BodyPosition")
                if bodyPosition then
                    bodyPosition:Destroy()
                end
            end
            
            print("🌟 Flutuação desativada automaticamente")
        end
    end
end)

-- Botão de Flutuar
floatBtn.MouseButton1Click:Connect(function()
    floatEnabled = not floatEnabled
    floatBtn.Text = "Flutuar: " .. (floatEnabled and "ON" or "OFF")
    floatBtn.BackgroundColor3 = floatEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(44,44,60)
    
    if floatEnabled then
        if not floatConnection then
            floatConnection = RunService.Heartbeat:Connect(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local bodyPosition = hrp:FindFirstChild("BodyPosition")
                    if not bodyPosition then
                        bodyPosition = Instance.new("BodyPosition")
                        bodyPosition.MaxForce = Vector3.new(4000, 4000, 4000)
                        bodyPosition.Position = hrp.Position + Vector3.new(0, 5, 0)
                        bodyPosition.Parent = hrp
                    else
                        -- Manter flutuando 5 studs acima da posição atual
                        local currentPos = hrp.Position
                        bodyPosition.Position = Vector3.new(currentPos.X, currentPos.Y + 5, currentPos.Z)
                    end
                end
            end)
        end
    else
        if floatConnection then
            floatConnection:Disconnect()
            floatConnection = nil
        end
        
        -- Remover BodyPosition quando desativar flutuação
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local bodyPosition = char.HumanoidRootPart:FindFirstChild("BodyPosition")
            if bodyPosition then
                bodyPosition:Destroy()
            end
        end
    end
end)

-- Botão para listar remotes manualmente
debugBtn.MouseButton1Click:Connect(function()
    print("\n=== LISTANDO REMOTES MANUALMENTE ===")
    printAllRemotes()
end)

-- Botão para buscar mobs manualmente
mobBtn.MouseButton1Click:Connect(function()
    print("\n=== BUSCANDO MOBS MANUALMENTE ===")
    local mobs = findMobs()
    if #mobs > 0 then
        print("Mobs encontrados:")
        for i, mob in pairs(mobs) do
            print(i .. ".", mob.Name, "- Health:", mob.Humanoid.Health, "- Position:", tostring(mob.HumanoidRootPart.Position))
        end
    else
        print("Nenhum mob encontrado!")
    end
end)

print("=== DUNGEON HEROES AUTO KILL SCRIPT CARREGADO ===")
print("Versão 3.3 - FLUTUAÇÃO CORRIGIDA E DEBUG MELHORADO")
print("\nBotões disponíveis:")
print("- Auto Hit: Ativa/Desativa ataque automático")
print("- Flutuar: Ativa/Desativa modo de voo")
print("- Listar Remotes: Mostra todos os RemoteEvents encontrados")
print("- Buscar Mobs: Procura por mobs/NPCs no jogo")
print("\nO script irá mostrar informações detalhadas no console F9!")
print("✅ Todas as verificações de segurança foram adicionadas")
print("✅ Erros de table.concat foram corrigidos")
print("✅ Proteções pcall adicionadas em todas as funções críticas")
print("✅ Script carregado com sucesso! Versão 3.2 - Sistema AUTO HIT OTIMIZADO")
print("📋 Use os botões: Auto Hit, Float, Listar Remotes, Buscar Mobs")
print("🔍 Verifique o console F9 para logs de debugging e atividade de rede")
print("🎯 Sistema AUTO HIT: Otimizado baseado em análise de console de script funcional")
print("⚔️ RemoteEvents específicos: DamageNumber, HitEffect, DoEffect, PlaySFX")
print("🗡️ Equipamento automático de armas incluído")
print("🌀 Teleporte automático mais agressivo")
print("")
print("📖 COMO USAR O AUTO HIT:")
print("1. Clique em 'Auto Hit' para ativar/desativar")
print("2. O sistema encontrará APENAS mobs/NPCs automaticamente (alcance: 50 studs)")
print("3. Equipará armas da mochila automaticamente")
print("4. Teleportará automaticamente para mobs (distância: 5 studs)")
print("5. Usará RemoteEvents específicos observados em console funcional")
print("6. Delay otimizado entre hits (atual: 0.05s - mais agressivo)")
print("")
print("🛡️ FILTROS DE DETECÇÃO DE MOBS:")
print("• Exclui todos os players automaticamente")
print("• Detecta NPCs por DisplayName diferente")
print("• Verifica pastas típicas de mobs (npc, mob, enemy, monster)")
print("• Identifica scripts de NPCs")
print("")
print("🔥 MELHORIAS VERSÃO 3.3:")
print("• ✅ FLUTUAÇÃO CORRIGIDA: BodyPosition ao invés de BodyVelocity")
print("• ✅ Posição específica (X,Y,Z) baseada na posição do mob")
print("• ✅ Sistema de debug melhorado com logs detalhados")
print("• ✅ Flutuação a 8 studs de altura para melhor visibilidade")
print("• ✅ Logs de detecção de mobs e estatísticas de busca")
print("• ✅ Correção do loop infinito de flutuação")
print("• RemoteEvents baseados em console de script funcional")
print("• Parâmetros específicos: DamageNumber(mob, 773, vector, boolean)")
print("• HitEffect(mob, vector, 773) e DoEffect(mob, 'MagicBoltProjectile')")
print("• Delay reduzido para 0.05s (20 hits/segundo)")
print("• Distância de ataque reduzida para 5 studs (mais agressivo)")
print("================================================")

--[[
=== INSTRUÇÕES DE USO ===

1. PRIMEIRO: Clique em 'Listar Remotes' para ver todos os RemoteEvents do jogo
2. SEGUNDO: Clique em 'Buscar Mobs' para verificar se o script encontra mobs
3. TERCEIRO: Ative 'Auto Kill' e observe o console F9 para ver:
   - Quais mobs estão sendo encontrados
   - Quais remotes estão sendo testados
   - Se algum ataque está funcionando

=== O QUE OBSERVAR NO CONSOLE F9 ===

- Se aparecer "Mob encontrado: [Nome]", o script está detectando mobs corretamente
- Se aparecer "Tentando FireServer em: [Remote]", o script está testando ataques
- Se aparecer "[REMOTE EVENT RECEBIDO]" ou "[FIRE SERVER DETECTADO]", significa que há atividade de rede
- Se algum mob morrer ou receber dano, anote qual remote aparece no log

=== PROBLEMAS COMUNS ===

- Se não aparecer "Mob encontrado": O jogo pode usar uma estrutura diferente para mobs
- Se não aparecer remotes sendo testados: O jogo pode não ter RemoteEvents com nomes comuns
- Se o personagem teleportar mas não causar dano: Precisa descobrir o RemoteEvent correto

=== PRÓXIMOS PASSOS ===

Se você encontrar informações úteis no console F9, envie prints ou copie as mensagens
para que possamos ajustar o script especificamente para este jogo.
]]
