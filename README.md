local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "MenuPortatilESP"

-- Variáveis de controle
local espPlayers = {}
local moveTimestamps = {}
local espEnabled = true
local monitorEnabled = false

-- Função para criar ESP vermelho
local function createESP(p)
    if p.Character and p.Character:FindFirstChild("Head") and not p.Character.Head:FindFirstChild("ESP_Red") then
        local esp = Instance.new("BillboardGui", p.Character.Head)
        esp.Name = "ESP_Red"
        esp.Size = UDim2.new(0, 120, 0, 30)
        esp.Adornee = p.Character.Head
        esp.AlwaysOnTop = true

        local nameLabel = Instance.new("TextLabel", esp)
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = p.Name
        nameLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        nameLabel.Font = Enum.Font.SourceSansBold
        nameLabel.TextSize = 18
        nameLabel.TextStrokeTransparency = 0
    end
end

local function removeESP(p)
    if p.Character and p.Character:FindFirstChild("Head") then
        local esp = p.Character.Head:FindFirstChild("ESP_Red")
        if esp then esp:Destroy() end
    end
end

-- Criação da interface principal
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 180)
frame.Position = UDim2.new(0.5, -150, 0.5, -90)
frame.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
frame.BorderSizePixel = 4
frame.BorderColor3 = Color3.fromRGB(255, 0, 0)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "Monitor ESP Movimento"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 24

local monitorBtn = Instance.new("TextButton", frame)
monitorBtn.Size = UDim2.new(0.9, 0, 0, 40)
monitorBtn.Position = UDim2.new(0.05, 0, 0, 50)
monitorBtn.Text = "Monitorar Movimento"
monitorBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)
monitorBtn.TextColor3 = Color3.fromRGB(255,255,255)
monitorBtn.Font = Enum.Font.SourceSansBold
monitorBtn.TextSize = 20

local unmarkBtn = Instance.new("TextButton", frame)
unmarkBtn.Size = UDim2.new(0.9, 0, 0, 40)
unmarkBtn.Position = UDim2.new(0.05, 0, 0, 100)
unmarkBtn.Text = "Desmarcar Todos"
unmarkBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)
unmarkBtn.TextColor3 = Color3.fromRGB(255,255,255)
unmarkBtn.Font = Enum.Font.SourceSansBold
unmarkBtn.TextSize = 20

local closeBtn = Instance.new("TextButton", frame)
closeBtn.Size = UDim2.new(0.9, 0, 0, 30)
closeBtn.Position = UDim2.new(0.05, 0, 0, 150)
closeBtn.Text = "Fechar Menu"
closeBtn.BackgroundColor3 = Color3.fromRGB(200,0,0)
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 18

-- Ícone portátil
local miniBtn = Instance.new("TextButton")
miniBtn.Size = UDim2.new(0, 60, 0, 60)
miniBtn.Position = UDim2.new(0.04, 0, 0.8, 0)
miniBtn.Text = "ESP"
miniBtn.BackgroundColor3 = Color3.fromRGB(200,0,0)
miniBtn.TextColor3 = Color3.fromRGB(255,255,255)
miniBtn.Font = Enum.Font.SourceSansBold
miniBtn.TextSize = 28
miniBtn.Visible = false
miniBtn.Active = true
miniBtn.Draggable = true
miniBtn.Parent = gui

-- Função de monitoramento
local function monitorMovimento()
    if monitorEnabled then return end
    monitorEnabled = true
    monitorBtn.Text = "Monitorando..."
    game:GetService("RunService").Heartbeat:Connect(function()
        if not espEnabled then return end
        for _, p in ipairs(game.Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local pos = p.Character.HumanoidRootPart.Position
                -- Verifica se se moveu
                if moveTimestamps[p.Name] then
                    if (moveTimestamps[p.Name].lastPos - pos).magnitude > 0.5 then
                        moveTimestamps[p.Name].lastTime = tick()
                        moveTimestamps[p.Name].lastPos = pos
                        createESP(p)
                        espPlayers[p.Name] = true
                    elseif tick() - moveTimestamps[p.Name].lastTime > 30 then
                        removeESP(p)
                        espPlayers[p.Name] = nil
                    end
                else
                    moveTimestamps[p.Name] = {lastPos = pos, lastTime = tick()}
                end
            end
        end
    end)
end

-- Botão Monitorar Movimento
monitorBtn.MouseButton1Click:Connect(monitorMovimento)

-- Botão Desmarcar Todos
unmarkBtn.MouseButton1Click:Connect(function()
    for _, p in ipairs(game.Players:GetPlayers()) do
        removeESP(p)
    end
    espPlayers = {}
end)

-- Botão Fechar Menu
closeBtn.MouseButton1Click:Connect(function()
    frame.Visible = false
    miniBtn.Visible = true
end)

-- Botão do menu portátil
miniBtn.MouseButton1Click:Connect(function()
    frame.Visible = true
    miniBtn.Visible = false
end)

-- Permite arrastar o mini menu
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    miniBtn.Position = UDim2.new(miniBtn.Position.X.Scale, startPos.X.Offset + delta.X, miniBtn.Position.Y.Scale, startPos.Y.Offset + delta.Y)
end

miniBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = miniBtn.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

miniBtn.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        update(input)
    end
end)
