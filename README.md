-- PiterHub_LocalScript.lua
-- Coloque este LocalScript em StarterPlayerScripts (ou StarterGui como LocalScript)
-- Versão final: flash dos botões agora também pinta o botão de verde durante a mensagem e depois reverte

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local GuiService = game:GetService("GuiService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- CORES
local BG_COLOR = Color3.fromRGB(0, 0, 0)           -- fundo preto (com transparência aplicada)
local BUTTON_PURPLE = Color3.fromRGB(95, 22, 121)  -- roxo principal dos botões
local CIRCLE_BG = Color3.fromRGB(10, 10, 20)       -- centro botão redondo
local CIRCLE_BORDER = Color3.fromRGB(95, 22, 121)  -- borda roxa do anel
local TEXT_WHITE = Color3.fromRGB(255, 255, 255)
local ESP_GREEN = Color3.fromRGB(53, 167, 84)      -- verde para ESP ON / flashes

-- TRANSPARÊNCIA DO FUNDO (ajustável)
local MAIN_BG_TRANSPARENCY = 0.15

-- Limpa GUIs antigas com mesmo nome (útil em testes)
for _, child in ipairs(PlayerGui:GetChildren()) do
	if child.Name == "PiterHubGui" then
		child:Destroy()
	end
end

-- cria ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PiterHubGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

-- Função: torna um alvo movível quando um handle é arrastado.
local function makeDraggableOnHandle(handle, target)
	local dragging = false
	local dragInput = nil
	local dragStart = nil
	local startPos = nil

	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = target.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	handle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging and dragStart and startPos then
			local delta = input.Position - dragStart
			local newX = startPos.X.Offset + delta.X
			local newY = startPos.Y.Offset + delta.Y
			local screenX, screenY = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize.X or 1920, workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize.Y or 1080
			local absSize = target.AbsoluteSize
			if newX < 0 then newX = 0 end
			if newY < 0 then newY = 0 end
			if newX + absSize.X > screenX then newX = screenX - absSize.X end
			if newY + absSize.Y > screenY then newY = screenY - absSize.Y end
			target.Position = UDim2.new(0, math.floor(newX), 0, math.floor(newY))
		end
	end)
end

-- Criando um anel (border) ao redor do botão circular
local openBorder = Instance.new("Frame")
openBorder.Name = "OpenButtonBorder"
openBorder.Size = UDim2.new(0, 56, 0, 56)
openBorder.Position = UDim2.new(0, 10, 0, 10)
openBorder.AnchorPoint = Vector2.new(0, 0)
openBorder.BackgroundColor3 = CIRCLE_BORDER
openBorder.BorderSizePixel = 0
openBorder.Parent = screenGui

local openBorderCorner = Instance.new("UICorner")
openBorderCorner.CornerRadius = UDim.new(1, 0)
openBorderCorner.Parent = openBorder

-- Inner button (o botão real, centrado dentro do anel)
local openButton = Instance.new("TextButton")
openButton.Name = "OpenButton"
openButton.Size = UDim2.new(0, 48, 0, 48)
openButton.AnchorPoint = Vector2.new(0.5, 0.5)
openButton.Position = UDim2.new(0.5, 0, 0.5, 0)
openButton.BackgroundColor3 = CIRCLE_BG
openButton.Text = "Dnhub"
openButton.Font = Enum.Font.Arcade
openButton.TextSize = 16
openButton.TextColor3 = TEXT_WHITE
openButton.Parent = openBorder
openButton.AutoButtonColor = true
openButton.BorderSizePixel = 0

local openInnerCorner = Instance.new("UICorner")
openInnerCorner.CornerRadius = UDim.new(1, 0)
openInnerCorner.Parent = openButton

-- tornar o anel movível arrastando o próprio anel ou o botão interno
makeDraggableOnHandle(openBorder, openBorder)
makeDraggableOnHandle(openButton, openBorder)

-- main frame (caixa principal) com leve transparência no fundo
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 220, 0, 260)
mainFrame.Position = UDim2.new(0, 76, 0, 10)
mainFrame.BackgroundColor3 = BG_COLOR
mainFrame.BackgroundTransparency = MAIN_BG_TRANSPARENCY
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Visible = false

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 14)
mainCorner.Parent = mainFrame

-- tornar a GUI principal arrastável
makeDraggableOnHandle(mainFrame, mainFrame)

-- título "Piter HUB"
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 36)
title.Position = UDim2.new(0, 0, 0, 6)
title.BackgroundTransparency = 1
title.Text = "DN HUB"
title.Font = Enum.Font.Arcade
title.TextSize = 20
title.TextColor3 = TEXT_WHITE
title.Parent = mainFrame

-- Top tag único "ESP BEST" (ocupando largura maior)
local function createTopTagWide(parent, xOffset, width, text)
	local tag = Instance.new("TextButton")
	tag.Size = UDim2.new(0, width, 0, 28)
	tag.Position = UDim2.new(0, xOffset, 0, 38)
	tag.BackgroundColor3 = BUTTON_PURPLE
	tag.Text = text
	tag.Font = Enum.Font.Arcade
	tag.TextSize = 14
	tag.TextColor3 = TEXT_WHITE
	tag.BorderSizePixel = 0
	tag.Parent = parent
	local corner = Instance.new("UICorner", tag)
	corner.CornerRadius = UDim.new(0, 10)
	return tag
end

local topLeftTag = createTopTagWide(mainFrame, 12, 196, "ESP BEST")
topLeftTag.Name = "ESP_BEST_Tag"

-- container para os botões grandes
local buttonContainer = Instance.new("Frame")
buttonContainer.Name = "ButtonContainer"
buttonContainer.Size = UDim2.new(1, -16, 1, -84)
buttonContainer.Position = UDim2.new(0, 8, 0, 86)
buttonContainer.BackgroundTransparency = 1
buttonContainer.Parent = mainFrame

-- função para criar botões grandes
local function createBigButton(parent, yOffset, text)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, 0, 0, 48)
	btn.Position = UDim2.new(0, 0, 0, yOffset)
	btn.BackgroundColor3 = BUTTON_PURPLE
	btn.Text = text
	btn.Font = Enum.Font.Arcade
	btn.TextSize = 20
	btn.TextColor3 = TEXT_WHITE
	btn.BorderSizePixel = 0
	btn.Parent = parent
	local corner = Instance.new("UICorner", btn)
	corner.CornerRadius = UDim.new(0, 12)
	return btn
end

local spacing = 12
local btnTP = createBigButton(buttonContainer, 0, "TP to Saved")
local btnSave = createBigButton(buttonContainer, 48 + spacing, "Save Position")
local btnDiscord = createBigButton(buttonContainer, (48 + spacing) * 2, "Discord")

-- armazenar textos originais (garantindo que existam) e tokens para flashes
local originalTexts = {}
originalTexts[btnTP] = btnTP.Text
originalTexts[btnSave] = btnSave.Text
originalTexts[btnDiscord] = btnDiscord.Text

local flashTokens = {} -- mapa btn -> token

-- função utilitária: troca o texto do botão para a mensagem por 'duration' segundos, pinta de verde e depois reverte
local function flashButtonText(btn, message, duration)
	duration = duration or 1.8
	if not btn or not btn.Parent then return end

	-- guarda texto original no atributo, se ainda não existir
	if not btn:GetAttribute("origText") then
		btn:SetAttribute("origText", originalTexts[btn] or btn.Text or "")
	end
	-- guarda cor de fundo original no atributo, se ainda não existir
	if not btn:GetAttribute("origBg") then
		btn:SetAttribute("origBg", btn.BackgroundColor3)
	end

	-- cria novo token e substitui o atual (cancela flashes anteriores)
	local token = {}
	flashTokens[btn] = token

	-- aplica mensagem atual e pinta de verde
	btn.Text = message
	btn.BackgroundColor3 = ESP_GREEN

	-- thread que aguarda e reverte (só reverte se token ainda for o mesmo)
	spawn(function()
		local elapsed = 0
		local step = 0.05
		while elapsed < duration do
			wait(step)
			elapsed = elapsed + step
			-- se token mudou, aborta esta goroutine (outro flash assumiu)
			if flashTokens[btn] ~= token then
				return
			end
			-- se botão removido, aborta
			if not btn or not btn.Parent then
				flashTokens[btn] = nil
				return
			end
		end

		-- ao terminar, somente reverte se token ainda for o atual
		if flashTokens[btn] == token and btn and btn.Parent then
			-- restaura texto original
			local orig = btn:GetAttribute("origText") or originalTexts[btn] or ""
			btn.Text = orig
			-- restaura cor de fundo original
			local origBg = btn:GetAttribute("origBg")
			if origBg then
				btn.BackgroundColor3 = origBg
			end
			flashTokens[btn] = nil
		end
	end)
end

-- animação abrir/fechar: usa a transparência desejada como alvo ao abrir
local tweenInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
openButton.MouseButton1Click:Connect(function()
	-- evita que clique para abrir seja confundido com início de drag:
	if not mainFrame.Visible then
		mainFrame.BackgroundTransparency = 1
		mainFrame.Visible = true
		local t = TweenService:Create(mainFrame, tweenInfo, {BackgroundTransparency = MAIN_BG_TRANSPARENCY})
		t:Play()
	else
		local t = TweenService:Create(mainFrame, tweenInfo, {BackgroundTransparency = 1})
		t:Play()
		delay(0.15, function()
			mainFrame.Visible = false
			mainFrame.BackgroundTransparency = MAIN_BG_TRANSPARENCY
		end)
	end
end)

-- utilitário para obter HumanoidRootPart com segurança
local function getHRP()
	local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
	local hrp = char:FindFirstChild("HumanoidRootPart") or char:WaitForChild("HumanoidRootPart")
	return hrp, char
end

-- variável para armazenar posição salva
local savedCFrame = nil

-- IMPLEMENTAÇÃO DO X-RAY (ESP) LOCAL
local espOn = false
local descConn = nil
local X_RAY_TRANSPARENCY = 0.6

local function applyXRayToPart(part)
	if not part or not part:IsA("BasePart") then return end
	if LocalPlayer.Character and part:IsDescendantOf(LocalPlayer.Character) then return end
	if part:IsA("Terrain") then return end
	pcall(function()
		part.LocalTransparencyModifier = X_RAY_TRANSPARENCY
	end)
end

local function removeXRayFromPart(part)
	if not part or not part:IsA("BasePart") then return end
	if part:IsA("Terrain") then return end
	pcall(function()
		part.LocalTransparencyModifier = 0
	end)
end

local function enableXRay()
	for _, v in ipairs(workspace:GetDescendants()) do
		applyXRayToPart(v)
	end
	if descConn then descConn:Disconnect() end
	descConn = workspace.DescendantAdded:Connect(function(d)
		if espOn then
			applyXRayToPart(d)
		end
	end)
end

local function disableXRay()
	for _, v in ipairs(workspace:GetDescendants()) do
		removeXRayFromPart(v)
	end
	if descConn then
		descConn:Disconnect()
		descConn = nil
	end
end

-- Clique no topLeftTag para alternar ESP BEST
-- garantir que o atributo de cor original exista para o topLeftTag (usado por flash)
if not topLeftTag:GetAttribute("origBg") then
	topLeftTag:SetAttribute("origBg", topLeftTag.BackgroundColor3)
end
topLeftTag.MouseButton1Click:Connect(function()
	espOn = not espOn
	if espOn then
		topLeftTag.Text = "ESP BEST ON"
		topLeftTag.BackgroundColor3 = ESP_GREEN
		enableXRay()
	else
		topLeftTag.Text = "ESP BEST"
		topLeftTag.BackgroundColor3 = BUTTON_PURPLE
		disableXRay()
	end
end)

-- salvar posição do personagem: mostra a mensagem no próprio botão e pinta de verde durante o flash
btnSave.MouseButton1Click:Connect(function()
	local ok, hrpOrErr = pcall(getHRP)
	if not ok or not hrpOrErr then
		flashButtonText(btnSave, "NÃO FOI POSSÍVEL ACESSAR O PERSONAGEM.", 2.2)
		return
	end
	local hrp = hrpOrErr
	savedCFrame = hrp.CFrame
	flashButtonText(btnSave, "POSIÇÃO SALVA!", 1.8)
end)

-- teleport instantâneo para a posição salva
btnTP.MouseButton1Click:Connect(function()
	if not savedCFrame then
		flashButtonText(btnTP, "NENHUMA POSIÇÃO SALVA.", 2.2)
		return
	end
	local ok, hrpOrErr = pcall(getHRP)
	if not ok or not hrpOrErr then
		flashButtonText(btnTP, "PERSONAGEM INDISPONÍVEL.", 2.2)
		return
	end
	local hrp = hrpOrErr
	local success, err = pcall(function()
		hrp.CFrame = savedCFrame
	end)
	if success then
		flashButtonText(btnTP, "TELEPORTADO!", 1.4)
	else
		flashButtonText(btnTP, "ERRO AO TELEPORTAR", 3)
	end
end)

-- Discord: mostra estado no próprio botão e pinta de verde durante o flash
btnDiscord.MouseButton1Click:Connect(function()
	local url = "https://discord.gg/" -- substitua pelo invite desejado
	local ok, err = pcall(function() GuiService:OpenBrowserWindow(url) end)
	if not ok then
		flashButtonText(btnDiscord, "NÃO FOI POSSÍVEL ABRIR O LINK.", 2.2)
	else
		flashButtonText(btnDiscord, "ABRINDO DISCORD...", 1.6)
	end
end)

-- mantém savedCFrame entre respawns (se quiser limpar ao morrer, descomente)
LocalPlayer.CharacterAdded:Connect(function()
	-- savedCFrame = nil
end)

-- fim do script
