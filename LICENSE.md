-- script.lua
-- Complex Roblox LocalScript for detecting malicious player scripts in testing
-- Features: ESP Box, Aimbot, Hologram, Shoot on Look, Shoot on Aim, Auto Kill
-- Includes a UI to toggle and configure features
-- IMPORTANT: Must be run as a LocalScript in Roblox environment

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Variables
local enabledFeatures = {
    ESP = true,
    Aimbot = true,
    Hologram = true,
    ShootOnLook = true,
    ShootOnAim = true,
    AutoKill = false,
}

local settings = {
    AimFOV = 40,           -- degrees
    AimSmoothness = 0.3,   -- 0-1, lower means smoother/slower
    ShootInterval = 0.2,   -- seconds between shots
    HologramTransparency = 0.6,
    ESPBoxColor = Color3.fromRGB(0,255,0),
}

local lastShotTime = 0

-- UI Creation
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MaliciousScriptDetectionUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 300, 0, 350)
MainFrame.Position = UDim2.new(0, 20, 0.5, -175)
MainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Parent = ScreenGui
MainFrame.Active = true
MainFrame.Draggable = true

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.BackgroundTransparency = 1
title.Text = "Malicious Script Detection"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.Parent = MainFrame

local function CreateToggle(parent, text, defaultChecked, onToggle)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1,-20,0,30)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7,0,1,0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 18
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0.3,0,1,0)
    toggleButton.Position = UDim2.new(0.7,0,0,0)
    toggleButton.BackgroundColor3 = (defaultChecked and Color3.fromRGB(0,170,0) or Color3.fromRGB(170,0,0))
    toggleButton.Text = (defaultChecked and "ON" or "OFF")
    toggleButton.TextColor3 = Color3.new(1,1,1)
    toggleButton.Font = Enum.Font.SourceSansBold
    toggleButton.TextSize = 18
    toggleButton.Parent = frame

    toggleButton.MouseButton1Click:Connect(function()
        local isOn = toggleButton.Text == "ON"
        if isOn then
            toggleButton.Text = "OFF"
            toggleButton.BackgroundColor3 = Color3.fromRGB(170,0,0)
            onToggle(false)
        else
            toggleButton.Text = "ON"
            toggleButton.BackgroundColor3 = Color3.fromRGB(0,170,0)
            onToggle(true)
        end
    end)
end

local function CreateSlider(parent, text, min, max, default, decimals, onChange)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1,-20,0,50)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1,0,0,20)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. tostring(default)
    label.TextColor3 = Color3.new(1,1,1)
    label.Font = Enum.Font.SourceSans
    label.TextSize = 16
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local slider = Instance.new("TextBox")
    slider.Size = UDim2.new(1,0,0,20)
    slider.Position = UDim2.new(0,0,0,25)
    slider.Text = tostring(default)
    slider.ClearTextOnFocus = false
    slider.TextColor3 = Color3.new(1,1,1)
    slider.BackgroundColor3 = Color3.fromRGB(50,50,50)
    slider.Font = Enum.Font.SourceSans
    slider.TextSize = 18
    slider.Parent = frame

    local function updateValue(val)
        local num = tonumber(val)
        if num and num >= min and num <= max then
            onChange(num)
            label.Text = text .. ": " .. string.format("%."..decimals.."f", num)
            slider.TextColor3 = Color3.new(1,1,1)
        else
            slider.TextColor3 = Color3.fromRGB(255,100,100)
        end
    end

    slider.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            updateValue(slider.Text)
        else
            slider.Text = string.format("%."..decimals.."f", onChange and settings[text:gsub("%s","")] or default)
            slider.TextColor3 = Color3.new(1,1,1)
        end
    end)
end

local UIContainer = Instance.new("Frame")
UIContainer.Size = UDim2.new(1,-20,1,-40)
UIContainer.BackgroundTransparency = 1
UIContainer.Position = UDim2.new(0,10,0,35)
UIContainer.Parent = MainFrame

CreateToggle(UIContainer, "ESP", enabledFeatures.ESP, function(v) enabledFeatures.ESP = v end)
CreateToggle(UIContainer, "Aimbot", enabledFeatures.Aimbot, function(v) enabledFeatures.Aimbot = v end)
CreateToggle(UIContainer, "Hologram", enabledFeatures.Hologram, function(v) enabledFeatures.Hologram = v end)
CreateToggle(UIContainer, "ShootOnLook", enabledFeatures.ShootOnLook, function(v) enabledFeatures.ShootOnLook = v end)
CreateToggle(UIContainer, "ShootOnAim", enabledFeatures.ShootOnAim, function(v) enabledFeatures.ShootOnAim = v end)
CreateToggle(UIContainer, "AutoKill", enabledFeatures.AutoKill, function(v) enabledFeatures.AutoKill = v end)

CreateSlider(UIContainer, "AimFOV", 5, 180, settings.AimFOV, 0, function(v) settings.AimFOV = v end)
CreateSlider(UIContainer, "AimSmoothness", 0, 1, settings.AimSmoothness, 2, function(v) settings.AimSmoothness = v end)
CreateSlider(UIContainer, "ShootInterval", 0.05, 1, settings.ShootInterval, 2, function(v) settings.ShootInterval = v end)
CreateSlider(UIContainer, "HologramTransparency", 0, 1, settings.HologramTransparency, 2, function(v) settings.HologramTransparency = v end)

-- Utility functions

local function getCharacterRoot(char)
    return char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart
end

local function isAlive(player)
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        return humanoid and humanoid.Health > 0
    end
    return false
end

local function getClosestPlayerToMouse()
    local closestDistance = math.huge
    local closestPlayer = nil
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isAlive(player) then
            local char = player.Character
            local root = getCharacterRoot(char)
            if root then
                local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(root.Position)
                if onScreen then
                    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                    local delta = Vector2.new(screenPos.X, screenPos.Y) - mousePos
                    local dist = delta.Magnitude
                    if dist < closestDistance then
                        closestDistance = dist
                        closestPlayer = player
                    end
                end
            end
        end
    end
    if closestDistance <= settings.AimFOV then
        return closestPlayer
    else
        return nil
    end
end

local function drawESPBox(player)
    if not player.Character then return end
    local rootPart = getCharacterRoot(player.Character)
    if not rootPart then return end

    -- We'll use BillboardGui attached to rootPart for drawing box
    -- Check if already has billboardgui
    if rootPart:FindFirstChild("ESPBox") then
        return
    end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESPBox"
    billboard.Adornee = rootPart
    billboard.AlwaysOnTop = true
    billboard.Size = UDim2.new(4, 0, 6, 0)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.Parent = rootPart

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1,0,1,0)
    frame.BackgroundColor3 = settings.ESPBoxColor
    frame.BorderColor3 = Color3.new(0,0,0)
    frame.BorderSizePixel = 2
    frame.BackgroundTransparency = 0.7
    frame.Parent = billboard
end

local function removeESPBox(player)
    if player.Character then
        local rootPart = getCharacterRoot(player.Character)
        if rootPart and rootPart:FindFirstChild("ESPBox") then
            rootPart.ESPBox:Destroy()
        end
    end
end

-- Hologram creation
local function createHologram(player)
    if not player.Character then return end
    if player.Character:FindFirstChild("HologramClone") then return end

    local clone = player.Character:Clone()
    clone.Name = "HologramClone"
    for _, obj in pairs(clone:GetDescendants()) do
        if obj:IsA("BasePart") then
            obj.Transparency = settings.HologramTransparency
            obj.Material = Enum.Material.Neon
            obj.CanCollide = false
            obj.Anchored = true
        elseif obj:IsA("Decal") then
            obj.Transparency = settings.HologramTransparency
        end
    end
    clone.Parent = workspace

    -- Keep it in sync with original: loop to update position
    spawn(function()
        while clone and clone.Parent and player.Character and player.Character.Parent do
            for _, part in pairs(clone:GetDescendants()) do
                if part:IsA("BasePart") then
                    local originalPart = player.Character:FindFirstChild(part.Name)
                    if originalPart then
                        part.CFrame = originalPart.CFrame
                    end
                end
            end
            wait(0.05)
        end
        if clone then
            clone:Destroy()
        end
    end)
end

local function removeHologram(player)
    if player.Character then
        local holo = workspace:FindFirstChild("HologramClone")
        if holo then
            holo:Destroy()
        end
    end
end

-- Aimbot  
local function aimbot()
    if not enabledFeatures.Aimbot then return end
    if not isAlive(LocalPlayer) then return end

    local target = getClosestPlayerToMouse()
    if target then
        local root = getCharacterRoot(target.Character)
        if root then
            local camera = workspace.CurrentCamera
            local localRoot = getCharacterRoot(LocalPlayer.Character)
            if localRoot then
                local camCF = camera.CFrame
                local targetPos = root.Position + Vector3.new(0, 1.5, 0) -- Aim towards head approx

                local direction = (targetPos - camCF.Position).Unit
                local lookVector = camCF.LookVector

                -- Smooth aim adjustment
                local currentLookVector = lookVector
                local smoothDir = currentLookVector:Lerp(direction, settings.AimSmoothness)
                camera.CFrame = CFrame.new(camCF.Position, camCF.Position + smoothDir)
            end
        end
    end
end

-- Shoot action (stub)
local function shoot()
    local now = tick()
    if now - lastShotTime < settings.ShootInterval then
        return
    end
    lastShotTime = now
    -- Here you must implement firing logic; for demonstration:
    print("[Action] Shoot")
    -- Example: fire remote event or simulate mouse button1 down if possible
end

local function isLookingAt(player)
    local mouseTarget = Mouse.Target
    local char = player.Character
    if not (mouseTarget and char) then return false end

    return mouseTarget:IsDescendantOf(char)
end

local function isAimingAt(player)
    -- We estimate aiming as close to mouse target being the player
    return isLookingAt(player)
end

local function performShootOnLookAim()
    if not (enabledFeatures.ShootOnLook or enabledFeatures.ShootOnAim) then return end
    if not isAlive(LocalPlayer) then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isAlive(player) then
            if enabledFeatures.ShootOnLook and isLookingAt(player) then
                shoot()
                return
            end
            if enabledFeatures.ShootOnAim and isAimingAt(player) then
                shoot()
                return
            end
        end
    end
end

-- AutoKill
local function autoKill()
    if not enabledFeatures.AutoKill then return end
    if not isAlive(LocalPlayer) then return end

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isAlive(player) then
            local root = getCharacterRoot(player.Character)
            if root then
                -- Teleport close or simulate killing (game dependent)
                -- Here, just print a message (replace with real kill logic)
                print("[AutoKill] Attempting kill on "..player.Name)
                -- Example: Fire server event, or perform melee/hit action here
            end
        end
    end
end

-- Main update loop
RunService.RenderStepped:Connect(function()
    if enabledFeatures.ESP then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and isAlive(player) then
                drawESPBox(player)
                if enabledFeatures.Hologram then
                    createHologram(player)
                else
                    removeHologram(player)
                end
            else
                removeESPBox(player)
                removeHologram(player)
            end
        end
    else
        -- Clean up ESP and hologram if disabled
        for _, player in pairs(Players:GetPlayers()) do
            removeESPBox(player)
            removeHologram(player)
        end
    end

    if enabledFeatures.Aimbot then
        aimbot()
    end

    performShootOnLookAim()
    autoKill()
end)

print("Malicious script detection tool loaded. Use GUI to modify settings.")
