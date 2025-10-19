-- ESP Script com Team Check e Interface
-- Desenvolvido por forwalkin(victor)
-- Para uso autorizado em testes

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

-- Variáveis globais
local LocalPlayer = Players.LocalPlayer
local ESPEnabled = false
local TeamCheckEnabled = true
local ESPObjects = {}

-- Criação da Interface
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ESPGui"
ScreenGui.Parent = game:GetService("CoreGui") or LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0,, 200)
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Arredondar cantos
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- Título
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(1, 0, 0, 40)
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
TitleLabel.Text = "ESP System v1.0"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 18
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Parent = MainFrame

-- Botão de ativar/desativar ESP
local ESPToggle = Instance.new("TextButton")
ESPToggle.Name = "ESPToggle"
ESPToggle.Size = UDim2.new(0.8, 0, 0, 35)
ESPToggle.Position = UDim2.new(0.1, 0, 0.25, 0)
ESPToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
ESPToggle.Text = "Ativar ESP"
ESPToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
ESPToggle.TextSize = 14
ESPToggle.Font = Enum.Font.Gotham
ESPToggle.Parent = MainFrame

---- Botão de Team Check
local TeamCheckToggle = Instance.new("TextButton")
TeamCheckToggle.Name = "TeamCheckToggle"
TeamCheckToggle.Size = UDim2.new(0.8, 0, 0, 35)
TeamCheckToggle.Position = UDim2.new(0..1, 0, 0.5, 0)
TeamCheckToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
TeamCheckToggle.Text = "Team Check: ON"
TeamCheckToggle.TextColor3 = Color3.fromRGB(0, 255, 0)
TeamCheckToggle.TextSize = 14
TeamCheckToggle.Font = Enum.Font.Gotham
TeamCheckToggle.Parent = MainFrame

-- Label de créditos
local CreditsLabel = Instance.new("TextLabel")
CreditsLabel.Name = "CreditsLabel"
CreditsLabel.Size = UDim2.new(1, 0, 0, 30)
CreditsLabel.Position = UDim2.new(0, 0, 0.85, 0)
CreditsLabel.BackgroundTransparency = 1
CreditsLabel.Text = "Desenvolvido por forwalkin(victor)"
CreditsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
CreditsLabel.TextSize = 12
CreditsLabel.Font = Enum.Font.Gotham
CreditsLabel.Parent = MainFrame

-- Função para criar ESP de um jogador
function CreateESP(player)
    if not player.Character then return end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    
    if not humanoidRootPart or not humanoid then return end
    
    -- Verificar se o ESP já existe para este jogador
    if ESPObjects[player] then
        ESPObjects[player].Box:Destroy()
        ESPObjects[player].Label:Destroy()
        ESPObjects[player] = nil
    end
    
    -- Criar caixa do ESP
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESPBox"
    box box.Adornee = humanoidRootPart
    box.AlwaysOnTop = true
    box.ZIndex = 5
    box.Size = Vector3.new(4, 6, 4)
    box.Color3 = GetTeamColor(player)
    box.Transparency = 0.3
    box.Parent = humanoidRootPart
    
    -- Criar label do ESP
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESPLabel"
    billboard.Adornee = humanoidRootPart
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3.5, 0)
    billboard.AlwaysOnTop = true
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = player.Name .. "
[" .. math.floor(humanoid.Health) .. " HP]"
    label.TextColor3 = GetTeamColor(player)
    label.TextSize = 14
    label.Font = Enum.Font.GothamBold
    label.Parent = billboard
    
    billboard.Parent = humanoidRootPart
    
    -- Armazenar referências
    ESPObjects[player] = {
        Box = box,
        Label = billboard,
        Connection = nil
    }
    
    -- Atualizar cor baseada no time
    local function updateColor()
        if ESPObjects[player] and ESPObjects[player].Box and ESPObjects[player].Label then
            local color = GetTeamColor(player)
            ESPObjects[player].Box.Color3 = color
            ESPObjects[player].Label.TextLabel.TextColor3 = color
            
            -- Atualizar texto do HP
            if humanoid and humanoidRootPart then
                ESPObjects[player].Label.TextLabel.Text = player.Name .. "
[" .. math.floor(humanoid.Health) .. " HP]"
            end
        end
    end
    
    -- Conectar para atualizações
    ESPObjects[player].Connection = RunService.Heartbeat:Connect(updateColor)
end

-- Função para obter cor baseada no time
function GetTeamColor(player)
    if TeamCheckEnabled and LocalPlayer.Team and player.Team then
        if LocalPlayer.Team == player.Team then
            return Color3.fromRGB(0, 255, 0) -- Verde para aliados
        else
            return Color3.fromRGB(255, 0, 0) -- Vermelho para inimigos
        end
    else
        return Color3.fromRGB(0, 120, 255) -- Azul padrão
    end
end

-- Função para remover ESP de um jogador
function RemoveESP(player)
    if ESPObjects[player] then
        if ESPObjects[[player].Box then
            ESPObjects[player].Box:Destroy()
        end
        if ESPObjects[player].Label then
            ESPObjects[player].Label:Destroy()
        end
        if ESPObjects[player].Connection then
            ESPObjects[player].Connection:Disconnect()
        end
        ESPObjects[player] = nil
    end
end

-- Função para limpar todos os ESPs
function ClearAllESP()
    for player, espData in pairs(ESPObjects) do
        RemoveESP(player)
    end
    ESPObjects = {}
end

-- Função para atualizar todos os ESPs
function UpdateAllESP()
    for player, espData in pairs(ESPObjects) do
        RemoveESP(player)
        if ESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            CreateESP(player)
        end
    end
end

-- Função para ativar/desativar ESP
function ToggleESP()
    ESPEnabled = not ESPEnabled
    
    if ESPEnabled then
        ESPToggle.Text = "Desativar ESP"
        ESPToggle.BackgroundColor3 = Color3.fromRGB(80, 180, 80)
        
        -- Criar ESP para todos os jogadores
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                CreateESP(player)
            end
        end
    else
        ESPToggle.Text = "Ativar ESP"
        ESPToggle.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        ClearAllESP()
    end
end

-- Função para ativar/desativar Team Check
function ToggleTeamCheck()
    TeamCheckEnabled = not TeamCheckEnabled
    
    if TeamCheckEnabled then
        TeamCheckToggle.Text = "Team Check: ON"
        TeamCheckToggle.TextColor3 = Color3.fromRGB(0, 255, 0)
    else
        TeamCheckToggle.Text = "Team Check: OFF"
        TeamCheckToggle.TextColor3 = Color3.fromRGB(255, 0, 0)
    end
    
    -- Atualizar cores dos ESPs existentes
    if ESPEnabled then
        UpdateAllESP()
    end
end

-- Conexões de eventos
ESPToggle.MouseButton1Click:Connect(ToggleESP)
TeamCheckToggle.MouseButton1Click:Connect(ToggleTeamCheck)

-- Quando um jogador entra no jogo
Players.PlayerAdded:Connect(function(player)
    if ESPEnabled then
        player.CharacterAdded:Connect(function(character)
            wait(1) -- Esperar o personagem carregar completamente
            CreateESP(player)
        end)
    end
end)

-- Quando um jogador sai do jogo
Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
end)

-- Quando o personagem local muda
LocalPlayer.CharacterAdded:Connect(function(character)
    if ESPEnabled then
        -- Recriar ESPs após respawn
        wait(2)
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                CreateESP(player)
            end
        end
    end
end)

-- Efeitos de hover nos botões
local function AddHoverEffect(button)
    local originalColor = button.BackgroundColor3
    
    button.MouseEnter:Connect(function()
        local tween = TweenService:Create(
            button,
            TweenInfo.new(0.2),
            {BackgroundColor3 = Color3.fromRGB(80, 80, 100)}
        )
        tween:Play()
    end)
    
    button.MouseLeave:Connect(function()
        local tween = TweenService:Create(
            button,
            TweenInfo.new(0.2),
            {BackgroundColor3 = originalColor}
        )
        tween:Play()
    end)
end

AddHoverEffect(ESPToggle)
AddHoverEffect(TeamCheckToggle)

-- Mensagem de inicialização
print("ESP System carregado com sucesso!")
print("Desenvolvido por forwalkin(victor)")
print("Use a interface para ativar/desativar o ESP e Team Check")
