-- LocalScript: Karateka Hub - Fly Waypoints + Jump Slider + Volta ao Safe
-- Coloque em StarterPlayer -> StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ===================== WAYPOINTS =====================
local waypoints = {
    { pos = Vector3.new(50.403,   108.399,  19.838), look = Vector3.new(32.000,  110.448,  21.125) },
    { pos = Vector3.new(4012.053, 164.871,  32.950), look = Vector3.new(4307.109, 71.000, -24.403) },
    { pos = Vector3.new(4027.311,  -2.765,  32.727), look = Vector3.new(4021.472,  -6.000,  27.470) },
}

-- ===================== ESTADO =====================
local flySpeed      = 50
local jumpPower     = 50
local isFlying      = false
local flyConn       = nil
local autoRoute     = false
local routeConn     = nil
local noclipEnabled = false

-- ===================== GUI =====================
local sg = Instance.new("ScreenGui")
sg.Name = "KaratekaHubFinal"
sg.ResetOnSpawn = false
sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
sg.Parent = PlayerGui

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 46, 0, 46)
toggleBtn.Position = UDim2.new(0, 12, 0.5, -23)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 22
toggleBtn.Text = "☰"
toggleBtn.ZIndex = 10
toggleBtn.Parent = sg
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(1, 0)

local win = Instance.new("Frame")
win.Size = UDim2.new(0, 290, 0, 430)
win.Position = UDim2.new(0, 68, 0.5, -215)
win.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
win.BorderSizePixel = 0
win.Visible = true
win.ZIndex = 5
win.Parent = sg
Instance.new("UICorner", win).CornerRadius = UDim.new(0, 12)

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 44)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
titleBar.BorderSizePixel = 0
titleBar.ZIndex = 6
titleBar.Parent = win
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 12)

local titleLbl = Instance.new("TextLabel")
titleLbl.Size = UDim2.new(1, 0, 1, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Font = Enum.Font.GothamBold
titleLbl.TextSize = 18
titleLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLbl.Text = "🥋 Karateka Hub"
titleLbl.ZIndex = 7
titleLbl.Parent = titleBar

local function mkLabel(parent, text, y, h)
    local l = Instance.new("TextLabel")
    l.Size = UDim2.new(1, -20, 0, h or 20)
    l.Position = UDim2.new(0, 10, 0, y)
    l.BackgroundTransparency = 1
    l.Font = Enum.Font.SourceSans
    l.TextSize = 14
    l.TextColor3 = Color3.fromRGB(210, 210, 210)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Text = text
    l.ZIndex = 6
    l.Parent = parent
    return l
end

local function mkBtn(parent, text, y, h, color)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, -20, 0, h or 34)
    b.Position = UDim2.new(0, 10, 0, y)
    b.BackgroundColor3 = color or Color3.fromRGB(50, 50, 50)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.Text = text
    b.ZIndex = 6
    b.BorderSizePixel = 0
    b.Parent = parent
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
    return b
end

-- ── Fly Manual ──
local flyStatusLbl = mkLabel(win, "✈ Fly Manual: OFF  [E = voar]", 54, 22)
local flySpeedLbl  = mkLabel(win, "Velocidade Fly: " .. tostring(flySpeed), 82, 20)

local minusBtn = Instance.new("TextButton")
minusBtn.Size = UDim2.new(0, 50, 0, 30)
minusBtn.Position = UDim2.new(0, 10, 0, 106)
minusBtn.BackgroundColor3 = Color3.fromRGB(55,55,55)
minusBtn.Font = Enum.Font.GothamBold
minusBtn.TextSize = 18
minusBtn.TextColor3 = Color3.fromRGB(255,255,255)
minusBtn.Text = "−"
minusBtn.ZIndex = 6
minusBtn.BorderSizePixel = 0
minusBtn.Parent = win
Instance.new("UICorner", minusBtn).CornerRadius = UDim.new(0, 8)

local plusBtn = Instance.new("TextButton")
plusBtn.Size = UDim2.new(0, 50, 0, 30)
plusBtn.Position = UDim2.new(0, 68, 0, 106)
plusBtn.BackgroundColor3 = Color3.fromRGB(55,55,55)
plusBtn.Font = Enum.Font.GothamBold
plusBtn.TextSize = 18
plusBtn.TextColor3 = Color3.fromRGB(255,255,255)
plusBtn.Text = "+"
plusBtn.ZIndex = 6
plusBtn.BorderSizePixel = 0
plusBtn.Parent = win
Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 8)

-- ── Rota: Ida e Volta lado a lado ──
local routeBtn = Instance.new("TextButton")
routeBtn.Size = UDim2.new(0, 125, 0, 38)
routeBtn.Position = UDim2.new(0, 10, 0, 148)
routeBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
routeBtn.Font = Enum.Font.GothamBold
routeBtn.TextSize = 13
routeBtn.TextColor3 = Color3.fromRGB(255,255,255)
routeBtn.Text = "🏝 Ir à Ilha Final"
routeBtn.ZIndex = 6
routeBtn.BorderSizePixel = 0
routeBtn.Parent = win
Instance.new("UICorner", routeBtn).CornerRadius = UDim.new(0, 8)

local voltaBtn = Instance.new("TextButton")
voltaBtn.Size = UDim2.new(0, 125, 0, 38)
voltaBtn.Position = UDim2.new(0, 145, 0, 148)
voltaBtn.BackgroundColor3 = Color3.fromRGB(0, 160, 80)
voltaBtn.Font = Enum.Font.GothamBold
voltaBtn.TextSize = 13
voltaBtn.TextColor3 = Color3.fromRGB(255,255,255)
voltaBtn.Text = "🏠 Voltar ao Safe"
voltaBtn.ZIndex = 6
voltaBtn.BorderSizePixel = 0
voltaBtn.Parent = win
Instance.new("UICorner", voltaBtn).CornerRadius = UDim.new(0, 8)

local routeStatusLbl = mkLabel(win, "Rota: aguardando...", 192, 18)
routeStatusLbl.TextColor3 = Color3.fromRGB(160, 200, 255)

-- ── Jump Slider ──
local jumpTitleLbl = mkLabel(win, "⬆ Pulo Turbinado: " .. tostring(jumpPower), 222, 20)

local sliderBg = Instance.new("Frame")
sliderBg.Size = UDim2.new(1, -20, 0, 10)
sliderBg.Position = UDim2.new(0, 10, 0, 248)
sliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
sliderBg.BorderSizePixel = 0
sliderBg.ZIndex = 6
sliderBg.Parent = win
Instance.new("UICorner", sliderBg).CornerRadius = UDim.new(1, 0)

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(0, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(0, 180, 255)
sliderFill.BorderSizePixel = 0
sliderFill.ZIndex = 7
sliderFill.Parent = sliderBg
Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)

local sliderKnob = Instance.new("TextButton")
sliderKnob.Size = UDim2.new(0, 20, 0, 20)
sliderKnob.AnchorPoint = Vector2.new(0.5, 0.5)
sliderKnob.Position = UDim2.new(0, 0, 0.5, 0)
sliderKnob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderKnob.Text = ""
sliderKnob.ZIndex = 8
sliderKnob.BorderSizePixel = 0
sliderKnob.Parent = sliderBg
Instance.new("UICorner", sliderKnob).CornerRadius = UDim.new(1, 0)

local applyJumpBtn = mkBtn(win, "💥 Aplicar Pulo", 268, 34, Color3.fromRGB(180, 60, 0))
local closeWinBtn  = mkBtn(win, "✕ Fechar Menu",   312, 34, Color3.fromRGB(80, 20, 20))
mkLabel(win, "Insert = toggle  |  E = fly  |  G = noclip", 356, 18).TextColor3 = Color3.fromRGB(100,100,100)

-- ===================== SLIDER =====================
local JUMP_MIN, JUMP_MAX = 50, 500
local sliderDragging = false

local function setSlider(value)
    jumpPower = math.clamp(math.floor(value), JUMP_MIN, JUMP_MAX)
    local pct = (jumpPower - JUMP_MIN) / (JUMP_MAX - JUMP_MIN)
    sliderFill.Size = UDim2.new(pct, 0, 1, 0)
    sliderKnob.Position = UDim2.new(pct, 0, 0.5, 0)
    jumpTitleLbl.Text = "⬆ Pulo Turbinado: " .. tostring(jumpPower)
end
setSlider(jumpPower)

sliderKnob.MouseButton1Down:Connect(function() sliderDragging = true end)
UserInputService.InputEnded:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 or
       inp.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = false
    end
end)
RunService.RenderStepped:Connect(function()
    if sliderDragging then
        local mx = UserInputService:GetMouseLocation().X
        local bx = sliderBg.AbsolutePosition.X
        local bw = sliderBg.AbsoluteSize.X
        setSlider(JUMP_MIN + math.clamp((mx - bx) / bw, 0, 1) * (JUMP_MAX - JUMP_MIN))
    end
end)

applyJumpBtn.MouseButton1Click:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.JumpPower = jumpPower
        applyJumpBtn.Text = "✅ Aplicado!"
        task.delay(1.5, function() applyJumpBtn.Text = "💥 Aplicar Pulo" end)
    end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.3)
    char:WaitForChild("Humanoid").JumpPower = jumpPower
end)

-- ===================== FLY MANUAL =====================
plusBtn.MouseButton1Click:Connect(function()
    flySpeed = math.min(flySpeed + 10, 2000)
    flySpeedLbl.Text = "Velocidade Fly: " .. tostring(flySpeed)
end)
minusBtn.MouseButton1Click:Connect(function()
    flySpeed = math.max(flySpeed - 10, 10)
    flySpeedLbl.Text = "Velocidade Fly: " .. tostring(flySpeed)
end)

local manualBG, manualBV

local function stopFly()
    if not isFlying then return end
    isFlying = false
    flyStatusLbl.Text = "✈ Fly Manual: OFF  [E = voar]"
    if flyConn then flyConn:Disconnect(); flyConn = nil end
    pcall(function()
        if manualBG and manualBG.Parent then manualBG:Destroy() end
        if manualBV and manualBV.Parent then manualBV:Destroy() end
    end)
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = false end
    end
end

local function startFly()
    if isFlying then return end
    isFlying = true
    flyStatusLbl.Text = "✈ Fly Manual: ON"
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then hum.PlatformStand = true end

    manualBG = Instance.new("BodyGyro")
    manualBG.MaxTorque = Vector3.new(4e5, 4e5, 4e5)
    manualBG.P = 30000
    manualBG.CFrame = hrp.CFrame
    manualBG.Parent = hrp

    manualBV = Instance.new("BodyVelocity")
    manualBV.MaxForce = Vector3.new(4e5, 4e5, 4e5)
    manualBV.Velocity = Vector3.new(0, 0, 0)
    manualBV.Parent = hrp

    flyConn = RunService.RenderStepped:Connect(function()
        if not isFlying then stopFly() return end
        local cam = workspace.CurrentCamera.CFrame
        manualBG.CFrame = cam
        local dir = Vector3.new(0, 0, 0)
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.RightVector end
        manualBV.Velocity = dir * flySpeed
    end)
end

-- ===================== ROTA GENÉRICA (ida ou volta) =====================
local ROUTE_SPEED = 500
local ARRIVE_DIST = 12

local function cancelRoute()
    autoRoute = false
    if routeConn then routeConn:Disconnect(); routeConn = nil end
    local char = LocalPlayer.Character
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            for _, v in ipairs(hrp:GetChildren()) do
                if v:IsA("BodyMover") then v:Destroy() end
            end
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = false end
    end
    routeBtn.Text = "🏝 Ir à Ilha Final"
    voltaBtn.Text = "🏠 Voltar ao Safe"
end

-- order: lista de índices dos waypoints na ordem a percorrer
local function startRoute(order, btnAtivo, labelFim)
    if autoRoute then cancelRoute() end

    autoRoute = true
    btnAtivo.Text = "⏹ Cancelar"

    local char = LocalPlayer.Character
    if not char then cancelRoute() return end
    local hrp = char:WaitForChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")

    stopFly()

    for _, v in ipairs(hrp:GetChildren()) do
        if v:IsA("BodyMover") then v:Destroy() end
    end

    if hum then hum.PlatformStand = true end

    local routeBG = Instance.new("BodyGyro")
    routeBG.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    routeBG.P = 50000
    routeBG.D = 1000
    routeBG.CFrame = hrp.CFrame
    routeBG.Parent = hrp

    local routeBV = Instance.new("BodyVelocity")
    routeBV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    routeBV.Velocity = Vector3.new(0, 0, 0)
    routeBV.Parent = hrp

    local step = 1

    routeConn = RunService.Heartbeat:Connect(function()
        if not autoRoute then
            pcall(function() routeBG:Destroy(); routeBV:Destroy() end)
            if hum then hum.PlatformStand = false end
            if routeConn then routeConn:Disconnect(); routeConn = nil end
            return
        end

        if step > #order then
            pcall(function() routeBG:Destroy(); routeBV:Destroy() end)
            if hum then hum.PlatformStand = false end
            if routeConn then routeConn:Disconnect(); routeConn = nil end
            autoRoute = false
            routeStatusLbl.Text = labelFim
            routeBtn.Text = "🏝 Ir à Ilha Final"
            voltaBtn.Text = "🏠 Voltar ao Safe"
            return
        end

        local wp      = waypoints[order[step]]
        local pos     = hrp.Position
        local toTarget = wp.pos - pos
        local dist    = toTarget.Magnitude

        routeStatusLbl.Text = "🚀 Lock " .. order[step] .. " — " .. math.floor(dist) .. " studs"

        if dist < ARRIVE_DIST then
            routeBV.Velocity = Vector3.new(0, 0, 0)
            routeBG.CFrame = CFrame.lookAt(pos, wp.look)
            step = step + 1
        else
            local dir = toTarget.Unit
            routeBV.Velocity = dir * ROUTE_SPEED
            routeBG.CFrame = CFrame.lookAt(pos, pos + dir)
        end
    end)
end

-- Botão IDA: Lock1 → Lock2 → Lock3
routeBtn.MouseButton1Click:Connect(function()
    if autoRoute then
        cancelRoute()
        routeStatusLbl.Text = "Rota cancelada."
    else
        startRoute({1, 2, 3}, routeBtn, "✅ Chegou na Ilha Final!")
    end
end)

-- Botão VOLTA: Lock3 → Lock2 → Lock1
voltaBtn.MouseButton1Click:Connect(function()
    if autoRoute then
        cancelRoute()
        routeStatusLbl.Text = "Rota cancelada."
    else
        startRoute({3, 2, 1}, voltaBtn, "🏠 Chegou no Safe!")
    end
end)

-- ===================== NOCLIP =====================
RunService.Stepped:Connect(function()
    if noclipEnabled then
        local char = LocalPlayer.Character
        if char then
            for _, p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") then
                    pcall(function() p.CanCollide = false end)
                end
            end
        end
    end
end)

-- ===================== INPUT =====================
UserInputService.InputBegan:Connect(function(inp, gp)
    if gp then return end
    if inp.KeyCode == Enum.KeyCode.E then startFly() end
    if inp.KeyCode == Enum.KeyCode.G then noclipEnabled = not noclipEnabled end
    if inp.KeyCode == Enum.KeyCode.Insert then win.Visible = not win.Visible end
end)

UserInputService.InputEnded:Connect(function(inp)
    if inp.KeyCode == Enum.KeyCode.E then stopFly() end
end)

toggleBtn.MouseButton1Click:Connect(function() win.Visible = not win.Visible end)
closeWinBtn.MouseButton1Click:Connect(function() win.Visible = false end)

-- ===================== RESPAWN =====================
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.3)
    isFlying  = false
    autoRoute = false
    flyStatusLbl.Text = "✈ Fly Manual: OFF  [E = voar]"
    routeStatusLbl.Text = "Rota: aguardando..."
    routeBtn.Text = "🏝 Ir à Ilha Final"
    voltaBtn.Text = "🏠 Voltar ao Safe"
    char:WaitForChild("Humanoid").JumpPower = jumpPower
end)

print("✅ Karateka Hub | E=fly | G=noclip | Insert=menu | Ida + Volta 500/s")
