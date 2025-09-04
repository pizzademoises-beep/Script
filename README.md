-- Plataforma rosa brilhante com interface móvel persistente (PC e Mobile) + notificação minimalista maior
local player = game.Players.LocalPlayer
local humanoid
local hrp
local ativo = false
local criando = false
local ultimoPulo = 0

local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Função para criar GUI
local function criarGUI()
    -- Verifica se já existe GUI
    if player:FindFirstChild("PlayerGui") and player.PlayerGui:FindFirstChild("PlataformaGUI") then
        return
    end

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "PlataformaGUI"
    ScreenGui.ResetOnSpawn = false -- mantém após morrer
    ScreenGui.Parent = player:WaitForChild("PlayerGui")

    -- Interface principal
    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 300, 0, 150)
    MainFrame.Position = UDim2.new(0.5, -150, 0.1, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui

    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0.1,0)
    UICorner.Parent = MainFrame

    local UIStroke = Instance.new("UIStroke")
    UIStroke.Thickness = 4
    UIStroke.Parent = MainFrame

    -- Bordas animadas RGB
    task.spawn(function()
        local hue = 0
        while UIStroke.Parent do
            hue = (hue + 0.01) % 1
            UIStroke.Color = Color3.fromHSV(hue,1,1)
            task.wait(0.05)
        end
    end)

    -- Texto superior com efeito RGB
    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1,0,0,40)
    Title.Position = UDim2.new(0,0,0,10)
    Title.BackgroundTransparency = 1
    Title.Text = "TTK: MOISESCHA"  -- mantém o texto normal
    Title.Font = Enum.Font.SourceSansBold
    Title.TextScaled = true
    Title.Parent = MainFrame

    -- Animar cor RGB do título
    task.spawn(function()
        local hue = 0
        while Title.Parent do
            hue = (hue + 0.01) % 1
            Title.TextColor3 = Color3.fromHSV(hue,1,1)
            task.wait(0.05)
        end
    end)

    -- Botão Auto Floor
    local Toggle = Instance.new("TextButton")
    Toggle.Size = UDim2.new(0,200,0,60)
    Toggle.Position = UDim2.new(0.5,-100,0,70)
    Toggle.BackgroundColor3 = Color3.fromRGB(255,0,0)
    Toggle.Text = "ATIVAR AUTO FLOOR"
    Toggle.TextColor3 = Color3.fromRGB(255,255,255)
    Toggle.Font = Enum.Font.SourceSansBold
    Toggle.TextScaled = true
    Toggle.Parent = MainFrame
    Toggle.Active = true
    Toggle.AutoButtonColor = false

    local ToggleCorner = Instance.new("UICorner")
    ToggleCorner.CornerRadius = UDim.new(0.2,0)
    ToggleCorner.Parent = Toggle

    -- Tornar a interface móvel (PC + Mobile)
    local dragging = false
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end

    MainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or
           input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    MainFrame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or
           input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)

    -- Notificação minimalista maior
    local NotifyFrame = Instance.new("Frame")
    NotifyFrame.Size = UDim2.new(0, 220, 0, 45)
    NotifyFrame.Position = UDim2.new(0.5, -110, 0.9, 0)
    NotifyFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
    NotifyFrame.BackgroundTransparency = 0.7
    NotifyFrame.BorderSizePixel = 0
    NotifyFrame.Visible = false
    NotifyFrame.Parent = ScreenGui

    local NotifyText = Instance.new("TextLabel")
    NotifyText.Size = UDim2.new(1,0,1,0)
    NotifyText.BackgroundTransparency = 1
    NotifyText.TextColor3 = Color3.fromRGB(255,255,255)
    NotifyText.Font = Enum.Font.SourceSansSemibold
    NotifyText.TextScaled = true
    NotifyText.Text = ""
    NotifyText.Parent = NotifyFrame

    local function notificar(msg)
        NotifyText.Text = msg
        NotifyFrame.Visible = true
        task.wait(1.2)
        NotifyFrame.Visible = false
    end

    -- Efeito RGB botão
    task.spawn(function()
        local hue = 0
        while Toggle.Parent do
            hue = (hue + 0.01) % 1
            if ativo then
                Toggle.BackgroundColor3 = Color3.fromHSV(hue,1,1)
            else
                Toggle.BackgroundColor3 = Color3.fromHSV(hue,0.8,0.3)
            end
            task.wait(0.05)
        end
    end)

    -- Alternar estado
    Toggle.MouseButton1Click:Connect(function()
        ativo = not ativo
        if ativo then
            Toggle.Text = "AUTO FLOOR ATIVADO ✅"
            notificar("Plataforma Ativada ✅")
        else
            Toggle.Text = "ATIVAR AUTO FLOOR"
            notificar("Plataforma Desativada ❌")
        end
    end)
end

-- Função para criar plataforma rosa brilhante
local function criarPlataforma()
    if not ativo or not hrp then return end
    local p = Instance.new("Part")
    p.Size = Vector3.new(10,1,10)
    p.Anchored = true
    p.CanCollide = true
    p.Material = Enum.Material.Neon
    p.Color = Color3.fromRGB(255,0,255)
    p.Position = hrp.Position - Vector3.new(0,3,0)
    p.Parent = workspace
    game:GetService("Debris"):AddItem(p, 2)
end

-- Atualizar humanoid e HRP quando o personagem spawnar
local function setupCharacter(char)
    humanoid = char:WaitForChild("Humanoid")
    hrp = char:WaitForChild("HumanoidRootPart")

    -- Pulo
    humanoid.StateChanged:Connect(function(_, state)
        if ativo and state == Enum.HumanoidStateType.Jumping then
            local agora = tick()
            if agora - ultimoPulo > 0.5 then
                ultimoPulo = agora
                criarPlataforma()
            end
        end
    end)

    -- Andar
    RunService.Heartbeat:Connect(function()
        if ativo and humanoid and humanoid.MoveDirection.Magnitude > 0 then
            if not criando then
                criando = true
                task.spawn(function()
                    while ativo and humanoid.MoveDirection.Magnitude > 0 do
                        criarPlataforma()
                        task.wait(0.3)
                    end
                    criando = false
                end)
            end
        end
    end)
end

-- Detectar spawn do personagem
player.CharacterAdded:Connect(function(char)
    criarGUI()
    setupCharacter(char)
end)

-- Caso o personagem já exista
if player.Character then
    criarGUI()
    setupCharacter(player.Character)
end
