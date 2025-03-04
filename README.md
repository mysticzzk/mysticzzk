lua
-- Services
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Check if device is mobile
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

-- Variables
local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera
local Network = game:GetService("Stats").Network

-- Silent Aim Variables
local fov = 45
local silent_aime = false
local blatant_mode = false
local prediction_duration = 0.1

-- Create FOV Circle
local fov_circle = Drawing.new("Circle")
fov_circle.Thickness = 2
fov_circle.Color = Color3.fromRGB(0, 0, 0)
fov_circle.Transparency = 1
fov_circle.Radius = fov
fov_circle.Filled = false

-- Previous functions remain the same
-- [Insert all the silent aim functions here from the previous script]

-- Create UI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SilentAimUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = PlayerGui

-- Create main frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = isMobile and UDim2.new(0, 250, 0, 200) or UDim2.new(0, 200, 0, 160)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -100)
MainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
MainFrame.Visible = false
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- Create title bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -30, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Silent Aim"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = isMobile and 18 or 16
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Create toggle buttons with larger size for mobile
local function CreateToggleButton(text, position)
    local button = Instance.new("TextButton")
    button.Size = isMobile and UDim2.new(0.8, 0, 0, 40) or UDim2.new(0.8, 0, 0, 30)
    button.Position = position
    button.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = isMobile and 16 or 14
    button.Font = Enum.Font.GothamSemibold
    button.Parent = MainFrame
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button
    
    return button
end

local SilentAimToggle = CreateToggleButton("Silent Aim: OFF", UDim2.new(0.1, 0, 0.3, 0))
local BlatantToggle = CreateToggleButton("Blatant: OFF", UDim2.new(0.1, 0, 0.5, 0))

-- Create FOV slider for mobile
if isMobile then
    local FOVSlider = Instance.new("TextLabel")
    FOVSlider.Size = UDim2.new(0.8, 0, 0, 20)
    FOVSlider.Position = UDim2.new(0.1, 0, 0.7, 0)
    FOVSlider.BackgroundTransparency = 1
    FOVSlider.Text = "FOV: " .. fov
    FOVSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
    FOVSlider.TextSize = 16
    FOVSlider.Font = Enum.Font.GothamSemibold
    FOVSlider.Parent = MainFrame

    local SliderFrame = Instance.new("Frame")
    SliderFrame.Size = UDim2.new(0.8, 0, 0, 10)
    SliderFrame.Position = UDim2.new(0.1, 0, 0.8, 0)
    SliderFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    SliderFrame.Parent = MainFrame

    local SliderCorner = Instance.new("UICorner")
    SliderCorner.CornerRadius = UDim.new(0, 5)
    SliderCorner.Parent = SliderFrame

    local SliderButton = Instance.new("TextButton")
    SliderButton.Size = UDim2.new(0.1, 0, 1, 0)
    SliderButton.Position = UDim2.new((fov - 15) / 150, 0, 0, 0)
