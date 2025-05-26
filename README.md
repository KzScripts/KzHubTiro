-- Carregar Rayfield UI
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Kz Hub | Fps",
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

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local VIM = game:GetService("VirtualInputManager")

-- Variáveis
local aimbotEnabled = false
local FOV = 100
local SpeedValue = 16
local JumpPower = 50
local InfJump = false
local ESPEnabled = false

-- Anti-AFK
local AntiAFK = true
spawn(function()
    while AntiAFK do
        VIM:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
        wait(60)
    end
end)

-- Círculo FOV
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
            local newCF = CFrame.new(Camera.CFrame.Position, headPos)
            Camera.CFrame = Camera.CFrame:Lerp(newCF, 0.08)
        end
    end
end)

-- ESP
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
    if ESPEnabled then AddESP(player) end
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

-- Pulo Infinito
UIS.JumpRequest:Connect(function()
    if InfJump then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

-- Interface
MainTab:CreateSection("Aimbot")

MainTab:CreateToggle({
    Name = "Ativar Aimbot",
    CurrentValue = false,
    Callback = function(Value)
        aimbotEnabled = Value
    end,
})

MainTab:CreateSlider({
    Name = "FOV (Tamanho do Círculo)",
    Range = {50, 500},
    Increment = 5,
    Suffix = "px",
    CurrentValue = FOV,
    Callback = function(Value)
        FOV = Value
    end,
})

MainTab:CreateSection("Jogador")

MainTab:CreateSlider({
    Name = "Velocidade",
    Range = {16, 300},
    Increment = 1,
    Suffix = "WalkSpeed",
    CurrentValue = SpeedValue,
    Callback = function(Value)
        SpeedValue = Value
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = Value end
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
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.JumpPower = Value end
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

MainTab:CreateButton({
    Name = "FPS Boost",
    Callback = function()
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.Material = Enum.Material.SmoothPlastic
                obj.Reflectance = 0
            elseif obj:IsA("Decal") then
                obj.Transparency = 1
            end
        end
        sethiddenproperty(game:GetService("Lighting"), "Technology", Enum.Technology.Compatibility)
        game:GetService("Lighting").FogEnd = 9e9
        game:GetService("Lighting").GlobalShadows = false
        game:GetService("Lighting").Brightness = 0
    end,
})
