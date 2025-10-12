-- LocalScript dentro de StarterPlayerScripts
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local mouse = player:GetMouse()
local uis = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Variáveis do Auto Teleport
local autoTeleporting = false
local autoDelay = 0.5 -- meio segundo entre cada clique

-- Primeiro sistema de teleportes (11 locais)
local teleports1 = {
    {name = "Condenada 1", pos = Vector3.new(4253.15, 29.67, -6964.59)},
    {name = "Condenada 2", pos = Vector3.new(4299.07, 44.31, -6897.23)},
    {name = "Condenada 3", pos = Vector3.new(4345.58, 74.50, -7019.68)},
    {name = "Condenada 4", pos = Vector3.new(4437.02, 95.86, -6966.37)},
    {name = "Condenada 5", pos = Vector3.new(4499.72, 104.85, -7015.65)},
    {name = "Condenada 6", pos = Vector3.new(4450.35, 106.52, -7039.21)},
    {name = "Condenada 7", pos = Vector3.new(4398.37, 85.95, -6977.11)},
    {name = "Condenada 8", pos = Vector3.new(4346.50, 71.56, -7018.40)},
    {name = "Condenada 9", pos = Vector3.new(4305.84, 43.87, -6898.48)},
    {name = "Condenada 10", pos = Vector3.new(4260.06, 27.13, -6960.53)},
    {name = "Condenada 11", pos = Vector3.new(4192.87, 21.01, -6990.53)}
}

-- Segundo sistema de teleportes (4 locais)
local teleports2 = {
    {name = "Local 1", pos = Vector3.new(4009.78, 21.37, -6705.96)},
    {name = "Local 2", pos = Vector3.new(4125.84, 37.00, -6733.84)},
    {name = "Local 3", pos = Vector3.new(4124.55, 21.37, -6744.80)},
    {name = "Local 4", pos = Vector3.new(3953.47, 21.00, -6689.76)}
}

-- Variável para controlar qual sistema está ativo
local currentTeleportSystem = teleports1

-- GUI Criada via script
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "AdminGUI"
screenGui.ResetOnSpawn = false

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 320, 0, 450)
frame.Position = UDim2.new(0.5, -160, 0.5, -225)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Active = true
frame.Draggable = true
frame.Visible = false

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 50)
title.Text = "👑 Painel ADM - Teleports"
title.TextScaled = true
title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.FredokaOne

-- ========== SELEÇÃO DE SISTEMA ==========

local systemLabel = Instance.new("TextLabel", frame)
systemLabel.Size = UDim2.new(0.8, 0, 0, 25)
systemLabel.Position = UDim2.new(0.1, 0, 0.12, 0)
systemLabel.Text = "Sistema de Teleporte:"
systemLabel.TextScaled = true
systemLabel.BackgroundTransparency = 1
systemLabel.TextColor3 = Color3.new(1, 1, 1)
systemLabel.Font = Enum.Font.GothamBold

local system1Btn = Instance.new("TextButton", frame)
system1Btn.Size = UDim2.new(0.35, 0, 0, 30)
system1Btn.Position = UDim2.new(0.1, 0, 0.17, 0)
system1Btn.Text = "11 Locais"
system1Btn.TextScaled = true
system1Btn.BackgroundColor3 = Color3.fromRGB(60, 80, 60)
system1Btn.TextColor3 = Color3.new(1, 1, 1)
system1Btn.Font = Enum.Font.GothamBold

local system2Btn = Instance.new("TextButton", frame)
system2Btn.Size = UDim2.new(0.35, 0, 0, 30)
system2Btn.Position = UDim2.new(0.55, 0, 0.17, 0)
system2Btn.Text = "4 Locais"
system2Btn.TextScaled = true
system2Btn.BackgroundColor3 = Color3.fromRGB(80, 60, 60)
system2Btn.TextColor3 = Color3.new(1, 1, 1)
system2Btn.Font = Enum.Font.GothamBold

-- ========== SISTEMA AUTO TELEPORT ==========

-- Título do Teleport
local teleportTitle = Instance.new("TextLabel", frame)
teleportTitle.Size = UDim2.new(0.8, 0, 0, 25)
teleportTitle.Position = UDim2.new(0.1, 0, 0.25, 0)
teleportTitle.Text = "🔗 Auto Teleport"
teleportTitle.TextScaled = true
teleportTitle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
teleportTitle.TextColor3 = Color3.new(1, 1, 1)
teleportTitle.Font = Enum.Font.GothamBold

-- Botão Auto Teleport
local autoTeleportBtn = Instance.new("TextButton", frame)
autoTeleportBtn.Size = UDim2.new(0.8, 0, 0, 40)
autoTeleportBtn.Position = UDim2.new(0.1, 0, 0.32, 0)
autoTeleportBtn.Text = "Auto Teleport: OFF"
autoTeleportBtn.TextScaled = true
autoTeleportBtn.BackgroundColor3 = Color3.fromRGB(80, 50, 50)
autoTeleportBtn.TextColor3 = Color3.new(1, 1, 1)
autoTeleportBtn.Font = Enum.Font.GothamBold

-- Status do Delay
local delayLabel = Instance.new("TextLabel", frame)
delayLabel.Size = UDim2.new(0.8, 0, 0, 25)
delayLabel.Position = UDim2.new(0.1, 0, 0.4, 0)
delayLabel.Text = "Delay: 500ms"
delayLabel.TextScaled = true
delayLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
delayLabel.TextColor3 = Color3.new(1, 1, 1)
delayLabel.Font = Enum.Font.Gotham

-- Botões de controle do delay
local delayControls = Instance.new("Frame", frame)
delayControls.Size = UDim2.new(0.8, 0, 0, 30)
delayControls.Position = UDim2.new(0.1, 0, 0.45, 0)
delayControls.BackgroundTransparency = 1

local minusBtn = Instance.new("TextButton", delayControls)
minusBtn.Size = UDim2.new(0.45, 0, 1, 0)
minusBtn.Position = UDim2.new(0, 0, 0, 0)
minusBtn.Text = "- Diminuir"
minusBtn.TextScaled = true
minusBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
minusBtn.TextColor3 = Color3.new(1, 1, 1)
minusBtn.Font = Enum.Font.GothamBold

local plusBtn = Instance.new("TextButton", delayControls)
plusBtn.Size = UDim2.new(0.45, 0, 1, 0)
plusBtn.Position = UDim2.new(0.55, 0, 0, 0)
plusBtn.Text = "+ Aumentar"
plusBtn.TextScaled = true
plusBtn.BackgroundColor3 = Color3.fromRGB(40, 80, 40)
plusBtn.TextColor3 = Color3.new(1, 1, 1)
plusBtn.Font = Enum.Font.GothamBold

-- Lista de locais ativos
local locationsLabel = Instance.new("TextLabel", frame)
locationsLabel.Size = UDim2.new(0.8, 0, 0, 25)
locationsLabel.Position = UDim2.new(0.1, 0, 0.52, 0)
locationsLabel.Text = "Locais Ativos:"
locationsLabel.TextScaled = true
locationsLabel.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
locationsLabel.TextColor3 = Color3.new(1, 1, 1)
locationsLabel.Font = Enum.Font.GothamBold

-- Frame para lista de locais
local locationsFrame = Instance.new("ScrollingFrame", frame)
locationsFrame.Size = UDim2.new(0.8, 0, 0, 150)
locationsFrame.Position = UDim2.new(0.1, 0, 0.58, 0)
locationsFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
locationsFrame.BorderSizePixel = 0
locationsFrame.ScrollBarThickness = 6

local layout = Instance.new("UIListLayout", locationsFrame)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 5)

-- Botão de abrir/fechar
local toggleButton = Instance.new("TextButton", screenGui)
toggleButton.Size = UDim2.new(0, 120, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Abrir Painel"
toggleButton.TextScaled = true
toggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 255)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.GothamBold

-- ========== FUNÇÕES DO AUTO TELEPORT ==========

-- Função Teleporte (suporta cadeira)
local function teleportTo(pos)
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    
    local tweenInfo = TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    
    if humanoid.SeatPart then
        local seat = humanoid.SeatPart
        TweenService:Create(seat, tweenInfo, {CFrame = CFrame.new(pos + Vector3.new(0,3,0))}):Play()
    else
        TweenService:Create(hrp, tweenInfo, {CFrame = CFrame.new(pos + Vector3.new(0,3,0))}):Play()
    end
end

-- Função para atualizar label do delay
local function updateDelayLabel()
    delayLabel.Text = "Delay: "..math.floor(autoDelay*1000).."ms"
end

-- Função para atualizar lista de locais
local function updateLocationsList()
    -- Limpar lista atual
    for _, child in ipairs(locationsFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    -- Adicionar novos botões
    for i, teleport in ipairs(currentTeleportSystem) do
        local locButton = Instance.new("TextButton")
        locButton.Size = UDim2.new(1, -10, 0, 30)
        locButton.Text = teleport.name
        locButton.TextScaled = true
        locButton.BackgroundColor3 = Color3.fromRGB(70, 70, 90)
        locButton.TextColor3 = Color3.new(1, 1, 1)
        locButton.Font = Enum.Font.Gotham
        locButton.Parent = locationsFrame
        
        locButton.MouseButton1Click:Connect(function()
            -- Teleportar para este local 3 vezes
            for j = 1, 3 do
                teleportTo(teleport.pos)
                task.wait(0.1)
            end
        end)
    end
    
    -- Ajustar tamanho do canvas
    locationsFrame.CanvasSize = UDim2.new(0, 0, 0, #currentTeleportSystem * 35)
end

-- Função do Auto Teleport
local function toggleAutoTeleport()
    autoTeleporting = not autoTeleporting
    autoTeleportBtn.Text = "Auto Teleport: "..(autoTeleporting and "ON" or "OFF")
    
    if autoTeleporting then
        autoTeleportBtn.BackgroundColor3 = Color3.fromRGB(50, 80, 50)
        task.spawn(function()
            while autoTeleporting do
                for _, info in ipairs(currentTeleportSystem) do
                    if not autoTeleporting then break end
                    -- 3 cliques antes de ir pro próximo
                    for i=1,3 do
                        if not autoTeleporting then break end
                        teleportTo(info.pos)
                        task.wait(autoDelay)
                    end
                end
            end
        end)
    else
        autoTeleportBtn.BackgroundColor3 = Color3.fromRGB(80, 50, 50)
    end
end

-- Função para trocar sistema de teleporte
local function switchTeleportSystem(system)
    currentTeleportSystem = system
    
    if system == teleports1 then
        system1Btn.BackgroundColor3 = Color3.fromRGB(60, 80, 60)
        system2Btn.BackgroundColor3 = Color3.fromRGB(80, 60, 60)
    else
        system1Btn.BackgroundColor3 = Color3.fromRGB(80, 60, 60)
        system2Btn.BackgroundColor3 = Color3.fromRGB(60, 80, 60)
    end
    
    updateLocationsList()
    
    -- Parar auto teleport se estiver ativo
    if autoTeleporting then
        toggleAutoTeleport()
    end
end

-- ========== EVENTOS ==========

-- Eventos dos botões de sistema
system1Btn.MouseButton1Click:Connect(function()
    switchTeleportSystem(teleports1)
end)

system2Btn.MouseButton1Click:Connect(function()
    switchTeleportSystem(teleports2)
end)

-- Eventos dos botões de delay
minusBtn.MouseButton1Click:Connect(function()
    autoDelay = math.max(0.1, autoDelay - 0.1) -- mínimo 100ms
    updateDelayLabel()
end)

plusBtn.MouseButton1Click:Connect(function()
    autoDelay = math.min(10, autoDelay + 0.1) -- máximo 10000ms
    updateDelayLabel()
end)

-- Evento do botão Auto Teleport
autoTeleportBtn.MouseButton1Click:Connect(function()
    toggleAutoTeleport()
end)

-- Evento do botão toggle
toggleButton.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
    if frame.Visible then
        toggleButton.Text = "Fechar Painel"
    else
        toggleButton.Text = "Abrir Painel"
    end
end)

-- ========== INICIALIZAÇÃO ==========

-- Inicializar
updateDelayLabel()
switchTeleportSystem(teleports1) -- Sistema 1 como padrão

print("Painel ADM com 2 Sistemas de Teleporte carregado!")
