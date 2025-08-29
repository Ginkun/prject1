--[[
    DUNGEON HEROES SCRIPT - SISTEMA DE PERSISTÊNCIA AUTOMÁTICA
    
    Este script possui um sistema avançado de persistência que mantém o script ativo
    mesmo quando você muda de mundo/servidor. Para usar:
    
    1. Substitua "YOUR_SCRIPT_URL_HERE" pela URL real do seu script (linha ~690)
    2. O script irá automaticamente se reinjetar quando necessário
    3. Suas configurações serão salvas automaticamente antes de teleportes
    
    Recursos de persistência:
    - Detecção automática de mudança de mundo
    - Reinjeção automática após teleporte
    - Proteção contra múltiplas instâncias
    - Monitor contínuo para garantir funcionamento
    - Sistema de backup em caso de falhas
--]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local player = Players.LocalPlayer

local blur = Instance.new("BlurEffect", Lighting)
blur.Size = 0
TweenService:Create(blur, TweenInfo.new(0.5), {Size = 24}):Play()

local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "DungeonHeroesLoader"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(1, 0, 1, 0)
frame.BackgroundTransparency = 1

local bg = Instance.new("Frame", frame)
bg.Size = UDim2.new(1, 0, 1, 0)
bg.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
bg.BackgroundTransparency = 1
bg.ZIndex = 0
TweenService:Create(bg, TweenInfo.new(0.5), {BackgroundTransparency = 0.3}):Play()

local word = "STELLAR"
local letters = {}

local function tweenOutAndDestroy()
	for _, label in ipairs(letters) do
		TweenService:Create(label, TweenInfo.new(0.3), {TextTransparency = 1, TextSize = 20}):Play()
	end
	TweenService:Create(bg, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
	TweenService:Create(blur, TweenInfo.new(0.5), {Size = 0}):Play()
	wait(0.6)
	screenGui:Destroy()
	blur:Destroy()
end

for i = 1, #word do
	local char = word:sub(i, i)

	local label = Instance.new("TextLabel")
	label.Text = char
	label.Font = Enum.Font.GothamBlack
	label.TextColor3 = Color3.new(1, 1, 1)
	label.TextStrokeTransparency = 1 
	label.TextTransparency = 1
	label.TextScaled = false
	label.TextSize = 30 
	label.Size = UDim2.new(0, 60, 0, 60)
	label.AnchorPoint = Vector2.new(0.5, 0.5)
	label.Position = UDim2.new(0.5, (i - (#word / 2 + 0.5)) * 65, 0.5, 0)
	label.BackgroundTransparency = 1
	label.Parent = frame

	local gradient = Instance.new("UIGradient")
	gradient.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 170, 255)), -- biru muda cerah
		ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 100, 160))   -- biru muda gelap
	})
	gradient.Rotation = 90
	gradient.Parent = label

	local tweenIn = TweenService:Create(label, TweenInfo.new(0.3), {TextTransparency = 0, TextSize = 60})
	tweenIn:Play()

	table.insert(letters, label)
	wait(0.25)
end

wait(2)

tweenOutAndDestroy()
repeat task.wait() until game.Players.LocalPlayer and game.Players.LocalPlayer.Character

if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- Proteção contra múltiplas instâncias do script
local function checkExistingScript()
    local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    local existingGui = playerGui:FindFirstChild("DungeonHeroesHubMini")
    
    if existingGui then
        -- Se já existe uma instância, remove a antiga
        existingGui:Destroy()
        task.wait(0.5)
    end
end

checkExistingScript()

local Fluent = loadstring(game:HttpGet("https://github.com/ActualMasterOogway/Fluent-Renewed/releases/latest/download/Fluent.luau"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/x2zu/OPEN-SOURCE-UI-ROBLOX/refs/heads/main/X2ZU%20UI%20ROBLOX%20OPEN%20SOURCE/SaveManager.luau"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/x2zu/OPEN-SOURCE-UI-ROBLOX/refs/heads/main/X2ZU%20UI%20ROBLOX%20OPEN%20SOURCE/InterfaceManager.luau"))()

local Window = Fluent:CreateWindow({
    Title = game:GetService("MarketplaceService"):GetProductInfo(94845773826960).Name .. " 〢 Stellar",
    SubTitle = "discord.gg/FmMuvkaWvG",
    TabWidth = 160,
    Size = UDim2.fromOffset(520, 400),
    Acrylic = false,
    Theme = "Viow Arabian Mix",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "DungeonHeroesHubMini"
gui.ResetOnSpawn = false

local icon = Instance.new("ImageButton")
icon.Name = "DungeonHeroesIcon"
icon.Size = UDim2.new(0, 55, 0, 50)
icon.Position = UDim2.new(0, 200, 0, 150)
icon.BackgroundTransparency = 1
icon.Image = "rbxassetid://105059922903197" -- replace with your real asset ID
icon.Parent = gui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8) -- You can tweak the '8' for more or less rounding
corner.Parent = icon

local dragging, dragInput, dragStart, startPos

icon.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = icon.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

icon.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
	if dragging and input == dragInput then
		local delta = input.Position - dragStart
		icon.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

local isMinimized = false
icon.MouseButton1Click:Connect(function()
	isMinimized = not isMinimized
	Window:Minimize(isMinimized)
end)

--Fluent provides Lucide Icons https://lucide.dev/icons/ for the tabs, icons are optional
local Tabs = {
    DevUpd = Window:AddTab({ Title = "Informações", Icon = "circle-alert"}),
    Main = Window:AddTab({ Title = "Farm Automático", Icon = "star" }),
    Sell = Window:AddTab({ Title = "Vender", Icon = "dollar-sign" }),
    Dungeon = Window:AddTab({ Title = "Lobby", Icon = "play" }),
    AntiAfk = Window:AddTab({ Title = "Anti-Afk", Icon = "clock" }),
    Settings = Window:AddTab({ Title = "Configurações", Icon = "settings" })
}

local Options = Fluent.Options

do
    Tabs.DevUpd:CreateParagraph("Aligned Paragraph", {
    Title = "Dungeon Heroes Script",
    Content = "Obrigado por usar o script! Entre no discord se tiver problemas ou sugestões com o script",
    TitleAlignment = "Middle",
    ContentAlignment = Enum.TextXAlignment.Center
})
    Tabs.DevUpd:CreateParagraph("Aligned Paragraph", {
    Title = "Informações",
    Content = "Se você encontrou este script exigindo uma chave, não é a versão oficial. Entre no nosso Discord para obter a versão sem chave!",
    TitleAlignment = "Middle",
    ContentAlignment = Enum.TextXAlignment.Center
})

    Tabs.DevUpd:AddSection("Discord")
    Tabs.DevUpd:AddButton({
        Title = "Discord",
        Description = "Copie o link para entrar no discord!",
        Callback = function()
            setclipboard("https://discord.gg/FmMuvkaWvG")
            Fluent:Notify({
                Title = "Notificação",
                Content = "Link copiado para a área de transferência com sucesso!",
                SubContent = "", -- Optional
                Duration = 3 
            })
        end
    })

    local Toggle = Tabs.Main:AddToggle("KillAura", {
        Title = "Aura de Morte",  
        Default = false 
    })

    local killAuraRunning = false

    Toggle:OnChanged(function(value)
        if value then
            killAuraRunning = true

            task.spawn(function()
                local replicated_storage = cloneref(game:GetService("ReplicatedStorage"))
                local workspace = cloneref(game:GetService("Workspace"))
                local enemies = workspace:FindFirstChild("Mobs")

                local delay = 0.3

                while killAuraRunning do
                    if enemies then
                        local mobs = {}
                        for _, v in ipairs(enemies:GetChildren()) do
                            table.insert(mobs, v)
                        end

                        replicated_storage:WaitForChild("Systems")
                            :WaitForChild("Combat")
                            :WaitForChild("PlayerAttack")
                            :FireServer(mobs)
                    end

                    task.wait(delay)
                end
            end)

        else
            killAuraRunning = false
        end
    end)

    local Toggle2 = Tabs.Main:AddToggle("AutoStart", {Title = "Início Automático", Default = false })

    Toggle2:OnChanged(function()
        while Options.AutoStart.Value do
            game:GetService("ReplicatedStorage"):WaitForChild("Systems"):WaitForChild("Dungeons"):WaitForChild("TriggerStartDungeon"):FireServer()
            wait(0.1)
        end
    end)

    Options.AutoStart:SetValue(false)

    local Toggle4 = Tabs.Main:AddToggle("AutoPlayAgain", {Title = "Jogar Novamente", Default = false })

    Toggle4:OnChanged(function()
        while Options.AutoPlayAgain.Value do
            local args = {
                "GoAgain"
            }
            game:GetService("ReplicatedStorage"):WaitForChild("Systems"):WaitForChild("Dungeons"):WaitForChild("SetExitChoice"):FireServer(unpack(args))
            wait(0.1)
        end
    end)

    Options.AutoPlayAgain:SetValue(false)
    
    local goto_closest = false
    
    local replicated_storage = cloneref(game:GetService('ReplicatedStorage'))
    local user_input_service = cloneref(game:GetService('UserInputService'))
    local local_player = cloneref(game:GetService('Players').LocalPlayer)
    local tween_service = cloneref(game:GetService('TweenService'))
    local run_service = cloneref(game:GetService('RunService'))
    local workspace = cloneref(game:GetService('Workspace'))

    function closest_mob()
        local mob = nil
        local distance = math.huge
        local enemies = workspace:FindFirstChild("Mobs")

        for _, v in next, enemies:GetChildren() do
            if not v:GetAttribute("Owner") and v:GetAttribute("HP") > 0 and v.Name ~= "Side Room Rune Disabled" and v.Name ~= "TargetDummy" then
                local dist = (v:GetPivot().Position - local_player.Character:GetPivot().Position).Magnitude
                if dist < distance then
                    distance = dist
                    mob = v
                end
            end
        end

        return mob
    end
    local y = 50
    local tweenspeed = 200
    local Toggle3 = Tabs.Main:AddToggle("AutoFarm", {
        Title = "Farm Automático de Dungeon",
        Default = false
    })

    Toggle3:OnChanged(function(Value)
        goto_closest = Value
        if Value then
            repeat
                local mob = closest_mob()
                if mob then
                    task.wait(.1)
                    local velocity_connection = run_service.Heartbeat:Connect(function()
                        if local_player.Character and local_player.Character:FindFirstChild("HumanoidRootPart") then
                            local_player.Character.HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
                            local_player.Character.HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
                        end
                    end)
                    local character = local_player.Character
                    local hrp = character and character:FindFirstChild("HumanoidRootPart")
                    if hrp and mob then
                        local to = mob:GetPivot().Position
                        local distance = (to - hrp.Position).Magnitude
                        local tween = tween_service:Create(hrp, TweenInfo.new(distance / tweenspeed, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {
                            CFrame = CFrame.new(to + Vector3.new(0, y, 0))
                        })
                        tween:Play()
                        tween.Completed:Wait()
                    end
                    if velocity_connection then
                        velocity_connection:Disconnect()
                    end
                end
                task.wait()
            until not goto_closest
        end
    end)

    local tweenspeedslider = Tabs.Main:AddSlider("tweenspeedslider", {
        Title = "Velocidade de Movimento",
        Description = "Ajuste até não ser expulso do jogo",
        Default = 200,
        Min = 20,
        Max = 300,
        Rounding = 0.1,
        Callback = function(Value)
            tweenspeed = Value
        end
    })

    tweenspeedslider:SetValue(200)
    
    local Distance = Tabs.Main:AddSlider("Distance", {
        Title = "Distância Y dos mobs",
        Default = 50,
        Min = -100,
        Max = 100,
        Rounding = 1,
        Callback = function(Value)
            y = Value
        end
    })

    Distance:SetValue(50)
    

    local selected_dungeon = "AstralDungeon"
    local dungeons = Tabs.Dungeon:AddDropdown("dungeons", {
        Title = "Selecionar Dungeon",
        Values = {"AstralDungeon", "CastleDungeon", "CoveDungeon", "DesertDungeon", "ForestDungeon", "JungleDungeon", "MountainDungeon", "CaveDungeon", "MushroomDungeon"},
        Multi = false,
        Default = 1,
    })

    dungeons:SetValue("AstralDungeon")

    dungeons:OnChanged(function(Value)
        selected_dungeon = Value
    end)

    local selected_difficulties = 1

    local difficulties = Tabs.Dungeon:AddDropdown("difficulties", {
        Title = "Escolher Dificuldade",
        Values = {"Normal", "Medium", "Hard", "Insane"},
        Multi = false,
        Default = 1,
    })

    difficulties:SetValue("Normal") -- Set default to a valid option

    difficulties:OnChanged(function(Value)
        if Value == "Normal" then
            selected_difficulties = 1 
        elseif Value == "Medium" then
            selected_difficulties = 2
        elseif Value == "Hard" then
            selected_difficulties = 3
        elseif Value == "Insane" then
            selected_difficulties = 4
        end
    end)

    local selected_player = 1
    local players = Tabs.Dungeon:AddDropdown("players", {
        Title = "Jogadores",
        Values = {"1", "2", "3", "4", "5"},
        Multi = false,
        Default = 1,
    })

    players:SetValue("1")

    players:OnChanged(function(Value)
        if Value == "1" then
            selected_player = 1 
        elseif Value == "2" then
            selected_player = 2
        elseif Value == "3" then
            selected_player = 3
        elseif Value == "4" then
            selected_player = 4
        elseif Value == "5" then
            selected_player = 5
        end
    end)

    Tabs.Dungeon:AddButton({
        Title = "Entrar na Dungeon",
        Callback = function()
            local args = {
                selected_dungeon,
                selected_difficulties,
                selected_player,
                false
            }
            game:GetService("ReplicatedStorage"):WaitForChild("Systems"):WaitForChild("Parties"):WaitForChild("SetSettings"):FireServer(unpack(args))
        end
    })

    Tabs.Dungeon:AddButton({
        Title = "Voltar ao Lobby",
        Callback = function()
            game:GetService("ReplicatedStorage").Systems.Dungeons.ExitDungeon:FireServer()
        end
    })

    -- Auto Anti-Afk
    local Toggle4 = Tabs.AntiAfk:AddToggle("AntiAfk", {
        Title = "Anti-Afk", 
        Description = "Isso impedirá que você seja expulso quando estiver AFK", 
        Default = false 
    })


    Toggle4:OnChanged(function()
        task.spawn(function()
            while Options.AntiAfk.Value do
                -- Simulate player activity to prevent AFK kick
                local VirtualUser = game:GetService("VirtualUser")
                
                -- Move the mouse slightly to simulate activity
                VirtualUser:CaptureController()
                VirtualUser:ClickButton2(Vector2.new())
                
                print("Anti-AFK activated")
                task.wait(10)
            end
        end)
    end)
    Options.AntiAfk:SetValue(false)
    
    -- Rarity mapping: name to string value
    local rarityMap = {
        Common = "1",
        Uncommon = "2",
        Rare = "3",
        Epic = "4",
        Legendary = "5",
        Mythic = "6",
    }

    -- Selected rarities (string numbers like "1", "2", etc.)
    local selectrarity = {}

    -- Rarity dropdown
    local raritymulti = Tabs.Sell:AddDropdown("raritymulti", {
        Title = "Selecionar Raridade do Item",
        Values = {"Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic"},
        Multi = true,
        Default = {"Common", "Uncommon"},
    })

    -- Convert selected names to string rarity values ("1" to "6")
    raritymulti:OnChanged(function(Value)
        local converted = {}
        for RarityName, Selected in pairs(Value) do
            if Selected and rarityMap[RarityName] then
                table.insert(converted, rarityMap[RarityName])
            end
        end
        selectrarity = converted
    end)

    -- Autosell toggle
    local autosellall = Tabs.Sell:AddToggle("autosellall", {
        Title = "Vender Itens",
        Default = false,
    })

    -- Selling logic
    autosellall:OnChanged(function()
        while Options.autosellall.Value do
            local player = game:GetService("Players").LocalPlayer
            local inventory = player:WaitForChild("PlayerGui"):FindFirstChild("Profile"):FindFirstChild("Inventory")

            if inventory then
                local itemsToSell = {}

                for _, item in pairs(inventory:GetChildren()) do
                    local rarity = item:FindFirstChild("Rarity")
                    if rarity and table.find(selectrarity, tostring(rarity.Value)) then
                        table.insert(itemsToSell, item)
                    end
                end

                if #itemsToSell > 0 then
                    local args = {
                        itemsToSell,
                        {}
                    }

                    game:GetService("ReplicatedStorage")
                        :WaitForChild("Systems")
                        :WaitForChild("ItemSelling")
                        :WaitForChild("SellItem")
                        :FireServer(unpack(args)) -- or InvokeServer depending on your game
                end
            end

            task.wait(0.1)
        end
    end)


    local rarityMap = {
        Common = "1",
        Uncommon = "2",
        Rare = "3",
        Epic = "4",
        Legendary = "5",
        Mythic = "6",
    }


    local rarities = {"Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic"}


    local selectrarityPets = {}


    local raritymultiPets = Tabs.Sell:AddDropdown("raritymultiPets", {
        Title = "Selecionar Raridade do Pet",
        Values = rarities,
        Multi = true,
        Default = {"Common", "Uncommon"},
    })

    raritymultiPets:OnChanged(function(Value)
        local Values = {}
        for RarityName, State in next, Value do
            local rarityNum = rarityMap[RarityName]
            if rarityNum then
                table.insert(Values, rarityNum)
            end
        end
        selectrarityPets = Values
    end)

    -- Autosell pets toggle
    local autosellallpet = Tabs.Sell:AddToggle("autosellallpet", {
        Title = "Vender Pets",
        Default = false,
    })

    autosellallpet:OnChanged(function()
        while Options.autosellallpet.Value do
            local player = game:GetService("Players").LocalPlayer
            local petsFolder = player:WaitForChild("PlayerGui"):FindFirstChild("Profile"):FindFirstChild("Pets")

            if petsFolder then
                local petsToSell = {}

                for _, pet in pairs(petsFolder:GetChildren()) do
                    local rarity = pet:FindFirstChild("Rarity")
                    if rarity and table.find(selectrarityPets, tostring(rarity.Value)) then
                        table.insert(petsToSell, pet)
                    end
                end

                if #petsToSell > 0 then
                    local args = {
                        petsToSell,
                        {}
                    }

                    game:GetService("ReplicatedStorage")
                        :WaitForChild("Systems")
                        :WaitForChild("ItemSelling")
                        :WaitForChild("SellItem")
                        :FireServer(unpack(args))
                end
            end

            task.wait(0.1)
        end
    end)

    -- Ensure toggle is off by default
    Options.autosellallpet:SetValue(false)


    local autoopenpetchest = Tabs.Main:AddToggle("autoopenpetchest", {Title = "Abrir Todos os Baús de Pet", Default = false })

    autoopenpetchest:OnChanged(function()
        while Options.autoopenpetchest.Value do
            local inventory = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui"):FindFirstChild("Profile"):FindFirstChild("Inventory")
            if inventory then
                for _, petchest in pairs(inventory:GetChildren()) do
                    local count = petchest:FindFirstChild("Count")
                    if count then
                        local args = {
                            petchest,
                            1
                        }
                        game:GetService("ReplicatedStorage"):WaitForChild("Systems"):WaitForChild("Pets"):WaitForChild("OpenPetChest"):FireServer(unpack(args))
                    end
                end
            end
            task.wait(0.1)
        end
    end)
    
    Options.autoopenpetchest:SetValue(false)

end  

-- Addons:
-- SaveManager (Allows you to have a configuration system)
-- InterfaceManager (Allows you to have a interface managment system)

-- Hand the library over to our managers
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)

-- Ignore keys that are used by ThemeManager.
-- (we dont want configs to save themes, do we?)

SaveManager:IgnoreThemeSettings()
-- You can add indexes of elements the save manager should ignore
SaveManager:SetIgnoreIndexes({})

-- use case for doing it this way:
-- a script hub could have themes in a global folder
-- and game configs in a separate folder per game
InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/Dungeon Heroes")



InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)


Window:SelectTab(1)

Fluent:Notify({
    Title = "Dungeon Heroes Script",
    Content = "O script foi carregado com sucesso.",
    Duration = 3
})
task.wait(3)
Fluent:Notify({
    Title = "Dungeon Heroes Script",
    Content = "Entre no discord para mais atualizações e scripts",
    Duration = 8
})
-- Sistema de persistência entre mundos melhorado
local function setupWorldPersistence()
    local currentPlaceId = game.PlaceId
    local scriptUrl = "YOUR_SCRIPT_URL_HERE" -- Substitua pela URL do seu script
    
    -- Salva o script na pasta _G para persistência global
    if not _G.DungeonHeroesScriptLoaded then
        _G.DungeonHeroesScriptLoaded = true
        _G.DungeonHeroesScriptUrl = scriptUrl
    end
    
    -- Detecta mudança de mundo/lugar
    game:GetService("Players").LocalPlayer.OnTeleport:Connect(function(State)
        if State == Enum.TeleportState.Started then
            -- Salva configurações antes do teleporte
            pcall(function()
                SaveManager:Save()
            end)
            
            -- Marca para reinjeção após teleporte
            _G.DungeonHeroesNeedsReinjection = true
        end
    end)
    
    -- Sistema de reinjeção automática após spawn
    game:GetService("Players").LocalPlayer.CharacterAdded:Connect(function(character)
        task.wait(5) -- Aguarda o mundo carregar completamente
        
        -- Verifica se precisa reinjetar
        if _G.DungeonHeroesNeedsReinjection then
            _G.DungeonHeroesNeedsReinjection = false
            
            local existingGui = player:WaitForChild("PlayerGui"):FindFirstChild("DungeonHeroesHubMini")
            if not existingGui and _G.DungeonHeroesScriptUrl then
                -- Reinjeta o script
                pcall(function()
                    loadstring(game:HttpGet(_G.DungeonHeroesScriptUrl))()
                end)
            end
        end
    end)
    
    -- Monitor contínuo para garantir que o script permaneça ativo
    task.spawn(function()
        while _G.DungeonHeroesScriptLoaded do
            task.wait(10)
            
            -- Verifica se a GUI ainda existe
            local playerGui = game:GetService("Players").LocalPlayer:FindFirstChild("PlayerGui")
            if playerGui then
                local existingGui = playerGui:FindFirstChild("DungeonHeroesHubMini")
                
                -- Se a GUI não existe e não estamos em processo de teleporte
                if not existingGui and not _G.DungeonHeroesNeedsReinjection then
                    task.wait(2)
                    
                    -- Verifica novamente após delay
                    existingGui = playerGui:FindFirstChild("DungeonHeroesHubMini")
                    if not existingGui and _G.DungeonHeroesScriptUrl then
                        -- Reinjeta o script
                        pcall(function()
                            loadstring(game:HttpGet(_G.DungeonHeroesScriptUrl))()
                        end)
                    end
                end
            end
        end
    end)
end

-- Inicia o sistema de persistência
setupWorldPersistence()

-- You can use the SaveManager:LoadAutoloadConfig() to load a config
-- which has been marked to be one that auto loads!
SaveManager:LoadAutoloadConfig()
