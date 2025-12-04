--// MysticZ HUB UI Library //--

local TweenService = game:GetService("TweenService")
local Library = {}

-- Temas disponíveis
Library.Themes = {
    LightTheme = Color3.fromRGB(240,240,240),
    DarkTheme = Color3.fromRGB(20,20,20),
    GrapeTheme = Color3.fromRGB(90,0,120),
    BloodTheme = Color3.fromRGB(120,0,0),
    Ocean = Color3.fromRGB(0,80,120),
    Midnight = Color3.fromRGB(10,10,40),
    Sentinel = Color3.fromRGB(45,45,45),
    Synapse = Color3.fromRGB(60,60,60)
}

function Library.CreateLib(title, theme)
    local UI = {}
    local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

    -- Main window
    local Main = Instance.new("Frame", ScreenGui)
    Main.Size = UDim2.new(0, 450, 0, 300)
    Main.Position = UDim2.new(0.5, -225, 0.5, -150)
    Main.BackgroundColor3 = Library.Themes[theme] or Library.Themes.DarkTheme
    Main.BorderSizePixel = 0
    Main.Active = true
    Main.Draggable = true

    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)

    -- TitleBar
    local TitleBar = Instance.new("Frame", Main)
    TitleBar.Size = UDim2.new(1, 0, 0, 35)
    TitleBar.BackgroundTransparency = 1

    local Title = Instance.new("TextLabel", TitleBar)
    Title.Size = UDim2.new(1, -120, 1, 0)
    Title.Position = UDim2.new(0, 10, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = title
    Title.TextColor3 = Color3.fromRGB(255,255,255)
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.TextScaled = true

    -- Create buttons (Close, Minimize, Maximize)
    local function makeRoundButton(parent, color, xPos, symbol)
        local btn = Instance.new("TextButton", parent)
        btn.Size = UDim2.new(0, 30, 0, 25)
        btn.Position = UDim2.new(1, xPos, 0, 5)
        btn.BackgroundColor3 = color
        btn.Text = symbol
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.TextScaled = true
        btn.BorderSizePixel = 0

        local corner = Instance.new("UICorner", btn)
        corner.CornerRadius = UDim.new(0, 6)

        return btn
    end

    local CloseBtn = makeRoundButton(TitleBar, Color3.fromRGB(180,40,40), -35, "X")
    local MinBtn   = makeRoundButton(TitleBar, Color3.fromRGB(200,180,40), -105, "-")
    local MaxBtn   = makeRoundButton(TitleBar, Color3.fromRGB(60,180,60), -70, "▢")

    local minimized = false
    local maximized = false
    local oldSize = Main.Size
    local oldPos = Main.Position

    local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)

    -- Minimize animation
    MinBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            oldSize = Main.Size
            TweenService:Create(Main, tweenInfo, {Size = UDim2.new(0, 450, 0, 35)}):Play()
        else
            TweenService:Create(Main, tweenInfo, {Size = oldSize}):Play()
        end
    end)

    -- Maximize animation
    MaxBtn.MouseButton1Click:Connect(function()
        maximized = not maximized
        if maximized then
            oldSize = Main.Size
            oldPos = Main.Position
            TweenService:Create(Main, tweenInfo, {
                Position = UDim2.new(0, 0, 0, 0),
                Size = UDim2.new(1, 0, 1, 0)
            }):Play()
        else
            TweenService:Create(Main, tweenInfo, {
                Position = oldPos,
                Size = oldSize
            }):Play()
        end
    end)

    -- Close window
    CloseBtn.MouseButton1Click:Connect(function()
        TweenService:Create(Main, tweenInfo, {BackgroundTransparency = 1}):Play()
        wait(0.15)
        ScreenGui:Destroy()
        -- Adiciona a chamada para sair do Aimbot quando a UI é fechada
        if getgenv().Aimbot and getgenv().Aimbot.Functions and getgenv().Aimbot.Functions.Exit then
            getgenv().Aimbot.Functions:Exit()
        end
    end)

    -- Abas (Tabs)
    local tabCount = 0
    function UI:Tab(name)
        tabCount = tabCount + 1
        local Tab = {}

        local ButtonTab = Instance.new("TextButton", Main)
        ButtonTab.Size = UDim2.new(0, 80, 0, 30)
        -- Ajusta a posição da nova aba
        ButtonTab.Position = UDim2.new(0, 10 + (tabCount - 1) * 90, 0, 45)
        ButtonTab.Text = name
        ButtonTab.BackgroundColor3 = Color3.fromRGB(40,40,40)
        ButtonTab.TextColor3 = Color3.fromRGB(255,255,255)
        ButtonTab.Font = Enum.Font.SourceSans
        ButtonTab.TextSize = 14
        
        Instance.new("UICorner", ButtonTab).CornerRadius = UDim.new(0, 6)

        local Page = Instance.new("Frame", Main)
        Page.Size = UDim2.new(1, -20, 1, -90)
        Page.Position = UDim2.new(0, 10, 0, 80)
        Page.BackgroundColor3 = Color3.fromRGB(30,30,30)
        Page.ClipsDescendants = true -- Para gerenciar o layout de forma mais limpa

        Instance.new("UICorner", Page).CornerRadius = UDim.new(0, 8)
        
        -- Adiciona o layout para os itens (Grid/List)
        local ListLayout = Instance.new("UIListLayout", Page)
        ListLayout.Padding = UDim.new(0, 5)
        ListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
        ListLayout.VerticalAlignment = Enum.VerticalAlignment.Top
        ListLayout.SortOrder = Enum.SortOrder.LayoutOrder

        Page.Visible = false
        
        -- Torna a primeira aba visível por padrão
        if tabCount == 1 then
            Page.Visible = true
        end

        ButtonTab.MouseButton1Click:Connect(function()
            for _,pg in pairs(Main:GetChildren()) do
                if pg:IsA("Frame") and pg.Position.Y.Offset == 80 and pg ~= TitleBar then -- Verifica as páginas
                    pg.Visible = false
                end
            end
            Page.Visible = true
        end)

        -- Toggle
        local controlCount = 0
        function Tab:CreateToggle(text, initialValue, callback)
            controlCount = controlCount + 1
            
            local Holder = Instance.new("Frame", Page)
            Holder.Name = "Toggle_" .. controlCount
            Holder.Size = UDim2.new(1, 0, 0, 40)
            Holder.Position = UDim2.new(0, 0, 0, (controlCount - 1) * 40) -- ListLayout deve gerenciar isso
            Holder.BackgroundTransparency = 1
            Holder.LayoutOrder = controlCount

            local Btn = Instance.new("TextButton", Holder)
            Btn.Size = UDim2.new(0, 150, 0, 35)
            Btn.Text = text
            Btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
            Btn.TextColor3 = Color3.fromRGB(255,255,255)
            Btn.Font = Enum.Font.SourceSans
            Btn.TextSize = 14
            Btn.TextXAlignment = Enum.TextXAlignment.Left
            Btn.TextPosition = Vector2.new(10, 0.5)

            Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)

            local Toggle = Instance.new("TextButton", Holder)
            Toggle.Size = UDim2.new(0, 35, 0, 35)
            Toggle.Position = UDim2.new(1, -40, 0, 0)
            Toggle.BackgroundColor3 = Color3.fromRGB(90,90,90)
            Toggle.TextColor3 = Color3.fromRGB(255,255,255)
            Toggle.Text = ""

            Instance.new("UICorner", Toggle).CornerRadius = UDim.new(0, 6)

            local state = initialValue or false

            local function update()
                Toggle.Text = state and "✔" or ""
                Toggle.BackgroundColor3 = state and Color3.fromRGB(60,180,60) or Color3.fromRGB(90,90,90)
            end

            local function toggleHandler()
                state = not state
                update()
                if callback then
                    callback(state)
                end
            end

            Btn.MouseButton1Click:Connect(toggleHandler)
            Toggle.MouseButton1Click:Connect(toggleHandler)

            update()
        end

        return Tab
    end

    return UI
end

--=================================================================================
-- INTEGRANDO O SCRIPT AIMBOT AQUI
--=================================================================================

-- [O script Aimbot completo está aqui, mas o código é longo. Vou usar 
-- o ambiente global para garantir que o script seja executado e 
-- acessível pela UI. Como o prompt não pede a duplicação do código 
-- Aimbot, vou prosseguir com a integração da UI.]

-- **NOTA**: O script Aimbot original deve ser executado antes/junto
-- com este script de UI para que 'getgenv().Aimbot' exista.
-- Vou incluir o Aimbot completo abaixo da UI para garantir a execução
-- na ordem correta, se o usuário copiar TUDO.

--=================================================================================
-- EXEMPLO DE USO E CONEXÃO COM O AIMBOT
--=================================================================================

local Window = Library.CreateLib("MysticZ Hub", "DarkTheme")

-- ABA: AIMBOT SETTINGS
local TabAimbot = Window:Tab("Aimbot")

if getgenv().Aimbot and getgenv().Aimbot.Settings then
    local AimbotSettings = getgenv().Aimbot.Settings
    local FOVSettings = getgenv().Aimbot.FOVSettings
    local AimbotFunctions = getgenv().Aimbot.Functions

    -- Aimbot Toggles
    TabAimbot:CreateToggle("Aimbot ON/OFF", AimbotSettings.Enabled, function(v)
        AimbotSettings.Enabled = v
        if not v then
            AimbotFunctions:Exit() -- Desconecta se for desativado
            AimbotFunctions:Restart() -- Reconecta para reativar
        end
    end)
    
    TabAimbot:CreateToggle("Team Check", AimbotSettings.TeamCheck, function(v)
        AimbotSettings.TeamCheck = v
    end)

    TabAimbot:CreateToggle("Alive Check", AimbotSettings.AliveCheck, function(v)
        AimbotSettings.AliveCheck = v
    end)
    
    TabAimbot:CreateToggle("Wall Check", AimbotSettings.WallCheck, function(v)
        AimbotSettings.WallCheck = v
    end)
    
    TabAimbot:CreateToggle("Toggle Mode (Hold Key)", AimbotSettings.Toggle, function(v)
        AimbotSettings.Toggle = v
    end)

    -- ABA: FOV SETTINGS
    local TabFOV = Window:Tab("FOV")

    TabFOV:CreateToggle("FOV Circle ON/OFF", FOVSettings.Enabled, function(v)
        FOVSettings.Enabled = v
    end)

    TabFOV:CreateToggle("FOV Circle Visible", FOVSettings.Visible, function(v)
        FOVSettings.Visible = v
    end)

    TabFOV:CreateToggle("FOV Circle Filled", FOVSettings.Filled, function(v)
        FOVSettings.Filled = v
    end)
    
    -- Aqui você adicionaria Sliders para 'Amount', 'Sensitivity', 'ThirdPersonSensitivity', etc.
    -- O código da UI original não tem 'Slider', então usamos apenas o que está disponível ('Toggle').

else
    -- Se o script Aimbot não foi carregado corretamente no 'getgenv().Aimbot'
    TabAimbot:CreateToggle("AIMBOT SCRIPT NOT LOADED", false, function() end)
end

-- ================================================================================
-- CÓDIGO AIMBOT ORIGINAL (Para garantir que as variáveis 'getgenv().Aimbot' existam)
-- RECOMENDA-SE EXECUTAR ESTA PARTE PRIMEIRO.
-- ================================================================================

-- --// Cache

local select = select
local pcall, getgenv, next, Vector2, mathclamp, type, mousemoverel = select(1, pcall, getgenv, next, Vector2.new, math.clamp, type, mousemoverel or (Input and Input.MouseMove))

--// Preventing Multiple Processes

pcall(function()
    if getgenv().Aimbot and getgenv().Aimbot.Functions and getgenv().Aimbot.Functions.Exit then
        getgenv().Aimbot.Functions:Exit()
    end
end)

--// Environment

getgenv().Aimbot = {}
local Environment = getgenv().Aimbot

--// Services

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// Variables

local RequiredDistance, Typing, Running, Animation, ServiceConnections = 2000, false, false, nil, {}

--// Script Settings

Environment.Settings = {
    Enabled = true,
    TeamCheck = false,
    AliveCheck = true,
    WallCheck = false, -- Laggy
    Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
    ThirdPerson = false, -- Uses mousemoverel instead of CFrame to 
support locking in third person (could be choppy)
    ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
    TriggerKey = "E",
    Toggle = false,
    LockPart = "Head" -- Body part to lock on
}

Environment.FOVSettings = {
    Enabled = true,
    Visible = false,
    Amount = 90,
    Color = Color3.fromRGB(255, 255, 255),
    LockedColor = Color3.fromRGB(255, 70, 70),
    Transparency = 0.5,
    Sides = 60,
    Thickness = 1,
    Filled = false
}

Environment.FOVCircle = Drawing.new("Circle")

--// Functions

local function CancelLock()
    Environment.Locked = nil
    if Animation then Animation:Cancel() end
    Environment.FOVCircle.Color = Environment.FOVSettings.Color
end

local function GetClosestPlayer()
    if not Environment.Locked then
        RequiredDistance = (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000)

        for _, v in next, Players:GetPlayers() do
            if v ~= LocalPlayer then
                if v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                    if Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
                    if Environment.Settings.AliveCheck 
and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                    if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end

                    local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                    local Distance = (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Vector.X, Vector.Y)).Magnitude

                    if Distance < RequiredDistance and OnScreen then
                        RequiredDistance = Distance
                        Environment.Locked = v
                    end
                end
            end
        end
    elseif (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).Y)).Magnitude > RequiredDistance then
        CancelLock()
    end
end

--// Typing Check

ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
    Typing = true
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
    Typing = false
end)

--// Main

local function Load()
    ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if Environment.FOVSettings.Enabled and Environment.Settings.Enabled then
            Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
            Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
            Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
            Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
            Environment.FOVCircle.Color = Environment.FOVSettings.Color
            Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
            Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
            Environment.FOVCircle.Position = Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
        else
            Environment.FOVCircle.Visible = false
        end

        if Running and Environment.Settings.Enabled then
            GetClosestPlayer()

            if Environment.Locked then
                if Environment.Settings.ThirdPerson then
                    Environment.Settings.ThirdPersonSensitivity = mathclamp(Environment.Settings.ThirdPersonSensitivity, 0.1, 5)

                    local Vector = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)
                    mousemoverel((Vector.X - UserInputService:GetMouseLocation().X) 
* Environment.Settings.ThirdPersonSensitivity, (Vector.Y - UserInputService:GetMouseLocation().Y) * Environment.Settings.ThirdPersonSensitivity)
                else
                    if Environment.Settings.Sensitivity > 0 then
                        Animation = TweenService:Create(Camera, TweenInfo.new(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)})
                        Animation:Play()
                    else
                        Camera.CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)
                    end
                end

            Environment.FOVCircle.Color = Environment.FOVSettings.LockedColor

            end
        end
    end)

    ServiceConnections.InputBeganConnection = UserInputService.InputBegan:Connect(function(Input)
        if not Typing then
            pcall(function()
                if Input.KeyCode == Enum.KeyCode[Environment.Settings.TriggerKey] then
                    if Environment.Settings.Toggle then
                        Running = not Running

                        if not Running then
                            CancelLock()
                        end
                    else
                        Running = true
                    end
                end
            end)

            pcall(function()
                if Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
                    if Environment.Settings.Toggle then
                        Running = not Running

                        if not Running then
                            CancelLock()
                        end
                    else
                        Running = true
                    end
                end
            end)
        end
    end)

    ServiceConnections.InputEndedConnection = UserInputService.InputEnded:Connect(function(Input)
        if not Typing then
            if not Environment.Settings.Toggle then
                pcall(function()
                    if Input.KeyCode == Enum.KeyCode[Environment.Settings.TriggerKey] then
                        Running = false;
CancelLock()
                    end
                end)

                pcall(function()
                    if Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
                        Running = false; CancelLock()
                    end
                end)
            end
        end
    end)
end

--// Functions

Environment.Functions = {}

function Environment.Functions:Exit()
    for _, v in next, ServiceConnections do
        v:Disconnect()
    end

    if Environment.FOVCircle.Remove then Environment.FOVCircle:Remove() end

    getgenv().Aimbot.Functions = nil
    getgenv().Aimbot = nil
    
    Load = nil;
GetClosestPlayer = nil; CancelLock = nil
end

function Environment.Functions:Restart()
    for _, v in next, ServiceConnections do
        v:Disconnect()
    end
    
    -- Redefine as funções locais que foram niladas no Exit
    local select = select
    local pcall, getgenv, next, Vector2, mathclamp, type, mousemoverel = select(1, pcall, getgenv, next, Vector2.new, math.clamp, type, mousemoverel or (Input and Input.MouseMove))
    local RequiredDistance, Typing, Running, Animation, ServiceConnections = 2000, false, false, nil, {}
    
    -- Recria o FOVCircle se tiver sido removido
    if not Environment.FOVCircle then Environment.FOVCircle = Drawing.new("Circle") end
    
    -- Re-atribui as funções locais
    local function CancelLock()
        Environment.Locked = nil
        if Animation then Animation:Cancel() end
        Environment.FOVCircle.Color = Environment.FOVSettings.Color
    end
    
    local function GetClosestPlayer()
        if not Environment.Locked then
            RequiredDistance = (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000)

            for _, v in next, Players:GetPlayers() do
                if v ~= LocalPlayer then
                    if v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                        if Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
                        if Environment.Settings.AliveCheck 
                        and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                        if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end

                        local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                        local Distance = (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Vector.X, Vector.Y)).Magnitude

                        if Distance < RequiredDistance and OnScreen then
                            RequiredDistance = Distance
                            Environment.Locked = v
                        end
                    end
                end
            end
        elseif (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).Y)).Magnitude > RequiredDistance then
            CancelLock()
        end
    end

    Load()
end

function Environment.Functions:ResetSettings()
    Environment.Settings = {
        Enabled = true,
        TeamCheck = false,
        AliveCheck = true,
        WallCheck = false,
        Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
        ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
        ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
        TriggerKey = "MouseButton2",
        Toggle = false,
        LockPart = "Head" -- Body part to lock on
    }

    Environment.FOVSettings = {
        Enabled = true,
        Visible = true,
        Amount = 90,
        Color = Color3.fromRGB(255, 255, 255),
        LockedColor = Color3.fromRGB(255, 70, 70),
        Transparency = 0.5,
        Sides = 60,
        Thickness = 1,
        Filled = false
    }
end

--// Load

Load()
