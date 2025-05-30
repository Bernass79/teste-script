-- Carrega a biblioteca Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Cria a janela
local Window = Fluent:CreateWindow({
    Title = "🔥 GRAND-Menu Mini City(ACESSO ANTECIPADO) 🔥",
    SubTitle = "by Bernardo",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

-- Função para notificação
local function notificar(titulo, mensagem, duracao)
    Fluent:Notify({
        Title = titulo,
        Content = mensagem,
        Duration = duracao or 3
    })
end





-- Aba Infos
local InfosTab = Window:AddTab({ Title = "Infos", Icon = "info" })
notificar("GRAND-Menu Injected 💸", "Obrigado por Adquirir nosso menu!", 36)
InfosTab:AddParagraph({
    Title = "Grand Menu ℹ️",
    Content = "+Menu Reconstruído "
})
InfosTab:AddButton({
    Title = "Ver Atualizações 📜",
    Callback = function()
        print("Abrindo atualizações...")
    end
})

-- Botão para copiar invite do Discord
InfosTab:AddButton({
    Title = "Copiar Invite do Discord",
    Callback = function()
        setclipboard("https://discord.gg/jKYYKd6ncE") -- Substitua por um invite válido
        notificar("Discord", "Invite copiado para a área de transferência!")
    end
})

-- Aba Combate
local CombatTab = Window:AddTab({ Title = "Combate 🗡️", Icon = "sword" })

-- Noclip
local noclipEnabled = false
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()

local function toggleNoclip(state)
    noclipEnabled = state
    notificar(noclipEnabled and "Noclip Ativado" or "Noclip Desativado", noclipEnabled and "Agora você pode atravessar objetos!" or "Agora você colide normalmente.", 2)
end

RunService.Stepped:Connect(function()
    if noclipEnabled then
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

CombatTab:AddToggle("NoclipToggle", {
    Title = "NoClip 💥",
    Default = false,
    Callback = function(state)
        toggleNoclip(state)
    end
})

-- Speed
local speedValue = 16
local speedEnabled = false
local function getHumanoid()
    if localPlayer.Character then
        return localPlayer.Character:FindFirstChildOfClass("Humanoid")
    end
    return nil
end

local function setSpeed(value)
    local humanoid = getHumanoid()
    if humanoid then
        humanoid.WalkSpeed = value
    end
end

task.spawn(function()
    while task.wait(0.05) do
        if speedEnabled then
            local humanoid = getHumanoid()
            if humanoid and humanoid.WalkSpeed ~= speedValue then
                humanoid.WalkSpeed = speedValue
            end
        end
    end
end)

localPlayer.CharacterAdded:Connect(function(character)
    repeat wait() until character:FindFirstChildOfClass("Humanoid")
    if speedEnabled then
        setSpeed(speedValue)
    end
end)

CombatTab:AddToggle("SpeedToggle", {
    Title = "Ativar ajuste Velocidade⚡",
    Default = false,
    Callback = function(value)
        speedEnabled = value
        if speedEnabled then
            setSpeed(speedValue)
        else
            setSpeed(16)
        end
    end
})

CombatTab:AddSlider("SpeedSlider", {
    Title = "Ajustar Velocidade🌪️",
    Min = 16,
    Max = 100,
    Default = 16,
    Rounding = 1,
    Suffix = "Studs/s",
    Callback = function(value)
        speedValue = value
    end
})

-- Anti-Fall Damage
local antiFallEnabled = false

local function antiFallDamage()
    while antiFallEnabled do
        task.wait(0.1)
        if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and rootPart then
                if rootPart.Velocity.Y < -50 then
                    rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z)
                end
            end
        end
    end
end

CombatTab:AddToggle("AntiFallToggle", {
    Title = "Anti-Queda🪂",
    Default = false,
    Callback = function(value)
        antiFallEnabled = value
        if antiFallEnabled then
            task.spawn(antiFallDamage)
        end
    end
})

-- Hitbox
local hitboxEnabled = false
local defaultHitboxSize = Vector3.new(5, 5, 5)
local hitboxSize = defaultHitboxSize
local originalSizes = {}
local minSize = 1
local maxSize = 20
local sizeStep = 1

local function expandHitbox()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            if humanoidRootPart and head then
                if not originalSizes[player] then
                    originalSizes[player] = {
                        HumanoidRootPart = humanoidRootPart.Size,
                        Head = head.Size
                    }
                end
                if hitboxEnabled then
                    humanoidRootPart.Size = hitboxSize
                    head.Size = hitboxSize
                    humanoidRootPart.Transparency = 0.7
                    head.Transparency = 0.7
                    humanoidRootPart.CanCollide = false
                    head.CanCollide = false
                else
                    humanoidRootPart.Size = originalSizes[player].HumanoidRootPart
                    head.Size = originalSizes[player].Head
                    humanoidRootPart.Transparency = 0
                    head.Transparency = 0
                    humanoidRootPart.CanCollide = true
                    head.CanCollide = true
                end
            end
        end
    end
end

game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if hitboxEnabled then
            task.wait(0.1)
            expandHitbox()
        end
    end)
end)

game.Players.PlayerRemoving:Connect(function(player)
    originalSizes[player] = nil
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if hitboxEnabled then
        expandHitbox()
    end
end)

CombatTab:AddToggle("HitboxToggle", {
    Title = "Ativar Hitbox 📦",
    Default = false,
    Callback = function(value)
        hitboxEnabled = value
        if not hitboxEnabled then
            for player, sizes in pairs(originalSizes) do
                if player.Character then
                    local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
                    local head = player.Character:FindFirstChild("Head")
                    if humanoidRootPart and head then
                        humanoidRootPart.Size = sizes.HumanoidRootPart
                        head.Size = sizes.Head
                        humanoidRootPart.Transparency = 0
                        head.Transparency = 0
                        humanoidRootPart.CanCollide = true
                        head.CanCollide = true
                    end
                end
            end
        end
        print(hitboxEnabled and "Hitbox ativada!" or "Hitbox desativada!")
    end
})

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HitboxGui"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false
screenGui.Enabled = false

local frame = Instance.new("Frame")
frame.Name = "HitboxFrame"
frame.Size = UDim2.new(0, 220, 0, 50)
frame.Position = UDim2.new(0.5, -110, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

local sizeLabel = Instance.new("TextLabel")
sizeLabel.Name = "SizeLabel"
sizeLabel.Size = UDim2.new(0.9, 0, 0.8, 0)
sizeLabel.Position = UDim2.new(0.05, 0, 0.1, 0)
sizeLabel.BackgroundTransparency = 1
sizeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
sizeLabel.Font = Enum.Font.Gotham
sizeLabel.TextSize = 16
sizeLabel.Text = "Tamanho: " .. hitboxSize.X
sizeLabel.Parent = frame

CombatTab:AddSlider("HitboxSizeSlider", {
    Title = "Tamanho da Hitbox 🎯",
    Min = minSize,
    Max = maxSize,
    Default = hitboxSize.X,
    Rounding = sizeStep,
    Suffix = " studs",
    Callback = function(value)
        hitboxSize = Vector3.new(value, value, value)
        sizeLabel.Text = "Tamanho: " .. hitboxSize.X
        if hitboxEnabled then
            expandHitbox()
        end
    end
})

-- Aba Auto-Farms
local AutoFarmsTab = Window:AddTab({ Title = "AUTO-FARMS🌱", Icon = "sprout" })
AutoFarmsTab:AddButton({
    Title = "Injetar Farms NECESSÁRIO 🔧",
    Callback = function()
        print("INJETADOOO")
        for _, prompt in ipairs(workspace:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
            end
        end
        workspace.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("ProximityPrompt") then
                descendant.HoldDuration = 0
            end
        end)
    end
})

AutoFarmsTab:AddButton({
    Title = "Auto-Colect Gari🚛",
    Callback = function()
        print("Auto-Farm Gari ativado!")
        local function findNearestLEXOS()
            local player = game:GetService("Players").LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
            local lexosObjects = {}
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj.Name == "LEXOS" and obj:IsA("BasePart") and obj.CanCollide then
                    table.insert(lexosObjects, obj)
                end
            end
            if #lexosObjects == 0 then
                print("Nenhum LEXOS com colisão ativada encontrado!")
                return nil
            end
            local nearestLEXOS = nil
            local minDistance = math.huge
            for _, lexos in ipairs(lexosObjects) do
                local distance = (humanoidRootPart.Position - lexos.Position).Magnitude
                if distance < minDistance then
                    minDistance = distance
                    nearestLEXOS = lexos
                end
            end
            return nearestLEXOS
        end
        local function teleportToNearestLEXOS()
            local nearestLEXOS = findNearestLEXOS()
            if nearestLEXOS then
                local player = game:GetService("Players").LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
                local centerPosition = nearestLEXOS.Position
                humanoidRootPart.CFrame = CFrame.new(centerPosition)
            end
        end
        for _, prompt in ipairs(workspace:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
            end
        end
        workspace.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("ProximityPrompt") then
                descendant.HoldDuration = 0
            end
        end)
        while true do
            teleportToNearestLEXOS()
            wait(0.3)
            local nearestLEXOS = findNearestLEXOS()
            if nearestLEXOS then
                for _, child in ipairs(nearestLEXOS:GetChildren()) do
                    if child:IsA("ProximityPrompt") then
                        fireproximityprompt(child)
                        break
                    end
                end
            end
            wait(0.3)
        end
    end
})

AutoFarmsTab:AddButton({
    Title = "Auto-Colect Gás⛽",
    Callback = function()
        print("Auto-Farm Gás ativado!")
        loadstring(game:HttpGet('https://raw.githubusercontent.com/Rafaasxs/Nexus-Menu-/refs/heads/main/tesao'))()
    end
})

AutoFarmsTab:AddButton({
    Title = "Auto-Colect Minerador⛏️",
    Callback = function()
        print("Auto-Farm Rock ativado!")
        local function findNearestRock()
            local player = game:GetService("Players").LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
            local rockObjects = {}
            for _, obj in ipairs(workspace:GetDescendants()) do
                if obj.Name == "Rock" and obj:IsA("BasePart") and obj.CanCollide then
                    table.insert(rockObjects, obj)
                end
            end
            if #rockObjects == 0 then
                print("Nenhum Rock com colisão ativada encontrado!")
                return nil
            end
            local nearestRock = nil
            local minDistance = math.huge
            for _, rock in ipairs(rockObjects) do
                local distance = (humanoidRootPart.Position - rock.Position).Magnitude
                if distance < minDistance then
                    minDistance = distance
                    nearestRock = rock
                end
            end
            return nearestRock
        end
        local function teleportToNearestRock()
            local nearestRock = findNearestRock()
            if nearestRock then
                local player = game:GetService("Players").LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
                local centerPosition = nearestRock.Position
                humanoidRootPart.CFrame = CFrame.new(centerPosition)
            end
        end
        for _, prompt in ipairs(workspace:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
            end
        end
        workspace.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
            end
        end)
        while true do
            teleportToNearestRock()
            wait(0.3)
            local nearestRock = findNearestRock()
            if nearestRock then
                for _, child in ipairs(nearestRock:GetChildren()) do
                    if child:IsA("ProximityPrompt") then
                        fireproximityprompt(child)
                        break
                    end
                end
            end
            wait(0.3)
        end
    end
})

AutoFarmsTab:AddSection("Auto farm peça")

AutoFarmsTab:AddButton({
    Title = "Salvar farm Posição da fac",
    Callback = function()
        local player = game.Players.LocalPlayer
        local character = player.Character
        if character then
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            if humanoidRootPart then
                _G.SavedPosition = humanoidRootPart.Position
                notificar("Posição Salva", "Sua posição foi salva com sucesso!", 2)
            end
        end
    end
})

local function fireAllProximityPrompts()
    local objetosMissao = workspace:FindFirstChild("MapaGeral")
        and workspace.MapaGeral:FindFirstChild("FavelaV2")
        and workspace.MapaGeral.FavelaV2:FindFirstChild("objetosMissao")
    if objetosMissao then
        for _, prompt in ipairs(objetosMissao:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
                prompt:InputHoldBegin()
                prompt:InputHoldEnd()
            end
        end
    end
end

local function findPlayerProximityPrompt()
    local playerName = game.Players.LocalPlayer.Name
    local objetosMissao = workspace:FindFirstChild("MapaGeral")
        and workspace.MapaGeral:FindFirstChild("FavelaV2")
        and workspace.MapaGeral.FavelaV2:FindFirstChild("objetosMissao")
    if objetosMissao then
        for _, prompt in ipairs(objetosMissao:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") and prompt.Name == playerName then
                return prompt
            end
        end
    end
    return nil
end

AutoFarmsTab:AddToggle("FarmToggle", {
    Title = "Ativar Farm",
    Default = false,
    Callback = function(value)
        while value and _G.SavedPosition do
            local player = game.Players.LocalPlayer
            local character = player.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then
                break
            end
            fireAllProximityPrompts()
            character.HumanoidRootPart.CFrame = CFrame.new(_G.SavedPosition)
            task.wait(0.35)
            game:GetService("ReplicatedStorage"):WaitForChild("RemoteNovos")
                :WaitForChild("trabalhos"):FireServer("missaoPECAS")
            local playerPrompt = findPlayerProximityPrompt()
            if playerPrompt and playerPrompt.Parent then
                character.HumanoidRootPart.CFrame = CFrame.new(playerPrompt.Parent.Position)
            end
            task.wait(0.2)
        end
    end
})

AutoFarmsTab:AddButton({
    Title = "Auto Click",
    Callback = function()
        print("Auto Click ativado!")
        for _, prompt in ipairs(workspace:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
            end
        end
        workspace.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("ProximityPrompt") then
                descendant.HoldDuration = 0
            end
        end)
        local function findNearestPrompt()
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
            local nearestPrompt = nil
            local shortestDistance = math.huge
            for _, prompt in ipairs(workspace:GetDescendants()) do
                if prompt:IsA("ProximityPrompt") and prompt.Parent and prompt.Parent:IsA("BasePart") then
                    local distance = (humanoidRootPart.Position - prompt.Parent.Position).Magnitude
                    if distance <= prompt.MaxActivationDistance and distance < shortestDistance then
                        shortestDistance = distance
                        nearestPrompt = prompt
                    end
                end
            end
            return nearestPrompt
        end
        while true do
            local nearestPrompt = findNearestPrompt()
            if nearestPrompt then
                fireproximityprompt(nearestPrompt)
                print("Auto Click ativou um ProximityPrompt!")
            end
            wait(0.1)
        end
    end
})



-- Inicialização
Window:SelectTab(1) -- Seleciona a aba Key System ao iniciar
notificar("Menu Carregado", "GRAND-SHOP Menu Mini City carregado com sucesso!", 5)

-- Sistema de Voo com GUI
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Variáveis do sistema (nomes únicos)
local isFlying = false
local flightSpeed = 15
local minSpeed = 1
local maxSpeed = 999
local speedStep = 5
local isAntiFallEnabled = false
local isUIMinimized = false

-- Criação da GUI
local ok, err = pcall(function()
    local screenGuiFly = Instance.new("ScreenGui")
    screenGuiFly.Name = "FlyGui"
    screenGuiFly.Parent = LocalPlayer:WaitForChild("PlayerGui", 10)
    screenGuiFly.ResetOnSpawn = false
    screenGuiFly.Enabled = true -- Garante que a GUI esteja habilitada
    screenGuiFly.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local frameFly = Instance.new("Frame")
    frameFly.Name = "FlyFrame"
    frameFly.Size = UDim2.new(0, 220, 0, 120)
    frameFly.Position = UDim2.new(0.5, -110, 0.1, 0) -- Centralizado no topo
    frameFly.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    frameFly.BorderSizePixel = 0
    frameFly.Active = true
    frameFly.Draggable = true
    frameFly.Parent = screenGuiFly
    local cornerFly = Instance.new("UICorner")
    cornerFly.CornerRadius = UDim.new(0, 12)
    cornerFly.Parent = frameFly

    local buttonFly = Instance.new("TextButton")
    buttonFly.Name = "FlyButton"
    buttonFly.Size = UDim2.new(0.9, 0, 0.3, 0)
    buttonFly.Position = UDim2.new(0.05, 0, 0.1, 0)
    buttonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    buttonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
    buttonFly.Font = Enum.Font.GothamBold
    buttonFly.TextSize = 18
    buttonFly.Text = "Ativar Voo ✈️"
    buttonFly.Parent = frameFly
    local buttonCornerFly = Instance.new("UICorner")
    buttonCornerFly.CornerRadius = UDim.new(0, 8)
    buttonCornerFly.Parent = buttonFly

    local speedLabel = Instance.new("TextLabel")
    speedLabel.Name = "SpeedLabel"
    speedLabel.Size = UDim2.new(0.9, 0, 0.2, 0)
    speedLabel.Position = UDim2.new(0.05, 0, 0.45, 0)
    speedLabel.BackgroundTransparency = 1
    speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedLabel.Font = Enum.Font.Gotham
    speedLabel.TextSize = 16
    speedLabel.Text = "Velocidade: " .. flightSpeed
    speedLabel.Parent = frameFly

    local plusButtonFly = Instance.new("TextButton")
    plusButtonFly.Name = "PlusButton"
    plusButtonFly.Size = UDim2.new(0.2, 0, 0.2, 0)
    plusButtonFly.Position = UDim2.new(0.75, 0, 0.45, 0)
    plusButtonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    plusButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
    plusButtonFly.Font = Enum.Font.GothamBold
    plusButtonFly.TextSize = 18
    plusButtonFly.Text = "+"
    plusButtonFly.Parent = frameFly
    local plusCornerFly = Instance.new("UICorner")
    plusCornerFly.CornerRadius = UDim.new(0, 8)
    plusCornerFly.Parent = plusButtonFly

    local minusButtonFly = Instance.new("TextButton")
    minusButtonFly.Name = "MinusButton"
    minusButtonFly.Size = UDim2.new(0.2, 0, 0.2, 0)
    minusButtonFly.Position = UDim2.new(0.05, 0, 0.45, 0)
    minusButtonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    minusButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
    minusButtonFly.Font = Enum.Font.GothamBold
    minusButtonFly.TextSize = 18
    minusButtonFly.Text = "-"
    minusButtonFly.Parent = frameFly
    local minusCornerFly = Instance.new("UICorner")
    minusCornerFly.CornerRadius = UDim.new(0, 8)
    minusCornerFly.Parent = minusButtonFly

    local antiFallToggle = Instance.new("TextButton")
    antiFallToggle.Name = "AntiFallToggle"
    antiFallToggle.Size = UDim2.new(0.9, 0, 0.2, 0)
    antiFallToggle.Position = UDim2.new(0.05, 0, 0.7, 0)
    antiFallToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    antiFallToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    antiFallToggle.Font = Enum.Font.GothamBold
    antiFallToggle.TextSize = 18
    antiFallToggle.Text = "Anti-Dano: OFF"
    antiFallToggle.Parent = frameFly
    local antiFallCorner = Instance.new("UICorner")
    antiFallCorner.CornerRadius = UDim.new(0, 8)
    antiFallCorner.Parent = antiFallToggle

    local minimizeButtonFly = Instance.new("TextButton")
    minimizeButtonFly.Name = "MinimizeButton"
    minimizeButtonFly.Size = UDim2.new(0.1, 0, 0.1, 0)
    minimizeButtonFly.Position = UDim2.new(0.95, 0, 0.05, 0)
    minimizeButtonFly.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    minimizeButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
    minimizeButtonFly.Font = Enum.Font.GothamBold
    minimizeButtonFly.TextSize = 14
    minimizeButtonFly.Text = "-"
    minimizeButtonFly.Parent = frameFly
    local minimizeCornerFly = Instance.new("UICorner")
    minimizeCornerFly.CornerRadius = UDim.new(0, 4)
    minimizeCornerFly.Parent = minimizeButtonFly

    local closeButtonFly = Instance.new("TextButton")
    closeButtonFly.Name = "CloseButton"
    closeButtonFly.Size = UDim2.new(0.1, 0, 0.1, 0)
    closeButtonFly.Position = UDim2.new(0.85, 0, 0.05, 0)
    closeButtonFly.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
    closeButtonFly.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButtonFly.Font = Enum.Font.GothamBold
    closeButtonFly.TextSize = 14
    closeButtonFly.Text = "X"
    closeButtonFly.Parent = frameFly
    local closeCornerFly = Instance.new("UICorner")
    closeCornerFly.CornerRadius = UDim.new(0, 4)
    closeCornerFly.Parent = closeButtonFly

    local controlFrame = Instance.new("Frame")
    controlFrame.Name = "ControlFrame"
    controlFrame.Size = UDim2.new(0, 100, 0, 50)
    controlFrame.Position = UDim2.new(0.1, 0, 0.8, 0)
    controlFrame.BackgroundTransparency = 1
    controlFrame.Parent = screenGuiFly

    local function createControlButton(name, position, text)
        local btn = Instance.new("TextButton")
        btn.Name = name
        btn.Size = UDim2.new(0, 50, 0, 50)
        btn.Position = position
        btn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 20
        btn.Text = text
        btn.Parent = controlFrame
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 8)
        btnCorner.Parent = btn
        return btn
    end

    local flyToggleButton = createControlButton("FlyToggleButton", UDim2.new(0, 0, 0, 0), "✈️")

    -- Funções de voo
    local function disableCollisions(character)
        if not character then return end
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end

    local function enableCollisions(character)
        if not character then return end
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end

    local function antiFallDamage()
        while isAntiFallEnabled do
            task.wait(0.1)
            local character = LocalPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                local rootPart = character.HumanoidRootPart
                if humanoid and rootPart and not isFlying and rootPart.Velocity.Y < -50 then
                    rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z)
                end
            end
        end
    end

    local function startFlying()
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") or not character:FindFirstChildOfClass("Humanoid") then
            print("Personagem não encontrado!")
            return false
        end
        local humanoidRootPart = character.HumanoidRootPart
        local humanoid = character:FindFirstChildOfClass("Humanoid")

        humanoid.PlatformStand = true
        humanoidRootPart.Anchored = true
        disableCollisions(character)
        isFlying = true
        buttonFly.Text = "Desativar Voo"
        buttonFly.BackgroundColor3 = Color3.fromRGB(100, 50, 50)
        flyToggleButton.Text = "✈️ OFF"

        local connection
        connection = RunService.Heartbeat:Connect(function(deltaTime)
            if not isFlying or not character or not humanoidRootPart or not humanoid then
                if connection then connection:Disconnect() end
                return
            end

            local moveDirection = Vector3.new(0, 0, 0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + workspace.CurrentCamera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection - workspace.CurrentCamera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection - workspace.CurrentCamera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + workspace.CurrentCamera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDirection = moveDirection - Vector3.new(0, 1, 0) end

            if moveDirection.Magnitude > 0 then
                moveDirection = moveDirection.Unit
                local speed = flightSpeed * deltaTime * 10
                humanoidRootPart.Anchored = false
                humanoidRootPart.CFrame = humanoidRootPart.CFrame + moveDirection * speed
                local lookVector = workspace.CurrentCamera.CFrame.LookVector
                humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position) * CFrame.Angles(0, math.atan2(lookVector.X, lookVector.Z), 0)
                humanoidRootPart.Anchored = true
            end
        end)
        print("Voo iniciado!")
        return true
    end

    local function stopFlying()
        local character = LocalPlayer.Character
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        local humanoidRootPart = character and character:FindFirstChild("HumanoidRootPart")
        if humanoid then humanoid.PlatformStand = false end
        if character then enableCollisions(character) end
        if humanoidRootPart then humanoidRootPart.Anchored = false end
        isFlying = false
        buttonFly.Text = "Ativar Voo ✈️"
        buttonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        flyToggleButton.Text = "✈️ ON"
        print("Voo parado!")
    end

    -- Conexões dos botões
    buttonFly.MouseButton1Click:Connect(function()
        if isFlying then
            stopFlying()
        else
            startFlying()
        end
    end)

    flyToggleButton.MouseButton1Click:Connect(function()
        if isFlying then
            stopFlying()
        else
            startFlying()
        end
    end)

    antiFallToggle.MouseButton1Click:Connect(function()
        isAntiFallEnabled = not isAntiFallEnabled
        antiFallToggle.Text = "Anti-Dano: " .. (isAntiFallEnabled and "ON" or "OFF")
        antiFallToggle.BackgroundColor3 = isAntiFallEnabled and Color3.fromRGB(100, 50, 50) or Color3.fromRGB(60, 60, 60)
        if isAntiFallEnabled then
            task.spawn(antiFallDamage)
        end
    end)

    local function toggleUIMinimize()
        if isUIMinimized then
            frameFly.Size = UDim2.new(0, 220, 0, 120)
            buttonFly.Size = UDim2.new(0.9, 0, 0.3, 0)
            speedLabel.Size = UDim2.new(0.9, 0, 0.2, 0)
            plusButtonFly.Size = UDim2.new(0.2, 0, 0.2, 0)
            minusButtonFly.Size = UDim2.new(0.2, 0, 0.2, 0)
            antiFallToggle.Size = UDim2.new(0.9, 0, 0.2, 0)
            minimizeButtonFly.Text = "-"
            isUIMinimized = false
        else
            frameFly.Size = UDim2.new(0, 150, 0, 80)
            buttonFly.Size = UDim2.new(0.9, 0, 0.4, 0)
            speedLabel.Size = UDim2.new(0.9, 0, 0.3, 0)
            plusButtonFly.Size = UDim2.new(0.2, 0, 0.3, 0)
            minusButtonFly.Size = UDim2.new(0.2, 0, 0.3, 0)
            antiFallToggle.Size = UDim2.new(0.9, 0, 0.3, 0)
            minimizeButtonFly.Text = "+"
            isUIMinimized = true
        end
    end

    minimizeButtonFly.MouseButton1Click:Connect(toggleUIMinimize)

    closeButtonFly.MouseButton1Click:Connect(function()
        screenGuiFly.Enabled = false
        if isFlying then stopFlying() end
    end)

    local function adjustSpeed(delta)
        flightSpeed = math.clamp(flightSpeed + delta, minSpeed, maxSpeed)
        speedLabel.Text = "Velocidade: " .. flightSpeed
    end

    plusButtonFly.MouseButton1Click:Connect(function() adjustSpeed(speedStep) end)
    minusButtonFly.MouseButton1Click:Connect(function() adjustSpeed(-speedStep) end)

    -- Drag functionality
    local dragging, dragInput, dragStart, startPos
    frameFly.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frameFly.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    frameFly.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frameFly.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    print("GUI 'FlyGui' criada e habilitada!")
end)

if not ok then
    warn("Erro ao criar a GUI: " .. tostring(err))
end



-- Criar nova aba para Aimbot
local AimbotTab = Window:AddTab({ Title = "Aimbot 🎯", Icon = "target" })

-- Variáveis de controle
local aimbotEnabled = false
local isMouseButton2Down = false
local Typing = false
local isDragging = false
local dragStart, startPos

-- Serviços
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Ambiente isolado
local Environment = {
    Settings = {
        Enabled = true,
        TeamCheck = false,
        AliveCheck = true,
        WallCheck = false,
        Sensitivity = 0.07, -- Suavização mais fluida, mas ainda forte
        LockPart = "Head" -- Mira na cabeça
    },
    FOVSettings = {
        Enabled = true,
        Visible = true,
        Amount = 90,
        Color = Color3.fromRGB(0, 255, 255), -- Ciano claro para suavidade
        LockedColor = Color3.fromRGB(255, 0, 0), -- Vermelho ao travar
        Transparency = 0.8, -- Levemente translúcido
        Sides = 60,
        Thickness = 1, -- Borda mais fina
        Filled = false
    },
    FOVCircle = nil, -- Inicializado em InitializeFOVCircle
    Locked = nil,
    ServiceConnections = {},
    Animation = nil
}

-- Inicializar o círculo de FOV
local function InitializeFOVCircle()
    if Environment.FOVCircle and Environment.FOVCircle.Remove then
        Environment.FOVCircle:Remove()
    end
    if Drawing then
        Environment.FOVCircle = Drawing.new("Circle")
        Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
        Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
        Environment.FOVCircle.Color = Environment.FOVSettings.Color
        Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
        Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
        Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
        Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
        print("Aimbot: Círculo de FOV inicializado")
    else
        print("Aimbot: ERRO: Biblioteca 'Drawing' não suportada pelo executor")
    end
end

-- Funções do aimbot
local function CancelLock()
    Environment.Locked = nil
    if Environment.Animation then Environment.Animation:Cancel() end
    if Environment.FOVCircle then
        Environment.FOVCircle.Color = Environment.FOVSettings.Color
    end
end

local function GetClosestPlayer()
    if not Environment.Locked then
        local RequiredDistance = Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000
        local closestPlayer = nil
        for _, v in ipairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                local humanoid = v.Character:FindFirstChildOfClass("Humanoid")
                local rootPart = v.Character:FindFirstChild("HumanoidRootPart")
                if not (humanoid and rootPart) then
                elseif Environment.Settings.AliveCheck and humanoid.Health <= 0 then
                elseif Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then
                elseif Environment.Settings.WallCheck then
                    local parts = Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, {LocalPlayer.Character, v.Character})
                    if #parts > 0 then
                    else
                        local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                        local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                        if Distance < RequiredDistance and OnScreen then
                            RequiredDistance = Distance
                            closestPlayer = v
                        end
                    end
                else
                    local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                    local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                    if Distance < RequiredDistance and OnScreen then
                        RequiredDistance = Distance
                        closestPlayer = v
                    end
                end
            end
        end
        Environment.Locked = closestPlayer
        if closestPlayer then
            print("Aimbot: Jogador travado: " .. closestPlayer.Name)
        end
    elseif Environment.Locked and Environment.Locked.Character and Environment.Locked.Character:FindFirstChild(Environment.Settings.LockPart) then
        local Vector, OnScreen = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)
        local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude
        if Distance > (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000) or not OnScreen then
            CancelLock()
        end
    else
        CancelLock()
    end
end

-- Conexões de eventos
local function SetupConnections()
    Environment.ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
        Typing = true
        print("Aimbot: Digitando: Aimbot pausado")
    end)

    Environment.ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
        Typing = false
        print("Aimbot: Digitando finalizado: Aimbot retomado")
    end)

    Environment.ServiceConnections.MouseButton2Down = UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            isMouseButton2Down = true
        end
    end)

    Environment.ServiceConnections.MouseButton2Up = UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            isMouseButton2Down = false
            CancelLock()
        end
    end)
end

-- Loop principal
local function LoadAimbot()
    InitializeFOVCircle() -- Reinicializa o círculo ao ativar
    Environment.ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if aimbotEnabled and Environment.Settings.Enabled and not Typing then
            if Environment.FOVCircle and Environment.FOVSettings.Enabled then
                Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
                Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
                Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
                Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
                Environment.FOVCircle.Color = Environment.FOVSettings.Color
                Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
                Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
                Environment.FOVCircle.Position = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
            elseif Environment.FOVCircle then
                Environment.FOVCircle.Visible = false
            end

            if isMouseButton2Down then
                GetClosestPlayer()
                if Environment.Locked and Environment.Locked.Character and Environment.Locked.Character:FindFirstChild(Environment.Settings.LockPart) then
                    local targetPos = Environment.Locked.Character[Environment.Settings.LockPart].Position
                    if Environment.Settings.Sensitivity > 0 then
                        Environment.Animation = TweenService:Create(Camera, TweenInfo.new(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, targetPos)})
                        Environment.Animation:Play()
                    else
                        Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
                    end
                    if Environment.FOVCircle then
                        Environment.FOVCircle.Color = Environment.FOVSettings.LockedColor
                    end
                end
            end
        else
            if Environment.FOVCircle then
                Environment.FOVCircle.Visible = false
            end
            CancelLock()
        end
    end)
end

-- Função para limpar conexões
local function CleanupAimbot()
    for _, connection in pairs(Environment.ServiceConnections) do
        connection:Disconnect()
    end
    Environment.ServiceConnections = {}
    if Environment.FOVCircle and Environment.FOVCircle.Remove then
        Environment.FOVCircle:Remove()
        Environment.FOVCircle = nil
    end
    CancelLock()
    print("Aimbot: Aimbot encerrado")
end

-- Criar ScreenGui (inicialmente invisível)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotGui"
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false
screenGui.Enabled = false -- Invisível até clicar no botão

-- Criar Frame
local frame = Instance.new("Frame")
frame.Name = "AimbotFrame"
frame.Size = UDim2.new(0, 220, 0, 150)
frame.Position = UDim2.new(0.5, -110, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Sistema de arraste personalizado
frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

frame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Criar TextButton para ativar/desativar aimbot
local button = Instance.new("TextButton")
button.Name = "AimbotButton"
button.Size = UDim2.new(0.9, 0, 0.2, 0)
button.Position = UDim2.new(0.05, 0, 0.1, 0)
button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.GothamBold
button.TextSize = 18
button.Text = "Ativar Aimbot 🎯"
button.Parent = frame
local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 8)
buttonCorner.Parent = button

-- Criar TextLabel para mostrar o FOV
local fovLabel = Instance.new("TextLabel")
fovLabel.Name = "FovLabel"
fovLabel.Size = UDim2.new(0.9, 0, 0.2, 0)
fovLabel.Position = UDim2.new(0.05, 0, 0.35, 0)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextSize = 16
fovLabel.Text = "FOV: 90"
fovLabel.Parent = frame

-- Criar botões + e - para ajustar FOV
local plusButton = Instance.new("TextButton")
plusButton.Name = "PlusButton"
plusButton.Size = UDim2.new(0.2, 0, 0.2, 0)
plusButton.Position = UDim2.new(0.75, 0, 0.35, 0)
plusButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
plusButton.TextColor3 = Color3.fromRGB(255, 255, 255)
plusButton.Font = Enum.Font.GothamBold
plusButton.TextSize = 18
plusButton.Text = "+"
plusButton.Parent = frame
local plusCorner = Instance.new("UICorner")
plusCorner.CornerRadius = UDim.new(0, 8)
plusCorner.Parent = plusButton

local minusButton = Instance.new("TextButton")
minusButton.Name = "MinusButton"
minusButton.Size = UDim2.new(0.2, 0, 0.2, 0)
minusButton.Position = UDim2.new(0.05, 0, 0.35, 0)
minusButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
minusButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minusButton.Font = Enum.Font.GothamBold
minusButton.TextSize = 18
minusButton.Text = "-"
minusButton.Parent = frame
local minusCorner = Instance.new("UICorner")
minusCorner.CornerRadius = UDim.new(0, 8)
minusCorner.Parent = minusButton

-- Criar toggle para TeamCheck
local teamCheckToggle = Instance.new("TextButton")
teamCheckToggle.Name = "TeamCheckToggle"
teamCheckToggle.Size = UDim2.new(0.9, 0, 0.2, 0)
teamCheckToggle.Position = UDim2.new(0.05, 0, 0.6, 0)
teamCheckToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
teamCheckToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
teamCheckToggle.Font = Enum.Font.GothamBold
teamCheckToggle.TextSize = 18
teamCheckToggle.Text = "TeamCheck: OFF"
teamCheckToggle.Parent = frame
local teamCheckCorner = Instance.new("UICorner")
teamCheckCorner.CornerRadius = UDim.new(0, 8)
teamCheckCorner.Parent = teamCheckToggle

-- Criar botão de fechar
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0.1, 0, 0.1, 0)
closeButton.Position = UDim2.new(0.85, 0, 0.05, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 14
closeButton.Text = "X"
closeButton.Parent = frame
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 4)
closeCorner.Parent = closeButton

-- Criar botão para minimizar/maximizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Name = "MinimizeButton"
minimizeButton.Size = UDim2.new(0.1, 0, 0.1, 0)
minimizeButton.Position = UDim2.new(0.95, 0, 0.05, 0)
minimizeButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.TextSize = 14
minimizeButton.Text = "-"
minimizeButton.Parent = frame
local minimizeCorner = Instance.new("UICorner")
minimizeCorner.CornerRadius = UDim.new(0, 4)
minimizeCorner.Parent = minimizeButton

-- Variáveis de controle para UI
local minimized = false
local originalSize = frame.Size
local minimizedSize = UDim2.new(0, 50, 0, 50)

-- Conectar botões da UI
button.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    button.Text = aimbotEnabled and "Desativar Aimbot ❌" or "Ativar Aimbot 🎯"
    if aimbotEnabled then
        SetupConnections()
        LoadAimbot()
    else
        CleanupAimbot()
    end
    print("Aimbot: " .. (aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!"))
end)

plusButton.MouseButton1Click:Connect(function()
    Environment.FOVSettings.Amount = math.clamp(Environment.FOVSettings.Amount + 10, 10, 500)
    fovLabel.Text = "FOV: " .. Environment.FOVSettings.Amount
    if Environment.FOVCircle then
        Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
    end
    print("Aimbot: FOV ajustado para " .. Environment.FOVSettings.Amount)
end)

minusButton.MouseButton1Click:Connect(function()
    Environment.FOVSettings.Amount = math.clamp(Environment.FOVSettings.Amount - 10, 10, 500)
    fovLabel.Text = "FOV: " .. Environment.FOVSettings.Amount
    if Environment.FOVCircle then
        Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
    end
    print("Aimbot: FOV ajustado para " .. Environment.FOVSettings.Amount)
end)

teamCheckToggle.MouseButton1Click:Connect(function()
    Environment.Settings.TeamCheck = not Environment.Settings.TeamCheck
    teamCheckToggle.Text = "TeamCheck: " .. (Environment.Settings.TeamCheck and "ON" or "OFF")
    print("Aimbot: TeamCheck: " .. (Environment.Settings.TeamCheck and "ON" or "OFF"))
end)

closeButton.MouseButton1Click:Connect(function()
    screenGui.Enabled = false
    print("Aimbot: UI fechada")
end)

minimizeButton.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        frame.Size = minimizedSize
        minimizeButton.Text = "+"
        button.Visible = false
        fovLabel.Visible = false
        plusButton.Visible = false
        minusButton.Visible = false
        teamCheckToggle.Visible = false
        closeButton.Visible = false
    else
        frame.Size = originalSize
        minimizeButton.Text = "-"
        button.Visible = true
        fovLabel.Visible = true
        plusButton.Visible = true
        minusButton.Visible = true
        teamCheckToggle.Visible = true
        closeButton.Visible = true
    end
    print("Aimbot: UI " .. (minimized and "minimizada" or "maximizada"))
end)

-- Adicionar controles na aba Aimbot
AimbotTab:AddToggle("AimbotToggle", {
    Title = "Ativar Aimbot 🔥",
    Default = false,
    Callback = function(value)
        aimbotEnabled = value
        button.Text = aimbotEnabled and "Desativar Aimbot ❌" or "Ativar Aimbot 🎯"
        if aimbotEnabled then
            SetupConnections()
            LoadAimbot()
        else
            CleanupAimbot()
        end
        print("Aimbot: " .. (aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!"))
    end
})

AimbotTab:AddKeybind("AimbotKeybind", {
    Title = "Ativar/Desativar Aimbot 🥷",
    Default = "E",
    Callback = function()
        aimbotEnabled = not aimbotEnabled
        button.Text = aimbotEnabled and "Desativar Aimbot ❌" or "Ativar Aimbot 🎯"
        if aimbotEnabled then
            SetupConnections()
            LoadAimbot()
        else
            CleanupAimbot()
        end
        print("Aimbot: " .. (aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!"))
    end
})

AimbotTab:AddButton({
    Title = "Abrir/Fechar UI Aimbot 🎯",
    Callback = function()
        screenGui.Enabled = not screenGui.Enabled
        print("Aimbot: " .. (screenGui.Enabled and "UI Aimbot aberta!" or "UI Aimbot fechada!"))
    end
})

-- Inicializar o aimbot
local success, errorMsg = pcall(function()
    InitializeFOVCircle()
    SetupConnections()
    if aimbotEnabled then LoadAimbot() end
end)
if not success then
    print("Aimbot: Erro ao inicializar aimbot: " .. tostring(errorMsg))
else
    print("Aimbot: Aimbot inicializado com sucesso!")
end


-- Tabela para armazenar os ESPs dos jogadores
local players = game:GetService("Players")
local espInstances = {}

-- Função para criar o ESP
local function createESPName(player)
    local character = player.Character or player.CharacterAdded:Wait()
    if character:FindFirstChild("ESPName") then return end 
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "ESPName"
    billboardGui.Adornee = character:WaitForChild("Head")
    billboardGui.Size = UDim2.new(27, 0, 2, 0) 
    billboardGui.StudsOffset = Vector3.new(0, 3, 0)
    billboardGui.AlwaysOnTop = true

    local nameLabel = Instance.new("TextLabel", billboardGui)
    nameLabel.Text = player.Name
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextStrokeTransparency = 0.2 
    nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0) 
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = 27 
    nameLabel.TextScaled = false

    -- Efeito arco-íris
    local function updateRainbow()
        local hue = tick() % 5 / 5
        nameLabel.TextColor3 = Color3.fromHSV(hue, 1, 1)
    end

    game:GetService("RunService").RenderStepped:Connect(updateRainbow)

    billboardGui.Parent = character
    espInstances[player] = billboardGui 
    print("ESP criado para " .. player.Name)
end

-- Função para remover o ESP
local function removeESPName(player)
    local esp = espInstances[player]
    if esp then
        esp:Destroy()
        espInstances[player] = nil 
        print("ESP removido para " .. player.Name)
    end
end

-- Função para monitorar o ESP
local monitorValue = false -- Armazenar o estado do toggle
local function monitorESP(Value)
    monitorValue = Value
    for _, player in ipairs(players:GetPlayers()) do
        if player ~= players.LocalPlayer then
            local character = player.Character
            if character then
                if Value then
                    createESPName(player)
                else
                    removeESPName(player) 
                end
            end
        end
    end

    -- Monitorar novos jogadores
    players.PlayerAdded:Connect(function(player)
        if player ~= players.LocalPlayer then
            player.CharacterAdded:Connect(function(character)
                if monitorValue then
                    createESPName(player)
                end
            end)
        end
    end)

    players.PlayerRemoving:Connect(function(player)
        removeESPName(player) 
    end)
    print("MonitorESP atualizado: " .. tostring(Value))
end


print("Janela da Fluent criada!")

local VisualTab = Window:AddTab({
    Title = "Visual 👁️‍🗨️",
    Icon = "" -- Adicione um ícone, se desejar (ex.: "rbxassetid://1234567890")
})

-- Tentar criar o toggle com AddToggle
local successToggle, toggleError = pcall(function()
    VisualTab:AddToggle({
        Title = "ESP Name",
        Default = false,
        Callback = function(Value)
            monitorESP(Value)
            print("ESP Nome " .. (Value and "ativado!" or "desativado!"))
        end
    })
end)
if successToggle then
    print("Toggle ESP Nome criado com AddToggle!")
else
    warn("Erro ao criar toggle com AddToggle: " .. tostring(toggleError))
    -- Tentar criar com CreateToggle como alternativa
    local successCreateToggle, createToggleError = pcall(function()
        VisualTab:CreateToggle({
            Title = "ESP Nome",
            Default = false,
            Callback = function(Value)
                monitorESP(Value)
                print("ESP Nome " .. (Value and "ativado!" or "desativado!"))
            end
        })
    end)
    if successCreateToggle then
        print("Toggle ESP Nome criado com CreateToggle!")
    else
        warn("Erro ao criar toggle com CreateToggle: " .. tostring(createToggleError))
    end
end

-- Adicionar um botão como alternativa para testar a interface
VisualTab:AddButton({
    Title = "ESP NAME (BETA)",
    Callback = function()
        monitorESP(true)
        print("Botão Testar ESP clicado!")
    end
})
print("Botão Testar ESP criado!")


-- Serviços
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Encapsulamento em uma tabela local
local Exploit = {
    Services = {
        HttpService = game:GetService("HttpService"),
        Players = game:GetService("Players")
    },
    LocalPlayer = game:GetService("Players").LocalPlayer
}

-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PhysicsService = game:GetService("PhysicsService")

-- Configurações
local MAX_DISTANCE = 30 -- Distância máxima em metros
local buttonCooldowns = {} -- Cooldowns individuais
local FREEZE_DURATION = 3 -- Tempo que o jogador fica congelado (em segundos)
local noClipEnabled = false -- Estado do no-clip

-- Função de assinatura
local function signature()
    print("By Berninhas")
end

-- Função auxiliar para gerenciar cooldown
local function isOnCooldown(name)
    return buttonCooldowns[name] or false
end

local function setCooldown(name, duration)
    buttonCooldowns[name] = true
    task.delay(duration, function()
        buttonCooldowns[name] = false
    end)
end

-- Função auxiliar para alterar texto do botão e interação
local function flashButton(button, text, duration)
    button:SetInteractable(false)
    button:SetText(text)
    task.wait(duration)
    button:SetText(button.Name == "BugCarButton" and "Bug Car 🛞" or button.Name == "BugCar2Button" and "Bug Car 2 🛞" or button.Name == "NoClipButton" and (noClipEnabled and "No-Clip (Ativado) 🚶" or "No-Clip (Desativado) 🚶") or "Destrancar Veiculo 🚗")
    button:SetInteractable(true)
end

-- Função auxiliar para obter personagem com timeout
local function getCharacter()
    local Character = LocalPlayer.Character
    if not Character then
        local success, result = pcall(function()
            return LocalPlayer.CharacterAdded:Wait(5)
        end)
        if success and result then
            Character = result
        else
            warn("Personagem não encontrado.")
            return nil
        end
    end
    local HRP = Character:WaitForChild("HumanoidRootPart", 5)
    if not HRP then
        warn("HumanoidRootPart não encontrado.")
        return nil
    end
    return Character, HRP
end

-- Função para encontrar o DriveSeat mais próximo
local function findClosestDriveSeat(HRP)
    local closestSeat = nil
    local closestDistance = math.huge

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("VehicleSeat") and obj.Name == "DriveSeat" then
            local vehicleModel = obj:FindFirstAncestorOfClass("Model")
            if vehicleModel and vehicleModel:FindFirstChildWhichIsA("BasePart") then
                local distance = (HRP.Position - obj.Position).Magnitude
                if distance < closestDistance and distance <= MAX_DISTANCE then
                    closestDistance = distance
                    closestSeat = obj
                end
            end
        end
    end

    return closestSeat, closestDistance
end

-- Função para gerenciar No-Clip
local function toggleNoClip()
    noClipEnabled = not noClipEnabled
    local Character, HRP = getCharacter()
    if Character then
        for _, part in ipairs(Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = not noClipEnabled
            end
        end
    end
    print(noClipEnabled and "No-Clip ativado!" or "No-Clip desativado!")
end

-- Aba Veiculos
local VehicleTab = Window:AddTab({ Title = "Veiculos", Icon = "car" })

-- Botão para Destrancar Veículo
local VehicleButton = VehicleTab:AddButton({
    Title = "Destrancar Veiculo 🚗",
    Name = "UnlockCarButton",
    Callback = function()
        if isOnCooldown("UnlockCarButton") then return end
        setCooldown("UnlockCarButton", 2)

        signature()

        local Character, HRP = getCharacter()
        if not Character or not HRP then
            return
        end

        local closestSeat = findClosestDriveSeat(HRP)
        if closestSeat then
            local success, err = pcall(function()
                closestSeat.CanCollide = true
                closestSeat.CanTouch = true
                closestSeat.CanQuery = true
                closestSeat.Disabled = false

                local vehicleModel = closestSeat:FindFirstAncestorOfClass("Model")
                if vehicleModel then
                    local lockedValue = vehicleModel:FindFirstChild("Locked")
                    if lockedValue and lockedValue:IsA("BoolValue") then
                        lockedValue.Value = false
                    end
                end
            end)
            if success then
                print("Carro destrancado com sucesso!")
                flashButton(VehicleButton, "Veículo destrancado ✅", 2)
            else
                warn("Erro ao destrancar carro: " .. tostring(err))
            end
        else
            warn("Nenhum DriveSeat encontrado nas proximidades!")
        end
    end
})

-- Botão para Bug Car (girar, lançar e teleportar com congelamento)
local BugCarButton = VehicleTab:AddButton({
    Title = "Bug Car 🛞",
    Name = "BugCarButton",
    Callback = function()
        if isOnCooldown("BugCarButton") then return end
        setCooldown("BugCarButton", 3)

        signature()
        print("Bug Car ativado!")

        local Character, HRP = getCharacter()
        if not Character or not HRP then
            return
        end

        -- Capturar posição do clique
        local clickPosition = HRP.Position + Vector3.new(0, 3, 0) -- Ajuste para posicionar acima do chão

        local closestSeat, closestDistance = findClosestDriveSeat(HRP)
        if closestSeat then
            local vehicleModel = closestSeat:FindFirstAncestorOfClass("Model")
            if not vehicleModel then
                warn("Modelo do veículo não encontrado!")
                return
            end

            if not vehicleModel.PrimaryPart then
                vehicleModel.PrimaryPart = closestSeat
            end

            local success, err = pcall(function()
                -- Adicionar BodyAngularVelocity para giro caótico
                local bodyAngularVelocity = Instance.new("BodyAngularVelocity")
                bodyAngularVelocity.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
                bodyAngularVelocity.AngularVelocity = Vector3.new(0, 0, 100)
                bodyAngularVelocity.Parent = vehicleModel.PrimaryPart

                -- Adicionar BodyVelocity para lançamento
                local bodyVelocity = Instance.new("BodyVelocity")
                bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                bodyVelocity.Velocity = Vector3.new(0, 300, 0)
                bodyVelocity.Parent = vehicleModel.PrimaryPart

                -- Remover forças após 3 segundos
                task.spawn(function()
                    task.wait(3)
                    if bodyAngularVelocity then bodyAngularVelocity:Destroy() end
                    if bodyVelocity then bodyVelocity:Destroy() end
                end)

                -- Teleportar jogador após 5 segundos e congelar por 3 segundos
                task.spawn(function()
                    task.wait(5)
                    local newCharacter, newHRP = getCharacter()
                    if newHRP then
                        local humanoid = newCharacter:FindFirstChildOfClass("Humanoid")
                        if humanoid then
                            -- Teleportar e resetar física
                            newHRP.Position = clickPosition
                            newHRP.Velocity = Vector3.new(0, 0, 0) -- Resetar velocidade
                            newHRP.RotVelocity = Vector3.new(0, 0, 0) -- Resetar rotação
                            humanoid:ChangeState(Enum.HumanoidStateType.Landed) -- Forçar estado "aterrissado"

                            print("Teleport para a posição do clique!")

                            -- Congelar jogador
                            newHRP.Anchored = true
                            humanoid.PlatformStand = true
                            task.wait(FREEZE_DURATION)
                            newHRP.Anchored = false
                            humanoid.PlatformStand = false
                            newHRP.Velocity = Vector3.new(0, 0, 0) -- Resetar novamente após descongelar
                            humanoid:ChangeState(Enum.HumanoidStateType.Landed)
                            print("Jogador descongelado!")
                        else
                            warn("Humanoid não encontrado para congelamento.")
                        end
                    else
                        warn("Não foi possível teleportar: personagem não encontrado.")
                    end
                end)
            end)
            if success then
                print("Carro bugado com sucesso! Girando, lançado e teleportando!")
                flashButton(BugCarButton, "Carro bugado! 🌀", 2)
            else
                warn("Erro ao bugar carro: " .. tostring(err))
            end
        else
            warn("Nenhum DriveSeat encontrado nas proximidades!")
        end
    end
})

-- Botão para Bug Car 2 (sumir carro, sem afetar o jogador)
local BugCar2Button = VehicleTab:AddButton({
    Title = "Bug Car 2 🛞",
    Name = "BugCar2Button",
    Callback = function()
        if isOnCooldown("BugCar2Button") then return end
        setCooldown("BugCar2Button", 2)

        signature()
        print("Bug Car 2 ativado!")

        local Character, HRP = getCharacter()
        if not Character or not HRP then
            return
        end

        local closestSeat, closestDistance = findClosestDriveSeat(HRP)
        if closestSeat then
            local vehicleModel = closestSeat:FindFirstAncestorOfClass("Model")
            if not vehicleModel then
                warn("Modelo do veículo não encontrado!")
                return
            end

            local success, err = pcall(function()
                -- Garantir que o jogador não esteja sentado no veículo
                local humanoid = Character:FindFirstChildOfClass("Humanoid")
                if humanoid and humanoid.Sit then
                    humanoid.Sit = false
                    task.wait(0.1) -- Breve espera para garantir que o jogador saiu do assento
                end

                -- Remover o veículo do workspace
                vehicleModel:Destroy()
            end)
            if success then
                print("Carro removido com sucesso!")
                flashButton(BugCar2Button, "Carro sumiu! 💨", 2)
            else
                warn("Erro ao remover carro: " .. tostring(err))
            end
        else
            warn("Nenhum DriveSeat encontrado nas proximidades!")
        end
    end
})

-- Botão para No-Clip Toggle
local NoClipButton = VehicleTab:AddButton({
    Title = "No-Clip (Desativado) 🚶",
    Name = "NoClipButton",
    Callback = function()
        if isOnCooldown("NoClipButton") then return end
        setCooldown("NoClipButton", 1)

        signature()
        toggleNoClip()
        flashButton(NoClipButton, noClipEnabled and "No-Clip Ativado! 🚶" or "No-Clip Desativado! 🚶", 1)
    end
})

-- Monitorar mudanças no personagem para manter no-clip
LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    if noClipEnabled then
        task.wait() -- Esperar o personagem carregar completamente
        for _, part in ipairs(newCharacter:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Debug: Verificar criação dos botões
print("Botão 'Destrancar Veiculo 🚗' criado na aba 'Veiculos'.")
print("Botão 'Bug Car 🛞' criado na aba 'Veiculos'.")
print("Botão 'Bug Car 2 🛞' criado na aba 'Veiculos'.")
print("Botão 'No-Clip 🚶' criado na aba 'Veiculos'.")

-- Aba de Teleportes
local TeleportTab = Window:AddTab({
    Title = "Teleports 🌀",
    Icon = "portal" -- Ícone de teleporte (ícone padrão da Fluent)
})

-- Adiciona um parágrafo inicial
TeleportTab:AddParagraph({
    Title = "GRAND Teleports",
    Content = "Sistema de teleporte rápido - Feito por Bernardo"
})

-- Função para adicionar botões de teleporte
local function addTeleportButton(name, cframe)
    TeleportTab:AddButton({
        Title = name,
        Description = "Teleportar para " .. name,
        Callback = function()
            local player = game:GetService("Players").LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                -- Teleporta com offset para evitar bugs no chão
                player.Character.HumanoidRootPart.CFrame = cframe * CFrame.new(0, 3, 0)
                Fluent:Notify({
                    Title = "Teleporte",
                    Content = "Teleportado para " .. name .. "!",
                    Duration = 2
                })
            else
                Fluent:Notify({
                    Title = "Erro",
                    Content = "Personagem não encontrado!",
                    Duration = 2
                })
            end
        end
    })
end

-- Lista de locais com CFrames
local teleportLocations = {
    {Name = "Praça", CFrame = CFrame.new(-291.579559, 3.26299787, 342.192535)},
    {Name = "Gás", CFrame = CFrame.new(-469.959015, 3.25349784, -54.3936005)},
    {Name = "HP", CFrame = CFrame.new(-543.439941, 3.26299858, 645.16864)},
    {Name = "Tabacaria", CFrame = CFrame.new(-83.1141129, 13.1430578, 74.7073364)},
    {Name = "Garagem", CFrame = CFrame.new(-466.870148, 7.64567232, 350.242737)},
    {Name = "Concessionaria", CFrame = CFrame.new(-91.3902893, 8.07136822, 520.355347)},
    {Name = "Gari", CFrame = CFrame.new(-518.672852, 3.16749811, -1.16962147, 0, 0, -1, 0, 1, 0, 1, 0, 0)},
    {Name = "Imobiliaria", CFrame = CFrame.new(-284.904785, 8.26088619, -72.2896194, 0, 0, -1, 0, 1, 0, 1, 0, 0)},
    {Name = "PM", CFrame = CFrame.new(-980.181458, 2.27553082, 467.080536, 1, 0, 0, 0, 1, 0, 0, 0, 1)},
    {Name = "PRF", CFrame = CFrame.new(6662.24512, 36.6637421, 5047.83838, 0.707134247, 0, 0.707079291, 0, 1, 0, -0.707079291, 0, 0.707134247)},
    {Name = "Mineração", CFrame = CFrame.new(201.932144, 2.76136589, 145.50531, 0, 0, 1, 0, 1, -0, -1, 0, 0)},
    {Name = "Mecânica", CFrame = CFrame.new(-180.608261, 3.29813337, -532.4151, 0.422592998, -0, -0.906319618, 0, 1, -0, 0.906319618, 0, 0.422592998)},
    {Name = "Fazenda", CFrame = CFrame.new(817.243225, 3.26249814, -87.316864, 0, 0, 1, 0, 1, 0, -1, 0, 0)},
    {Name = "Prefeitura", CFrame = CFrame.new(-284.388458, 15.1148872, 88.0397873, 0, 0, -1, 0, 1, 0, 1, 0, 0)},
    {Name = "Banco", CFrame = CFrame.new(-27.2709007, 11.5685892, 418.200653, 1, 0, 0, 0, 1, 0, 0, 0, 1)},
    {Name = "Ilegal", CFrame = CFrame.new(12045.0146, 37.3188896, 12826.2041, -0.257956624, 0.00115467981, -0.966156006, -0.0795417428, 0.99657917, 0.0224280898, 0.962876916, 0.0826351941, -0.256982446)},
    {Name = "Predio 1", CFrame = CFrame.new(-1595.23328, 204.074341, 555.895386, 0.939687431, -0.34203434, 1.81794167e-06, 1.81794167e-06, 1.02519989e-05, 1, -0.34203434, -0.93968749, 1.02519989e-05)},
    {Name = "Devs Mini City", CFrame = CFrame.new(2555.44263, 303.167755, -1004.13763, -0.422592998, 0, 0.906319618, 0, 1, 0, -0.906319618, 0, -0.422592998)}
}

-- Adiciona botões para cada local
for _, location in ipairs(teleportLocations) do
    addTeleportButton(location.Name, location.CFrame)
end

-- Seção para teleporte personalizado
TeleportTab:AddSection("Teleporte Personalizado")

-- Variáveis para salvar posição
local savedPosition = nil

-- Botão para salvar posição atual
TeleportTab:AddButton({
    Title = "Salvar Posição Atual 📍",
    Description = "Salva sua posição atual",
    Callback = function()
        local player = game:GetService("Players").LocalPlayer
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            savedPosition = player.Character.HumanoidRootPart.CFrame
            Fluent:Notify({
                Title = "Posição Salva",
                Content = "Posição atual salva com sucesso!",
                Duration = 2
            })
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Personagem não encontrado!",
                Duration = 2
            })
        end
    end
})

-- Botão para teleportar para posição salva
TeleportTab:AddButton({
    Title = "Teleportar para Posição Salva 🏠",
    Description = "Teleporta para a posição salva",
    Callback = function()
        if savedPosition then
            local player = game:GetService("Players").LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = savedPosition
                Fluent:Notify({
                    Title = "Teleporte",
                    Content = "Teleportado para a posição salva!",
                    Duration = 2
                })
            else
                Fluent:Notify({
                    Title = "Erro",
                    Content = "Personagem não encontrado!",
                    Duration = 2
                })
            end
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Nenhuma posição salva!",
                Duration = 2
            })
        end
    end
})

-- Inputs para teleporte por coordenadas
local xInput = TeleportTab:AddInput("XInput", {
    Title = "Coordenada X",
    Placeholder = "Digite X",
    Numeric = true,
    Callback = function(value)
        -- Armazena o valor de X
    end
})

local yInput = TeleportTab:AddInput("YInput", {
    Title = "Coordenada Y",
    Placeholder = "Digite Y",
    Numeric = true,
    Callback = function(value)
        -- Armazena o valor de Y
    end
})

local zInput = TeleportTab:AddInput("ZInput", {
    Title = "Coordenada Z",
    Placeholder = "Digite Z",
    Numeric = true,
    Callback = function(value)
        -- Armazena o valor de Z
    end
})

-- Botão para teleporte por coordenadas
TeleportTab:AddButton({
    Title = "Teleportar para Coordenadas 📌",
    Description = "Teleporta para as coordenadas inseridas",
    Callback = function()
        local x = tonumber(xInput.Value) or 0
        local y = tonumber(yInput.Value) or 0
        local z = tonumber(zInput.Value) or 0
        local player = game:GetService("Players").LocalPlayer
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = CFrame.new(x, y, z)
            Fluent:Notify({
                Title = "Teleporte",
                Content = "Teleportado para (" .. x .. ", " .. y .. ", " .. z .. ")!",
                Duration = 2
            })
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Personagem não encontrado!",
                Duration = 2
            })
        end
    end
})

-- Debug: Confirmar criação da aba
print("Aba 'Teleports 🌀' criada com sucesso usando Fluent!")

-- Exibe a janela
Window:SelectTab(1)

-- Aba de Teleportes
local TeleportTab = Window:AddTab({
    Title = "Teleports players 🌀",
    Icon = "portal"
})

-- Parágrafo inicial
TeleportTab:AddParagraph({
    Title = "Teleporte de Jogadores",
    Content = "Selecione um jogador para teleportar - Feito por Bernardo"
})

-- Função para teleporte suave
local function smoothTeleport(character, targetCFrame)
    local rootPart = character and character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return false, "Sem HumanoidRootPart" end

    local tween = TweenService:Create(rootPart, 
        TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
        {CFrame = targetCFrame * CFrame.new(0, 2, 0)}
    )
    tween:Play()
    tween.Completed:Wait()
    return true
end

-- Seção: Controle de Jogadores
TeleportTab:AddSection("Controle de Jogadores")

-- Função para obter lista de jogadores
local function getPlayerList()
    local playerList = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(playerList, player.Name)
        end
    end
    return playerList
end

-- Dropdown para selecionar jogador
local selectedPlayer = nil
local playerDropdown = TeleportTab:AddDropdown("PlayerDropdown", {
    Title = "Selecionar Jogador",
    Values = getPlayerList(),
    Multi = false,
    Default = nil,
    Callback = function(value)
        selectedPlayer = Players:FindFirstChild(value)
        Fluent:Notify({
            Title = "Jogador Selecionado",
            Content = "Selecionado: " .. value,
            Duration = 1.5
        })
    end
})

-- Atualiza dropdown
local function updateDropdown()
    playerDropdown:SetValues(getPlayerList())
    if selectedPlayer and not Players:FindFirstChild(selectedPlayer.Name) then
        selectedPlayer = nil
        playerDropdown:SetValue(nil)
    end
end

Players.PlayerAdded:Connect(updateDropdown)
Players.PlayerRemoving:Connect(updateDropdown)

-- Botão para teleportar
TeleportTab:AddButton({
    Title = "Teleportar Para Jogador 🚶‍♂️",
    Description = "Vai até o jogador selecionado",
    Callback = function()
        if selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local success, err = smoothTeleport(LocalPlayer.Character, selectedPlayer.Character.HumanoidRootPart.CFrame)
            if success then
                Fluent:Notify({
                    Title = "Teleporte",
                    Content = "Teleportado para " .. selectedPlayer.Name .. "!",
                    Duration = 2
                })
            else
                Fluent:Notify({
                    Title = "Erro",
                    Content = "Erro ao teleportar: " .. err,
                    Duration = 2
                })
            end
        else
            Fluent:Notify({
                Title = "Erro",
                Content = "Jogador não selecionado ou sem personagem!",
                Duration = 2
            })
        end
    end
})

-- Botão para atualizar lista
TeleportTab:AddButton({
    Title = "🔄 Atualizar Lista de Jogadores",
    Description = "Atualiza a lista de jogadores",
    Callback = function()
        updateDropdown()
        Fluent:Notify({
            Title = "Lista Atualizada",
            Content = "Lista de jogadores atualizada!",
            Duration = 1.5
        })
    end
})

-- Debug
print("Aba 'Teleports 🌀' para jogadores criada com sucesso!")

-- Exibe a janela
Window:SelectTab(1)

loadstring(game:HttpGet("https://raw.githubusercontent.com/Bernass79/KEY-SISTEM/refs/heads/main/README.md"))()
