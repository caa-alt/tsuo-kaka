local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = game:GetService("Workspace").CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- 🔍 ESP (Ver jogadores através das paredes)
function CreateESP(Player)
    if Player ~= LocalPlayer then
        local Highlight = Instance.new("Highlight")
        Highlight.Parent = Player.Character
        Highlight.FillColor = Color3.fromRGB(255, 0, 0) -- Vermelho
        Highlight.OutlineColor = Color3.fromRGB(255, 255, 255) -- Branco
        Highlight.FillTransparency = 0.5
        Highlight.OutlineTransparency = 0
    end
end

for _, Player in pairs(Players:GetPlayers()) do
    if Player.Character then
        CreateESP(Player)
    end
end

Players.PlayerAdded:Connect(function(Player)
    Player.CharacterAdded:Connect(function()
        CreateESP(Player)
    end)
end)

-- ✈️ Fly
local Flying = false
local FlySpeed = 5
local BodyGyro, BodyVelocity

function ToggleFly()
    if Flying then
        Flying = false
        if BodyGyro then BodyGyro:Destroy() end
        if BodyVelocity then BodyVelocity:Destroy() end
    else
        Flying = true
        local Character = LocalPlayer.Character
        if Character and Character:FindFirstChild("HumanoidRootPart") then
            BodyGyro = Instance.new("BodyGyro")
            BodyGyro.Parent = Character.HumanoidRootPart
            BodyGyro.MaxTorque = Vector3.new(400000, 400000, 400000)
            BodyGyro.CFrame = Character.HumanoidRootPart.CFrame

            BodyVelocity = Instance.new("BodyVelocity")
            BodyVelocity.Parent = Character.HumanoidRootPart
            BodyVelocity.Velocity = Vector3.new(0, FlySpeed, 0)
            BodyVelocity.MaxForce = Vector3.new(400000, 400000, 400000)
        end
    end
end

-- 🏃 Speed
local Speed = 100
function ToggleSpeed()
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = Speed
    end
end

-- 🚪 NoClip (Atravessar paredes)
local NoClip = false
RunService.Stepped:Connect(function()
    if NoClip then
        for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
            if v:IsA("BasePart") and v.CanCollide == true then
                v.CanCollide = false
            end
        end
    end
end)

function ToggleNoClip()
    NoClip = not NoClip
end

-- 🎯 Aimbot (Travamento no inimigo)
local AimbotEnabled = false
function GetClosestPlayer()
    local ClosestPlayer = nil
    local ShortestDistance = math.huge

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            local Pos, OnScreen = Camera:WorldToViewportPoint(Player.Character.HumanoidRootPart.Position)
            if OnScreen then
                local Distance = (Vector2.new(Pos.X, Pos.Y) - Vector2.new(Mouse.X, Mouse.Y)).magnitude
                if Distance < ShortestDistance then
                    ClosestPlayer = Player
                    ShortestDistance = Distance
                end
            end
        end
    end

    return ClosestPlayer
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local Target = GetClosestPlayer()
        if Target and Target.Character and Target.Character:FindFirstChild("HumanoidRootPart") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Target.Character.HumanoidRootPart.Position)
        end
    end
end)

function ToggleAimbot()
    AimbotEnabled = not AimbotEnabled
end

-- 🎮 Controles
UIS.InputBegan:Connect(function(Input, GameProcessed)
    if not GameProcessed then
        if Input.KeyCode == Enum.KeyCode.F then -- Fly
            ToggleFly()
        elseif Input.KeyCode == Enum.KeyCode.V then -- Speed
            ToggleSpeed()
        elseif Input.KeyCode == Enum.KeyCode.N then -- NoClip
            ToggleNoClip()
        elseif Input.KeyCode == Enum.KeyCode.B then -- Aimbot
            ToggleAimbot()
        end
    end
end)
