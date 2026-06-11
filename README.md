local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- ==========================================
-- CONFIGURATION & VARIABLES
-- ==========================================
local Config = {
    Aimbot = {
        Enabled = true,
        ToggleKey = Enum.KeyCode.V, 
        Speed = 0.2,                -- Lerp Factor (0-1)
        Smoothness = 0.1            -- Additional smoothing factor for CFrame
    },
    ESP = {
        Enabled = true,
        ToggleKey = Enum.KeyCode.E,
        Color = Color3.fromRGB(0, 255, 0),
        Thickness = 1,
        ShowHealth = false,
        ShowDistance = true,
    },
    Triggerbot = {
        Enabled = true,
        ToggleKey = Enum.KeyCode.F,
        Range = 50,                 -- Raycast range (meters)
        Delay = 0.1,                -- Seconds delay before firing
        FireOnRange = true,         -- Auto-fire when in range
    }
}

-- UI References
local MainGui = nil
local AimFrame = nil
local ESPFrame = nil
local TriggerFrame = nil

-- State Variables
local CurrentTargetPart = "Head"   -- Target Part Name (e.g., "Head", "Torso")
local IsAimOn = false
local IsTriggerOn = false
local LastFireTime = 0
local espFrames = {} 

-- ==========================================
-- UI CREATION
-- ==========================================
function CreateUI()
    MainGui = Instance.new("ScreenGui")
    MainGui.Name = "ExploitMenu"
    MainGui.Parent = LocalPlayer
    
    local Background = Instance.new("Frame")
    Background.Name = "Background"
    Background.Size = UDim2.fromScale(0.45, 1)
    Background.Position = UDim2.fromOffset(-50, -50)
    Background.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Background.BorderSizePixel = 0
    Background.Parent = MainGui

    local Title = Instance.new("TextLabel")
    Title.Name = "Title"
    Title.Size = UDim2.fromScale(1, 0.15)
    Title.Position = UDim2.fromOffset(0, -50)
    Title.BackgroundTransparency = 1
    Title.Text = "EXPLOIT MENU"
    Title.Font = Enum.Font.GothamBold
    Title.Size = UDim2.fromScale(1, 0.15)
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Parent = Background

    -- Helper to create sections with consistent layout
    local function CreateSection(name, yPos)
        local Section = Instance.new("Frame")
        Section.Name = name .. "Section"
        Section.Size = UDim2.fromScale(1, 0.35)
        Section.Position = UDim2.fromOffset(0, yPos)
        Section.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        Section.BorderSizePixel = 0
        Section.Parent = Background
        
        local Header = Instance.new("TextLabel")
        Header.Name = "Header"
        Header.Size = UDim2.fromScale(1, 0.15)
        Header.Position = UDim2.fromOffset(0, -50)
        Header.BackgroundTransparency = 1
        Header.Text = name
        Header.Font = Enum.Font.GothamBold
        Header.TextColor3 = Color3.fromRGB(255, 255, 255)
        Header.Parent = Section

        return Section
    end

    -- Aimbot Section
    AimFrame = CreateSection("Aimbot", -100)
    
    local AimToggleBtn = Instance.new("TextButton")
    AimToggleBtn.Name = "Toggle"
    AimToggleBtn.Size = UDim2.fromScale(1, 0.15)
    AimToggleBtn.Position = UDim2.fromOffset(0, -50)
    AimToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    AimToggleBtn.Text = "ON"
    AimToggleBtn.Font = Enum.Font.GothamBold
    AimToggleBtn.Parent = AimFrame
    
    -- Connect Toggle Button Event
    AimToggleBtn.MouseButton1Click:Connect(function()
        IsAimOn = not IsAimOn
        AimToggleBtn.Text = IsAimOn and "ON" or "OFF"
        AimToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- Reset color
    end)

    local SpeedContainer = Instance.new("Frame")
    SpeedContainer.Name = "Speed"
    SpeedContainer.Size = UDim2.fromScale(1, 0.3)
    SpeedContainer.Position = UDim2.fromOffset(0, -50)
    SpeedContainer.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    SpeedContainer.Parent = AimFrame

    local SpeedLabel = Instance.new("TextLabel")
    SpeedLabel.Name = "SpeedLabel"
    SpeedLabel.Size = UDim2.fromScale(1, 0.15)
    SpeedLabel.Position = UDim2.fromOffset(0, -50)
    SpeedLabel.BackgroundTransparency = 1
    SpeedLabel.Text = "Aimbot Speed: 0.2"
    SpeedLabel.Font = Enum.Font.GothamBold
    SpeedLabel.Parent = SpeedContainer

    local SliderBg = Instance.new("Frame")
    SliderBg.Name = "SliderBg"
    SliderBg.Size = UDim2.fromScale(1, 0.7)
    SliderBg.Position = UDim2.fromOffset(0, -50)
    SliderBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    SliderBg.Parent = SpeedContainer

    local SliderFill = Instance.new("Frame")
    -- Initial Size based on Config (0.2 / 1.0 = 20%)
    SliderFill.Size = UDim2.fromScale(0.2, 1) 
    SliderFill.Position = UDim2.fromOffset(-50, -50)
    SliderFill.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    SliderFill.Parent = SliderBg

    local SpeedValue = Instance.new("TextLabel")
    SpeedValue.Name = "SpeedValue"
    SpeedValue.Size = UDim2.fromScale(1, 0.1)
    SpeedValue.Position = UDim2.fromOffset(0, -50)
    SpeedValue.BackgroundTransparency = 1
    SpeedValue.Text = "0.2"
    SpeedValue.Font = Enum.Font.GothamBold
    SpeedValue.Parent = SliderBg

    -- Connect Slider Event (Update Config & UI Text)
    local function OnSliderChanged()
        local NewSpeed = math.clamp(SliderFill.Size.X.Scale, 0.1, 1.0) * 1.0
        Config.Aimbot.Speed = NewSpeed
        
        SpeedValue.Text = tostring(NewSpeed)
        SpeedLabel.Text = "Aimbot Speed: " .. tostring(NewSpeed)
        
        -- Update Fill Size
        SliderFill.Size = UDim2.fromScale(NewSpeed, 1) 
    end
    
    SliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            local MousePos = input.Position
            local RelativePos = SliderBg.AbsolutePosition + (MousePos - SliderBg.AbsolutePosition)
            
            -- Simple slider logic for demo purposes
            local NewScale = math.clamp((RelativePos.X / SliderBg.AbsoluteSize.X), 0.1, 1.0) * 1.0
            OnSliderChanged()
        end)

    -- ESP Section
    ESPFrame = CreateSection("ESP", 20)
    
    local ESPColorPicker = Instance.new("TextButton")
    ESPColorPicker.Name = "Color"
    ESPColorPicker.Size = UDim2.fromScale(1, 0.3)
    ESPColorPicker.Position = UDim2.fromOffset(0, -50)
    ESPColorPicker.BackgroundColor3 = Config.ESP.Color
    ESPColorPicker.Text = "Pick Color (Click)"
    ESPColorPicker.Font = Enum.Font.GothamBold
    ESPColorPicker.Parent = ESPFrame

    local ThicknessContainer = Instance.new("Frame")
    ThicknessContainer.Name = "Thickness"
    ThicknessContainer.Size = UDim2.fromScale(1, 0.3)
    ThicknessContainer.Position = UDim2.fromOffset(0, -50)
    ThicknessContainer.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    ThicknessContainer.Parent = ESPFrame

    local ThicknessLabel = Instance.new("TextLabel")
    ThicknessLabel.Name = "ThicknessLabel"
    ThicknessLabel.Size = UDim2.fromScale(1, 0.15)
    ThicknessLabel.Position = UDim2.fromOffset(0, -50)
    ThicknessLabel.BackgroundTransparency = 1
    ThicknessLabel.Text = "ESP Thickness: 1"
    ThicknessLabel.Font = Enum.Font.GothamBold
    ThicknessLabel.Parent = ThicknessContainer

    local ThicknessSliderBg = Instance.new("Frame")
    ThicknessSliderBg.Name = "ThicknessBg"
    ThicknessSliderBg.Size = UDim2.fromScale(1, 0.7)
    ThicknessSliderBg.Position = UDim2.fromOffset(0, -50)
    ThicknessSliderBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    ThicknessSliderBg.Parent = ThicknessContainer

    local ThicknessFill = Instance.new("Frame")
    ThicknessFill.Name = "ThicknessFill"
    ThicknessFill.Size = UDim2.fromScale(1, 1) -- Full width initially
    ThicknessFill.Position = UDim2.fromOffset(-50, -50)
    ThicknessFill.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    ThicknessFill.Parent = ThicknessSliderBg

    local ThicknessValue = Instance.new("TextLabel")
    ThicknessValue.Name = "ThicknessValue"
    ThicknessValue.Size = UDim2.fromScale(1, 0.1)
    ThicknessValue.Position = UDim2.fromOffset(0, -50)
    ThicknessValue.BackgroundTransparency = 1
    ThicknessValue.Text = "1"
    ThicknessValue.Font = Enum.Font.GothamBold
    ThicknessValue.Parent = ThicknessSliderBg

    -- Triggerbot Section
    TriggerFrame = CreateSection("Triggerbot", 60)
    
    local TriggerToggleBtn = Instance.new("TextButton")
    TriggerToggleBtn.Name = "Toggle"
    TriggerToggleBtn.Size = UDim2.fromScale(1, 0.15)
    TriggerToggleBtn.Position = UDim2.fromOffset(0, -50)
    TriggerToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    TriggerToggleBtn.Text = "ON"
    TriggerToggleBtn.Font = Enum.Font.GothamBold
    TriggerToggleBtn.Parent = TriggerFrame

    local DelayContainer = Instance.new("Frame")
    DelayContainer.Name = "Delay"
    DelayContainer.Size = UDim2.fromScale(1, 0.3)
    DelayContainer.Position = UDim2.fromOffset(0, -50)
    DelayContainer.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    DelayContainer.Parent = TriggerFrame

    local DelayLabel = Instance.new("TextLabel")
    DelayLabel.Name = "DelayLabel"
    DelayLabel.Size = UDim2.fromScale(1, 0.15)
    DelayLabel.Position = UDim2.fromOffset(0, -50)
    DelayLabel.BackgroundTransparency = 1
    DelayLabel.Text = "Fire Delay: 0.1s"
    DelayLabel.Font = Enum.Font.GothamBold
    DelayLabel.Parent = DelayContainer

    local DelaySliderBg = Instance.new("Frame")
    DelaySliderBg.Name = "DelayBg"
    DelaySliderBg.Size = UDim2.fromScale(1, 0.7)
    DelaySliderBg.Position = UDim2.fromOffset(0, -50)
    DelaySliderBg.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    DelaySliderBg.Parent = DelayContainer

    local DelayFill = Instance.new("Frame")
    DelayFill.Name = "DelayFill"
    DelayFill.Size = UDim2.fromScale(1, 1) -- Full width initially
    DelayFill.Position = UDim2.fromOffset(-50, -50)
    DelayFill.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    DelayFill.Parent = DelaySliderBg

    local DelayValue = Instance.new("TextLabel")
    DelayValue.Name = "DelayValue"
    DelayValue.Size = UDim2.fromScale(1, 0.1)
    DelayValue.Position = UDim2.fromOffset(0, -50)
    DelayValue.BackgroundTransparency = 1
    DelayValue.Text = "0.1"
    DelayValue.Font = Enum.Font.GothamBold
    DelayValue.Parent = DelaySliderBg

    -- Connect Triggerbot Slider (Update Config & UI Text)
    local function OnTriggerDelayChanged()
        local NewDelay = math.clamp(DelayFill.Size.X.Scale, 0.1, 2.0) * 0.1
        Config.Triggerbot.Delay = NewDelay
        
        DelayValue.Text = tostring(NewDelay)
        DelayLabel.Text = "Fire Delay: " .. tostring(NewDelay) .. "s"
        
        -- Update Fill Size
        DelayFill.Size = UDim2.fromScale(NewDelay, 1) 
    end
    
    DelaySliderBg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            local MousePos = input.Position
            local RelativePos = DelaySliderBg.AbsolutePosition + (MousePos - DelaySliderBg.AbsolutePosition)
            
            -- Simple slider logic for demo purposes
            local NewScale = math.clamp((RelativePos.X / DelaySliderBg.AbsoluteSize.X), 0.1, 2.0) * 0.1
            OnTriggerDelayChanged()
        end)

    -- ==========================================
    -- MAIN GAME LOGIC (RunService Loop)
    -- ==========================================
    
    local function GetNearestEnemy()
        local minDist = math.huge
        local targetPart = nil
        
        for _, player in ipairs(Players:GetChildren()) do
            if player ~= LocalPlayer and not player:IsLocal() then
                local char = player.Character
                if char and char:FindFirstChild("Head") then
                    local dist = (char.Head.Position - Camera.CFrame.Position).Magnitude
                    if dist < minDist then
                        minDist = dist
                        targetPart = char.Head
                    end
                elseif char and char:FindFirstChild("Torso") then
                    local dist = (char.Torso.Position - Camera.CFrame.Position).Magnitude
                    if dist < minDist then
                        minDist = dist
                        targetPart = char.Torso
                    end
                end
            end
        end
        
        return targetPart, minDist
    end

    -- Aimbot Logic
    local function OnRenderStepped()
        if Config.Aimbot.Enabled and IsAimOn then
            local targetPart, _ = GetNearestEnemy()
            
            if targetPart then
                local aimCFrame = Camera.CFrame.LookAt(targetPart)
                local currentCFrame = LocalPlayer.Character.HumanoidRootPart.CFrame
                
                -- Lerp Logic for Smooth Aimbot
                currentCFrame = currentCFrame:Lerp(aimCFrame, Config.Aimbot.Speed)
                
                if LocalPlayer.Character then
                    LocalPlayer.Character.HumanoidRootPart.CFrame = currentCFrame
                end
            end
        end

        -- ESP Logic (Draws boxes around enemies)
        if Config.ESP.Enabled and IsAimOn or true then -- Always run ESP loop if enabled
            local espColor = Config.ESP.Color
            
            for _, player in ipairs(Players:GetChildren()) do
                if player ~= LocalPlayer and not player:IsLocal() then
                    local char = player.Character
                    if char and char:FindFirstChild("Head") then
                        -- Create ESP Frame (or update existing)
                        local headPos = char.Head.Position
                        
                        -- Simple 2D Projection for ScreenGui
                        local screenPoint = Camera.WorldToScreenPoint(headPos)
                        
                        if screenPoint then
                            -- Update or Create ESP Frame
                            local espFrame = nil
                            
                            -- Check if we already have an ESP frame for this player
                            for _, existingEsp in ipairs(espFrames) do
                                if existingEsp.Name == "ESP_" .. tostring(player.UserId) then
                                    espFrame = existingEsp
                                    break
                                end
                            end

                            if not espFrame then
                                -- Create new ESP Frame
                                espFrame = Instance.new("Frame")
                                espFrame.Name = "ESP_" .. tostring(player.UserId)
                                espFrame.Size = UDim2.fromScale(0.1, 0.5)
                                espFrame.Position = UDim2.fromOffset(screenPoint.X - 50, screenPoint.Y - 50)
                                espFrame.BackgroundColor3 = espColor
                                espFrame.BorderSizePixel = Config.ESP.Thickness
                                espFrame.Parent = MainGui
                            end

                            -- Update ESP Frame Position/Size
                            espFrame.Position = UDim2.fromOffset(screenPoint.X - 50, screenPoint.Y - 50)
                            
                            if Config.ESP.ShowHealth then
                                local health = char:FindFirstChild("Humanoid") and char.Humanoid.Health or 100
                                espFrame.Size = UDim2.fromScale(0.1, (health / 100) * 0.5)
                            end

                            if Config.ESP.ShowDistance then
                                local dist = (char.Head.Position - Camera.CFrame.Position).Magnitude
                                espFrame.Text = tostring(math.floor(dist)) .. "m"
                                espFrame.Size = UDim2.fromScale(0.1, 0.6)
                            end

                            -- Store in list for cleanup later
                            table.insert(espFrames, espFrame)
                        end
                    end
                end
            end
            
            -- Cleanup old ESP frames if player died or moved out of view (Simple cleanup)
            local currentEspCount = 0
            for _, existingEsp in ipairs(espFrames) do
                if existingEsp.Parent == MainGui then
                    currentEspCount += 1
                else
                    existingEsp:Destroy()
                end
            end
            
            -- If we have fewer frames than expected, remove extras (Simple memory management)
            for i = #espFrames, 1, -1 do
                if espFrames[i].Parent ~= MainGui then
                    table.remove(espFrames, i)
                else
                    break
                end
            end
        end

        -- Triggerbot Logic
        if Config.Triggerbot.Enabled and IsTriggerOn then
            local targetPart, dist = GetNearestEnemy()
            
            if targetPart and dist <= Config.Triggerbot.Range then
                local currentTime = tick()
                
                if (currentTime - LastFireTime) >= Config.Triggerbot.Delay then
                    -- Fire Logic: Check for a Tool in inventory or use generic impulse
                    local tool = LocalPlayer.Backpack:FindFirstChildWhichIsA("Tool")
                    
                    if tool and tool.Parent ~= LocalPlayer.Character then
                        tool.Parent = LocalPlayer.Character
                        
                        -- Simple fire function (Adjust based on specific game mechanics)
                        if tool.Fired then
                            tool:Fired()
                        else
                            -- Fallback: Impulse velocity for shooting
                            local rootPart = LocalPlayer.Character.HumanoidRootPart
                            rootPart.Velocity = Vector3.new(0, 0, -500) 
                        end
                        
                        LastFireTime = currentTime
                    end
                end
            else
                -- Reset fire time if out of range (optional)
                LastFireTime = 0
            end
        end
    end

    RunService.RenderStepped:Connect(OnRenderStepped)
    
    -- ==========================================
    -- INITIALIZATION
    -- ==========================================
    CreateUI()
end

-- Load the UI and Logic
CreateUI()
