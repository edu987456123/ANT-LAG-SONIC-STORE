local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local gui = Instance.new("ScreenGui")
local frame = Instance.new("Frame")
local hideButton = Instance.new("ImageButton")
local showButton = Instance.new("ImageButton")
local antiLagButton = Instance.new("TextButton")
local titleLabel = Instance.new("TextLabel")

-- Configuração do GUI
gui.Name = "AntiLagGUI"
gui.Parent = game.CoreGui

frame.Size = UDim2.new(0, 250, 0, 200)
frame.Position = UDim2.new(0.5, -125, 0.5, -100)
frame.BackgroundColor3 = Color3.new(0, 0, 1) -- Azul
frame.BackgroundTransparency = 0.1
frame.BorderSizePixel = 0
frame.AnchorPoint = Vector2.new(0.5, 0.5)
frame.Parent = gui

-- Tornar o frame com formato de escudo
local shieldShape = Instance.new("UICorner")
shieldShape.CornerRadius = UDim.new(0.1, 0)
shieldShape.Parent = frame

local gradient = Instance.new("UIGradient")
gradient.Color = ColorSequence.new(Color3.new(0, 0, 1), Color3.new(0, 0, 0.5))
gradient.Parent = frame

hideButton.Size = UDim2.new(0, 20, 0, 20)
hideButton.Position = UDim2.new(1, -30, 0, 10)
hideButton.Image = "rbxassetid://6035047409" -- ID da imagem do botão de minimizar (X)
hideButton.Parent = frame

showButton.Size = UDim2.new(0, 50, 0, 50)
showButton.Position = UDim2.new(0, 10, 0, 10)
showButton.BackgroundColor3 = Color3.new(0, 0, 0.5) -- Azul escuro
showButton.Image = "rbxassetid://6031068420" -- ID da imagem do botão de mostrar (bola com símbolo de raio)
showButton.Parent = gui
showButton.Visible = false

titleLabel.Size = UDim2.new(0, 200, 0, 50)
titleLabel.Position = UDim2.new(0.5, 0, 0, 10)
titleLabel.Text = "SONIC STORE"
titleLabel.TextScaled = true
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.BackgroundTransparency = 1
titleLabel.AnchorPoint = Vector2.new(0.5, 0)
titleLabel.Parent = frame

antiLagButton.Size = UDim2.new(0, 200, 0, 50)
antiLagButton.Position = UDim2.new(0.5, 0, 0.5, 25)
antiLagButton.Text = "Ativar Anti-Lag"
antiLagButton.BackgroundColor3 = Color3.new(0.7, 0.7, 0.7)
antiLagButton.AnchorPoint = Vector2.new(0.5, 0.5)
antiLagButton.Parent = frame

local antiLagEnabled = false
local originalSettings = {
    QualityLevel = settings().Rendering.QualityLevel,
    GlobalShadows = Lighting.GlobalShadows,
    FogEnd = Lighting.FogEnd,
    Brightness = Lighting.Brightness,
    PartSettings = {}
}

for _, v in pairs(workspace:GetDescendants()) do
    if v:IsA("BasePart") then
        table.insert(originalSettings.PartSettings, {part = v, CastShadow = v.CastShadow, Material = v.Material})
    end
end

local function optimizeGame()
    -- Configurações para otimizar o jogo e aumentar o FPS
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 5000
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") then
            v.CastShadow = false
            v.Material = Enum.Material.SmoothPlastic -- Reduzir a qualidade das texturas
        end
    end
    Lighting.Brightness = 1 -- Reduzir o brilho para otimização
end

local function resetGameSettings()
    -- Configurações para reverter as otimizações e restaurar o estado original
    settings().Rendering.QualityLevel = originalSettings.QualityLevel
    Lighting.GlobalShadows = originalSettings.GlobalShadows
    Lighting.FogEnd = originalSettings.FogEnd
    Lighting.Brightness = originalSettings.Brightness
    for _, v in pairs(originalSettings.PartSettings) do
        v.part.CastShadow = v.CastShadow
        v.part.Material = v.Material
    end
end

antiLagButton.MouseButton1Click:Connect(function()
    antiLagEnabled = not antiLagEnabled
    antiLagButton.Text = antiLagEnabled and "Desativar Anti-Lag" or "Ativar Anti-Lag"
    if antiLagEnabled then
        optimizeGame()
    else
        resetGameSettings()
    end
end)

hideButton.MouseButton1Click:Connect(function()
    frame.Visible = false
    showButton.Visible = true
end)

showButton.MouseButton1Click:Connect(function()
    frame.Visible = true
    showButton.Visible = false
end)

-- Adicionar funcionalidade de arrastar para frame e showButton
local function makeDraggable(frame)
    local dragging
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
end

makeDraggable(frame)
makeDraggable(showButton)
