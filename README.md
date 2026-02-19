--[[ 
    URIEL XIT V2 - ULTIMATE
    - Abrir: V | Fechar: B
    - ESP: Box (Vermelho) e Linhas
    - Speed Aimbot: 0 a 10
    - Team & Wall Check Integrados
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local Settings = {
    Aimbot = false,
    ESP = false,
    TeamCheck = true,
    WallCheck = true,
    AimPart = "Head",
    AimSpeed = 5, -- 0 a 10
    FOVRadius = 150,
    OpenKey = Enum.KeyCode.V,
    CloseKey = Enum.KeyCode.B,
    Colors = {
        Accent = Color3.fromRGB(0, 180, 255),
        Enemy = Color3.fromRGB(255, 0, 0),
        Panel = Color3.fromRGB(15, 15, 15)
    }
}

--// UI OTIMIZADA
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 280, 0, 480)
Main.Position = UDim2.new(0.5, -140, 0.5, -240)
Main.BackgroundColor3 = Settings.Colors.Panel
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", Main).Color = Color3.fromRGB(40, 40, 40)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.Text = "URIEL XIT V2"
Title.TextColor3 = Settings.Colors.Accent
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.BackgroundTransparency = 1

-- BOTÕES E SLIDERS (FACTORY)
local function CreateToggle(text, pos, callback)
    local Btn = Instance.new("TextButton", Main)
    Btn.Size = UDim2.new(0.85, 0, 0, 35)
    Btn.Position = pos
    Btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    Btn.Text = text .. ": OFF"
    Btn.TextColor3 = Color3.fromRGB(200, 200, 200)
    Btn.Font = Enum.Font.GothamBold
    Instance.new("UICorner", Btn)
    
    Btn.MouseButton1Click:Connect(function()
        local state = callback()
        Btn.Text = text .. (state and ": ON" or ": OFF")
        Btn.BackgroundColor3 = state and Settings.Colors.Accent or Color3.fromRGB(25, 25, 25)
        Btn.TextColor3 = state and Color3.new(1,1,1) or Color3.fromRGB(200, 200, 200)
    end)
end

local function CreateSlider(name, pos, min, max, default, callback)
    local Label = Instance.new("TextLabel", Main)
    Label.Size = UDim2.new(0.85, 0, 0, 20)
    Label.Position = pos
    Label.Text = name .. ": " .. default
    Label.TextColor3 = Color3.fromRGB(180, 180, 180)
    Label.BackgroundTransparency = 1
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11

    local Bar = Instance.new("Frame", Main)
    Bar.Size = UDim2.new(0.85, 0, 0, 4)
    Bar.Position = pos + UDim2.new(0, 0, 0, 22)
    Bar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    
    local Fill = Instance.new("Frame", Bar)
    Fill.Size = UDim2.new((default-min)/(max-min), 0, 1, 0)
    Fill.BackgroundColor3 = Settings.Colors.Accent
    
    Bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local conn
            conn = RunService.RenderStepped:Connect(function()
                local m = UserInputService:GetMouseLocation()
                local perc = math.clamp((m.X - Bar.AbsolutePosition.X) / Bar.AbsoluteSize.X, 0, 1)
                local val = math.floor(min + (max-min) * perc)
                Fill.Size = UDim2.new(perc, 0, 1, 0)
                Label.Text = name .. ": " .. val
                callback(val)
            end)
            UserInputService.InputEnded:Connect(function(i2) if i2.UserInputType == Enum.UserInputType.MouseButton1 then conn:Disconnect() end end)
        end
    end)
end

CreateToggle("AIMBOT", UDim2.new(0.075, 0, 0.12, 0), function() Settings.Aimbot = not Settings.Aimbot return Settings.Aimbot end)
CreateToggle("ESP MASTER", UDim2.new(0.075, 0, 0.22, 0), function() Settings.ESP = not Settings.ESP return Settings.ESP end)
CreateSlider("VELOCIDADE MIRA", UDim2.new(0.075, 0, 0.32, 0), 0, 10, Settings.AimSpeed, function(v) Settings.AimSpeed = v end)
CreateSlider("RAIO DO FOV", UDim2.new(0.075, 0, 0.42, 0), 30, 500, Settings.FOVRadius, function(v) Settings.FOVRadius = v end)

local Box = Instance.new("TextBox", Main)
Box.Size = UDim2.new(0.85, 0, 0, 100)
Box.Position = UDim2.new(0.075, 0, 0.65, 0)
Box.BackgroundColor3 = Color3.new(0,0,0)
Box.PlaceholderText = "Notas do URIEL XIT..."
Box.TextColor3 = Color3.new(1,1,1)
Box.TextWrapped = true
Instance.new("UICorner", Box)

--// SISTEMA ESP (BOX & LINHA)
local function CreateESP(plr)
    local Box = Drawing.new("Square")
    local Line = Drawing.new("Line")
    
    RunService.RenderStepped:Connect(function()
        if Settings.ESP and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr ~= LocalPlayer then
            if Settings.TeamCheck and plr.Team == LocalPlayer.Team then 
                Box.Visible = false Line.Visible = false return 
            end
            
            local hrp = plr.Character.HumanoidRootPart
            local pos, screen = Camera:WorldToViewportPoint(hrp.Position)
            
            if screen then
                -- BOX
                local sizeX = 1000 / pos.Z
                local sizeY = 1500 / pos.Z
                Box.Size = Vector2.new(sizeX, sizeY)
                Box.Position = Vector2.new(pos.X - sizeX/2, pos.Y - sizeY/2)
                Box.Color = Settings.Colors.Enemy
                Box.Thickness = 1
                Box.Visible = true
                
                -- LINHA
                Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                Line.To = Vector2.new(pos.X, pos.Y + sizeY/2)
                Line.Color = Settings.Colors.Enemy
                Line.Visible = true
            else Box.Visible = false Line.Visible = false end
        else Box.Visible = false Line.Visible = false end
    end)
end

for _, p in pairs(Players:GetPlayers()) do CreateESP(p) end
Players.PlayerAdded:Connect(CreateESP)

--// AIMBOT ENGINE
local FOV = Drawing.new("Circle")
FOV.Thickness = 1

RunService.RenderStepped:Connect(function()
    FOV.Visible = Settings.Aimbot
    FOV.Radius = Settings.FOVRadius
    FOV.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    
    if Settings.Aimbot then
        local target, dist = nil, Settings.FOVRadius
        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Settings.AimPart) then
                if Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
                local part = v.Character[Settings.AimPart]
                local pos, screen = Camera:WorldToViewportPoint(part.Position)
                if screen then
                    local mag = (Vector2.new(pos.X, pos.Y) - FOV.Position).Magnitude
                    if mag < dist then
                        target = part
                        dist = mag
                    end
                end
            end
        end
        
        if target then
            -- Converte escala 0-10 para smoothing (0.01 a 1.0)
            local smooth = math.clamp(Settings.AimSpeed / 10, 0.01, 1)
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Position), smooth)
        end
    end
end)

UserInputService.InputBegan:Connect(function(i, g)
    if g then return end
    if i.KeyCode == Settings.OpenKey then Main.Visible = true
    elseif i.KeyCode == Settings.CloseKey then Main.Visible = false end
end)
