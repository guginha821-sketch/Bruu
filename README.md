-- Leaked by Munchkin Hub https://discord.gg/3vUhgE7PAw
-- Radiant Hub GUI - Complete Edition with Config System
local UserInputService = game:GetService('UserInputService')
local TweenService = game:GetService('TweenService')
local Players = game:GetService('Players')
local RunService = game:GetService('RunService')
local ContextActionService = game:GetService('ContextActionService')
local CoreGui = game:GetService('CoreGui')
local Stats = game:GetService('Stats')
local Workspace = game:GetService('Workspace')
local Lighting = game:GetService('Lighting')
local HttpService = game:GetService("HttpService")
local lp = Players.LocalPlayer

-- Colors
local AccentColor = Color3.fromRGB(0, 170, 0)
local TextDimColor = Color3.fromRGB(140, 140, 140)

-- ============ CONFIG SYSTEM ============
local FolderName = "RadiantHub"
local FileName = FolderName .. "/config.json"

local Config = {
    -- Toggles
    ["TP Down"] = false,
    ["DROP"] = false,
    ["Bat Aimbot"] = false,
    ["Auto Play"] = false,
    ["Medusa Counter"] = false,
    ["Auto Steal"] = false,
    ["Speed Checker"] = false,
    ["FOV Changer"] = false,
    ["FPS Boost"] = false,
    ["Dark Mode"] = false,
    ["Infinite Jump"] = false,
    ["No Walk Animation"] = false,
    ["Anti Ragdoll"] = false,
    ["Speed Customizer"] = false,
    ["Insta Reset"] = false,
    -- Minimized states
    ["AutoPlay Minimized"] = false,
    ["Speed Minimized"] = false,
    -- Values
    ["Steal Radius"] = 60,
    ["Steal Duration"] = 1.4,
    ["FOV Value"] = 70,
    ["Auto Play Speed"] = 60,
    ["Selected Side"] = "LEFT",
    ["Speed Value"] = 53,
    ["Steal Speed Value"] = 29,
    ["Lagger Speed Value"] = 10.1,
    ["Lagger Steal Value"] = 8,
    -- GUI Positions
    ["AutoPlay GUI X"] = 1,
    ["AutoPlay GUI OffsetX"] = -210,
    ["AutoPlay GUI Y"] = 0.5,
    ["AutoPlay GUI OffsetY"] = 50,
    ["Speed GUI X"] = 1,
    ["Speed GUI OffsetX"] = -210,
    ["Speed GUI Y"] = 0.5,
    ["Speed GUI OffsetY"] = -100,
}

local function LoadConfig()
    pcall(function()
        if not isfolder(FolderName) then
            makefolder(FolderName)
        end
        if isfile(FileName) then
            local data = readfile(FileName)
            local decoded = HttpService:JSONDecode(data)
            for key, value in pairs(decoded) do
                Config[key] = value
            end
        else
            writefile(FileName, HttpService:JSONEncode(Config))
        end
    end)
end

local function SaveConfig()
    pcall(function()
        if not isfolder(FolderName) then
            makefolder(FolderName)
        end
        writefile(FileName, HttpService:JSONEncode(Config))
    end)
end

LoadConfig()

-- ============ DRAG SYSTEM WITH DELAY ============
local function makeDraggableWithDelay(frame, onDragEnd)
    local dragging = false
    local dragInput = nil
    local dragStart = nil
    local startPos = nil
    local holdTime = 0
    local holding = false
    local holdConnection = nil
    
    local function startDrag(input)
        dragStart = input.Position
        startPos = frame.Position
        dragging = true
    end
    
    local function updateDrag(input)
        if dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            if onDragEnd then onDragEnd() end
        end
    end
    
    local function stopDrag()
        dragging = false
        if holdConnection then
            holdConnection:Disconnect()
            holdConnection = nil
        end
        holding = false
        holdTime = 0
    end
    
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            holding = true
            holdTime = 0
            if holdConnection then holdConnection:Disconnect() end
            holdConnection = RunService.Heartbeat:Connect(function()
                if holding then
                    holdTime = holdTime + RunService.Heartbeat:Wait()
                    if holdTime >= 0.5 then
                        startDrag(input)
                        if holdConnection then holdConnection:Disconnect() end
                        holdConnection = nil
                    end
                end
            end)
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    stopDrag()
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
            updateDrag(input)
        end
    end)
end

-- Create main GUI
local ScreenGui = Instance.new('ScreenGui')
ScreenGui.Name = 'RadiantHub'
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

-- ============ BANNER + PROGRESS BAR ============
local autoStealContainer = nil
local bannerFrame = nil
local stealProgressBarBg = nil
local stealProgressFill = nil
local stealPercentLabel = nil

local function createAutoStealUI()
    if autoStealContainer then return end
    
    autoStealContainer = Instance.new("Frame")
    autoStealContainer.Size = UDim2.new(0, 260, 0, 55)
    autoStealContainer.Position = UDim2.new(0.5, -130, 0, 80)
    autoStealContainer.BackgroundTransparency = 1
    autoStealContainer.Parent = ScreenGui
    
    bannerFrame = Instance.new("Frame")
    bannerFrame.Size = UDim2.new(1, 0, 0, 30)
    bannerFrame.Position = UDim2.new(0, 0, 0, 0)
    bannerFrame.BackgroundColor3 = Color3.fromRGB(11, 14, 20)
    bannerFrame.BackgroundTransparency = 0
    bannerFrame.Parent = autoStealContainer
    Instance.new("UICorner", bannerFrame).CornerRadius = UDim.new(0, 8)
    
    local bannerStroke = Instance.new("UIStroke", bannerFrame)
    bannerStroke.Color = AccentColor
    bannerStroke.Thickness = 1.5
    
    local bannerLabel = Instance.new("TextLabel")
    bannerLabel.Size = UDim2.new(1, 0, 1, 0)
    bannerLabel.BackgroundTransparency = 1
    bannerLabel.Font = Enum.Font.GothamBold
    bannerLabel.TextSize = 14
    bannerLabel.TextColor3 = AccentColor
    bannerLabel.Text = "Radiant Hub | Ping: 0ms | FPS: 0"
    bannerLabel.TextXAlignment = Enum.TextXAlignment.Center
    bannerLabel.Parent = bannerFrame
    
    stealProgressBarBg = Instance.new("Frame")
    stealProgressBarBg.Size = UDim2.new(1, 0, 0, 16)
    stealProgressBarBg.Position = UDim2.new(0, 0, 0, 32)
    stealProgressBarBg.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    stealProgressBarBg.BackgroundTransparency = 0.25
    stealProgressBarBg.Visible = true
    stealProgressBarBg.Parent = autoStealContainer
    Instance.new("UICorner", stealProgressBarBg).CornerRadius = UDim.new(0, 10)
    
    local progressBarStroke = Instance.new("UIStroke", stealProgressBarBg)
    progressBarStroke.Color = AccentColor
    progressBarStroke.Thickness = 1.5
    
    stealProgressFill = Instance.new("Frame")
    stealProgressFill.Size = UDim2.new(0, 0, 1, 0)
    stealProgressFill.BackgroundColor3 = AccentColor
    stealProgressFill.Parent = stealProgressBarBg
    Instance.new("UICorner", stealProgressFill).CornerRadius = UDim.new(0, 10)
    
    stealPercentLabel = Instance.new("TextLabel")
    stealPercentLabel.Size = UDim2.new(1, 0, 1, 0)
    stealPercentLabel.Position = UDim2.new(0, 0, 0, 52)
    stealPercentLabel.BackgroundTransparency = 1
    stealPercentLabel.Font = Enum.Font.GothamBold
    stealPercentLabel.TextSize = 11
    stealPercentLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
    stealPercentLabel.Text = "Stealing..."
    stealPercentLabel.Parent = autoStealContainer
    
    local fps = 60
    local framesCount = 0
    local last = tick()
    
    RunService.RenderStepped:Connect(function()
        framesCount = framesCount + 1
        if tick() - last >= 1 then
            fps = framesCount
            framesCount = 0
            last = tick()
        end
        
        local ping = 0
        local network = Stats:FindFirstChild("Network")
        if network and network:FindFirstChild("ServerStatsItem") then
            local dataPing = network.ServerStatsItem:FindFirstChild("Data Ping")
            if dataPing then ping = math.floor(dataPing:GetValue()) end
        end
        
        bannerLabel.Text = "Radiant Hub | Ping: " .. ping .. "ms | FPS: " .. fps
    end)
end

local function destroyAutoStealUI()
    if autoStealContainer then
        autoStealContainer:Destroy()
        autoStealContainer = nil
        bannerFrame = nil
        stealProgressBarBg = nil
        stealProgressFill = nil
        stealPercentLabel = nil
    end
end

-- Main Frame
local MainFrame = Instance.new('Frame')
MainFrame.Visible = true
MainFrame.BackgroundTransparency = 0.2
MainFrame.Position = UDim2.new(0.5, -170, 0.5, -190)
MainFrame.BackgroundColor3 = Color3.fromRGB(11, 14, 20)
MainFrame.Size = UDim2.new(0, 340, 0, 380)
MainFrame.Parent = ScreenGui
makeDraggableWithDelay(MainFrame)

local MainCorner = Instance.new('UICorner')
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new('UIStroke')
MainStroke.Color = AccentColor
MainStroke.Thickness = 2
MainStroke.Parent = MainFrame

-- Floating Toggle Button
local FloatingButton = Instance.new('TextButton')
FloatingButton.Size = UDim2.new(0, 58, 0, 58)
FloatingButton.Position = UDim2.new(1, -70, 0, 60)
FloatingButton.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
FloatingButton.Text = "âš™ï¸"
FloatingButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatingButton.Font = Enum.Font.GothamBold
FloatingButton.TextSize = 32
FloatingButton.Parent = ScreenGui
FloatingButton.Active = true
FloatingButton.Draggable = true

local FloatingCorner = Instance.new('UICorner')
FloatingCorner.CornerRadius = UDim.new(0, 12)
FloatingCorner.Parent = FloatingButton

local FloatingStroke = Instance.new('UIStroke')
FloatingStroke.Color = AccentColor
FloatingStroke.Thickness = 1.5
FloatingStroke.Parent = FloatingButton

-- Menu Header
local MenuHeader = Instance.new('Frame')
MenuHeader.BackgroundTransparency = 1
MenuHeader.Size = UDim2.new(1, 0, 0, 45)
MenuHeader.Parent = MainFrame

local MenuTitle = Instance.new('TextLabel')
MenuTitle.TextColor3 = AccentColor
MenuTitle.Text = 'Radiant Hub'
MenuTitle.Font = Enum.Font.GothamBold
MenuTitle.BackgroundTransparency = 1
MenuTitle.Position = UDim2.new(0.05, 0, 0, 0)
MenuTitle.TextXAlignment = Enum.TextXAlignment.Left
MenuTitle.TextSize = 18
MenuTitle.Size = UDim2.new(0.5, 0, 1, 0)
MenuTitle.Parent = MenuHeader

-- Tab Bar
local TabBar = Instance.new('Frame')
TabBar.BackgroundColor3 = Color3.fromRGB(15, 20, 28)
TabBar.Position = UDim2.new(0.04, 0, 0.13, 0)
TabBar.Size = UDim2.new(0.92, 0, 0, 30)
TabBar.Parent = MainFrame

local TabCorner = Instance.new('UICorner')
TabCorner.CornerRadius = UDim.new(0, 6)
TabCorner.Parent = TabBar

local TabLayout = Instance.new('UIListLayout')
TabLayout.FillDirection = Enum.FillDirection.Horizontal
TabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
TabLayout.Parent = TabBar

-- Content Container
local ContentContainer = Instance.new('Frame')
ContentContainer.Position = UDim2.new(0, 0, 0.22, 0)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Size = UDim2.new(1, 0, 0.75, 0)
ContentContainer.Parent = MainFrame

-- Create tabs
local Tabs = {}

local function CreateTab(Name)
    local TabFrame = Instance.new('ScrollingFrame')
    TabFrame.Name = Name .. "Tab"
    TabFrame.Visible = false
    TabFrame.BackgroundTransparency = 1
    TabFrame.ScrollBarThickness = 6
    TabFrame.ScrollBarImageColor3 = AccentColor
    TabFrame.Size = UDim2.new(1, 0, 1, 0)
    TabFrame.Parent = ContentContainer
    
    local UIListLayout = Instance.new('UIListLayout')
    UIListLayout.Padding = UDim.new(0, 8)
    UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    UIListLayout.Parent = TabFrame
    
    UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        TabFrame.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 10)
    end)
    
    return TabFrame
end

local function CreateToggle(parent, text, callback)
    local container = Instance.new('Frame')
    container.Size = UDim2.new(0.92, 0, 0, 40)
    container.BackgroundColor3 = Color3.fromRGB(15, 20, 28)
    container.BackgroundTransparency = 0.5
    container.Parent = parent
    Instance.new('UICorner', container).CornerRadius = UDim.new(0, 8)
    
    local label = Instance.new('TextLabel')
    label.Size = UDim2.new(0.6, 0, 1, 0)
    label.Position = UDim2.new(0, 12, 0, 0)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.TextSize = 13
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Text = text
    label.Parent = container
    
    local toggleBtn = Instance.new('TextButton')
    toggleBtn.Size = UDim2.new(0, 50, 0, 24)
    toggleBtn.Position = UDim2.new(1, -60, 0.5, -12)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    toggleBtn.Text = ""
    toggleBtn.Parent = container
    Instance.new('UICorner', toggleBtn).CornerRadius = UDim.new(1, 0)
    
    local circle = Instance.new('Frame')
    circle.Size = UDim2.new(0, 20, 0, 20)
    circle.Position = UDim2.new(0, 2, 0.5, -10)
    circle.BackgroundColor3 = Color3.fromRGB(220, 220, 220)
    circle.Parent = toggleBtn
    Instance.new('UICorner', circle).CornerRadius = UDim.new(1, 0)
    
    local enabled = Config[text] or false
    
    if enabled then
        toggleBtn.BackgroundColor3 = AccentColor
        circle.Position = UDim2.new(1, -22, 0.5, -10)
    end
    
    toggleBtn.MouseButton1Click:Connect(function()
        enabled = not enabled
        Config[text] = enabled
        SaveConfig()
        
        if enabled then
            TweenService:Create(toggleBtn, TweenInfo.new(0.2), {BackgroundColor3 = AccentColor}):Play()
            TweenService:Create(circle, TweenInfo.new(0.2), {Position = UDim2.new(1, -22, 0.5, -10)}):Play()
        else
            TweenService:Create(toggleBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50, 50, 50)}):Play()
            TweenService:Create(circle, TweenInfo.new(0.2), {Position = UDim2.new(0, 2, 0.5, -10)}):Play()
        end
        if callback then callback(enabled) end
    end)
    
    if enabled and callback then callback(enabled) end
    
    return {container = container, setEnabled = function(state)
        enabled = state
        Config[text] = state
        SaveConfig()
        if enabled then
            toggleBtn.BackgroundColor3 = AccentColor
            circle.Position = UDim2.new(1, -22, 0.5, -10)
        else
            toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            circle.Position = UDim2.new(0, 2, 0.5, -10)
        end
        if callback then callback(enabled) end
    end, getEnabled = function() return enabled end}
end

local function CreateInputBox(parent, labelText, defaultValue, minVal, maxVal, callback)
    local container = Instance.new('Frame')
    container.Size = UDim2.new(0.92, 0, 0, 40)
    container.BackgroundColor3 = Color3.fromRGB(15, 20, 28)
    container.BackgroundTransparency = 0.5
    container.Parent = parent
    Instance.new('UICorner', container).CornerRadius = UDim.new(0, 8)
    
    local label = Instance.new('TextLabel')
    label.Size = UDim2.new(0.5, 0, 1, 0)
    label.Position = UDim2.new(0, 12, 0, 0)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.GothamBold
    label.TextSize = 13
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Text = labelText
    label.Parent = container
    
    local inputBox = Instance.new('TextBox')
    inputBox.Size = UDim2.new(0.35, 0, 0, 30)
    inputBox.Position = UDim2.new(0.63, 0, 0.5, -15)
    inputBox.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    inputBox.Text = tostring(Config[labelText] or defaultValue)
    inputBox.Font = Enum.Font.GothamBold
    inputBox.TextSize = 13
    inputBox.ClearTextOnFocus = false
    inputBox.Parent = container
    Instance.new('UICorner', inputBox).CornerRadius = UDim.new(0, 6)
    local stroke = Instance.new('UIStroke', inputBox)
    stroke.Color = AccentColor
    
    inputBox.FocusLost:Connect(function()
        local num = tonumber(inputBox.Text) or defaultValue
        num = math.clamp(num, minVal, maxVal)
        inputBox.Text = tostring(num)
        Config[labelText] = num
        SaveConfig()
        if callback then callback(num) end
    end)
    
    return inputBox
end

-- Create tabs
Tabs.Combat = CreateTab("Combat")
Tabs.Movement = CreateTab("Movement")
Tabs.Visual = CreateTab("Visual")
Tabs.Settings = CreateTab("Settings")

-- ============ TP DOWN FEATURE ============
local tpDownGui = nil
local tpDownButton = nil

local function doTpDown()
    pcall(function()
        local c = lp.Character
        if not c then return end
        local root = c:FindFirstChild("HumanoidRootPart")
        if not root then return end
        
        local rp = RaycastParams.new()
        rp.FilterDescendantsInstances = {c}
        rp.FilterType = Enum.RaycastFilterType.Exclude
        
        local res = workspace:Raycast(root.Position, Vector3.new(0, -1000, 0), rp)
        
        if res then 
            local newPos = res.Position + Vector3.new(0, root.Size.Y/2 + 0.5, 0)
            root.CFrame = CFrame.new(newPos)
            root.AssemblyLinearVelocity = Vector3.zero
        end
    end)
end

local function createTpDownButton()
    if tpDownGui then return end
    
    tpDownGui = Instance.new("ScreenGui")
    tpDownGui.Name = "TpDownButtonGui"
    tpDownGui.ResetOnSpawn = false
    tpDownGui.Parent = CoreGui
    
    tpDownButton = Instance.new("TextButton")
    tpDownButton.Size = UDim2.new(0, 140, 0, 50)
    tpDownButton.Position = UDim2.new(1, -155, 0, 120)
    tpDownButton.Text = "TP DOWN"
    tpDownButton.Font = Enum.Font.GothamBold
    tpDownButton.TextSize = 16
    tpDownButton.TextColor3 = Color3.new(1, 1, 1)
    tpDownButton.BackgroundColor3 = AccentColor
    tpDownButton.Active = true
    tpDownButton.Draggable = true
    tpDownButton.Parent = tpDownGui
    Instance.new("UICorner", tpDownButton).CornerRadius = UDim.new(0, 16)
    
    tpDownButton.MouseButton1Click:Connect(function()
        doTpDown()
        tpDownButton.BackgroundColor3 = Color3.fromRGB(200, 100, 0)
        task.wait(0.2)
        tpDownButton.BackgroundColor3 = AccentColor
    end)
end

local function destroyTpDownButton()
    if tpDownGui then
        tpDownGui:Destroy()
        tpDownGui = nil
        tpDownButton = nil
    end
end

-- ============ DROP FEATURE ============
local dropConnections = {}
local AUTO_OFF_DELAY = 0.15
local dropGui = nil
local dropButton = nil
local dropEnabled = false

local function stopDrop()
    dropEnabled = false
    for _, conn in ipairs(dropConnections) do
        pcall(function() conn:Disconnect() end)
    end
    dropConnections = {}
end

local function runDrop()
    if dropEnabled then return end
    dropEnabled = true
    
    local colConn = RunService.Stepped:Connect(function()
        if not dropEnabled then return end
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= lp and player.Character then
                for _, part in ipairs(player.Character:GetChildren()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end
    end)
    table.insert(dropConnection
