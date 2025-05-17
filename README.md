-- Aimbot + ESP - Roblox Local Script (StarterPlayerScripts)
-- Para uso em testes locais com permissão no Roblox Studio

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local aiming = false

-- CONFIG
local ESP_COLOR = Color3.fromRGB(255, 0, 0)
local AIM_RANGE = 1000 -- distância máxima do aimbot

-- Função para criar ESP
function createESP(player)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and not player:FindFirstChild("ESP") then
		local box = Drawing.new("Square")
		box.Thickness = 1
		box.Color = ESP_COLOR
		box.Transparency = 1
		box.Filled = false

		local nameTag = Drawing.new("Text")
		nameTag.Size = 13
		nameTag.Center = true
		nameTag.Outline = true
		nameTag.Color = ESP_COLOR

		local folder = Instance.new("Folder", player)
		folder.Name = "ESP"
		folder:SetAttribute("Box", box)
		folder:SetAttribute("Text", nameTag)
	end
end

-- Atualizar ESP
RunService.RenderStepped:Connect(function()
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer then
			if not player:FindFirstChild("ESP") then
				createESP(player)
			end

			local espFolder = player:FindFirstChild("ESP")
			if espFolder and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
				local hrp = player.Character.HumanoidRootPart
				local pos, onScreen = camera:WorldToViewportPoint(hrp.Position)

				local box = espFolder:GetAttribute("Box")
				local text = espFolder:GetAttribute("Text")

				if onScreen then
					local size = 60 / (hrp.Position - camera.CFrame.Position).Magnitude * 100
					box.Size = Vector2.new(size, size * 1.5)
					box.Position = Vector2.new(pos.X - size / 2, pos.Y - size * 0.75)
					box.Visible = true

					text.Text = player.Name
					text.Position = Vector2.new(pos.X, pos.Y - size)
					text.Visible = true
				else
					box.Visible = false
					text.Visible = false
				end
			end
		end
	end
end)

-- Aimbot lógica
function getClosestTarget()
	local closestPlayer = nil
	local shortestDistance = AIM_RANGE

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local pos, onScreen = camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
				if dist < shortestDistance then
					closestPlayer = player
					shortestDistance = dist
				end
			end
		end
	end

	return closestPlayer
end

-- Mira automática
RunService.RenderStepped:Connect(function()
	if aiming then
		local target = getClosestTarget()
		if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
			local pos = target.Character.HumanoidRootPart.Position
			camera.CFrame = CFrame.new(camera.CFrame.Position, pos)
		end
	end
end)

-- Input da tecla E
UserInputService.InputBegan:Connect(function(input, processed)
	if not processed and input.KeyCode == Enum.KeyCode.E then
		aiming = true
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.E then
		aiming = false
	end
end)

print("✅ Aimbot + ESP ativado.")
