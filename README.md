local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local RunService = game:GetService("RunService")

local flying = false
local speed = 50
local killAuraOn = false

-- GUI
local gui = Instance.new("ScreenGui", plr:WaitForChild("PlayerGui"))
gui.Name = "FlyAuraGUI"

-- Botões
local toggleFly = Instance.new("TextButton", gui)
toggleFly.Size = UDim2.new(0, 120, 0, 40)
toggleFly.Position = UDim2.new(0.05, 0, 0.7, 0)
toggleFly.Text = "Ativar Fly"
toggleFly.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
toggleFly.TextColor3 = Color3.new(1,1,1)
toggleFly.Font = Enum.Font.SourceSansBold
toggleFly.TextScaled = true
toggleFly.BorderSizePixel = 0

local toggleAura = Instance.new("TextButton", gui)
toggleAura.Size = UDim2.new(0, 120, 0, 40)
toggleAura.Position = UDim2.new(0.05, 0, 0.76, 0)
toggleAura.Text = "Ativar Kill Aura"
toggleAura.BackgroundColor3 = Color3.fromRGB(255, 70, 70)
toggleAura.TextColor3 = Color3.new(1,1,1)
toggleAura.Font = Enum.Font.SourceSansBold
toggleAura.TextScaled = true
toggleAura.BorderSizePixel = 0

-- Velocidade
local speedLabel = Instance.new("TextLabel", gui)
speedLabel.Size = UDim2.new(0, 100, 0, 30)
speedLabel.Position = UDim2.new(0.05, 0, 0.65, 0)
speedLabel.Text = "Velocidade: " .. speed
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.Font = Enum.Font.SourceSans
speedLabel.TextScaled = true

local btnMais = Instance.new("TextButton", gui)
btnMais.Size = UDim2.new(0, 40, 0, 40)
btnMais.Position = UDim2.new(0.05, 130, 0.7, 0)
btnMais.Text = "+"
btnMais.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
btnMais.TextColor3 = Color3.new(1,1,1)
btnMais.Font = Enum.Font.SourceSansBold
btnMais.TextScaled = true

local btnMenos = Instance.new("TextButton", gui)
btnMenos.Size = UDim2.new(0, 40, 0, 40)
btnMenos.Position = UDim2.new(0.05, 180, 0.7, 0)
btnMenos.Text = "-"
btnMenos.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
btnMenos.TextColor3 = Color3.new(1,1,1)
btnMenos.Font = Enum.Font.SourceSansBold
btnMenos.TextScaled = true

-- Fly física
local bodyGyro = Instance.new("BodyGyro")
bodyGyro.P = 9e4
bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)

local bodyVelocity = Instance.new("BodyVelocity")
bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

-- Fly controles
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

toggleFly.MouseButton1Click:Connect(function()
	if flying then stopFly() else startFly() end
end)

btnMais.MouseButton1Click:Connect(function()
	speed += 10
	speedLabel.Text = "Velocidade: " .. speed
end)

btnMenos.MouseButton1Click:Connect(function()
	speed = math.max(10, speed - 10)
	speedLabel.Text = "Velocidade: " .. speed
end)

toggleAura.MouseButton1Click:Connect(function()
	killAuraOn = not killAuraOn
	toggleAura.Text = killAuraOn and "Desativar Kill Aura" or "Ativar Kill Aura"
end)

-- NPC Detection
local function isNPC(model)
	return model:FindFirstChild("Humanoid") and not game.Players:GetPlayerFromCharacter(model)
end

-- Visual Highlight
local function addHighlight(target)
	if target:FindFirstChild("HighlightAura") then return end
	local h = Instance.new("Highlight", target)
	h.Name = "HighlightAura"
	h.FillColor = Color3.fromRGB(255, 0, 0)
	h.OutlineColor = Color3.fromRGB(255, 255, 255)
	h.FillTransparency = 0.5
	h.OutlineTransparency = 0
end

local function removeHighlight(target)
	local h = target:FindFirstChild("HighlightAura")
	if h then h:Destroy() end
end

-- Loop
RunService.RenderStepped:Connect(function()
	if flying then
		local cam = workspace.CurrentCamera
		bodyGyro.CFrame = cam.CFrame
		bodyVelocity.Velocity = cam.CFrame.LookVector * speed
	end

	if killAuraOn then
		local tool = char:FindFirstChildOfClass("Tool")
		if tool and tool:FindFirstChild("Handle") then
			for _, npc in ipairs(workspace:GetDescendants()) do
				if npc:IsA("Model") and isNPC(npc) and npc:FindFirstChild("HumanoidRootPart") then
					local dist = (npc.HumanoidRootPart.Position - hrp.Position).Magnitude
					if dist < 10 then
						addHighlight(npc)
						firetouchinterest(tool.Handle, npc.HumanoidRootPart, 0)
						firetouchinterest(tool.Handle, npc.HumanoidRootPart, 1)
					else
						removeHighlight(npc)
					end
				end
			end
		end
	else
		for _, npc in ipairs(workspace:GetDescendants()) do
			if npc:IsA("Model") and npc:FindFirstChild("HighlightAura") then
				removeHighlight(npc)
			end
		end
	end
end)
