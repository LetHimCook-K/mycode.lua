# mycode.lualocal ReplicatedStorage = game:GetService("ReplicatedStorage")
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
WindUI:AddTheme({
    Name = "Lunar",
    Background = Color3.fromHex("#1a2a57"),
    Notification = Color3.fromHex("#101010"),

    NotificationTitle = Color3.fromHex("#FFFFFF"),
    NotificationTitleTransparency = 0,

    NotificationContent = Color3.fromHex("#D6E4FF"),
    NotificationContentTransparency = 0,

    NotificationDuration = Color3.fromHex("#4F7DFF"),
    NotificationDurationTransparency = 0,

    NotificationBorder = Color3.fromHex("#4F7DFF"),
    NotificationBorderTransparency = 0.35,
})
local Window = WindUI:CreateWindow({
    Title = "TZ HUB X LUNAR HUB || Warden",
    Folder = "TZHub",
    Icon = "solar:compass-big-bold",
    Theme = "Lunar",
    Transparent = true,
    NewElements = true,
    HideSearchBar = false,
    Background = "rbxassetid://92245353940912",
    BackgroundImageTransparency = 0.42,
})
Window:EditOpenButton({
    Title = "TZ HUB X LUNAR HUB || Warden",
    Icon = "solar:compass-big-bold", -- matches your main window icon
    CornerRadius = UDim.new(0,16),
    StrokeThickness = 2,
    Color = ColorSequence.new( -- gradient theme
        Color3.fromHex("DC143C"), -- Crimson start
        Color3.fromHex("8B0000")  -- Darker Crimson end
    ),
    OnlyMobile = false,
    Enabled = true,
    Draggable = true,
})

local Players      = game:GetService("Players")
local RunService   = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local lp  = Players.LocalPlayer
local cam = workspace.CurrentCamera
if setsimulationradius then
    setsimulationradius(9e9)
end
local flyEnabled  = false
local FLY_SPEED   = 50
local _flyConn    = nil
local _bodyVel    = nil
local _bodyGyro   = nil


local function cleanFly()
    if _flyConn  then _flyConn:Disconnect(); _flyConn  = nil end
    if _bodyVel  then _bodyVel:Destroy();    _bodyVel  = nil end
    if _bodyGyro then _bodyGyro:Destroy();   _bodyGyro = nil end
    local char = lp.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = false end
    end
end

local function startFly()
    cleanFly()
    local char = lp.Character
    local hrp  = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then hum.PlatformStand = true end

    _bodyVel = Instance.new("BodyVelocity")
    _bodyVel.Velocity  = Vector3.zero
    _bodyVel.MaxForce  = Vector3.new(1e5, 1e5, 1e5)
    _bodyVel.Parent    = hrp

    _bodyGyro = Instance.new("BodyGyro")
    _bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    _bodyGyro.P         = 1e4
    _bodyGyro.CFrame    = hrp.CFrame
    _bodyGyro.Parent    = hrp

    _flyConn = RunService.RenderStepped:Connect(function()
        local hrp2 = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
        if not hrp2 or not _bodyVel or not _bodyVel.Parent then return end

        local cf      = cam.CFrame
        local forward = cf.LookVector          -- W = forward INTO screen
        local right   = cf.RightVector
        local up      = Vector3.new(0, 1, 0)

        local move = Vector3.zero

        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            move = move + forward
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            move = move - forward
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            move = move - right
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            move = move + right
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) or UserInputService:IsKeyDown(Enum.KeyCode.E) then
            move = move + up
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.Q) then
            move = move - up
        end

        pcall(function()
            local controls = require(lp.PlayerScripts:WaitForChild("PlayerModule")):GetControls()
            local ms = controls:GetMoveVector()
            if ms.Magnitude > 0.05 then
                move = move + (right * ms.X) + (forward * -ms.Z)
            end
        end)

        if move.Magnitude > 1 then move = move.Unit end

        _bodyVel.Velocity  = move * FLY_SPEED
        _bodyGyro.CFrame   = cf
    end)
end

lp.CharacterAdded:Connect(function()
    task.wait(1)
    if flyEnabled then startFly() end
end)


local PlayerTab = Window:Tab({
    Title = "Player",
    Icon  = "solar:running-round-bold",
})


PlayerTab:Toggle({
    Title    = "Enable Fly",
    Icon     = "bird",
    Value    = false,
    Callback = function(state)
        flyEnabled = state
        if state then
            startFly()
        else
            cleanFly()
        end
    end,
})


PlayerTab:Slider({
    Title = "Fly Speed",
    Value = {
        Min     = 10,
        Max     = 100,
        Default = 50,
    },
    Step      = 1,
    IsTooltip = true,
    Callback  = function(v)
        FLY_SPEED = v
    end,
})

local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")

local lp = Players.LocalPlayer


local noclipEnabled = false
local noclipConn    = nil

local function setNoclip(state)
    noclipEnabled = state
    if state then
        noclipConn = RunService.Stepped:Connect(function()
            local char = lp.Character
            if not char then return end
            for _, p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") then
                    p.CanCollide = false
                end
            end
        end)
    else
        if noclipConn then noclipConn:Disconnect(); noclipConn = nil end
        local char = lp.Character
        if char then
            for _, p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") then
                    p.CanCollide = true
                end
            end
        end
    end
end

lp.CharacterAdded:Connect(function()
    task.wait(1)
    if noclipEnabled then setNoclip(true) end
end)


PlayerTab:Toggle({
    Title    = "NoClip",
    Icon     = "ghost",
    Value    = false,
    Callback = function(v)
        setNoclip(v)
    end,
})


local ClientStaminaTracker = require(ReplicatedStorage._replicationFolder.ClientStaminaTracker)

local staminaConfig = {
	setValue = 1000,
	lockEnabled = false,
}

PlayerTab:Input({
	Title = "Set Stamina",
	Placeholder = "1000",
	Callback = function(Value)
		Value = tonumber(Value)
		if Value then
			staminaConfig.setValue = Value
		end
	end,
})

PlayerTab:Toggle({
	Title = "Lock Stamina",
	Value = false,
	Callback = function(Value)
		staminaConfig.lockEnabled = Value
	end,
})

RunService.RenderStepped:Connect(function()
	if not staminaConfig.lockEnabled then
		return
	end

	local success, tracker = pcall(function()
		return ClientStaminaTracker.GetTracker()
	end)

	if success and tracker then
		if tracker.Stamina ~= staminaConfig.setValue then
			tracker.Stamina = staminaConfig.setValue

			if tracker.StaminaChangedSignal then
				tracker.StaminaChangedSignal:Fire(tracker.Stamina)
			end
		end
	end
end)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local EspParent = (gethui and gethui()) or game:GetService("CoreGui")

local EspTab = Window:Tab({
    Title = "ESP",
})

local CONFIG = {
    Players = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(0, 255, 255),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Enemies = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(255, 0, 0),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Items = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(255, 255, 0),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Doors = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(255, 140, 0),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Objectives = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(255, 0, 255),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Loot = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(0, 255, 0),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Interactables = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(0, 170, 255),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
    Exit = {
        Enabled = false,
        Name = true,
        Highlight = true,
        Tracer = false,
        TextSize = 16,
        Color = Color3.fromRGB(255, 255, 255),
        TextColor = Color3.fromRGB(255, 255, 255),
    },
}

local tracked = {}

local function getRoot(inst)
    if not inst then
        return nil
    end

    if inst:IsA("BasePart") then
        return inst
    end

    if inst:IsA("Model") then
        return inst:FindFirstChild("HumanoidRootPart")
            or inst.PrimaryPart
            or inst:FindFirstChildWhichIsA("BasePart", true)
    end

    return inst:FindFirstChildWhichIsA("BasePart", true)
end

local function getEnemyName(modelName)
    local name = modelName:match("^Corrupted(.-)%-")
    return name or modelName
end

local function matchKeyword(name, words)
    name = string.lower(name)
    for _, word in ipairs(words) do
        if string.find(name, string.lower(word), 1, true) then
            return true
        end
    end
    return false
end

local function cleanupEntry(key)
    local entry = tracked[key]
    if not entry then
        return
    end

    if entry.Gui then
        entry.Gui:Destroy()
    end

    if entry.Highlight then
        entry.Highlight:Destroy()
    end

    if entry.Tracer then
        pcall(function()
            entry.Tracer:Remove()
        end)
    end

    tracked[key] = nil
end

local function makeBillboard(entry)
    local adornee = getRoot(entry.Instance)
    if not adornee then
        return
    end

    local gui = Instance.new("BillboardGui")
    gui.Name = "ESP_Billboard"
    gui.AlwaysOnTop = true
    gui.Size = UDim2.fromOffset(250, 50)
    gui.StudsOffset = Vector3.new(0, 3, 0)
    gui.Adornee = adornee
    gui.Parent = EspParent

    local label = Instance.new("TextLabel")
    label.BackgroundTransparency = 1
    label.Size = UDim2.fromScale(1, 1)
    label.Font = Enum.Font.SourceSansBold
    label.TextScaled = false
    label.TextSize = entry.Config.TextSize
    label.TextStrokeTransparency = 0.3
    label.TextColor3 = entry.Config.TextColor
    label.Text = entry.Label
    label.Parent = gui

    entry.Gui = gui
    entry.LabelObject = label
end

local function makeHighlight(entry)
    local h = Instance.new("Highlight")
    h.Name = "ESP_Highlight"
    h.Adornee = entry.Instance
    h.FillColor = entry.Config.Color
    h.OutlineColor = entry.Config.Color
    h.FillTransparency = 0.65
    h.OutlineTransparency = 0
    h.Parent = EspParent
    entry.Highlight = h
end

local function makeTracer(entry)
    if not Drawing then
        return
    end

    local line = Drawing.new("Line")
    line.Visible = false
    line.Thickness = 1
    line.Transparency = 1
    line.Color = entry.Config.Color
    entry.Tracer = line
end

local function createEntry(key, inst, cfg, labelText)
    if not cfg.Enabled or not inst or tracked[key] then
        return
    end

    local entry = {
        Instance = inst,
        Config = cfg,
        Label = labelText,
    }

    tracked[key] = entry

    if cfg.Name then
        makeBillboard(entry)
    end

    if cfg.Highlight then
        makeHighlight(entry)
    end

    if cfg.Tracer then
        makeTracer(entry)
    end

    inst.AncestryChanged:Connect(function(_, parent)
        if not parent then
            cleanupEntry(key)
        end
    end)
end

local function updateEntry(key)
    local entry = tracked[key]
    if not entry then
        return
    end

    local inst = entry.Instance
    if not inst or not inst.Parent then
        cleanupEntry(key)
        return
    end

    local root = getRoot(inst)
    if entry.Gui then
        entry.Gui.Enabled = root ~= nil
        if root then
            entry.Gui.Adornee = root
        end
        if entry.LabelObject then
            entry.LabelObject.Text = entry.Label
            entry.LabelObject.TextSize = entry.Config.TextSize
            entry.LabelObject.TextColor3 = entry.Config.TextColor
        end
    end

    if entry.Highlight then
        entry.Highlight.Adornee = inst
        entry.Highlight.FillColor = entry.Config.Color
        entry.Highlight.OutlineColor = entry.Config.Color
        entry.Highlight.Enabled = root ~= nil
    end

    if entry.Tracer and Camera and root then
        local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
        if onScreen then
            entry.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
            entry.Tracer.To = Vector2.new(pos.X, pos.Y)
            entry.Tracer.Color = entry.Config.Color
            entry.Tracer.Visible = true
        else
            entry.Tracer.Visible = false
        end
    end
end

local function setFeature(key, state)
    CONFIG[key].Enabled = state

    if not state then
        for entryKey, entry in pairs(tracked) do
            if entry.Config == CONFIG[key] then
                cleanupEntry(entryKey)
            end
        end
    end
end

local function featureSection(title, key)
    local section = EspTab:Section({
        Title = title,
    })

    section:Toggle({
        Title = "Enable",
        Value = false,
        Callback = function(v)
            setFeature(key, v)
        end,
    })

    section:Toggle({
        Title = "Name",
        Value = CONFIG[key].Name,
        Callback = function(v)
            CONFIG[key].Name = v
        end,
    })

    section:Toggle({
        Title = "Highlight",
        Value = CONFIG[key].Highlight,
        Callback = function(v)
            CONFIG[key].Highlight = v
        end,
    })

    section:Toggle({
        Title = "Tracer",
        Value = CONFIG[key].Tracer,
        Callback = function(v)
            CONFIG[key].Tracer = v
        end,
    })

    section:Slider({
        Title = "Text Size",
        Value = {
            Min = 10,
            Max = 40,
            Default = CONFIG[key].TextSize,
        },
        Step = 1,
        IsTooltip = true,
        Callback = function(v)
            CONFIG[key].TextSize = v
        end,
    })

    section:Colorpicker({
        Title = "Color",
        Default = CONFIG[key].Color,
        Transparency = 0,
        Callback = function(c)
            CONFIG[key].Color = c
        end,
    })
end

featureSection("Players", "Players")
featureSection("Enemies/NPCs", "Enemies")
featureSection("Items", "Items")
featureSection("Doors", "Doors")
featureSection("Objectives", "Objectives")
featureSection("Loot", "Loot")
featureSection("Interactables", "Interactables")
featureSection("Escape/Exit", "Exit")

local function scanPlayers()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            createEntry("Player_" .. plr.UserId, plr.Character, CONFIG.Players, plr.DisplayName)
        end
    end
end

local function scanEnemies()
    for _, inst in ipairs(Workspace:GetDescendants()) do
        if inst:IsA("Model") and inst.Name:match("^Corrupted") and inst.Name:find("%-") then
            createEntry("Enemy_" .. tostring(inst), inst, CONFIG.Enemies, getEnemyName(inst.Name))
        end
    end
end

local function scanObjectives()
    local terrain = Workspace:FindFirstChild("Terrain")
    if not terrain then
        return
    end

    local objectives = terrain:FindFirstChild("Objectives")
    if not objectives then
        return
    end

    local model = objectives:FindFirstChild("Model")
    if model and model:IsA("Model") then
        createEntry("Objective_Main", model, CONFIG.Objectives, "Objective")
    end
end

local function scanSpecificDoors()
    local terrain = Workspace:FindFirstChild("Terrain")
    if not terrain then
        return
    end

    local homeCave = terrain:FindFirstChild("HomeCave")
    if homeCave then
        local portal = homeCave:FindFirstChild("Portal")
        if portal then
            local door = portal:FindFirstChild("Door")
            if door then
                createEntry("Door_HomeCave", door, CONFIG.Doors, "Door")
            end
        end
    end

    local cherry = terrain:FindFirstChild("CherryBlossomMap")
    if cherry then
        local doors = cherry:FindFirstChild("Doors")
        if doors then
            local brokenDoor = doors:FindFirstChild("BrokenDoor")
            if brokenDoor then
                createEntry("Door_BrokenDoor", brokenDoor, CONFIG.Doors, "Broken Door")
            end
        end
    end
end

local function scanGeneric()
    for _, inst in ipairs(Workspace:GetDescendants()) do
        if inst:IsA("Model") or inst:IsA("BasePart") then
            local n = inst.Name

            if CONFIG.Items.Enabled and matchKeyword(n, {"item", "supply", "consumable", "bottle"}) then
                createEntry("Item_" .. tostring(inst), inst, CONFIG.Items, n)
            end

            if CONFIG.Loot.Enabled and matchKeyword(n, {"loot", "cash", "coin", "chest", "bag", "safe"}) then
                createEntry("Loot_" .. tostring(inst), inst, CONFIG.Loot, n)
            end

            if CONFIG.Exit.Enabled and matchKeyword(n, {"exit", "escape"}) then
                createEntry("Exit_" .. tostring(inst), inst, CONFIG.Exit, n)
            end

            if CONFIG.Interactables.Enabled then
                if inst:FindFirstChildOfClass("ProximityPrompt") or inst:FindFirstChildOfClass("ClickDetector") then
                    createEntry("Interact_" .. tostring(inst), inst, CONFIG.Interactables, n)
                end
            end

            if CONFIG.Doors.Enabled and matchKeyword(n, {"door"}) then
                createEntry("Door_" .. tostring(inst), inst, CONFIG.Doors, n)
            end
        end
    end
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function(char)
        task.wait(1)
        if CONFIG.Players.Enabled then
            createEntry("Player_" .. plr.UserId, char, CONFIG.Players, plr.DisplayName)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(plr)
    cleanupEntry("Player_" .. plr.UserId)
end)

Workspace.DescendantAdded:Connect(function(inst)
    task.wait(0.2)

    if CONFIG.Enemies.Enabled and inst:IsA("Model") and inst.Name:match("^Corrupted") and inst.Name:find("%-") then
        createEntry("Enemy_" .. tostring(inst), inst, CONFIG.Enemies, getEnemyName(inst.Name))
    end

    if CONFIG.Objectives.Enabled then
        local terrain = Workspace:FindFirstChild("Terrain")
        local objectives = terrain and terrain:FindFirstChild("Objectives")
        if objectives and inst == objectives:FindFirstChild("Model") then
            createEntry("Objective_Main", inst, CONFIG.Objectives, "Objective")
        end
    end

    if CONFIG.Doors.Enabled then
        local terrain = Workspace:FindFirstChild("Terrain")
        if terrain then
            local homeCave = terrain:FindFirstChild("HomeCave")
            if homeCave then
                local portal = homeCave:FindFirstChild("Portal")
                local door = portal and portal:FindFirstChild("Door")
                if inst == door then
                    createEntry("Door_HomeCave", inst, CONFIG.Doors, "Door")
                end
            end

            local cherry = terrain:FindFirstChild("CherryBlossomMap")
            if cherry then
                local doors = cherry:FindFirstChild("Doors")
                local brokenDoor = doors and doors:FindFirstChild("BrokenDoor")
                if inst == brokenDoor then
                    createEntry("Door_BrokenDoor", inst, CONFIG.Doors, "Broken Door")
                end
            end
        end
    end

    if CONFIG.Items.Enabled and (inst:IsA("Model") or inst:IsA("BasePart")) and matchKeyword(inst.Name, {"item", "supply", "consumable", "bottle"}) then
        createEntry("Item_" .. tostring(inst), inst, CONFIG.Items, inst.Name)
    end

    if CONFIG.Loot.Enabled and (inst:IsA("Model") or inst:IsA("BasePart")) and matchKeyword(inst.Name, {"loot", "cash", "coin", "chest", "bag", "safe"}) then
        createEntry("Loot_" .. tostring(inst), inst, CONFIG.Loot, inst.Name)
    end

    if CONFIG.Exit.Enabled and (inst:IsA("Model") or inst:IsA("BasePart")) and matchKeyword(inst.Name, {"exit", "escape"}) then
        createEntry("Exit_" .. tostring(inst), inst, CONFIG.Exit, inst.Name)
    end

    if CONFIG.Interactables.Enabled and (inst:IsA("Model") or inst:IsA("BasePart")) then
        if inst:FindFirstChildOfClass("ProximityPrompt") or inst:FindFirstChildOfClass("ClickDetector") then
            createEntry("Interact_" .. tostring(inst), inst, CONFIG.Interactables, inst.Name)
        end
    end
end)

RunService.RenderStepped:Connect(function()
    Camera = Workspace.CurrentCamera or Camera

    if CONFIG.Players.Enabled then
        scanPlayers()
    end

    if CONFIG.Enemies.Enabled then
        scanEnemies()
    end

    if CONFIG.Objectives.Enabled then
        scanObjectives()
    end

    if CONFIG.Doors.Enabled then
        scanSpecificDoors()
    end

    if CONFIG.Items.Enabled or CONFIG.Loot.Enabled or CONFIG.Interactables.Enabled or CONFIG.Exit.Enabled then
        scanGeneric()
    end

    for key in pairs(tracked) do
        updateEntry(key)
    end
end)
