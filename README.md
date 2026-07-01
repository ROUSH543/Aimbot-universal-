--[[
    ================================================================
    MODERN DARK UI - COMBAT MINI (HEADSHOT UPDATE)
    Painel compacto com Aimbot (Cabeça), Filtros, Kill Aura e ESP.
    ================================================================
]]

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local Theme = {
    Background = Color3.fromRGB(20, 20, 23),
    Topbar = Color3.fromRGB(30, 30, 36),
    Accent = Color3.fromRGB(90, 105, 250),
    Text = Color3.fromRGB(240, 240, 245),
    TextMuted = Color3.fromRGB(150, 150, 160),
    ComponentBg = Color3.fromRGB(32, 32, 38),
    ComponentHover = Color3.fromRGB(40, 40, 48),
    Font = Enum.Font.BuilderSans,
    FontBold = Enum.Font.BuilderSansBold,
    TweenInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
}

local UiLib = {}
UiLib.__index = UiLib

local function ApplyCorner(parent, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius or 8)
    corner.Parent = parent
    return corner
end

local function ApplyStroke(parent, color, thickness)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color or Color3.fromRGB(45, 45, 55)
    stroke.Thickness = thickness or 1
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    stroke.Parent = parent
    return stroke
end

local function CreateTween(object, goals, info)
    local tween = TweenService:Create(object, info or Theme.TweenInfo, goals)
    tween:Play()
    return tween
end

local function MakeDraggable(dragFrame, parentFrame)
    local dragging = false
    local dragInput
    local dragStart
    local startPos

    dragFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = parentFrame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    dragFrame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            parentFrame.Position = UDim2.new(
                startPos.X.Scale, 
                startPos.X.Offset + delta.X, 
                startPos.Y.Scale, 
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

function UiLib:CreateWindow(titleText)
    local selfObj = setmetatable({}, UiLib)
    
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "UiLib_MiniCombatESP"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent = PlayerGui

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 340, 0, 440)
    MainFrame.Position = UDim2.new(0.5, -170, 0.5, -220)
    MainFrame.BackgroundColor3 = Theme.Background
    MainFrame.Parent = ScreenGui
    ApplyCorner(MainFrame, 10)
    ApplyStroke(MainFrame, nil, 1)

    local Topbar = Instance.new("Frame")
    Topbar.Size = UDim2.new(1, 0, 0, 35)
    Topbar.BackgroundColor3 = Theme.Topbar
    Topbar.Parent = MainFrame
    ApplyCorner(Topbar, 10)
    MakeDraggable(Topbar, MainFrame)

    local Fix = Instance.new("Frame")
    Fix.Size = UDim2.new(1, 0, 0, 10)
    Fix.Position = UDim2.new(0, 0, 1, -10)
    Fix.BackgroundColor3 = Theme.Topbar
    Fix.BorderSizePixel = 0
    Fix.Parent = Topbar

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0, 180, 1, 0)
    Title.Position = UDim2.new(0, 12, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = titleText or "Combat Mini"
    Title.TextColor3 = Theme.Text
    Title.Font = Theme.FontBold
    Title.TextSize = 14
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = Topbar

    local ContentFrame = Instance.new("ScrollingFrame")
    ContentFrame.Size = UDim2.new(1, -20, 1, -50)
    ContentFrame.Position = UDim2.new(0, 10, 0, 45)
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.ScrollBarThickness = 2
    ContentFrame.ScrollBarImageColor3 = Theme.Accent
    ContentFrame.Parent = MainFrame

    local PageLayout = Instance.new("UIListLayout")
    PageLayout.Padding = UDim.new(0, 8)
    PageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    PageLayout.Parent = ContentFrame

    PageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        ContentFrame.CanvasSize = UDim2.new(0, 0, 0, PageLayout.AbsoluteContentSize.Y + 10)
    end)

    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 24, 0, 24)
    CloseBtn.Position = UDim2.new(1, -30, 0, 5.5)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(240, 70, 70)
    CloseBtn.Text = "✕"
    CloseBtn.TextColor3 = Theme.Text
    CloseBtn.Font = Theme.FontBold
    CloseBtn.TextSize = 10
    CloseBtn.Parent = Topbar
    ApplyCorner(CloseBtn, 5)

    CloseBtn.MouseButton1Click:Connect(function()
        ScreenGui:Destroy()
    end)

    local MinBtn = Instance.new("TextButton")
    MinBtn.Size = UDim2.new(0, 24, 0, 24)
    MinBtn.Position = UDim2.new(1, -58, 0, 5.5)
    MinBtn.BackgroundColor3 = Theme.ComponentBg
    MinBtn.Text = "—"
    MinBtn.TextColor3 = Theme.Text
    MinBtn.Font = Theme.FontBold
    MinBtn.TextSize = 10
    MinBtn.Parent = Topbar
    ApplyCorner(MinBtn, 5)

    local isMinimized = false
    MinBtn.MouseButton1Click:Connect(function()
        isMinimized = not isMinimized
        if isMinimized then
            ContentFrame.Visible = false
            CreateTween(MainFrame, {Size = UDim2.new(0, 340, 0, 35)}, Theme.TweenInfo)
        else
            CreateTween(MainFrame, {Size = UDim2.new(0, 340, 0, 440)}, Theme.TweenInfo)
            task.wait(0.2)
            ContentFrame.Visible = true
        end
    end)

    selfObj.Container = ContentFrame
    return selfObj
end

function UiLib:CreateToggle(text, default, callback)
    local state = default or false
    callback = callback or function() end

    local Tog = Instance.new("TextButton")
    Tog.Size = UDim2.new(1, 0, 0, 36)
    Tog.BackgroundColor3 = Theme.ComponentBg
    Tog.Text = "  " .. text
    Tog.TextColor3 = Theme.Text
    Tog.Font = Theme.Font
    Tog.TextSize = 13
    Tog.TextXAlignment = Enum.TextXAlignment.Left
    Tog.Parent = self.Container
    ApplyCorner(Tog, 6)
    ApplyStroke(Tog)

    local Box = Instance.new("Frame")
    Box.Size = UDim2.new(0, 20, 0, 20)
    Box.Position = UDim2.new(1, -30, 0.5, -10)
    Box.BackgroundColor3 = Theme.Background
    Box.Parent = Tog
    ApplyCorner(Box, 4)
    local BoxStroke = ApplyStroke(Box)

    local Indicator = Instance.new("Frame")
    Indicator.Size = UDim2.new(0, 10, 0, 10)
    Indicator.Position = UDim2.new(0.5, -5, 0.5, -5)
    Indicator.BackgroundColor3 = Theme.Accent
    Indicator.BackgroundTransparency = state and 0 or 1
    Indicator.Parent = Box
    ApplyCorner(Indicator, 3)

    local function Update()
        CreateTween(Indicator, {BackgroundTransparency = state and 0 or 1})
        CreateTween(BoxStroke, {Color = state and Theme.Accent or Color3.fromRGB(45, 45, 55)})
        callback(state)
    end

    Tog.MouseButton1Click:Connect(function()
        state = not state
        Update()
    end)
    Update()
end

function UiLib:CreateSlider(text, min, max, default, callback)
    min = min or 0; max = max or 100; default = default or min
    local value = default
    callback = callback or function() end

    local Sld = Instance.new("Frame")
    Sld.Size = UDim2.new(1, 0, 0, 42)
    Sld.BackgroundColor3 = Theme.ComponentBg
    Sld.Parent = self.Container
    ApplyCorner(Sld, 6)
    ApplyStroke(Sld)

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0, 180, 0, 18)
    Title.Position = UDim2.new(0, 10, 0, 4)
    Title.BackgroundTransparency = 1
    Title.Text = text
    Title.TextColor3 = Theme.Text
    Title.Font = Theme.Font
    Title.TextSize = 13
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = Sld

    local ValueLabel = Instance.new("TextLabel")
    ValueLabel.Size = UDim2.new(0, 50, 0, 18)
    ValueLabel.Position = UDim2.new(1, -60, 0, 4)
    ValueLabel.BackgroundTransparency = 1
    ValueLabel.Text = tostring(value)
    ValueLabel.TextColor3 = Theme.Accent
    ValueLabel.Font = Theme.FontBold
    ValueLabel.TextSize = 13
    ValueLabel.TextXAlignment = Enum.TextXAlignment.Right
    ValueLabel.Parent = Sld

    local SlideBar = Instance.new("TextButton")
    SlideBar.Size = UDim2.new(1, -20, 0, 5)
    SlideBar.Position = UDim2.new(0, 10, 1, -10)
    SlideBar.BackgroundColor3 = Theme.Background
    SlideBar.Text = ""
    SlideBar.Parent = Sld
    ApplyCorner(SlideBar, 3)

    local Fill = Instance.new("Frame")
    Fill.Size = UDim2.new((value - min) / (max - min), 0, 1, 0)
    Fill.BackgroundColor3 = Theme.Accent
    Fill.Parent = SlideBar
    ApplyCorner(Fill, 3)

    local function Snap(input)
        local pct = math.clamp((input.Position.X - SlideBar.AbsolutePosition.X) / SlideBar.AbsoluteSize.X, 0, 1)
        value = math.floor(min + (max - min) * pct)
        ValueLabel.Text = tostring(value)
        CreateTween(Fill, {Size = UDim2.new(pct, 0, 1, 0)}, TweenInfo.new(0.05))
        callback(value)
    end

    local sliding = false
    SlideBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            sliding = true
            Snap(input)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if sliding and input.UserInputType == Enum.UserInputType.MouseMovement then
            Snap(input)
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            sliding = false
        end
    end)
end

-- [[ CONFIGURAÇÕES DO SCRIPT ]]
local Config = {
    SilentAimEnabled = false,
    AimRange = 100,
    TeamCheck = false,
    TargetInCars = false,
    KillAuraEnabled = false,
    AuraRange = 50,
    
    -- Configurações de ESP
    EspChams = false,
    EspNames = false
}

-- [[ LÓGICA DE ESP (CONTORNO AZUL E NOMES) ]]
local function AdicionarESP(player)
    if player == LocalPlayer then return end

    local function SetupCharacter(character)
        local head = character:WaitForChild("Head", 5)
        if not head then return end

        -- 1. Criação do Contorno Azul (Highlight)
        local highlight = character:FindFirstChild("CombatESP_Highlight") or Instance.new("Highlight")
        highlight.Name = "CombatESP_Highlight"
        highlight.FillColor = Color3.fromRGB(0, 120, 255)
        highlight.FillTransparency = 0.6
        highlight.OutlineColor = Color3.fromRGB(0, 180, 255)
        highlight.OutlineTransparency = 0
        highlight.Adornee = character
        highlight.Enabled = Config.EspChams
        highlight.Parent = character

        -- 2. Criação do Nome (BillboardGui)
        local bbGui = head:FindFirstChild("CombatESP_NameGui") or Instance.new("BillboardGui")
        bbGui.Name = "CombatESP_NameGui"
        bbGui.Size = UDim2.new(0, 100, 0, 30)
        bbGui.StudsOffset = Vector3.new(0, 2.5, 0)
        bbGui.AlwaysOnTop = true
        bbGui.Enabled = Config.EspNames
        
        local label = bbGui:FindFirstChild("NameLabel") or Instance.new("TextLabel")
        label.Name = "NameLabel"
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = player.Name
        label.TextColor3 = Color3.fromRGB(240, 240, 255)
        label.Font = Theme.FontBold
        label.TextSize = 12
        label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        label.TextStrokeTransparency = 0
        label.Parent = bbGui
        
        bbGui.Parent = head
    end

    if player.Character then SetupCharacter(player.Character) end
    player.CharacterAdded:Connect(SetupCharacter)
end

for _, p in ipairs(Players:GetPlayers()) do AdicionarESP(p) end
Players.PlayerAdded:Connect(AdicionarESP)

local function ProfilerESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            local highlight = player.Character:FindFirstChild("CombatESP_Highlight")
            if highlight then highlight.Enabled = Config.EspChams end

            local head = player.Character:FindFirstChild("Head")
            if head then
                local nameGui = head:FindFirstChild("CombatESP_NameGui")
                if nameGui then nameGui.Enabled = Config.EspNames end
            end
        end
    end
end

-- [[ LÓGICA DE ALVO AVANÇADA ]]
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge
    
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        return nil, math.huge
    end

    local myPos = LocalPlayer.Character.HumanoidRootPart.Position

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local canTarget = true
            if Config.TeamCheck and player.Team == LocalPlayer.Team and player.Team ~= nil then
                canTarget = false
            end

            if canTarget and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
                if player.Character.Humanoid.Health > 0 then
                    local targetPos = player.Character.HumanoidRootPart.Position
                    local distance = (targetPos - myPos).Magnitude

                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end

    if Config.TargetInCars and LocalPlayer:FindFirstChild("CarrosPasta") then
        for _, car in ipairs(LocalPlayer.CarrosPasta:GetChildren()) do
            local humanoid = car:FindFirstChildOfClass("Humanoid", true)
            local rootPart = car:FindFirstChild("HumanoidRootPart", true) or car:FindFirstChild("DriveSeat", true)

            if humanoid and rootPart and humanoid.Health > 0 then
                local playerObj = Players:GetPlayerFromCharacter(humanoid.Parent)
                if playerObj and playerObj ~= LocalPlayer then
                    local canTargetCar = true
                    if Config.TeamCheck and playerObj.Team == LocalPlayer.Team and playerObj.Team ~= nil then
                        canTargetCar = false
                    end

                    if canTargetCar then
                        local distance = (rootPart.Position - myPos).Magnitude
                        if distance < shortestDistance then
                            shortestDistance = distance
                            closestPlayer = playerObj
                        end
                    end
                end
            end
        end
    end

    return closestPlayer, shortestDistance
end

-- [[ LOOP PRINCIPAL REVISADO PARA MIRAR NA CABEÇA ]]
RunService.RenderStepped:Connect(function()
    local target, distance = GetClosestPlayer()
    
    if target and target.Character then
        -- Tenta mirar na Cabeça (Head); Se não achar, puxa no tronco como contingência
        local targetPart = target.Character:FindFirstChild("Head") or target.Character:FindFirstChild("HumanoidRootPart")
        
        if targetPart and Config.SilentAimEnabled and distance <= Config.AimRange then
            local camera = workspace.CurrentCamera
            camera.CFrame = CFrame.new(camera.CFrame.Position, targetPart.Position)
        end
        
        if Config.KillAuraEnabled and distance <= Config.AuraRange then
            local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
            if tool then
                tool:Activate()
            end
        end
    end
end)

-- [[ INICIALIZAÇÃO DA UI ]]
local Menu = UiLib:CreateWindow("COMBAT - MINI PANEL")

Menu:CreateToggle("Ativar Aimbot", false, function(state)
    Config.SilentAimEnabled = state
end)

Menu:CreateSlider("Alcance do Aimbot (Studs)", 10, 500, 100, function(value)
    Config.AimRange = value
end)

Menu:CreateToggle("Filtro de Time (Team Check)", false, function(state)
    Config.TeamCheck = state
end)

Menu:CreateToggle("Mirar em Players nos Carros", false, function(state)
    Config.TargetInCars = state
end)

Menu:CreateToggle("ESP Contorno Corpo (Azul)", false, function(state)
    Config.EspChams = state
    ProfilerESP()
end)

Menu:CreateToggle("ESP Show Names", false, function(state)
    Config.EspNames = state
    ProfilerESP()
end)

Menu:CreateToggle("Ativar Kill Aura", false, function(state)
    Config.KillAuraEnabled = state
end)

Menu:CreateSlider("Alcance da Aura (Studs)", 10, 150, 50, function(value)
    Config.AuraRange = value
end)
