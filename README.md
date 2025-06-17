-- Proteção contra kick
local mt = getrawmetatable(game)
setreadonly(mt, false)
local old = mt.__namecall
mt.__namecall = newcclosure(function(self, ...)
    if getnamecallmethod() == "Kick" then return end
    return old(self, ...)
end)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local lp = Players.LocalPlayer
repeat wait() until lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
local char = lp.Character
local hrp = char:WaitForChild("HumanoidRootPart")
local humanoid = char:WaitForChild("Humanoid")

-- Criação do GUI simples
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "AFKFarmGui"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 140, 0, 60)
frame.Position = UDim2.new(0, 20, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
frame.Active = true
frame.Draggable = true
Instance.new("UICorner", frame)

local toggleBtn = Instance.new("TextButton", frame)
toggleBtn.Size = UDim2.new(1, -10, 1, -10)
toggleBtn.Position = UDim2.new(0, 5, 0, 5)
toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleBtn.TextColor3 = Color3.new(1,1,1)
toggleBtn.Text = "🔴 AFK FARM OFF"
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextScaled = true
Instance.new("UICorner", toggleBtn)

-- Variáveis de controle
local autofarm = false
local bv = Instance.new("BodyVelocity")
bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
bv.P = 12500
bv.Parent = hrp

-- Botão para ligar/desligar
toggleBtn.MouseButton1Click:Connect(function()
    autofarm = not autofarm
    toggleBtn.Text = autofarm and "🟢 AFK FARM ON" or "🔴 AFK FARM OFF"
end)

-- Loop de auto farm
RunService.RenderStepped:Connect(function()
    if autofarm and lp.Character and hrp and humanoid.Health > 0 then
        local closest = nil
        local shortest = math.huge

        for _, player in pairs(Players:GetPlayers()) do
            if player ~= lp and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
                local dist = (player.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                if dist < shortest and player.Character.Humanoid.Health > 0 then
                    closest = player
                    shortest = dist
                end
            end
        end

        if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = closest.Character.HumanoidRootPart.Position
            local direction = (targetPos - hrp.Position).Unit
            bv.Velocity = direction * 100

            if shortest < 25 then
                closest.Character.Humanoid.Health = 0
            end
        else
            bv.Velocity = Vector3.new(0, 10, 0)
        end
    else
        bv.Velocity = Vector3.new(0, 0, 0)
    end
end)
