-- Script Farm de Moedas com Interface Rayfield + Anti-AFK (CORRIGIDO)
-- Coloque no StarterGui ou StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ========== CARREGAR RAYFIELD ==========
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- ========== CONFIGURAÇÕES ==========
local farmAtivado = false
local modoAtual = "Desligado"
local velocidadeFly = 50
local distanciaMaxima = 300
local antiAFKAtivado = false

-- NOMES PARA IGNORAR
local nomesIgnorados = {
    ["Node"] = true,
    ["ShopInteractionPart"] = true,
    ["Bounds"] = true,
    ["CompetitiveRaceStaminaBoost"] = true,
    ["CompetitiveRaceSpeedBoost"] = true,
    ["Bound"] = true,
    ["SlidePart"] = true,
}

-- PARENTS PARA IGNORAR
local parentsIgnorados = {
    ["CompetitiveRaceItem"] = true,
}

-- MATERIAIS PARA IGNORAR
local materiaisIgnorados = {
    [Enum.Material.Glass] = true,
}

-- Cache de moedas visitadas
local moedasVisitadas = {}
local tempoCache = 3

-- ========== SISTEMA ANTI-AFK (CORRIGIDO) ==========
local antiAFKConnection

local function ativarAntiAFK()
    if antiAFKConnection then return end
    
    antiAFKConnection = player.Idled:Connect(function()
        local VirtualUser = game:GetService("VirtualUser")
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end)
    
    Rayfield:Notify({
        Title = "Anti-AFK Ativado",
        Content = "Você não será kickado por inatividade",
        Duration = 3,
        Image = nil,
    })
end

local function desativarAntiAFK()
    if antiAFKConnection then
        antiAFKConnection:Disconnect()
        antiAFKConnection = nil
    end
    
    Rayfield:Notify({
        Title = "Anti-AFK Desativado",
        Content = "Sistema anti-AFK foi desligado",
        Duration = 3,
        Image = nil,
    })
end

-- ========== SISTEMA DE FLY ==========
local flyConnection
local flyPart

local function ativarFly()
    flyPart = Instance.new("Part")
    flyPart.Size = Vector3.new(2, 1, 2)
    flyPart.Transparency = 1
    flyPart.Anchored = true
    flyPart.CanCollide = false
    flyPart.CFrame = humanoidRootPart.CFrame
    flyPart.Parent = workspace
    
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bodyVelocity.Parent = humanoidRootPart
    
    local bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(0, 0, 0)
    bodyGyro.P = 9e9
    bodyGyro.Parent = humanoidRootPart
    
    flyConnection = RunService.Heartbeat:Connect(function()
        if not farmAtivado or modoAtual ~= "Fly" then
            bodyVelocity.Velocity = Vector3.new(0, 0, 0)
            return
        end
        
        humanoidRootPart.CFrame = flyPart.CFrame
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    end)
end

local function desativarFly()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    if flyPart then
        flyPart:Destroy()
        flyPart = nil
    end
    
    for _, obj in pairs(humanoidRootPart:GetChildren()) do
        if obj:IsA("BodyVelocity") or obj:IsA("BodyGyro") then
            obj:Destroy()
        end
    end
end

-- ========== BUSCAR MOEDAS ==========
local function encontrarMoedasProximas()
    local moedas = {}
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj:FindFirstChildOfClass("TouchTransmitter") then
            
            local deveIgnorar = false
            local nomeObj = obj.Name
            local nomeLower = string.lower(nomeObj)
            
            if nomesIgnorados[nomeObj] then
                deveIgnorar = true
            end
            
            if string.find(nomeLower, "bound") or 
               string.find(nomeLower, "node") or
               string.find(nomeLower, "shop") or
               string.find(nomeLower, "boost") or
               string.find(nomeLower, "stamina") or
               string.find(nomeLower, "speed") or
               string.find(nomeLower, "slide") then
                deveIgnorar = true
            end
            
            if obj.Parent and parentsIgnorados[obj.Parent.Name] then
                deveIgnorar = true
            end
            
            if materiaisIgnorados[obj.Material] then
                deveIgnorar = true
            end
            
            if obj.Parent then
                local parentLower = string.lower(obj.Parent.Name)
                if string.find(parentLower, "competitiverace") then
                    deveIgnorar = true
                end
            end
            
            if not deveIgnorar and not moedasVisitadas[obj] then
                local distancia = (humanoidRootPart.Position - obj.Position).Magnitude
                
                if distancia <= distanciaMaxima then
                    table.insert(moedas, {
                        part = obj,
                        distancia = distancia,
                        nome = obj.Name
                    })
                end
            end
        end
    end
    
    table.sort(moedas, function(a, b)
        return a.distancia < b.distancia
    end)
    
    return moedas
end

-- ========== TELEPORT MODO 1 ==========
local function teleportModo1(moeda)
    if not moeda or not moeda.Parent then return end
    
    for i = 1, 3 do
        humanoidRootPart.CFrame = moeda.CFrame
        task.wait(0.03)
    end
    
    moedasVisitadas[moeda] = true
end

-- ========== TELEPORT MODO 2 ==========
local function teleportModo2(moeda)
    if not moeda or not moeda.Parent then return end
    
    local posicoes = {
        moeda.CFrame * CFrame.new(0, 0, -3),
        moeda.CFrame * CFrame.new(3, 0, 0),
        moeda.CFrame * CFrame.new(0, 0, 3),
        moeda.CFrame * CFrame.new(-3, 0, 0),
        moeda.CFrame
    }
    
    for _, pos in ipairs(posicoes) do
        humanoidRootPart.CFrame = pos
        task.wait(0.05)
    end
    
    moedasVisitadas[moeda] = true
end

-- ========== FLY ==========
local function voarParaMoeda(moeda)
    if not moeda or not moeda.Parent or not flyPart then return end
    
    local destino = moeda.Position
    local distancia = (flyPart.Position - destino).Magnitude
    local tempo = distancia / velocidadeFly
    
    local tweenInfo = TweenInfo.new(
        tempo,
        Enum.EasingStyle.Linear,
        Enum.EasingDirection.InOut
    )
    
    local tween = TweenService:Create(flyPart, tweenInfo, {
        CFrame = CFrame.new(destino)
    })
    
    tween:Play()
    tween.Completed:Wait()
    
    task.wait(0.2)
    
    moedasVisitadas[moeda] = true
end

-- ========== LOOP PRINCIPAL ==========
local function loopFarmPrincipal()
    while farmAtivado do
        local moedas = encontrarMoedasProximas()
        
        if #moedas > 0 then
            local maisProxima = moedas[1].part
            
            if modoAtual == "Teleport1" then
                teleportModo1(maisProxima)
                task.wait(0.1)
                
            elseif modoAtual == "Teleport2" then
                teleportModo2(maisProxima)
                task.wait(0.1)
                
            elseif modoAtual == "Fly" then
                voarParaMoeda(maisProxima)
            end
        else
            moedasVisitadas = {}
            task.wait(1)
        end
        
        task.wait()
    end
end

-- Limpar cache
spawn(function()
    while true do
        task.wait(tempoCache)
        moedasVisitadas = {}
    end
end)

-- ========== CRIAR INTERFACE RAYFIELD ==========
local Window = Rayfield:CreateWindow({
    Name = "💰 Farm de Moedas",
    LoadingTitle = "Farm de Moedas",
    LoadingSubtitle = "by Você",
    ConfigurationSaving = {
        Enabled = false,
    },
    Discord = {
        Enabled = false,
    },
    KeySystem = false,
})

-- ========== TAB PRINCIPAL ==========
local MainTab = Window:CreateTab("🏠 Principal", nil)

local StatusSection = MainTab:CreateSection("📊 Status")

local StatusLabel = MainTab:CreateLabel("Status: 🔴 Desligado")
local ModoLabel = MainTab:CreateLabel("Modo: Nenhum")
local MoedasLabel = MainTab:CreateLabel("Moedas Encontradas: 0")
local AntiAFKLabel = MainTab:CreateLabel("Anti-AFK: 🔴 Desligado")

-- Atualizar labels
spawn(function()
    while task.wait(0.5) do
        local moedas = encontrarMoedasProximas()
        MoedasLabel:Set("Moedas Encontradas: " .. #moedas)
    end
end)

-- ========== TAB DE MODOS ==========
local ModosTab = Window:CreateTab("⚙️ Modos de Farm", nil)

local ModosSection = ModosTab:CreateSection("Escolha o Modo")

-- Botão Teleport Modo 1
local BotaoTeleport1 = ModosTab:CreateButton({
    Name = "⚡ Teleport Modo 1 (Rápido)",
    Callback = function()
        if modoAtual == "Teleport1" and farmAtivado then
            farmAtivado = false
            desativarFly()
            modoAtual = "Desligado"
            StatusLabel:Set("Status: 🔴 Desligado")
            ModoLabel:Set("Modo: Nenhum")
            Rayfield:Notify({
                Title = "Farm Desativado",
                Content = "Teleport Modo 1 desligado",
                Duration = 3,
                Image = nil,
            })
        else
            farmAtivado = false
            desativarFly()
            modoAtual = "Teleport1"
            farmAtivado = true
            StatusLabel:Set("Status: 🟢 Ativo")
            ModoLabel:Set("Modo: Teleport Rápido")
            spawn(loopFarmPrincipal)
            Rayfield:Notify({
                Title = "Farm Ativado",
                Content = "Teleport Modo 1 ligado (3x rápido)",
                Duration = 3,
                Image = nil,
            })
        end
    end,
})

-- Botão Teleport Modo 2
local BotaoTeleport2 = ModosTab:CreateButton({
    Name = "🎯 Teleport Modo 2 (Círculo)",
    Callback = function()
        if modoAtual == "Teleport2" and farmAtivado then
            farmAtivado = false
            desativarFly()
            modoAtual = "Desligado"
            StatusLabel:Set("Status: 🔴 Desligado")
            ModoLabel:Set("Modo: Nenhum")
            Rayfield:Notify({
                Title = "Farm Desativado",
                Content = "Teleport Modo 2 desligado",
                Duration = 3,
                Image = nil,
            })
        else
            farmAtivado = false
            desativarFly()
            modoAtual = "Teleport2"
            farmAtivado = true
            StatusLabel:Set("Status: 🟢 Ativo")
            ModoLabel:Set("Modo: Teleport Círculo")
            spawn(loopFarmPrincipal)
            Rayfield:Notify({
                Title = "Farm Ativado",
                Content = "Teleport Modo 2 ligado (5 posições)",
                Duration = 3,
                Image = nil,
            })
        end
    end,
})

-- Botão Fly
local BotaoFly = ModosTab:CreateButton({
    Name = "✈️ Fly (Voa Suave)",
    Callback = function()
        if modoAtual == "Fly" and farmAtivado then
            farmAtivado = false
            desativarFly()
            modoAtual = "Desligado"
            StatusLabel:Set("Status: 🔴 Desligado")
            ModoLabel:Set("Modo: Nenhum")
            Rayfield:Notify({
                Title = "Farm Desativado",
                Content = "Fly desligado",
                Duration = 3,
                Image = nil,
            })
        else
            farmAtivado = false
            desativarFly()
            modoAtual = "Fly"
            farmAtivado = true
            ativarFly()
            StatusLabel:Set("Status: 🟢 Ativo")
            ModoLabel:Set("Modo: Fly")
            spawn(loopFarmPrincipal)
            Rayfield:Notify({
                Title = "Farm Ativado",
                Content = "Fly ligado (movimento suave)",
                Duration = 3,
                Image = nil,
            })
        end
    end,
})

-- ========== TAB DE CONFIGURAÇÕES ==========
local ConfigTab = Window:CreateTab("⚙️ Configurações", nil)

local ConfigSection = ConfigTab:CreateSection("Ajustes")

-- Slider de Velocidade
local VelocidadeSlider = ConfigTab:CreateSlider({
    Name = "⚡ Velocidade do Fly",
    Range = {1, 500},
    Increment = 5,
    CurrentValue = 50,
    Flag = "VelocidadeFly",
    Callback = function(Value)
        velocidadeFly = Value
        Rayfield:Notify({
            Title = "Velocidade Alterada",
            Content = "Velocidade do Fly: " .. Value,
            Duration = 2,
            Image = nil,
        })
    end,
})

-- Slider de Distância
local DistanciaSlider = ConfigTab:CreateSlider({
    Name = "📏 Distância Máxima",
    Range = {50, 1000},
    Increment = 50,
    CurrentValue = 300,
    Flag = "DistanciaMaxima",
    Callback = function(Value)
        distanciaMaxima = Value
        Rayfield:Notify({
            Title = "Distância Alterada",
            Content = "Distância Máxima: " .. Value .. " studs",
            Duration = 2,
            Image = nil,
        })
    end,
})

-- ========== TAB UTILITÁRIOS ==========
local UtilTab = Window:CreateTab("🛠️ Utilitários", nil)

local UtilSection = UtilTab:CreateSection("Ferramentas")

-- Toggle Anti-AFK
local AntiAFKToggle = UtilTab:CreateToggle({
    Name = "🔄 Anti-AFK",
    CurrentValue = false,
    Flag = "AntiAFK",
    Callback = function(Value)
        antiAFKAtivado = Value
        
        if Value then
            ativarAntiAFK()
            AntiAFKLabel:Set("Anti-AFK: 🟢 Ligado")
        else
            desativarAntiAFK()
            AntiAFKLabel:Set("Anti-AFK: 🔴 Desligado")
        end
    end,
})

-- Botão Limpar Cache
local BotaoLimparCache = UtilTab:CreateButton({
    Name = "🗑️ Limpar Cache de Moedas",
    Callback = function()
        moedasVisitadas = {}
        Rayfield:Notify({
            Title = "Cache Limpo",
            Content = "Todas as moedas podem ser coletadas novamente",
            Duration = 3,
            Image = nil,
        })
    end,
})

-- ========== TAB DE INFO ==========
local InfoTab = Window:CreateTab("ℹ️ Informações", nil)

local InfoSection = InfoTab:CreateSection("🚫 Filtros Ativos")

InfoTab:CreateLabel("Ignora os seguintes nomes:")
InfoTab:CreateLabel("• Node, Bounds, SlidePart")
InfoTab:CreateLabel("• ShopInteractionPart")
InfoTab:CreateLabel("• CompetitiveRace (boosts)")
InfoTab:CreateLabel("")
InfoTab:CreateLabel("Ignora Material Glass")
InfoTab:CreateLabel("")

local ModosInfoSection = InfoTab:CreateSection("📖 Descrição dos Modos")

InfoTab:CreateLabel("⚡ Modo 1: Teleporte direto 3x")
InfoTab:CreateLabel("   → Mais rápido, pode errar algumas")
InfoTab:CreateLabel("")
InfoTab:CreateLabel("🎯 Modo 2: Teleporte em círculo")
InfoTab:CreateLabel("   → 5 posições ao redor, mais eficiente")
InfoTab:CreateLabel("")
InfoTab:CreateLabel("✈️ Fly: Voa suave até moeda")
InfoTab:CreateLabel("   → Movimento natural e fluido")

-- Botão Desligar Tudo
local DesligarSection = InfoTab:CreateSection("🔴 Controles")

local BotaoDesligar = InfoTab:CreateButton({
    Name = "🔴 DESLIGAR TUDO",
    Callback = function()
        farmAtivado = false
        desativarFly()
        modoAtual = "Desligado"
        StatusLabel:Set("Status: 🔴 Desligado")
        ModoLabel:Set("Modo: Nenhum")
        Rayfield:Notify({
            Title = "Tudo Desligado",
            Content = "Farm completamente desativado",
            Duration = 3,
            Image = nil,
        })
    end,
})

-- ========== ATUALIZAR CHARACTER ==========
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    
    farmAtivado = false
    desativarFly()
    modoAtual = "Desligado"
    StatusLabel:Set("Status: 🔴 Desligado")
    ModoLabel:Set("Modo: Nenhum")
end)

-- Notificação inicial
Rayfield:Notify({
    Title = "Farm de Moedas Carregado!",
    Content = "Escolha um modo de farm para começar",
    Duration = 5,
    Image = nil,
})
