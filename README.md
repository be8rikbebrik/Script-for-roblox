-- Roblox Lua Script для Delta Executor
-- Полный скрипт с раскладным GUI: FOV, WalkSpeed, JumpPower, ESP (Highlight + Names)

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

-- Настройки
local Settings = {
    FOV = 70,
    WalkSpeed = 16,
    JumpPower = 50,
    ESP_Enabled = false,
    ShowNames = true
}

-- Хранилища ESP
local Highlights = {}
local Billboards = {}
local Connections = {}

-- Создание ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeltaCustomGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

-- Кнопка сворачивания/разворачивания (всегда видна)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 50, 0, 50)
ToggleButton.Position = UDim2.new(0, 20, 0.5, -25)
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.Text = "≡"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextScaled = true
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Parent = ScreenGui

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 12)
ToggleCorner.Parent = ToggleButton

local ToggleStroke = Instance.new("UIStroke")
ToggleStroke.Thickness = 2
ToggleStroke.Color = Color3.fromRGB(80, 80, 255)
ToggleStroke.Parent = ToggleButton

-- Основной фрейм
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 320, 0, 420)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -210)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Thickness = 1.5
MainStroke.Color = Color3.fromRGB(60, 60, 255)
MainStroke.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "Delta Custom Menu"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.Parent = MainFrame

-- Функция создания слайдера
local function CreateSlider(parent, name, minVal, maxVal, defaultVal, callback)
    local SliderFrame = Instance.new("Frame")
    SliderFrame.Size = UDim2.new(1, -20, 0, 60)
    SliderFrame.BackgroundTransparency = 1
    SliderFrame.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(1, 0, 0, 20)
    Label.BackgroundTransparency = 1
    Label.Text = name .. ": " .. defaultVal
    Label.TextColor3 = Color3.fromRGB(220, 220, 220)
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Font = Enum.Font.GothamSemibold
    Label.TextSize = 14
    Label.Parent = SliderFrame
    
    local Bar = Instance.new("Frame")
    Bar.Size = UDim2.new(1, 0, 0, 8)
    Bar.Position = UDim2.new(0, 0, 0.6, 0)
    Bar.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    Bar.Parent = SliderFrame
    
    local BarCorner = Instance.new("UICorner")
    BarCorner.CornerRadius = UDim.new(1, 0)
    BarCorner.Parent = Bar
    
    local Fill = Instance.new("Frame")
    Fill.Size = UDim2.new((defaultVal - minVal) / (maxVal - minVal), 0, 1, 0)
    Fill.BackgroundColor3 = Color3.fromRGB(80, 120, 255)
    Fill.Parent = Bar
    
    local FillCorner = Instance.new("UICorner")
    FillCorner.CornerRadius = UDim.new(1, 0)
    FillCorner.Parent = Fill
    
    local Knob = Instance.new("TextButton")
    Knob.Size = UDim2.new(0, 16, 0, 16)
    Knob.Position = UDim2.new((defaultVal - minVal) / (maxVal - minVal), -8, 0.5, -8)
    Knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    Knob.Text = ""
    Knob.Parent = Bar
    
    local KnobCorner = Instance.new("UICorner")
    KnobCorner.CornerRadius = UDim.new(1, 0)
    KnobCorner.Parent = Knob
    
    local value = defaultVal
    local dragging = false
    
    local function updateSlider(pos)
        local relative = math.clamp((pos - Bar.AbsolutePosition.X) / Bar.AbsoluteSize.X, 0, 1)
        value = math.floor(minVal + relative * (maxVal - minVal))
        Fill.Size = UDim2.new(relative, 0, 1, 0)
        Knob.Position = UDim2.new(relative, -8, 0.5, -8)
        Label.Text = name .. ": " .. value
        callback(value)
    end
    
    Knob.MouseButton1Down:Connect(function()
        dragging = true
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    RunService.RenderStepped:Connect(function()
        if dragging then
            local mousePos = UserInputService:GetMouseLocation()
            updateSlider(mousePos.X)
        end
    end)
    
    -- Клик по бару
    Bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local mousePos = UserInputService:GetMouseLocation()
            updateSlider(mousePos.X)
        end
    end)
    
    return SliderFrame
end

-- Контейнер для элементов
local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.new(1, -20, 1, -60)
Content.Position = UDim2.new(0, 10, 0, 50)
Content.BackgroundTransparency = 1
Content.ScrollBarThickness = 6
Content.CanvasSize = UDim2.new(0, 0, 0, 340)
Content.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Padding = UDim.new(0, 10)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Parent = Content

-- Слайдеры
CreateSlider(Content, "FOV", 70, 200, Settings.FOV, function(val)
    Settings.FOV = val
    if Camera then
        Camera.FieldOfView = val
    end
end)

CreateSlider(Content, "WalkSpeed", 16, 500, Settings.WalkSpeed, function(val)
    Settings.WalkSpeed = val
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.WalkSpeed = val
        end
    end
end)

CreateSlider(Content, "JumpPower", 50, 500, Settings.JumpPower, function(val)
    Settings.JumpPower = val
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum.JumpPower = val
        end
    end
end)

-- Toggle ESP
local ESPToggleFrame = Instance.new("Frame")
ESPToggleFrame.Size = UDim2.new(1, -20, 0, 50)
ESPToggleFrame.BackgroundTransparency = 1
ESPToggleFrame.Parent = Content

local ESPToggleLabel = Instance.new("TextLabel")
ESPToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
ESPToggleLabel.BackgroundTransparency = 1
ESPToggleLabel.Text = "ESP (Highlight)"
ESPToggleLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
ESPToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
ESPToggleLabel.Font = Enum.Font.GothamSemibold
ESPToggleLabel.TextSize = 16
ESPToggleLabel.Parent = ESPToggleFrame

local ESPToggleButton = Instance.new("TextButton")
ESPToggleButton.Size = UDim2.new(0, 80, 0, 30)
ESPToggleButton.Position = UDim2.new(1, -90, 0.5, -15)
ESPToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
ESPToggleButton.Text = "OFF"
ESPToggleButton.TextColor3 = Color3.fromRGB(255, 80, 80)
ESPToggleButton.Font = Enum.Font.GothamBold
ESPToggleButton.TextSize = 14
ESPToggleButton.Parent = ESPToggleFrame

local ESPToggleCorner = Instance.new("UICorner")
ESPToggleCorner.CornerRadius = UDim.new(0, 8)
ESPToggleCorner.Parent = ESPToggleButton

local function updateESPToggle()
    if Settings.ESP_Enabled then
        ESPToggleButton.Text = "ON"
        ESPToggleButton.BackgroundColor3 = Color3.fromRGB(80, 200, 120)
        ESPToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        ESPToggleButton.Text = "OFF"
        ESPToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        ESPToggleButton.TextColor3 = Color3.fromRGB(255, 80, 80)
    end
end

ESPToggleButton.MouseButton1Click:Connect(function()
    Settings.ESP_Enabled = not Settings.ESP_Enabled
    updateESPToggle()
    RefreshESP()
end)

-- Toggle Names
local NameToggleFrame = Instance.new("Frame")
NameToggleFrame.Size = UDim2.new(1, -20, 0, 50)
NameToggleFrame.BackgroundTransparency = 1
NameToggleFrame.Parent = Content

local NameToggleLabel = Instance.new("TextLabel")
NameToggleLabel.Size = UDim2.new(0.7, 0, 1, 0)
NameToggleLabel.BackgroundTransparency = 1
NameToggleLabel.Text = "Показывать ники"
NameToggleLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
NameToggleLabel.TextXAlignment = Enum.TextXAlignment.Left
NameToggleLabel.Font = Enum.Font.GothamSemibold
NameToggleLabel.TextSize = 16
NameToggleLabel.Parent = NameToggleFrame

local NameToggleButton = Instance.new("TextButton")
NameToggleButton.Size = UDim2.new(0, 80, 0, 30)
NameToggleButton.Position = UDim2.new(1, -90, 0.5, -15)
NameToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
NameToggleButton.Text = "OFF"
NameToggleButton.TextColor3 = Color3.fromRGB(255, 80, 80)
NameToggleButton.Font = Enum.Font.GothamBold
NameToggleButton.TextSize = 14
NameToggleButton.Parent = NameToggleFrame

local NameToggleCorner = Instance.new("UICorner")
NameToggleCorner.CornerRadius = UDim.new(0, 8)
NameToggleCorner.Parent = NameToggleButton

local function updateNameToggle()
    if Settings.ShowNames then
        NameToggleButton.Text = "ON"
        NameToggleButton.BackgroundColor3 = Color3.fromRGB(80, 200, 120)
    else
        NameToggleButton.Text = "OFF"
        NameToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    end
end

NameToggleButton.MouseButton1Click:Connect(function()
    Settings.ShowNames = not Settings.ShowNames
    updateNameToggle()
    RefreshESP()
end)

-- Логика ESP
local function CreateHighlight(character)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "CustomESPHighlight"
    highlight.FillColor = Color3.fromRGB(255, 80, 80)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.7
    highlight.OutlineTransparency = 0
    highlight.Adornee = character
    highlight.Parent = character
    
    return highlight
end

local function CreateBillboard(character, player)
    if not character or not character:FindFirstChild("Head") then return end
    
    local head = character.Head
    
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "CustomESPName"
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.LightInfluence = 0
    billboard.Parent = character
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = player.Name
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextStrokeTransparency = 0
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.GothamBold
    textLabel.Parent = billboard
    
    return billboard
end

local function RemoveESPForPlayer(player)
    if Highlights[player] then
        Highlights[player]:Destroy()
        Highlights[player] = nil
    end
    if Billboards[player] then
        Billboards[player]:Destroy()
        Billboards[player] = nil
    end
end

function RefreshESP()
    -- Очистка
    for player, _ in pairs(Highlights) do
        RemoveESPForPlayer(player)
    end
    
    if not Settings.ESP_Enabled then return end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local char = player.Character
            Highlights[player] = CreateHighlight(char)
            if Settings.ShowNames then
                Billboards[player] = CreateBillboard(char, player)
            end
        end
    end
end

local function SetupPlayer(player)
    if player == LocalPlayer then return end
    
    local conn1 = player.CharacterAdded:Connect(function(char)
        task.wait(0.5) -- Ждём загрузки
        if Settings.ESP_Enabled then
            Highlights[player] = CreateHighlight(char)
            if Settings.ShowNames then
                Billboards[player] = CreateBillboard(char, player)
            end
        end
        
        -- Применяем скорости при респавне
        local hum = char:WaitForChild("Humanoid", 3)
        if hum then
            hum.WalkSpeed = Settings.WalkSpeed
            hum.JumpPower = Settings.JumpPower
        end
    end)
    
    local conn2 = player.CharacterRemoving:Connect(function()
        RemoveESPForPlayer(player)
    end)
    
    table.insert(Connections, conn1)
    table.insert(Connections, conn2)
    
    -- Если персонаж уже существует
    if player.Character then
        task.spawn(function()
            local char = player.Character
            task.wait(0.5)
            if Settings.ESP_Enabled then
                Highlights[player] = CreateHighlight(char)
                if Settings.ShowNames then
                    Billboards[player] = CreateBillboard(char, player)
                end
            end
        end)
    end
end

-- Обработка всех игроков
local function InitESP()
    for _, player in ipairs(Players:GetPlayers()) do
        SetupPlayer(player)
    end
    
    Players.PlayerAdded:Connect(SetupPlayer)
    Players.PlayerRemoving:Connect(function(player)
        RemoveESPForPlayer(player)
    end)
end

-- Применение настроек при респавне локального игрока
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.6)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.WalkSpeed = Settings.WalkSpeed
        hum.JumpPower = Settings.JumpPower
    end
    Camera.FieldOfView = Settings.FOV
end)

-- Логика сворачивания
local isVisible = true
ToggleButton.MouseButton1Click:Connect(function()
    isVisible = not isVisible
    if isVisible then
        MainFrame.Visible = true
        TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quint), {Size = UDim2.new(0, 320, 0, 420)}):Play()
    else
        TweenService:Create(MainFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {Size = UDim2.new(0, 320, 0, 0)}):Play()
        task.wait(0.25)
        MainFrame.Visible = false
    end
end)

-- Инициализация
updateESPToggle()
updateNameToggle()

-- Применяем начальные значения
Camera.FieldOfView = Settings.FOV

task.spawn(function()
    if LocalPlayer.Character then
        local hum = LocalPlayer.Character:WaitForChild("Humanoid", 5)
        if hum then
            hum.WalkSpeed = Settings.WalkSpeed
            hum.JumpPower = Settings.JumpPower
        end
    end
end)

InitESP()

print("Delta Custom GUI загружен успешно!")
