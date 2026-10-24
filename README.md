-- // Configurações & Variáveis
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Remote = ReplicatedStorage:WaitForChild("Features"):WaitForChild("PumpkinGun"):WaitForChild("Remotes"):WaitForChild("Action")

local AutoPerfectEnabled = true

-- // Lógica de Mira (Raycast local para onde o mouse aponta)
local function GetAimPosition()
    local mousePos = UserInputService:GetMouseLocation()
    local ray = Camera:ViewportPointToRay(mousePos.X, mousePos.Y)
    
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.IgnoreWater = true
    if LocalPlayer.Character then
        params.FilterDescendantsInstances = { LocalPlayer.Character }
    end

    local result = workspace:Raycast(ray.Origin, ray.Direction * 1000, params)
    if result then
        return result.Position
    else
        return ray.Origin + ray.Direction * 1000
    end
end

-- // Hooks no Tool (Ativa quando você usa a PumpkinGun)
local function HookTool(tool)
    if tool.Name == "PumpkinGun" or tool.Name:find("Pumpkin") then
        tool.Activated:Connect(function()
            if not AutoPerfectEnabled then return end
            
            local targetPos = GetAimPosition()
            -- Envia imediatamente o disparo perfeito (Força = 1) para o servidor
            Remote:FireServer("Fire", targetPos, 1)
        end)
    end
end

-- Monitora as ferramentas do jogador
local function WatchCharacter(char)
    char.ChildAdded:Connect(function(child)
        if child:IsA("Tool") then
            HookTool(child)
        end
    end)
    
    for _, child in pairs(char:GetChildren()) do
        if child:IsA("Tool") then
            HookTool(child)
        end
    end
end

if LocalPlayer.Character then WatchCharacter(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(WatchCharacter)

-- // Interface Gráfica (GUI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PumpkinGunGui"
ScreenGui.ResetOnSpawn = false

-- Proteção contra detecção de GUI simples
if syn and syn.protect_gui then
    syn.protect_gui(ScreenGui)
    ScreenGui.Parent = CoreGui
elseif gethui then
    ScreenGui.Parent = gethui()
else
    ScreenGui.Parent = CoreGui
end

-- Frame Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 220, 0, 130)
MainFrame.Position = UDim2.new(0.5, -110, 0.4, -65)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = MainFrame

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -30, 0, 35)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🎃 Pumpkin Gun Mod"
Title.TextColor3 = Color3.fromRGB(255, 140, 0)
Title.TextSize = 16
Title.Font = Enum.Font.SourceSansBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = MainFrame

-- Botão de Minimizar / Fechar no Menu
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 25, 0, 25)
CloseBtn.Position = UDim2.new(1, -30, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextSize = 14
CloseBtn.Parent = MainFrame

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = CloseBtn

-- Toggle Button (Auto Perfect)
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0.9, 0, 0, 40)
ToggleBtn.Position = UDim2.new(0.05, 0, 0.5, -10)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(46, 139, 87)
ToggleBtn.Text = "Auto Perfect: ATIVADO"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.TextSize = 15
ToggleBtn.Parent = MainFrame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleBtn

-- Botão Flutuante para Abrir/Fechar o Menu
local OpenBtn = Instance.new("TextButton")
OpenBtn.Name = "OpenButton"
OpenBtn.Size = UDim2.new(0, 50, 0, 50)
OpenBtn.Position = UDim2.new(0, 15, 0.5, -25)
OpenBtn.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
OpenBtn.Text = "🎃"
OpenBtn.TextSize = 25
OpenBtn.Visible = false
OpenBtn.Active = true
OpenBtn.Draggable = true
OpenBtn.Parent = ScreenGui

local OpenCorner = Instance.new("UICorner")
OpenCorner.CornerRadius = UDim.new(1, 0)
OpenCorner.Parent = OpenBtn

-- // Eventos dos Botões
ToggleBtn.MouseButton1Click:Connect(function()
    AutoPerfectEnabled = not AutoPerfectEnabled
    if AutoPerfectEnabled then
        ToggleBtn.Text = "Auto Perfect: ATIVADO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(46, 139, 87)
    else
        ToggleBtn.Text = "Auto Perfect: DESATIVADO"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(178, 34, 34)
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
    OpenBtn.Visible = true
end)

OpenBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = true
    OpenBtn.Visible = false
end)
