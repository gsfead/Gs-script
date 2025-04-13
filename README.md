-- Fly Script com Controle de Velocidade (Mobile) - Roblox

local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50

local RunService = game:GetService("RunService")

-- Criar GUI
local gui = Instance.new("ScreenGui", plr:WaitForChild("PlayerGui"))
gui.Name = "FlyGUI"

-- Botão principal (Ativar/Desativar Fly)
local toggleBtn = Instance.new("TextButton", gui)
toggleBtn.Size = UDim2.new(0, 120, 0, 40)
toggleBtn.Position = UDim2.new(0.05, 0, 0.75, 0)
toggleBtn.Text = "Ativar Fly"
toggleBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.TextScaled = true
toggleBtn.BorderSizePixel = 0
toggleBtn.Draggable = true

-- Botão aumentar velocidade
local speedUpBtn = Instance.new("TextButton", gui)
speedUpBtn.Size = UDim2.new(0, 40, 0, 40)
speedUpBtn.Position = UDim2.new(0.05, 130, 0.75, 0)
speedUpBtn.Text = "+"
speedUpBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
speedUpBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedUpBtn.Font = Enum.Font.SourceSansBold
speedUpBtn.TextScaled = true
speedUpBtn.BorderSizePixel = 0

-- Botão diminuir velocidade
local speedDownBtn = Instance.new("TextButton", gui)
speedDownBtn.Size = UDim2.new(0, 40, 0, 40)
speedDownBtn.Position = UDim2.new(0.05, 180, 0.75, 0)
speedDownBtn.Text = "-"
speedDownBtn.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
speedDownBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
speedDownBtn.Font = Enum.Font.SourceSansBold
speedDownBtn.TextScaled = true
speedDownBtn.BorderSizePixel = 0

-- Label de velocidade
local speedLabel = Instance.new("TextLabel", gui)
speedLabel.Size = UDim2.new(0, 100, 0, 30)
speedLabel.Position = UDim2.new(0.05, 0, 0.7, 0)
speedLabel.Text = "Velocidade: " .. speed
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.Font = Enum.Font.SourceSans
speedLabel.TextScaled = true

-- Fly System
local bodyGyro = Instance.new("BodyGyro")
bodyGyro.P = 9e4
bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)

local bodyVelocity = Instance.new("BodyVelocity")
bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

local function startFly()
	bodyGyro.Parent = hrp
	bodyVelocity.Parent = hrp
	flying = true
	toggleBtn.Text = "Desativar Fly"
end

local function stopFly()
	bodyGyro:Destroy()
	bodyVelocity:Destroy()
	flying = false
	toggleBtn.Text = "Ativar Fly"
end

toggleBtn.MouseButton1Click:Connect(function()
	if flying then
		stopFly()
	else
		startFly()
	end
end)

speedUpBtn.MouseButton1Click:Connect(function()
	speed = speed + 10
	speedLabel.Text = "Velocidade: " .. speed
end)

speedDownBtn.MouseButton1Click:Connect(function()
	speed = math.max(10, speed - 10)
	speedLabel.Text = "Velocidade: " .. speed
end)

RunService.RenderStepped:Connect(function()
	if flying then
		local cam = workspace.CurrentCamera
		bodyGyro.CFrame = cam.CFrame
		bodyVelocity.Velocity = cam.CFrame.LookVector * speed
	end
end)
