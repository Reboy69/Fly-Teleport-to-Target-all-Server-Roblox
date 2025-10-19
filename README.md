if _G.CoreScriptExecuted then return end
_G.CoreScriptExecuted = true

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")

-- Player vars
local lp = Players.LocalPlayer
local char, hrp, hum
local flySpeed = 200
local flying, manualFlying = false, false
local bv, bg
local currentVelocity = Vector3.new(0,0,0)
local minHeightAboveGround = 10

-- Follow vars
local namaTeman = "Pilih Nama"
local followAktif, autoTeleport = false, false
local followDistance, stopThreshold = 3, 0.5
local lastTargetPos = nil

-- Fly buttons vars
local popupUpBtn, popupDownBtn
local holdUp, holdDown = false,false
local noclip = false

-- Setup character
local function setupCharacter(c)
    char = c
    hrp = char:WaitForChild("HumanoidRootPart")
    hum = char:WaitForChild("Humanoid")
end
if lp.Character then setupCharacter(lp.Character) end
lp.CharacterAdded:Connect(setupCharacter)

-- GUI
local sg = Instance.new("ScreenGui", lp:WaitForChild("PlayerGui"))
sg.Name = "Reboy69GUI"
sg.ResetOnSpawn = false

local mainGui = Instance.new("Frame", sg)
mainGui.Size = UDim2.new(0,200,0,300)
mainGui.Position = UDim2.new(0,10,0.5,-150)
mainGui.BackgroundColor3 = Color3.fromRGB(30,30,35)
mainGui.BackgroundTransparency = 0.1
mainGui.BorderSizePixel = 0
mainGui.ClipsDescendants = true
mainGui.Active = true
mainGui.Draggable = true

local title = Instance.new("TextLabel", mainGui)
title.Size = UDim2.new(1,0,0,20)
title.Position = UDim2.new(0,0,0,0)
title.Text = "👑 Reboy69"
title.TextColor3 = Color3.fromRGB(255,215,0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBlack
title.TextSize = 16
title.TextStrokeTransparency = 0.6

-- Dropdown pilih teman
local dropdown = Instance.new("TextButton", mainGui)
dropdown.Size = UDim2.new(1,-16,0,25)
dropdown.Position = UDim2.new(0,8,0,25)
dropdown.Text = "🔽 Pilih Nama Teman"
dropdown.BackgroundColor3 = Color3.fromRGB(50,50,55)
dropdown.TextColor3 = Color3.fromRGB(255,255,255)
dropdown.Font = Enum.Font.GothamBold
dropdown.TextSize = 14
dropdown.BorderSizePixel = 0

local listFrame = Instance.new("ScrollingFrame", mainGui)
listFrame.Size = UDim2.new(1,-16,0,70)
listFrame.Position = UDim2.new(0,8,0,50)
listFrame.BackgroundTransparency = 0.3
listFrame.BackgroundColor3 = Color3.fromRGB(40,40,45)
listFrame.BorderSizePixel = 0
listFrame.CanvasSize = UDim2.new(0,0,0,0)
listFrame.ScrollBarThickness = 5
listFrame.Visible = false

dropdown.MouseButton1Click:Connect(function()
    listFrame.Visible = not listFrame.Visible
end)

-- Update daftar pemain
local function updatePlayerList()
    listFrame:ClearAllChildren()
    local allPlayers={}
    for _,p in ipairs(Players:GetPlayers()) do
        if p~=lp then table.insert(allPlayers,p.Name) end
    end
    table.sort(allPlayers)
    local y=0
    for _,name in ipairs(allPlayers) do
        local btn=Instance.new("TextButton", listFrame)
        btn.Size=UDim2.new(1,-4,0,20)
        btn.Position=UDim2.new(0,2,0,y)
        btn.Text=name
        btn.BackgroundColor3=Color3.fromRGB(60,60,60)
        btn.TextColor3=Color3.fromRGB(255,255,255)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 12
        btn.BorderSizePixel = 0
        btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(80,80,80) end)
        btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(60,60,60) end)
        btn.MouseButton1Click:Connect(function()
            namaTeman=name
            dropdown.Text="🎯 "..name
            listFrame.Visible=false
        end)
        -- style
        local corner = Instance.new("UICorner", btn)
        corner.CornerRadius = UDim.new(0, 6)
        local stroke = Instance.new("UIStroke", btn)
        stroke.Thickness = 1
        stroke.Color = Color3.fromRGB(90,90,110)
        y=y+24
    end
    listFrame.CanvasSize=UDim2.new(0,0,0,y)
end
updatePlayerList()
Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)

-- Tombol utama
local flyBtn = Instance.new("TextButton", mainGui)
local followBtn = Instance.new("TextButton", mainGui)
local autoTpBtn = Instance.new("TextButton", mainGui)
local tpBtn = Instance.new("TextButton", mainGui)
local noclipBtn = Instance.new("TextButton", mainGui)

flyBtn.Text = "🕊️ Fly [OFF]"
followBtn.Text = "🔁 Ikuti "..namaTeman.." [OFF]"
autoTpBtn.Text = "⚙️ Auto Teleport [OFF]"
tpBtn.Text = "⚡ Teleport ke "..namaTeman
noclipBtn.Text = "🚫 Noclip [OFF]"

-- Styling
local function styleButton(btn)
    btn.BackgroundColor3 = Color3.fromRGB(60,60,70)
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0, 8)
    local stroke = Instance.new("UIStroke", btn)
    stroke.Thickness = 1
    stroke.Color = Color3.fromRGB(90, 90, 110)
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(80,80,100) end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(60,60,70) end)
end
for _,btn in ipairs({flyBtn, followBtn, autoTpBtn, tpBtn, noclipBtn, dropdown}) do
    styleButton(btn)
end

local function styleFrame(frame)
    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0, 10)
    local stroke = Instance.new("UIStroke", frame)
    stroke.Thickness = 1.5
    stroke.Color = Color3.fromRGB(100, 100, 120)
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    local padding = Instance.new("UIPadding", frame)
    padding.PaddingTop = UDim.new(0, 4)
    padding.PaddingBottom = UDim.new(0, 4)
    padding.PaddingLeft = UDim.new(0, 6)
    padding.PaddingRight = UDim.new(0, 6)
end
styleFrame(mainGui)
styleFrame(listFrame)

-- Posisi tombol
local yPos = 130
for _,btn in ipairs({followBtn, autoTpBtn, flyBtn, tpBtn, noclipBtn}) do
    btn.Position = UDim2.new(0,8,0,yPos)
    btn.Size = UDim2.new(1,-16,0,25)
    yPos = yPos + 30
end
mainGui.Size = UDim2.new(0,200,0,yPos+8)

-- Fly buttons HP
local function createFlyButtonsHP()
    if not popupUpBtn then
        popupUpBtn = Instance.new("TextButton", sg)
        popupUpBtn.Size=UDim2.new(0,60,0,60)
        popupUpBtn.Position=UDim2.new(1,-150,0.5,-50)
        popupUpBtn.Text="⬆️"
        popupUpBtn.BackgroundTransparency = 1
        popupUpBtn.TextColor3=Color3.new(1,1,1)
        popupUpBtn.ZIndex = 10
        popupUpBtn.MouseButton1Down:Connect(function() holdUp=true end)
        popupUpBtn.MouseButton1Up:Connect(function() holdUp=false end)
        popupUpBtn.MouseLeave:Connect(function() holdUp=false end)
    end
    popupUpBtn.Visible = true
    if not popupDownBtn then
        popupDownBtn = Instance.new("TextButton", sg)
        popupDownBtn.Size=UDim2.new(0,60,0,60)
        popupDownBtn.Position=UDim2.new(1,-150,0.5,0)
        popupDownBtn.Text="⬇️"
        popupDownBtn.BackgroundTransparency = 1
        popupDownBtn.TextColor3=Color3.new(1,1,1)
        popupDownBtn.ZIndex = 10
        popupDownBtn.MouseButton1Down:Connect(function() holdDown=true end)
        popupDownBtn.MouseButton1Up:Connect(function() holdDown=false end)
        popupDownBtn.MouseLeave:Connect(function() holdDown=false end)
    end
    popupDownBtn.Visible = true
end
local function destroyFlyButtonsHP()
    if popupUpBtn then popupUpBtn.Visible=false end
    if popupDownBtn then popupDownBtn.Visible=false end
end

-- Fly
local function startFlying(manual)
    if not hum or not hrp then return end
    flying = true
    manualFlying = manual or false
    if bv then bv:Destroy() end
    if bg then bg:Destroy() end
    bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(1e5,1e5,1e5)
    bv.Velocity = Vector3.new(0,0,0)
    bv.Parent = hrp
    bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(1e5,1e5,1e5)
    bg.P = 1e4
    bg.CFrame = hrp.CFrame
    bg.Parent = hrp
    createFlyButtonsHP()
end
local function stopFlying()
    flying = false
    manualFlying = false
    if bv then bv:Destroy() end
    if bg then bg:Destroy() end
    destroyFlyButtonsHP()
end

-- Button actions
flyBtn.MouseButton1Click:Connect(function()
    if flying then stopFlying() else startFlying(true) end
end)
followBtn.MouseButton1Click:Connect(function()
    followAktif = not followAktif
end)
autoTpBtn.MouseButton1Click:Connect(function()
    autoTeleport = not autoTeleport
end)
tpBtn.MouseButton1Click:Connect(function()
    local p = Players:FindFirstChild(namaTeman)
    if p and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
        hrp.CFrame = CFrame.new(p.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.CFrame.LookVector)
    end
end)
noclipBtn.MouseButton1Click:Connect(function()
    noclip = not noclip
end)

-- Toggle GUI buka/tutup
local menuToggleBtn = Instance.new("TextButton", sg)
menuToggleBtn.Size = UDim2.new(0,50,0,50)
menuToggleBtn.Position = UDim2.new(0,10,1,-60)
menuToggleBtn.Text = "🕹️"
menuToggleBtn.BackgroundTransparency = 1
menuToggleBtn.TextColor3 = Color3.fromRGB(255, 215, 0)
menuToggleBtn.Font = Enum.Font.GothamBlack
menuToggleBtn.TextSize = 24
menuToggleBtn.ZIndex = 10

local menuTweenInfo = TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
menuToggleBtn.MouseButton1Click:Connect(function()
    if mainGui.Visible then
        local tween = TweenService:Create(mainGui, menuTweenInfo, {Position = UDim2.new(0,-250, mainGui.Position.Y.Scale, mainGui.Position.Y.Offset)})
        tween:Play()
        tween.Completed:Connect(function() mainGui.Visible=false end)
    else
        mainGui.Visible = true
        local tween = TweenService:Create(mainGui, menuTweenInfo, {Position = UDim2.new(0,10, mainGui.Position.Y.Scale, mainGui.Position.Y.Offset)})
        tween:Play()
    end
end)

-- Main loop
RunService.RenderStepped:Connect(function(dt)
    if not hrp or not hum then return end
    local yVel = 0
    if holdUp then yVel = flySpeed*0.5 elseif holdDown then yVel = -flySpeed*0.5 end

    local followVel = Vector3.new(0,0,0)
    local targetHRP = nil
    local moving = false

    local p = Players:FindFirstChild(namaTeman)
    if followAktif and p and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
        targetHRP = p.Character.HumanoidRootPart
        local targetPos = targetHRP.Position
        local desiredPos = targetPos - targetHRP.CFrame.LookVector * followDistance

        if not manualFlying then
            local rayOrigin = Vector3.new(desiredPos.X, targetPos.Y + 50, desiredPos.Z)
            local rayDirection = Vector3.new(0, -200, 0)
            local rayParams = RaycastParams.new()
            rayParams.FilterDescendantsInstances = {char, p.Character}
            rayParams.FilterType = Enum.RaycastFilterType.Blacklist
            local rayResult = Workspace:Raycast(rayOrigin, rayDirection, rayParams)
            if rayResult then
                desiredPos = Vector3.new(desiredPos.X, rayResult.Position.Y + minHeightAboveGround + 1, desiredPos.Z)
            end
        end

        local dist = (desiredPos - hrp.Position).Magnitude
        if lastTargetPos then
            moving = (targetPos - lastTargetPos).Magnitude > 0.05
        else
            moving = true
        end
        lastTargetPos = targetPos

        if autoTeleport and dist > 15 then
            hrp.CFrame = CFrame.new(desiredPos)
            followVel = Vector3.new(0,0,0)
        elseif moving and dist > stopThreshold then
            local newPos = hrp.Position:Lerp(desiredPos, 0.2)
            followVel = (newPos - hrp.Position)/dt
        else
            followVel = Vector3.new(0,0,0)
        end
    else
        lastTargetPos = nil
    end

    -- Flying
    if flying and bv and bg then
        if followAktif and targetHRP then
            local targetDir = (targetHRP.Position - hrp.Position).Unit
            bg.CFrame = CFrame.new(hrp.Position, hrp.Position + targetDir)
        else
            local camCF = workspace.CurrentCamera.CFrame
            local moveDir = hum.MoveDirection
            local targetVelocity = Vector3.new(0,0,0)
            if moveDir.Magnitude>0 then
                local forwardDot = camCF.LookVector:Dot(moveDir.Unit)
                if forwardDot>0 then
                    targetVelocity = camCF.LookVector*flySpeed + Vector3.new(0,flySpeed*0.6,0)
                else
                    targetVelocity = moveDir*flySpeed
                end
            end
            currentVelocity = currentVelocity:Lerp(targetVelocity,0.15)
            bg.CFrame = CFrame.new(hrp.Position, hrp.Position + camCF.LookVector)
        end
        bv.Velocity = currentVelocity + Vector3.new(followVel.X,yVel,followVel.Z)
    elseif not flying then
        if followAktif and followVel.Magnitude>0 then
            hum:Move(Vector3.new(followVel.X,0,followVel.Z))
        end
    end

    -- Noclip
    if noclip and char then
        for _,v in pairs(char:GetDescendants()) do
            if v:IsA("BasePart") and v.CanCollide then
                v.CanCollide = false
            end
        end
    end

    -- Reset jika teman keluar
    if namaTeman ~= "Pilih Nama" and not Players:FindFirstChild(namaTeman) then
        namaTeman = "Pilih Nama"
        dropdown.Text = "🔽 Pilih Nama Teman"
        lastTargetPos = nil
    end

    -- Update teks tombol
    followBtn.Text = followAktif and "🔁 Ikuti "..namaTeman.." [ON]" or "🔁 Ikuti "..namaTeman.." [OFF]"
    tpBtn.Text = "⚡ Teleport ke "..namaTeman
    autoTpBtn.Text = autoTeleport and "⚙️ Auto Teleport [ON]" or "⚙️ Auto Teleport [OFF]"
    flyBtn.Text = flying and "🕊️ Fly [ON]" or "🕊️ Fly [OFF]"
    noclipBtn.Text = noclip and "🚫 Noclip [ON]" or "🚫 Noclip [OFF]"
end)
