--// Anti-Ban e Anti-Detected Básico
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- Proteção contra Kick
local function protect()
    if setreadonly then
        setreadonly(getrawmetatable(game), false)
        local mt = getrawmetatable(game)
        local old = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method == "Kick" or method == "kick" then
                return warn("Tentativa de Kick bloqueada.")
            end
            return old(self, ...)
        end)
    end
end
protect()

-- Desativa logs suspeitos
if hookfunction and getconnections then
    for _,v in pairs(getconnections(game:GetService("LogService").MessageOut)) do
        v:Disable()
    end
end

--// Variáveis de Controle
local Speed = 16
local JumpPower = 50
local InfiniteJump = true
local Noclip = true
local SlowFall = true

--// Pulo Infinito Corrigido
local UserInputService = game:GetService("UserInputService")
local canJump = true

UserInputService.JumpRequest:Connect(function()
    if InfiniteJump and canJump and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        local state = humanoid:GetState()

        if state == Enum.HumanoidStateType.Running or state == Enum.HumanoidStateType.Freefall then
            canJump = false
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            task.delay(0.3, function()
                canJump = true
            end)
        end
    end
end)

--// Queda Lenta
RunService.Stepped:Connect(function()
    if SlowFall and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid:GetState() == Enum.HumanoidStateType.Freefall then
            LocalPlayer.Character.HumanoidRootPart.Velocity = Vector3.new(0, -10, 0)
        end
    end
end)

--// Noclip (atravessar paredes)
RunService.Stepped:Connect(function()
    if Noclip and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide == true then
                part.CanCollide = false
            end
        end
    end
end)

--// Comando "+" no chat aumenta a velocidade
LocalPlayer.Chatted:Connect(function(msg)
    if msg == "+" then
        Speed += 1
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = Speed
        end
        game.StarterGui:SetCore("SendNotification", {
            Title = "Velocidade Aumentada";
            Text = "Sua velocidade aumentou para " .. tostring(Speed);
            Duration = 3;
        })
    end
end)

--// Aplica valores ao entrar no jogo
LocalPlayer.CharacterAdded:Connect(function(char)
    wait(1)
    local humanoid = char:WaitForChild("Humanoid")
    humanoid.WalkSpeed = Speed
    humanoid.JumpPower = JumpPower
end)

-- Aplicar já se personagem existir
if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
    LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = Speed
    LocalPlayer.Character:FindFirstChildOfClass("Humanoid").JumpPower = JumpPower
end

print("✅ Script ativado com Anti-Ban, Pulo Infinito Corrigido, Queda Lenta, Noclip e Aumento de Velocidade com '+'.")
