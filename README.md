--// ESP inimigo com Highlight e menu branco de ativação/desativação, arrastável e persistente
-- Somente para uso educativo no Roblox Studio

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")

local ESP_Enabled = false

-- MENU BRANCO (Arrastável)
local gui = Instance.new("ScreenGui")
gui.Name = "ESPMenu"
gui.ResetOnSpawn = false -- Persiste entre respawns!
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 90)
frame.Position = UDim2.new(0, 12, 0, 12)
frame.BackgroundColor3 = Color3.new(1,1,1)
frame.BackgroundTransparency = 0.05
frame.Parent = gui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0.18,0)

local title = Instance.new("TextLabel")
title.Parent = frame
title.Size = UDim2.new(1, 0, 0.4, 0)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Text = "ESP"
title.TextScaled = true
title.Font = Enum.Font.SourceSansBold
title.TextColor3 = Color3.new(0.1,0.1,0.1)

local btnToggle = Instance.new("TextButton")
btnToggle.Parent = frame
btnToggle.Size = UDim2.new(0.8, 0, 0.35, 0)
btnToggle.Position = UDim2.new(0.1, 0, 0.5, 0)
btnToggle.BackgroundColor3 = Color3.fromRGB(220,220,220)
btnToggle.TextColor3 = Color3.new(0.2,0.2,0.2)
btnToggle.TextScaled = true
btnToggle.Font = Enum.Font.SourceSansBold
btnToggle.Text = "Ativar ESP"
Instance.new("UICorner", btnToggle).CornerRadius = UDim.new(0.4,0)

-- Arrastar menu
local dragging = false
local dragStart = nil
local startPos = nil

frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

-- Função para ativar/desativar ESP nos inimigos
local function updateAllESP()
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character then
			local esp = player.Character:FindFirstChild("EnemyESP")
			if esp then
				esp.Enabled = ESP_Enabled and (player.Team ~= LocalPlayer.Team)
			end
		end
	end
end

btnToggle.MouseButton1Click:Connect(function()
	ESP_Enabled = not ESP_Enabled
	btnToggle.Text = ESP_Enabled and "Desativar ESP" or "Ativar ESP"
	updateAllESP()
end)

-- Função para criar e atualizar ESP Highlight
local function applyHighlight(player, character)
	if player == LocalPlayer then return end
	local highlight = character:FindFirstChild("EnemyESP")
	if not highlight then
		highlight = Instance.new("Highlight")
		highlight.Name = "EnemyESP"
		highlight.FillTransparency = 1
		highlight.OutlineTransparency = 0
		highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
		highlight.Parent = character
	end
	highlight.Enabled = ESP_Enabled and (player.Team ~= LocalPlayer.Team)
end

-- Monitorar todos os jogadores e reaplicar ESP quando respawnam
local function onPlayer(player)
	if player == LocalPlayer then return end
	player.CharacterAdded:Connect(function(character)
		applyHighlight(player, character)
	end)
	-- Caso já esteja com personagem ao entrar no jogo
	if player.Character then
		applyHighlight(player, player.Character)
	end
	player:GetPropertyChangedSignal("Team"):Connect(function()
		if player.Character then
			applyHighlight(player, player.Character)
		end
	end)
end

-- Aplica a todos os jogadores atuais
for _, player in pairs(Players:GetPlayers()) do
	onPlayer(player)
end

-- Quando novos jogadores entrarem
Players.PlayerAdded:Connect(onPlayer)

-- Atualiza ESPs se você mudar de time
LocalPlayer:GetPropertyChangedSignal("Team"):Connect(function()
	updateAllESP()
end)

-- Se algum inimigo respawnar, garanta que o ESP aparece
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function(character)
		applyHighlight(player, character)
	end)
end)

-- Também garanta persistência do menu entre respawns do seu personagem
gui.ResetOnSpawn = false
