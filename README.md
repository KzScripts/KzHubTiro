-- Biblioteca Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Kz Hub | Aimbot",
    LoadingTitle = "Kz Hub",
    LoadingSubtitle = "by Kz",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "KzHub"
    },
    Discord = {
        Enabled = false
    },
    KeySystem = false,
})

local MainTab = Window:CreateTab("Main", 4483362458)

-- Variáveis gerais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local aimbotEnabled = false
local FOV = 100
local SpeedValue = 16
local JumpPower = 50
local InfJump = false
local ESPEnabled = false

-- Aimbot + FOV círculo
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 0)
fovCircle.Thickness = 2
fovCircle.Radius = FOV
fovCircle.Filled = false
fovCircle.Visible = true
fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

local function GetClosestPlayer()
    local closest, shortestDistance = nil, FOV
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - fovCircle.Position).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closest = player
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    fovCircle.Radius = FOV

    if aimbotEnabled then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, headPos)
        end
    end
end)

-- ESP persistente
local function AddESP(player)
    if not player.Character then return end
    if player.Character:FindFirstChild("PlayerESP") then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "PlayerESP"
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.new(0, 0, 0)
    highlight.FillTransparency = 0.3
    highlight.OutlineTransparency = 0.1
    highlight.Parent = player.Character
end

local function HandleCharacter(player)
    if ESPEnabled then
        AddESP(player)
    end
    player.CharacterAdded:Connect(function()
        if ESPEnabled then
            task.wait(0.5)
            AddESP(player)
        end
    end)
end

local function UpdateESP(enabled)
    ESPEnabled = enabled
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if enabled then
                HandleCharacter(player)
            else
                if player.Character and player.Character:FindFirstChild("PlayerESP") then
                    player.Character.PlayerESP:Destroy()
                end
            end
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if ESPEnabled then
            task.wait(0.5)
            AddESP(player)
        end
    end)
end)

-- Inf Jump
UIS.JumpRequest:Connect(function()
    if InfJump then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

-- UI Controles
MainTab:CreateSection("Aimbot Sistem")

MainTab:CreateToggle({
    Name = "Aimbot Ativar",
    CurrentValue = false,
    Callback = function(Value)
        aimbotEnabled = Value
    end,
})

MainTab:CreateSlider({
    Name = "Tamanho do FOV",
    Range = {50, 500},
    Increment = 5,
    Suffix = "px",
    CurrentValue = FOV,
    Callback = function(Value)
        FOV = Value
    end,
})

MainTab:CreateSection("Player Sistem")

MainTab:CreateSlider({
    Name = "Velocidade (Speed)",
    Range = {16, 300},
    Increment = 1,
    Suffix = "WalkSpeed",
    CurrentValue = SpeedValue,
    Callback = function(Value)
        SpeedValue = Value
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.WalkSpeed = Value end
        end
    end,
})

MainTab:CreateSlider({
    Name = "Altura do Pulo",
    Range = {50, 300},
    Increment = 1,
    Suffix = "JumpPower",
    CurrentValue = JumpPower,
    Callback = function(Value)
        JumpPower = Value
        if LocalPlayer.Character then
            local hum = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if hum then hum.JumpPower = Value end
        end
    end,
})

MainTab:CreateToggle({
    Name = "Pulo Infinito",
    CurrentValue = false,
    Callback = function(Value)
        InfJump = Value
    end,
})

MainTab:CreateToggle({
    Name = "ESP Jogadores",
    CurrentValue = false,
    Callback = function(Value)
        UpdateESP(Value)
    end,
})
