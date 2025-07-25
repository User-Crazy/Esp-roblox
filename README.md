local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local espEnabled = false
local highlights = {}
local connection

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESPSystem"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 80)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "Title"
titleLabel.Size = UDim2.new(1, 0, 0, 25)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
titleLabel.BorderSizePixel = 0
titleLabel.Text = "ESP System"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleLabel

local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0, 80, 0, 35)
toggleButton.Position = UDim2.new(0.5, -40, 0, 35)
toggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
toggleButton.BorderSizePixel = 0
toggleButton.Text = "OFF"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.GothamBold
toggleButton.Parent = mainFrame

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 6)
buttonCorner.Parent = toggleButton

local function createHighlight(character)
    if not character or highlights[character] then return end
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESPHighlight"
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.FillTransparency = 0.5
    highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineTransparency = 0
    highlight.Parent = character
    highlights[character] = highlight
end

local function removeHighlight(character)
    if highlights[character] then
        highlights[character]:Destroy()
        highlights[character] = nil
    end
end

local function updateESP()
    if espEnabled then
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                createHighlight(otherPlayer.Character)
            end
        end
    else
        for character, highlight in pairs(highlights) do
            highlight:Destroy()
        end
        highlights = {}
    end
end

local function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        toggleButton.Text = "ON"
        toggleButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        connection = RunService.Heartbeat:Connect(function()
            updateESP()
        end)
        Players.PlayerAdded:Connect(function(newPlayer)
            newPlayer.CharacterAdded:Connect(function(character)
                if espEnabled then
                    wait(0.1)
                    createHighlight(character)
                end
            end)
        end)  
    else
        toggleButton.Text = "OFF"
        toggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        if connection then
            connection:Disconnect()
            connection = nil
        end
    end  
    updateESP()
end

toggleButton.MouseButton1Click:Connect(toggleESP)

local dragging = false
local dragStart = nil
local startPos = nil

local function updateInput(input)
    local delta = input.Position - dragStart
    local position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    mainFrame.Position = position
end

titleLabel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

titleLabel.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        if dragging then
            updateInput(input)
        end
    end
end)

Players.PlayerRemoving:Connect(function(leavingPlayer)
    if leavingPlayer.Character and highlights[leavingPlayer.Character] then
        removeHighlight(leavingPlayer.Character)
    end
end)

mainFrame.BackgroundTransparency = 1
titleLabel.BackgroundTransparency = 1
titleLabel.TextTransparency = 1
toggleButton.BackgroundTransparency = 1
toggleButton.TextTransparency = 1

local fadeIn = TweenService:Create(mainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0})
local fadeInTitle = TweenService:Create(titleLabel, TweenInfo.new(0.3), {BackgroundTransparency = 0, TextTransparency = 0})
local fadeInButton = TweenService:Create(toggleButton, TweenInfo.new(0.3), {BackgroundTransparency = 0, TextTransparency = 0})

fadeIn:Play()
fadeInTitle:Play()
fadeInButton:Play()

