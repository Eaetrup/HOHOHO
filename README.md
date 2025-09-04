-- UpWalk com botão arrastável
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local UIS = game:GetService("UserInputService")

-- CONFIGURAÇÕES
local plataformaAltura = 90
local plataformaVel = 10
local plataformaTamanho = Vector3.new(8, 1, 8)
local plataformaCor = Color3.fromRGB(0, 170, 255)

-- VARIÁVEIS
local plataforma = nil
local conexaoPlataforma = nil
local plataformaAtiva = false

-- CRIAR GUI COM BOTÃO
local screenGui = Instance.new("ScreenGui")
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local btnPlataforma = Instance.new("TextButton")
btnPlataforma.Size = UDim2.new(0, 200, 0, 60)
btnPlataforma.Position = UDim2.new(0.5, -100, 0.8, 0)
btnPlataforma.Text = "Ativar UpWalk"
btnPlataforma.Font = Enum.Font.SourceSansBold
btnPlataforma.TextSize = 22
btnPlataforma.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
btnPlataforma.TextColor3 = Color3.new(1, 1, 1)
btnPlataforma.Parent = screenGui

-- FUNÇÃO PARA INICIAR PLATAFORMA
local function iniciarPlataforma()
	if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
	local hrp = player.Character.HumanoidRootPart

	if plataforma then
		plataforma:Destroy()
		plataforma = nil
	end
	if conexaoPlataforma then
		conexaoPlataforma:Disconnect()
		conexaoPlataforma = nil
	end

	plataforma = Instance.new("Part")
	plataforma.Size = plataformaTamanho
	plataforma.Anchored = true
	plataforma.CanCollide = true
	plataforma.Color = plataformaCor
	plataforma.Material = Enum.Material.Neon
	plataforma.Transparency = 0.2
	plataforma.CFrame = CFrame.new(hrp.Position.X, hrp.Position.Y - 3, hrp.Position.Z)
	plataforma.Parent = workspace

	local alturaFinal = plataforma.Position.Y + plataformaAltura

	conexaoPlataforma = RunService.RenderStepped:Connect(function(dt)
		if not plataforma or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
		local hrp = player.Character.HumanoidRootPart
		local novaPos = Vector3.new(hrp.Position.X, plataforma.Position.Y, hrp.Position.Z)
		plataforma.CFrame = CFrame.new(novaPos)

		if plataforma.Position.Y < alturaFinal then
			plataforma.CFrame = plataforma.CFrame + Vector3.new(0, plataformaVel * dt, 0)
		end
	end)
end

-- CLIQUE NO BOTÃO
btnPlataforma.MouseButton1Click:Connect(function()
	if plataformaAtiva then
		if plataforma then
			plataforma:Destroy()
			plataforma = nil
		end
		if conexaoPlataforma then
			conexaoPlataforma:Disconnect()
			conexaoPlataforma = nil
		end
		plataformaAtiva = false
		btnPlataforma.Text = "Ativar UpWalk"
		btnPlataforma.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
	else
		iniciarPlataforma()
		plataformaAtiva = true
		btnPlataforma.Text = "Resetar UpWalk"
		btnPlataforma.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
	end
end)

-- =========================
-- LÓGICA DE ARRASTAR BOTÃO
-- =========================
local dragging = false
local dragInput, mousePos, framePos

btnPlataforma.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		mousePos = input.Position
		framePos = btnPlataforma.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

btnPlataforma.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement then
		dragInput = input
	end
end)

RunService.RenderStepped:Connect(function()
	if dragging and dragInput then
		local delta = dragInput.Position - mousePos
		btnPlataforma.Position = UDim2.new(
			framePos.X.Scale,
			framePos.X.Offset + delta.X,
			framePos.Y.Scale,
			framePos.Y.Offset + delta.Y
		)
	end
end)
