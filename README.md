-- Carrega a biblioteca Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Criando a Janela da Interface com Discord e Key System
local Window = Rayfield:CreateWindow({
    Name = "🔥 GRAND-SHOP Menu Mini City 🔥",
    LoadingTitle = "GRAND Menu",
    LoadingSubtitle = "by Bernardo",
    ConfigurationSaving = {
        Enabled = false
    },
    Theme = "Serenity", -- Movido para o nível correto, conforme https://docs.sirius.menu/rayfield/configuration/themes
    -- Configurações do Discord
    Discord = {
        Enabled = true,
        Invite = "https://discord.gg/jKYYKd6ncE", -- Substitua por um invite válido, ex.: "discord.gg/hzmenu"
        RememberJoins = false
    },
    -- Configurações do Key System
    KeySystem = true,
    KeySettings = {
        Title = "GRAND-SHOP Menu Key System",
        Subtitle = "Insira a key para entrar",
        Note = "Você precisa de uma key válida para acessar este menu",
        SaveKey = true,
        GrabKeyFromSite = false,
        Key = "HZ1234" -- Substitua por uma chave real
    }
})



local Tab = Window:CreateTab("Infos", 9405926389)
Rayfield:Notify({
   Title = "GRAND-Menu Injected 💸",
   Content = "Obrigado por Adquirir nosso menu!",
   Duration = 36.0,
   Image = 118582375849783,
})

local Paragraph = Tab:CreateParagraph({Title = "Grand Menu ℹ️", Content = "+Menu Reeconstruido!!!! "})
local Button = Tab:CreateButton({ Name = "Ver Atualizações 📜", Callback = function() 
    print("Abrindo atualizações...")
end })



local CombatTab = Window:CreateTab("Combate 🗡️", 4483362458) -- Categoria Combat

----------------------
-- Noclip
----------------------
local noclipEnabled = false
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()

local function toggleNoclip(state)
    noclipEnabled = state
    if noclipEnabled then
        Rayfield:Notify({Title = "Noclip Ativado", Content = "Agora você pode atravessar objetos!", Duration = 2})
    else
        Rayfield:Notify({Title = "Noclip Desativado", Content = "Agora você colide normalmente.", Duration = 2})
    end
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

CombatTab:CreateToggle({
    Name = "NoClip 💥",
    CurrentValue = false,
    Callback = function(state)
        toggleNoclip(state)
    end
})

-- Speed
----------------------
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

CombatTab:CreateToggle({
    Name = "Ativar ajuste Velocidade⚡",
    CurrentValue = false,
    Callback = function(value)
        speedEnabled = value
        if speedEnabled then
            setSpeed(speedValue)
        else
            setSpeed(16)
        end
    end
})

CombatTab:CreateSlider({
    Name = "Ajustar Velocidade🌪️",
    Range = {16, 100},
    Increment = 1,
    Suffix = "Studs/s",
    CurrentValue = 16,
    Callback = function(value)
        speedValue = value
    end
})

-- Anti-Fall Damage
----------------------
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

CombatTab:CreateToggle({
    Name = "Anti-Queda🪂",
    CurrentValue = false,
    Callback = function(value)
        antiFallEnabled = value
        if antiFallEnabled then
            task.spawn(antiFallDamage)
        end
    end
})

-- ### Aba Hitbox ###


-- Variáveis para controle da hitbox
local hitboxEnabled = false
local defaultHitboxSize = Vector3.new(5, 5, 5) -- Tamanho padrão da hitbox
local hitboxSize = defaultHitboxSize
local originalSizes = {} -- Para armazenar tamanhos originais das hitboxes
local minSize = 1 -- Tamanho mínimo da hitbox
local maxSize = 20 -- Tamanho máximo da hitbox
local sizeStep = 1 -- Incremento para ajustar o tamanho

-- Função para expandir a hitbox dos jogadores
local function expandHitbox()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            if humanoidRootPart and head then
                -- Salva o tamanho original apenas na primeira vez
                if not originalSizes[player] then
                    originalSizes[player] = {
                        HumanoidRootPart = humanoidRootPart.Size,
                        Head = head.Size
                    }
                end
                -- Aplica o novo tamanho da hitbox
                if hitboxEnabled then
                    humanoidRootPart.Size = hitboxSize
                    head.Size = hitboxSize
                    humanoidRootPart.Transparency = 0.7 -- Visualização semi-transparente
                    head.Transparency = 0.7
                    humanoidRootPart.CanCollide = false -- Evita colisões indesejadas
                    head.CanCollide = false
                else
                    -- Restaura o tamanho original
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

-- Monitora novos jogadores
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if hitboxEnabled then
            task.wait(0.1) -- Pequeno atraso para garantir que o personagem esteja carregado
            expandHitbox()
        end
    end)
end)

-- Monitora a remoção de jogadores
game.Players.PlayerRemoving:Connect(function(player)
    originalSizes[player] = nil -- Remove os dados do jogador ao sair
end)

-- Atualiza a hitbox em tempo real
game:GetService("RunService").Heartbeat:Connect(function()
    if hitboxEnabled then
        expandHitbox()
    end
end)

-- Toggle para ativar/desativar a hitbox
CombatTab:CreateToggle({
    Name = "Ativar Hitbox 📦",
    CurrentValue = false,
    Flag = "HitboxToggle",
    Callback = function(Value)
        hitboxEnabled = Value
        if not hitboxEnabled then
            -- Restaura as hitboxes ao desativar
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

-- UI flutuante para exibir o tamanho (criada para suportar o slider)
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

-- Adiciona controles de ajuste via slider
CombatTab:CreateSlider({
    Name = "Tamanho da Hitbox 🎯",
    Range = {minSize, maxSize},
    Increment = sizeStep,
    Suffix = " studs",
    CurrentValue = hitboxSize.X,
    Flag = "HitboxSizeSlider",
    Callback = function(Value)
        hitboxSize = Vector3.new(Value, Value, Value)
        sizeLabel.Text = "Tamanho: " .. hitboxSize.X
        if hitboxEnabled then
            expandHitbox()
        end
    end
})



local Tab = Window:CreateTab("AUTO-FARMS🌱", 484395794)
local Button = Tab:CreateButton({ Name = "Injetar Farms NESSESARIO 🔧", Callback = function() 
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

end })
local Button = Tab:CreateButton({
    Name = "Auto-Colect Gari🚛",
    Callback = function()
        print("Auto-Farm Gari ativado!")
        
        -- Função para encontrar o LEXOS mais próximo com colisão ativada
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
        
        -- Função para teleportar para dentro do LEXOS mais próximo com colisão ativada
        local function teleportToNearestLEXOS()
            local nearestLEXOS = findNearestLEXOS()
            if nearestLEXOS then
                local player = game:GetService("Players").LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
                
                -- Calcular o centro do LEXOS
                local centerPosition = nearestLEXOS.Position
                
                -- Teleportar para o centro do LEXOS
                humanoidRootPart.CFrame = CFrame.new(centerPosition)
            end
        end
        
        -- Modificar todos os ProximityPrompts para HoldDuration = 0
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
        
        -- Loop para teleportar e acionar o ProximityPrompt
        while true do
            teleportToNearestLEXOS()
            wait(0.3) -- Intervalo entre teleportes
            
            -- Acionar o ProximityPrompt mais próximo no LEXOS
            local nearestLEXOS = findNearestLEXOS()
            if nearestLEXOS then
                for _, child in ipairs(nearestLEXOS:GetChildren()) do
                    if child:IsA("ProximityPrompt") then
                        fireproximityprompt(child) -- Aciona o prompt diretamente
                        break
                    end
                end
            end
            wait(0.3) -- Intervalo após acionar o prompt
        end
    end
})
local Button = Tab:CreateButton({ Name = "Auto-Colect Gás⛽", Callback = function() 
    print("Auto-Farm Gás ativado!")
loadstring(game:HttpGet('https://raw.githubusercontent.com/Rafaasxs/Nexus-Menu-/refs/heads/main/tesao'))()
end })

local Button = Tab:CreateButton({
    Name = "Auto-Colect Minerador⛏️ ️",
    Callback = function()
        print("Auto-Farm Rock ativado!")
        
        -- Função para encontrar o Rock mais próximo com colisão ativada
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
        
        -- Função para teleportar para dentro do Rock mais próximo com colisão ativada
        local function teleportToNearestRock()
            local nearestRock = findNearestRock()
            if nearestRock then
                local player = game:GetService("Players").LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
                
                -- Calcular o centro do Rock
                local centerPosition = nearestRock.Position
                
                -- Teleportar para o centro do Rock
                humanoidRootPart.CFrame = CFrame.new(centerPosition)
            end
        end
        
        -- Modificar todos os ProximityPrompts para HoldDuration = 0
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
        
        -- Loop para teleportar e acionar o ProximityPrompt (autoclick no E)
        while true do
            teleportToNearestRock()
            wait(0.3) -- Intervalo entre teleportes
            
            -- Acionar o ProximityPrompt mais próximo no Rock
            local nearestRock = findNearestRock()
            if nearestRock then
                for _, child in ipairs(nearestRock:GetChildren()) do
                    if child:IsA("ProximityPrompt") then
                        fireproximityprompt(child) -- Aciona o prompt diretamente
                        break
                    end
                end
            end
            wait(0.3) -- Intervalo após acionar o prompt
        end
    end
})
Tab:CreateSection("Auto farm peça")

-- Botão para salvar posição
local Button = Tab:CreateButton({
    Name = "Salvar farm Posição da fac",
    Callback = function()
        local player = game.Players.LocalPlayer
        local character = player.Character
        if character then
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
            if humanoidRootPart then
                _G.SavedPosition = humanoidRootPart.Position
                Rayfield:Notify({Title = "Posição Salva", Content = "Sua posição foi salva com sucesso!", Duration = 2})
            end
        end
    end
})

-- Função para acionar os ProximityPrompts
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

-- Função para encontrar o prompt do jogador
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

-- Toggle para iniciar o farm
local Button = Tab:CreateToggle({
    Name = "Ativar Farm",
    CurrentValue = false,
    Callback = function(Value)
        while Value and _G.SavedPosition do
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


local Button = Tab:CreateButton({
    Name = "Auto Click",
    Callback = function()
        print("Auto Click ativado!")
        
        -- Modificar todos os ProximityPrompts para HoldDuration = 0
        for _, prompt in ipairs(workspace:GetDescendants()) do
            if prompt:IsA("ProximityPrompt") then
                prompt.HoldDuration = 0
            end
        end
        
        -- Monitorar novos ProximityPrompts adicionados
        workspace.DescendantAdded:Connect(function(descendant)
            if descendant:IsA("ProximityPrompt") then
                descendant.HoldDuration = 0
            end
        end)

        -- Função para encontrar o ProximityPrompt mais próximo
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

        -- Loop para auto click nos ProximityPrompts
        while true do
            local nearestPrompt = findNearestPrompt()
            if nearestPrompt then
                fireproximityprompt(nearestPrompt) -- Ativa o prompt automaticamente
                print("Auto Click ativou um ProximityPrompt!")
            end
            wait(0.1) -- Intervalo pequeno para evitar sobrecarga (ajuste conforme necessário)
        end
    end
})





local Tab = Window:CreateTab("AIMBOTS", 15990136399)




-- Toggle para ativar/desativar o aimbot
local aimbotEnabled = false
local Toggle = Tab:CreateToggle({
    Name = "Aimbot-Legit 🔫",
    CurrentValue = false,
    Flag = "AimbotToggle",
    Callback = function(Value)
        aimbotEnabled = Value
        print(aimbotEnabled and "Aimbot Legit ativado!" or "Aimbot Legit desativado!")
        if not aimbotEnabled then
            CancelLock()
        end
    end
	
})
print("Toggle Aimbot criado!")

--// Cache
local Vector2 = Vector2
local mathclamp = math.clamp

--// Ambiente isolado
local AimbotEnvironment = {
    Settings = {
        Enabled = true,
        AliveCheck = true, -- Evita mirar em mortos
        WallCheck = false, -- Desativado para performance
        Sensitivity = 0.05, -- Travamento muito rápido, mas com suavização mínima
        ThirdPerson = false, -- Desativado
        TriggerKey = "MouseButton2", -- Ativa com botão direito
        LockPart = "Head" -- Mira na cabeça
    },
    FOVSettings = {
        Enabled = true,
        Visible = false, -- Círculo invisível para discrição
        Amount = 150, -- FOV amplo
        Color = Color3.fromRGB(255, 255, 255),
        LockedColor = Color3.fromRGB(255, 70, 70),
        Transparency = 0.5,
        Sides = 60,
        Thickness = 2,
        Filled = false
    },
    FOVCircle = Drawing.new("Circle"),
    Locked = nil,
    Functions = {}
}

--// Services
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// Variables
local RequiredDistance, Typing, Running, Animation, ServiceConnections = 2000, false, false, nil, {}

--// Functions
local function CancelLock()
    AimbotEnvironment.Locked = nil
    if Animation then Animation:Cancel() end
    AimbotEnvironment.FOVCircle.Color = AimbotEnvironment.FOVSettings.Color
    print("Lock cancelado")
end

local function GetClosestPlayer()
    if not AimbotEnvironment.Locked then
        RequiredDistance = AimbotEnvironment.FOVSettings.Amount

        for _, v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild(AimbotEnvironment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                if AimbotEnvironment.Settings.AliveCheck and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                if AimbotEnvironment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[AimbotEnvironment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end

                local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[AimbotEnvironment.Settings.LockPart].Position)
                local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                if Distance < RequiredDistance and OnScreen then
                    RequiredDistance = Distance
                    AimbotEnvironment.Locked = v
                    print("Jogador travado: " .. v.Name)
                end
            end
        end
    elseif AimbotEnvironment.Locked and AimbotEnvironment.Locked.Character and AimbotEnvironment.Locked.Character:FindFirstChild(AimbotEnvironment.Settings.LockPart) then
        local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Camera:WorldToViewportPoint(AimbotEnvironment.Locked.Character[AimbotEnvironment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(AimbotEnvironment.Locked.Character[AimbotEnvironment.Settings.LockPart].Position).Y)).Magnitude
        if Distance > RequiredDistance then
            CancelLock()
        end
    else
        CancelLock()
    end
end

--// Typing Check
ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
    Typing = true
    print("Digitando: Aimbot pausado")
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
    Typing = false
    print("Digitando finalizado: Aimbot retomado")
end)

--// Main
local function Load()
    ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if aimbotEnabled and AimbotEnvironment.Settings.Enabled and not Typing then
            -- Atualizar círculo de FOV
            AimbotEnvironment.FOVCircle.Radius = AimbotEnvironment.FOVSettings.Amount
            AimbotEnvironment.FOVCircle.Thickness = AimbotEnvironment.FOVSettings.Thickness
            AimbotEnvironment.FOVCircle.Filled = AimbotEnvironment.FOVSettings.Filled
            AimbotEnvironment.FOVCircle.NumSides = AimbotEnvironment.FOVSettings.Sides
            AimbotEnvironment.FOVCircle.Color = AimbotEnvironment.FOVSettings.Color
            AimbotEnvironment.FOVCircle.Transparency = AimbotEnvironment.FOVSettings.Transparency
            AimbotEnvironment.FOVCircle.Visible = AimbotEnvironment.FOVSettings.Visible
            AimbotEnvironment.FOVCircle.Position = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)

            if Running then
                GetClosestPlayer()
                if AimbotEnvironment.Locked then
                    local targetPosition = AimbotEnvironment.Locked.Character[AimbotEnvironment.Settings.LockPart].Position
                    if AimbotEnvironment.Settings.Sensitivity > 0 then
                        Animation = TweenService:Create(Camera, TweenInfo.new(AimbotEnvironment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)})
                        Animation:Play()
                    else
                        Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)
                    end
                    AimbotEnvironment.FOVCircle.Color = AimbotEnvironment.FOVSettings.LockedColor
                end
            end
        else
            AimbotEnvironment.FOVCircle.Visible = false
            CancelLock()
        end
    end)

    ServiceConnections.InputBeganConnection = UserInputService.InputBegan:Connect(function(Input)
        if not Typing and aimbotEnabled then
            if Input.UserInputType == Enum.UserInputType[AimbotEnvironment.Settings.TriggerKey] then
                Running = true
                print("Aimbot ativado (segurando M2)")
            end
        end
    end)

    ServiceConnections.InputEndedConnection = UserInputService.InputEnded:Connect(function(Input)
        if not Typing and aimbotEnabled then
            if Input.UserInputType == Enum.UserInputType[AimbotEnvironment.Settings.TriggerKey] then
                Running = false
                CancelLock()
                print("Aimbot desativado (M2 solto)")
            end
        end
    end)
end

--// Functions
function AimbotEnvironment.Functions:Exit()
    for _, v in pairs(ServiceConnections) do
        v:Disconnect()
    end
    if AimbotEnvironment.FOVCircle.Remove then AimbotEnvironment.FOVCircle:Remove() end
    AimbotEnvironment = nil
    print("Aimbot encerrado")
end

--// Load com proteção contra erros
local success, errorMsg = pcall(Load)
if not success then
    print("Erro ao inicializar aimbot: " .. tostring(errorMsg))
else
    print("Aimbot inicializado com sucesso!")
end


-- Variáveis de controle
local aimbotEnabled = false
local isMouseButton2Down = false

-- Serviços
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Ambiente isolado
local Environment = {}
local RequiredDistance, Typing, Animation, ServiceConnections = 2000, false, nil, {}

-- Configurações
Environment.Settings = {
    Enabled = true,
    AliveCheck = true,
    WallCheck = false,
    Sensitivity = 0, -- Travamento instantâneo
    ThirdPerson = false,
    ThirdPersonSensitivity = 3,
    LockPart = "Head" -- Mira na cabeça
}

Environment.FOVSettings = {
    Enabled = true,
    Visible = true, -- Círculo visível
    Amount = 90,
    Color = Color3.fromRGB(0, 0, 0),
    LockedColor = Color3.fromRGB(255, 70, 70),
    Transparency = 0.5,
    Sides = 60,
    Thickness = 1,
    Filled = false
}

Environment.FOVCircle = Drawing.new("Circle")

-- Funções do aimbot
local function CancelLock()
    Environment.Locked = nil
    if Animation then Animation:Cancel() end
    Environment.FOVCircle.Color = Environment.FOVSettings.Color
    print("Lock cancelado")
end

local function GetClosestNPC()
    if not Environment.Locked then
        RequiredDistance = (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000)

        for _, model in pairs(Workspace:GetDescendants()) do
            if model:IsA("Model") and model ~= LocalPlayer.Character and model:FindFirstChildOfClass("Humanoid") and not Players:GetPlayerFromCharacter(model) then
                if model:FindFirstChild(Environment.Settings.LockPart) then
                    if Environment.Settings.AliveCheck and model:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                    if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({model[Environment.Settings.LockPart].Position}, model:GetDescendants())) > 0 then continue end

                    local Vector, OnScreen = Camera:WorldToViewportPoint(model[Environment.Settings.LockPart].Position)
                    local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                    if Distance < RequiredDistance and OnScreen then
                        RequiredDistance = Distance
                        Environment.Locked = model
                        print("NPC travado: " .. model.Name)
                    end
                end
            end
        end
    elseif Environment.Locked and Environment.Locked:FindFirstChild(Environment.Settings.LockPart) then
        local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Camera:WorldToViewportPoint(Environment.Locked[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked[Environment.Settings.LockPart].Position).Y)).Magnitude
        if Distance > RequiredDistance then
            CancelLock()
        end
    else
        CancelLock()
    end
end

-- Conexões de eventos
ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
    Typing = true
    print("Digitando: Aimbot pausado")
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
    Typing = false
    print("Digitando finalizado: Aimbot retomado")
end)

ServiceConnections.MouseButton2Down = UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        isMouseButton2Down = true
        print("M2 pressionado")
    end
end)

ServiceConnections.MouseButton2Up = UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        isMouseButton2Down = false
        CancelLock()
        print("M2 solto")
    end
end)

-- Loop principal
local function LoadAimbot()
    ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if aimbotEnabled and Environment.Settings.Enabled and not Typing then
            if Environment.FOVSettings.Enabled then
                Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
                Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
                Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
                Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
                Environment.FOVCircle.Color = Environment.FOVSettings.Color
                Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
                Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
                Environment.FOVCircle.Position = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
            else
                Environment.FOVCircle.Visible = false
            end

            if isMouseButton2Down then
                GetClosestNPC()
                if Environment.Locked then
                    if Environment.Settings.ThirdPerson then
                        Environment.Settings.ThirdPersonSensitivity = math.clamp(Environment.Settings.ThirdPersonSensitivity, 0.1, 5)
                        local Vector = Camera:WorldToViewportPoint(Environment.Locked[Environment.Settings.LockPart].Position)
                        mousemoverel((Vector.X - UserInputService:GetMouseLocation().X) * Environment.Settings.ThirdPersonSensitivity, (Vector.Y - UserInputService:GetMouseLocation().Y) * Environment.Settings.ThirdPersonSensitivity)
                    else
                        if Environment.Settings.Sensitivity > 0 then
                            Animation = TweenService:Create(Camera, TweenInfo.new(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked[Environment.Settings.LockPart].Position)})
                            Animation:Play()
                        else
                            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked[Environment.Settings.LockPart].Position)
                        end
                    end
                    Environment.FOVCircle.Color = Environment.FOVSettings.LockedColor
                end
            end
        else
            Environment.FOVCircle.Visible = false
            CancelLock()
        end
    end)
end

-- Toggle no Rayfield
local Toggle = Tab:CreateToggle({
    Name = "Ativar Aimbot (NPCs)",
    CurrentValue = false,
    Flag = "AtivarAimbot",
    Callback = function(Value)
        aimbotEnabled = Value
        print(aimbotEnabled and "Aimbot ativado (NPCs)!" or "Aimbot desativado!")
        if not Value then
            CancelLock()
        end
    end
})

-- Iniciar o aimbot
LoadAimbot()



-- Criar ScreenGui (inicialmente desativada)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotGui"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false
screenGui.Enabled = false

-- Criar Frame arrastável
local frame = Instance.new("Frame")
frame.Name = "AimbotFrame"
frame.Size = UDim2.new(0, 220, 0, 150)
frame.Position = UDim2.new(0.5, -110, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

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

-- Criar botão para minimizar/maximizar UI
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

-- Variáveis de controle
local aimbotEnabled = false
local minimized = false
local originalSize = frame.Size
local minimizedSize = UDim2.new(0, 50, 0, 50)
local isMouseButton2Down = false

-- Serviços
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Ambiente isolado
local Environment = {}
local RequiredDistance, Typing, Animation, ServiceConnections = 2000, false, nil, {}

-- Configurações
Environment.Settings = {
    Enabled = true,
    TeamCheck = false,
    AliveCheck = true,
    WallCheck = false,
    Sensitivity = 0,
    ThirdPerson = false,
    ThirdPersonSensitivity = 3,
    LockPart = "Head"
}

Environment.FOVSettings = {
    Enabled = true,
    Visible = true,
    Amount = 90,
    Color = Color3.fromRGB(0, 0, 0),
    LockedColor = Color3.fromRGB(255, 70, 70),
    Transparency = 0.5,
    Sides = 60,
    Thickness = 1,
    Filled = false
}

Environment.FOVCircle = Drawing.new("Circle")

-- Funções do aimbot
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
                    if Environment.Settings.AliveCheck and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                    if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end
                    if Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end

                    local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                    local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

                    if Distance < RequiredDistance and OnScreen then
                        RequiredDistance = Distance
                        Environment.Locked = v
                    end
                end
            end
        end
    elseif (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).Y)).Magnitude > RequiredDistance then
        CancelLock()
    end
end

-- Conexões de eventos
ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
    Typing = true
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
    Typing = false
end)

ServiceConnections.MouseButton2Down = UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        isMouseButton2Down = true
    end
end)

ServiceConnections.MouseButton2Up = UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        isMouseButton2Down = false
        CancelLock()
    end
end)

-- Loop principal
local function LoadAimbot()
    ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if aimbotEnabled and Environment.Settings.Enabled then
            if Environment.FOVSettings.Enabled then
                Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
                Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
                Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
                Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
                Environment.FOVCircle.Color = Environment.FOVSettings.Color
                Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
                Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
                Environment.FOVCircle.Position = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
            else
                Environment.FOVCircle.Visible = false
            end

            if isMouseButton2Down then
                GetClosestPlayer()
                if Environment.Locked then
                    if Environment.Settings.ThirdPerson then
                        Environment.Settings.ThirdPersonSensitivity = math.clamp(Environment.Settings.ThirdPersonSensitivity, 0.1, 5)
                        local Vector = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)
                        mousemoverel((Vector.X - UserInputService:GetMouseLocation().X) * Environment.Settings.ThirdPersonSensitivity, (Vector.Y - UserInputService:GetMouseLocation().Y) * Environment.Settings.ThirdPersonSensitivity)
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
        else
            Environment.FOVCircle.Visible = false
        end
    end)
end


-- Toggle no Rayfield
local Toggle = Tab:CreateToggle({
    Name = "Ativar Aimbot🔥",
    CurrentValue = false,
    Flag = "AtivarAimbot",
    Callback = function(Value)
        aimbotEnabled = Value
        button.Text = aimbotEnabled and "Desativar Aimbot 🎯" or "Ativar Aimbot 🎯"
        if not Value then CancelLock() end
        print(aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!")
    end
})

-- Keybind no Rayfield
local Keybind = Tab:CreateKeybind({
    Name = "Ativar/Desativar Aimbot 🥷",
    CurrentKeybind = "E",
    Flag = "AimbotKeybind",
    Callback = function(Key)
        aimbotEnabled = not aimbotEnabled
        Toggle:Set(aimbotEnabled)
        button.Text = aimbotEnabled and "Desativar Aimbot 🎯" or "Ativar Aimbot 🎯"
        if not aimbotEnabled then CancelLock() end
        print(aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!")
    end
})

-- Botão para abrir a UI flutuante
local Button = Tab:CreateButton({
    Name = "Abrir UI Aimbot 🎯",
    Callback = function()
        screenGui.Enabled = not screenGui.Enabled
        print(screenGui.Enabled and "UI Aimbot aberta!" or "UI Aimbot fechada!")
    end
})

-- Conectar botões da UI flutuante
button.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    Toggle:Set(aimbotEnabled)
    button.Text = aimbotEnabled and "Desativar Aimbot 🎯" or "Ativar Aimbot 🎯"
    if not aimbotEnabled then CancelLock() end
    print(aimbotEnabled and "Aimbot ativado!" or "Aimbot desativado!")
end)

plusButton.MouseButton1Click:Connect(function()
    Environment.FOVSettings.Amount = math.clamp(Environment.FOVSettings.Amount + 10, 10, 500)
    fovLabel.Text = "FOV: " .. Environment.FOVSettings.Amount
end)

minusButton.MouseButton1Click:Connect(function()
    Environment.FOVSettings.Amount = math.clamp(Environment.FOVSettings.Amount - 10, 10, 500)
    fovLabel.Text = "FOV: " .. Environment.FOVSettings.Amount
end)

teamCheckToggle.MouseButton1Click:Connect(function()
    Environment.Settings.TeamCheck = not Environment.Settings.TeamCheck
    teamCheckToggle.Text = "TeamCheck: " .. (Environment.Settings.TeamCheck and "ON" or "OFF")
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
    else
        frame.Size = originalSize
        minimizeButton.Text = "-"
        button.Visible = true
        fovLabel.Visible = true
        plusButton.Visible = true
        minusButton.Visible = true
        teamCheckToggle.Visible = true
    end
end)

-- Iniciar o aimbot
LoadAimbot()


















-- Aba para ESP
local ESPTab = Window:CreateTab("Visual 👁️‍🗨️", 4483362458)

local function createInventoryESP(player)
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return
    end

    local existingGui = character:FindFirstChild("InventoryESP")
    if existingGui then existingGui:Destroy() end

    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "InventoryESP"
    billboardGui.Parent = character
    billboardGui.Size = UDim2.new(0, 150, 0, 50)
    billboardGui.StudsOffset = Vector3.new(0, -2, 0)
    billboardGui.Adornee = character.HumanoidRootPart
    billboardGui.AlwaysOnTop = true
    billboardGui.MaxDistance = 500
    if not billboardGui.Adornee then
        warn("Adornee não encontrado para " .. player.Name)
        return
    end

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Parent = billboardGui
    nameLabel.Size = UDim2.new(1, 0, 0.3, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.new(1, 1, 0)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = 10
    nameLabel.Text = player.Name
    nameLabel.TextWrapped = true

    local textLabel = Instance.new("TextLabel")
    textLabel.Parent = billboardGui
    textLabel.Size = UDim2.new(1, 0, 0.7, 0)
    textLabel.Position = UDim2.new(0, 0, 0.3, 0)
    textLabel.BackgroundTransparency = 0.5
    textLabel.BackgroundColor3 = Color3.new(0, 0, 0)
    textLabel.TextColor3 = Color3.new(1, 1, 1)
    textLabel.TextStrokeTransparency = 0
    textLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 12
    textLabel.TextWrapped = true

    local function updateInventory()
        local backpack = player:FindFirstChild("Backpack")
        local items = {}
        if backpack then
            for _, obj in ipairs(backpack:GetChildren()) do
                if obj:IsA("Tool") or obj:IsA("HopperBin") then
                    table.insert(items, obj.Name)
                end
            end
        else
            warn("Backpack de " .. player.Name .. " não acessível")
        end

        if character then
            for _, obj in ipairs(character:GetChildren()) do
                if obj:IsA("Tool") or obj:IsA("HopperBin") then
                    table.insert(items, obj.Name)
                end
            end
        end

        if #items == 0 then
            textLabel.Text = "Inventário vazio"
        else
            local itemList = "Itens:\n"
            for i, item in ipairs(items) do
                itemList = itemList .. i .. ". " .. item .. "\n"
            end
            textLabel.Text = itemList
        end
    end

    updateInventory()
    spawn(function()
        while wait(1) do
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                updateInventory()
            else
                billboardGui:Destroy()
                break
            end
        end
    end)
end

local isESPActive = false
local playerAddedConnection
ESPTab:CreateToggle({
    Name = "Esp-Inventário 🎒",
    CurrentValue = false,
    Flag = "InventoryESP",
    Callback = function(value)
        isESPActive = value
        if isESPActive then
            local success = pcall(function()
                for _, player in ipairs(game.Players:GetPlayers()) do
                    if player ~= game.Players.LocalPlayer then
                        createInventoryESP(player)
                    end
                end
                if playerAddedConnection then
                    playerAddedConnection:Disconnect()
                end
                playerAddedConnection = game.Players.PlayerAdded:Connect(function(player)
                    if isESPActive and player ~= game.Players.LocalPlayer then
                        local success, err = pcall(function()
                            createInventoryESP(player)
                        end)
                        if not success then
                            warn("Erro ao criar ESP para novo jogador " .. player.Name .. ": " .. err)
                        end
                    end
                end)
            end)
            if success then
                Rayfield:Notify({
                    Title = "ESP Ativado",
                    Content = "ESP ativado para outros jogadores.",
                    Duration = 3,
                    Image = 4483362458
                })
            else
                Rayfield:Notify({
                    Title = "Erro",
                    Content = "Falha ao ativar o ESP.",
                    Duration = 5,
                    Image = 4483362458
                })
                warn("Erro na ativação do ESP.")
            end
        else
            if playerAddedConnection then
                playerAddedConnection:Disconnect()
                playerAddedConnection = nil
            end
            local success = pcall(function()
                for _, player in ipairs(game.Players:GetPlayers()) do
                    local esp = player.Character and player.Character:FindFirstChild("InventoryESP")
                    if esp then esp:Destroy() end
                end
            end)
            if success then
                Rayfield:Notify({
                    Title = "ESP Desativado",
                    Content = "ESP de inventário desativado!",
                    Duration = 3,
                    Image = 4483362458
                })
            else
                Rayfield:Notify({
                    Title = "Erro",
                    Content = "Falha ao desativar o ESP.",
                    Duration = 5,
                    Image = 4483362458
                })
                warn("Erro na desativação do ESP.")
            end
        end
    end
})

-- ESP
local ESP = {
    Enabled = false,
    Settings = {
        TeamCheck = false,
        AliveCheck = true,
        BorderColor = Color3.fromRGB(255, 255, 255),
        SkeletonColor = Color3.fromRGB(255, 0, 0),
        BorderTransparency = 0.5,
        SkeletonTransparency = 0.5,
        BorderThickness = 3,
        SkeletonThickness = 1
    },
    Borders = {},
    Skeletons = {},
    Connections = {}
}

local function CreateEsp(player)
    if player == game.Players.LocalPlayer then return end
    local success, result = pcall(function()
        local border = Drawing.new("Square")
        border.Visible = false
        border.Color = ESP.Settings.BorderColor
        border.Thickness = ESP.Settings.BorderThickness
        border.Transparency = ESP.Settings.BorderTransparency
        border.Filled = false
        ESP.Borders[player] = border

        local skeleton = {
            headToTorso = Drawing.new("Line"),
            torsoToLeftArm = Drawing.new("Line"),
            torsoToRightArm = Drawing.new("Line"),
            torsoToLeftLeg = Drawing.new("Line"),
            torsoToRightLeg = Drawing.new("Line")
        }

        for _, line in pairs(skeleton) do
            line.Visible = false
            line.Color = ESP.Settings.SkeletonColor
            line.Thickness = ESP.Settings.SkeletonThickness
            line.Transparency = ESP.Settings.SkeletonTransparency
        end

        ESP.Skeletons[player] = skeleton
        print("[DEBUG] ESP criado para: " .. player.Name)
    end)
    if not success then warn("[ERRO] Falha ao criar ESP para " .. player.Name .. ": " .. result) end
end

local function UpdateEsp()
    local success, result = pcall(function()
        for player, border in pairs(ESP.Borders) do
            local skeleton = ESP.Skeletons[player]
            if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") or not player.Character:FindFirstChildOfClass("Humanoid") then
                border.Visible = false
                for _, line in pairs(skeleton) do line.Visible = false end
                continue
            end

            if ESP.Settings.AliveCheck and player.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then
                border.Visible = false
                for _, line in pairs(skeleton) do line.Visible = false end
                continue
            end

            if ESP.Settings.TeamCheck and player.Team == game.Players.LocalPlayer.Team then
                border.Visible = false
                for _, line in pairs(skeleton) do line.Visible = false end
                continue
            end

            local rootPart = player.Character.HumanoidRootPart
            local head = player.Character:FindFirstChild("Head")
            if not rootPart or not head then
                border.Visible = false
                for _, line in pairs(skeleton) do line.Visible = false end
                continue
            end

            local headPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
            local rootPos = workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position - Vector3.new(0, 3, 0))

            if onScreen then
                local boxSize = Vector2.new((headPos.Y - rootPos.Y) * 1.5, (headPos.Y - rootPos.Y) * 1.5)
                border.Size = Vector2.new(boxSize.X + 4, boxSize.Y + 4)
                border.Position = Vector2.new(headPos.X - (boxSize.X + 4) / 2, headPos.Y - (boxSize.Y + 4) / 1.5)
                border.Visible = ESP.Enabled

                local torsoPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("UpperTorso") and player.Character.UpperTorso.Position or rootPart.Position)
                local leftArmPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("LeftHand") and player.Character.LeftHand.Position or (rootPart.Position - Vector3.new(1, 0, 0)))
                local rightArmPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("RightHand") and player.Character.RightHand.Position or (rootPart.Position + Vector3.new(1, 0, 0)))
                local leftLegPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("LeftFoot") and player.Character.LeftFoot.Position or (rootPart.Position - Vector3.new(0.5, 2, 0)))
                local rightLegPos = workspace.CurrentCamera:WorldToViewportPoint(player.Character:FindFirstChild("RightFoot") and player.Character.RightFoot.Position or (rootPart.Position + Vector3.new(0.5, 2, 0)))

                skeleton.headToTorso.From = Vector2.new(headPos.X, headPos.Y)
                skeleton.headToTorso.To = Vector2.new(torsoPos.X, torsoPos.Y)
                skeleton.torsoToLeftArm.From = Vector2.new(torsoPos.X, torsoPos.Y)
                skeleton.torsoToLeftArm.To = Vector2.new(leftArmPos.X, leftArmPos.Y)
                skeleton.torsoToRightArm.From = Vector2.new(torsoPos.X, torsoPos.Y)
                skeleton.torsoToRightArm.To = Vector2.new(rightArmPos.X, rightArmPos.Y)
                skeleton.torsoToLeftLeg.From = Vector2.new(torsoPos.X, torsoPos.Y)
                skeleton.torsoToLeftLeg.To = Vector2.new(leftLegPos.X, leftLegPos.Y)
                skeleton.torsoToRightLeg.From = Vector2.new(torsoPos.X, torsoPos.Y)
                skeleton.torsoToRightLeg.To = Vector2.new(rightLegPos.X, rightLegPos.Y)

                for _, line in pairs(skeleton) do line.Visible = ESP.Enabled end
            else
                border.Visible = false
                for _, line in pairs(skeleton) do line.Visible = false end
            end
        end
    end)
    if not success then warn("[ERRO] Falha ao atualizar ESP: " .. result) end
end

for _, player in pairs(game.Players:GetPlayers()) do CreateEsp(player) end

ESP.Connections.PlayerAdded = game.Players.PlayerAdded:Connect(CreateEsp)
ESP.Connections.PlayerRemoving = game.Players.PlayerRemoving:Connect(function(player)
    if ESP.Borders[player] then
        ESP.Borders[player]:Remove()
        ESP.Borders[player] = nil
        print("[DEBUG] Borda ESP removida para: " .. player.Name)
    end
    if ESP.Skeletons[player] then
        for _, line in pairs(ESP.Skeletons[player]) do line:Remove() end
        ESP.Skeletons[player] = nil
        print("[DEBUG] Esqueleto ESP removido para: " .. player.Name)
    end
end)
ESP.Connections.RenderStepped = game:GetService("RunService").RenderStepped:Connect(UpdateEsp)

ESPTab:CreateToggle({
    Name = "Esp-Players🔍",
    CurrentValue = false,
    Flag = "AtivarESP",
    Callback = function(Value)
        ESP.Enabled = Value
        print("[DEBUG] ESP " .. (ESP.Enabled and "ativado" or "desativado"))
    end
})



-- Tabela para armazenar os dados do ESP
local espData = {}

-- Variáveis para controlar o ciclo RGB e a cor estática
local rgbCycleEnabled = false
local currentStaticColor = Color3.fromRGB(0, 255, 0) -- Cor inicial verde

-- Função para verificar se o jogador é da team STAFF
local function isStaff(player)
    return player and player.Team and player.Team.Name == "STAFF"
end

-- Função para criar ESP para Staff (sem linhas)
local function createESPStaff(targetPlayer)
    if not isStaff(targetPlayer) or not targetPlayer.Character then
        return
    end

    local targetCharacter = targetPlayer.Character
    local targetRootPart = targetCharacter:WaitForChild("HumanoidRootPart", 5)
    local targetHumanoid = targetCharacter:WaitForChild("Humanoid", 5)
    if not targetRootPart or not targetHumanoid then
        return
    end

    -- Cria o Highlight
    local highlight = Instance.new("Highlight")
    highlight.Adornee = targetCharacter
    highlight.FillColor = currentStaticColor
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Enabled = false
    highlight.Parent = targetCharacter

    -- Cria o texto (BillboardGui)
    local billboard = Instance.new("BillboardGui")
    billboard.Adornee = targetRootPart
    billboard.Size = UDim2.new(0, 100, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.TextStrokeTransparency = 0
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextScaled = true
    textLabel.Parent = billboard
    billboard.Enabled = false
    billboard.Parent = targetCharacter

    return {highlight = highlight, billboard = billboard, textLabel = textLabel}
end

-- Função para criar ESP para Jogadores (com linhas)
local function createESPPlayer(targetPlayer)
    if targetPlayer == game.Players.LocalPlayer or isStaff(targetPlayer) or not targetPlayer.Character then
        return
    end

    local targetCharacter = targetPlayer.Character
    local targetRootPart = targetCharacter:WaitForChild("HumanoidRootPart", 5)
    local targetHumanoid = targetCharacter:WaitForChild("Humanoid", 5)
    if not targetRootPart or not targetHumanoid then
        return
    end

    -- Cria a linha (Beam)
    local attachment0 = Instance.new("Attachment")
    local attachment1 = Instance.new("Attachment")
    local beam = Instance.new("Beam")
    attachment0.Parent = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    attachment1.Parent = targetRootPart
    beam.Attachment0 = attachment0
    beam.Attachment1 = attachment1
    beam.Color = ColorSequence.new(currentStaticColor)
    beam.Width0 = 0.2
    beam.Width1 = 0.2
    beam.Transparency = NumberSequence.new(0)
    beam.Enabled = false
    beam.Parent = targetCharacter

    -- Cria o Highlight
    local highlight = Instance.new("Highlight")
    highlight.Adornee = targetCharacter
    highlight.FillColor = Color3.fromRGB(0, 255, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Enabled = false
    highlight.Parent = targetCharacter

    -- Cria o texto (BillboardGui)
    local billboard = Instance.new("BillboardGui")
    billboard.Adornee = targetRootPart
    billboard.Size = UDim2.new(0, 100, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.TextStrokeTransparency = 0
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextScaled = true
    textLabel.Parent = billboard
    billboard.Enabled = false
    billboard.Parent = targetCharacter

    return {beam = beam, attachment0 = attachment0, attachment1 = attachment1, highlight = highlight, billboard = billboard, textLabel = textLabel}
end

-- Função para atualizar o ESP
local function updateESP()
    spawn(function()
        while getgenv().ESPStaffEnabled or getgenv().ESPPlayersEnabled do
            local localCharacter = game.Players.LocalPlayer.Character
            local localRootPart = localCharacter and localCharacter:FindFirstChild("HumanoidRootPart")
            if not localRootPart then
                for _, data in pairs(espData) do
                    if data.beam then data.beam.Enabled = false end
                    data.highlight.Enabled = false
                    data.billboard.Enabled = false
                end
                wait()
                continue
            end

            for targetPlayer, data in pairs(espData) do
                local targetCharacter = targetPlayer.Character
                local targetRootPart = targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart")
                local targetHumanoid = targetCharacter and targetCharacter:FindFirstChild("Humanoid")

                if targetRootPart and targetHumanoid then
                    if isStaff(targetPlayer) and getgenv().ESPStaffEnabled then
                        data.highlight.Enabled = true
                        local distance = (localRootPart.Position - targetRootPart.Position).Magnitude
                        data.textLabel.Text = string.format("%s\nHP: %d/%d\nDist: %.1f", targetPlayer.Name, targetHumanoid.Health, targetHumanoid.MaxHealth, distance)
                        data.billboard.Enabled = getgenv().ESPTextEnabled or false
                    elseif not isStaff(targetPlayer) and getgenv().ESPPlayersEnabled then
                        if data.beam then data.beam.Enabled = true end
                        data.highlight.Enabled = true
                        local distance = (localRootPart.Position - targetRootPart.Position).Magnitude
                        data.textLabel.Text = string.format("%s\nHP: %d/%d\nDist: %.1f", targetPlayer.Name, targetHumanoid.Health, targetHumanoid.MaxHealth, distance)
                        data.billboard.Enabled = getgenv().ESPTextEnabled or false
                    else
                        if data.beam then data.beam.Enabled = false end
                        data.highlight.Enabled = false
                        data.billboard.Enabled = false
                    end
                else
                    if data.beam then data.beam.Enabled = false end
                    data.highlight.Enabled = false
                    data.billboard.Enabled = false
                end
            end
            wait()
        end
        for _, data in pairs(espData) do
            if data.beam then data.beam.Enabled = false end
            data.highlight.Enabled = false
            data.billboard.Enabled = false
        end
    end)
end

-- Função para ativar/desativar o ESP Staff
local function toggleESPStaff(state)
    getgenv().ESPStaffEnabled = state
    if state then
        for _, player in pairs(game.Players:GetPlayers()) do
            if isStaff(player) and not espData[player] then
                espData[player] = createESPStaff(player)
            end
        end
        game.Players.PlayerAdded:Connect(function(player)
            if isStaff(player) then
                espData[player] = createESPStaff(player)
            end
        end)
        updateESP()
        if rgbCycleEnabled then
            cycleRGB()
        end
    else
        for player, data in pairs(espData) do
            if isStaff(player) then
                if data.highlight then data.highlight:Destroy() end
                if data.billboard then data.billboard:Destroy() end
                espData[player] = nil
            end
        end
    end
end

-- Função para ativar/desativar o ESP Jogadores
local function toggleESPPlayers(state)
    getgenv().ESPPlayersEnabled = state
    if state then
        for _, player in pairs(game.Players:GetPlayers()) do
            if not isStaff(player) and player ~= game.Players.LocalPlayer and not espData[player] then
                espData[player] = createESPPlayer(player)
            end
        end
        game.Players.PlayerAdded:Connect(function(player)
            if not isStaff(player) and player ~= game.Players.LocalPlayer then
                espData[player] = createESPPlayer(player)
            end
        end)
        updateESP()
        if rgbCycleEnabled then
            cycleRGB()
        end
    else
        for player, data in pairs(espData) do
            if not isStaff(player) then
                if data.beam then data.beam:Destroy() end
                if data.attachment0 then data.attachment0:Destroy() end
                if data.attachment1 then data.attachment1:Destroy() end
                if data.highlight then data.highlight:Destroy() end
                if data.billboard then data.billboard:Destroy() end
                espData[player] = nil
            end
        end
    end
end

-- Função para atualizar a cor estática
local function updateStaticColor(color)
    currentStaticColor = color
    if not rgbCycleEnabled then
        for _, data in pairs(espData) do
            if data.highlight and isStaff(_) then
                data.highlight.FillColor = color
            end
            if data.beam and not isStaff(_) then
                data.beam.Color = ColorSequence.new(color)
            end
        end
    end
end

-- Função para o ciclo RGB
local function cycleRGB()
    spawn(function()
        local hue = 0
        while rgbCycleEnabled and (getgenv().ESPStaffEnabled or getgenv().ESPPlayersEnabled) do
            hue = (hue + 0.01) % 1
            local color = Color3.fromHSV(hue, 1, 1)
            for player, data in pairs(espData) do
                if isStaff(player) and data.highlight then
                    data.highlight.FillColor = color
                elseif not isStaff(player) and data.beam then
                    data.beam.Color = ColorSequence.new(color)
                end
            end
            wait(0.05)
        end
        if not rgbCycleEnabled and (getgenv().ESPStaffEnabled or getgenv().ESPPlayersEnabled) then
            updateStaticColor(currentStaticColor)
        end
    end)
end

-- Botão para ativar/desativar o ESP Staff
ESPTab:CreateButton({
    Name = "Esp-Staff(BETA)",
    Callback = function()
        getgenv().ESPStaffEnabled = not getgenv().ESPStaffEnabled
        toggleESPStaff(getgenv().ESPStaffEnabled)
        Rayfield:Notify({
            Title = "ESP Staff",
            Content = getgenv().ESPStaffEnabled and "ESP Staff ativado!" or "ESP Staff desativado!",
            Duration = 3,
            Image = 4483362458
        })
    end
})

-- Toggle para ativar/desativar o ESP Jogadores
ESPTab:CreateToggle({
    Name = "ESP Jogadores 👁️",
    CurrentValue = false,
    Callback = function(state)
        toggleESPPlayers(state)
        Rayfield:Notify({
            Title = "ESP Jogadores",
            Content = state and "ESP Jogadores ativado!" or "ESP Jogadores desativado!",
            Duration = 3,
            Image = 4483362458
        })
    end
})

-- Toggle para o texto do ESP
ESPTab:CreateToggle({
    Name = "Texto ESP 📝",
    CurrentValue = false,
    Callback = function(state)
        getgenv().ESPTextEnabled = state
    end
})

-- Toggle para o Ciclo RGB (ambos os ESPs)
ESPTab:CreateToggle({
    Name = "Ciclo RGB 🌈",
    CurrentValue = false,
    Callback = function(state)
        rgbCycleEnabled = state
        if state and (getgenv().ESPStaffEnabled or getgenv().ESPPlayersEnabled) then
            cycleRGB()
        elseif not state and (getgenv().ESPStaffEnabled or getgenv().ESPPlayersEnabled) then
            updateStaticColor(currentStaticColor)
        end
    end
})

-- ColorPicker para selecionar a cor estática (ambos os ESPs)
ESPTab:CreateColorPicker({
    Name = "Cor ESP",
    Color = Color3.fromRGB(0, 255, 0), -- Cor inicial verde
    Flag = "ColorPickerESP",
    Callback = function(Value)
        updateStaticColor(Value)
    end
})

-- Aba de Teleportes
local TeleportTab = Window:CreateTab("Teleports 🌀")
local Paragraph = TeleportTab:CreateParagraph({Title = "GRAND Teleports", Content = "Feito por Bernardo"})

-- Função para adicionar botões de teleporte
local function addTeleportButton(name, cframe)
    TeleportTab:CreateButton({
        Name = name,
        Callback = function()
            local player = game.Players.LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character:MoveTo(cframe.Position + Vector3.new(0, 3, 0))
            end
        end,
    })
end

-- Adicionando os botões de teletransporte para locais fixos
addTeleportButton("Teleport Praça", CFrame.new(-291.579559, 3.26299787, 342.192535))
addTeleportButton("Teleport Gás", CFrame.new(-469.959015, 3.25349784, -54.3936005))
addTeleportButton("Teleport HP", CFrame.new(-543.439941, 3.26299858, 645.16864))
addTeleportButton("Teleport Tabacaria", CFrame.new(-83.1141129, 13.1430578, 74.7073364))
addTeleportButton("Teleport Garagem", CFrame.new(-466.870148, 7.64567232, 350.242737))
addTeleportButton("Teleport Concessionaria", CFrame.new(-91.3902893, 8.07136822, 520.355347))
addTeleportButton("Teleport Gari", CFrame.new(-518.672852, 3.16749811, -1.16962147, 0, 0, -1, 0, 1, 0, 1, 0, 0))
addTeleportButton("Teleport Imobiliaria", CFrame.new(-284.904785, 8.26088619, -72.2896194, 0, 0, -1, 0, 1, 0, 1, 0, 0))
addTeleportButton("Teleport PM", CFrame.new(-980.181458, 2.27553082, 467.080536, 1, 0, 0, 0, 1, 0, 0, 0, 1))
addTeleportButton("Teleport PRF", CFrame.new(6662.24512, 36.6637421, 5047.83838, 0.707134247, 0, 0.707079291, 0, 1, 0, -0.707079291, 0, 0.707134247))
addTeleportButton("Teleport Mineração", CFrame.new(201.932144, 2.76136589, 145.50531, 0, 0, 1, 0, 1, -0, -1, 0, 0))
addTeleportButton("Teleport Mecânica", CFrame.new(-180.608261, 3.29813337, -532.4151, 0.422592998, -0, -0.906319618, 0, 1, -0, 0.906319618, 0, 0.422592998))
addTeleportButton("Teleport Fazenda", CFrame.new(817.243225, 3.26249814, -87.316864, 0, 0, 1, 0, 1, 0, -1, 0, 0))
addTeleportButton("Teleport Prefeitura", CFrame.new(-284.388458, 15.1148872, 88.0397873, 0, 0, -1, 0, 1, 0, 1, 0, 0))
addTeleportButton("Teleport Banco", CFrame.new(-27.2709007, 11.5685892, 418.200653, 1, 0, 0, 0, 1, 0, 0, 0, 1))
addTeleportButton("Teleport Ilegal", CFrame.new(12045.0146, 37.3188896, 12826.2041, -0.257956624, 0.00115467981, -0.966156006, -0.0795417428, 0.99657917, 0.0224280898, 0.962876916, 0.0826351941, -0.256982446))
addTeleportButton("Teleport predio 1", CFrame.new(-1595.23328, 204.074341, 555.895386, 0.939687431, -0.34203434, 1.81794167e-06, 1.81794167e-06, 1.02519989e-05, 1, -0.34203434, -0.93968749, 1.02519989e-05))
addTeleportButton("Teleport Devs Mini City", CFrame.new(2555.44263, 303.167755, -1004.13763, -0.422592998, 0, 0.906319618, 0, 1, 0, -0.906319618, 0, -0.422592998))


local rev = Window:CreateTab("Revistar 🔍")
local Paragraph = rev:CreateParagraph({Title = "Grand-Menu revist", Content = "Feito por Polengo 👨‍💻"})
local Section = rev:CreateSection("NECESSARIO")
local Button = rev:CreateButton({
   Name = "Puxar Itens 🎒",
   Callback = function()
   -- Função para deletar todas as NotifyGui
local function deletarNotifyGui()
    local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    for _, gui in ipairs(playerGui:GetChildren()) do
        if gui.Name == "NotifyGui" and gui:IsA("ScreenGui") then
            gui:Destroy() -- Deleta a NotifyGui
        end
    end
end

-- Lista de itens para pegar
local itens = {"AK47", "Uzi", " Planta Limpa", "IA2",  "Parafal", "Faca", "AR-15", "Glock 17", "IA2", "G3", "IPhone 14", "Agua", "Hamburguer", "Hi Power", "Natalina"}

-- Argumentos para a requisição
local args = {
    [1] = "mudaInv",
    [2] = "2",
    [4] = "1"
}

-- Loop principal
while true do
    -- Deletar todas as NotifyGui antes de pegar os itens
    deletarNotifyGui()

    -- Pegar itens
    for i, item in ipairs(itens) do
        if i <= 16 then  -- Só tenta alocar até 16 slots
            args[3] = item  -- Atualiza o item para o valor da vez
            args[2] = tostring(i)  -- Atribui o slot dinamicamente (1, 2, 3, ..., 16)

            -- Usar task.spawn() para execução sem delay
            task.spawn(function()
                -- Envia a requisição para o servidor
                game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("InvRemotes"):WaitForChild("InvRequest"):InvokeServer(unpack(args))
            end)
        end
    end

    wait(0)  -- Espera um frame para evitar lag
end
   end,
})
local Section = rev:CreateSection("PC")
local ww = rev:CreateToggle({
   Name = "mandar revistar (TECLA T)",
   CurrentValue = false,
   Flag = "rvst",
   Callback = function(Value)
       getgenv().Enabled = Value
   end,
})
local Section = rev:CreateSection("MOBILE")
local mob = rev:CreateButton({
   Name = "mandar revistar UI",
   Callback = function()
   local TextChatService = game:GetService("TextChatService")

-- Função para enviar a mensagem /revistar morto
local function sendRevistarMessage()
    local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
    channel:SendAsync("/revistar morto")
end

-- Cria a interface
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local CloseButton = Instance.new("TextButton")
local RevistarButton = Instance.new("TextButton")
local Title = Instance.new("TextLabel")

ScreenGui.Name = "RevistarUI"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Estilo do Frame
Frame.Size = UDim2.new(0, 300, 0, 150)
Frame.Position = UDim2.new(0.5, -150, 0.5, -75)
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.BackgroundTransparency = 0.1
Frame.Active = true
Frame.Draggable = true

-- Título
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Title.Text = "WAVE reviste"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18

-- Botão Fechar
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.SourceSansBold
CloseButton.TextSize = 18
CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)


-- Botão Revistar
RevistarButton.Size = UDim2.new(0.8, 0, 0.4, 0)
RevistarButton.Position = UDim2.new(0.1, 0, 0.5, -30)
RevistarButton.BackgroundColor3 = Color3.fromRGB(50, 150, 250)
RevistarButton.Text = "manda /revistar morto"
RevistarButton.TextColor3 = Color3.fromRGB(255, 255, 255)
RevistarButton.Font = Enum.Font.SourceSansBold
RevistarButton.TextSize = 20
RevistarButton.AutoButtonColor = true
RevistarButton.MouseButton1Click:Connect(function()
    sendRevistarMessage()
end)

-- Adiciona os elementos ao frame
Frame.Parent = ScreenGui
Title.Parent = Frame
CloseButton.Parent = Frame
RevistarButton.Parent = Frame
   end,
})
local Section = rev:CreateSection("Info")
local Paragraph = rev:CreateParagraph({Title = "Como usar?", Content = "Ensinamos a usar corretamente no nosso discord."})
-- Criar uma aba para teleporte
local Tab = Window:CreateTab("Teleporte Players", 4483362458)

-- Adicionar rótulo explicativo
Tab:CreateLabel("Clique no nome do jogador para teleportar")

-- Função para notificação simples
local function notificar(titulo, mensagem)
    Rayfield:Notify({
        Title = titulo,
        Content = mensagem,
        Duration = 3
    })
end

-- ===== INÍCIO DAS FUNÇÕES DE TELEPORTE AVANÇADAS =====

-- Função para teleportar usando método direto
local function teleporteDireto(personagemLocal, personagemAlvo)
    local hrpLocal = personagemLocal:FindFirstChild("HumanoidRootPart")
    local hrpAlvo = personagemAlvo:FindFirstChild("HumanoidRootPart")
    
    if hrpLocal and hrpAlvo then
        hrpLocal.CFrame = hrpAlvo.CFrame
        return true
    end
    return false
end

-- Função para teleportar usando TweenService (movimento suave)
local function teleporteTween(personagemLocal, personagemAlvo)
    local hrpLocal = personagemLocal:FindFirstChild("HumanoidRootPart")
    local hrpAlvo = personagemAlvo:FindFirstChild("HumanoidRootPart")
    
    if hrpLocal and hrpAlvo then
        local TweenService = game:GetService("TweenService")
        local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Linear)
        local tween = TweenService:Create(hrpLocal, tweenInfo, {CFrame = hrpAlvo.CFrame})
        tween:Play()
        return true
    end
    return false
end

-- Função para teleportar usando velocidade (bypass para alguns jogos)
local function teleporteVelocidade(personagemLocal, personagemAlvo)
    local hrpLocal = personagemLocal:FindFirstChild("HumanoidRootPart")
    local hrpAlvo = personagemAlvo:FindFirstChild("HumanoidRootPart")
    
    if hrpLocal and hrpAlvo then
        -- Salvar a velocidade original
        local velocidadeOriginal = hrpLocal.Velocity
        
        -- Aplicar velocidade zero
        hrpLocal.Velocity = Vector3.new(0, 0, 0)
        
        -- Teleportar
        hrpLocal.CFrame = hrpAlvo.CFrame
        
        -- Restaurar velocidade após pequeno atraso
        delay(0.1, function()
            if hrpLocal then
                hrpLocal.Velocity = velocidadeOriginal
            end
        end)
        
        return true
    end
    return false
end

-- Função para teleportar alterando estado do humanoid
local function teleporteComEstado(personagemLocal, personagemAlvo)
    local hrpLocal = personagemLocal:FindFirstChild("HumanoidRootPart")
    local hrpAlvo = personagemAlvo:FindFirstChild("HumanoidRootPart")
    local humanoidLocal = personagemLocal:FindFirstChildOfClass("Humanoid")
    
    if hrpLocal and hrpAlvo and humanoidLocal then
        -- Mudar estado para física e teleportar
        humanoidLocal:ChangeState(Enum.HumanoidStateType.Physics)
        wait(0.1)
        hrpLocal.CFrame = hrpAlvo.CFrame
        wait(0.1)
        humanoidLocal:ChangeState(Enum.HumanoidStateType.GettingUp)
        
        return true
    end
    return false
end

-- Função para teleportar usando várias tentativas e manipulação de network ownership
local function teleporteMaster(nomeJogadorAlvo)
    local jogadorLocal = game:GetService("Players").LocalPlayer
    local jogadorAlvo = game:GetService("Players"):FindFirstChild(nomeJogadorAlvo)
    
    if not jogadorAlvo then
        notificar("Erro", "Jogador não encontrado!")
        return false
    end
    
    local personagemLocal = jogadorLocal.Character
    local personagemAlvo = jogadorAlvo.Character
    
    if not personagemLocal or not personagemAlvo then
        notificar("Erro", "Personagem não carregado!")
        return false
    end
    
    -- Desativar colisões temporariamente
    local parts = personagemLocal:GetDescendants()
    local originalCollisionGroup = {}
    
    for _, part in ipairs(parts) do
        if part:IsA("BasePart") then
            originalCollisionGroup[part] = part.CollisionGroup
            pcall(function() part.CollisionGroup = "TeleportGroup" end)
        end
    end
    
    -- Tentar cada método com um pequeno atraso entre eles
    local sucesso = false
    
    -- Método 1: Teleporte direto
    sucesso = teleporteDireto(personagemLocal, personagemAlvo)
    
    -- Método 2: Se o primeiro não funcionou, tente com TweenService
    if not sucesso then
        wait(0.2)
        sucesso = teleporteTween(personagemLocal, personagemAlvo)
    end
    
    -- Método 3: Teleporte com manipulação de velocidade
    if not sucesso then
        wait(0.2)
        sucesso = teleporteVelocidade(personagemLocal, personagemAlvo)
    end
    
    -- Método 4: Teleporte com alteração de estado
    if not sucesso then
        wait(0.2)
        sucesso = teleporteComEstado(personagemLocal, personagemAlvo)
    end
    
    -- Método 5: Método forçado com CFrame + mudança de propriedades físicas
    if not sucesso then
        wait(0.2)
        local hrpLocal = personagemLocal:FindFirstChild("HumanoidRootPart")
        local hrpAlvo = personagemAlvo:FindFirstChild("HumanoidRootPart")
        
        if hrpLocal and hrpAlvo then
            -- Salvar propriedades originais
            local anchored = hrpLocal.Anchored
            local massless = hrpLocal.Massless
            local cancelForces = hrpLocal.CustomPhysicalProperties
            
            -- Modificar propriedades para facilitar o teleporte
            pcall(function()
                hrpLocal.Anchored = true
                hrpLocal.Massless = true
                hrpLocal.CustomPhysicalProperties = PhysicalProperties.new(0, 0, 0, 0, 0)
            end)
            
            -- Teleportar
            hrpLocal.CFrame = hrpAlvo.CFrame
            
            -- Restaurar propriedades
            delay(0.2, function()
                pcall(function()
                    if hrpLocal then
                        hrpLocal.Anchored = anchored
                        hrpLocal.Massless = massless
                        hrpLocal.CustomPhysicalProperties = cancelForces
                    end
                end)
            end)
            
            sucesso = true
        end
    end
    
    -- Restaurar colisões
    delay(1, function()
        for part, group in pairs(originalCollisionGroup) do
            pcall(function() part.CollisionGroup = group end)
        end
    end)
    
    -- Notificar resultado
    if sucesso then
        notificar("Teleporte", "Tentativa de teleporte executada para " .. nomeJogadorAlvo)
    else
        notificar("Falha", "Todas as tentativas de teleporte falharam!")
    end
    
    return sucesso
end

-- ===== FIM DAS FUNÇÕES DE TELEPORTE AVANÇADAS =====

-- Função para atualizar a lista de jogadores como botões (em ordem alfabética)
local function atualizarBotoes()
    -- Limpar botões anteriores (remover a seção e recriar)
    Tab:CreateSection("Jogadores Online")
    
    -- Obter lista de jogadores
    local jogadores = game:GetService("Players"):GetPlayers()
    
    -- Criar uma tabela com os nomes dos jogadores (excluindo o jogador local)
    local nomesJogadores = {}
    for _, player in pairs(jogadores) do
        if player ~= game:GetService("Players").LocalPlayer then
            table.insert(nomesJogadores, player.Name)
        end
    end
    
    -- Ordenar nomes em ordem alfabética
    table.sort(nomesJogadores)
    
    -- Criar botão para cada jogador
    for _, nome in ipairs(nomesJogadores) do
        Tab:CreateButton({
            Name = nome,
            Callback = function()
                teleporteMaster(nome)
            end
        })
    end
end






-- Botão para tentar corrigir personagem (usar se bugado após teleporte)
Tab:CreateButton({
    Name = "Corrigir Personagem",
    Callback = function()
        local localPlayer = game:GetService("Players").LocalPlayer
        
        if not localPlayer.Character then
            notificar("Erro", "Seu personagem não está carregado!")
            return
        end
        
        local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            -- Resetar estado do humanoid
            humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
            
            -- Tentar destravar
            pcall(function()
                for _, part in pairs(localPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.Velocity = Vector3.new(0, 0, 0)
                        part.RotVelocity = Vector3.new(0, 0, 0)
                        if part.Anchored then
                            part.Anchored = false
                        end
                    end
                end
            end)
            
            notificar("Corrigido", "Tentativa de correção do personagem realizada!")
        else
            notificar("Erro", "Humanoid não encontrado!")
        end
    end
})

-- Inicializar com a lista de jogadores
atualizarBotoes()

-- Definir grupo de colisão especial para teleporte (evita colisões durante teleporte)
pcall(function()
    local PhysicsService = game:GetService("PhysicsService")
    PhysicsService:CreateCollisionGroup("TeleportGroup")
    PhysicsService:CollisionGroupSetCollidable("TeleportGroup", "TeleportGroup", false)
end)

notificar("Script Carregado", "Script de teleporte avançado iniciado!")
print("Script de teleporte avançado iniciado!")

local otoTab = Window:CreateTab("Outros 🧩")
local Paragraph = otoTab:CreateParagraph({Title = "Grand-menu", Content = "feito por Bernardo"})
local Toggle = otoTab:CreateToggle({
    Name = "Anti-Staff(BETA)",
    CurrentValue = false,
    Flag = "ToggleKickCheck",
    Callback = function(Value)
        if Value then
            getgenv().KickCheck = true
            local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Lista de nomes a serem monitorados
local targetUsernames = {
    "jake56839ad", "CleitinDoGrau_Eb", "21peteca", "Lucalarte", "SPTmatheus123",
    "GuilhermeDRTgg", "Briessxz", "hardstyless", "Mundaka", "Isabelaaaaafofs",
    "HANRLLEY25", "kaleb_iaee", "brunizoraa", "rip_propleyfran", "pepezicador",
    "Jjhgul", "Dariosantos21048", "JEKER_2009", "tttonas", "MZPlug14k",
    "Dudubeterotv5", "Sargento_admOficial", "Cassiopia84un", "Hakplays", "Cleo_ptr"
}

-- Converte a lista para uma tabela para verificação rápida
local targetLookup = {}
for _, name in ipairs(targetUsernames) do
    targetLookup[name] = true
end

-- Inicializa a configuração global
getgenv().KickCheck = getgenv().KickCheck or true

-- Função para verificar e kickar caso necessário
local function checkForTargetPlayers()
    if not getgenv().KickCheck then return end

    for _, player in pairs(Players:GetPlayers()) do
        if targetLookup[player.Name] then
            warn("Usuário indesejado detectado:", player.Name)
            LocalPlayer:Kick("Um staff foi detectado no servidor.")
            break
        end
    end
end

-- Verifica inicialmente
checkForTargetPlayers()

-- Verifica sempre que um novo jogador entrar
Players.PlayerAdded:Connect(function(player)
    if getgenv().KickCheck and targetLookup[player.Name] then
        warn("Usuário indesejado detectado:", player.Name)
        LocalPlayer:Kick("Um staff foi detectado no servidor.")
    end
end)
print("check")
        else
            getgenv().KickCheck = false
        end
    end
})


-- Loop para checar a cada 5 segundos, apenas se ativado
task.spawn(function()
    while true do
        if checkActive then
            checkStaffTeam()
        end
        task.wait(5)
    end
end)


-- Verifica se getgenv está disponível
if getgenv == nil then
    error("getgenv não está definido neste ambiente.")
end

-- Configuração inicial do getgenv
getgenv().TeleportOnLowHealth = false -- Ativar/desativar o auto teleporte
getgenv().HealthThreshold = 10       -- Vida mínima antes de teleportar
getgenv().TargetCFrame = CFrame.new(-290.886688, 2.72974682, 363.851135, 1, 0, 0, 0, 1, 0, 0, 0, 1) -- CFrame de destino

-- Função para monitorar a vida do jogador
local function monitorHealth()
    local player = game.Players.LocalPlayer
    local character = player.Character
    if character and character:FindFirstChild("Humanoid") and character:FindFirstChild("HumanoidRootPart") then
        local humanoid = character.Humanoid
        local humanoidRootPart = character.HumanoidRootPart
        humanoid.HealthChanged:Connect(function(health)
            if health <= getgenv().HealthThreshold and getgenv().TeleportOnLowHealth then
                humanoidRootPart.CFrame = getgenv().TargetCFrame -- Teleportar para o CFrame
            end
        end)
    end
end

-- Conectar monitoramento ao evento CharacterAdded
game.Players.LocalPlayer.CharacterAdded:Connect(function()
    monitorHealth()
end)

-- Monitorar imediatamente se o personagem já existe
if game.Players.LocalPlayer.Character then
    monitorHealth()
end

-- Criação da seção na aba otoTab
local Section = otoTab:CreateSection("AUTO CL CONFIG")

-- Criação do Toggle para ativar/desativar o auto teleporte
local Toggle = otoTab:CreateToggle({
    Name = "Auto-CL(BETA)",
    CurrentValue = getgenv().TeleportOnLowHealth,
    Flag = "AutoTeleportToggle",
    Callback = function(Value)
        getgenv().TeleportOnLowHealth = Value
    end,
})

-- Criação do Slider para ajustar o limite de vida
local Slider = otoTab:CreateSlider({
    Name = "teleportar quando vida =",
    Range = {0, 100},
    Increment = 1,
    Suffix = " HP",
    CurrentValue = getgenv().HealthThreshold,
    Flag = "HealthThresholdSlider",
    Callback = function(Value)
        getgenv().HealthThreshold = Value
    end,
})

getgenv().Key = Enum.KeyCode.T
getgenv().Enabled = getgenv().Enabled or false
local UserInputService = game:GetService("UserInputService")

-- Configurações via getgenv
getgenv().Key = getgenv().Key or Enum.KeyCode.R -- Tecla padrão: R
getgenv().Enabled = getgenv().Enabled or false -- Estado inicial: desativado

-- Função para enviar a mensagem /revistar
local function sendRevistarMessage()
    local TextChatService = game:GetService("TextChatService")
    local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
    channel:SendAsync("/revistar morto")
end

-- Listener para detectar teclas pressionadas
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- Verifica se o script está ativado e se a tecla correta foi pressionada
    if getgenv().Enabled and input.KeyCode == getgenv().Key and not gameProcessed then
        sendRevistarMessage()
    end
end)


-- Sistema de Voo
local flying = false
local flySpeed = 15
local minSpeed = 1
local maxSpeed = 999
local speedStep = 5
local antiFallEnabled = false
local uiMinimized = false

local screenGuiFly = Instance.new("ScreenGui")
screenGuiFly.Name = "FlyGui"
screenGuiFly.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGuiFly.ResetOnSpawn = false
screenGuiFly.Enabled = false

local frameFly = Instance.new("Frame")
frameFly.Name = "FlyFrame"
frameFly.Size = UDim2.new(0, 220, 0, 120)
frameFly.Position = UDim2.new(0.5, -110, 0.1, 0)
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
speedLabel.Text = "Velocidade: " .. flySpeed
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

local function disableCollisions(character)
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = false end
    end
end

local function enableCollisions(character)
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = true end
    end
end

local function antiFallDamage()
    while antiFallEnabled do
        task.wait(0.1)
        if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            local rootPart = game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
            if humanoid and rootPart and not flying and rootPart.Velocity.Y < -50 then
                rootPart.Velocity = Vector3.new(rootPart.Velocity.X, 0, rootPart.Velocity.Z)
            end
        end
    end
end

local function startFlying()
    local character = game.Players.LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") or not character:FindFirstChildOfClass("Humanoid") then return false end
    local humanoidRootPart = character.HumanoidRootPart
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    humanoid.PlatformStand = true
    humanoidRootPart.Anchored = true
    disableCollisions(character)
    flying = true
    buttonFly.Text = "Desativar Voo"
    buttonFly.BackgroundColor3 = Color3.fromRGB(100, 50, 50)
    flyToggleButton.Text = "✈️ OFF"

    local connection = game:GetService("RunService").Heartbeat:Connect(function(deltaTime)
        if not flying or not character or not humanoidRootPart or not humanoid then
            if connection then connection:Disconnect() end
            return
        end

        local moveDirection = Vector3.new(0, 0, 0)
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + workspace.CurrentCamera.CFrame.LookVector end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection - workspace.CurrentCamera.CFrame.LookVector end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection - workspace.CurrentCamera.CFrame.RightVector end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + workspace.CurrentCamera.CFrame.RightVector end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.LeftShift) then moveDirection = moveDirection - Vector3.new(0, 1, 0) end

        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
            local speed = flySpeed * deltaTime * 10
            humanoidRootPart.Anchored = false
            humanoidRootPart.CFrame = humanoidRootPart.CFrame + moveDirection * speed
            local lookVector = workspace.CurrentCamera.CFrame.LookVector
            humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position) * CFrame.Angles(0, math.atan2(lookVector.X, lookVector.Z), 0)
            humanoidRootPart.Anchored = true
        end
    end)
    return true
end

local function stopFlying()
    local character = game.Players.LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local humanoidRootPart = character and character:FindFirstChild("HumanoidRootPart")
    if humanoid then humanoid.PlatformStand = false end
    if character then enableCollisions(character) end
    if humanoidRootPart then humanoidRootPart.Anchored = false end
    flying = false
    buttonFly.Text = "Ativar Voo ✈️"
    buttonFly.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    flyToggleButton.Text = "✈️ ON"
end

buttonFly.MouseButton1Click:Connect(function()
    if flying then stopFlying() else startFlying() end
end)

flyToggleButton.MouseButton1Click:Connect(function()
    if flying then stopFlying() else startFlying() end
end)

antiFallToggle.MouseButton1Click:Connect(function()
    antiFallEnabled = not antiFallEnabled
    antiFallToggle.Text = "Anti-Dano: " .. (antiFallEnabled and "ON" or "OFF")
    antiFallToggle.BackgroundColor3 = antiFallEnabled and Color3.fromRGB(100, 50, 50) or Color3.fromRGB(60, 60, 60)
    if antiFallEnabled then task.spawn(antiFallDamage) end
end)

local function toggleUIMinimize()
    if uiMinimized then
        frameFly.Size = UDim2.new(0, 220, 0, 120)
        buttonFly.Size = UDim2.new(0.9, 0, 0.3, 0)
        speedLabel.Size = UDim2.new(0.9, 0, 0.2, 0)
        plusButtonFly.Size = UDim2.new(0.2, 0, 0.2, 0)
        minusButtonFly.Size = UDim2.new(0.2, 0, 0.2, 0)
        antiFallToggle.Size = UDim2.new(0.9, 0, 0.2, 0)
        minimizeButtonFly.Text = "-"
        uiMinimized = false
    else
        frameFly.Size = UDim2.new(0, 150, 0, 80)
        buttonFly.Size = UDim2.new(0.9, 0, 0.4, 0)
        speedLabel.Size = UDim2.new(0.9, 0, 0.3, 0)
        plusButtonFly.Size = UDim2.new(0.2, 0, 0.3, 0)
        minusButtonFly.Size = UDim2.new(0.2, 0, 0.3, 0)
        antiFallToggle.Size = UDim2.new(0.9, 0, 0.3, 0)
        minimizeButtonFly.Text = "+"
        uiMinimized = true
    end
end

minimizeButtonFly.MouseButton1Click:Connect(toggleUIMinimize)

closeButtonFly.MouseButton1Click:Connect(function()
    screenGuiFly.Enabled = false
    if flying then stopFlying() end
end)

local function adjustSpeed(delta)
    flySpeed = math.clamp(flySpeed + delta, minSpeed, maxSpeed)
    speedLabel.Text = "Velocidade: " .. flySpeed
end

plusButtonFly.MouseButton1Click:Connect(function() adjustSpeed(speedStep) end)
minusButtonFly.MouseButton1Click:Connect(function() adjustSpeed(-speedStep) end)

local dragging, dragInput, dragStart, startPos
frameFly.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frameFly.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
frameFly.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
game:GetService("UserInputService").InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        frameFly.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

otoTab:CreateToggle({
    Name = "Fly ✈️",
    CurrentValue = false,
    Flag = "AtivarVoo",
    Callback = function(Value)
        screenGuiFly.Enabled = Value
        if not Value and flying then stopFlying() end
    end
})

otoTab:CreateKeybind({
    Name = "Ativar/Desativar FLY",
    CurrentKeybind = "G",
    Flag = "FlyKeybind",
    Callback = function()
        if screenGuiFly.Enabled then
            if flying then stopFlying() else startFlying() end
        end
    end
})

otoTab:CreateButton({
    Name = "Rejoin",
    Callback = function()
        print("Botão Rejoin clicado!")
        
        -- Enviar mensagem no chat
        local player = game.Players.LocalPlayer
        game:GetService("Chat"):Chat(player.Character.Head, "!rejoin", Enum.ChatColor.Blue)
        
        -- Aguardar um breve momento para a mensagem ser visível
        wait(1)
        
        -- Realizar o rejoin
        local teleportService = game:GetService("TeleportService")
        teleportService:Teleport(game.PlaceId, player)
    end
})

local CrediTab = Window:CreateTab("CREDITOS", 9405926389)
CrediTab:CreateParagraph({
    Title = "Creditos ℹ️",
    Content = "Feito por polenginho"
})




loadstring(game:HttpGet("https://raw.githubusercontent.com/Bernass79/KEY-SISTEM/refs/heads/main/README.md"))()
