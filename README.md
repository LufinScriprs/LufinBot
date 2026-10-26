--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
	Redesigned with movable Status UI, compact sizing, gear/save buttons, collapse/expand header controls.
]]

local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer
local Cam = game.Workspace.CurrentCamera

-- Variáveis de Configuração
local aimbotEnabled = true
local smoothness = 0
local fov = 40
local cameraFov = Cam.FieldOfView
local espActive = false
local transparencyActive = false
local originalTransparencies = {}

-- Arquivo para Salvar a Key
local saveFileName = "SatanKilla_KeyData.json"
local globalExpiresAt = 0

-- Função utilitária para Tween super fluido (Quart Out)
local function animar(objeto, tempo, propriedades)
	local info = TweenInfo.new(
		tempo or 0.2, 
		Enum.EasingStyle.Quart, 
		Enum.EasingDirection.Out
	)
	local tween = TweenService:Create(objeto, info, propriedades)
	tween:Play()
	return tween
end

-- Função de animação suave para botões (Hover/Press)
local function applyButtonAnimation(btn)
	local originalSize = btn.Size
	
	btn.MouseEnter:Connect(function()
		animar(btn, 0.2, {
			Size = UDim2.new(
				originalSize.X.Scale, 
				originalSize.X.Offset + 4, 
				originalSize.Y.Scale, 
				originalSize.Y.Offset + 2
			)
		})
	end)
	
	btn.MouseLeave:Connect(function()
		animar(btn, 0.2, { Size = originalSize })
	end)
end

-- Função para abrir um painel suavemente
local function abrirPainel(frame)
	frame.Visible = true
	frame.Size = UDim2.new(0, 0, 0, 0)
	frame.BackgroundTransparency = 1
	
	local tamanhoAlvo = UDim2.new(0, 360, 0, 260)
	if frame.Name == "MainFrame" then
		tamanhoAlvo = UDim2.new(0, 480, 0, 340)
	elseif frame.Name == "StatsFrame" then
		tamanhoAlvo = UDim2.new(0, 230, 0, 145)
	elseif frame.Name == "KeyFrame" then
		tamanhoAlvo = UDim2.new(0, 360, 0, 230)
	end
	
	animar(frame, 0.3, {
		Size = tamanhoAlvo,
		BackgroundTransparency = 0.05
	})
end

-- Função para fechar um painel suavemente
local function fecharPainel(frame, destroyOnComplete)
	local t = animar(frame, 0.2, {
		Size = UDim2.new(0, 0, 0, 0),
		BackgroundTransparency = 1
	})
	t.Completed:Connect(function()
		if destroyOnComplete then
			frame:Destroy()
		else
			frame.Visible = false
		end
	end)
end

-- Função para iniciar o Hub Principal após a Verificação da Key
local function iniciarHubPrincipal()
	local ScreenGui = Instance.new("ScreenGui")
	ScreenGui.Name = "SatanKillaHub"
	ScreenGui.Parent = CoreGui
	ScreenGui.IgnoreGuiInset = true

	-- Fundo Preto da Animação de Entrada
	local LoadingBackground = Instance.new("Frame")
	LoadingBackground.Size = UDim2.new(1, 0, 1, 0)
	LoadingBackground.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
	LoadingBackground.BorderSizePixel = 0
	LoadingBackground.ZIndex = 10
	LoadingBackground.Parent = ScreenGui

	-- Botão de Abertura Circular Flutuante com Sombra e Gradiente
	local OpenButton = Instance.new("ImageButton")
	OpenButton.Size = UDim2.new(0, 56, 0, 56)
	OpenButton.Position = UDim2.new(0.5, -28, 0, 12)
	OpenButton.BackgroundColor3 = Color3.fromRGB(20, 20, 28)
	OpenButton.Image = "rbxassetid://102299300364102"
	OpenButton.Visible = false
	OpenButton.Parent = ScreenGui

	local OpenBtnCorner = Instance.new("UICorner")
	OpenBtnCorner.CornerRadius = UDim.new(1, 0)
	OpenBtnCorner.Parent = OpenButton

	local OpenBtnStroke = Instance.new("UIStroke")
	OpenBtnStroke.Color = Color3.fromRGB(90, 90, 140)
	OpenBtnStroke.Thickness = 2
	OpenBtnStroke.Parent = OpenButton

	-- Imagem de Loading Inicial
	local LoadingImage = Instance.new("ImageLabel")
	LoadingImage.Size = UDim2.new(0, 80, 0, 80)
	LoadingImage.Position = UDim2.new(0.5, -40, 0.45, -50)
	LoadingImage.BackgroundTransparency = 1
	LoadingImage.Image = "rbxassetid://102299300364102"
	LoadingImage.ImageTransparency = 1
	LoadingImage.ZIndex = 11
	LoadingImage.Parent = ScreenGui

	local LoadingImgCorner = Instance.new("UICorner")
	LoadingImgCorner.CornerRadius = UDim.new(1, 0)
	LoadingImgCorner.Parent = LoadingImage

	local LoadingImgStroke = Instance.new("UIStroke")
	LoadingImgStroke.Color = Color3.fromRGB(120, 120, 180)
	LoadingImgStroke.Transparency = 1
	LoadingImgStroke.Thickness = 2
	LoadingImgStroke.Parent = LoadingImage

	local LoadingTextLabel = Instance.new("TextLabel")
	LoadingTextLabel.Size = UDim2.new(1, 0, 0, 30)
	LoadingTextLabel.Position = UDim2.new(0, 0, 0.45, 45)
	LoadingTextLabel.BackgroundTransparency = 1
	LoadingTextLabel.Font = Enum.Font.SourceSansBold
	LoadingTextLabel.Text = "Carregando Lufin Bot ! ✌️"
	LoadingTextLabel.TextColor3 = Color3.fromRGB(240, 240, 255)
	LoadingTextLabel.TextSize = 16
	LoadingTextLabel.TextTransparency = 1
	LoadingTextLabel.TextXAlignment = Enum.TextXAlignment.Center
	LoadingTextLabel.ZIndex = 11
	LoadingTextLabel.Parent = ScreenGui

	-- Painel Principal (Lufin Bot) Estilo Retangular Rayfield Melhorado
	local MainFrame = Instance.new("Frame")
	MainFrame.Name = "MainFrame"
	MainFrame.Size = UDim2.new(0, 480, 0, 340)
	MainFrame.Position = UDim2.new(0.5, -240, 0.5, -170)
	MainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
	MainFrame.BackgroundTransparency = 0.05
	MainFrame.BorderSizePixel = 0
	MainFrame.Visible = false
	MainFrame.Parent = ScreenGui

	local UICorner = Instance.new("UICorner")
	UICorner.CornerRadius = UDim.new(0, 10)
	UICorner.Parent = MainFrame

	local MainStroke = Instance.new("UIStroke")
	MainStroke.Color = Color3.fromRGB(60, 60, 90)
	MainStroke.Thickness = 1.5
	MainStroke.Parent = MainFrame

	-- Barra de Título Estilo Rayfield
	local TitleBar = Instance.new("Frame")
	TitleBar.Size = UDim2.new(1, 0, 0, 42)
	TitleBar.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	TitleBar.BorderSizePixel = 0
	TitleBar.Parent = MainFrame

	local TitleBarCorner = Instance.new("UICorner")
	TitleBarCorner.CornerRadius = UDim.new(0, 10)
	TitleBarCorner.Parent = TitleBar

	local FixCornerFrame = Instance.new("Frame")
	FixCornerFrame.Size = UDim2.new(1, 0, 0, 10)
	FixCornerFrame.Position = UDim2.new(0, 0, 1, -10)
	FixCornerFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	FixCornerFrame.BorderSizePixel = 0
	FixCornerFrame.Parent = TitleBar

	local HeaderLabel = Instance.new("TextLabel")
	HeaderLabel.Size = UDim2.new(1, -50, 1, 0)
	HeaderLabel.Position = UDim2.new(0, 16, 0, 0)
	HeaderLabel.BackgroundTransparency = 1
	HeaderLabel.Font = Enum.Font.SourceSansBold
	HeaderLabel.Text = "Lufin Bot"
	HeaderLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	HeaderLabel.TextSize = 16
	HeaderLabel.TextXAlignment = Enum.TextXAlignment.Left
	HeaderLabel.Parent = TitleBar

	local CloseBtn = Instance.new("TextButton")
	CloseBtn.Size = UDim2.new(0, 26, 0, 26)
	CloseBtn.Position = UDim2.new(1, -34, 0, 8)
	CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
	CloseBtn.Font = Enum.Font.SourceSansBold
	CloseBtn.Text = "X"
	CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	CloseBtn.TextSize = 12
	CloseBtn.Parent = TitleBar

	local CloseCorner = Instance.new("UICorner")
	CloseCorner.CornerRadius = UDim.new(0, 6)
	CloseCorner.Parent = CloseBtn
	applyButtonAnimation(CloseBtn)

	local ScrollingFrame = Instance.new("ScrollingFrame")
	ScrollingFrame.Size = UDim2.new(1, -20, 1, -56)
	ScrollingFrame.Position = UDim2.new(0, 10, 0, 48)
	ScrollingFrame.BackgroundTransparency = 1
	ScrollingFrame.BorderSizePixel = 0
	ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 360)
	ScrollingFrame.ScrollBarThickness = 4
	ScrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(110, 110, 160)
	ScrollingFrame.Parent = MainFrame

	-- Painel de Estatísticas Redesenhado (Título alterado para "Status")
	local StatsFrame = Instance.new("Frame")
	StatsFrame.Name = "StatsFrame"
	StatsFrame.Size = UDim2.new(0, 230, 0, 145)
	StatsFrame.Position = UDim2.new(0.05, 0, 0.5, -72)
	StatsFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
	StatsFrame.BackgroundTransparency = 0.05
	StatsFrame.BorderSizePixel = 0
	StatsFrame.Visible = false
	StatsFrame.Parent = ScreenGui

	local StatsCorner = Instance.new("UICorner")
	StatsCorner.CornerRadius = UDim.new(0, 10)
	StatsCorner.Parent = StatsFrame

	local StatsStroke = Instance.new("UIStroke")
	StatsStroke.Color = Color3.fromRGB(60, 60, 90)
	StatsStroke.Thickness = 1.2
	StatsStroke.Parent = StatsFrame

	local StatsTitleBar = Instance.new("Frame")
	StatsTitleBar.Size = UDim2.new(1, 0, 0, 32)
	StatsTitleBar.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	StatsTitleBar.BorderSizePixel = 0
	StatsTitleBar.Parent = StatsFrame

	local StatsTitleCorner = Instance.new("UICorner")
	StatsTitleCorner.CornerRadius = UDim.new(0, 10)
	StatsTitleCorner.Parent = StatsTitleBar

	local StatsTitleFix = Instance.new("Frame")
	StatsTitleFix.Size = UDim2.new(1, 0, 0, 8)
	StatsTitleFix.Position = UDim2.new(0, 0, 1, -8)
	StatsTitleFix.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	StatsTitleFix.BorderSizePixel = 0
	StatsTitleFix.Parent = StatsTitleBar

	local StatsTitle = Instance.new("TextLabel")
	StatsTitle.Size = UDim2.new(1, -40, 1, 0)
	StatsTitle.Position = UDim2.new(0, 12, 0, 0)
	StatsTitle.BackgroundTransparency = 1
	StatsTitle.Font = Enum.Font.SourceSansBold
	StatsTitle.Text = "Status"
	StatsTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
	StatsTitle.TextSize = 13
	StatsTitle.TextXAlignment = Enum.TextXAlignment.Left
	StatsTitle.Parent = StatsTitleBar

	local StatsCloseBtn = Instance.new("TextButton")
	StatsCloseBtn.Size = UDim2.new(0, 20, 0, 20)
	StatsCloseBtn.Position = UDim2.new(1, -24, 0, 6)
	StatsCloseBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
	StatsCloseBtn.Font = Enum.Font.SourceSansBold
	StatsCloseBtn.Text = "X"
	StatsCloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	StatsCloseBtn.TextSize = 10
	StatsCloseBtn.Parent = StatsTitleBar

	local StatsCloseCorner = Instance.new("UICorner")
	StatsCloseCorner.CornerRadius = UDim.new(0, 5)
	StatsCloseCorner.Parent = StatsCloseBtn
	applyButtonAnimation(StatsCloseBtn)

	local StatsContentContainer = Instance.new("Frame")
	StatsContentContainer.Name = "ContentContainer"
	StatsContentContainer.Size = UDim2.new(1, 0, 1, -32)
	StatsContentContainer.Position = UDim2.new(0, 0, 0, 32)
	StatsContentContainer.BackgroundTransparency = 1
	StatsContentContainer.Parent = StatsFrame

	local StatsAimbot = Instance.new("TextLabel")
	StatsAimbot.Size = UDim2.new(1, 0, 0, 16)
	StatsAimbot.Position = UDim2.new(0, 12, 0, 8)
	StatsAimbot.BackgroundTransparency = 1
	StatsAimbot.Font = Enum.Font.SourceSansBold
	StatsAimbot.Text = "• Aimbot: ON"
	StatsAimbot.TextColor3 = Color3.fromRGB(0, 240, 0)
	StatsAimbot.TextSize = 11
	StatsAimbot.TextXAlignment = Enum.TextXAlignment.Left
	StatsAimbot.Parent = StatsContentContainer

	local StatsSmooth = Instance.new("TextLabel")
	StatsSmooth.Size = UDim2.new(1, 0, 0, 16)
	StatsSmooth.Position = UDim2.new(0, 12, 0, 26)
	StatsSmooth.BackgroundTransparency = 1
	StatsSmooth.Font = Enum.Font.SourceSansBold
	StatsSmooth.Text = "• Smoothness: 0"
	StatsSmooth.TextColor3 = Color3.fromRGB(210, 210, 220)
	StatsSmooth.TextSize = 11
	StatsSmooth.TextXAlignment = Enum.TextXAlignment.Left
	StatsSmooth.Parent = StatsContentContainer

	local StatsCamFov = Instance.new("TextLabel")
	StatsCamFov.Size = UDim2.new(1, 0, 0, 16)
	StatsCamFov.Position = UDim2.new(0, 12, 0, 44)
	StatsCamFov.BackgroundTransparency = 1
	StatsCamFov.Font = Enum.Font.SourceSansBold
	StatsCamFov.Text = "• Campo de visão: " .. math.floor(Cam.FieldOfView)
	StatsCamFov.TextColor3 = Color3.fromRGB(210, 210, 220)
	StatsCamFov.TextSize = 11
	StatsCamFov.TextXAlignment = Enum.TextXAlignment.Left
	StatsCamFov.Parent = StatsContentContainer

	local StatsFovSize = Instance.new("TextLabel")
	StatsFovSize.Size = UDim2.new(1, 0, 0, 16)
	StatsFovSize.Position = UDim2.new(0, 12, 0, 62)
	StatsFovSize.BackgroundTransparency = 1
	StatsFovSize.Font = Enum.Font.SourceSansBold
	StatsFovSize.Text = "• Tamanho Fov: " .. fov
	StatsFovSize.TextColor3 = Color3.fromRGB(210, 210, 220)
	StatsFovSize.TextSize = 11
	StatsFovSize.TextXAlignment = Enum.TextXAlignment.Left
	StatsFovSize.Parent = StatsContentContainer

	local StatsEsp = Instance.new("TextLabel")
	StatsEsp.Size = UDim2.new(1, 0, 0, 16)
	StatsEsp.Position = UDim2.new(0, 12, 0, 80)
	StatsEsp.BackgroundTransparency = 1
	StatsEsp.Font = Enum.Font.SourceSansBold
	StatsEsp.Text = "• ESP: OFF"
	StatsEsp.TextColor3 = Color3.fromRGB(240, 60, 60)
	StatsEsp.TextSize = 11
	StatsEsp.TextXAlignment = Enum.TextXAlignment.Left
	StatsEsp.Parent = StatsContentContainer

	local StatsTransparency = Instance.new("TextLabel")
	StatsTransparency.Size = UDim2.new(1, 0, 0, 16)
	StatsTransparency.Position = UDim2.new(0, 12, 0, 98)
	StatsTransparency.BackgroundTransparency = 1
	StatsTransparency.Font = Enum.Font.SourceSansBold
	StatsTransparency.Text = "• Transparency: OFF"
	StatsTransparency.TextColor3 = Color3.fromRGB(240, 60, 60)
	StatsTransparency.TextSize = 11
	StatsTransparency.TextXAlignment = Enum.TextXAlignment.Left
	StatsTransparency.Parent = StatsContentContainer

	local OpenStatusBtn = Instance.new("TextButton")
	OpenStatusBtn.Name = "OpenStatusBtn"
	OpenStatusBtn.Size = UDim2.new(0, 36, 0, 36)
	OpenStatusBtn.Position = UDim2.new(0.05, 0, 0.5, -18)
	OpenStatusBtn.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	OpenStatusBtn.Font = Enum.Font.SourceSansBold
	OpenStatusBtn.Text = "🔓"
	OpenStatusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	OpenStatusBtn.TextSize = 14
	OpenStatusBtn.Visible = false
	OpenStatusBtn.Parent = ScreenGui

	local OpenStatusCorner = Instance.new("UICorner")
	OpenStatusCorner.CornerRadius = UDim.new(0, 8)
	OpenStatusCorner.Parent = OpenStatusBtn
	local OpenStatusStroke = Instance.new("UIStroke")
	OpenStatusStroke.Color = Color3.fromRGB(60, 60, 90)
	OpenStatusStroke.Thickness = 1.2
	OpenStatusStroke.Parent = OpenStatusBtn
	applyButtonAnimation(OpenStatusBtn)

	StatsCloseBtn.MouseButton1Click:Connect(function()
		StatsContentContainer.Visible = false
		animar(StatsFrame, 0.2, {Size = UDim2.new(0, 230, 0, 32)})
		task.wait(0.2)
		StatsFrame.Visible = false
		OpenStatusBtn.Position = StatsFrame.Position
		OpenStatusBtn.Visible = true
	end)

	OpenStatusBtn.MouseButton1Click:Connect(function()
		OpenStatusBtn.Visible = false
		StatsFrame.Visible = true
		StatsFrame.Size = UDim2.new(0, 230, 0, 32)
		animar(StatsFrame, 0.2, {Size = UDim2.new(0, 230, 0, 145)})
		task.wait(0.2)
		StatsContentContainer.Visible = true
	end)

	local function updateStatsDisplay()
		if aimbotEnabled then
			StatsAimbot.Text = "• Aimbot: ON"
			StatsAimbot.TextColor3 = Color3.fromRGB(0, 240, 0)
		else
			StatsAimbot.Text = "• Aimbot: OFF"
			StatsAimbot.TextColor3 = Color3.fromRGB(240, 60, 60)
		end
		StatsSmooth.Text = "• Smoothness: " .. smoothness
		StatsCamFov.Text = "• Campo de visão: " .. math.floor(cameraFov)
		StatsFovSize.Text = "• Tamanho Fov: " .. fov
		if espActive then
			StatsEsp.Text = "• ESP: ON"
			StatsEsp.TextColor3 = Color3.fromRGB(0, 240, 0)
		else
			StatsEsp.Text = "• ESP: OFF"
			StatsEsp.TextColor3 = Color3.fromRGB(240, 60, 60)
		end
		if transparencyActive then
			StatsTransparency.Text = "• Transparency: ON"
			StatsTransparency.TextColor3 = Color3.fromRGB(0, 240, 0)
		else
			StatsTransparency.Text = "• Transparency: OFF"
			StatsTransparency.TextColor3 = Color3.fromRGB(240, 60, 60)
		end
	end

	-- Elementos internos do ScrollingFrame (Layout Rayfield Largo)
	local AimLabel = Instance.new("TextLabel")
	AimLabel.Size = UDim2.new(0, 420, 0, 24)
	AimLabel.Position = UDim2.new(0, 15, 0, 10)
	AimLabel.BackgroundTransparency = 1
	AimLabel.Font = Enum.Font.SourceSansBold
	AimLabel.Text = "Configurações de Aim"
	AimLabel.TextColor3 = Color3.fromRGB(240, 240, 255)
	AimLabel.TextSize = 15
	AimLabel.TextXAlignment = Enum.TextXAlignment.Left
	AimLabel.Parent = ScrollingFrame

	local AimbotBtn = Instance.new("TextButton")
	AimbotBtn.Size = UDim2.new(0, 420, 0, 38)
	AimbotBtn.Position = UDim2.new(0, 15, 0, 40)
	AimbotBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
	AimbotBtn.Font = Enum.Font.SourceSansBold
	AimbotBtn.Text = "Aimbot: ON"
	AimbotBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	AimbotBtn.TextSize = 14
	AimbotBtn.Parent = ScrollingFrame

	local BtnCorner1 = Instance.new("UICorner")
	BtnCorner1.CornerRadius = UDim.new(0, 8)
	BtnCorner1.Parent = AimbotBtn
	applyButtonAnimation(AimbotBtn)

	local SmoothLabel = Instance.new("TextLabel")
	SmoothLabel.Size = UDim2.new(0, 420, 0, 18)
	SmoothLabel.Position = UDim2.new(0, 15, 0, 88)
	SmoothLabel.BackgroundTransparency = 1
	SmoothLabel.Font = Enum.Font.SourceSansBold
	SmoothLabel.Text = "Smoothness: 0"
	SmoothLabel.TextColor3 = Color3.fromRGB(210, 210, 220)
	SmoothLabel.TextSize = 13
	SmoothLabel.TextXAlignment = Enum.TextXAlignment.Left
	SmoothLabel.Parent = ScrollingFrame

	local SliderBar = Instance.new("Frame")
	SliderBar.Size = UDim2.new(0, 420, 0, 8)
	SliderBar.Position = UDim2.new(0, 15, 0, 110)
	SliderBar.BackgroundColor3 = Color3.fromRGB(35, 35, 48)
	SliderBar.BorderSizePixel = 0
	SliderBar.Parent = ScrollingFrame

	local SliderCorner = Instance.new("UICorner")
	SliderCorner.CornerRadius = UDim.new(1, 0)
	SliderCorner.Parent = SliderBar

	local SliderBtn = Instance.new("TextButton")
	SliderBtn.Size = UDim2.new(0, 16, 0, 16)
	SliderBtn.Position = UDim2.new(0, 0, 0, -4)
	SliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	SliderBtn.Text = ""
	SliderBtn.Parent = SliderBar

	local SliderBtnCorner = Instance.new("UICorner")
	SliderBtnCorner.CornerRadius = UDim.new(1, 0)
	SliderBtnCorner.Parent = SliderBtn

	local FovSizeLabel = Instance.new("TextLabel")
	FovSizeLabel.Size = UDim2.new(0, 420, 0, 18)
	FovSizeLabel.Position = UDim2.new(0, 15, 0, 134)
	FovSizeLabel.BackgroundTransparency = 1
	FovSizeLabel.Font = Enum.Font.SourceSansBold
	FovSizeLabel.Text = "Tamanho Fov: 40"
	FovSizeLabel.TextColor3 = Color3.fromRGB(210, 210, 220)
	FovSizeLabel.TextSize = 13
	FovSizeLabel.TextXAlignment = Enum.TextXAlignment.Left
	FovSizeLabel.Parent = ScrollingFrame

	local FovSliderBar = Instance.new("Frame")
	FovSliderBar.Size = UDim2.new(0, 420, 0, 8)
	FovSliderBar.Position = UDim2.new(0, 15, 0, 156)
	FovSliderBar.BackgroundColor3 = Color3.fromRGB(35, 35, 48)
	FovSliderBar.BorderSizePixel = 0
	FovSliderBar.Parent = ScrollingFrame

	local FovSliderCorner = Instance.new("UICorner")
	FovSliderCorner.CornerRadius = UDim.new(1, 0)
	FovSliderCorner.Parent = FovSliderBar

	local FovSliderBtn = Instance.new("TextButton")
	FovSliderBtn.Size = UDim2.new(0, 16, 0, 16)
	FovSliderBtn.Position = UDim2.new(0, 0, 0, -4)
	FovSliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	FovSliderBtn.Text = ""
	FovSliderBtn.Parent = FovSliderBar

	local FovSliderBtnCorner = Instance.new("UICorner")
	FovSliderBtnCorner.CornerRadius = UDim.new(1, 0)
	FovSliderBtnCorner.Parent = FovSliderBtn

	local CamFovLabel = Instance.new("TextLabel")
	CamFovLabel.Size = UDim2.new(0, 420, 0, 18)
	CamFovLabel.Position = UDim2.new(0, 15, 0, 180)
	CamFovLabel.BackgroundTransparency = 1
	CamFovLabel.Font = Enum.Font.SourceSansBold
	CamFovLabel.Text = "Campo de Visão: " .. math.floor(Cam.FieldOfView)
	CamFovLabel.TextColor3 = Color3.fromRGB(210, 210, 220)
	CamFovLabel.TextSize = 13
	CamFovLabel.TextXAlignment = Enum.TextXAlignment.Left
	CamFovLabel.Parent = ScrollingFrame

	local CamFovBar = Instance.new("Frame")
	CamFovBar.Size = UDim2.new(0, 420, 0, 8)
	CamFovBar.Position = UDim2.new(0, 15, 0, 202)
	CamFovBar.BackgroundColor3 = Color3.fromRGB(35, 35, 48)
	CamFovBar.BorderSizePixel = 0
	CamFovBar.Parent = ScrollingFrame

	local CamFovCorner = Instance.new("UICorner")
	CamFovCorner.CornerRadius = UDim.new(1, 0)
	CamFovCorner.Parent = CamFovBar

	local CamFovBtn = Instance.new("TextButton")
	CamFovBtn.Size = UDim2.new(0, 16, 0, 16)
	CamFovBtn.Position = UDim2.new(0, 0, 0, -4)
	CamFovBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	CamFovBtn.Text = ""
	CamFovBtn.Parent = CamFovBar

	local CamFovBtnCorner = Instance.new("UICorner")
	CamFovBtnCorner.CornerRadius = UDim.new(1, 0)
	CamFovBtnCorner.Parent = CamFovBtn

	local VisualsLabel = Instance.new("TextLabel")
	VisualsLabel.Size = UDim2.new(0, 420, 0, 24)
	VisualsLabel.Position = UDim2.new(0, 15, 0, 226)
	VisualsLabel.BackgroundTransparency = 1
	VisualsLabel.Font = Enum.Font.SourceSansBold
	VisualsLabel.Text = "Visuals e ESP"
	VisualsLabel.TextColor3 = Color3.fromRGB(240, 240, 255)
	VisualsLabel.TextSize = 15
	VisualsLabel.TextXAlignment = Enum.TextXAlignment.Left
	VisualsLabel.Parent = ScrollingFrame

	local EspBtn = Instance.new("TextButton")
	EspBtn.Size = UDim2.new(0, 420, 0, 38)
	EspBtn.Position = UDim2.new(0, 15, 0, 256)
	EspBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
	EspBtn.Font = Enum.Font.SourceSansBold
	EspBtn.Text = "ESP: OFF"
	EspBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	EspBtn.TextSize = 14
	EspBtn.Parent = ScrollingFrame

	local EspCorner = Instance.new("UICorner")
	EspCorner.CornerRadius = UDim.new(0, 8)
	EspCorner.Parent = EspBtn
	applyButtonAnimation(EspBtn)

	local TransparencyBtn = Instance.new("TextButton")
	TransparencyBtn.Size = UDim2.new(0, 420, 0, 38)
	TransparencyBtn.Position = UDim2.new(0, 15, 0, 304)
	TransparencyBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
	TransparencyBtn.Font = Enum.Font.SourceSansBold
	TransparencyBtn.Text = "Transparency: OFF"
	TransparencyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	TransparencyBtn.TextSize = 14
	TransparencyBtn.Parent = ScrollingFrame

	local TransparencyCorner = Instance.new("UICorner")
	TransparencyCorner.CornerRadius = UDim.new(0, 8)
	TransparencyCorner.Parent = TransparencyBtn
	applyButtonAnimation(TransparencyBtn)

	OpenButton.MouseButton1Click:Connect(function()
		if MainFrame.Visible then
			fecharPainel(MainFrame, false)
		else
			abrirPainel(MainFrame)
		end
	end)

	CloseBtn.MouseButton1Click:Connect(function()
		fecharPainel(MainFrame, false)
	end)

	local sliding = false
	SliderBtn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			sliding = true
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			sliding = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local mousePos = input.Position.X
			local barPos = SliderBar.AbsolutePosition.X
			local barSize = SliderBar.AbsoluteSize.X
			local percentage = math.clamp((mousePos - barPos) / barSize, 0, 1)
			
			SliderBtn.Position = UDim2.new(percentage, -8, 0, -4)
			smoothness = math.floor(percentage * 10) / 10
			SmoothLabel.Text = "Smoothness: " .. smoothness
			updateStatsDisplay()
		end
	end)

	local slidingFov = false
	FovSliderBtn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			slidingFov = true
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			slidingFov = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if slidingFov and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local mousePos = input.Position.X
			local barPos = FovSliderBar.AbsolutePosition.X
			local barSize = FovSliderBar.AbsoluteSize.X
			local percentage = math.clamp((mousePos - barPos) / barSize, 0, 1)
			
			FovSliderBtn.Position = UDim2.new(percentage, -8, 0, -4)
			fov = math.floor(10 + (percentage * (10000 - 10)))
			FovSizeLabel.Text = "Tamanho Fov: " .. fov
			updateStatsDisplay()
		end
	end)

	local slidingCamFov = false
	CamFovBtn.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			slidingCamFov = true
		end
	end)

	UserInputService.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			slidingCamFov = false
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if slidingCamFov and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			local mousePos = input.Position.X
			local barPos = CamFovBar.AbsolutePosition.X
			local barSize = CamFovBar.AbsoluteSize.X
			local percentage = math.clamp((mousePos - barPos) / barSize, 0, 1)
			
			CamFovBtn.Position = UDim2.new(percentage, -8, 0, -4)
			cameraFov = math.floor(60 + (percentage * (175 - 60)))
			Cam.FieldOfView = cameraFov
			CamFovLabel.Text = "Campo de Visão: " .. cameraFov
			updateStatsDisplay()
		end
	end)

	local dragging, dragInput, dragStart, startPos
	TitleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = MainFrame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	TitleBar.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			local delta = input.Position - dragStart
			MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)

	AimbotBtn.MouseButton1Click:Connect(function()
		aimbotEnabled = not aimbotEnabled
		if aimbotEnabled then
			AimbotBtn.Text = "Aimbot: ON"
			AimbotBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
		else
			AimbotBtn.Text = "Aimbot: OFF"
			AimbotBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
		end
		updateStatsDisplay()
	end)

	local function applyHighlight(char)
		if not char then return end
		if not char:FindFirstChild("SatanKillaHighlight") then
			local hl = Instance.new("Highlight")
			hl.Name = "SatanKillaHighlight"
			hl.Adornee = char
			hl.FillColor = Color3.fromRGB(255, 50, 50)
			hl.OutlineColor = Color3.fromRGB(255, 255, 255)
			hl.FillTransparency = 0.5
			hl.OutlineTransparency = 0
			hl.Enabled = espActive
			hl.Parent = char
		else
			char.SatanKillaHighlight.Enabled = espActive
		end
	end

	Players.PlayerAdded:Connect(function(plr)
		plr.CharacterAdded:Connect(function(char)
			if espActive then
				task.wait(0.5)
				applyHighlight(char)
			end
		end)
	end)

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			plr.CharacterAdded:Connect(function(char)
				if espActive then
					task.wait(0.5)
					applyHighlight(char)
				end
			end)
			if plr.Character then
				applyHighlight(plr.Character)
			end
		end
	end

	EspBtn.MouseButton1Click:Connect(function()
		espActive = not espActive
		if espActive then
			EspBtn.Text = "ESP: ON"
			EspBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
			for _, plr in ipairs(Players:GetPlayers()) do
				if plr ~= LocalPlayer and plr.Character then
					applyHighlight(plr.Character)
				end
			end
		else
			EspBtn.Text = "ESP: OFF"
			EspBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
			for _, plr in ipairs(Players:GetPlayers()) do
				if plr ~= LocalPlayer and plr.Character then
					local hl = plr.Character:FindFirstChild("SatanKillaHighlight")
					if hl then
						hl.Enabled = false
					end
				end
			end
		end
		updateStatsDisplay()
	end)

	TransparencyBtn.MouseButton1Click:Connect(function()
		transparencyActive = not transparencyActive
		if transparencyActive then
			TransparencyBtn.Text = "Transparency: ON"
			TransparencyBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
			originalTransparencies = {}
			for _, descendant in ipairs(workspace:GetDescendants()) do
				if descendant:IsA("BasePart") then
					originalTransparencies[descendant] = descendant.Transparency
					descendant.Transparency = 0.55
				end
			end
		else
			TransparencyBtn.Text = "Transparency: OFF"
			TransparencyBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
			for _, descendant in ipairs(workspace:GetDescendants()) do
				if descendant:IsA("BasePart") and originalTransparencies[descendant] ~= nil then
					descendant.Transparency = originalTransparencies[descendant]
				end
			end
		end
		updateStatsDisplay()
	end)

	task.spawn(function()
		local tweenInfoIn = TweenInfo.new(0.8, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
		local tweenInfoOut = TweenInfo.new(0.8, Enum.EasingStyle.Quart, Enum.EasingDirection.In)

		TweenService:Create(LoadingImage, tweenInfoIn, {ImageTransparency = 0}):Play()
		TweenService:Create(LoadingImgStroke, tweenInfoIn, {Transparency = 0}):Play()
		TweenService:Create(LoadingTextLabel, tweenInfoIn, {TextTransparency = 0}):Play()
		task.wait(2.3)

		local out1 = TweenService:Create(LoadingImage, tweenInfoOut, {ImageTransparency = 1})
		local out2 = TweenService:Create(LoadingImgStroke, tweenInfoOut, {Transparency = 1})
		local out3 = TweenService:Create(LoadingTextLabel, tweenInfoOut, {TextTransparency = 1})
		local outBg = TweenService:Create(LoadingBackground, tweenInfoOut, {BackgroundTransparency = 1})
		
		out1:Play()
		out2:Play()
		out3:Play()
		outBg:Play()
		outBg.Completed:Wait()

		LoadingImage:Destroy()
		LoadingTextLabel:Destroy()
		LoadingBackground:Destroy()
		
		MainFrame.Visible = false
		abrirPainel(StatsFrame)
		updateStatsDisplay()
		OpenButton.Visible = true
	end)

	local maxTransparency = 0.1 
	local FOVring = Drawing.new("Circle")
	FOVring.Visible = true
	FOVring.Thickness = 2
	FOVring.Color = Color3.fromRGB(255, 50, 50)
	FOVring.Filled = false
	FOVring.Position = Cam.ViewportSize / 2

	local function updateDrawings()
		FOVring.Position = Cam.ViewportSize / 2
		FOVring.Radius = fov
		FOVring.Visible = true
	end

	local function onKeyDown(input)
		if input.KeyCode == Enum.KeyCode.Delete then
			RunService:UnbindFromRenderStep("FOVUpdate")
			FOVring:Remove()
			ScreenGui:Destroy()
		end
	end

	UserInputService.InputBegan:Connect(onKeyDown)

	local function lookAt(target)
		local currentCFrame = Cam.CFrame
		local lookVector = (target - currentCFrame.Position).unit
		local targetCFrame = CFrame.new(currentCFrame.Position, currentCFrame.Position + lookVector)
		
		if smoothness <= 0 then
			Cam.CFrame = targetCFrame
		else
			Cam.CFrame = currentCFrame:Lerp(targetCFrame, 1 - smoothness)
		end
	end

	local function calculateTransparency(distance)
		local maxDistance = fov > 0 and fov or 1
		local transparency = (1 - (distance / maxDistance)) * maxTransparency
		return transparency
	end

	local function isValidTarget(model)
		if model == LocalPlayer.Character then return false end
		local player = Players:GetPlayerFromCharacter(model)
		if player then
			return player ~= LocalPlayer
		end
		local humanoid = model:FindFirstChildOfClass("Humanoid")
		local head = model:FindFirstChild("Head")
		if humanoid and head and humanoid.Health > 0 then
			return true
		end
		return false
	end

	local function isVisibleCheck(targetPart)
		if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("Head") then return false end
		local origin = LocalPlayer.Character.Head.Position
		local direction = targetPart.Position - origin
		
		local raycastParams = RaycastParams.new()
		raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
		raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, Cam}
		raycastParams.IgnoreWater = true
		
		local result = workspace:Raycast(origin, direction, raycastParams)
		if result then
			local hitCharacter = result.Instance:FindFirstAncestorOfClass("Model")
			if hitCharacter and isValidTarget(hitCharacter) then
				return true
			end
			return false
		end
		return true
	end

	local function getClosestTargetInFOV(trg_part)
		local nearest = nil
		local last = math.huge
		local playerMousePos = Cam.ViewportSize / 2

		local function checkCandidate(char)
			if char and isValidTarget(char) then
				local part = char:FindFirstChild(trg_part)
				if part then
					local ePos, onScreen = Cam:WorldToViewportPoint(part.Position)
					local distance = (Vector2.new(ePos.x, ePos.y) - playerMousePos).Magnitude

					if distance < last and onScreen and distance < fov and isVisibleCheck(part) then
						last = distance
						nearest = char
					end
				end
			end
		end

		for _, player in ipairs(Players:GetPlayers()) do
			if player ~= LocalPlayer and player.Character then
				checkCandidate(player.Character)
			end
		end

		for _, obj in ipairs(workspace:GetChildren()) do
			if obj:IsA("Model") and obj ~= LocalPlayer.Character then
				checkCandidate(obj)
			end
		end

		return nearest
	end

	RunService.RenderStepped:Connect(function()
		updateDrawings()

		if aimbotEnabled then
			local closestChar = getClosestTargetInFOV("Head")
			if closestChar and closestChar:FindFirstChild("Head") then
				lookAt(closestChar.Head.Position)
			end
			
			if closestChar and closestChar:FindFirstChild("Head") then
				local ePos, isVisible = Cam:WorldToViewportPoint(closestChar.Head.Position)
				local distance = (Vector2.new(ePos.x, ePos.y) - (Cam.ViewportSize / 2)).Magnitude
				FOVring.Transparency = calculateTransparency(distance)
			else
				FOVring.Transparency = 0.1 
			end
		else
			FOVring.Transparency = 0.1
		end
	end)
end

-- Sistema de Key com Validação de Key_1 até KEY_24
local function verificarKeySalva()
	local success, exists = pcall(function() return isfile and isfile(saveFileName) end)
	if success and exists then
		local readSuccess, content = pcall(function() return readfile(saveFileName) end)
		if readSuccess and content then
			local decodeSuccess, data = pcall(function() return HttpService:JSONDecode(content) end)
			if decodeSuccess and data and data.expiresAt then
				if os.time() < data.expiresAt then
					globalExpiresAt = data.expiresAt
					return true
				end
			end
		end
	end
	return false
end

if verificarKeySalva() then
	iniciarHubPrincipal()
else
	local KeyScreenGui = Instance.new("ScreenGui")
	KeyScreenGui.Name = "SatanKillaKeySystem"
	KeyScreenGui.Parent = CoreGui
	KeyScreenGui.IgnoreGuiInset = true

	local KeyBackground = Instance.new("Frame")
	KeyBackground.Size = UDim2.new(1, 0, 1, 0)
	KeyBackground.BackgroundColor3 = Color3.fromRGB(12, 12, 18)
	KeyBackground.BackgroundTransparency = 0.5
	KeyBackground.Parent = KeyScreenGui

	local KeyFrame = Instance.new("Frame")
	KeyFrame.Name = "KeyFrame"
	KeyFrame.Size = UDim2.new(0, 360, 0, 230)
	KeyFrame.Position = UDim2.new(0.5, -180, 0.5, -115)
	KeyFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
	KeyFrame.BackgroundTransparency = 0.05
	KeyFrame.BorderSizePixel = 0
	KeyFrame.Parent = KeyScreenGui

	local KeyCorner = Instance.new("UICorner")
	KeyCorner.CornerRadius = UDim.new(0, 10)
	KeyCorner.Parent = KeyFrame

	local KeyStroke = Instance.new("UIStroke")
	KeyStroke.Color = Color3.fromRGB(60, 60, 90)
	KeyStroke.Thickness = 1.2
	KeyStroke.Parent = KeyFrame

	local KeyTitleBar = Instance.new("Frame")
	KeyTitleBar.Size = UDim2.new(1, 0, 0, 36)
	KeyTitleBar.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	KeyTitleBar.BorderSizePixel = 0
	KeyTitleBar.Parent = KeyFrame

	local KeyTitleBarCorner = Instance.new("UICorner")
	KeyTitleBarCorner.CornerRadius = UDim.new(0, 10)
	KeyTitleBarCorner.Parent = KeyTitleBar

	local KeyTitleBarFix = Instance.new("Frame")
	KeyTitleBarFix.Size = UDim2.new(1, 0, 0, 8)
	KeyTitleBarFix.Position = UDim2.new(0, 0, 1, -8)
	KeyTitleBarFix.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	KeyTitleBarFix.BorderSizePixel = 0
	KeyTitleBarFix.Parent = KeyTitleBar

	local KeyTitle = Instance.new("TextLabel")
	KeyTitle.Size = UDim2.new(1, -28, 1, 0)
	KeyTitle.Position = UDim2.new(0, 14, 0, 0)
	KeyTitle.BackgroundTransparency = 1
	KeyTitle.Font = Enum.Font.SourceSansBold
	KeyTitle.Text = "Lufin Key System"
	KeyTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
	KeyTitle.TextSize = 15
	KeyTitle.TextXAlignment = Enum.TextXAlignment.Left
	KeyTitle.Parent = KeyTitleBar

	local KeyTextBox = Instance.new("TextBox")
	KeyTextBox.Size = UDim2.new(0, 324, 0, 42)
	KeyTextBox.Position = UDim2.new(0.5, -162, 0, 52)
	KeyTextBox.BackgroundColor3 = Color3.fromRGB(24, 24, 34)
	KeyTextBox.Font = Enum.Font.SourceSansBold
	KeyTextBox.PlaceholderText = "Cole sua Key Aqui"
	KeyTextBox.Text = ""
	KeyTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	KeyTextBox.PlaceholderColor3 = Color3.fromRGB(130, 130, 160)
	KeyTextBox.TextSize = 13
	KeyTextBox.ClearTextOnFocus = false
	KeyTextBox.Parent = KeyFrame

	local KeyBoxCorner = Instance.new("UICorner")
	KeyBoxCorner.CornerRadius = UDim.new(0, 8)
	KeyBoxCorner.Parent = KeyTextBox

	local KeyBoxStroke = Instance.new("UIStroke")
	KeyBoxStroke.Color = Color3.fromRGB(70, 70, 100)
	KeyBoxStroke.Thickness = 1
	KeyBoxStroke.Parent = KeyTextBox

	local GetKeyBtn = Instance.new("TextButton")
	GetKeyBtn.Size = UDim2.new(0, 324, 0, 36)
	GetKeyBtn.Position = UDim2.new(0.5, -162, 0, 104)
	GetKeyBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
	GetKeyBtn.Font = Enum.Font.SourceSansBold
	GetKeyBtn.Text = "Pegue sua Key !"
	GetKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	GetKeyBtn.TextSize = 13
	GetKeyBtn.Parent = KeyFrame

	local GetKeyCorner = Instance.new("UICorner")
	GetKeyCorner.CornerRadius = UDim.new(0, 8)
	GetKeyCorner.Parent = GetKeyBtn
	applyButtonAnimation(GetKeyBtn)

	local VerifyKeyBtn = Instance.new("TextButton")
	VerifyKeyBtn.Size = UDim2.new(0, 324, 0, 36)
	VerifyKeyBtn.Position = UDim2.new(0.5, -162, 0, 148)
	VerifyKeyBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 80)
	VerifyKeyBtn.Font = Enum.Font.SourceSansBold
	VerifyKeyBtn.Text = "Verify Key"
	VerifyKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
	VerifyKeyBtn.TextSize = 13
	VerifyKeyBtn.Parent = KeyFrame

	local VerifyCorner = Instance.new("UICorner")
	VerifyCorner.CornerRadius = UDim.new(0, 8)
	VerifyCorner.Parent = VerifyKeyBtn
	applyButtonAnimation(VerifyKeyBtn)

	local KeyStatusLabel = Instance.new("TextLabel")
	KeyStatusLabel.Size = UDim2.new(1, 0, 0, 20)
	KeyStatusLabel.Position = UDim2.new(0, 0, 0, 194)
	KeyStatusLabel.BackgroundTransparency = 1
	KeyStatusLabel.Font = Enum.Font.SourceSansBold
	KeyStatusLabel.Text = ""
	KeyStatusLabel.TextColor3 = Color3.fromRGB(240, 60, 60)
	KeyStatusLabel.TextSize = 12
	KeyStatusLabel.TextXAlignment = Enum.TextXAlignment.Center
	KeyStatusLabel.Parent = KeyFrame

	abrirPainel(KeyFrame)

	GetKeyBtn.MouseButton1Click:Connect(function()
		pcall(function()
			setclipboard("https://getkeyherelufin.netlify.app")
		end)
		GetKeyBtn.Text = "Link Copiado! ✅"
		task.wait(1.5)
		GetKeyBtn.Text = "Pegue sua Key !"
	end)

	VerifyKeyBtn.MouseButton1Click:Connect(function()
		local textInput = KeyTextBox.Text
		local isValid = false
		
		-- Verifica se a key está no intervalo de Key_1 até KEY_24 (case-insensitive ou formato exato)
		if textInput:match("^[Kk][Ee][Yy]_(%d+)$") then
			local num = tonumber(textInput:match("^[Kk][Ee][Yy]_(%d+)$"))
			if num and num >= 1 and num <= 24 then
				isValid = true
			end
		end

		if isValid then
			local hours = 24
			local expireTime = os.time() + (hours * 3600)
			globalExpiresAt = expireTime
			pcall(function()
				if writefile then
					writefile(saveFileName, HttpService:JSONEncode({expiresAt = expireTime}))
				end
			end)

			KeyStatusLabel.TextColor3 = Color3.fromRGB(0, 240, 0)
			KeyStatusLabel.Text = "Key Válida!"
			task.wait(0.6)
			fecharPainel(KeyFrame, true)
			task.wait(0.2)
			KeyScreenGui:Destroy()
			iniciarHubPrincipal()
			return
		end
		
		KeyStatusLabel.TextColor3 = Color3.fromRGB(240, 60, 60)
		KeyStatusLabel.Text = "Key Incorreta"
	end)
end
