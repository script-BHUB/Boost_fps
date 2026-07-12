local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local FrameCorner = Instance.new("UICorner")
local FrameStroke = Instance.new("UIStroke")

local UIListLayout = Instance.new("UIListLayout")
local StatLabel = Instance.new("TextLabel")

local UltraFastButton = Instance.new("TextButton")
local UltraFastCorner = Instance.new("UICorner")
local UltraFastStroke = Instance.new("UIStroke")

pcall(function()
    ScreenGui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
    ScreenGui.ResetOnSpawn = false
end)

MainFrame.Name = "FPSMainFrameV137"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 15)
MainFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
MainFrame.Size = UDim2.new(0, 180, 0, 95)
MainFrame.Active = true
MainFrame.ZIndex = 10

FrameCorner.CornerRadius = UDim.new(0, 14)
FrameCorner.Parent = MainFrame

FrameStroke.Color = Color3.fromRGB(35, 35, 45)
FrameStroke.Thickness = 2.5
FrameStroke.Parent = MainFrame

UIListLayout.Parent = MainFrame
UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 0)

local UIPadding = Instance.new("UIPadding")
UIPadding.Parent = MainFrame
UIPadding.PaddingTop = UDim.new(0, 18)

local Color_Off = Color3.fromRGB(22, 22, 26)
local Color_On = Color3.fromRGB(0, 180, 130)
local TextColor_Off = Color3.fromRGB(160, 160, 170)
local TextColor_On = Color3.fromRGB(255, 255, 255)

UltraFastButton.Parent = MainFrame
UltraFastButton.BackgroundColor3 = Color_Off
UltraFastButton.Size = UDim2.new(0, 144, 0, 56)
UltraFastButton.Font = Enum.Font.GothamBold
UltraFastButton.Text = "Boost FPS: Off"
UltraFastButton.TextColor3 = TextColor_Off
UltraFastButton.TextSize = 14
UltraFastButton.ZIndex = 12
UltraFastCorner.CornerRadius = UDim.new(0, 10)
UltraFastCorner.Parent = UltraFastButton
UltraFastStroke.Color = Color3.fromRGB(40, 40, 50)
UltraFastStroke.Thickness = 1.5
UltraFastStroke.Parent = UltraFastButton

StatLabel.Parent = ScreenGui
StatLabel.BackgroundTransparency = 0.4
StatLabel.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
StatLabel.Position = UDim2.new(0.5, -110, 0, 5)
StatLabel.Size = UDim2.new(0, 220, 0, 30)
StatLabel.Font = Enum.Font.Code
StatLabel.TextColor3 = Color3.fromRGB(0, 255, 180)
StatLabel.TextSize = 15
StatLabel.TextXAlignment = Enum.TextXAlignment.Center
StatLabel.Visible = false
local StatCorner = Instance.new("UICorner")
StatCorner.CornerRadius = UDim.new(0, 6)
StatCorner.Parent = StatLabel

local isUltraFastEnabled = false
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")

local originalStates = {}
local isBladeBall = false

local function checkGameType()
    if workspace:FindFirstChild("Balls") or game:GetService("ReplicatedStorage"):FindFirstChild("Remotes") or workspace:FindFirstChild("Alive") then
        isBladeBall = true
    else
        isBladeBall = false
    end
end

local function checkIsBall(obj)
    if not obj then return false end
    local current = obj
    while current and current ~= workspace and current ~= game do
        local name = current.Name:lower()
        if name:find("ball") or name:find("volley") or name:find("sphere") or current:IsA("BallInning") then
            return true
        end
        current = current.Parent
    end
    return false
end

local function isEssentialObject(v)
    if checkIsBall(v) then return true end
    
    if v:IsDescendantOf(localPlayer.Character) then return true end
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character and v:IsDescendantOf(player.Character) then
            return true
        end
    end
    
    local name = v.Name:lower()
    if name:find("court") or name:find("net") or name:find("floor") or name:find("line") or name:find("spawn") or name:find("arena") or name:find("plate") or name:find("base") or name:find("ground") or name:find("stadium") then
        return true
    end
    
    return false
end

local function processPart(v)
    if not v or not v:IsDescendantOf(workspace) then return end
    if isEssentialObject(v) then return end

    if v:IsA("BasePart") and v.Transparency == 1 and v.Name == "Part" then
        if not originalStates[v] then originalStates[v] = { Parent = v.Parent } end
        v.Parent = nil
        return
    end

    if v:IsA("ParticleEmitter") or v:IsA("Fire") or v:IsA("Smoke") or v:IsA("Sparkles") then
        if not originalStates[v] then 
            originalStates[v] = { Enabled = v.Enabled, Rate = v:IsA("ParticleEmitter") and v.Rate or nil, Speed = v:IsA("ParticleEmitter") and v.Speed or nil } 
        end
        v.Enabled = false
        if v:IsA("ParticleEmitter") then
            v.Rate = 0
            v.Speed = NumberRange.new(0, 0)
        end
    end

    if v:IsA("Trail") or v:IsA("Beam") or v:IsA("Highlight") or v:IsA("SelectionBox") then
        if not originalStates[v] then originalStates[v] = { Enabled = v.Enabled } end
        v.Enabled = false
    end

    if v:IsA("BasePart") or v:IsA("MeshPart") then
        if not originalStates[v] then 
            originalStates[v] = { Material = v.Material, CastShadow = v.CastShadow, Reflectance = v.Reflectance, TextureId = v:IsA("MeshPart") and v.TextureId or nil } 
        end
        v.Material = Enum.Material.SmoothPlastic
        v.CastShadow = false
        v.Reflectance = 0
        
        if not isBladeBall and v:IsA("MeshPart") and v.TextureId ~= "" then 
            v.TextureId = "" 
        end
    end
    
    if not isBladeBall then
        if v:IsA("SpecialMesh") and v.TextureId ~= "" then
            if not originalStates[v] then originalStates[v] = { TextureId = v.TextureId } end
            v.TextureId = ""
        end
        if v:IsA("Decal") or v:IsA("Texture") then
            if not originalStates[v] then originalStates[v] = { Transparency = v.Transparency } end
            v.Transparency = 1
        end
    end
end

local function runInstantPurge()
    checkGameType()
    local allObjects = workspace:GetDescendants()
    for i = 1, #allObjects do
        pcall(processPart, allObjects[i])
    end
end

local function makeDraggable(frame)
    pcall(function()
        local dragging, dragInput, dragStart, startPos
        frame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true; dragStart = input.Position; startPos = frame.Position
                input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
            end
        end)
        frame.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
        end)
        game:GetService("UserInputService").InputChanged:Connect(function(input)
            if input == dragInput and dragging then
                local delta = input.Position - dragStart
                frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end)
    end)
end
makeDraggable(MainFrame)

UltraFastButton.MouseButton1Click:Connect(function()
    isUltraFastEnabled = not isUltraFastEnabled
    
    if isUltraFastEnabled then
        UltraFastButton.Text = "Boost FPS: On"
        UltraFastButton.BackgroundColor3 = Color_On
        UltraFastButton.TextColor3 = TextColor_On
        UltraFastStroke.Color = Color3.fromRGB(0, 255, 180)
        StatLabel.Visible = true
        
        if setfpscap then pcall(function() setfpscap(360) end) end
        
        Lighting.GlobalShadows = false
        Lighting.OutdoorAmbient = Color3.fromRGB(140, 140, 140)
        Lighting.Ambient = Color3.fromRGB(140, 140, 140)
        
        pcall(function()
            for _, effect in pairs(Lighting:GetChildren()) do
                if effect:IsA("PostEffect") or effect:IsA("Atmosphere") or effect:IsA("Sky") or effect:IsA("Clouds") or effect:IsA("SunRaysEffect") or effect:IsA("BloomEffect") or effect:IsA("BlurEffect") or effect:IsA("ColorCorrectionEffect") then
                    if not originalStates[effect] then originalStates[effect] = { Parent = effect.Parent } end
                    effect.Parent = nil
                end
            end
        end)
        
        runInstantPurge()
    else
        UltraFastButton.Text = "Boost FPS: Off"
        UltraFastButton.BackgroundColor3 = Color_Off
        UltraFastButton.TextColor3 = TextColor_Off
        UltraFastStroke.Color = Color3.fromRGB(40, 40, 50)
        StatLabel.Visible = false
        
        if setfpscap then pcall(function() setfpscap(60) end) end
        
        for obj, state in pairs(originalStates) do
            pcall(function()
                if obj then
                    if state.Parent then obj.Parent = state.Parent end
                    if state.Material then obj.Material = state.Material end
                    if state.CastShadow ~= nil then obj.CastShadow = state.CastShadow end
                    if state.Reflectance then obj.Reflectance = state.Reflectance end
                    if state.TextureId then obj.TextureId = state.TextureId end
                    if state.Transparency then obj.Transparency = state.Transparency end
                    if state.Enabled ~= nil then obj.Enabled = state.Enabled end
                    if state.Rate and obj:IsA("ParticleEmitter") then obj.Rate = state.Rate end
                    if state.Speed and obj:IsA("ParticleEmitter") then obj.Speed = state.Speed end
                end
            end)
        end
        originalStates = {}
    end
end)

task.spawn(function()
    local lastTime = os.clock()
    local frameCount = 0
    RunService.Heartbeat:Connect(function()
        if StatLabel.Visible then
            frameCount = frameCount + 1
            local currentTime = os.clock()
            if currentTime - lastTime >= 0.5 then
                local fps = math.floor(frameCount / (currentTime - lastTime))
                frameCount = 0; lastTime = currentTime
                local ping = 0
                pcall(function() ping = math.floor(localPlayer:GetNetworkPing() * 1000) end)
                StatLabel.Text = string.format("FPS: %d  |  PING: %d ms", fps, ping)
            end
        end
    end)
end)
