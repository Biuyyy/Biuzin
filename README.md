local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- by Boston and ChatGPT
local function createHighlight(player)
    if player.Character and not player.Character:FindFirstChild("ESPHighlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESPHighlight"
        highlight.FillColor = Color3.fromRGB(255, 0, 0)
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.FillTransparency = 0.6 -- ปรับให้จางลง
        highlight.OutlineTransparency = 0
        highlight.Adornee = player.Character
        highlight.Parent = player.Character
    end
end


local function createNameTag(player)
    if player.Character and player.Character:FindFirstChild("Head") and not player.Character.Head:FindFirstChild("NameTag") then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "NameTag"
        billboard.Adornee = player.Character.Head
        billboard.Size = UDim2.new(0, 130, 0, 25) -- ปรับให้เล็กลง
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = player.Character.Head

        local textLabel = Instance.new("TextLabel")
        textLabel.Name = "TagLabel"
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.BackgroundTransparency = 1
        textLabel.TextColor3 = Color3.new(1, 1, 1)
        textLabel.Font = Enum.Font.Cartoon
        textLabel.TextScaled = true
        textLabel.TextStrokeTransparency = 0.6
        textLabel.Text = ""
        textLabel.Parent = billboard
    end
end


local function updateNameTag(player)
    if player.Character and player.Character:FindFirstChild("Head") and player.Character.Head:FindFirstChild("NameTag") and player.Character:FindFirstChild("Humanoid") then
        local tag = player.Character.Head.NameTag.TagLabel
        local distance = (LocalPlayer.Character.PrimaryPart.Position - player.Character.PrimaryPart.Position).Magnitude
        local health = math.floor(player.Character.Humanoid.Health)
        tag.Text = player.Name .. " | " .. string.format("%.0f", distance).."m | ❤️"..health
    end
end


local function updateHighlight(player)
    if player.Character and player.Character:FindFirstChild("ESPHighlight") and player.Character:FindFirstChild("Humanoid") then
        if player.Character.Humanoid.Health <= 0 then
            player.Character.ESPHighlight.FillColor = Color3.fromRGB(120, 0, 0) -- แดงเข้มตอนตาย
        else
            player.Character.ESPHighlight.FillColor = Color3.fromRGB(255, 0, 0) -- แดงสดตอนอยู่
        end
    end
end


local function setupESP(player)
    if player ~= LocalPlayer then
        player.CharacterAdded:Connect(function()
            wait(0.1)
            createHighlight(player)
            createNameTag(player)
        end)
        if player.Character then
            createHighlight(player)
            createNameTag(player)
        end
    end
end


for _, player in ipairs(Players:GetPlayers()) do
    setupESP(player)
end


Players.PlayerAdded:Connect(function(player)
    setupESP(player)
end)


while true do
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if player.Character and player.Character:FindFirstChild("Humanoid") then
                updateNameTag(player)
                updateHighlight(player)
            end
        end
    end
    wait(0.3)
end
local fov = 40
local maxTransparency = 0.1 -- Transparência máxima dentro do círculo (0.1 = 10% de transparência)
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local Cam = game.Workspace.CurrentCamera

local FOVring = Drawing.new("Circle")
FOVring.Visible = true
FOVring.Thickness = 2
FOVring.Color = Color3.fromRGB(128, 0, 128) -- Cor roxa
FOVring.Filled = false
FOVring.Radius = fov
FOVring.Position = Cam.ViewportSize / 2

local function updateDrawings()
    local camViewportSize = Cam.ViewportSize
    FOVring.Position = camViewportSize / 2
end

local function onKeyDown(input)
    if input.KeyCode == Enum.KeyCode.Delete then
        RunService:UnbindFromRenderStep("FOVUpdate")
        FOVring:Remove()
    end
end

UserInputService.InputBegan:Connect(onKeyDown)

local function lookAt(target)
    local lookVector = (target - Cam.CFrame.Position).unit
    local newCFrame = CFrame.new(Cam.CFrame.Position, Cam.CFrame.Position + lookVector)
    Cam.CFrame = newCFrame
end

local function calculateTransparency(distance)
    -- Ajuste a transparência com base na distância do centro do círculo
    local maxDistance = fov -- A distância máxima do centro do círculo
    local transparency = (1 - (distance / maxDistance)) * maxTransparency
    return transparency
end

local function getClosestPlayerInFOV(trg_part)
    local nearest = nil
    local last = math.huge
    local playerMousePos = Cam.ViewportSize / 2

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer then
            local part = player.Character and player.Character:FindFirstChild(trg_part)
            if part then
                local ePos, isVisible = Cam:WorldToViewportPoint(part.Position)
                local distance = (Vector2.new(ePos.x, ePos.y) - playerMousePos).Magnitude

                if distance < last and isVisible and distance < fov then
                    last = distance
                    nearest = player
                end
            end
        end
    end

    return nearest
end

RunService.RenderStepped:Connect(function()
    updateDrawings()
    local closest = getClosestPlayerInFOV("Head")
    if closest and closest.Character:FindFirstChild("Head") then
        lookAt(closest.Character.Head.Position)
    end
    
    if closest then
        local ePos, isVisible = Cam:WorldToViewportPoint(closest.Character.Head.Position)
        local distance = (Vector2.new(ePos.x, ePos.y) - (Cam.ViewportSize / 2)).Magnitude
        FOVring.Transparency = calculateTransparency(distance)
    else
        FOVring.Transparency = 0.1 -- Mantenha completamente visível quando nenhum jogador estiver no FOV
    end
end)
-- Toggle Flutuante Mobile (Luau - Roblox Style)
-- Script simples, flutuante, otimizado para mobile
-- O toggle ativa/desativa o que estiver "acima" dele (ex: menu, features, etc)

local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FloatingToggleGui"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

-- Frame principal flutuante (Toggle)
local toggleFrame = Instance.new("Frame")
toggleFrame.Name = "ToggleFrame"
toggleFrame.Size = UDim2.new(0, 60, 0, 60)
toggleFrame.Position = UDim2.new(1, -80, 0.5, -30)  -- Canto direito central
toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
toggleFrame.BorderSizePixel = 0
toggleFrame.BackgroundTransparency = 0.1
toggleFrame.Parent = screenGui

-- Arredondar
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 30)
uiCorner.Parent = toggleFrame

-- Stroke para visual bonito
local uiStroke = Instance.new("UIStroke")
uiStroke.Color = Color3.fromRGB(0, 170, 255)
uiStroke.Thickness = 2
uiStroke.Parent = toggleFrame

-- Ícone ou texto dentro do toggle
local toggleText = Instance.new("TextLabel")
toggleText.Name = "Status"
toggleText.Size = UDim2.new(1, 0, 1, 0)
toggleText.BackgroundTransparency = 1
toggleText.Text = "ON"
toggleText.TextColor3 = Color3.fromRGB(0, 255, 100)
toggleText.TextScaled = true
toggleText.Font = Enum.Font.GothamBold
toggleText.Parent = toggleFrame

-- Tornar flutuante / arrastável no mobile
local dragging = false
local dragInput
local dragStart
local startPos

local function updateInput(input)
	if dragging then
		local delta = input.Position - dragStart
		toggleFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end

toggleFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = toggleFrame.Position
		
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

toggleFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		updateInput(input)
	end
end)

-- Estado do Toggle
local toggled = true

-- Função que ativa/desativa o que está "acima" dele
local function toggleFunction(state)
	if state then
		-- === ATIVE AQUI O QUE VOCÊ QUER CONTROLAR ===
		-- Exemplo: mostrar um menu acima do toggle
		print("✅ Toggle ATIVADO - Ativando features acima...")
		
		toggleText.Text = "ON"
		toggleText.TextColor3 = Color3.fromRGB(0, 255, 100)
		toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
		-- toggleFrame.BackgroundColor3 = Color3.fromRGB(0, 100, 0) -- verde
		
		-- Exemplo: ativar algo acima
		-- if menuFrame then menuFrame.Visible = true end
		
	else
		print("⛔ Toggle DESATIVADO")
		toggleText.Text = "OFF"
		toggleText.TextColor3 = Color3.fromRGB(255, 80, 80)
		-- toggleFrame.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
	end
end

-- Clique/toque no toggle
toggleFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		toggled = not toggled
		toggleFunction(toggled)
	end
end)

-- Inicializar
toggleFunction(toggled)

print("Toggle flutuante mobile carregado! Arraste para mover.")
-- =============================================
--  OTIMIZADOR DE FPS - Roblox Lua
--  Versao limpa sem emojis
-- =============================================

local Players = game:GetService("Players")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

print("Otimizador de FPS ativado!")

-- Configuracoes
local settings = {
    DisableShadows = true,
    DisablePostEffects = true,
    DisableParticles = true,
    DisableFog = true,
}

local function Optimize()
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    settings().Graphics.QualityLevel = 1
    
    Lighting.Brightness = 1
    Lighting.ClockTime = 12
    Lighting.FogEnd = 1000000
    Lighting.FogStart = 1000000
    Lighting.GlobalShadows = false
    Lighting.EnvironmentDiffuseScale = 0.1
    Lighting.EnvironmentSpecularScale = 0.1

    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Fire") or obj:IsA("Smoke") then
            obj.Enabled = false
        elseif obj:IsA("BloomEffect") or obj:IsA("BlurEffect") or obj:IsA("ColorCorrectionEffect") or obj:IsA("SunRaysEffect") then
            obj.Enabled = false
        end
    end

    print("Otimizacoes aplicadas!")
end

-- Interface
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FPSOptimizer"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = playerGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 140)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = Frame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = "Otimizador FPS"
Title.TextColor3 = Color3.fromRGB(0, 255, 100)
Title.TextSize = 16
Title.Font = Enum.Font.GothamBold
Title.Parent = Frame

local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(1, 0, 0, 30)
Status.Position = UDim2.new(0, 0, 0, 35)
Status.BackgroundTransparency = 1
Status.Text = "FPS: Calculando..."
Status.TextColor3 = Color3.fromRGB(255, 255, 255)
Status.TextSize = 14
Status.Parent = Frame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.9, 0, 0, 40)
ToggleButton.Position = UDim2.new(0.05, 0, 0, 75)
ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
ToggleButton.Text = "Otimizacoes Ativadas"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 15
ToggleButton.Font = Enum.Font.GothamSemibold
ToggleButton.Parent = Frame

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 6)
buttonCorner.Parent = ToggleButton

local enabled = true

ToggleButton.MouseButton1Click:Connect(function()
    enabled = not enabled
    if enabled then
        ToggleButton.Text = "Otimizacoes Ativadas"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 170, 100)
        Optimize()
    else
        ToggleButton.Text = "Otimizacoes Desativadas"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(170, 0, 0)
        print("Otimizacoes desativadas.")
    end
end)

-- Monitor de FPS
RunService.RenderStepped:Connect(function()
    local fps = math.floor(1 / RunService.RenderStepped:Wait())
    Status.Text = "FPS: " .. fps
end)

Optimize()

print("Script de otimizacao carregado com sucesso!")
