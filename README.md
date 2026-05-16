-- LocalScript: Fusion Hub — círculo central fixo (máx 1000)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local p = Players.LocalPlayer
local cam = workspace.CurrentCamera

-- Variáveis de controle
local aimbotEnabled = false
local FOV, SPEED = 120, 8
local esp = {}

-- Função ESP
local function setESP(state)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= p and plr.Character then
            local c = plr.Character
            if state then
                if not esp[c] then
                    local h = Instance.new("Highlight")
                    h.FillTransparency = 1
                    h.OutlineColor = Color3.fromRGB(255,0,0)
                    h.Parent = c
                    esp[c] = h
                end
            else
                if esp[c] then esp[c]:Destroy() esp[c] = nil end
            end
        end
    end
end

-- Retorna o melhor alvo dentro do círculo
local function getTargetInCircle(circleFrame)
    local best, angMin = nil, FOV
    if not circleFrame or not circleFrame.Parent then return nil end

    local centerX = circleFrame.AbsolutePosition.X + circleFrame.AbsoluteSize.X/2
    local centerY = circleFrame.AbsolutePosition.Y + circleFrame.AbsoluteSize.Y/2
    local radius = math.max(circleFrame.AbsoluteSize.X, circleFrame.AbsoluteSize.Y) / 2

    for _, plr in ipairs(Players:GetPlayers()) do
        local c = plr.Character
        if plr ~= p and c and c:FindFirstChild("HumanoidRootPart") then
            local hrp = c.HumanoidRootPart
            local screenPos, onScreen = cam:WorldToViewportPoint(hrp.Position)
            if onScreen then
                local dx = screenPos.X - centerX
                local dy = screenPos.Y - centerY
                local dist = math.sqrt(dx*dx + dy*dy)
                if dist <= radius then
                    local dir = (hrp.Position - cam.CFrame.Position).Unit
                    local ang = math.deg(math.acos(math.clamp(cam.CFrame.LookVector:Dot(dir), -1, 1)))
                    if ang < angMin then angMin, best = ang, hrp end
                end
            end
        end
    end
    return best
end

-- Loop do aimbot
local circleRef = nil
RunService.RenderStepped:Connect(function(dt)
    if aimbotEnabled and circleRef then
        local t = getTargetInCircle(circleRef)
        if t then
            cam.CFrame = cam.CFrame:Lerp(
                CFrame.new(cam.CFrame.Position, t.Position),
                dt * SPEED
            )
        end
    end
end)

-- Tecla L para alternar
UserInputService.InputBegan:Connect(function(i,gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.L then
        aimbotEnabled = not aimbotEnabled
        print("Aimbot FUSION HUB:", aimbotEnabled and "ATIVADO" or "DESATIVADO")
        setESP(aimbotEnabled)
        local gui = p:FindFirstChild("PlayerGui")
        if gui then
            local sg = gui:FindFirstChild("FusionHubGui")
            if sg and sg:FindFirstChild("Main") then
                local btn = sg.Main.ToggleTest
                if btn then
                    btn.Text = aimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
                end
                local status = sg.Main:FindFirstChild("AimbotStatus")
                if status then
                    status.Text = aimbotEnabled and "Aimbot ATIVADO" or "Aimbot DESATIVADO"
                    status.TextColor3 = aimbotEnabled and Color3.fromRGB(225,16,0) or Color3.fromRGB(200,200,200)
                end
            end
        end
    end
end)

-- =========================
-- GUI: menu maior, organizado; círculo central com tamanho até 1000
-- =========================
local function createMenu()
    local playerGui = p:WaitForChild("PlayerGui")

    if playerGui:FindFirstChild("FusionHubGui") then
        return playerGui.FusionHubGui
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FusionHubGui"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui

    -- Main frame aumentado
    local main = Instance.new("Frame")
    main.Name = "Main"
    main.Size = UDim2.new(0, 460, 0, 360)
    main.Position = UDim2.new(0, 20, 0, 20)
    main.BackgroundColor3 = Color3.fromRGB(10,10,10)
    main.BorderSizePixel = 0
    main.Parent = screenGui
    Instance.new("UICorner", main).CornerRadius = UDim.new(0, 10)

    -- Cabeçalho
    local header = Instance.new("Frame", main)
    header.Name = "Header"
    header.Size = UDim2.new(1, -16, 0, 48)
    header.Position = UDim2.new(0, 8, 0, 8)
    header.BackgroundTransparency = 1

    local title = Instance.new("TextLabel", header)
    title.Name = "Title"
    title.Size = UDim2.new(0.6, 0, 1, 0)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "Fusion Hub"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 20
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextXAlignment = Enum.TextXAlignment.Left

    local minBtn = Instance.new("TextButton", header)
    minBtn.Name = "Minimize"
    minBtn.Size = UDim2.new(0, 36, 0, 28)
    minBtn.Position = UDim2.new(1, -108, 0, 10)
    minBtn.BackgroundColor3 = Color3.fromRGB(25,25,25)
    minBtn.Text = "_"
    minBtn.Font = Enum.Font.GothamBold
    minBtn.TextSize = 18
    minBtn.TextColor3 = Color3.fromRGB(255,255,255)
    Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,6)

    local closeBtn = Instance.new("TextButton", header)
    closeBtn.Name = "Close"
    closeBtn.Size = UDim2.new(0, 36, 0, 28)
    closeBtn.Position = UDim2.new(1, -64, 0, 10)
    closeBtn.BackgroundColor3 = Color3.fromRGB(25,25,25)
    closeBtn.Text = "X"
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 14
    closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0,6)
    closeBtn.MouseButton1Click:Connect(function() main.Visible = not main.Visible end)

    -- Linha decorativa
    local deco = Instance.new("Frame", main)
    deco.Name = "Deco"
    deco.Size = UDim2.new(1, -16, 0, 6)
    deco.Position = UDim2.new(0, 8, 0, 64)
    deco.BackgroundColor3 = Color3.fromRGB(225,16,0)
    deco.BorderSizePixel = 0
    Instance.new("UICorner", deco).CornerRadius = UDim.new(0, 4)

    -- Conteúdo principal dividido em colunas
    local content = Instance.new("Frame", main)
    content.Name = "Content"
    content.Size = UDim2.new(1, -16, 1, -86)
    content.Position = UDim2.new(0, 8, 0, 78)
    content.BackgroundTransparency = 1

    -- Coluna esquerda (controles principais)
    local leftCol = Instance.new("Frame", content)
    leftCol.Name = "LeftCol"
    leftCol.Size = UDim2.new(0.5, -8, 1, 0)
    leftCol.Position = UDim2.new(0, 0, 0, 0)
    leftCol.BackgroundTransparency = 1

    -- Coluna direita (sliders e status)
    local rightCol = Instance.new("Frame", content)
    rightCol.Name = "RightCol"
    rightCol.Size = UDim2.new(0.5, -8, 1, 0)
    rightCol.Position = UDim2.new(0.5, 8, 0, 0)
    rightCol.BackgroundTransparency = 1

    -- ROW helper function
    local function makeRow(parent, y)
        local row = Instance.new("Frame", parent)
        row.Size = UDim2.new(1, 0, 0, 44)
        row.Position = UDim2.new(0, 0, 0, y)
        row.BackgroundTransparency = 1
        return row
    end

    -- Left column rows
    local row0 = makeRow(leftCol, 0)   -- botões principais
    local row1 = makeRow(leftCol, 52)  -- esp toggle + circle toggle
    local row2 = makeRow(leftCol, 104) -- status
    local row3 = makeRow(leftCol, 156) -- espaço extra

    -- Right column rows (sliders)
    local rrow0 = makeRow(rightCol, 0)
    local rrow1 = makeRow(rightCol, 72)
    local rrow2 = makeRow(rightCol, 144)

    -- Botões principais (Left row0)
    local toggleBtn = Instance.new("TextButton", row0)
    toggleBtn.Name = "ToggleTest"
    toggleBtn.Size = UDim2.new(0.48, 0, 1, 0)
    toggleBtn.Position = UDim2.new(0, 0, 0, 0)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    toggleBtn.TextColor3 = Color3.fromRGB(255,255,255)
    toggleBtn.Font = Enum.Font.GothamSemibold
    toggleBtn.TextSize = 16
    toggleBtn.Text = aimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
    Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0,6)
    local tStroke = Instance.new("UIStroke", toggleBtn)
    tStroke.Color = Color3.fromRGB(180,0,0)
    tStroke.Thickness = 1

    local espBtn = Instance.new("TextButton", row0)
    espBtn.Name = "ToggleESP"
    espBtn.Size = UDim2.new(0.48, 0, 1, 0)
    espBtn.Position = UDim2.new(0.52, 0, 0, 0)
    espBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    espBtn.TextColor3 = Color3.fromRGB(255,255,255)
    espBtn.Font = Enum.Font.GothamSemibold
    espBtn.TextSize = 16
    espBtn.Text = "ESP: OFF"
    Instance.new("UICorner", espBtn).CornerRadius = UDim.new(0,6)
    local eStroke = Instance.new("UIStroke", espBtn)
    eStroke.Color = Color3.fromRGB(180,0,0)
    eStroke.Thickness = 1

    -- Circle toggle (Left row1)
    local circleToggle = Instance.new("TextButton", row1)
    circleToggle.Name = "CircleToggle"
    circleToggle.Size = UDim2.new(0.98, 0, 1, 0)
    circleToggle.Position = UDim2.new(0, 0, 0, 0)
    circleToggle.BackgroundColor3 = Color3.fromRGB(30,30,30)
    circleToggle.TextColor3 = Color3.fromRGB(255,255,255)
    circleToggle.Font = Enum.Font.Gotham
    circleToggle.TextSize = 14
    circleToggle.Text = "Mostrar Círculo: ON"
    Instance.new("UICorner", circleToggle).CornerRadius = UDim.new(0,6)

    -- Status label (Left row2)
    local statusLabel = Instance.new("TextLabel", row2)
    statusLabel.Name = "AimbotStatus"
    statusLabel.Size = UDim2.new(1, 0, 1, 0)
    statusLabel.Position = UDim2.new(0, 0, 0, 0)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = aimbotEnabled and "Aimbot ATIVADO" or "Aimbot DESATIVADO"
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextSize = 14
    statusLabel.TextColor3 = aimbotEnabled and Color3.fromRGB(225,16,0) or Color3.fromRGB(200,200,200)
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left

    -- Sliders on right column
    local function makeSlider(parent, labelText, y)
        local container = Instance.new("Frame", parent)
        container.Size = UDim2.new(1, 0, 0, 56)
        container.Position = UDim2.new(0, 0, 0, y)
        container.BackgroundTransparency = 1

        local lbl = Instance.new("TextLabel", container)
        lbl.Size = UDim2.new(1, 0, 0, 18)
        lbl.Position = UDim2.new(0, 0, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Font = Enum.Font.Gotham
        lbl.TextSize = 14
        lbl.TextColor3 = Color3.fromRGB(255,255,255)
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Text = labelText

        local bar = Instance.new("Frame", container)
        bar.Size = UDim2.new(1, 0, 0, 16)
        bar.Position = UDim2.new(0, 0, 0, 28)
        bar.BackgroundColor3 = Color3.fromRGB(30,30,30)
        bar.BorderSizePixel = 0
        Instance.new("UICorner", bar).CornerRadius = UDim.new(0,6)

        local fill = Instance.new("Frame", bar)
        fill.Name = "Fill"
        fill.Size = UDim2.new(0.5, 0, 1, 0)
        fill.BackgroundColor3 = Color3.fromRGB(225,16,0)
        Instance.new("UICorner", fill).CornerRadius = UDim.new(0,6)

        return lbl, bar, fill
    end

    local fovLabel, fovBar, fovFill = makeSlider(rightCol, "FOV: " .. tostring(FOV), 0)
    local spLabel, spBar, spFill = makeSlider(rightCol, "SPEED: " .. tostring(SPEED), 72)
    local sizeLabel, sizeBar, sizeFill = makeSlider(rightCol, "Tamanho Círculo: 200", 144)

    -- Minimized frame (ícone)
    local mini = Instance.new("Frame")
    mini.Name = "Minimized"
    mini.Size = UDim2.new(0, 56, 0, 34)
    mini.Position = UDim2.new(0, 20, 0, 20)
    mini.BackgroundColor3 = Color3.fromRGB(15,15,15)
    mini.BorderSizePixel = 0
    mini.Visible = false
    mini.Parent = screenGui
    Instance.new("UICorner", mini).CornerRadius = UDim.new(0,8)

    local miniLabel = Instance.new("TextLabel", mini)
    miniLabel.Size = UDim2.new(1, -8, 1, 0)
    miniLabel.Position = UDim2.new(0, 4, 0, 0)
    miniLabel.BackgroundTransparency = 1
    miniLabel.Text = "Fusion"
    miniLabel.Font = Enum.Font.GothamBold
    miniLabel.TextSize = 14
    miniLabel.TextColor3 = Color3.fromRGB(225,16,0)
    miniLabel.TextXAlignment = Enum.TextXAlignment.Center
    miniLabel.TextYAlignment = Enum.TextYAlignment.Center

    -- Círculo central (fixo no centro)
    local circle = Instance.new("Frame", screenGui)
    circle.Name = "AimbotCircle"
    circle.Size = UDim2.new(0, 200, 0, 200) -- inicial
    circle.BackgroundColor3 = Color3.fromRGB(0,0,0)
    circle.BackgroundTransparency = 0.6
    circle.BorderSizePixel = 0
    circle.ZIndex = 10
    Instance.new("UICorner", circle).CornerRadius = UDim.new(1, 0)
    local stroke = Instance.new("UIStroke", circle)
    stroke.Color = Color3.fromRGB(225,16,0)
    stroke.Thickness = 2

    -- Função para centralizar o círculo com base no seu tamanho atual
    local function centerCircle()
        local sizeX = circle.AbsoluteSize.X
        local sizeY = circle.AbsoluteSize.Y
        circle.Position = UDim2.new(0.5, -math.floor(sizeX/2), 0.5, -math.floor(sizeY/2))
    end

    -- Atualiza centralização quando a viewport muda (mantém sempre no centro)
    RunService.RenderStepped:Connect(function()
        if circle and circle.Parent then
            centerCircle()
        end
    end)

    -- Função utilitária para sliders
    local function sliderInput(bar, fill, minVal, maxVal, onChange)
        local dragging = false
        local function update(input)
            local absPos = input.Position.X - bar.AbsolutePosition.X
            local ratio = math.clamp(absPos / bar.AbsoluteSize.X, 0, 1)
            fill.Size = UDim2.new(ratio, 0, 1, 0)
            local value = minVal + (maxVal - minVal) * ratio
            onChange(value)
        end

        bar.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                update(input)
            end
        end)
        bar.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                update(input)
            end
        end)
    end

    -- Conexões dos botões
    toggleBtn.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        toggleBtn.Text = aimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
        statusLabel.Text = aimbotEnabled and "Aimbot ATIVADO" or "Aimbot DESATIVADO"
        statusLabel.TextColor3 = aimbotEnabled and Color3.fromRGB(225,16,0) or Color3.fromRGB(200,200,200)
        setESP(aimbotEnabled)
    end)

    local espState = false
    espBtn.MouseButton1Click:Connect(function()
        espState = not espState
        espBtn.Text = espState and "ESP: ON" or "ESP: OFF"
        setESP(espState)
    end)

    local circleVisible = true
    circleToggle.MouseButton1Click:Connect(function()
        circleVisible = not circleVisible
        circle.Visible = circleVisible
        circleToggle.Text = "Mostrar Círculo: " .. (circleVisible and "ON" or "OFF")
    end)

    -- Sliders: FOV 30..180, SPEED 0..30, Size 50..1000 (máx 1000)
    sliderInput(fovBar, fovFill, 30, 180, function(val)
        FOV = math.floor(val + 0.5)
        fovLabel.Text = "FOV: " .. tostring(FOV)
    end)

    sliderInput(spBar, spFill, 0, 30, function(val)
        SPEED = math.floor(val + 0.5)
        spLabel.Text = "SPEED: " .. tostring(SPEED)
    end)

    -- Tamanho do círculo agora limitado a 1000
    sliderInput(sizeBar, sizeFill, 50, 1000, function(val)
        local size = math.floor(val + 0.5)
        circle.Size = UDim2.new(0, size, 0, size)
        sizeLabel.Text = "Tamanho Círculo: " .. tostring(size)
        centerCircle()
    end)

    -- Minimize behavior
    minBtn.MouseButton1Click:Connect(function()
        main.Visible = false
        mini.Visible = true
        mini.Position = UDim2.new(0, main.AbsolutePosition.X, 0, main.AbsolutePosition.Y)
    end)

    mini.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            main.Visible = true
            mini.Visible = false
        end
    end)

    -- expõe referência do círculo para o loop do aimbot
    circleRef = circle

    -- centraliza inicialmente (aguarda um frame para AbsoluteSize válido)
    RunService.Heartbeat:Wait()
    centerCircle()

    return screenGui
end

-- Cria o menu ao iniciar
createMenu()
