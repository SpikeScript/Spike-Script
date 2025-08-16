-- Spike Hub (VIP) - Admin GUI com botão arrastável e animação
-- Coloque dentro de StarterGui como LocalScript

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- GUI principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SpikeHubVIP"
ScreenGui.Parent = player:WaitForChild("PlayerGui")

-- Frame principal (menu)
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 300, 0, 420)
Frame.Position = UDim2.new(0.05, 0, 0.2, 0)
Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
Frame.BorderSizePixel = 0
Frame.Visible = false
Frame.BackgroundTransparency = 1 -- começa invisível
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 16)
UICorner.Parent = Frame

-- Botão flutuante para abrir/fechar
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 140, 0, 40)
ToggleButton.Position = UDim2.new(0.05, 0, 0.05, 0)
ToggleButton.Text = "📂 Abrir Spike Hub"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 16
ToggleButton.Parent = ScreenGui

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 10)
btnCorner.Parent = ToggleButton

-- Função de animação para abrir/fechar
local function toggleMenu()
    if Frame.Visible == false then
        Frame.Visible = true
        Frame.Position = UDim2.new(0.05, -50, 0.2, 0) -- fora da tela para deslizar
        Frame.BackgroundTransparency = 1
        -- Tween: mover + fade in
        TweenService:Create(Frame, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Position = UDim2.new(0.05, 0, 0.2, 0),
            BackgroundTransparency = 0
        }):Play()
        ToggleButton.Text = "📂 Fechar Spike Hub"
    else
        -- Tween: mover + fade out
        local tween = TweenService:Create(Frame, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
            Position = UDim2.new(0.05, -50, 0.2, 0),
            BackgroundTransparency = 1
        })
        tween:Play()
        tween.Completed:Connect(function()
            Frame.Visible = false
        end)
        ToggleButton.Text = "📂 Abrir Spike Hub"
    end
end

ToggleButton.MouseButton1Click:Connect(toggleMenu)

-- 🔄 Deixar o botão arrastável
local dragging = false
local dragStart, startPos

ToggleButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or 
       input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = ToggleButton.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or 
                     input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        ToggleButton.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

-- Título do menu
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 50)
Title.BackgroundTransparency = 1
Title.Text = "⚡ Spike Hub (VIP)"
Title.TextColor3 = Color3.fromRGB(255, 215, 0)
Title.TextScaled = true
Title.Font = Enum.Font.GothamBold
Title.Parent = Frame

local Bar = Instance.new("Frame")
Bar.Size = UDim2.new(1, -20, 0, 2)
Bar.Position = UDim2.new(0, 10, 0, 48)
Bar.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
Bar.BorderSizePixel = 0
Bar.Parent = Frame

-- Função para criar botões no menu
local function createButton(name, posY, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 260, 0, 45)
    btn.Position = UDim2.new(0, 20, 0, posY)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.Parent = Frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = btn

    btn.MouseEnter:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(60, 60, 90)
    end)
    btn.MouseLeave:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    end)

    btn.MouseButton1Click:Connect(callback)
end

-- Botões do hub
createButton("🏠 Teleportar para Base", 70, function()
    local base = workspace:FindFirstChild("Base")
    if base and base:IsA("Part") then
        character:MoveTo(base.Position + Vector3.new(0, 5, 0))
    end
end)

createButton("⚡ Velocidade Alta", 125, function()
    character.Humanoid.WalkSpeed = 50
end)

createButton("🦘 Pulo Alto", 180, function()
    character.Humanoid.JumpPower = 120
end)

createButton("♾️ Pulo Infinito", 235, function()
    local UIS = game:GetService("UserInputService")
    UIS.JumpRequest:Connect(function()
        character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end)
end)

createButton("🔍 Ativar ESP", 290, function()
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            if not plr.Character:FindFirstChild("Highlight") then
                local highlight = Instance.new("Highlight")
                highlight.Parent = plr.Character
                highlight.FillColor = Color3.fromRGB(255, 0, 0)
            end
        end
    end
end)

-- 🚪 Noclip
local noclip = false
createButton("🚪 Ativar Noclip", 345, function()
    noclip = not noclip
    if noclip then
        game:GetService("RunService").Stepped:Connect(function()
            if noclip and character and character:FindFirstChild("HumanoidRootPart") then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end)
