local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Cores
local RED = Color3.fromRGB(255, 40, 60)
local CYAN = Color3.fromRGB(0, 230, 255)
local DARK = Color3.fromRGB(12, 12, 20)
local PANEL = Color3.fromRGB(25, 25, 38)
local TEXT = Color3.fromRGB(235, 235, 245)

-- Configurações
local HIT_RANGE = 6          -- distância para contar como "encostou"
local HIT_COOLDOWN = 0.6     -- segundos antes de arremessar o mesmo jogador de novo
local MAX_SPEED = 450        -- limite de velocidade (evita sair do mapa)
local TOGGLE_KEY = Enum.KeyCode.F

local power = 50
local enabled = true

local rng = Random.new()
local lastHit = {}           -- [Player] = tick() do último arremesso
local connections = {}       -- todas as conexões, para limpar depois

local function connect(signal, fn)
	local c = signal:Connect(fn)
	table.insert(connections, c)
	return c
end

local function isPress(i)
	return i.UserInputType == Enum.UserInputType.MouseButton1
		or i.UserInputType == Enum.UserInputType.Touch
end

local function isMove(i)
	return i.UserInputType == Enum.UserInputType.MouseMovement
		or i.UserInputType == Enum.UserInputType.Touch
end

local function round(o, r)
	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, r or 8)
	c.Parent = o
end

local function stroke(o, c, t)
	local s = Instance.new("UIStroke")
	s.Color, s.Thickness = c, t or 1.5
	s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
	s.Parent = o
	return s
end

-- Limpa GUI antiga se houver
local old = playerGui:FindFirstChild("FlingGui")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "FlingGui"
gui.ResetOnSpawn = false
gui.Parent = playerGui

-- Quando a GUI é destruída, desconecta tudo (evita conexões duplicadas)
gui.Destroying:Connect(function()
	for _, c in ipairs(connections) do
		c:Disconnect()
	end
	table.clear(connections)
end)

-- Botão Flutuante (para reabrir o menu)
local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.fromOffset(45, 45)
openBtn.Position = UDim2.new(0, 20, 0.5, -130)
openBtn.BackgroundColor3 = DARK
openBtn.Text = "⚡"
openBtn.Font = Enum.Font.Michroma
openBtn.TextSize = 20
openBtn.TextColor3 = CYAN
openBtn.Visible = false
openBtn.Parent = gui
round(openBtn, 10)
stroke(openBtn, RED, 2)

-- Janela Principal
local window = Instance.new("Frame")
window.Size = UDim2.fromOffset(260, 0)
window.AutomaticSize = Enum.AutomaticSize.Y
window.Position = UDim2.new(0, 20, 0.5, -130)
window.BackgroundColor3 = DARK
window.BackgroundTransparency = 0.08
window.Parent = gui
round(window, 10)
stroke(window, RED, 2)

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 8)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = window

local pad = Instance.new("UIPadding")
pad.PaddingBottom = UDim.new(0, 12)
pad.Parent = window

-- Barra de Título (Arrastável + Botão Minimizar)
local bar = Instance.new("Frame")
bar.Size = UDim2.new(1, 0, 0, 36)
bar.BackgroundTransparency = 1
bar.LayoutOrder = 0
bar.Parent = window

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -40, 1, 0)
titleLabel.Position = UDim2.fromOffset(10, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚡ TOUCH FLING"
titleLabel.Font = Enum.Font.Michroma
titleLabel.TextSize = 13
titleLabel.TextColor3 = CYAN
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = bar

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.fromOffset(28, 28)
minBtn.Position = UDim2.new(1, -32, 0.5, -14)
minBtn.BackgroundColor3 = PANEL
minBtn.Text = "_"
minBtn.Font = Enum.Font.Michroma
minBtn.TextSize = 12
minBtn.TextColor3 = RED
minBtn.Parent = bar
round(minBtn, 6)
stroke(minBtn, RED)

connect(minBtn.MouseButton1Click, function()
	window.Visible = false
	openBtn.Visible = true
end)

connect(openBtn.MouseButton1Click, function()
	window.Visible = true
	openBtn.Visible = false
end)

-- Arrastar Menu (agora o botão flutuante também é arrastável)
local dragging, dragStart, startPos, dragTarget
local function beginDrag(target, i)
	dragging, dragStart, startPos, dragTarget = true, i.Position, target.Position, target
end

connect(bar.InputBegan, function(i)
	if isPress(i) then beginDrag(window, i) end
end)
connect(openBtn.InputBegan, function(i)
	if isPress(i) then beginDrag(openBtn, i) end
end)

connect(UserInputService.InputChanged, function(i)
	if dragging and isMove(i) then
		local d = i.Position - dragStart
		local newPos = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + d.X,
			startPos.Y.Scale, startPos.Y.Offset + d.Y
		)
		-- mantém janela e botão sempre no mesmo lugar
		window.Position = newPos
		openBtn.Position = newPos
	end
end)

connect(UserInputService.InputEnded, function(i)
	if isPress(i) then dragging = false end
end)

local function makeButton(text, order)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -30, 0, 30)
	btn.BackgroundColor3 = PANEL
	btn.Text = text
	btn.Font = Enum.Font.Michroma
	btn.TextSize = 12
	btn.TextColor3 = CYAN
	btn.AutoButtonColor = false
	btn.LayoutOrder = order
	btn.Parent = window
	round(btn, 6)
	local s = stroke(btn, CYAN)
	return btn, s
end

-- Slider de Força (1 a 100)
local sliderHolder = Instance.new("Frame")
sliderHolder.Size = UDim2.new(1, -30, 0, 44)
sliderHolder.BackgroundTransparency = 1
sliderHolder.LayoutOrder = 1
sliderHolder.Parent = window

local powerLabel = Instance.new("TextLabel")
powerLabel.Size = UDim2.new(1, 0, 0, 18)
powerLabel.BackgroundTransparency = 1
powerLabel.Font = Enum.Font.Code
powerLabel.TextSize = 12
powerLabel.TextColor3 = TEXT
powerLabel.TextXAlignment = Enum.TextXAlignment.Left
powerLabel.Parent = sliderHolder

local track = Instance.new("Frame")
track.Size = UDim2.new(1, 0, 0, 8)
track.Position = UDim2.fromOffset(0, 28)
track.BackgroundColor3 = PANEL
track.Parent = sliderHolder
round(track, 4)

local fill = Instance.new("Frame")
fill.BackgroundColor3 = RED
fill.Parent = track
round(fill, 4)

local function setPower(v)
	power = math.clamp(math.floor(v + 0.5), 1, 100)
	fill.Size = UDim2.new(power / 100, 0, 1, 0)
	powerLabel.Text = "FORÇA DO FLING: " .. power
end

-- Botão Liga/Desliga
local toggleBtn, toggleStroke = makeButton("STATUS: ON", 5)

local function applyToggleVisual()
	toggleBtn.Text = "STATUS: " .. (enabled and "ON" or "OFF")
	toggleBtn.TextColor3 = enabled and RED or CYAN
	toggleStroke.Color = enabled and RED or CYAN
end

local function toggle()
	enabled = not enabled
	applyToggleVisual()
end

connect(toggleBtn.MouseButton1Click, toggle)
applyToggleVisual()

-- Atalho de teclado (PC)
connect(UserInputService.InputBegan, function(i, processed)
	if processed then return end
	if i.KeyCode == TOGGLE_KEY then
		toggle()
	end
end)

local hint = Instance.new("TextLabel")
hint.Size = UDim2.new(1, -20, 0, 32)
hint.BackgroundTransparency = 1
hint.Text = "Quem encostar em você voa longe! (tecla F liga/desliga)"
hint.Font = Enum.Font.Code
hint.TextSize = 11
hint.TextColor3 = Color3.fromRGB(170, 170, 190)
hint.TextWrapped = true
hint.LayoutOrder = 6
hint.Parent = window

-- Lógica de arrastar o slider
local sliding = false
local function updateSlider(x)
	local width = track.AbsoluteSize.X
	if width <= 0 then return end
	setPower(1 + math.clamp((x - track.AbsolutePosition.X) / width, 0, 1) * 99)
end

connect(track.InputBegan, function(i)
	if isPress(i) then
		sliding = true
		updateSlider(i.Position.X)
	end
end)
connect(UserInputService.InputChanged, function(i)
	if sliding and isMove(i) then
		updateSlider(i.Position.X)
	end
end)
connect(UserInputService.InputEnded, function(i)
	if isPress(i) then sliding = false end
end)

setPower(power)

-- Limpa o registro de cooldown de quem saiu do jogo
connect(Players.PlayerRemoving, function(p)
	lastHit[p] = nil
end)

-- 💥 Lógica do Touch Fling (quando alguém encosta em você)
local function getRoot(p)
	local char = p.Character
	return char and char:FindFirstChild("HumanoidRootPart")
end

connect(RunService.Stepped, function()
	if not enabled then return end

	local root = getRoot(player)
	if not root then return end

	local now = os.clock()
	local rangeSq = HIT_RANGE * HIT_RANGE

	for _, other in ipairs(Players:GetPlayers()) do
		if other ~= player then
			local oRoot = getRoot(other)
			if oRoot and now - (lastHit[other] or 0) >= HIT_COOLDOWN then
				local offset = oRoot.Position - root.Position
				if offset.X * offset.X + offset.Y * offset.Y + offset.Z * offset.Z <= rangeSq then
					lastHit[other] = now

					-- Direção horizontal: sempre para longe de você
					local flat = Vector3.new(offset.X, 0, offset.Z)
					local dir
					if flat.Magnitude > 0.01 then
						dir = flat.Unit
					else
						local a = rng:NextNumber(0, math.pi * 2)
						dir = Vector3.new(math.cos(a), 0, math.sin(a))
					end

					local horizontal = power * 4
					local vertical = power * 1.2
					local vel = dir * horizontal + Vector3.new(0, vertical, 0)
					if vel.Magnitude > MAX_SPEED then
						vel = vel.Unit * MAX_SPEED
					end

					local spin = power * 0.2
					oRoot.AssemblyLinearVelocity = vel
					oRoot.AssemblyAngularVelocity = Vector3.new(
						rng:NextNumber(-spin, spin),
						rng:NextNumber(-spin, spin),
						rng:NextNumber(-spin, spin)
					)
				end
			end
		end
	end
end)
