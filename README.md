--// ESP vermelho para jogadores inimigos (somente para uso educativo no Roblox Studio)

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")

-- Função para criar o ESP
local function createESP(player)
	if player == LocalPlayer then return end
	
	local highlight = Instance.new("Highlight")
	highlight.FillTransparency = 1
	highlight.OutlineTransparency = 0
	highlight.OutlineColor = Color3.fromRGB(255, 0, 0) -- Vermelho
	highlight.Name = "EnemyESP"
	
	-- Coloca o ESP no personagem
	player.CharacterAdded:Connect(function(character)
		if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then
			highlight.Enabled = false
		else
			highlight.Enabled = true
		end
		highlight.Parent = character
	end)
	
	-- Se já tiver personagem
	if player.Character then
		if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then
			highlight.Enabled = false
		else
			highlight.Enabled = true
		end
		highlight.Parent = player.Character
	end
end

-- Aplica a todos os jogadores atuais
for _, player in pairs(Players:GetPlayers()) do
	createESP(player)
end

-- Quando novos jogadores entrarem
Players.PlayerAdded:Connect(function(player)
	createESP(player)
end)
