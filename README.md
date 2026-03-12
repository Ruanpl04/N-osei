-- GaramaHop.lua — cola no executor ou autoexec do Xeno
-- Reabre sozinho quando hoppar pro novo servidor

local TeleportService = game:GetService("TeleportService")
local HttpService     = game:GetService("HttpService")
local TweenService    = game:GetService("TweenService")
local Players         = game:GetService("Players")
local CoreGui         = game:GetService("CoreGui")

local player   = Players.LocalPlayer
local PLACE_ID = 109983668079237

if CoreGui:FindFirstChild("HopGui") then
	CoreGui:FindFirstChild("HopGui"):Destroy()
end

local cores = {
	Color3.fromRGB(255, 50,  50),
	Color3.fromRGB(255, 165,  0),
	Color3.fromRGB(50,  255, 50),
	Color3.fromRGB(50,  200, 255),
	Color3.fromRGB(255,  50, 255),
	Color3.fromRGB(255, 255,  0),
}

-- ══════════════════════════════════════
--  GUI
-- ══════════════════════════════════════
local gui = Instance.new("ScreenGui")
gui.Name           = "HopGui"
gui.ResetOnSpawn   = false -- controlamos manualmente
gui.IgnoreGuiInset = true
gui.DisplayOrder   = 999
gui.Parent         = CoreGui

local win = Instance.new("Frame")
win.Size             = UDim2.new(0, 0, 0, 0)
win.AnchorPoint      = Vector2.new(0.5, 0.5)
win.Position         = UDim2.new(0.5, 0, 0.5, 0)
win.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
win.BorderSizePixel  = 0
win.Active           = true
win.Draggable        = true
win.Parent           = gui
Instance.new("UICorner", win).CornerRadius = UDim.new(0, 16)

local stroke = Instance.new("UIStroke")
stroke.Thickness       = 3
stroke.Color           = Color3.fromRGB(255, 80, 0)
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
stroke.Parent          = win

local titulo = Instance.new("TextLabel")
titulo.Size                   = UDim2.new(1, -44, 0, 42)
titulo.Position               = UDim2.new(0, 10, 0, 8)
titulo.BackgroundTransparency = 1
titulo.Text                   = "💀 SERVER HOP 💀"
titulo.Font                   = Enum.Font.GothamBlack
titulo.TextScaled             = true
titulo.TextColor3             = Color3.fromRGB(255, 80, 0)
titulo.Parent                 = win

local btnX = Instance.new("TextButton")
btnX.Size             = UDim2.new(0, 28, 0, 28)
btnX.Position         = UDim2.new(1, -34, 0, 10)
btnX.BackgroundColor3 = Color3.fromRGB(180, 30, 30)
btnX.Text             = "✕"
btnX.Font             = Enum.Font.GothamBold
btnX.TextSize         = 15
btnX.TextColor3       = Color3.fromRGB(255, 255, 255)
btnX.Parent           = win
Instance.new("UICorner", btnX).CornerRadius = UDim.new(0, 6)

local statusBg = Instance.new("Frame")
statusBg.Size             = UDim2.new(1, -20, 0, 42)
statusBg.Position         = UDim2.new(0, 10, 0, 58)
statusBg.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
statusBg.BorderSizePixel  = 0
statusBg.Parent           = win
Instance.new("UICorner", statusBg).CornerRadius = UDim.new(0, 8)

local statusLabel = Instance.new("TextLabel")
statusLabel.Size                   = UDim2.new(1, -10, 1, 0)
statusLabel.Position               = UDim2.new(0, 5, 0, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Font                   = Enum.Font.Gotham
statusLabel.TextScaled             = true
statusLabel.TextWrapped            = true
statusLabel.TextColor3             = Color3.fromRGB(180, 180, 180)
statusLabel.Text                   = "pronto pra hoppar 🗿"
statusLabel.Parent                 = statusBg

local function criarBtn(txt, cor, posY)
	local b = Instance.new("TextButton")
	b.Size             = UDim2.new(1, -20, 0, 46)
	b.Position         = UDim2.new(0, 10, 0, posY)
	b.BackgroundColor3 = cor
	b.Text             = txt
	b.Font             = Enum.Font.GothamBlack
	b.TextSize         = 18
	b.TextColor3       = Color3.fromRGB(255, 255, 255)
	b.AutoButtonColor  = true
	b.Parent           = win
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
	return b
end

local btnHop  = criarBtn("🚀  HOPPAR",          Color3.fromRGB(255, 80,  0),   110)
local btnAuto = criarBtn("⚡  AUTO HOP",         Color3.fromRGB(160, 50, 220),  164)
local btnStop = criarBtn("🛑  PARAR",            Color3.fromRGB(100, 100, 100), 218)

TweenService:Create(win,
	TweenInfo.new(0.4, Enum.EasingStyle.Back, Enum.EasingDirection.Out),
	{ Size = UDim2.new(0, 320, 0, 274) }
):Play()

-- ══════════════════════════════════════
--  LÓGICA
-- ══════════════════════════════════════
local stopFlag = false
local autoAtivo = false

local function setStatus(txt, cor)
	statusLabel.Text       = txt
	statusLabel.TextColor3 = cor or Color3.fromRGB(180, 180, 180)
end

local function hoppar()
	setStatus("⏳ buscando servidores...", Color3.fromRGB(255, 200, 0))

	local ok, resp = pcall(game.HttpGet, game,
		"https://games.roblox.com/v1/games/" .. PLACE_ID .. "/servers/Public?sortOrder=Asc&limit=100")
	if not ok then
		setStatus("❌ erro na API", Color3.fromRGB(255, 80, 80))
		return
	end

	local ok2, data = pcall(HttpService.JSONDecode, HttpService, resp)
	if not ok2 or not data.data then
		setStatus("❌ erro ao ler servidores", Color3.fromRGB(255, 80, 80))
		return
	end

	local jobs = {}
	for _, srv in ipairs(data.data) do
		if (srv.maxPlayers or 0) - (srv.playing or 0) >= 1 and srv.id ~= game.JobId then
			table.insert(jobs, srv.id)
		end
	end

	if #jobs == 0 then
		setStatus("😭 nenhum servidor com vaga", Color3.fromRGB(255, 80, 80))
		return
	end

	setStatus("✅ " .. #jobs .. " servidores! teleportando... 💀", Color3.fromRGB(50, 255, 50))
	task.wait(0.4)
	pcall(TeleportService.Teleport, TeleportService, PLACE_ID, player)
end

-- HOP MANUAL
btnHop.MouseButton1Click:Connect(function()
	stopFlag = true -- para auto se tiver rodando
	autoAtivo = false
	btnAuto.Text             = "⚡  AUTO HOP"
	btnAuto.BackgroundColor3 = Color3.fromRGB(160, 50, 220)
	task.wait(0.05)
	hoppar()
end)

-- AUTO HOP
btnAuto.MouseButton1Click:Connect(function()
	autoAtivo = not autoAtivo
	if autoAtivo then
		stopFlag = false
		btnAuto.Text             = "⚡  AUTO HOP ON 🟢"
		btnAuto.BackgroundColor3 = Color3.fromRGB(50, 180, 50)
		task.spawn(function()
			while autoAtivo and not stopFlag do
				hoppar()
				-- espera teleportar ou dá um intervalo antes do próximo
				task.wait(6)
			end
		end)
	else
		stopFlag = true
		btnAuto.Text             = "⚡  AUTO HOP"
		btnAuto.BackgroundColor3 = Color3.fromRGB(160, 50, 220)
		setStatus("⏹ auto hop parado 🗿", Color3.fromRGB(180, 180, 180))
	end
end)

-- PARAR
btnStop.MouseButton1Click:Connect(function()
	stopFlag  = true
	autoAtivo = false
	btnAuto.Text             = "⚡  AUTO HOP"
	btnAuto.BackgroundColor3 = Color3.fromRGB(160, 50, 220)
	setStatus("⏹ parado 🗿", Color3.fromRGB(180, 180, 180))
end)

-- FECHAR
btnX.MouseButton1Click:Connect(function()
	stopFlag  = true
	autoAtivo = false
	TweenService:Create(win,
		TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In),
		{ Size = UDim2.new(0, 0, 0, 0) }
	):Play()
	task.wait(0.3)
	gui:Destroy()
end)

-- reabre a GUI no novo servidor
-- troca SUAURL pelo link raw do pastebin com este script
local SCRIPT_URL = "https://raw.githubusercontent.com/Ruanpl04/N-osei/refs/heads/main/README.md"
player.CharacterAdded:Connect(function()
	task.wait(2)
	if SCRIPT_URL ~= "SUAURL" then
		pcall(function()
			loadstring(game:HttpGet(SCRIPT_URL))()
		end)
	end
end)

-- ══════════════════════════════════════
--  ANIMAÇÕES
-- ══════════════════════════════════════
task.spawn(function()
	local i = 1
	while gui and gui.Parent do
		i = i % #cores + 1
		TweenService:Create(stroke, TweenInfo.new(0.3), { Color = cores[i] }):Play()
		task.wait(0.35)
	end
end)



task.spawn(function()
	local i = 1
	while gui and gui.Parent do
		i = i % #cores + 1
		TweenService:Create(titulo, TweenInfo.new(0.45, Enum.EasingStyle.Sine),
			{ TextColor3 = cores[i] }):Play()
		task.wait(0.5)
	end
end)
