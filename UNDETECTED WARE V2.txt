local repo = 'https://raw.githubusercontent.com/mstudio45/LinoriaLib/main/'

local Library = loadstring(game:HttpGet(repo .. 'Library.lua'))()
local ThemeManager = loadstring(game:HttpGet(repo .. 'addons/ThemeManager.lua'))()
local SaveManager = loadstring(game:HttpGet(repo .. 'addons/SaveManager.lua'))()

local Options = Library.Options
local Toggles = Library.Toggles

Library.ShowToggleFrameInKeybinds = true
Library.ShowCustomCursor = true
Library.NotifySide = "Left"

local Window = Library:CreateWindow({
    Title = 'UNDETECTEDWARE',
    Center = true,
    AutoShow = true,
    Resizable = true,
    TabPadding = 8,
    MenuFadeTime = 0.2
})

local Tabs = {
    Main = Window:AddTab('Main'),
    Combat = Window:AddTab('Combat'),
    Misc = Window:AddTab('Misc'),
    Esp = Window:AddTab('Esp'),
    ['UI Settings'] = Window:AddTab('UI Settings'),
}


-- Create Left and Right groupboxes in the Main tab
local ClientBox = Tabs.Main:AddLeftGroupbox('Player Features')
local NewRightBox = Tabs.Main:AddRightGroupbox('Alt Features')

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Infinite Stamina Toggle Logic
local staminaLoop
local function SetupInfiniteStamina()
    local StaminaTbl = {}
    for _, v in pairs(getgc(true)) do
        if type(v) == "table" and rawget(v, "S") then
            table.insert(StaminaTbl, v)
        end
    end

    if staminaLoop then
        staminaLoop:Disconnect()
    end

    staminaLoop = RunService.RenderStepped:Connect(function()
        if Toggles.InfiniteStaminaToggle.Value then
            for _, tbl in pairs(StaminaTbl) do
                tbl.S = 100
            end
        end
    end)
end

ClientBox:AddToggle('InfiniteStaminaToggle', {
    Text = 'Infinite Stamina',
    Default = false,
    Tooltip = 'Prevents stamina from decreasing.',

    Callback = function(Value)
        if Value then
            SetupInfiniteStamina()
        elseif staminaLoop then
            staminaLoop:Disconnect()
            staminaLoop = nil
        end
    end
})

-- WalkSpeed Bypass Setup
local DesiredSpeed = 35
local OriginalWalkspeed = 16
local _Humanoid

local OldNewIndex
OldNewIndex = hookmetamethod(game, "__newindex", function(self, key, value)
    if not checkcaller() and key == "WalkSpeed" and typeof(self) == "Instance" and self:IsA("Humanoid") then
        OriginalWalkspeed = value
        if value ~= DesiredSpeed then
            return
        end
    end
    return OldNewIndex(self, key, value)
end)

local OldIndex
OldIndex = hookmetamethod(game, "__index", function(self, key)
    if not checkcaller() and self == _Humanoid and key == "WalkSpeed" then
        return OriginalWalkspeed
    end
    return OldIndex(self, key)
end)

local OldNamecall
OldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
    local method = getnamecallmethod()
    if not checkcaller() then
        if method == "FireServer" and (self.Name == "__DFfDD" or self.Name == "0924023902330") then
            return wait(9000000000)
        elseif method == "Kick" then
            return wait(9000000000)
        end
    end
    return OldNamecall(self, ...)
end)

local function applyWalkSpeedBypass()
    RunService:BindToRenderStep("WalkSpeedBypass", Enum.RenderPriority.Character.Value + 1, function()
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                _Humanoid = humanoid
                if humanoid.WalkSpeed ~= DesiredSpeed then
                    humanoid.WalkSpeed = DesiredSpeed
                end
            end
        end
    end)
end

local function removeWalkSpeedBypass()
    RunService:UnbindFromRenderStep("WalkSpeedBypass")
    if _Humanoid then
        _Humanoid.WalkSpeed = OriginalWalkspeed
    end
end

LocalPlayer.CharacterAdded:Connect(function(char)
    repeat task.wait() until char:FindFirstChildOfClass("Humanoid")
    if getgenv().WalkSpeedToggle then
        applyWalkSpeedBypass()
    end
end)

getgenv().WalkSpeedToggle = false

ClientBox:AddToggle('WalkSpeedToggle', {
    Text = 'Walk Speed',
    Default = false,
    Callback = function(enabled)
        getgenv().WalkSpeedToggle = enabled
        if enabled then
            applyWalkSpeedBypass()
            
        else
            removeWalkSpeedBypass()
            
        end
    end,
    Tooltip = 'Toggle Walk Speed Bypass',
})

ClientBox:AddSlider('WalkSpeedSlider', {
    Text = 'Walk Speed Value',
    Default = 35,
    Min = 0,
    Max = 100,
    Rounding = 1,
    Callback = function(val)
        DesiredSpeed = val
       
    end,
    Tooltip = 'Adjust your walk speed',
})



-- Spinbot Logic
local spinEnabled = false
local spinSpeed = 100 -- default speed
local spinLoop

local function spinCharacter()
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local root = character.HumanoidRootPart
        root.CFrame = root.CFrame * CFrame.Angles(0, math.rad(spinSpeed), 0)
    end
end

local function startSpin()
    if spinLoop then spinLoop:Disconnect() end
    spinLoop = RunService.RenderStepped:Connect(function()
        if spinEnabled then
            spinCharacter()
        end
    end)
end

local function stopSpin()
    if spinLoop then
        spinLoop:Disconnect()
        spinLoop = nil
    end
end

LocalPlayer.CharacterAdded:Connect(function(character)
    character:WaitForChild("HumanoidRootPart")
    if spinEnabled then
        startSpin()
        
    end
end)

ClientBox:AddToggle('SpinbotToggle', {
    Text = 'Enable Spinbot',
    Default = false,
    Tooltip = 'Toggle spinbot on/off',
    Callback = function(value)
        spinEnabled = value
        if value then
            startSpin()
            
        else
            stopSpin()
            
        end
    end
})

ClientBox:AddSlider('SpinbotSpeed', {
    Text = 'Spinbot Speed',
    Default = spinSpeed,
    Min = 0,
    Max = 500,
    Rounding = 1,
    Tooltip = 'Adjust the spin speed',
    Callback = function(value)
        spinSpeed = math.max(value, 0)
        
    end
})


-- Global variable to store color
getgenv().forceFieldColor = Color3.fromRGB(255, 255, 255)
getgenv().forceFieldToggle = false

-- ForceField Toggle
NewRightBox:AddToggle('ForceFieldToggle', {
    Text = 'ForceField ViewModel',
    Default = false,
    Tooltip = 'Apply ForceField material to ViewModel arms',
    Callback = function(value)
        getgenv().forceFieldToggle = value
        if value then
            task.spawn(function()
                local camera = workspace.CurrentCamera
                while getgenv().forceFieldToggle do
                    local viewModel = camera:FindFirstChild("ViewModel")
                    if viewModel then
                        local leftArm = viewModel:FindFirstChild("Left Arm") or viewModel:FindFirstChild("LeftArm")
                        local rightArm = viewModel:FindFirstChild("Right Arm") or viewModel:FindFirstChild("RightArm")
                        for _, arm in pairs({leftArm, rightArm}) do
                            if arm and arm:IsA("BasePart") then
                                arm.Material = Enum.Material.ForceField
                                arm.Color = getgenv().forceFieldColor
                            end
                        end
                    end
                    task.wait(0.5)
                end

                -- Optional: Reset arms when toggle is turned off
                local camera = workspace.CurrentCamera
                local viewModel = camera:FindFirstChild("ViewModel")
                if viewModel then
                    for _, name in ipairs({"Left Arm", "LeftArm", "Right Arm", "RightArm"}) do
                        local arm = viewModel:FindFirstChild(name)
                        if arm and arm:IsA("BasePart") then
                            arm.Material = Enum.Material.Plastic
                        end
                    end
                end
            end)
        end
    end
})

-- Color Picker for ForceField and FOV
NewRightBox:AddLabel('ForceField Color'):AddColorPicker('ForceFieldColor', {
    Default = Color3.new(1, 1, 1),
    Title = 'ForceField Color',
    Transparency = nil,

    Callback = function(Value)
      
        getgenv().forceFieldColor = Value
    end
})



-- TPkill Toggle
NewRightBox:AddToggle('TPkillToggle', {
    Text = 'TPkill',
    Default = false,
    Tooltip = 'Teleport to nearby players within 23 studs',
    Callback = function(value)
        if getgenv().tpKillLoop then
            getgenv().tpKillLoop:Disconnect()
            getgenv().tpKillLoop = nil
        end
        if value then
            getgenv().tpKillLoop = game:GetService("RunService").RenderStepped:Connect(function()
                local char = game.Players.LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                for _, p in pairs(game.Players:GetPlayers()) do
                    if p ~= game.Players.LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                        local target = p.Character.HumanoidRootPart
                        if (hrp.Position - target.Position).Magnitude <= 23 then
                            hrp.CFrame = target.CFrame + Vector3.new(0, 2, 0)
                            break
                        end
                    end
                end
            end)
        end
    end
})
--// Variables
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local originalFOV = Camera.FieldOfView
getgenv().FOVChangerEnabled = false
getgenv().DesiredFOV = originalFOV

--// FOV Logic
RunService.RenderStepped:Connect(function()
    if getgenv().FOVChangerEnabled and Camera then
        Camera.FieldOfView = getgenv().DesiredFOV
    else
        Camera.FieldOfView = originalFOV
    end
end)

--// UI Elements
NewRightBox:AddToggle('FOVChangerToggle', {
    Text = 'FOV Changer',
    Default = false,
    Tooltip = 'Toggle custom Field of View',
    Callback = function(enabled)
        getgenv().FOVChangerEnabled = enabled
    end
})

NewRightBox:AddSlider('FOVSlider', {
    Text = 'FOV Value',
    Default = 90,
    Min = 30,
    Max = 120,
    Rounding = 1,
    Tooltip = 'Adjust the camera FOV',
    Callback = function(val)
        getgenv().DesiredFOV = val
        end
        
})

-- Remove Camshake Button (updated)
NewRightBox:AddButton({
    Text = 'Remove Camshake',
    Func = function()
        local char = game.Players.LocalPlayer.Character
        if char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local part = hrp:FindFirstChild("CamShakePart")
                if part then
                    part:Destroy()
                    Library:Notify("CamShakePart removed")
                end
            end
        end
    end,
    Tooltip = 'Remove camera shake part from character',
    DisabledTooltip = 'You can\'t use this right now!',
    Disabled = false,
    Visible = true
})



-- Hide Head Button (updated)
NewRightBox:AddButton({
    Text = 'Hide Head',
    Func = function()
        local char = game.Players.LocalPlayer.Character
        if char then
            local torso = char:FindFirstChild("Torso")
            if torso then
                local neck = torso:FindFirstChild("Neck")
                if neck then
                    neck:Destroy()
                end
            end

            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local CTs = hrp:FindFirstChild("CTs")
                if CTs then
                    local RGCT_Neck = CTs:FindFirstChild("RGCT_Neck")
                    if RGCT_Neck then
                        RGCT_Neck.TwistLowerAngle = 170
                        RGCT_Neck.TwistUpperAngle = 155
                    end
                end
            end

            local head = char:FindFirstChild("Head")
            if head then
                head.CollisionGroup = nil
                local collider = head:FindFirstChild("HeadCollider")
                if collider then
                    collider:Destroy()
                end
            end

            Library:Notify("Head hidden")
        end
    end,
    Tooltip = 'Hides your character\'s head and disables neck twist limits',
    DisabledTooltip = 'Unavailable now',
    Disabled = false,
    Visible = true
})
local CombatLeft = Tabs.Combat:AddLeftGroupbox('Combat Features')
local CombatRight = Tabs.Combat:AddRightGroupbox('Silent Aim Features')

-- SERVICES
local plrs = game:GetService("Players")
local run = game:GetService("RunService")
local rs = game:GetService("ReplicatedStorage")
local me = plrs.LocalPlayer

-- REMOTES
local remote1 = rs.Events["XMHH.2"]
local remote2 = rs.Events["XMHH2.2"]
local remotes = {}

-- SETTINGS
local GlobalWhiteList = {}
local SectionSettings = {
    MeleeAura = {
        Distance = 20,
        CheckWhitelist = false,
        CheckTeam = false,
        TargetPart = "Random",
        ShowAnim = true
    }
}

local ValidMeleeTargetParts = {"Head", "Torso", "Right Arm", "Left Arm", "Right Leg", "Left Leg"}
local meleeAuraEnabled = false

-- MAIN TOGGLE
CombatLeft:AddToggle('MeleeAuraToggle', {
    Text = 'Melee Aura',
    Default = false,
    Tooltip = 'Automatically attack nearby players with melee',
    Callback = function(enabled)
        meleeAuraEnabled = enabled

        if enabled then
            if remotes.MeleeAuraTask then return end -- already running

            remotes.MeleeAuraTask = task.spawn(function()
                local AttachTick = tick()
                local AttachCD = {
                    ["Fists"] = .05, ["Knuckledusters"] = .05, ["Nunchucks"] = 0.05, ["Shiv"] = .05,
                    ["Bat"] = 1, ["Metal-Bat"] = 1, ["Chainsaw"] = 2.5, ["Balisong"] = .05, ["Rambo"] = .3,
                    ["Shovel"] = 3, ["Sledgehammer"] = 2, ["Katana"] = .1, ["Wrench"] = .1
                }

                local function Attack(target)
                    if not (target and target:FindFirstChild("Head")) then return end
                    if not me.Character then return end
                    local TOOL = me.Character:FindFirstChildOfClass("Tool")
                    if not TOOL then return end

                    local attachcd = AttachCD[TOOL.Name] or 0.5
                    if tick() - AttachTick < attachcd then return end

                    local result = remote1:InvokeServer("🍞", tick(), TOOL, "43TRFWX", "Normal", tick(), true)

                    if SectionSettings.MeleeAura.ShowAnim then
                        local anim = TOOL:FindFirstChild("AnimsFolder") and TOOL.AnimsFolder:FindFirstChild("Slash1")
                        if anim then
                            local humanoid = me.Character:FindFirstChildOfClass("Humanoid")
                            local animator = humanoid and humanoid:FindFirstChild("Animator")
                            if animator then
                                animator:LoadAnimation(anim):Play(0.1, 1, 1.3)
                            end
                        end
                    end

                    task.wait(0.3 + math.random() * 0.2)

                    local Handle = TOOL:FindFirstChild("WeaponHandle") or TOOL:FindFirstChild("Handle") or me.Character:FindFirstChild("Left Arm")
                    if TOOL then
                        local targetPart
                        if SectionSettings.MeleeAura.TargetPart == "Random" then
                            targetPart = target:FindFirstChild(ValidMeleeTargetParts[math.random(1, #ValidMeleeTargetParts)])
                        else
                            targetPart = target:FindFirstChild(SectionSettings.MeleeAura.TargetPart) or target:FindFirstChild("Right Arm")
                        end
                        if not targetPart then return end

                        local args = {
                            "🍞",
                            tick(),
                            TOOL,
                            "2389ZFX34",
                            result,
                            true,
                            Handle,
                            targetPart,
                            target,
                            me.Character.HumanoidRootPart.Position,
                            targetPart.Position
                        }

                        if TOOL.Name == "Chainsaw" then
                            for _ = 1, 15 do
                                remote2:FireServer(unpack(args))
                            end
                        else
                            remote2:FireServer(unpack(args))
                        end

                        AttachTick = tick()
                    end
                end

                while meleeAuraEnabled do
                    local mychar = me.Character or me.CharacterAdded:Wait()
                    if mychar and mychar:FindFirstChild("HumanoidRootPart") then
                        local myhrp = mychar.HumanoidRootPart
                        for _, player in ipairs(plrs:GetPlayers()) do
                            if player ~= me and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                                local hrp = player.Character.HumanoidRootPart
                                local distance = (myhrp.Position - hrp.Position).Magnitude
                                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                                local forceField = player.Character:FindFirstChildOfClass("ForceField")

                                if distance < (SectionSettings.MeleeAura.Distance or 20)
                                    and humanoid and humanoid.Health > 15
                                    and not forceField
                                    and (not SectionSettings.MeleeAura.CheckWhitelist or not GlobalWhiteList[player.Name])
                                    and (not SectionSettings.MeleeAura.CheckTeam or player.Team ~= me.Team) then
                                    Attack(player.Character)
                                end
                            end
                        end
                    end
                    run.Heartbeat:Wait()
                end
            end)
        else
            if remotes.MeleeAuraTask then
                task.cancel(remotes.MeleeAuraTask)
                remotes.MeleeAuraTask = nil
            end
        end
    end
})

-- 🧠 HIT PART DROPDOWN
CombatLeft:AddDropdown('MeleeAuraTargetPartDropdown', {
    Values = { "Random", unpack(ValidMeleeTargetParts) },
    Default = 1,
    Multi = false,
    Text = 'Target Part',
    Tooltip = 'Choose the target body part for melee aura',
    Callback = function(value)
        SectionSettings.MeleeAura.TargetPart = value
        
    end
})

-- 🎞️ SHOW ANIMATION TOGGLE
CombatLeft:AddToggle('MeleeAuraShowAnimToggle', {
    Text = 'Show Animations',
    Default = SectionSettings.MeleeAura.ShowAnim,
    Tooltip = 'Enable or disable melee attack animations',
    Callback = function(enabled)
        SectionSettings.MeleeAura.ShowAnim = enabled
        
    end
})
CombatLeft:AddSlider('MeleeAuraDistanceSlider', {
    Text = 'Melee Aura Distance',
    Default = SectionSettings.MeleeAura.Distance,
    Min = 1,
    Max = 20,
    Rounding = 1,
    Tooltip = 'Distance For Melee Aura',
    Callback = function(value)
        SectionSettings.MeleeAura.Distance = value
        
    end
})


local weaponTables = {}
local originalValues = {}
local noRecoilRunning = false
local noRecoilThread

local function FindWeapons()
    weaponTables = {}
    for _, v in pairs(getgc(true)) do
        if type(v) == "table" and rawget(v, "EquipTime") and rawget(v, "Recoil") then
            table.insert(weaponTables, v)
            
        end
    end
    
end

local function PatchWeapons()
    for _, v in pairs(weaponTables) do
        if type(v) == "table" then
            if not originalValues[v] then
                originalValues[v] = {
                    Recoil = v.Recoil or 0,
                    AngleX_Min = v.AngleX_Min or 0,
                    AngleX_Max = v.AngleX_Max or 0,
                    AngleY_Min = v.AngleY_Min or 0,
                    AngleY_Max = v.AngleY_Max or 0,
                    AngleZ_Min = v.AngleZ_Min or 0,
                    AngleZ_Max = v.AngleZ_Max or 0,
                    RecoilSpeed = v.RecoilSpeed or 0,
                    RecoilDamper = v.RecoilDamper or 1,
                    Accuracy = v.Accuracy or 1,
                    RecoilReduction = v.RecoilReduction or 1,
                    CameraRecoilingEnabled = v.CameraRecoilingEnabled or true
                }
            end

            v.CameraRecoilingEnabled = false
            v.Recoil = 0
            v.AngleX_Min = 0
            v.AngleX_Max = 0
            v.AngleY_Min = 0
            v.AngleY_Max = 0
            v.AngleZ_Min = 0
            v.AngleZ_Max = 0
            v.RecoilSpeed = 0
            v.RecoilDamper = 1
            v.Accuracy = 1
            v.RecoilReduction = 1

            if v.SprayLerp then
                v.SprayLerp.Enabled = false
            end
        end
    end
end

local function RestoreWeapons()
    for v, vals in pairs(originalValues) do
        if type(v) == "table" then
            v.CameraRecoilingEnabled = vals.CameraRecoilingEnabled
            v.Recoil = vals.Recoil
            v.AngleX_Min = vals.AngleX_Min
            v.AngleX_Max = vals.AngleX_Max
            v.AngleY_Min = vals.AngleY_Min
            v.AngleY_Max = vals.AngleY_Max
            v.AngleZ_Min = vals.AngleZ_Min
            v.AngleZ_Max = vals.AngleZ_Max
            v.RecoilSpeed = vals.RecoilSpeed
            v.RecoilDamper = vals.RecoilDamper
            v.Accuracy = vals.Accuracy
            v.RecoilReduction = vals.RecoilReduction

            if v.SprayLerp then
                v.SprayLerp.Enabled = true
            end
        end
    end
end

local function StartNoRecoil()
    if noRecoilRunning then return end
    noRecoilRunning = true
    FindWeapons()

    noRecoilThread = task.spawn(function()
        while noRecoilRunning do
            PatchWeapons()
            task.wait(0.5)
        end
        RestoreWeapons()
    end)
end

local function StopNoRecoil()
    noRecoilRunning = false
end

CombatLeft:AddToggle('NoRecoilToggle', {
    Text = 'No Recoil',
    Default = false,
    Tooltip = 'Removes recoil from all weapons',
    Callback = function(enabled)
        if enabled then
            StartNoRecoil()
        else
            StopNoRecoil()
        end
    end
})
local plrs = game:GetService("Players")
local rs = game:GetService("ReplicatedStorage")
local me = plrs.LocalPlayer

-- Instant Reload Variables
local instant_reloadF = false
local reloadConnections = {}

local gunR_remote = rs.Events["GNX_R"]

local function clearReloadConnections()
    for _, conn in pairs(reloadConnections) do
        conn:Disconnect()
    end
    reloadConnections = {}
end

local function setupTool(tool)
    if tool and tool:FindFirstChild("IsGun") and instant_reloadF then
        local values = tool:FindFirstChild("Values")
        if not values then return end

        local ammoVal = values:FindFirstChild("SERVER_Ammo")
        local storedAmmoVal = values:FindFirstChild("SERVER_StoredAmmo")

        if storedAmmoVal then
            reloadConnections[#reloadConnections + 1] = storedAmmoVal:GetPropertyChangedSignal("Value"):Connect(function()
                if instant_reloadF and storedAmmoVal.Value ~= 0 then
                    gunR_remote:FireServer(tick(), "KLWE89U0", tool)
                end
            end)
        end

        if ammoVal then
            reloadConnections[#reloadConnections + 1] = ammoVal:GetPropertyChangedSignal("Value"):Connect(function()
                if instant_reloadF and storedAmmoVal and storedAmmoVal.Value ~= 0 then
                    gunR_remote:FireServer(tick(), "KLWE89U0", tool)
                end
            end)
        end
    end
end

local function InstantReloadSetup()
    if me.Character then
        local charme = me.Character
        local tool = charme:FindFirstChildOfClass("Tool")
        setupTool(tool)

        reloadConnections[#reloadConnections + 1] = charme.ChildAdded:Connect(function(obj)
            if obj:IsA("Tool") then
                setupTool(obj)
            end
        end)
    end

    reloadConnections[#reloadConnections + 1] = me.CharacterAdded:Connect(function(charr)
        repeat task.wait() until charr and charr.Parent
        clearReloadConnections()
        local tool = charr:FindFirstChildOfClass("Tool")
        setupTool(tool)

        reloadConnections[#reloadConnections + 1] = charr.ChildAdded:Connect(function(obj)
            if obj:IsA("Tool") then
                setupTool(obj)
            end
        end)
    end)
end

CombatLeft:AddToggle('InstantReloadToggle', {
    Text = 'Instant Reload',
    Default = false,
    Tooltip = 'Automatically reload guns instantly',
    Callback = function(enabled)
        instant_reloadF = enabled
        clearReloadConnections()
        if enabled then
            InstantReloadSetup()
        end
    end
})

-- Services
local plrs = game:GetService("Players")
local rs = game:GetService("ReplicatedStorage")
local me = plrs.LocalPlayer
local camera = workspace.CurrentCamera

-- Settings (with toggle variables)
local SectionSettings = {
    Ragebot = {
        DownedCheck = false,
        CheckWhitelist = false,
        CheckTarget = false,
        CheckTeam = false
    }
}

local GlobalWhiteList = {}
local GlobalTarget = {}

-- Ragebot enabled toggle variable
local RagebotEnabled = false

-- Toggles variables
local DownedCheckEnabled = SectionSettings.Ragebot.DownedCheck
local CheckWhitelistEnabled = SectionSettings.Ragebot.CheckWhitelist
local CheckTargetEnabled = SectionSettings.Ragebot.CheckTarget
local CheckTeamEnabled = SectionSettings.Ragebot.CheckTeam

-- Helper: Random string generator
local function RandomString(length)
    local chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"
    local str = ""
    for i = 1, length do
        local rand = math.random(1, #chars)
        str = str .. chars:sub(rand, rand)
    end
    return str
end

-- Optional: Headshot sound
local function PlayHeadshotSound()
    local sound = Instance.new("Sound", me:WaitForChild("PlayerGui"))
    sound.SoundId = "rbxassetid://5228367231" -- Headshot SFX
    sound.Volume = 1
    sound:Play()
    game.Debris:AddItem(sound, 2)
end

-- Get Closest Valid Enemy (checks respect toggles)
local function GetClosestEnemy()
    if not me.Character or not me.Character:FindFirstChild("HumanoidRootPart") then return nil end
    local closestEnemy = nil
    local shortestDistance = 100

    for _, player in pairs(plrs:GetPlayers()) do
        if player == me then continue end

        local character = player.Character
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
        local rootPart = character and character:FindFirstChild("HumanoidRootPart")
        local forceField = character and character:FindFirstChildOfClass("ForceField")

        if character and rootPart and humanoid and not forceField then
            if (not DownedCheckEnabled or humanoid.Health > 15) then
                local distance = (rootPart.Position - me.Character.HumanoidRootPart.Position).Magnitude
                if distance > 100 then continue end
                if CheckWhitelistEnabled and GlobalWhiteList[player.Name] then continue end
                if CheckTargetEnabled and not GlobalTarget[player.Name] then continue end
                if CheckTeamEnabled and player.Team == me.Team then continue end
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestEnemy = player
                end
            end
        end
    end

    return closestEnemy
end

-- Shoot Logic
local function Shoot(target)
    if not target or not target.Character then return end

    local head = target.Character:FindFirstChild("Head")
    if not head then return end

    local tool = me.Character and me.Character:FindFirstChildOfClass("Tool")
    if not tool then return end

    local values = tool:FindFirstChild("Values")
    local hitMarker = tool:FindFirstChild("Hitmarker")
    if not values or not hitMarker then return end

    local ammo = values:FindFirstChild("SERVER_Ammo")
    local storedAmmo = values:FindFirstChild("SERVER_StoredAmmo")
    if not ammo or not storedAmmo or ammo.Value <= 0 then return end

    local hitPosition = head.Position
    local hitDirection = (hitPosition - camera.CFrame.Position).Unit
    local randomKey = RandomString(30) .. "0"

    -- Shoot
    rs.Events.GNX_S:FireServer(
        tick(),
        randomKey,
        tool,
        "FDS9I83",
        camera.CFrame.Position,
        { hitDirection },
        false
    )

    -- Hit
    rs.Events["ZFKLF__H"]:FireServer(
        "🧈",
        tool,
        randomKey,
        1,
        head,
        hitPosition,
        hitDirection
    )

    -- Simulate shot and SFX
    ammo.Value = math.max(ammo.Value - 1, 0)
    hitMarker:Fire(head)
    PlayHeadshotSound()
end

-- Ragebot Loop (runs always, shoots only if enabled)
task.spawn(function()
    while true do
        if RagebotEnabled then
            if me.Character and me.Character:FindFirstChild("HumanoidRootPart") then
                if me.Character:FindFirstChildOfClass("Tool") then
                    local target = GetClosestEnemy()
                    if target then
                        Shoot(target)
                    end
                end
            end
        end
        task.wait(0.25)
    end
end)

-- UI Toggles using CombatLeft:AddToggle

CombatLeft:AddToggle('RagebotToggle', {
    Text = 'Ragebot',
    Default = false,
    Tooltip = 'Automatically shoot nearest enemies',
    Callback = function(enabled)
        RagebotEnabled = enabled
    end
})

CombatLeft:AddToggle('DownedCheckToggle', {
    Text = 'Downed Check',
    Default = false,
    Tooltip = 'Ignore downed enemies',
    Callback = function(enabled)
        DownedCheckEnabled = enabled
        SectionSettings.Ragebot.DownedCheck = enabled
    end
})

CombatLeft:AddToggle('WhitelistCheckToggle', {
    Text = 'Whitelist Check',
    Default = false,
    Tooltip = 'Ignore whitelisted players',
    Callback = function(enabled)
        CheckWhitelistEnabled = enabled
        SectionSettings.Ragebot.CheckWhitelist = enabled
    end
})

CombatLeft:AddToggle('TargetCheckToggle', {
    Text = 'Target Check',
    Default = false,
    Tooltip = 'Only target specific players',
    Callback = function(enabled)
        CheckTargetEnabled = enabled
        SectionSettings.Ragebot.CheckTarget = enabled
    end
})

CombatLeft:AddToggle('TeamCheckToggle', {
    Text = 'Team Check',
    Default = false,
    Tooltip = 'Ignore teammates',
    Callback = function(enabled)
        CheckTeamEnabled = enabled
        SectionSettings.Ragebot.CheckTeam = enabled
    end
})


-- Example button: Hitbox Expander (like your example)
CombatLeft:AddButton({
    Text = 'Hitbox Expander',
    Func = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Vuubvyc/UNDETECTEDWAREEE/refs/heads/main/Hitbox"))()
    end,
    Tooltip = 'Expand hitboxes for easier targeting',
    Disabled = false,
    Visible = true
})

-- SETTINGS
local SilentAimSettings = {
    Enabled = false,          -- Start disabled
    DrawSize = 150,
    TargetPart = "Head",
    CheckWhitelist = false,
    CheckWall = false,
    UseHitChance = false,
    HitChance = 100,
    CheckTeam = false,
    DrawCircle = true,
    DrawColor = Color3.fromRGB(255, 255, 255)
}

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local currentTarget = nil

-- CENTER OF SCREEN FUNCTION
local function getScreenCenter()
    local viewportSize = Camera.ViewportSize
    return Vector2.new(viewportSize.X / 2, viewportSize.Y / 2)
end

-- CREATE FOV CIRCLE
local FOVCircle = Drawing.new("Circle")
FOVCircle.Radius = SilentAimSettings.DrawSize
FOVCircle.Color = SilentAimSettings.DrawColor
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.Visible = SilentAimSettings.Enabled and SilentAimSettings.DrawCircle

RunService.RenderStepped:Connect(function()
    FOVCircle.Position = getScreenCenter()
    FOVCircle.Radius = SilentAimSettings.DrawSize
    FOVCircle.Color = SilentAimSettings.DrawColor
    FOVCircle.Visible = SilentAimSettings.Enabled and SilentAimSettings.DrawCircle
end)

-- GET CLOSEST TARGET FUNCTION
local function GetClosestTarget()
    if not SilentAimSettings.Enabled then return nil end

    local closest, minDistance = nil, SilentAimSettings.DrawSize
    local screenCenter = getScreenCenter()

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if SilentAimSettings.CheckTeam and player.Team == LocalPlayer.Team then continue end
            if SilentAimSettings.CheckWhitelist then continue end

            local part = player.Character:FindFirstChild(SilentAimSettings.TargetPart) or player.Character:FindFirstChild("Head")
            if not part then continue end

            local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
            if not onScreen then continue end

            if SilentAimSettings.CheckWall then
                local ignore = {Camera, LocalPlayer.Character, player.Character}
                if #Camera:GetPartsObscuringTarget({part.Position}, ignore) > 0 then continue end
            end

            local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
            if dist < minDistance then
                minDistance = dist
                closest = player
            end
        end
    end

    if closest and SilentAimSettings.UseHitChance and math.random(1, 100) > SilentAimSettings.HitChance then
        return nil
    end

    return closest
end

-- SILENT AIM EXECUTION
local VisualizeEvent = ReplicatedStorage:WaitForChild("Events2"):WaitForChild("Visualize")
local DamageEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("ZFKLF__H")

task.spawn(function()
    while true do
        currentTarget = SilentAimSettings.Enabled and GetClosestTarget() or nil
        task.wait()
    end
end)

VisualizeEvent.Event:Connect(function(_, ShotCode, _, Gun, _, StartPos, BulletsPerShot)
    if not SilentAimSettings.Enabled then return end
    if not currentTarget or not currentTarget.Character or currentTarget.Character:FindFirstChildOfClass("ForceField") then return end

    local tool = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool")
    if not tool or Gun ~= tool then return end

    local part = currentTarget.Character:FindFirstChild(SilentAimSettings.TargetPart)
    if not part then return end

    local hitPos = part.Position
    local bullets = {}

    for i = 1, math.clamp(#BulletsPerShot, 1, 100) do
        bullets[i] = CFrame.new(StartPos, hitPos).LookVector
    end

    task.wait(0.005)

    for i, dir in ipairs(bullets) do
        DamageEvent:FireServer("🧈", Gun, ShotCode, i, part, hitPos, dir)
    end

    if Gun:FindFirstChild("Hitmarker") then
        Gun.Hitmarker:Fire(part)
    end
end)

-- UI TOGGLES using CombatRight:AddToggle and AddSlider

CombatRight:AddToggle('SilentAimMasterToggle', {
    Text = 'Silent Aim',
    Default = SilentAimSettings.Enabled,
    Tooltip = 'Silent aim',
    Callback = function(enabled)
        SilentAimSettings.Enabled = enabled
        FOVCircle.Visible = enabled and SilentAimSettings.DrawCircle
        
    end
})

CombatRight:AddToggle('CheckWhitelistToggle', {
    Text = 'Check Whitelist',
    Default = SilentAimSettings.CheckWhitelist,
    Tooltip = 'Ignore whitelisted players',
    Callback = function(enabled)
        SilentAimSettings.CheckWhitelist = enabled
    end
})

CombatRight:AddToggle('CheckWallToggle', {
    Text = 'Wall Check',
    Default = SilentAimSettings.CheckWall,
    Tooltip = 'May Cause Lag',
    Callback = function(enabled)
        SilentAimSettings.CheckWall = enabled
    end
})

CombatRight:AddToggle('UseHitChanceToggle', {
    Text = 'Use Hit Chance',
    Default = SilentAimSettings.UseHitChance,
    Tooltip = 'Random chance to hit',
    Callback = function(enabled)
        SilentAimSettings.UseHitChance = enabled
    end
})

CombatRight:AddToggle('CheckTeamToggle', {
    Text = 'Check Team',
    Default = SilentAimSettings.CheckTeam,
    Tooltip = 'Ignore teammates',
    Callback = function(enabled)
        SilentAimSettings.CheckTeam = enabled
    end
})

CombatRight:AddToggle('DrawCircleToggle', {
    Text = 'Draw Circle',
    Default = SilentAimSettings.DrawCircle,
    Tooltip = 'Draw FOV circle',
    Callback = function(enabled)
        SilentAimSettings.DrawCircle = enabled
        FOVCircle.Visible = SilentAimSettings.Enabled and enabled
    end
})

CombatRight:AddSlider('SilentAimFOVSlider', {
    Text = 'Silent Aim FOV',
    Default = SilentAimSettings.DrawSize,
    Min = 20,
    Max = 500,
    Rounding = 0,
    Tooltip = 'Adjust silent aim FOV radius',
    Callback = function(value)
        SilentAimSettings.DrawSize = value
    end
})
CombatRight:AddDropdown('SilentAimParts', {
    Values = {"Head", "Torso", "Right Arm", "Left Arm", "Right Leg", "Left Leg"},
    Default = 1,
    Multi = false,
    Text = 'Target part',
    Tooltip = 'Select parts that the silent-aim will target',
    Callback = function(selected)
        SilentAimSettings.TargetPart = selected
        
    end
})




local MiscLeft = Tabs.Misc:AddLeftGroupbox('Misc Features')
local MiscRight = Tabs.Misc:AddRightGroupbox('Farm Features')

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

local PlayerTPSettings = {
    TargetPlayer = nil,
    LoopTeleport = false
}

-- Function to get player names
local function getPlayerNames()
    local names = {}
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            table.insert(names, player.Name)
        end
    end
    return names
end

-- Create dropdown
local playerDropdown = MiscLeft:AddDropdown('Players', {
    Values = getPlayerNames(),
    Default = 1,
    Multi = false,
    Text = 'Choose Player',
    Tooltip = 'Teleport to selected player',
    Callback = function(selected)
        PlayerTPSettings.TargetPlayer = selected
    end
})

-- Create loop toggle
MiscLeft:AddToggle('LoopPlayerTP', {
    Text = 'Teleport',
    Default = false,
    Tooltip = 'Teleport To Player',
    Callback = function(enabled)
        PlayerTPSettings.LoopTeleport = enabled

        if enabled then
            task.spawn(function()
                while PlayerTPSettings.LoopTeleport do
                    local targetPlayer = game.Players:FindFirstChild(PlayerTPSettings.TargetPlayer or "")
                    local localPlayer = game.Players.LocalPlayer

                    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                        local localChar = localPlayer.Character or localPlayer.CharacterAdded:Wait()
                        local localHRP = localChar:FindFirstChild("HumanoidRootPart")

                        if localHRP then
                            localHRP.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(0, 5, 0)
                        end
                    end

                    task.wait(1)
                end
            end)
        end
    end
})

-- Add Refresh button (CombatLeft style like your Hitbox Expander)
MiscLeft:AddButton({
    Text = 'Refresh Player List',
    Func = function()
        -- Update dropdown options
        playerDropdown:SetValues(getPlayerNames())
    end,
    Tooltip = 'Manually refresh player list',
    Disabled = false,
    Visible = true
})

-- Admin Check System
local AdminCheck_Enabled = false
local AdminCheck_Connection = nil
local AdminCheck_Coroutine = nil
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Admin list
local AdminList = {
    ["tabootvcat"] = true, ["Revenantic"] = true, ["Saabor"] = true, ["MoIitor"] = true, ["IAmUnderAMask"] = true,
    ["SheriffGorji"] = true, ["xXFireyScorpionXx"] = true, ["LoChips"] = true, ["DeliverCreations"] = true,
    ["TDXiswinning"] = true, ["TZZV"] = true, ["FelixVenue"] = true, ["SIEGFRlED"] = true, ["ARRYvvv"] = true,
    ["z_papermoon"] = true, ["Malpheasance"] = true, ["ModHandIer"] = true, ["valphex"] = true, ["J_anday"] = true,
    ["tvdisko"] = true, ["yIlehs"] = true, ["COLOSSUSBUILTOFSTEEL"] = true, ["SeizedHolder"] = true, ["r3shape"] = true,
    ["RVVZ"] = true, ["adurize"] = true, ["codedcosmetics"] = true, ["QuantumCaterpillar"] = true,
    ["FractalHarmonics"] = true, ["GalacticSculptor"] = true, ["oTheSilver"] = true, ["Kretacaous"] = true,
    ["icarus_xs1goliath"] = true, ["GlamorousDradon"] = true, ["rainjeremy"] = true, ["parachuter2000"] = true,
    ["faintermercury"] = true, ["harht"] = true, ["Sansek1252"] = true, ["Snorpuwu"] = true, ["BenAzoten"] = true,
    ["Cand1ebox"] = true, ["KeenlyAware"] = true, ["mrzued"] = true, ["BruhmanVIII"] = true, ["Nystesia"] = true,
    ["fausties"] = true, ["zateopp"] = true, ["Iordnabi"] = true, ["ReviveTheDevil"] = true, ["jake_jpeg"] = true,
    ["UncrossedMeat3888"] = true, ["realpenyy"] = true, ["karateeeh"] = true, ["JayyMlg"] = true, ["Lo_Chips"] = true,
    ["Avelosky"] = true, ["king_ab09"] = true, ["TigerLe123"] = true, ["Dalvanuis"] = true, ["iSonMillions"] = true,
    ["DieYouOder"] = true, ["whosframed"] = true
}

-- Function to check all players for admin presence
local function CheckAdmins()
    for _, plr in ipairs(Players:GetPlayers()) do
        if AdminList[plr.Name] then
            LocalPlayer:Kick("Admin Detected")
            task.wait(2)
            game:Shutdown()
            return
        end
    end
end

-- Enable Admin Check
local function AdminCheck_Enable()
    if AdminCheck_Enabled then return end
    AdminCheck_Enabled = true
    CheckAdmins()

    AdminCheck_Connection = Players.PlayerAdded:Connect(function(plr)
        if not AdminCheck_Enabled then return end
        if AdminList[plr.Name] then
            LocalPlayer:Kick("Admin Joined")
            task.wait(2)
            game:Shutdown()
        end
    end)

    AdminCheck_Coroutine = coroutine.create(function()
        while AdminCheck_Enabled do
            CheckAdmins()
            task.wait(4)
        end
    end)
    coroutine.resume(AdminCheck_Coroutine)
end

-- Disable Admin Check
local function AdminCheck_Disable()
    if not AdminCheck_Enabled then return end
    AdminCheck_Enabled = false
    if AdminCheck_Connection then
        AdminCheck_Connection:Disconnect()
        AdminCheck_Connection = nil
    end
    AdminCheck_Coroutine = nil
end

-- Toggle for UI (Assigned to MiscLeft)
MiscLeft:AddToggle('AdminCheckToggle', {
    Text = "Admin Check",
    Default = false,
    Tooltip = "Automatically kicks you if a known admin joins.",
    Callback = function(Value)
        if Value then
            AdminCheck_Enable()
        else
            AdminCheck_Disable()
        end
    end
})

-- Optional: Auto update player list when players join/leave
game.Players.PlayerAdded:Connect(function()
    playerDropdown:SetValues(getPlayerNames())
end)
game.Players.PlayerRemoving:Connect(function()
    playerDropdown:SetValues(getPlayerNames())
end)
-- Infinite Pepper Spray Toggle (MiscLeft)
local pepperEnabled = false

MiscLeft:AddToggle('InfinitePepperMisc', {
    Text = "Infinite Pepper Spray",
    Default = false,
    Tooltip = "Grants unlimited ammo for your pepper spray.",
    Callback = function(Value)
        pepperEnabled = Value
    end
})

-- Function to adjust pepper spray ammo
local function pepper(obj)
    if pepperEnabled then
        obj:FindFirstChild("Ammo").MinValue = 100
        obj:FindFirstChild("Ammo").Value = 100
    else
        obj:FindFirstChild("Ammo").MinValue = 0
    end
end

-- Update pepper spray ammo every frame
game:GetService("RunService").RenderStepped:Connect(function()
    local Pepper = game.Players.LocalPlayer.Character:FindFirstChild("Pepper-spray")
    if Pepper then
        pepper(Pepper)
    end
end)

-- Pepper Spray Aura Toggle (MiscLeft)
local PepperSprayAura_Enabled = false

MiscLeft:AddToggle('PepperAuraMisc', {
    Text = "PepperSpray Aura",
    Default = false,
    Tooltip = "Automatically sprays nearby players within 15 studs.",
    Callback = function(State)
        PepperSprayAura_Enabled = State

        if PepperSprayAura_Enabled then
            task.spawn(function()
                while PepperSprayAura_Enabled do
                    game:GetService("RunService").RenderStepped:Wait()
                    local player = game.Players.LocalPlayer
                    local char = player.Character
                    local pepperTool = char and char:FindFirstChild("Pepper-spray")

                    if pepperTool then
                        for _, v in pairs(game.Players:GetPlayers()) do
                            if v ~= player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
                                local dist = (char.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
                                if dist < 15 then
                                    pepperTool.RemoteEvent:FireServer("Spray", true)
                                    pepperTool.RemoteEvent:FireServer("Hit", v.Character)
                                else
                                    pepperTool.RemoteEvent:FireServer("Spray", false)
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
})


-- Fast Pickup
local fastPickUpEnabled = false
local proximityPrompts = {}

workspace.DescendantAdded:Connect(function(item)
    if item:IsA("ProximityPrompt") then
        proximityPrompts[item] = { originalDuration = item.HoldDuration }
        item.AncestryChanged:Connect(function(_, parent)
            if not parent then proximityPrompts[item] = nil end
        end)
    end
end)

RunService.RenderStepped:Connect(function()
    for prompt, data in pairs(proximityPrompts) do
        if prompt:IsA("ProximityPrompt") then
            prompt.HoldDuration = fastPickUpEnabled and 0 or data.originalDuration
        end
    end
end)

MiscLeft:AddToggle('FastPickUpToggle', {
    Text = 'Instant Pickup',
    Default = false,
    Tooltip = 'Pickup items instantly',
    Callback = function(enabled)
        fastPickUpEnabled = enabled
    end
})



-- Auto Respawn
local autoRespawn = false
local deathRespawnEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("DeathRespawn")

local function checkAndRespawn()
    while autoRespawn do
        local character = LocalPlayer.Character
        local humanoid = character and character:FindFirstChildWhichIsA("Humanoid")
        if humanoid and humanoid.Health <= 0 then
            deathRespawnEvent:InvokeServer("KMG4R904")
        end
        wait(2)
    end
end

MiscLeft:AddToggle('AutoRespawnToggle', {
    Text = 'Auto Respawn',
    Default = false,
    Tooltip = 'Respawns you when you die',
    Callback = function(enabled)
        autoRespawn = enabled
        if enabled then
            task.spawn(checkAndRespawn)
        end
    end
})

-- Anti Flash
MiscLeft:AddToggle('AntiFlashBangToggle', {
    Text = 'Anti-Flash',
    Default = false,
    Tooltip = 'Blocks flashbang effect',
    Callback = function(enabled)
        _G.NoFlashBang = enabled

        workspace.Camera.ChildAdded:Connect(function(item)
            if item.Name == "BlindEffect" and _G.NoFlashBang then
                item.Enabled = false
            end
        end)

        LocalPlayer.PlayerGui.ChildAdded:Connect(function(item)
            if item.Name == "FlashedGUI" and _G.NoFlashBang then
                item.Enabled = false
            end
        end)
    end
})

-- Anti Overlay
MiscLeft:AddToggle('AntiOverlayToggle', {
    Text = 'Anti-Overlay',
    Default = false,
    Tooltip = 'Disables visual overlays',
    Callback = function(enabled)
        _G.NoOverlay = enabled
        LocalPlayer.PlayerGui.ChildAdded:Connect(function(item)
            if item.Name == "OverlayGUI" then
                item.Enabled = not _G.NoOverlay
            end
        end)

        local overlay = LocalPlayer.PlayerGui:FindFirstChild("OverlayGUI")
        if overlay then
            overlay.Enabled = not _G.NoOverlay
        end
    end
})

-- No Visor/Helmet
MiscLeft:AddToggle('NoVisorToggle', {
    Text = 'Anti Visor',
    Default = false,
    Tooltip = 'Disables helmet overlay GUI',
    Callback = function(enabled)
        for _, gui in pairs(LocalPlayer.PlayerGui:GetDescendants()) do
            if gui.Name == "HelmetOverlayGUI" then
                gui.Enabled = not enabled
                gui:GetPropertyChangedSignal("Enabled"):Connect(function()
                    if enabled then gui.Enabled = false end
                end)
            end
        end
    end
})

-- NoClip
getgenv().NoClipEnabled = false
MiscLeft:AddToggle('NoClipToggle', {
    Text = 'NoClip',
    Default = false,
    Tooltip = 'Walk through walls',
    Callback = function(enabled)
        getgenv().NoClipEnabled = enabled

        local function applyNoClip()
            if LocalPlayer.Character then
                for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = not enabled
                    end
                end
            end
        end

        if enabled then
            RunService.Stepped:Connect(function()
                if getgenv().NoClipEnabled then
                    applyNoClip()
                end
            end)
        else
            applyNoClip()
        end
    end
})
local TeleportSettings = {
    SelectedTarget = nil,
    LoopTeleport = false
}

-- 1. Safely get the BredMakurz folder
local teleportFolder = workspace:WaitForChild("Map"):WaitForChild("BredMakurz")

-- 2. Get all model names inside the folder
local modelNames = {}
for _, model in ipairs(teleportFolder:GetChildren()) do
    if model:IsA("Model") then
        table.insert(modelNames, model.Name)
    end
end

-- 3. Dropdown UI for selecting a model
MiscRight:AddDropdown('Safes', {
    Values = modelNames,
    Default = 1,
    Multi = false,
    Text = 'Choose Safe',
    Tooltip = 'Select Safe',
    Callback = function(selected)
        
        TeleportSettings.SelectedTarget = selected
    end
})

-- 4. Toggle UI for looping teleport to that model's MainPart
MiscRight:AddToggle('LoopTP', {
    Text = 'Teleport',
    Default = false,
    Tooltip = 'Telepprt To Safe',
    Callback = function(enabled)
        TeleportSettings.LoopTeleport = enabled
        
        if enabled then
            task.spawn(function()
                while TeleportSettings.LoopTeleport do
                    local player = game.Players.LocalPlayer
                    local char = player.Character or player.CharacterAdded:Wait()
                    local hrp = char:WaitForChild("HumanoidRootPart")
                    local modelName = TeleportSettings.SelectedTarget

                    if not modelName then
                        warn("No model selected!")
                        break
                    end

                    local model = teleportFolder:FindFirstChild(modelName)
                    if model and model:IsA("Model") then
                        local main = model:FindFirstChild("MainPart")
                        if main and main:IsA("BasePart") then
                            
                            hrp.CFrame = main.CFrame + Vector3.new(0, 5, 0)
                        else
                            warn("Model has no MainPart:", modelName)
                        end
                    else
                        warn("Model not found:", modelName)
                    end

                    task.wait(1)
                end
            end)
        end
    end
})
functions = functions or {}
local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")

local workspace = game:GetService("Workspace")
local replicatedStorage = game:GetService("ReplicatedStorage")

local toolsFolder = workspace:WaitForChild("Filter"):WaitForChild("SpawnedTools")
local cashFolder = workspace:WaitForChild("Filter"):WaitForChild("SpawnedBread")
local pilesFolder = workspace:WaitForChild("Filter"):WaitForChild("SpawnedPiles")

local pickupMethod = "Without Remote Event"
local cooldown = 0.8
local canPickup = true
local lastPickupTime = 0

local toolsEnabled, cashEnabled, scrapsEnabled, cratesEnabled = false, false, false, false
local toolsConnection, moneyConnection, scrapsConnection, cratesConnection = nil, nil, nil, nil

local function interactWithPrompt(v)
	if v:IsA("ProximityPrompt") and canPickup then
		v.HoldDuration = 0
		fireproximityprompt(v)
		canPickup = false
		lastPickupTime = tick()
	end
end

local function pickupWithoutRemote(v)
	if toolsEnabled and v:IsA("Model") and toolsFolder:FindFirstChild(v.Name) then
		for _, p in ipairs(v:GetDescendants()) do interactWithPrompt(p) end
	elseif cashEnabled and v:IsA("BasePart") and v.Name == "CashDrop1" then
		for _, p in ipairs(v:GetChildren()) do interactWithPrompt(p) end
	elseif scrapsEnabled and v:IsA("Model") and (v.Name == "S1" or v.Name == "S2") then
		for _, p in ipairs(v:GetDescendants()) do interactWithPrompt(p) end
	elseif cratesEnabled and v:IsA("Model") and (v.Name == "C1" or v.Name == "C2" or v.Name == "C3") then
		for _, p in ipairs(v:GetDescendants()) do interactWithPrompt(p) end
	end
end

local function scanItems()
	while toolsEnabled or cashEnabled or scrapsEnabled or cratesEnabled do
		if not canPickup and tick() - lastPickupTime >= cooldown then
			canPickup = true
		end
		for _, v in ipairs(toolsFolder:GetChildren()) do pickupWithoutRemote(v) end
		for _, v in ipairs(cashFolder:GetChildren()) do pickupWithoutRemote(v) end
		for _, v in ipairs(pilesFolder:GetChildren()) do pickupWithoutRemote(v) end
		task.wait(0.1)
	end
end

local function pickupRemote(folder, eventName, findCondition, posFn, cooldownTime, attrRev)
	local remote = replicatedStorage:WaitForChild("Events"):WaitForChild(eventName)
	local connection
	local canPickup = true
	local last = tick()

	connection = runService.RenderStepped:Connect(function()
		local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
		if not hrp then return end

		local closest, dist = nil, 15
		for _, item in pairs(folder:GetChildren()) do
			if findCondition(item) then
				local pos = posFn(item)
				local d = (hrp.Position - pos).Magnitude
				if d < dist then
					closest = item
					dist = d
				end
			end
		end

		if closest and canPickup then
			if attrRev then
				remote:FireServer(string.reverse(closest:GetAttribute(attrRev)))
			else
				remote:FireServer(closest:FindFirstChild("Handle") or closest:FindFirstChild("WeaponHandle") or closest)
			end
			canPickup = false
		elseif tick() - last >= cooldownTime then
			canPickup = true
			last = tick()
		end
	end)

	return connection
end

-- UI Elements with tooltips
MiscRight:AddToggle('ToggleScraps', {
	Text = "AutoPickup Scraps",
	Tooltip = "Automatically pick up scrap items nearby",
	Default = false,
	Callback = function(val)
		scrapsEnabled = val
		if scrapsConnection then scrapsConnection:Disconnect() end
		if val then
			if pickupMethod == "With Remote Event" then
				scrapsConnection = pickupRemote(pilesFolder, "PIC_PU", function(a)
					return a.Name == "S1" or a.Name == "S2"
				end, function(a) return a.MeshPart.Position end, 4.5, "jzu")
			else
				task.spawn(scanItems)
			end
		end
	end
})

MiscRight:AddToggle('ToggleTools', {
	Text = "AutoPickup Tools",
	Tooltip = "Automatically pick up tools dropped in the world",
	Default = false,
	Callback = function(val)
		toolsEnabled = val
		if toolsConnection then toolsConnection:Disconnect() end
		if val then
			if pickupMethod == "With Remote Event" then
				toolsConnection = pickupRemote(toolsFolder, "PIC_TLO", function(a)
					return true
				end, function(a)
					local h = a:FindFirstChild("Handle") or a:FindFirstChild("WeaponHandle")
					return h and h.Position or Vector3.new()
				end, 1.5)
			else
				task.spawn(scanItems)
			end
		end
	end
})

MiscRight:AddToggle('ToggleCrates', {
	Text = "AutoPickup Crates",
	Tooltip = "Automatically pick up crates around you",
	Default = false,
	Callback = function(val)
		cratesEnabled = val
		if cratesConnection then cratesConnection:Disconnect() end
		if val then
			if pickupMethod == "With Remote Event" then
				cratesConnection = pickupRemote(pilesFolder, "PIC_PU", function(a)
					return a.Name == "C1" or a.Name == "C2" or a.Name == "C3"
				end, function(a) return a.MeshPart.Position end, 7, "jzu")
			else
				task.spawn(scanItems)
			end
		end
	end
})

MiscRight:AddToggle('ToggleCash', {
	Text = "AutoPickup Money",
	Tooltip = "Automatically collect nearby dropped money",
	Default = false,
	Callback = function(val)
		cashEnabled = val
		if moneyConnection then moneyConnection:Disconnect() end
		if val then
			if pickupMethod == "With Remote Event" then
				moneyConnection = pickupRemote(cashFolder, "CZDPZUS", function(a)
					return a:IsA("BasePart")
				end, function(a) return a.Position end, 0.7)
			else
				task.spawn(scanItems)
			end
		end
	end
})

MiscRight:AddDropdown('PickupMethod', {
	Text = "Pickup Method",
	Tooltip = "Switch between ProximityPrompt or RemoteEvent-based pickup",
	Default = "1",
	Values = {"1", "2"},
	Callback = function(val)
		pickupMethod = val

		-- Reset all connections
		if scrapsConnection then scrapsConnection:Disconnect() end
		if toolsConnection then toolsConnection:Disconnect() end
		if cratesConnection then cratesConnection:Disconnect() end
		if moneyConnection then moneyConnection:Disconnect() end

		if scrapsEnabled then
			if val == "With Remote Event" then
				scrapsConnection = pickupRemote(pilesFolder, "PIC_PU", function(a)
					return a.Name == "S1" or a.Name == "S2"
				end, function(a) return a.MeshPart.Position end, 4.5, "jzu")
			else
				task.spawn(scanItems)
			end
		end

		if toolsEnabled then
			if val == "With Remote Event" then
				toolsConnection = pickupRemote(toolsFolder, "PIC_TLO", function(a)
					return true
				end, function(a)
					local h = a:FindFirstChild("Handle") or a:FindFirstChild("WeaponHandle")
					return h and h.Position or Vector3.new()
				end, 1.5)
			else
				task.spawn(scanItems)
			end
		end

		if cratesEnabled then
			if val == "With Remote Event" then
				cratesConnection = pickupRemote(pilesFolder, "PIC_PU", function(a)
					return a.Name == "C1" or a.Name == "C2" or a.Name == "C3"
				end, function(a) return a.MeshPart.Position end, 7, "jzu")
			else
				task.spawn(scanItems)
			end
		end

		if cashEnabled then
			if val == "With Remote Event" then
				moneyConnection = pickupRemote(cashFolder, "CZDPZUS", function(a)
					return a:IsA("BasePart")
				end, function(a) return a.Position end, 0.7)
			else
				task.spawn(scanItems)
			end
		end
	end
})

functions.AutoOpenDoorsF = false
MiscRight:AddToggle('AutoOpenDoorsToggle', {
	Text = "Auto Open Doors",
	Tooltip = "Automatically unlock and open nearby doors",
	Default = false,
	Callback = function(val)
		functions.AutoOpenDoorsF = val
		if val then
			task.spawn(function()
				while functions.AutoOpenDoorsF do
					local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
					if not hrp then runService.RenderStepped:Wait() continue end

					local map = workspace:FindFirstChild("Map")
					local folder = map and map:FindFirstChild("Doors")
					local closest, dist = nil, 15

					if folder then
						for _, door in pairs(folder:GetChildren()) do
							local db = door:FindFirstChild("DoorBase")
							if db then
								local d = (hrp.Position - db.Position).Magnitude
								if d < dist then
									closest, dist = door, d
								end
							end
						end
					end

					if closest then
						local values = closest:FindFirstChild("Values")
						local events = closest:FindFirstChild("Events")
						if values and events then
							local locked = values:FindFirstChild("Locked")
							local open = values:FindFirstChild("Open")
							local toggle = events:FindFirstChild("Toggle")
							if locked and open and toggle then
								if locked.Value then
									toggle:FireServer("Unlock", closest.Lock)
								elseif not open.Value then
									local knob1 = closest:FindFirstChild("Knob1")
									local knob2 = closest:FindFirstChild("Knob2")
									if knob1 and knob2 then
										local d1 = (hrp.Position - knob1.Position).Magnitude
										local d2 = (hrp.Position - knob2.Position).Magnitude
										local knob = d1 < d2 and knob1 or knob2
										toggle:FireServer("Open", knob)
									end
								end
							end
						end
					end

					runService.RenderStepped:Wait()
				end
			end)
		end
	end
})



getgenv().lockpickHBEEnabled = false

local function updateLockpickBars()
    local PlayerGui = game:GetService("Players").LocalPlayer:FindFirstChildOfClass("PlayerGui")
    if PlayerGui and PlayerGui:FindFirstChild("LockpickGUI") then
        local frames = PlayerGui.LockpickGUI.MF.LP_Frame.Frames
        for i = 1, 3 do
            local Bar = frames["B" .. i].Bar
            Bar.Size = getgenv().lockpickHBEEnabled and UDim2.new(0, 35, 0, 500) or UDim2.new(0, 35, 0, 30)
        end
    end
end

-- Listen for GUI appearance
game:GetService("Players").LocalPlayer:FindFirstChildOfClass("PlayerGui").ChildAdded:Connect(function(child)
    if child.Name == "LockpickGUI" then
        updateLockpickBars()
    end
end)

-- Toggle (in MiscRight)
MiscRight:AddToggle('LockpickHBEToggle', {
    Text = 'Auto Lockpick',
    Default = false,
    Tooltip = 'Stretch lockpick bars to make minigame easier',
    Callback = function(value)
        getgenv().lockpickHBEEnabled = value
        updateLockpickBars()
    end
})
getgenv().AutoClaimAllowanceEnabled = false
local AutoClaimAllowanceCoolDown = false

-- Find nearest ATM
local function GetATM(maxDistance)
    local nearestATM
    for _, v in ipairs(workspace.Map.ATMz:GetChildren()) do
        local part = v:FindFirstChild("MainPart")
        if part then
            local distance = (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - part.Position).Magnitude
            if distance < maxDistance then
                maxDistance = distance
                nearestATM = part
            end
        end
    end
    return nearestATM
end

-- Claim logic
game:GetService("RunService").RenderStepped:Connect(function()
    if getgenv().AutoClaimAllowanceEnabled and not AutoClaimAllowanceCoolDown then
        local player = game.Players.LocalPlayer
        local nextAllowance = game:GetService("ReplicatedStorage").PlayerbaseData2[player.Name].NextAllowance.Value
        if nextAllowance == 0 then
            local atm = GetATM(math.huge)
            if atm then
                AutoClaimAllowanceCoolDown = true
                task.spawn(function()
                    game:GetService("ReplicatedStorage").Events.CLMZALOW:InvokeServer(atm)
                    task.wait(0.5)
                    AutoClaimAllowanceCoolDown = false
                end)
            end
        end
    end
end)

-- Toggle (in MiscRight)
MiscRight:AddToggle('AutoClaimAllowanceToggle', {
    Text = 'Auto Claim Allowance',
    Default = false,
    Tooltip = 'Automatically claims allowance when near an ATM',
    Callback = function(value)
        getgenv().AutoClaimAllowanceEnabled = value
    end
})


local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Positions for teleporting
local targetPositions = {
    Vector3.new(-4627.25, 1.67, -980.48),      -- 1st updated
    Vector3.new(-5000.67, 1.95, -373.10),      -- 2nd updated
    Vector3.new(-4790.30, 1.95, -383.11),      -- 3rd updated
    Vector3.new(-4789.91, 1.96, -372.03),      -- 4th updated
    Vector3.new(-4789.91, 1.96, -372.03),      -- 5th updated
    Vector3.new(-4789.91, 1.96, -372.03),      -- 6th updated
    Vector3.new(-4789.91, 1.96, -372.03),      -- 7th added
    Vector3.new(-4789.91, 1.96, -372.03)       -- 8th added
}


local maxAllowances = 3
local allowanceCooldown = 45 * 60
local allowanceDuration = 15 * 60
local teleportStepDuration = 8

local allowancesUsed = 0
local lastClaimTime = 0

local AnimationID = "rbxassetid://282574440"
local AnimationTrack
local Animator
local Clip = false

local NoclipConnection
local RemoveLegsConnection
local RemoveAccessoriesConnection

getgenv().AutoFarmAndNoclipToggle = false

-- Helper: get root part
local function getRootPart()
    local char = player.Character or player.CharacterAdded:Wait()
    return char:FindFirstChild("HumanoidRootPart")
end

-- Teleport loop for duration
local function teleportTo(position, duration)
    local startTime = os.time()
    while os.time() - startTime < duration do
        if not getgenv().AutoFarmAndNoclipToggle then return end
        local rp = getRootPart()
        if rp then
            pcall(function()
                rp.CFrame = CFrame.new(position)
            end)
        end
        wait(0.1)
    end
end

-- Try to claim allowance logic
local function tryClaimAllowance()
    local currentTime = os.time()
    if allowancesUsed >= maxAllowances and (currentTime - lastClaimTime) < allowanceCooldown then
        return false
    end
    if (currentTime - lastClaimTime) >= allowanceCooldown then
        allowancesUsed = 0
    end
    allowancesUsed += 1
    lastClaimTime = currentTime
    return true
end

-- Autofarm main loop, cycles positions infinitely
local function autofarmLoop()
    while getgenv().AutoFarmAndNoclipToggle do
        for _, pos in ipairs(targetPositions) do
            if not getgenv().AutoFarmAndNoclipToggle then break end
            teleportTo(pos, teleportStepDuration)
            if tryClaimAllowance() then
                
                wait(allowanceDuration)
            else
                
                wait(allowanceCooldown)
            end
        end
    end
end

-- Noclip and animation features
local function noclipLoop()
    if not Clip and player.Character then
        for _, part in pairs(player.Character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide == true then
                part.CanCollide = false
            end
        end
    end
end

local function removeLegsLoop()
    local char = player.Character
    if char then
        local rightLeg = char:FindFirstChild("Right Leg")
        local leftLeg = char:FindFirstChild("Left Leg")
        if rightLeg then rightLeg.Parent = nil end
        if leftLeg then leftLeg.Parent = nil end
    end
end

local function removeAccessoriesLoop()
    local char = player.Character
    if char then
        for _, accessory in pairs(char:GetChildren()) do
            if accessory:IsA("Accessory") then
                accessory:Destroy()
            end
        end
    end
end

local function playLoopingAnimation()
    local char = player.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    Animator = humanoid:FindFirstChildOfClass("Animator")
    if not Animator then
        Animator = Instance.new("Animator", humanoid)
    end
    local anim = Instance.new("Animation")
    anim.AnimationId = AnimationID
    AnimationTrack = Animator:LoadAnimation(anim)
    AnimationTrack.Looped = true
    AnimationTrack:Play()
end

local function enableNoclipBriefly()
    Clip = false
    NoclipConnection = RunService.Stepped:Connect(noclipLoop)
    task.delay(0.1, function()
        Clip = true
        if NoclipConnection then
            NoclipConnection:Disconnect()
            NoclipConnection = nil
        end
    end)
end

local function startNoclipFeatures()
    playLoopingAnimation()
    RemoveLegsConnection = RunService.Stepped:Connect(removeLegsLoop)
    RemoveAccessoriesConnection = RunService.Stepped:Connect(removeAccessoriesLoop)
    enableNoclipBriefly()
end

local function stopNoclipFeatures()
    if AnimationTrack then
        AnimationTrack:Stop()
        AnimationTrack = nil
    end
    if RemoveLegsConnection then
        RemoveLegsConnection:Disconnect()
        RemoveLegsConnection = nil
    end
    if RemoveAccessoriesConnection then
        RemoveAccessoriesConnection:Disconnect()
        RemoveAccessoriesConnection = nil
    end
    Clip = true
end

-- On respawn, restart noclip and autofarm if toggle enabled
player.CharacterAdded:Connect(function()
    if getgenv().AutoFarmAndNoclipToggle then
        task.wait(1)
        startNoclipFeatures()
        task.spawn(autofarmLoop)
    else
        stopNoclipFeatures()
    end
end)

-- The single toggle controlling both autofarm and noclip
MiscRight:AddToggle('autofarmandnocliptoggle', {
    Text = 'Allowance Autofarm V2',
    Default = false,
    Tooltip = 'Allowance Autofarm',
    Callback = function(value)
        getgenv().AutoFarmAndNoclipToggle = value
        if value then
            startNoclipFeatures()
            task.spawn(autofarmLoop)
        else
            stopNoclipFeatures()
        end
    end
})
local EspLeft = Tabs.Esp:AddLeftGroupbox('Player Esp')

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Global Variables
getgenv().chamsEnabled = false
getgenv().chamsFillColor = Color3.fromRGB(175, 25, 255)
getgenv().chamsFillTransparency = 0.5
getgenv().chamsOutlineColor = Color3.fromRGB(255, 255, 255)
getgenv().chamsOutlineTransparency = 0

getgenv().ESPEnabled = false
getgenv().ESPShowNameDist = true
getgenv().ESPShowHealth = true
getgenv().ESPShowTracer = true
getgenv().ESPTracerColor = Color3.fromRGB(0, 255, 0)

-- Highlight storage
local storage = Instance.new("Folder", CoreGui)
storage.Name = "Highlight_Storage"

local function applyChams(player)
    if not getgenv().chamsEnabled or player == LocalPlayer then return end
    local highlight = storage:FindFirstChild(player.Name)
    if not highlight then
        highlight = Instance.new("Highlight")
        highlight.Name = player.Name
        highlight.Parent = storage
    end
    highlight.FillColor = getgenv().chamsFillColor
    highlight.FillTransparency = getgenv().chamsFillTransparency
    highlight.OutlineColor = getgenv().chamsOutlineColor
    highlight.OutlineTransparency = getgenv().chamsOutlineTransparency
    if player.Character then
        highlight.Adornee = player.Character
    end
    player.CharacterAdded:Connect(function(character)
        highlight.Adornee = character
    end)
end

local function removeChams(player)
    local highlight = storage:FindFirstChild(player.Name)
    if highlight then
        highlight:Destroy()
    end
end

local ESPObjects = {}

local function CreateESP(player)
    if player == LocalPlayer then return end
    local textLabel = Drawing.new("Text")
    textLabel.Visible = false
    textLabel.Center = true
    textLabel.Outline = true
    textLabel.Font = 2
    textLabel.Size = 13
    textLabel.Color = Color3.new(1, 1, 1)
    local tracerLine = Drawing.new("Line")
    tracerLine.Visible = false
    tracerLine.Color = getgenv().ESPTracerColor
    tracerLine.Thickness = 1.5
    ESPObjects[player] = {text = textLabel, tracer = tracerLine}
end

local function RemoveESP(player)
    if ESPObjects[player] then
        ESPObjects[player].text:Remove()
        ESPObjects[player].tracer:Remove()
        ESPObjects[player] = nil
    end
end

Players.PlayerAdded:Connect(function(player)
    applyChams(player)
    CreateESP(player)
end)

Players.PlayerRemoving:Connect(function(player)
    removeChams(player)
    RemoveESP(player)
end)

for _, player in ipairs(Players:GetPlayers()) do
    applyChams(player)
    CreateESP(player)
end

RunService.RenderStepped:Connect(function()
    if not getgenv().ESPEnabled then return end
    for player, esp in pairs(ESPObjects) do
        local character = player.Character
        if character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("Humanoid") then
            local hrp = character.HumanoidRootPart
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
            if onScreen and dist < 100000 then
                local displayText = ""
                if getgenv().ESPShowNameDist then
                    displayText = string.format("%s [%.0f]", player.Name, dist)
                end
                if getgenv().ESPShowHealth then
                    displayText = displayText .. string.format(" | HP: %d", math.floor(character.Humanoid.Health))
                end
                esp.text.Text = displayText
                esp.text.Position = Vector2.new(pos.X, pos.Y - 30)
                esp.text.Visible = true
                if getgenv().ESPShowTracer then
                    esp.tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    esp.tracer.To = Vector2.new(pos.X, pos.Y)
                    esp.tracer.Color = getgenv().ESPTracerColor
                    esp.tracer.Visible = true
                else
                    esp.tracer.Visible = false
                end
            else
                esp.text.Visible = false
                esp.tracer.Visible = false
            end
        else
            esp.text.Visible = false
            esp.tracer.Visible = false
        end
    end
end)

-- ==== UI ====

EspLeft:AddToggle('ChamsToggle', {
    Text = 'Chams',
    Default = false,
    Tooltip = 'Toggle player chams highlight',
    Callback = function(value)
        getgenv().chamsEnabled = value
        for _, player in pairs(Players:GetPlayers()) do
            if value then
                applyChams(player)
            else
                removeChams(player)
            end
        end
    end
})

EspLeft:AddLabel('Chams Fill Color'):AddColorPicker('ChamsFillColor', {
    Default = getgenv().chamsFillColor,
    Title = 'Chams Fill Color',
    Transparency = nil,
    Callback = function(color)
        getgenv().chamsFillColor = color
        for _, player in pairs(Players:GetPlayers()) do
            applyChams(player)
        end
    end
})

EspLeft:AddLabel('Chams Outline Color'):AddColorPicker('ChamsOutlineColor', {
    Default = getgenv().chamsOutlineColor,
    Title = 'Chams Outline Color',
    Transparency = nil,
    Callback = function(color)
        getgenv().chamsOutlineColor = color
        for _, player in pairs(Players:GetPlayers()) do
            applyChams(player)
        end
    end
})

EspLeft:AddToggle('ESPEnabledToggle', {
    Text = 'ESP',
    Default = false,
    Tooltip = 'Toggle ESP drawings',
    Callback = function(value)
        getgenv().ESPEnabled = value
    end
})

EspLeft:AddToggle('ESPShowNameDistToggle', {
    Text = 'ESP: Name + Distance',
    Default = true,
    Tooltip = 'Show player names and distances',
    Callback = function(value)
        getgenv().ESPShowNameDist = value
    end
})

EspLeft:AddToggle('ESPShowHealthToggle', {
    Text = 'ESP: Health',
    Default = true,
    Tooltip = 'Show player health',
    Callback = function(value)
        getgenv().ESPShowHealth = value
    end
})

EspLeft:AddToggle('ESPShowTracerToggle', {
    Text = 'ESP: Tracers',
    Default = true,
    Tooltip = 'Show tracer lines',
    Callback = function(value)
        getgenv().ESPShowTracer = value
    end
})

EspLeft:AddLabel('ESP Tracer Color'):AddColorPicker('ESPTracerColor', {
    Default = getgenv().ESPTracerColor,
    Title = 'ESP Tracer Color',
    Transparency = nil,
    Callback = function(color)
        getgenv().ESPTracerColor = color
    end
})


-- UI Settings Tab
local MenuGroup = Tabs['UI Settings']:AddLeftGroupbox('Menu')

MenuGroup:AddToggle("KeybindMenuOpen", {
    Default = Library.KeybindFrame.Visible,
    Text = "Open Keybind Menu",
    Callback = function(value)
        Library.KeybindFrame.Visible = value
    end
})

MenuGroup:AddToggle("ShowCustomCursor", {
    Text = "Custom Cursor",
    Default = false,
    Callback = function(Value)
        Library.ShowCustomCursor = Value
    end
})

MenuGroup:AddDivider()

MenuGroup:AddLabel("Menu bind"):AddKeyPicker("MenuKeybind", {
    Default = "RightShift",
    NoUI = true,
    Text = "Menu keybind"
})

MenuGroup:AddButton("Unload", function()
    Library:Unload()
end)

Library:OnUnload(function()
    if staminaLoop then
        staminaLoop:Disconnect()
    end
    print('Unloaded!')
    Library.Unloaded = true
end)

Library.ToggleKeybind = Options.MenuKeybind

-- Theme and Save Setup
ThemeManager:SetLibrary(Library)
SaveManager:SetLibrary(Library)

SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({ 'MenuKeybind' })


ThemeManager:SetFolder('UndetectedWare')
SaveManager:SetFolder('UndetectedWare/Criminality')
SaveManager:SetSubFolder('Place1')

SaveManager:BuildConfigSection(Tabs['UI Settings'])
ThemeManager:ApplyToTab(Tabs['UI Settings'])

SaveManager:LoadAutoloadConfig()
