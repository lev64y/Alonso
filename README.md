-- LocalScript para simular ações temporárias em Death Ball (upgrades de espadas) usando Fluent UI
-- Coloque este script dentro de StarterPlayerScripts
-- Requer o módulo Fluent em ReplicatedStorage
-- Interface estilizada em preto e cinza

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Player = Players.LocalPlayer

-- Carrega a biblioteca Fluent
local Fluent = require(ReplicatedStorage:WaitForChild("Fluent"))

-- Cria a interface com Fluent (paleta preta e cinza)
local Interface = Fluent:CreateInterface("ActionSimulator", {
    Size = UDim2.new(0, 400, 0, 500),
    Title = "Simulador de Upgrades - Death Ball",
    Theme = "Dark",
    Acrylic = true, -- Efeito de blur
    Minimizable = true,
    Closeable = true,
    ThemeOverrides = {
        PrimaryBackground = Color3.fromRGB(20, 20, 20), -- Fundo preto escuro
        SecondaryBackground = Color3.fromRGB(50, 50, 50), -- Cinza escuro para campos
        TextColor = Color3.fromRGB(200, 200, 200), -- Texto cinza claro
        ButtonBackground = Color3.fromRGB(70, 70, 70), -- Botões em cinza
        ButtonHover = Color3.fromRGB(100, 100, 100), -- Hover em cinza mais claro
        Accent = Color3.fromRGB(90, 90, 90) -- Detalhes em cinza médio
    }
})

-- Variáveis de estado
local currentGems = 0
local initialGems = 0
local currentSword = nil
local currentGrade = 0
local initialSword = nil
local initialGrade = 0
local isFrozen = false
local upgradeCost = 100 -- Custo fixo para tentar upgrade
local upgradeChance = 0.2 -- 20% de chance de sucesso

-- Componentes da UI
local GemsInput = Interface:TextInput({
    PlaceholderText = "Digite suas Gems",
    Size = UDim2.new(0.9, 0, 0, 40),
    Position = UDim2.new(0.05, 0, 0.12, 0),
    ClearTextOnFocus = true
})

local SwordInput = Interface:TextInput({
    PlaceholderText = "Digite o nome da espada (ex.: Espada Comum)",
    Size = UDim2.new(0.9, 0, 0, 40),
    Position = UDim2.new(0.05, 0, 0.22, 0),
    ClearTextOnFocus = true
})

local GradeInput = Interface:TextInput({
    PlaceholderText = "Digite o grau atual (ex.: 0)",
    Size = UDim2.new(0.9, 0, 0, 40),
    Position = UDim2.new(0.05, 0, 0.32, 0),
    ClearTextOnFocus = true
})

local StatusLabel = Interface:Paragraph({
    Text = "Estado: Vazio",
    Size = UDim2.new(0.9, 0, 0, 120),
    Position = UDim2.new(0.05, 0, 0.42, 0),
    TextWrapped = true,
    TextScaled = true
})

local UpgradeButton = Interface:Button({
    Text = "Tentar Upgrade (100 Gems)",
    Size = UDim2.new(0.43, 0, 0, 40),
    Position = UDim2.new(0.05, 0, 0.68, 0),
    Style = "Custom", -- Estilo personalizado para cinza
    BackgroundColor = Color3.fromRGB(70, 70, 70),
    HoverColor = Color3.fromRGB(100, 100, 100)
})

local FreezeButton = Interface:Button({
    Text = "Congelar Tempo",
    Size = UDim2.new(0.43, 0, 0, 40),
    Position = UDim2.new(0.52, 0, 0.68, 0),
    Style = "Custom", -- Estilo personalizado para cinza
    BackgroundColor = Color3.fromRGB(70, 70, 70),
    HoverColor = Color3.fromRGB(100, 100, 100)
})

local RevertButton = Interface:Button({
    Text = "Reverter ao Estado Congelado",
    Size = UDim2.new(0.9, 0, 0, 40),
    Position = UDim2.new(0.05, 0, 0.78, 0),
    Style = "Custom", -- Estilo personalizado para cinza
    BackgroundColor = Color3.fromRGB(70, 70, 70),
    HoverColor = Color3.fromRGB(100, 100, 100)
})

-- Função para atualizar a interface
local function updateUI()
    local statusText = "Estado:\n"
    statusText = statusText .. "Gems: " .. currentGems .. "\n"
    if currentSword then
        statusText = statusText .. "Espada: " .. currentSword .. " (Grau " .. currentGrade .. ")"
    else
        statusText = statusText .. "Espada: Nenhuma"
    end
    StatusLabel:SetText(statusText)
end

-- Inicializar gems
GemsInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local input = tonumber(GemsInput.Text)
        if input and input >= 0 then
            currentGems = input
            initialGems = input
            updateUI()
            GemsInput:SetText("Gems definidas!")
        else
            GemsInput:SetText("Digite um número válido")
        end
    end
end)

-- Inicializar espada
SwordInput.FocusLost:Connect(function(enterPressed)
    if enterPressed and SwordInput.Text ~= "" then
        currentSword = SwordInput.Text
        initialSword = SwordInput.Text
        updateUI()
        SwordInput:SetText("Espada definida!")
    end
end)

-- Inicializar grau
GradeInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local input = tonumber(GradeInput.Text)
        if input and input >= 0 then
            currentGrade = input
            initialGrade = input
            updateUI()
            GradeInput:SetText("Grau definido!")
        else
            GradeInput:SetText("Digite um número válido")
        end
    end
end)

-- Função para tentar upgrade
UpgradeButton.MouseButton1Click:Connect(function()
    if not currentSword then
        StatusLabel:SetText("Estado: Nenhuma espada selecionada!")
        return
    end
    if currentGems < upgradeCost then
        StatusLabel:SetText("Estado: Gems insuficientes!")
        return
    end

    currentGems = currentGems - upgradeCost
    local random = math.random()
    if random <= upgradeChance then
        -- Sucesso no upgrade
        currentGrade = currentGrade + 1
        StatusLabel:SetText("Estado: Sucesso! Espada upada para Grau " .. currentGrade)
    else
        -- Falha no upgrade (perde a espada)
        currentSword = nil
        currentGrade = 0
        StatusLabel:SetText("Estado: Falha! Espada perdida!")
    end
    updateUI()
end)

-- Função para congelar o estado
FreezeButton.MouseButton1Click:Connect(function()
    if not isFrozen then
        initialGems = currentGems
        initialSword = currentSword
        initialGrade = currentGrade
        isFrozen = true
        FreezeButton:SetText("Tempo Congelado")
        FreezeButton:SetBackgroundColor(Color3.fromRGB(90, 90, 90)) -- Cinza mais claro para indicar congelado
        StatusLabel:SetText(StatusLabel.Text .. "\nTempo congelado!")
    end
end)

-- Função para reverter ao estado congelado
RevertButton.MouseButton1Click:Connect(function()
    if isFrozen then
        currentGems = initialGems
        currentSword = initialSword
        currentGrade = initialGrade
        updateUI()
        StatusLabel:SetText("Estado: Revertido ao estado congelado!")
    else
        StatusLabel:SetText("Estado: Congele o tempo primeiro!")
    end
end)

-- Ativar/desativar com tecla T
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.T then
        Interface:Toggle()
        print("Simulador de ações " .. (Interface.Visible and "ativado" or "desativado"))
    end
end)

-- Inicializar interface
updateUI()
