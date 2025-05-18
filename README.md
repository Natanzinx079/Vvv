-- Coloque este script em StarterGui (LocalScript)
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "NatHub"

-- Janela Principal
local mainFrame = Instance.new("Frame", gui)
mainFrame.Size = UDim2.new(0, 400, 0, 300)
mainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true

-- Título
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, -60, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
title.Text = "NatHub | Dead Rails (0.3.1)"
title.Font = Enum.Font.SourceSansBold
title.TextSize = 20
title.TextColor3 = Color3.new(1, 1, 1)
title.TextXAlignment = Enum.TextXAlignment.Left
title.PaddingLeft = UDim.new(0, 10)

-- Botão fechar (X)
local closeButton = Instance.new("TextButton", mainFrame)
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0, 5)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.new(1, 0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
closeButton.MouseButton1Click:Connect(function()
    gui:Destroy()
end)

-- Botão minimizar (-)
local minimizeButton = Instance.new("TextButton", mainFrame)
minimizeButton.Size = UDim2.new(0, 30, 0, 30)
minimizeButton.Position = UDim2.new(1, -70, 0, 5)
minimizeButton.Text = "-"
minimizeButton.TextColor3 = Color3.new(1, 1, 0)
minimizeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

local minimized = false
minimizeButton.MouseButton1Click:Connect(function()
    minimized = not minimized
    for _, child in pairs(mainFrame:GetChildren()) do
        if child ~= title and child ~= closeButton and child ~= minimizeButton then
            child.Visible = not minimized
        end
    end
end)

-- Abas laterais
local tabs = {"Main", "Character", "Teleport", "Visual", "Combat", "Configuration"}
local tabButtons = {}
local contentFrames = {}

for i, tabName in ipairs(tabs) do
    local button = Instance.new("TextButton", mainFrame)
    button.Size = UDim2.new(0, 120, 0, 30)
    button.Position = UDim2.new(0, 0, 0, 40 + (i - 1) * 32)
    button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    button.Text = tabName
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Font = Enum.Font.SourceSans
    button.TextSize = 16
    tabButtons[tabName] = button

    local frame = Instance.new("Frame", mainFrame)
    frame.Name = tabName
    frame.Position = UDim2.new(0, 130, 0, 40)
    frame.Size = UDim2.new(1, -130, 1, -40)
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    frame.Visible = false
    contentFrames[tabName] = frame

    button.MouseButton1Click:Connect(function()
        for name, frame in pairs(contentFrames) do
            frame.Visible = (name == tabName)
        end
    end)
end

-- Ativar aba "Main" por padrão
contentFrames["Main"].Visible = true

--------------------
-- Conteúdo da Main
--------------------
local mainTab = contentFrames["Main"]

-- Webhook Box
local webhookBox = Instance.new("TextBox", mainTab)
webhookBox.Size = UDim2.new(0, 220, 0, 30)
webhookBox.Position = UDim2.new(0, 10, 0, 10)
webhookBox.PlaceholderText = "Your Webhook Link"
webhookBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
webhookBox.TextColor3 = Color3.new(1, 1, 1)

-- Auto Bond
local autoBond = Instance.new("TextButton", mainTab)
autoBond.Size = UDim2.new(0, 180, 0, 30)
autoBond.Position = UDim2.new(0, 10, 0, 50)
autoBond.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
autoBond.Text = "Auto Bond: OFF"
autoBond.TextColor3 = Color3.new(1, 1, 1)

local bondActive = false
autoBond.MouseButton1Click:Connect(function()
    bondActive = not bondActive
    autoBond.Text = bondActive and "Auto Bond: ON" or "Auto Bond: OFF"

    if bondActive then
        task.spawn(function()
            while bondActive do
                -- Verifica e coleta barras de ouro próximas
                local items = workspace:FindFirstChild("Bonds") or workspace:FindFirstChild("Gold")
                if items then
                    for _, item in pairs(items:GetChildren()) do
                        if item:IsA("BasePart") then
                            character:PivotTo(item.CFrame + Vector3.new(0, 2, 0))
                            wait(0.3)
                        end
                    end
                end
                wait(1)
            end
        end)
    end
end)

-- Auto Win Simulado
local autoWin = Instance.new("TextButton", mainTab)
autoWin.Size = UDim2.new(0, 180, 0, 30)
autoWin.Position = UDim2.new(0, 10, 0, 90)
autoWin.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
autoWin.Text = "Auto Win"
autoWin.TextColor3 = Color3.new(1, 1, 1)
autoWin.MouseButton1Click:Connect(function()
    -- Exemplo: Teleporta para local de vitória
    local victoryPos = workspace:FindFirstChild("SafeZone") or workspace:FindFirstChild("EndZone")
    if victoryPos then
        character:PivotTo(victoryPos.CFrame + Vector3.new(0, 3, 0))
    else
        warn("Zona de vitória não encontrada.")
    end
end)

-------------------------
-- Aba Teleport
-------------------------
local teleportTab = contentFrames["Teleport"]

local locations = {
    ["Spawn"] = Vector3.new(0, 5, 0),
    ["Estação"] = Vector3.new(100, 5, -50),
    ["Túnel"] = Vector3.new(-120, 5, 200),
    ["Topo da Torre"] = Vector3.new(0, 100, 0),
}

local y = 10
for name, pos in pairs(locations) do
    local tpButton = Instance.new("TextButton", teleportTab)
    tpButton.Size = UDim2.new(0, 200, 0, 30)
    tpButton.Position = UDim2.new(0, 10, 0, y)
    tpButton.Text = "Ir para " .. name
    tpButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    tpButton.TextColor3 = Color3.new(1, 1, 1)
    tpButton.MouseButton1Click:Connect(function()
        character:PivotTo(CFrame.new(pos))
    end)
    y = y + 40
end
