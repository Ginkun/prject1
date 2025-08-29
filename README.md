-- AUTO HIT para Dungeon Heroes (ataca como se estivesse clicando no mouse)
-- Ajuste o intervalo se quiser hits mais rápidos/lentos

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Troque aqui se descobrir outro nome de remote
local remote = ReplicatedStorage:FindFirstChild("Systems")
    and ReplicatedStorage.Systems:FindFirstChild("Combat")
    and ReplicatedStorage.Systems.Combat:FindFirstChild("PlayerAttack")

local autoHit = false

-- GUI simples
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DH_AutoHitGUI"
local btn = Instance.new("TextButton", gui)
btn.Size = UDim2.new(0, 200, 0, 40)
btn.Position = UDim2.new(0, 10, 0, 60)
btn.Text = "AUTO HIT: OFF"
btn.BackgroundColor3 = Color3.fromRGB(60,60,90)
btn.TextColor3 = Color3.new(1,1,1)
btn.Font = Enum.Font.SourceSansBold
btn.TextSize = 22

btn.MouseButton1Click:Connect(function()
    autoHit = not autoHit
    btn.Text = "AUTO HIT: " .. (autoHit and "ON" or "OFF")
    if autoHit then
        spawn(function()
            while autoHit do
                if remote then
                    -- O argumento pode ser nulo, Character, ou HumanoidRootPart dependendo do jogo
                    -- Teste os três abaixo para ver qual funciona melhor:
                    pcall(function() remote:FireServer() end)
                    -- pcall(function() remote:FireServer(LocalPlayer.Character) end)
                    -- pcall(function() remote:FireServer(LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) end)
                end
                wait(0.15) -- ajuste para atacar mais rápido/lento
            end
        end)
    end
end)

print("Auto Hit carregado. Aperte o botão na tela para ligar/desligar.")

--[[
Se não pegar, tente trocar o remote para "_PlayerSkillAttack" ou testar os argumentos comentados acima.
Se ao ativar o botão o personagem atacar igual ao clique normal, está correto!
Se não funcionar, me envie prints do console ao atacar manualmente, assim posso ajustar o script.
]]
