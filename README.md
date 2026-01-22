--// INTRO - LOGO ONLY + SOUND GRADIENT
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local StarterGui = game:GetService("StarterGui")
local ProximityPromptService = game:GetService("ProximityPromptService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local PlayerGui = player:WaitForChild("PlayerGui")

--------------------------------------------------
-- VARIÃVEIS GLOBAIS
--------------------------------------------------
local Character, HRP, Humanoid
local SpeedEnabled, SpeedValue = false, 30
local JumpEnabled, JumpValue = false, 50
local AntiLagEnabled = false
local AutoGrabEnabled = false
local AutoGrabScript = nil
local AllowFriendsEnabled = false
local AllowFriendsGui = nil
local XRayEnabled = false
local XRayConnection = nil

-- ConfiguraÃ§Ãµes
local TARGET_TOOL = "Flying Carpet"
local XRAY_TRANSPARENCY = 0.7
local XRAY_BASE_NAME = "Base"

--------------------------------------------------
-- INTRO
--------------------------------------------------
local SplashGui = Instance.new("ScreenGui")
SplashGui.Name = "FluxoIntro"
SplashGui.IgnoreGuiInset = true
SplashGui.ResetOnSpawn = false
SplashGui.Parent = PlayerGui

local Logo = Instance.new("ImageLabel")
Logo.Parent = SplashGui
Logo.AnchorPoint = Vector2.new(0.5, 0.5)
Logo.Position = UDim2.fromScale(0.5, 0.5)
Logo.Size = UDim2.fromScale(0.28, 0.28)
Logo.BackgroundTransparency = 1
Logo.Image = "rbxassetid://132564087673178"
Logo.ImageTransparency = 1

local Sound = Instance.new("Sound")
Sound.Parent = SplashGui
Sound.SoundId = "rbxassetid://184352963"
Sound.Volume = 0
Sound.Looped = false

Sound:Play()
TweenService:Create(Logo, TweenInfo.new(0.8), {ImageTransparency = 0.25}):Play()
TweenService:Create(Sound, TweenInfo.new(1.2), {Volume = 1}):Play()
task.wait(2.3)

TweenService:Create(Logo, TweenInfo.new(0.6), {ImageTransparency = 1}):Play()
TweenService:Create(Sound, TweenInfo.new(0.6), {Volume = 0}):Play()
task.wait(0.8)
SplashGui:Destroy()

--------------------------------------------------
-- SETUP DO PERSONAGEM
--------------------------------------------------
local function SetupCharacter(char)
    Character = char
    HRP = char:WaitForChild("HumanoidRootPart")
    Humanoid = char:WaitForChild("Humanoid")
end

if player.Character then
    SetupCharacter(player.Character)
end
player.CharacterAdded:Connect(SetupCharacter)

--------------------------------------------------
-- FUNÃ‡Ã•ES DAS FEATURES
--------------------------------------------------
-- ANTI LAG
local function AtivarAntiLag()
    print("ðŸ§Š Anti Lag ATIVADO")
    
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 9e9
    Lighting.EnvironmentDiffuseScale = 0
    Lighting.EnvironmentSpecularScale = 0

    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            obj.Material = Enum.Material.Plastic
            obj.CastShadow = false
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") then
            obj.Enabled = false
        end
    end
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "Anti Lag ATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

local function DesativarAntiLag()
    print("ðŸ§Š Anti Lag DESATIVADO")
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level21
    Lighting.GlobalShadows = true
    Lighting.FogEnd = 100000
    Lighting.EnvironmentDiffuseScale = 1
    Lighting.EnvironmentSpecularScale = 1
end

-- AUTO GRAB AVANÃ‡ADO
local function StartAdvancedAutoGrab()
    if AutoGrabScript then return end
    
    print("ðŸ”¥ Iniciando Auto Grab AvanÃ§ado...")
    
    AutoGrabScript = {
        Running = true,
        Connections = {}
    }
    
    local isTeleporting = false
    
    local function onPromptAdded(prompt)
        if not AutoGrabScript or not AutoGrabScript.Running then return end
        
        local combinedText = (prompt.ActionText .. prompt.ObjectText):lower()
        
        if combinedText:find("steal") or combinedText:find("take") or combinedText:find("grab") then
            task.spawn(function()
                local isHolding = false
                
                while prompt and prompt.Parent and AutoGrabScript and AutoGrabScript.Running do
                    task.wait(0.1)
                    
                    if isTeleporting then 
                        if isHolding then 
                            pcall(function() prompt:InputHoldEnd() end)
                            isHolding = false 
                        end
                        continue 
                    end

                    local function getPromptLocation(promptObj)
                        if not promptObj or not promptObj.Parent then return nil end
                        if promptObj.Parent:IsA("Attachment") then
                            return promptObj.Parent.WorldPosition
                        elseif promptObj.Parent:IsA("BasePart") then
                            return promptObj.Parent.Position
                        elseif promptObj.Parent:IsA("Model") then
                            return promptObj.Parent:GetPivot().Position
                        end
                        return nil
                    end

                    local promptPos = getPromptLocation(prompt)
                    if not promptPos then continue end
                    
                    local dist = player:DistanceFromCharacter(promptPos)
                    local maxDist = prompt.MaxActivationDistance or 10
                    
                    if dist <= maxDist then
                        if not isHolding then
                            local function equipToolNow()
                                local char = player.Character
                                if not char then return false end
                                
                                if char:FindFirstChild(TARGET_TOOL) then 
                                    return true 
                                end
                                
                                local backpack = player:FindFirstChild("Backpack")
                                if backpack then
                                    local tool = backpack:FindFirstChild(TARGET_TOOL)
                                    if tool then 
                                        tool.Parent = char
                                        task.wait(0.1)
                                        return true 
                                    end
                                end
                                return false
                            end
                            
                            equipToolNow()
                            task.wait(0.1)
                            pcall(function() 
                                prompt:InputHoldBegin() 
                                isHolding = true
                            end)
                        end
                    else
                        if isHolding then
                            pcall(function() prompt:InputHoldEnd() end)
                            isHolding = false
                        end
                    end
                end
            end)
        end
    end

    for _, p in pairs(Workspace:GetDescendants()) do
        if p:IsA("ProximityPrompt") then 
            onPromptAdded(p) 
        end
    end

    local promptConnection = ProximityPromptService.PromptShown:Connect(function(prompt)
        onPromptAdded(prompt)
    end)
    
    table.insert(AutoGrabScript.Connections, promptConnection)
    
    local workspaceConnection = Workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("ProximityPrompt") then
            onPromptAdded(descendant)
        end
    end)
    
    table.insert(AutoGrabScript.Connections, workspaceConnection)
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "Auto Grab ATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

local function StopAdvancedAutoGrab()
    if not AutoGrabScript then return end
    
    print("ðŸ›‘ Parando Auto Grab AvanÃ§ado...")
    
    for _, connection in ipairs(AutoGrabScript.Connections) do
        pcall(function() connection:Disconnect() end)
    end
    
    AutoGrabScript.Running = false
    AutoGrabScript = nil
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "Auto Grab DESATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

-- ALLOW FRIENDS
local function CreateAllowFriendsButton()
    if AllowFriendsGui then return end
    
    print("ðŸ‘¥ Criando botÃ£o Allow Friends...")
    
    AllowFriendsGui = Instance.new("ScreenGui")
    AllowFriendsGui.Name = "AllowFriends"
    AllowFriendsGui.ResetOnSpawn = false
    AllowFriendsGui.Parent = game:GetService("CoreGui")

    local button = Instance.new("TextButton")
    button.Name = "DraggableButton"
    button.Size = UDim2.fromOffset(140, 50)
    button.Position = UDim2.fromOffset(200, 200)  -- Starting position
    button.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    button.BorderSizePixel = 0
    button.Text = "Allow Friends"
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.GothamBold
    button.TextSize = 18
    button.Parent = AllowFriendsGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = button

    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(0, 125, 255)
    stroke.Thickness = 1.5
    stroke.Transparency = 0.3
    stroke.Parent = button

    local dragging = false
    local dragStart = nil
    local startPos = nil

    local function updateInput(input)
        if dragging then
            local delta = input.Position - dragStart
            button.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end

    button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = button.Position
        end
    end)

    button.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            updateInput(input)
        end
    end)

    button.MouseButton1Click:Connect(function()
        local success = pcall(function()
            game:GetService("ReplicatedStorage"):WaitForChild("Packages"):WaitForChild("Net"):WaitForChild("RE/PlotService/ToggleFriends"):FireServer()
        end)
        
        if success then
            pcall(function()
                StarterGui:SetCore("SendNotification", {
                    Title = "Allow Friends",
                    Text = "PermissÃ£o alterada com sucesso!",
                    Duration = 3,
                    Icon = "rbxassetid://6723928013"
                })
            end)
        end
    end)

    button.TouchTap:Connect(function()
        local success = pcall(function()
            game:GetService("ReplicatedStorage"):WaitForChild("Packages"):WaitForChild("Net"):WaitForChild("RE/PlotService/ToggleFriends"):FireServer()
        end)
        
        if success then
            pcall(function()
                StarterGui:SetCore("SendNotification", {
                    Title = "Allow Friends",
                    Text = "PermissÃ£o alterada com sucesso!",
                    Duration = 3,
                    Icon = "rbxassetid://6723928013"
                })
            end)
        end
    end)
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "BotÃ£o Allow Friends criado!",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

local function RemoveAllowFriendsButton()
    if AllowFriendsGui then
        AllowFriendsGui:Destroy()
        AllowFriendsGui = nil
        print("ðŸ‘¥ BotÃ£o Allow Friends removido")
        
        pcall(function()
            StarterGui:SetCore("SendNotification", {
                Title = "Fluxo Hub",
                Text = "BotÃ£o Allow Friends removido!",
                Duration = 3,
                Icon = "rbxassetid://6723928013"
            })
        end)
    end
end

-- X-RAY (BASES TRANSPARENTES)
local function AplicarTransparenciaXRay()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name:lower():find(XRAY_BASE_NAME:lower()) then
            v.LocalTransparencyModifier = XRAY_TRANSPARENCY
        end
    end
end

local function RemoverTransparenciaXRay()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name:lower():find(XRAY_BASE_NAME:lower()) then
            v.LocalTransparencyModifier = 0
        end
    end
end

local function AtivarXRay()
    if XRayEnabled then return end
    
    print("ðŸ‘ï¸ X-Ray ATIVADO")
    XRayEnabled = true
    
    AplicarTransparenciaXRay()
    
    XRayConnection = Workspace.DescendantAdded:Connect(function(obj)
        task.wait(0.1)
        if XRayEnabled and obj:IsA("BasePart") and obj.Name:lower():find(XRAY_BASE_NAME:lower()) then
            obj.LocalTransparencyModifier = XRAY_TRANSPARENCY
        end
    end)
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "X-Ray ATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
    
    -- Reconectar transparÃªncia quando voltar ao jogo
    UserInputService.WindowFocused:Connect(function()
        if not XRayEnabled then return end
        task.wait(0.5)
        AplicarTransparenciaXRay()
    end)
end

local function DesativarXRay()
    if not XRayEnabled then return end
    
    print("ðŸ‘ï¸ X-Ray DESATIVADO")
    XRayEnabled = false
    
    RemoverTransparenciaXRay()
    
    if XRayConnection then
        XRayConnection:Disconnect()
        XRayConnection = nil
    end
    
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "Fluxo Hub",
            Text = "X-Ray DESATIVADO",
            Duration = 3,
            Icon = "rbxassetid://6723928013"
        })
    end)
end

--------------------------------------------------
-- INTERFACE DO HUB
--------------------------------------------------
local Gui = Instance.new("ScreenGui")
Gui.Name = "FluxoHubSmall"
Gui.ResetOnSpawn = false
Gui.Parent = PlayerGui

-- Hub Principal
local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.fromOffset(200, 315)  -- Aumentado para 315 para caber o novo botÃ£o
Main.Position = UDim2.fromScale(0.03, 0.3)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
Main.Active = true
Main.Draggable = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0,14)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(160, 0, 255)

local WhiteStroke = Instance.new("UIStroke", Main)
WhiteStroke.Thickness = 1
WhiteStroke.Color = Color3.fromRGB(255,255,255)
WhiteStroke.Transparency = 0.4

-- TÃ­tulo
local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1,0,0,35)
Title.BackgroundTransparency = 1
Title.Text = "FLUXO HUB"
Title.Font = Enum.Font.GothamBlack
Title.TextScaled = true
Title.TextColor3 = Color3.fromRGB(200, 100, 255)

--------------------------------------------------
-- FUNÃ‡ÃƒO PARA CRIAR BOTÃ•ES (SEM ANIMAÃ‡Ã•ES)
--------------------------------------------------
local function CreateButton(text, y, isToggle, inputValue, showStateText)
    local btn = Instance.new("TextButton", Main)
    btn.Size = UDim2.new(0.85, 0, 0, 32)
    btn.Position = UDim2.new(0.075, 0, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.AutoButtonColor = false  -- Desativa efeitos automÃ¡ticos do Roblox
    
    -- Configurar alinhamento do texto para a esquerda
    btn.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Se showStateText for false, mostra apenas o texto, sem "OFF"
    if showStateText == false then
        btn.Text = text
    else
        btn.Text = text .. " OFF"
    end
    
    btn.BorderSizePixel = 0

    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,10)

    local valueBox
    if isToggle then
        valueBox = Instance.new("TextBox")
        valueBox.Parent = Main
        valueBox.Size = UDim2.new(0.3, 0, 0, 25)
        valueBox.Position = UDim2.new(0.65, 0, 0, y + 3)
        valueBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        valueBox.TextColor3 = Color3.fromRGB(255, 255, 255)
        valueBox.Font = Enum.Font.Gotham
        valueBox.TextSize = 12
        valueBox.Text = tostring(inputValue or 0)
        valueBox.ClearTextOnFocus = false
        valueBox.TextXAlignment = Enum.TextXAlignment.Center
        Instance.new("UICorner", valueBox).CornerRadius = UDim.new(0, 5)
        
        valueBox.FocusLost:Connect(function()
            local num = tonumber(valueBox.Text)
            if num then
                if text == "SPEED" then
                    SpeedValue = num
                elseif text == "INF JUMP" then
                    JumpValue = num
                end
            else
                if text == "SPEED" then
                    valueBox.Text = tostring(SpeedValue)
                elseif text == "INF JUMP" then
                    valueBox.Text = tostring(JumpValue)
                end
            end
        end)
    end

    local ligado = false
    
    btn.MouseButton1Click:Connect(function()
        ligado = not ligado
        
        if ligado then
            -- Apenas muda a cor para verde, instantaneamente
            btn.BackgroundColor3 = Color3.fromRGB(0, 150, 50)
            -- Se showStateText for false, mantÃ©m apenas o texto, sem "ON"
            if showStateText == false then
                btn.Text = text
            else
                btn.Text = text .. " ON"
            end
        else
            -- Volta para a cor original, instantaneamente
            btn.BackgroundColor3 = Color3.fromRGB(40, 0, 70)
            -- Se showStateText for false, mantÃ©m apenas o texto, sem "OFF"
            if showStateText == false then
                btn.Text = text
            else
                btn.Text = text .. " OFF"
            end
        end
        
        if text == "SPEED" then
            SpeedEnabled = ligado
        elseif text == "INF JUMP" then
            JumpEnabled = ligado
        elseif text == "ANTI LAG" then
            AntiLagEnabled = ligado
            if ligado then
                AtivarAntiLag()
            else
                DesativarAntiLag()
            end
        elseif text == "AUTO GRAB" then
            AutoGrabEnabled = ligado
            if ligado then
                StartAdvancedAutoGrab()
      
