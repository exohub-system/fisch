-- ╔═══════════════════════════════════╗
--   EXO HUB | FISCH
--   WindUI | discord.gg/6QzV9pTWs
-- ╚═══════════════════════════════════╝

local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage= game:GetService("ReplicatedStorage")
local StarterGui       = game:GetService("StarterGui")
local lp               = Players.LocalPlayer
local playerGui        = lp.PlayerGui

-- ══════════════════════════════════════
--  LOAD WINDUI
-- ══════════════════════════════════════
local WindUI = loadstring(game:HttpGet(
    "https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"
))()

-- ══════════════════════════════════════
--  STATE
-- ══════════════════════════════════════
local autoCastEnabled  = false
local autoSellEnabled  = false
local autoReelEnabled  = false
local flyActive        = false
local espActive        = false
local antiAfkEnabled   = false
local flySpeed         = 50
local flyConn          = nil
local espConn          = nil
local espObjects       = {}
local bodyGyro         = nil
local bodyVel          = nil
local mainLoop         = nil
local nowe             = false
local tpwalking        = false
local speeds           = 1

-- ══════════════════════════════════════
--  HELPERS
-- ══════════════════════════════════════
local function getChar() return lp.Character end
local function getHRP()  local c=getChar(); return c and c:FindFirstChild("HumanoidRootPart") end
local function getHum()  local c=getChar(); return c and c:FindFirstChildOfClass("Humanoid") end

local function notify(msg)
    WindUI:Notify({
        Title   = "EXO HUB",
        Content = msg,
        Duration = 3,
    })
end

-- ══════════════════════════════════════
--  GET CAST REMOTE
-- ══════════════════════════════════════
local function getCastRemote()
    local char = getChar()
    if not char then return end
    local tool = char:FindFirstChildOfClass("Tool")
    if not tool then return end
    local events = tool:FindFirstChild("events")
    if not events then return end
    return events:FindFirstChild("cast")
end

-- ══════════════════════════════════════
--  AUTO CAST
-- ══════════════════════════════════════
local function doCast()
    pcall(function()
        local remote = getCastRemote()
        if remote then
            remote:FireServer()
        end
    end)
end

-- ══════════════════════════════════════
--  AUTO REEL (minigame — snap playerbar to fish)
-- ══════════════════════════════════════
local reelConn = nil

local function startAutoReel()
    reelConn = RunService.Heartbeat:Connect(function()
        if not autoReelEnabled then return end
        pcall(function()
            local reel = playerGui:FindFirstChild("reel")
            if not reel then return end
            local bar = reel:FindFirstChild("bar")
            if not bar then return end
            local playerbar = bar:FindFirstChild("playerbar")
            local fish       = bar:FindFirstChild("fish")
            if playerbar and fish then
                playerbar.Position = fish.Position
            end
        end)
    end)
end

local function stopAutoReel()
    if reelConn then reelConn:Disconnect(); reelConn = nil end
end

-- ══════════════════════════════════════
--  AUTO SHAKE (click shake button)
-- ══════════════════════════════════════
local shakeConn = nil

local function startAutoShake()
    shakeConn = RunService.Heartbeat:Connect(function()
        if not autoCastEnabled then return end
        pcall(function()
            local shake = playerGui:FindFirstChild("shake")
            if not shake then return end
            for _, btn in ipairs(shake:GetDescendants()) do
                if btn:IsA("TextButton") or btn:IsA("ImageButton") then
                    btn.MouseButton1Click:Fire()
                end
            end
        end)
    end)
end

local function stopAutoShake()
    if shakeConn then shakeConn:Disconnect(); shakeConn = nil end
end

-- ══════════════════════════════════════
--  AUTO CAST LOOP (cast → wait for reel → cast again)
-- ══════════════════════════════════════
local function startAutoCast()
    task.spawn(function()
        while autoCastEnabled do
            pcall(function()
                -- cast
                doCast()
                -- wait for reel GUI to appear
                local t = 0
                while autoCastEnabled and t < 30 do
                    local reel = playerGui:FindFirstChild("reel")
                    if reel then break end
                    task.wait(0.2)
                    t += 0.2
                end
                -- wait for reel GUI to disappear (fish caught or lost)
                local t2 = 0
                while autoCastEnabled and t2 < 60 do
                    local reel = playerGui:FindFirstChild("reel")
                    if not reel then break end
                    task.wait(0.1)
                    t2 += 0.1
                end
                task.wait(0.5)
            end)
        end
    end)
end

-- ══════════════════════════════════════
--  AUTO SELL
-- ══════════════════════════════════════
local sellLoop = nil

local function startAutoSell()
    sellLoop = task.spawn(function()
        while autoSellEnabled do
            pcall(function()
                -- try Marc Merchant NPC sellall
                local merchant = workspace:FindFirstChild("world")
                    and workspace.world:FindFirstChild("npcs")
                    and workspace.world.npcs:FindFirstChild("Marc Merchant")
                    and workspace.world.npcs["Marc Merchant"]:FindFirstChild("merchant")
                    and workspace.world.npcs["Marc Merchant"].merchant:FindFirstChild("sellall")
                if merchant then
                    merchant:InvokeServer()
                end
                -- also try any sellall remote in ReplicatedStorage
                for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
                    if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
                        local n = v.Name:lower()
                        if n:find("sell") then
                            pcall(function()
                                if v:IsA("RemoteFunction") then
                                    v:InvokeServer()
                                else
                                    v:FireServer()
                                end
                            end)
                        end
                    end
                end
            end)
            task.wait(10)
        end
    end)
end

local function stopAutoSell()
    autoSellEnabled = false
end

-- ══════════════════════════════════════
--  FLY (using proper animation method)
-- ══════════════════════════════════════
local function startTpWalking()
    tpwalking = false
    spawn(function()
        local hb = RunService.Heartbeat
        tpwalking = true
        local chr = getChar()
        local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
        while tpwalking and hb:Wait() and chr and hum and hum.Parent do
            if hum.MoveDirection.Magnitude > 0 then
                chr:TranslateBy(hum.MoveDirection)
            end
        end
    end)
end

local function disableStates(hum)
    for _, s in pairs(Enum.HumanoidStateType:GetEnumItems()) do
        pcall(function() hum:SetStateEnabled(s, false) end)
    end
    hum:ChangeState(Enum.HumanoidStateType.Swimming)
end

local function enableStates(hum)
    for _, s in pairs(Enum.HumanoidStateType:GetEnumItems()) do
        pcall(function() hum:SetStateEnabled(s, true) end)
    end
    hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
end

local function startFly()
    local char = getChar()
    if not char then return end
    local hum = char:FindFirstChildWhichIsA("Humanoid")
    if not hum then return end
    local isR6   = hum.RigType == Enum.HumanoidRigType.R6
    local torso  = isR6 and char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
    if not torso then return end

    char.Animate.Disabled = true
    for _, t in pairs(hum:GetPlayingAnimationTracks()) do t:AdjustSpeed(0) end
    disableStates(hum)
    startTpWalking()
    hum.PlatformStand = true

    local bg = Instance.new("BodyGyro", torso)
    bg.P = 9e4
    bg.maxTorque = Vector3.new(9e9,9e9,9e9)
    bg.cframe = torso.CFrame

    local bv = Instance.new("BodyVelocity", torso)
    bv.velocity = Vector3.new(0,0.1,0)
    bv.maxForce = Vector3.new(9e9,9e9,9e9)

    local ctrl = {f=0,b=0,l=0,r=0}
    local lastctrl = {f=0,b=0,l=0,r=0}
    local maxspeed = flySpeed
    local spd = 0

    flyConn = RunService.RenderStepped:Connect(function()
        if not nowe then
            flyConn:Disconnect(); flyConn = nil
            bg:Destroy(); bv:Destroy()
            hum.PlatformStand = false
            char.Animate.Disabled = false
            tpwalking = false
            enableStates(hum)
            return
        end
        maxspeed = flySpeed
        ctrl.f = UserInputService:IsKeyDown(Enum.KeyCode.W) and 1 or 0
        ctrl.b = UserInputService:IsKeyDown(Enum.KeyCode.S) and -1 or 0
        ctrl.l = UserInputService:IsKeyDown(Enum.KeyCode.A) and -1 or 0
        ctrl.r = UserInputService:IsKeyDown(Enum.KeyCode.D) and 1 or 0
        if ctrl.l+ctrl.r ~= 0 or ctrl.f+ctrl.b ~= 0 then
            spd = math.min(spd+0.5+(spd/maxspeed), maxspeed)
        else
            spd = math.max(spd-1, 0)
        end
        local cam = workspace.CurrentCamera.CoordinateFrame
        if (ctrl.l+ctrl.r) ~= 0 or (ctrl.f+ctrl.b) ~= 0 then
            bv.velocity = ((cam.lookVector*(ctrl.f+ctrl.b))+((cam*CFrame.new(ctrl.l+ctrl.r,(ctrl.f+ctrl.b)*.2,0).p)-cam.p))*spd
            lastctrl = {f=ctrl.f,b=ctrl.b,l=ctrl.l,r=ctrl.r}
        elseif spd ~= 0 then
            bv.velocity = ((cam.lookVector*(lastctrl.f+lastctrl.b))+((cam*CFrame.new(lastctrl.l+lastctrl.r,(lastctrl.f+lastctrl.b)*.2,0).p)-cam.p))*spd
        else
            bv.velocity = Vector3.new(0,0,0)
        end
        bg.cframe = cam*CFrame.Angles(-math.rad((ctrl.f+ctrl.b)*50*spd/maxspeed),0,0)
    end)
end

local function stopFly()
    nowe = false
    tpwalking = false
end

-- ══════════════════════════════════════
--  ANTI AFK
-- ══════════════════════════════════════
local antiAfkConn = nil
local function startAntiAfk()
    antiAfkConn = task.spawn(function()
        while antiAfkEnabled do
            pcall(function()
                local vs = game:GetService("VirtualUser")
                vs:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                task.wait(0.1)
                vs:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            end)
            task.wait(60)
        end
    end)
end

-- ══════════════════════════════════════
--  SPEED
-- ══════════════════════════════════════
local function setSpeed(on)
    local hum = getHum()
    if hum then hum.WalkSpeed = on and 80 or 16 end
end

-- ══════════════════════════════════════
--  PLAYER ESP
-- ══════════════════════════════════════
local function clearESP()
    for _, o in pairs(espObjects) do pcall(function() o:Destroy() end) end
    espObjects = {}
end

local function startESP()
    espActive = true
    espConn = RunService.Heartbeat:Connect(function()
        if not espActive then clearESP(); return end
        clearESP()
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= lp and p.Character then
                local root = p.Character:FindFirstChild("HumanoidRootPart")
                if root then
                    local bb = Instance.new("BillboardGui")
                    bb.Size = UDim2.new(0,80,0,30)
                    bb.StudsOffset = Vector3.new(0,3,0)
                    bb.AlwaysOnTop = true
                    bb.Adornee = root
                    bb.Parent = workspace
                    local lbl = Instance.new("TextLabel")
                    lbl.Size = UDim2.new(1,0,1,0)
                    lbl.BackgroundTransparency = 1
                    lbl.Text = p.DisplayName
                    lbl.TextColor3 = Color3.fromRGB(0,180,255)
                    lbl.TextStrokeTransparency = 0
                    lbl.TextStrokeColor3 = Color3.fromRGB(0,0,0)
                    lbl.Font = Enum.Font.GothamBold
                    lbl.TextSize = 13
                    lbl.Parent = bb
                    local hl = Instance.new("SelectionBox")
                    hl.Adornee = p.Character
                    hl.Color3 = Color3.fromRGB(0,120,255)
                    hl.LineThickness = 0.04
                    hl.SurfaceTransparency = 0.85
                    hl.SurfaceColor3 = Color3.fromRGB(0,100,255)
                    hl.Parent = workspace
                    table.insert(espObjects, bb)
                    table.insert(espObjects, hl)
                end
            end
        end
    end)
end

local function stopESP()
    espActive = false
    if espConn then espConn:Disconnect(); espConn = nil end
    clearESP()
end

-- ══════════════════════════════════════
--  ISLAND TELEPORTS
-- ══════════════════════════════════════
local islands = {
    ["Spawn Island"]    = Vector3.new(0,    5,    0),
    ["Mushroom Island"] = Vector3.new(800,  5,    400),
    ["Winter Island"]   = Vector3.new(-600, 5,    800),
    ["Sandy Island"]    = Vector3.new(1200, 5,   -300),
    ["Deep Ocean"]      = Vector3.new(0,   -40,   0),
    ["Volcano Island"]  = Vector3.new(-900, 5,   -600),
    ["Ancient Ruins"]   = Vector3.new(400,  5,  -1000),
}

local function tpToIsland(name)
    local hrp = getHRP()
    if not hrp then return end
    local pos = islands[name]
    if pos then
        hrp.CFrame = CFrame.new(pos + Vector3.new(0,5,0))
        notify("Teleported to " .. name .. "! 🗺️")
    end
end

-- respawn
lp.CharacterAdded:Connect(function()
    task.wait(1)
    if nowe then nowe = false; task.wait(0.1); nowe = true; startFly() end
    if autoCastEnabled then startAutoCast(); startAutoShake() end
    if autoReelEnabled then startAutoReel() end
end)

-- ══════════════════════════════════════
--  WINDUI WINDOW
-- ══════════════════════════════════════
local Window = WindUI:CreateWindow({
    Title       = "EXO HUB",
    Icon        = "rbxassetid://71483961072989",
    Author      = "discord.gg/6QzV9pTWs",
    Folder      = "ExoHub",
    Size        = UDim2.fromOffset(580, 460),
    Transparent = true,
    Theme       = "Dark",
    Accent      = Color3.fromRGB(0, 85, 170),
})

-- ══════════════════════════════════════
--  TABS
-- ══════════════════════════════════════
local FishTab     = Window:Tab({ Title = "Fishing",   Icon = "fish" })
local TpTab       = Window:Tab({ Title = "Teleport",  Icon = "map-pin" })
local MovementTab = Window:Tab({ Title = "Movement",  Icon = "zap" })
local VisualsTab  = Window:Tab({ Title = "Visuals",   Icon = "eye" })
local MiscTab     = Window:Tab({ Title = "Misc",      Icon = "settings" })

-- ══════════════════════════════════════
--  FISHING TAB
-- ══════════════════════════════════════
local FishSection = FishTab:Section({ Title = "Auto Fishing" })

FishSection:Toggle({
    Title = "Auto Cast",
    Desc  = "Automatically casts your rod and catches fish",
    Icon  = "anchor",
    Value = false,
    Callback = function(state)
        autoCastEnabled = state
        autoReelEnabled = state
        if state then
            startAutoCast()
            startAutoShake()
            startAutoReel()
            notify("Auto Cast enabled 🎣")
        else
            stopAutoShake()
            stopAutoReel()
            notify("Auto Cast disabled")
        end
    end
})

FishSection:Toggle({
    Title = "Auto Reel",
    Desc  = "Snaps the reel bar to the fish — never lose a catch",
    Icon  = "circle",
    Value = false,
    Callback = function(state)
        autoReelEnabled = state
        if state then startAutoReel() else stopAutoReel() end
    end
})

FishSection:Toggle({
    Title = "Auto Sell",
    Desc  = "Sells all fish every 10 seconds automatically",
    Icon  = "shopping-cart",
    Value = false,
    Callback = function(state)
        autoSellEnabled = state
        if state then startAutoSell() end
    end
})

FishSection:Button({
    Title = "Sell Now",
    Desc  = "Instantly sells all fish in your inventory",
    Icon  = "dollar-sign",
    Callback = function()
        pcall(function()
            local merchant = workspace.world.npcs["Marc Merchant"].merchant.sellall
            if merchant then merchant:InvokeServer() end
        end)
        notify("Sold all fish! 💰")
    end
})

FishSection:Toggle({
    Title = "Anti AFK",
    Desc  = "Prevents you from being kicked while AFK fishing",
    Icon  = "clock",
    Value = false,
    Callback = function(state)
        antiAfkEnabled = state
        if state then startAntiAfk() end
    end
})

-- ══════════════════════════════════════
--  TELEPORT TAB
-- ══════════════════════════════════════
local TpSection = TpTab:Section({ Title = "Islands" })

for name, _ in pairs(islands) do
    TpSection:Button({
        Title = name,
        Desc  = "Teleport to " .. name,
        Icon  = "map-pin",
        Callback = function()
            tpToIsland(name)
        end
    })
end

-- ══════════════════════════════════════
--  MOVEMENT TAB
-- ══════════════════════════════════════
local MovSection = MovementTab:Section({ Title = "Movement" })

MovSection:Toggle({
    Title = "Fly",
    Desc  = "WASD to fly  •  smooth acceleration",
    Icon  = "plane",
    Value = false,
    Callback = function(state)
        nowe = state
        if state then startFly() else stopFly() end
    end
})

MovSection:Slider({
    Title = "Fly Speed",
    Desc  = "How fast you fly",
    Icon  = "gauge",
    Min   = 10,
    Max   = 200,
    Value = 50,
    Callback = function(val)
        flySpeed = val
    end
})

MovSection:Toggle({
    Title = "Speed Hack",
    Desc  = "Boosts walk speed to 80",
    Icon  = "zap",
    Value = false,
    Callback = function(state)
        setSpeed(state)
    end
})

-- ══════════════════════════════════════
--  VISUALS TAB
-- ══════════════════════════════════════
local VisSection = VisualsTab:Section({ Title = "ESP" })

VisSection:Toggle({
    Title = "Player ESP",
    Desc  = "See all players through walls",
    Icon  = "eye",
    Value = false,
    Callback = function(state)
        if state then startESP() else stopESP() end
    end
})

-- ══════════════════════════════════════
--  MISC TAB
-- ══════════════════════════════════════
local MiscSection = MiscTab:Section({ Title = "EXO HUB" })

MiscSection:Paragraph({
    Title = "EXO HUB — Fisch",
    Desc  = "Free forever. Always updating.\ndiscord.gg/6QzV9pTWs",
})

MiscSection:Button({
    Title    = "Join Discord",
    Desc     = "discord.gg/6QzV9pTWs",
    Icon     = "message-circle",
    Callback = function()
        notify("Join us at discord.gg/6QzV9pTWs 🔥")
    end
})

-- startup notification
task.wait(1)
notify("EXO HUB loaded! 🔥 discord.gg/6QzV9pTWs")

print("[EXO HUB] Fisch loaded | discord.gg/6QzV9pTWs")
