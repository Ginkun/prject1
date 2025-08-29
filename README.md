-- Hook em todos RemoteEvents e RemoteFunctions do jogo para identificar calls de ataque

local function hookRemote(obj)
    if obj:IsA("RemoteEvent") then
        local old; old = hookfunction(obj.FireServer, function(self, ...)
            print("[Hooked RemoteEvent] "..self:GetFullName(), ...)
            return old(self, ...)
        end)
    elseif obj:IsA("RemoteFunction") then
        local old; old = hookfunction(obj.InvokeServer, function(self, ...)
            print("[Hooked RemoteFunction] "..self:GetFullName(), ...)
            return old(self, ...)
        end)
    end
end

-- Hook nos remotes já existentes
for _,v in ipairs(game:GetDescendants()) do
    if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
        pcall(hookRemote, v)
    end
end

-- Hook nos remotes que forem adicionados posteriormente
game.DescendantAdded:Connect(function(obj)
    if obj:IsA("RemoteEvent") or obj:IsA("RemoteFunction") then
        pcall(hookRemote, obj)
    end
end)

print("Hook ativo! Bata manualmente nos monstros e veja o nome do Remote no console.")
