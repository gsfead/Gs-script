local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

-- VARIÁVEIS GLOBAIS
getgenv().flySpeed = getgenv().flySpeed or 50
getgenv().killAura = false
getgenv().auraRange = getgenv().auraRange or 15

local flying = false
local speed = getgenv().flySpeed
local auraRange = getgenv().auraRange
local maxSafeSpeed = 16
local attacked = {}

-- GUI SETUP
local gui = Instance.new("ScreenGui", plr:WaitForChild("PlayerGui"))
gui.Name = "UltimateFlyKillAura"

local function createButton(text, pos, color)
	local btn = Instance.new("TextButton", gui)
	btn.Size = UDim2.new(0, 140, 0, 40)
	btn.Position = pos
	btn.Text = text
	btn.BackgroundColor3 = color
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.SourceSansBold
	btn.TextScaled = true
	btn.BorderSizePixel = 0
	return btn
end

-- Botões
local toggleFly = createButton("Ativar Fly", UDim2.new(0.05, 0, 0.65, 0), Color3.fromRGB(0, 170, 255))
local toggleAura = createButton("Kill Aura: OFF", UDim2.new(0.05, 0, 0.71, 0), Color3.fromRGB(170, 0, 0))
local increaseSpeed = createButton("Velocidade +", UDim2.new(0.05, 0, 0.77, 0), Color3.fromRGB(0, 200, 0))
local decreaseSpeed = createButton("Velocidade -", UDim2.new(0.05, 0, 0.83, 0), Color3.fromRGB(200, 0, 0))
local increaseRange = createButton("Raio +", UDim2.new(0.22, 0, 0.77, 0), Color3.fromRGB(0, 120, 255))
local decreaseRange = createButton("Raio -", UDim2.new(0.22, 0, 0.83, 0), Color3.fromRGB(0, 60, 200))
local tpNearest = createButton("TP NPC + Próx", UDim2.new(0.05, 0, 0.89, 0), Color3.fromRGB(255, 128, 0))
local tpToClick = createButton("TP ao Clicar", UDim2.new(0.22, 0, 0.89, 0), Color3.fromRGB(255, 85, 0))

-- Labels
local statusLabel = Instance.new("TextLabel", gui)
statusLabel.Size = UDim2.new(0, 250, 0, 30)
statusLabel.Position = UDim2.new(0.05, 0, 0.6, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = Color3.new(1,1,1)
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextScaled = true
statusLabel.Text = "Vel: " .. speed .. " | Raio: " .. auraRange

-- FLY CONFIG
local bodyGyro = Instance.new("BodyGyro")
bodyGyro.P = 9e4
bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)

local bodyVelocity = Instance.new("BodyVelocity")
bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

local function startFly()
	bodyGyro.Parent = hrp
	bodyVelocity.Parent = hrp
	flying = true
	toggleFly.Text = "Desativar Fly"
end

local function stopFly()
	bodyGyro:Destroy()
	bodyVelocity:Destroy()
	flying = false
	toggleFly.Text = "Ativar Fly"
end

-- Kill Aura
local function isNPC(model)
	return model:IsA("Model") and model:FindFirstChild("Humanoid") and model ~= char and game.Players:GetPlayerFromCharacter(model) == nil
end

local function highlightTarget(model)
	if not model:FindFirstChild("Highlight") then
		local hl = Instance.new("Highlight", model)
		hl.FillColor = Color3.fromRGB(255, 0, 0)
		hl.OutlineColor = Color3.new(1, 1, 1)
	end
end

local function removeHighlight(model)
	local hl = model:FindFirstChild("Highlight")
	if hl then hl:Destroy() end
end

local function attack(npc)
	local hum = npc:FindFirstChild("Humanoid")
	if hum and hum.Health > 0 then
		hum:TakeDamage(10)
	end
end

-- Teleporte
local function tpToNearestNPC()
	local nearest, dist = nil, math.huge
	for _, obj in ipairs(workspace:GetDescendants()) do
		if isNPC(obj) and obj:FindFirstChild("HumanoidRootPart") then
			local d = (hrp.Position - obj.HumanoidRootPart.Position).Magnitude
			if d < dist then
				dist = d
				nearest = obj
			end
		end
	end
	if nearest then
		hrp.CFrame = nearest.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)
	end
end

local function tpToMouse()
	local mouse = plr:GetMouse()
	local pos = mouse.Hit and mouse.Hit.Position
	if pos then
		hrp.CFrame = CFrame.new(pos + Vector3.new(0, 3, 0))
	end
end

-- Botões
toggleFly.MouseButton1Click:Connect(function()
	if flying then stopFly() else startFly() end
end)

toggleAura.MouseButton1Click:Connect(function()
	getgenv().killAura = not getgenv().killAura
	toggleAura.Text = "Kill Aura: " .. (getgenv().killAura and "ON" or "OFF")
	toggleAura.BackgroundColor3 = getgenv().killAura and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(170, 0, 0)
end)

increaseSpeed.MouseButton1Click:Connect(function()
	speed += 10
	getgenv().flySpeed = speed
	statusLabel.Text = "Vel: " .. speed .. " | Raio: " .. auraRange
end)

decreaseSpeed.MouseButton1Click:Connect(function()
	speed = math.max(10, speed - 10)
	getgenv().flySpeed = speed
	statusLabel.Text = "Vel: " .. speed .. " | Raio: " .. auraRange
end)

increaseRange.MouseButton1Click:Connect(function()
	auraRange += 5
	getgenv().auraRange = auraRange
	statusLabel.Text = "Vel: " .. speed .. " | Raio: " .. auraRange
end)

decreaseRange.MouseButton1Click:Connect(function()
	auraRange = math.max(5, auraRange - 5)
	getgenv().auraRange = auraRange
	statusLabel.Text = "Vel: " .. speed .. " | Raio: " .. auraRange
end)

tpNearest.MouseButton1Click:Connect(tpToNearestNPC)
tpToClick.MouseButton1Click:Connect(tpToMouse)

-- Lista de TP seletivo
local tpFrame = Instance.new("Frame", gui)
tpFrame.Size = UDim2.new(0, 250, 0, 200)
tpFrame.Position = UDim2.new(0.5, -125, 0.6, 0)
tpFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
tpFrame.BorderSizePixel = 0

local uiList = Instance.new("UIListLayout", tpFrame)
uiList.Padding = UDim.new(0, 4)

local title = Instance.new("TextLabel", tpFrame)
title.Size = UDim2.new(1, 0, 0, 25)
title.Text = "TP para NPC ou Jogador"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
title.Font = Enum.Font.SourceSansBold
title.TextScaled = true
title.BorderSizePixel = 0

local function createTPButton(name, cf)
	local btn = Instance.new("TextButton", tpFrame)
	btn.Size = UDim2.new(1, 0, 0, 30)
	btn.Text = name
	btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Font = Enum.Font.SourceSans
	btn.TextScaled = true
	btn.BorderSizePixel = 0
	btn.MouseButton1Click:Connect(function()
		hrp.CFrame = cf + Vector3.new(0, 3, 0)
	end)
end

local function atualizarTPLista()
	for _, v in ipairs(tpFrame:GetChildren()) do
		if v:IsA("TextButton") then v:Destroy() end
	end

	for _, obj in ipairs(workspace:GetDescendants()) do
		if isNPC(obj) and obj:FindFirstChild("HumanoidRootPart") then
			createTPButton("[NPC] " .. obj.Name, obj.HumanoidRootPart.CFrame)
		end
	end

	for _, player in ipairs(game.Players:GetPlayers()) do
		if player ~= plr and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			createTPButton("[Player] " .. player.Name, player.Character.HumanoidRootPart.CFrame)
		end
	end
end

task.spawn(function()
	while true do
		atualizarTPLista()
		task.wait(10)
	end
end)

-- Loops
RunService.RenderStepped:Connect(function()
	if flying then
		local cam = workspace.CurrentCamera
		bodyGyro.CFrame = cam.CFrame
		local limitedSpeed = math.min(speed, maxSafeSpeed)
		bodyVelocity.Velocity = cam.CFrame.LookVector * limitedSpeed
	end

	if getgenv().killAura then
		for _, npc in ipairs(workspace:GetDescendants()) do
			if isNPC(npc) and npc:FindFirstChild("HumanoidRootPart") then
				local dist = (hrp.Position - npc.HumanoidRootPart.Position).Magnitude
				if dist <= auraRange then
					highlightTarget(npc)
					if not attacked[npc] or tick() - attacked[npc] > 1 then
						attack(npc)
						attacked[npc] = tick()
					end
				else
					removeHighlight(npc)
				end
			end
		end
	end
end)
