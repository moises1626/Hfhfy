local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 10)

-- Configurações iniciais
local espEnabled = false
local lineThickness = 0.03
local isMinimized = false
local speedBoxVisible = false
local defaultWalkSpeed = 16
local autoClickEnabled = false
local infiniteStaminaEnabled = false
local autoClickConnection = nil
local staminaConnection = nil
local speedConnection = nil

-- Tabela para armazenar elementos ESP
local espData = {}

-- Cria ScreenGui principal
local gui = Instance.new("ScreenGui")
gui.Name = "ESPControlGui"
gui.IgnoreGuiInset = true
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.DisplayOrder = 1000
gui.Parent = PlayerGui or game:GetService("CoreGui") -- Fallback para CoreGui
print("GUI principal criada: ESPControlGui")

-- Frame principal (180x220)
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 220)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
frame.BorderSizePixel = 0
frame.ZIndex = 1000
frame.Parent = gui
print("Frame principal criado (180x220)")

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -30, 0, 20)
titleLabel.Position = UDim2.new(0, 5, 0, 5)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Dono Moises"
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.TextScaled = true
titleLabel.ZIndex = 1001
titleLabel.Parent = frame
print("Título criado")

-- Botão de minimizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 20, 0, 20)
minimizeButton.Position = UDim2.new(1, -25, 0, 5)
minimizeButton.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
minimizeButton.Text = "-"
minimizeButton.TextColor3 = Color3.new(1, 1, 1)
minimizeButton.ZIndex = 1001
minimizeButton.Parent = frame
print("Botão de minimizar criado")

-- Botão para ativar/desativar ESP
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 160, 0, 30)
toggleButton.Position = UDim2.new(0, 10, 0, 30)
toggleButton.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
toggleButton.Text = "ESP: OFF"
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.TextScaled = true
toggleButton.ZIndex = 1001
toggleButton.Parent = frame
print("Botão ESP criado")

-- Botão de velocidade
local speedButton = Instance.new("TextButton")
speedButton.Size = UDim2.new(0, 160, 0, 30)
speedButton.Position = UDim2.new(0, 10, 0, 65)
speedButton.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
speedButton.Text = "Velocidade"
speedButton.TextColor3 = Color3.new(1, 1, 1)
speedButton.TextScaled = true
speedButton.ZIndex = 1001
speedButton.Parent = frame
print("Botão Velocidade criado")

-- Caixa de texto para velocidade
local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0, 160, 0, 20)
speedBox.Position = UDim2.new(0, 10, 0, 100)
speedBox.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
speedBox.TextColor3 = Color3.new(1, 1, 1)
speedBox.TextScaled = true
speedBox.Text = tostring(defaultWalkSpeed)
speedBox.Visible = false
speedBox.ZIndex = 1001
speedBox.Parent = frame
print("Caixa de velocidade criada")

-- Botão para ativar/desativar auto clique
local autoClickButton = Instance.new("TextButton")
autoClickButton.Size = UDim2.new(0, 160, 0, 30)
autoClickButton.Position = UDim2.new(0, 10, 0, 125)
autoClickButton.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
autoClickButton.Text = "Auto Clique: OFF"
autoClickButton.TextColor3 = Color3.new(1, 1, 1)
autoClickButton.TextScaled = true
autoClickButton.ZIndex = 1001
autoClickButton.Parent = frame
print("Botão Auto Clique criado")

-- Botão para estamina infinita
local staminaButton = Instance.new("TextButton")
staminaButton.Size = UDim2.new(0, 160, 0, 30)
staminaButton.Position = UDim2.new(0, 10, 0, 160)
staminaButton.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
staminaButton.Text = "Estamina Infinita: OFF"
staminaButton.TextColor3 = Color3.new(1, 1, 1)
staminaButton.TextScaled = true
staminaButton.ZIndex = 1001
staminaButton.Parent = frame
print("Botão Estamina Infinita criado")

-- Frame circular para estado minimizado (30x30)
local minimizedCircle = Instance.new("Frame")
minimizedCircle.Size = UDim2.new(0, 30, 0, 30)
minimizedCircle.Position = frame.Position
minimizedCircle.BackgroundColor3 = Color3.new(0.4, 0.4, 0.4)
minimizedCircle.BorderSizePixel = 0
minimizedCircle.Visible = false
minimizedCircle.ZIndex = 1000
minimizedCircle.Parent = gui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = minimizedCircle
local plusLabel = Instance.new("TextLabel")
plusLabel.Size = UDim2.new(1, 0, 1, 0)
plusLabel.BackgroundTransparency = 1
plusLabel.Text = "+"
plusLabel.TextColor3 = Color3.new(1, 1, 1)
plusLabel.TextScaled = true
plusLabel.ZIndex = 1001
plusLabel.Parent = minimizedCircle
print("Frame minimizado criado (30x30)")

-- Função para arrastar
local function makeDraggable(element)
    local dragging, dragInput, dragStart, startPos
    local function updateInput(input)
        local delta = input.Position - dragStart
        element.Position = UDim2.new(0, startPos.X.Offset + delta.X, 0, startPos.Y.Offset + delta.Y)
    end

    element.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = element.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    element.InputChanged:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            updateInput(input)
        end
    end)
end
makeDraggable(frame)
makeDraggable(minimizedCircle)
print("Funcionalidade de arrastar adicionada")

-- Função para minimizar/expandir
local function toggleMinimize()
    isMinimized = not isMinimized
    frame.Visible = not isMinimized
    minimizedCircle.Visible = isMinimized
    minimizedCircle.Position = frame.Position
    speedBox.Visible = speedBoxVisible and not isMinimized
    if isMinimized then
        if autoClickEnabled then toggleAutoClick() end
        if infiniteStaminaEnabled then toggleInfiniteStamina() end
    end
    print("GUI " .. (isMinimized and "minimizada" or "expandida"))
end

-- Função para mostrar/esconder caixa de velocidade
local function toggleSpeedBox()
    speedBoxVisible = not speedBoxVisible
    speedBox.Visible = speedBoxVisible and not isMinimized
    print("Caixa de velocidade " .. (speedBoxVisible and "visível" or "escondida"))
end

-- Função para aplicar velocidade
local function applySpeed(input)
    local value = tonumber(input)
    if not value then
        speedBox.Text = tostring(defaultWalkSpeed)
        print("Erro: Velocidade inválida")
        return
    end
    value = math.clamp(value, 1, 99999)
    pcall(function()
        -- Aplicar WalkSpeed no Humanoid
        local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = value
            print("WalkSpeed aplicado: " .. value)
        else
            print("Erro: Humanoid não encontrado")
        end
        -- Procurar valores de velocidade personalizados
        for _, obj in pairs(PlayerGui:GetDescendants()) do
            if obj:IsA("NumberValue") and (obj.Name:lower():find("speed") or obj.Name:lower():find("move")) then
                obj.Value = value
                print("Velocidade personalizada aplicada via PlayerGui: " .. obj.Name .. " = " .. value)
            end
        end
        for _, obj in pairs(LocalPlayer:GetDescendants()) do
            if obj:IsA("NumberValue") and (obj.Name:lower():find("speed") or obj.Name:lower():find("move")) then
                obj.Value = value
                print("Velocidade personalizada aplicada via LocalPlayer: " .. obj.Name .. " = " .. value)
            end
        end
        -- Loop para reaplicar contra sobrescrição
        if speedConnection then
            speedConnection:Disconnect()
        end
        speedConnection = RunService.Heartbeat:Connect(function()
            if humanoid then
                humanoid.WalkSpeed = value
            end
        end)
    end)
    speedBox.Text = tostring(value)
    print("Velocidade final aplicada: " .. value)
end

-- Filtrar entrada para aceitar apenas números
speedBox:GetPropertyChangedSignal("Text"):Connect(function()
    local text = speedBox.Text
    if text ~= "" and not tonumber(text) then
        speedBox.Text = tostring(defaultWalkSpeed)
        print("Entrada inválida, restaurada para: " .. defaultWalkSpeed)
    end
end)

-- Aplicar velocidade
speedBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        applySpeed(speedBox.Text)
    end
end)

-- Função para simular clique no centro da tela
local function simulateClick()
    local viewportSize = workspace.CurrentCamera.ViewportSize
    local centerX, centerY = viewportSize.X / 2, viewportSize.Y / 2
    pcall(function()
        VirtualInputManager:SendMouseButtonEvent(centerX, centerY, 0, true, game, 0)
        task.wait(0.01)
        VirtualInputManager:SendMouseButtonEvent(centerX, centerY, 0, false, game, 0)
        print("Clique simulado no centro da tela (" .. centerX .. ", " .. centerY .. ")")
    end)
end

-- Função para ativar/desativar auto clique
local function toggleAutoClick()
    autoClickEnabled = not autoClickEnabled
    autoClickButton.Text = autoClickEnabled and "Auto Clique: ON" or "Auto Clique: OFF"
    if autoClickEnabled then
        if autoClickConnection then
            autoClickConnection:Disconnect()
        end
        autoClickConnection = RunService.Heartbeat:Connect(function()
            if autoClickEnabled then
                simulateClick()
                task.wait(0.025) -- 40 cliques/segundo
            end
        end)
        print("Auto clique ativado")
    else
        if autoClickConnection then
            autoClickConnection:Disconnect()
            autoClickConnection = nil
        end
        print("Auto clique desativado")
    end
end

-- Função para ativar/desativar estamina infinita
local function toggleInfiniteStamina()
    infiniteStaminaEnabled = not infiniteStaminaEnabled
    staminaButton.Text = infiniteStaminaEnabled and "Estamina Infinita: ON" or "Estamina Infinita: OFF"
    if infiniteStaminaEnabled then
        if staminaConnection then
            staminaConnection:Disconnect()
        end
        staminaConnection = RunService.Heartbeat:Connect(function()
            if infiniteStaminaEnabled then
                pcall(function()
                    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid:FindFirstChild("Stamina") then
                        humanoid.Stamina = humanoid.MaxStamina or 100
                        print("Estamina mantida cheia via Humanoid")
                    end
                    for _, obj in pairs(PlayerGui:GetDescendants()) do
                        if obj:IsA("NumberValue") and obj.Name:lower():find("stamina") then
                            obj.Value = 100
                            print("Estamina mantida cheia via PlayerGui: " .. obj.Name)
                        end
                    end
                    for _, obj in pairs(LocalPlayer:GetDescendants()) do
                        if obj:IsA("NumberValue") and obj.Name:lower():find("stamina") then
                            obj.Value = 100
                            print("Estamina mantida cheia via LocalPlayer: " .. obj.Name)
                        end
                    end
                end)
            end
        end)
        print("Estamina infinita ativada")
    else
        if staminaConnection then
            staminaConnection:Disconnect()
            staminaConnection = nil
        end
        print("Estamina infinita desativada")
    end
end

-- Função para criar ESP
local function createESP(player)
    if player == LocalPlayer then return end

    local function onCharacterAdded(character)
        local head = character:FindFirstChild("Head")
        local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
        if not head or not humanoidRootPart then return end

        local outline = Instance.new("SelectionBox")
        outline.Name = "ESPOutline"
        outline.Adornee = character
        outline.LineThickness = lineThickness
        outline.Color3 = Color3.fromRGB(255, 0, 0)
        outline.SurfaceTransparency = 1
        outline.Transparency = 0
        outline.Visible = espEnabled
        outline.Parent = humanoidRootPart

        local nameBillboard = Instance.new("BillboardGui")
        nameBillboard.Name = "NameESP"
        nameBillboard.Adornee = head
        nameBillboard.Size = UDim2.new(0, 100, 0, 50)
        nameBillboard.StudsOffset = Vector3.new(0, 3, 0)
        nameBillboard.AlwaysOnTop = true
        nameBillboard.Enabled = espEnabled
        nameBillboard.Parent = head

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        nameLabel.TextScaled = true
        nameLabel.Parent = nameBillboard

        local distanceBillboard = Instance.new("BillboardGui")
        distanceBillboard.Name = "DistanceESP"
        distanceBillboard.Adornee = humanoidRootPart
        distanceBillboard.Size = UDim2.new(0, 100, 0, 20)
        distanceBillboard.StudsOffset = Vector3.new(0, -2, 0)
        distanceBillboard.AlwaysOnTop = true
        distanceBillboard.Enabled = espEnabled
        distanceBillboard.Parent = humanoidRootPart

        local distanceLabel = Instance.new("TextLabel")
        distanceLabel.Size = UDim2.new(1, 0, 1, 0)
        distanceLabel.BackgroundTransparency = 1
        distanceLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        distanceLabel.Text = "Distância: 0"
        distanceLabel.TextScaled = true
        distanceLabel.Parent = distanceBillboard

        espData[player] = {
            outline = outline,
            nameBillboard = nameBillboard,
            distanceBillboard = distanceBillboard,
            distanceLabel = distanceLabel,
            character = character
        }
    end

    if player.Character then
        onCharacterAdded(player.Character)
    end
    player.CharacterAdded:Connect(onCharacterAdded)
end

-- Função para atualizar ESP
local function updateESP()
    if not espEnabled then return end
    for player, data in pairs(espData) do
        if data.character and data.character.Parent then
            local localChar = LocalPlayer.Character
            if localChar and localChar:FindFirstChild("HumanoidRootPart") then
                local localPos = localChar.HumanoidRootPart.Position
                local targetPos = data.character.HumanoidRootPart.Position
                local distance = (localPos - targetPos).Magnitude
                data.distanceLabel.Text = string.format("Distância: %.1f", distance)
            end
        else
            data.outline:Destroy()
            data.nameBillboard:Destroy()
            data.distanceBillboard:Destroy()
            espData[player] = nil
        end
    end
end

-- Função para ativar/desativar ESP
local function toggleESP()
    espEnabled = not espEnabled
    toggleButton.Text = espEnabled and "ESP: ON" or "ESP: OFF"
    for player, data in pairs(espData) do
        data.outline.Visible = espEnabled
        data.nameBillboard.Enabled = espEnabled
        data.distanceBillboard.Enabled = espEnabled
    end
    print("ESP " .. (espEnabled and "ativado" or "desativado"))
end

-- Conectar botões
minimizeButton.MouseButton1Click:Connect(toggleMinimize)
toggleButton.MouseButton1Click:Connect(toggleESP)
speedButton.MouseButton1Click:Connect(toggleSpeedBox)
autoClickButton.MouseButton1Click:Connect(toggleAutoClick)
staminaButton.MouseButton1Click:Connect(toggleInfiniteStamina)
minimizedCircle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        toggleMinimize()
    end
end)
print("Botões conectados")

-- Inicializar ESP
for _, player in pairs(Players:GetPlayers()) do
    createESP(player)
end
Players.PlayerAdded:Connect(createESP)
print("ESP inicializado")

-- Limpar quando jogador sai
Players.PlayerRemoving:Connect(function(player)
    if espData[player] then
        espData[player].outline:Destroy()
        espData[player].nameBillboard:Destroy()
        espData[player].distanceBillboard:Destroy()
        espData[player] = nil
    end
end)

-- Atualizar ESP
RunService.RenderStepped:Connect(updateESP)
print("Script inicializado com sucesso")
