local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local enabled = true
local espObjects = {}

-- UI
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.CoreGui

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 150, 0, 40)
button.Position = UDim2.new(0, 20, 0, 100)
button.Text = "ESP: ON"
button.BackgroundColor3 = Color3.fromRGB(30,30,30)
button.TextColor3 = Color3.new(1,1,1)
button.Parent = screenGui

button.MouseButton1Click:Connect(function()
    enabled = not enabled
    button.Text = "ESP: " .. (enabled and "ON" or "OFF")

    for _, obj in pairs(espObjects) do
        obj.Highlight.Enabled = enabled
        obj.Billboard.Enabled = enabled
        obj.Box.Visible = enabled
    end
end)

-- Função cor por time
local function getColor(player)
    if player.Team then
        return player.TeamColor.Color
    end
    return Color3.fromRGB(255,0,0)
end

-- Criar ESP
local function createESP(player)
    if player == LocalPlayer then return end

    player.CharacterAdded:Connect(function(char)
        local root = char:WaitForChild("HumanoidRootPart")

        -- Highlight
        local highlight = Instance.new("Highlight")
        highlight.FillTransparency = 0.5
        highlight.OutlineColor = Color3.new(1,1,1)
        highlight.Parent = char

        -- Nome + distância
        local billboard = Instance.new("BillboardGui")
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.AlwaysOnTop = true
        billboard.StudsOffset = Vector3.new(0,3,0)
        billboard.Parent = root

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.new(1,1,1)
        label.TextStrokeTransparency = 0
        label.Parent = billboard

        -- Box fake (SelectionBox)
        local box = Instance.new("SelectionBox")
        box.Adornee = char
        box.LineThickness = 0.05
        box.Parent = char

        espObjects[player] = {
            Highlight = highlight,
            Billboard = billboard,
            Label = label,
            Box = box,
            Character = char
        }
    end)
end

-- Aplicar nos players
for _, p in pairs(Players:GetPlayers()) do
    createESP(p)
end

Players.PlayerAdded:Connect(createESP)

-- Atualização contínua
RunService.RenderStepped:Connect(function()
    for player, data in pairs(espObjects) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local root = player.Character.HumanoidRootPart
            local distance = (LocalPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude

            local color = getColor(player)

            data.Highlight.FillColor = color
            data.Box.Color3 = color
            data.Label.Text = player.Name .. " [" .. math.floor(distance) .. "]"
        end
    end
end)# Frutas-permanentes
