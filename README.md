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
            v.OnClientEvent:Connect(function(...)
                local args = {...}
                print("[REMOTE EVENT RECEBIDO]", v:GetFullName(), "Args:", table.concat(args, ", "))
            end)
        end
    end
    
    -- Hook para capturar FireServer calls
    local oldFireServer = game.ReplicatedStorage.RemoteEvent.FireServer
    game.ReplicatedStorage.RemoteEvent.FireServer = function(self, ...)
        local args = {...}
        print("[FIRE SERVER DETECTADO]", self:GetFullName(), "Args:", table.concat(args, ", "))
        return oldFireServer(self, ...)
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

local autoKillBtn = createButton("Auto Kill: OFF", 10)
local floatBtn    = createButton("Flutuar: OFF", 54)
local debugBtn    = createButton("Listar Remotes", 98)
local mobBtn      = createButton("Buscar Mobs", 142)

local floatY = 50
local floatEnabled = false
local autoKill = false
local autoKillConnection, floatConnection

-- TEST AUTO ATTACK FUNCTIONS COM DEBUGGING
local function tryAttackMethods(mob)
    print("Tentando atacar mob:", mob.Name)
    local char = LocalPlayer.Character
    
    -- 1: Tentar tocar fisicamente o mob (alguns jogos aceitam)
    if char and char:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("HumanoidRootPart") then
        print("Teleportando para o mob...")
        char.HumanoidRootPart.CFrame = mob.HumanoidRootPart.CFrame + Vector3.new(0,2,0)
    end

    -- 2: Tentar RemoteEvents comuns em ReplicatedStorage
    print("Testando remotes com 'attack'...")
    for _,v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and v.Name:lower():find("attack") then
            print("Tentando FireServer em:", v:GetFullName())
            pcall(function()
                v:FireServer(mob)
                v:FireServer(mob.HumanoidRootPart)
                v:FireServer(mob.Name)
                v:FireServer()
            end)
        elseif v:IsA("RemoteFunction") and v.Name:lower():find("attack") then
            print("Tentando InvokeServer em:", v:GetFullName())
            pcall(function()
                v:InvokeServer(mob)
                v:InvokeServer(mob.HumanoidRootPart)
                v:InvokeServer(mob.Name)
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
                        v:FireServer(mob.HumanoidRootPart)
                        v:FireServer(mob.Name)
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
                    v:FireServer(mob.HumanoidRootPart)
                    v:FireServer(mob.Name)
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
        if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") and obj ~= LocalPlayer.Character then
            if obj.Humanoid.Health > 0 then
                table.insert(mobs, obj)
                print("Mob encontrado:", obj.Name, "Health:", obj.Humanoid.Health)
            end
        end
    end
    
    -- Buscar em pastas específicas comuns
    local commonFolders = {"Enemies", "Mobs", "NPCs", "Monsters", "Characters"}
    for _, folderName in pairs(commonFolders) do
        local folder = workspace:FindFirstChild(folderName)
        if folder then
            print("Buscando em", folderName, "...")
            for _, obj in pairs(folder:GetChildren()) do
                if obj:IsA("Model") and obj:FindFirstChild("Humanoid") and obj:FindFirstChild("HumanoidRootPart") then
                    if obj.Humanoid.Health > 0 then
                        table.insert(mobs, obj)
                        print("Mob encontrado em", folderName, ":", obj.Name, "Health:", obj.Humanoid.Health)
                    end
                end
            end
        end
    end
    
    print("Total de mobs encontrados:", #mobs)
    return mobs
end

-- Função de Auto Kill melhorada
local function autoAttack()
    while autoKill do
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
        
        wait(1) -- Aguardar antes da próxima busca
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
print("Versão com debugging avançado")
print("\nBotões disponíveis:")
print("- Auto Kill: Ativa/Desativa ataque automático")
print("- Flutuar: Ativa/Desativa modo de voo")
print("- Listar Remotes: Mostra todos os RemoteEvents encontrados")
print("- Buscar Mobs: Procura por mobs/NPCs no jogo")
print("\nO script irá mostrar informações detalhadas no console F9!")
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
