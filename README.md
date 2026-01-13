--HUK被遗弃
--由HuK Team和zy制作

--加载WindUI库
本地WindUI=loadstring(游戏：HttpGet("https://raw.githubusercontent.com/Syndromehsh/Lua/refs/heads/main/AlienX/AlienX%20Wind%203.0%20UI.txt"))()

WindUI：通知({
title="HuK"，
内容="被遗弃-高级版"，
持续时间=4
})

本地玩家=game.Players.LocalPlayer

本地窗口=WindUI:CreateWindow({
title="Huk<字体颜色='#00FF00'>2.0</字体>/Huk被遗弃-高级版"，
icon="rbxassetid://4483362748"，
IconTransparency=0.5，
作者="Huk"，
文件夹="HuK"，
尺寸=UDim2。从偏移(100，150)，
透明=真，
主题="黑暗"，
UserEnabled=true，
SidebarWidth=145，
HasOutline=true，
用户={
enabled=true，
anonymous=false，
username=player.name，
displayName=player.DisplayName，
userid=player.UserId，
缩略图="https://www.roblox.com/headshot-thumbnail/image?userId="..播放器.userid.."&width=420&height=420&format=png"，
回调=函数()
WindUI：通知({
标题="用户信息"，
content="玩家："..player.name.."("..play.displayName..")"，
持续时间=3
            })
结束
    }
})

窗口：EditOpenButton({
title="AlienX"，
图标="监视器"，
角半径=UDim.new(1，10)，
行程厚度=2，
color=ColorSequence.new({
ColorSequenceKeypoint.New(0，Color3.FromHex("FF0000"))，
ColorSequenceKeypoint。新(0.16，颜色3.FromHex("FF7F00"))，
ColorSequenceKeypoint。新(0.33，颜色3.FromHex("FFFF00"))，
ColorSequenceKeypoint.新建(0.5，Color3.FromHex("00FF00"))，
ColorSequenceKeypoint.New(0.66，Color3.FromHex("0000FF"))，
ColorSequenceKeypoint。新(0.83，颜色3.FromHex("4B0082"))，
ColorSequenceKeypoint.New(1，Color3.FromHex("9400D3"))
    }),
可拖动=真，
})

-- ========== 创建选项卡 ==========
本地选项卡={
主=窗口：节({Title="主要功能"，opened=true})，
visual=窗口：节({Title="视觉功能"，opened=true})，
播放器=窗口：节({Title="玩家功能"，opened=true})，
anti=窗口：节({Title="阻止功能"，opened=true})
}

本地TabHandles={
elements=Tabs.Main:Tab({Title="全局设置"})，
幸存者=tabs.Main:Tab({Title="幸存者功能"})，
    Visual = Tabs.Visual:Tab({ Title = "透视绘制" }),
    Player = Tabs.Player:Tab({ Title = "玩家设置" }),
    Anti = Tabs.Anti:Tab({ Title = "阻止设置" })
}
Window:SelectTab(1)

-- ========== 全局变量 ==========
_G.REP = 1.8
_G.BTE = false

-- ========== 全局设置部分 ==========
TabHandles.Elements:Section({ Title = "全局设置" })

-- 修电箱延迟
local repairSlider = TabHandles.Elements:Slider({
    Title = "修电箱延迟[秒]",
    Step = 0.1,
    Value = { Min = 1.8, Max = 10, Default = _G.REP },
    Callback = function(value)
        _G.REP = value
        print("修理间隔设置为:", string.format("%.2f", _G.REP), "秒")
    end
})

-- 自动修电箱
local repairToggle = TabHandles.Elements:Toggle({
    Title = "自动修电箱",
    Default = false,
    Callback = function(state)
        _G.BTE = state

        local function RepairGenerators()
            local map = workspace:FindFirstChild("Map")
            local ingame = map and map:FindFirstChild("Ingame")
            local currentMap = ingame and ingame:FindFirstChild("Map")

            if currentMap then
                for _, obj in ipairs(currentMap:GetChildren()) do
                    if obj.Name == "Generator" and obj:FindFirstChild("Progress") and obj.Progress.Value < 100 then
                        local remote = obj:FindFirstChild("Remotes") and obj.Remotes:FindFirstChild("RE")
                        if remote then
                            remote:FireServer()
                        end
                    end
                end
            end
        end

        if state then
            task.spawn(function()
                while _G.BTE do
                    RepairGenerators()
                    task.wait(_G.REP or 1.80) 
                end
            end)
        end
    end
})

-- 无限体力
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local sprintModule
local isStaminaDrainDisabled = false
local staminaMonitorConnection = nil

local function modifyStaminaSettings()
    pcall(function()
        if not sprintModule then
            local success, module = pcall(require, ReplicatedStorage.Systems.Character.Game.Sprinting)
            if success and module then
                sprintModule = module
                return
            end
        end
        if sprintModule and sprintModule.StaminaLossDisabled ~= nil then
             sprintModule.StaminaLossDisabled = isStaminaDrainDisabled
        end
    end)
end

local function monitorAndReapplyStamina()
    if staminaMonitorConnection then
        staminaMonitorConnection:Disconnect()
    end
    staminaMonitorConnection = RunService.Heartbeat:Connect(function()
        if isStaminaDrainDisabled then
            modifyStaminaSettings()
        else
            if staminaMonitorConnection then
                staminaMonitorConnection:Disconnect()
                staminaMonitorConnection = nil
            end
        end
    end)
end

local infiniteStaminaToggle = TabHandles.Elements:Toggle({
    Title = "无限体力",
    Default = false,
    Callback = function(state)
        isStaminaDrainDisabled = state
        modifyStaminaSettings()
        
        if state then
            monitorAndReapplyStamina()
        else
            if staminaMonitorConnection then
                staminaMonitorConnection:Disconnect()
                staminaMonitorConnection = nil
            end
            if sprintModule and sprintModule.StaminaLossDisabled ~= nil then
                sprintModule.StaminaLossDisabled = false 
            end
        end
    end
})

-- 恢复体力按钮
local function restoreStamina()
    pcall(function()
        local SprintingModule = ReplicatedStorage:WaitForChild("Systems"):WaitForChild("Character"):WaitForChild("Game"):WaitForChild("Sprinting")
        local sprintModule = require(SprintingModule)
        
        if sprintModule and sprintModule.SetStamina then
            sprintModule.SetStamina(sprintModule.MaxStamina or 100)
        end
    end)
end

TabHandles.Elements:Button({
    Title = "恢复体力",
    Callback = function()
        restoreStamina()
    end
})

-- ========== 幸存者功能部分 ==========
TabHandles.Survivors:Section({ Title = "Shedletsky幸存者" })

-- 自动斩击
local autoSlashEnabled = false
local slashConnection = nil

local function checkAndSlash()
    local player = game.Players.LocalPlayer
    if not player or not player.Character then return end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    local killersFolder = workspace:FindFirstChild("Players")
    if not killersFolder then return end
    
    local killers = killersFolder:FindFirstChild("Killers")
    if not killers then return end
    
    local playerPosition = humanoidRootPart.Position
    
    for _, killer in ipairs(killers:GetChildren()) do
        local killerRoot = killer:FindFirstChild("HumanoidRootPart")
        if killerRoot then
            local distance = (playerPosition - killerRoot.Position).Magnitude
            if distance <= 10 then
                local args = {
                    [1] = "UseActorAbility",
                    [2] = "Slash"
                }
                game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
                break
            end
        end
    end
end

local autoSlashToggle = TabHandles.Survivors:Toggle({
    Title = "自动斩击(被动)",
    Default = false,
    Callback = function(state)
        autoSlashEnabled = state
        
        if state then
            if slashConnection then
                slashConnection:Disconnect()
            end
            slashConnection = RunService.Heartbeat:Connect(function()
                if autoSlashEnabled then
                    checkAndSlash()
                end
            end)
        else
            if slashConnection then
                slashConnection:Disconnect()
                slashConnection = nil
            end
        end
    end
})

-- Shedletsky自动瞄准
local shedletskyAimbotEnabled = false
local shedloop = nil

local function shedletskyAimbot(state)
    shedletskyAimbotEnabled = state
    
    if state then
        if game:GetService("Players").LocalPlayer.Character.Name ~= "Shedletsky" then
            return
        end
        
        shedloop = game:GetService("Players").LocalPlayer.Character.Sword.ChildAdded:Connect(function(child)
            if not shedletskyAimbotEnabled then return end
            if child:IsA("Sound") then 
                local FAN = child.Name
                if FAN == "rbxassetid://12222225" or FAN == "83851356262523" then 
                    local killersFolder = game.Workspace.Players:FindFirstChild("Killers")
                    if killersFolder then 
                        local killer = killersFolder:FindFirstChildOfClass("Model")
                        if killer and killer:FindFirstChild("HumanoidRootPart") then 
                            local killerHRP = killer.HumanoidRootPart
                            local playerHRP = game:GetService("Players").LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                            if playerHRP then 
                                local distance = (killerHRP.Position - playerHRP.Position).Magnitude
                               
                                if distance <= 30 then
                                    local num = 1
                                    local maxIterations = 100
                                    while num <= maxIterations do
                                        task.wait(0.01)
     num = num + 1
                                        workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, killerHRP.Position)
                                        playerHRP.CFrame = CFrame.lookAt(playerHRP.Position, killerHRP.Position)
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end)
    else
        if shedloop then 
            shedloop:Disconnect()
            shedloop = nil
        end
    end
end

local shedAimbotToggle = TabHandles.Survivors:Toggle({
    Title = "自动瞄准",
    Default = false,
    Callback = function(state)
        shedletskyAimbot(state)
    end
})

-- Chance幸存者
TabHandles.Survivors:Section({ Title = "Chance幸存者" })

local PredictionAim = {
    Enabled = false,
    Prediction = 1,
    Duration = 1.7,
    MaxDistance = 50,
    Targets = { "Jason", "c00lkidd", "JohnDoe", "1x1x1x1", "Noli" },
    TrackedAnimations = {
        ["103601716322988"] = true, ["133491532453922"] = true, ["86371356500204"] = true,
        ["76649505662612"] = true, ["81698196845041"] = true
    },
    Humanoid = nil,
    HRP = nil,
    LastTriggerTime = 0,
    IsAiming = false,
    OriginalState = nil
}

local function setupCharacter(char)
    PredictionAim.Humanoid = char:WaitForChild("Humanoid")
    PredictionAim.HRP = char:WaitForChild("HumanoidRootPart")
end

local function getValidTarget()
    local killersFolder = workspace:FindFirstChild("Players") and workspace.Players:FindFirstChild("Killers")
    if killersFolder then
        for _, name in ipairs(PredictionAim.Targets) do
            local target = killersFolder:FindFirstChild(name)
            if target and target:FindFirstChild("HumanoidRootPart") then
                if PredictionAim.HRP and (PredictionAim.HRP.Position - target.HumanoidRootPart.Position).Magnitude <= PredictionAim.MaxDistance then
                    return target.HumanoidRootPart
                end
            end
        end
    end
    return nil
end

local function getPlayingAnimationIds()
    local ids = {}
    if PredictionAim.Humanoid then
        for _, track in ipairs(PredictionAim.Humanoid:GetPlayingAnimationTracks()) do
            if track.Animation and track.Animation.AnimationId then
                local id = track.Animation.AnimationId:match("%d+")
                if id then ids[id] = true end
            end
        end
    end
    return ids
end

local function OnRenderStep()
    if not PredictionAim.Enabled or not PredictionAim.Humanoid or not PredictionAim.HRP then return end
    local playing = getPlayingAnimationIds()
    local triggered = false
    for id in pairs(PredictionAim.TrackedAnimations) do
        if playing[id] then triggered = true; break end
    end

    if triggered then
        PredictionAim.LastTriggerTime = tick()
        PredictionAim.IsAiming = true
    end

    if PredictionAim.IsAiming and tick() - PredictionAim.LastTriggerTime <= PredictionAim.Duration then
        if not PredictionAim.OriginalState then
            PredictionAim.OriginalState = {
                WalkSpeed = PredictionAim.Humanoid.WalkSpeed,
                JumpPower = PredictionAim.Humanoid.JumpPower,
                AutoRotate = PredictionAim.Humanoid.AutoRotate
            }
            PredictionAim.Humanoid.AutoRotate = false
            PredictionAim.HRP.AssemblyAngularVelocity = Vector3.zero
        end
        local targetHRP = getValidTarget()
        if targetHRP then
            local predictedPos = targetHRP.Position + (targetHRP.CFrame.LookVector * PredictionAim.Prediction)
            local direction = (predictedPos - PredictionAim.HRP.Position).Unit
            local yRot = math.atan2(-direction.X, -direction.Z)
            PredictionAim.HRP.CFrame = CFrame.new(PredictionAim.HRP.Position) * CFrame.Angles(0, yRot, 0)
        end
    elseif PredictionAim.IsAiming then
        PredictionAim.IsAiming = false
        if PredictionAim.OriginalState then
            PredictionAim.Humanoid.WalkSpeed = PredictionAim.OriginalState.WalkSpeed
            PredictionAim.Humanoid.JumpPower = PredictionAim.OriginalState.JumpPower
            PredictionAim.Humanoid.AutoRotate = PredictionAim.OriginalState.AutoRotate
            PredictionAim.OriginalState = nil
        end
    end
end

if player.Character then setupCharacter(player.Character) end
player.CharacterAdded:Connect(setupCharacter)

local aimToggle = TabHandles.Survivors:Toggle({
    Title = "自动瞄准",
    Default = false,
    Callback = function(state)
        PredictionAim.Enabled = state
    end
})

local predictionSlider = TabHandles.Survivors:Slider({
    Title = "预测",
    Value = { Min = 1, Max = 10, Default = PredictionAim.Prediction },
    Callback = function(value)
        PredictionAim.Prediction = value
    end
})

RunService.RenderStepped:Connect(OnRenderStep)

-- Chance自动翻转硬币
local AutoFlipCoins = false
local flipCoinsThread = nil

local autoCoinToggle = TabHandles.Survivors:Toggle({
    Title = "自动翻转硬币(3)",
    Default = false,
    Callback = function(state)
        AutoFlipCoins = state
        
        if AutoFlipCoins then
            flipCoinsThread = task.spawn(function()
                while AutoFlipCoins and task.wait() do
                    local playerGui = game:GetService("Players").LocalPlayer.PlayerGui
                    local chargesText = playerGui:FindFirstChild("MainUI") and 
                                       playerGui.MainUI:FindFirstChild("AbilityContainer") and
                                       playerGui.MainUI.AbilityContainer:FindFirstChild("Shoot") and
                                       playerGui.MainUI.AbilityContainer.Shoot:FindFirstChild("Charges")
                    
                    if chargesText and chargesText:IsA("TextLabel") and chargesText.Text == "3" then
                        break
                    else
                        local args = {
                            [1] = "UseActorAbility",
                            [2] = "CoinFlip"
                        }
                        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
                    end
                end
            end)
        elseif flipCoinsThread then
            task.cancel(flipCoinsThread)
            flipCoinsThread = nil
        end
    end
})

local autoCoinToggle2 = TabHandles.Survivors:Toggle({
    Title = "自动翻转硬币(1)",
    Default = false,
    Callback = function(state)
        AutoFlipCoins = state
        
        if AutoFlipCoins then
            flipCoinsThread = task.spawn(function()
                while AutoFlipCoins and task.wait() do
                    local playerGui = game:GetService("Players").LocalPlayer.PlayerGui
                    local chargesText = playerGui:FindFirstChild("MainUI") and 
                                       playerGui.MainUI:FindFirstChild("AbilityContainer") and
                                       playerGui.MainUI.AbilityContainer:FindFirstChild("Shoot") and
                                       playerGui.MainUI.AbilityContainer.Shoot:FindFirstChild("Charges")
                    
                    if chargesText and chargesText:IsA("TextLabel") and chargesText.Text == "1" then
                        break
                    else
                        local args = {
                            [1] = "UseActorAbility",
                            [2] = "CoinFlip"
                        }
                        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
                    end
                end
            end)
        elseif flipCoinsThread then
            task.cancel(flipCoinsThread)
            flipCoinsThread = nil
        end
    end
})

-- Guest 1337幸存者
TabHandles.Survivors:Section({ Title = "Guest 1337幸存者" })

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local AutoBlockEnabled = false
local BlockDistance = 18
local combatConnection = nil
local lastBlockTime = 0
local BlockCooldown = 0.35

local TargetSoundIds = {
    "rbxassetid://102228729296384", "rbxassetid://140242176732868", "rbxassetid://12222216", 
    "rbxassetid://86174610237192", "rbxassetid://101199185291628", "rbxassetid://95079963655241", 
    "rbxassetid://112809109188560", "rbxassetid://84307400688050", "rbxassetid://136323728355613", 
    "rbxassetid://115026634746636", "rbxassetid://119942598489800", "rbxassetid://108907358619313", 
    "rbxassetid://119942598489800"
}

local function HasTargetSound(character)
    if not character then return false end
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return false end
    
    for _, sound in ipairs(rootPart:GetDescendants()) do
        if sound:IsA("Sound") and sound.IsPlaying then
            for _, id in ipairs(TargetSoundIds) do
                if sound.SoundId == id then
                    return true
                end
            end
        end
    end
    return false
end

local function GetThreateningKillers()
    local killers = {}
    local killersFolder = workspace:FindFirstChild("Killers") or (workspace:FindFirstChild("Players") and workspace.Players:FindFirstChild("Killers"))
    if not killersFolder then return killers end
    
    local myCharacter = LocalPlayer.Character
    if not myCharacter then return killers end
    
    local myRoot = myCharacter:FindFirstChild("HumanoidRootPart")
    if not myRoot then return killers end
    
    for _, killer in ipairs(killersFolder:GetChildren()) do
        if killer:FindFirstChild("HumanoidRootPart") then
            local killerRoot = killer.HumanoidRootPart
            local distance = (killerRoot.Position - myRoot.Position).Magnitude
            
            if distance <= BlockDistance and HasTargetSound(killer) then
                table.insert(killers, killer)
            end
        end
    end
    
    return killers
end

local function PerformBlockAndPunch()
    local now = os.clock()
    if now - lastBlockTime >= BlockCooldown then
        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer("UseActorAbility", "Block")
        
        local args = {
            [1] = "UseActorAbility",
            [2] = "Punch"
        }
        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
        
        lastBlockTime = now
    end
end

local function CombatLoop()
    local killers = GetThreateningKillers()
    if #killers > 0 then
        PerformBlockAndPunch()
    end
end

local autoBlockToggle = TabHandles.Survivors:Toggle({
    Title = "自动格挡 + 拳击",
    Default = false,
    Callback = function(state)
        AutoBlockEnabled = state
        
        if state then
            combatConnection = RunService.Stepped:Connect(function()
                pcall(CombatLoop)
            end)
        elseif combatConnection then
            combatConnection:Disconnect()
            combatConnection = nil
        end
    end
})

local distanceInput = TabHandles.Survivors:Input({
    Title = "检测范围",
    Value = tostring(BlockDistance),
    Callback = function(value)
        local num = tonumber(value)
        if num and num > 0 then
            BlockDistance = num
        end
    end
})

LocalPlayer.CharacterAdded:Connect(function()
    if AutoBlockEnabled and combatConnection then
        combatConnection:Disconnect()
        combatConnection = RunService.Stepped:Connect(CombatLoop)
    end
end)

-- TwoTime幸存者
TabHandles.Survivors:Section({ Title = "TwoTime幸存者" })

local twoTimeAimbotEnabled = false
local twoTimeMaxDistance = 50
local TWOloop = nil

local TWOsounds = {
    "rbxassetid://86710781315432", 
    "rbxassetid://99820161736138"
}

local function TWO(state)
    twoTimeAimbotEnabled = state

    if game:GetService("Players").LocalPlayer.Character.Name ~= "TwoTime" and state then
        return 
    end

    if state then
        TWOloop = game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.ChildAdded:Connect(function(child)
            if not twoTimeAimbotEnabled then return end
            for _, v in pairs(TWOsounds) do
                if child.Name == v then
                    local survivors = {}
                    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                        if player ~= game:GetService("Players").LocalPlayer then
                            local character = player.Character
                            if character and character:FindFirstChild("HumanoidRootPart") then
                                table.insert(survivors, character)
                            end
                        end
                    end

                    local nearestSurvivor = nil
                    local shortestDistance = math.huge  
                    
                    for _, survivor in pairs(survivors) do
                        local survivorHRP = survivor.HumanoidRootPart
                        local playerHRP = game:GetService("Players").LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        
                        if playerHRP then
                            local distance = (survivorHRP.Position - playerHRP.Position).Magnitude
                            if distance < shortestDistance and distance <= twoTimeMaxDistance then
                                shortestDistance = distance
                                nearestSurvivor = survivor
                            end
                        end
                    end
                    
                    if nearestSurvivor then
                        local nearestHRP = nearestSurvivor.HumanoidRootPart
                        local playerHRP = game:GetService("Players").LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        
                        if playerHRP then
                            local direction = (nearestHRP.Position - playerHRP.Position).Unit
                            local num = 1
                            local maxIterations = 100 
                            
                            if child.Name == "rbxassetid://79782181585087" then
                                maxIterations = 220  
                            end

                            while num <= maxIterations do
                                task.wait(0.01)
                                num = num + 1
                                workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, nearestHRP.Position)
                                playerHRP.CFrame = CFrame.lookAt(playerHRP.Position, Vector3.new(nearestHRP.Position.X, nearestHRP.Position.Y, nearestHRP.Position.Z))  
                            end
                        end
                    end
                end
            end
        end)
    else
        if TWOloop then
            TWOloop:Disconnect()
            TWOloop = nil
        end
    end
end

local twoTimeAimbotToggle = TabHandles.Survivors:Toggle({
    Title = "自动瞄准(冲锋)",
    Default = false,
    Callback = function(state)
        TWO(state)
    end
})

-- John Doe自动404
local Backstab = { 
    Enabled = false, 
    Mode = "背后", 
    Range = 4, 
    Cooldown = false, 
    LastTarget = nil, 
    KillerNames = { "Jason", "c00lkidd", "JohnDoe", "1x1x1x1", "Noli" }, 
    DaggerRemote = game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"), 
    KillersFolder = workspace:WaitForChild("Players"):WaitForChild("Killers") 
}

local function isBehindTarget(hrp, targetHRP) 
    if not (hrp and targetHRP and hrp.Parent and targetHRP.Parent) then return false end 
    local distance = (hrp.Position - targetHRP.Position).Magnitude 
    if distance > Backstab.Range then return false end

    if Backstab.Mode == "周围" then
        return true
    else
        local direction = -targetHRP.CFrame.LookVector
        local toPlayer = (hrp.Position - targetHRP.Position)
        return toPlayer:Dot(direction) > 0.5
    end
end

local function OnHeartbeat() 
    if not Backstab.Enabled or Backstab.Cooldown then return end 
    local char = LocalPlayer.Character 
    if not (char and char:FindFirstChild("HumanoidRootPart")) then return end 
    local hrp = char.HumanoidRootPart

    for _, name in ipairs(Backstab.KillerNames) do
        local killer = Backstab.KillersFolder:FindFirstChild(name)
        if killer and killer:FindFirstChild("HumanoidRootPart") then
            local kHRP = killer.HumanoidRootPart
            if isBehindTarget(hrp, kHRP) and killer ~= Backstab.LastTarget then
                Backstab.Cooldown = true
                Backstab.LastTarget = killer
                
                if Backstab.Mode == "传送" then
                    hrp.CFrame = CFrame.new(kHRP.Position - kHRP.CFrame.LookVector * 2, kHRP.Position)
                else
                    hrp.CFrame = CFrame.lookAt(hrp.Position, kHRP.Position)
                end
                
                Backstab.DaggerRemote:FireServer("UseActorAbility", "Dagger")
                task.delay(1, function()
                    Backstab.LastTarget = nil
                    Backstab.Cooldown = false
                end)
                break
            end
        end
    end
end

local backstabToggle = TabHandles.Survivors:Toggle({
    Title = "自动瞄准/传送背刺",
    Default = false,
    Callback = function(state)
        Backstab.Enabled = state
    end
})

local detectionModeDropdown = TabHandles.Survivors:Dropdown({
    Title = "瞄准模式",
    Values = { "背后", "周围" },
    Default = "背后",
    Callback = function(value)
        Backstab.Mode = value
    end
})

local rangeInput = TabHandles.Survivors:Input({
    Title = "背刺范围",
    Default = tostring(Backstab.Range),
    Placeholder = "1~10",
    Callback = function(value)
        local num = tonumber(value)
        if num and num >= 1 and num <= 10 then
            Backstab.Range = num
        end
    end
})

local actionModeDropdown = TabHandles.Survivors:Dropdown({
    Title = "动作模式",
    Values = { "瞄准", "传送" },
    Default = "瞄准",
    Callback = function(value)
        Backstab.ActionMode = value
    end
})

RunService.Heartbeat:Connect(OnHeartbeat)

-- Noli VoidRush
local NoliCharacter = player.Character or player.CharacterAdded:Wait()
local NoliHumanoid = NoliCharacter:WaitForChild("Humanoid")
local NoliHumanoidRootPart = NoliCharacter:WaitForChild("HumanoidRootPart")
local ORIGINAL_DASH_SPEED = 60
local isOverrideActive = false
local connection

local function startOverride()
    if isOverrideActive then return end
    isOverrideActive = true

    connection = RunService.RenderStepped:Connect(function()
        NoliHumanoid.WalkSpeed = ORIGINAL_DASH_SPEED
        NoliHumanoid.AutoRotate = false
        
        local direction = NoliHumanoidRootPart.CFrame.LookVector
        local horizontalDirection = Vector3.new(direction.X, 0, direction.Z).Unit
        NoliHumanoid:Move(horizontalDirection)
    end)
end

local function stopOverride()
    if not isOverrideActive then return end
    isOverrideActive = false

    NoliHumanoid.WalkSpeed = 16 
    NoliHumanoid.AutoRotate = true
    NoliHumanoid:Move(Vector3.new(0, 0, 0)) 
    
    if connection then
        connection:Disconnect()
        connection = nil
    end
end

RunService.RenderStepped:Connect(function()
    local voidRushState = NoliCharacter:GetAttribute("VoidRushState")
    if voidRushState == "Dashing" then
        startOverride()
    else
        stopOverride()
    end
end)

-- ========== 视觉功能部分 ==========
-- 电机ESP
if not _G.RealGeneratorESP then
    _G.RealGeneratorESP = {
     Active = false,
        Data = {},
        Connections = {}
    }
end

if not _G.FakeGeneratorESP then
    _G.FakeGeneratorESP = {
        Active = false,
        Data = {},
        Connections = {}
    }
end

if not _G.NoliWarningESP then
    _G.NoliWarningESP = {
        Active = false,
        Data = {},
        Connections = {}
    }
end

TabHandles.Visual:Section({ Title = "电机ESP" })

-- 绘制真电机
TabHandles.Visual:Toggle({
    Title = "绘制真电机",
    Default = false,
    Callback = function(enabled)
        if not enabled then
            if _G.RealGeneratorESP.Active then
                for _, connection in _G.RealGeneratorESP.Connections do
                    if connection and connection.Connected then
                        connection:Disconnect()
                    end
                end
                
                for gen, data in _G.RealGeneratorESP.Data do
                    if type(data) == "table" then
                        if data.Billboard and data.Billboard.Parent then
                            data.Billboard:Destroy()
                        end
                        if data.DistanceBillboard and data.DistanceBillboard.Parent then
                            data.DistanceBillboard:Destroy()
                        end
                        if data.Highlight and data.Highlight.Parent then
                            data.Highlight:Destroy()
                        end
                    end
                end
                
                _G.RealGeneratorESP.Data = {}
                _G.RealGeneratorESP.Connections = {}
                _G.RealGeneratorESP.Active = false
            end
            return
        end
        
        if _G.RealGeneratorESP.Active then
            return
        end
        
        _G.RealGeneratorESP.Active = true
        
        local scanInterval = 1.0
        local lastScanTime = 0
        local maxGenerators = 20
        
        local distanceSettings = {
            MinDistance = 5,
            MaxDistance = 500,
            MinScale = 0.8,
            MaxScale = 1.5,
            MinTextSize = 8,
            MaxTextSize = 10
        }
        
        local function updateGeneratorESP(gen, data)
            if not gen or not gen.Parent or not gen:FindFirstChild("Main") then
                return false
            end
            
            if #_G.RealGeneratorESP.Data > maxGenerators then
                return false
            end
            
            if gen:FindFirstChild("Progress") then
                local progress = gen.Progress.Value
                if progress >= 99 then
                    return false
                end
                
                if data.TextLabel then
                    data.TextLabel.Text = string.format("真电机: %d%%", progress)
                end
                
                local character = game:GetService("Players").LocalPlayer.Character
                local humanoidRootPart = character and character:FindFirstChild("HumanoidRootPart")
                
                if humanoidRootPart and data.DistanceLabel then
                    local distance = (gen.Main.Position - humanoidRootPart.Position).Magnitude
                    
                    data.DistanceLabel.Text = string.format("距离: %d米", math.floor(distance))
                    
                    local distanceRatio = math.clamp(
                        (distance - distanceSettings.MinDistance) / 
                        (distanceSettings.MaxDistance - distanceSettings.MinDistance),
                        0, 1
                    )
                    
                    local scale = distanceSettings.MinScale + 
                        distanceRatio * (distanceSettings.MaxScale - distanceSettings.MinScale)
                    
                    local textSize = distanceSettings.MinTextSize + 
                        distanceRatio * (distanceSettings.MaxTextSize - distanceSettings.MinTextSize)
                    
                    if data.Billboard then 
                        data.Billboard.Size = UDim2.new(4 * scale, 0, 1 * scale, 0)
                        data.Billboard.Enabled = true
                    end
                    
                    if data.DistanceBillboard then 
                        data.DistanceBillboard.Size = UDim2.new(4 * scale, 0, 1 * scale, 0)
                        data.DistanceBillboard.Enabled = true
                    end
                    
                    if data.TextLabel then 
                        data.TextLabel.TextSize = textSize
                        data.TextLabel.Visible = true
                    end
                    
                    if data.DistanceLabel then 
                        data.DistanceLabel.TextSize = textSize
                        data.DistanceLabel.Visible = true
                    end
                    
                    if data.Highlight then
                        data.Highlight.Enabled = true
                        local transparency = math.clamp((distance - 50) / 100, 0, 0.4)
                        data.Highlight.FillTransparency = 0.85 + (transparency * 0.5)
                        data.Highlight.OutlineColor = Color3.fromRGB(0, 255, 0)
                        data.Highlight.FillColor = Color3.fromRGB(0, 255, 0)
                    end
                end
            end
            
            return true
        end
        
        local function createGeneratorESP(gen)
            if not gen or not gen:FindFirstChild("Main") or _G.RealGeneratorESP.Data[gen] then 
                return 
            end
            
            if #_G.RealGeneratorESP.Data >= maxGenerators then
                return
            end
            
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "RealGeneratorESP"
            billboard.Size = UDim2.new(4, 0, 1, 0)
            billboard.StudsOffset = Vector3.new(0, 2.5, 0)
            billboard.Adornee = gen.Main
            billboard.Parent = gen.Main
            billboard.AlwaysOnTop = true
            billboard.Enabled = true
            
            local textLabel = Instance.new("TextLabel")
            textLabel.Size = UDim2.new(1, 0, 0.5, 0)
            textLabel.BackgroundTransparency = 1
            textLabel.TextScaled = false
            textLabel.Text = "真电机加载中..."
            textLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
            textLabel.Font = Enum.Font.Arcade
            textLabel.TextStrokeTransparency = 0
            textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            textLabel.TextSize = 8
            textLabel.Parent = billboard
            
            local distanceBillboard = Instance.new("BillboardGui")
            distanceBillboard.Name = "RealGeneratorDistanceESP"
            distanceBillboard.Size = UDim2.new(4, 0, 1, 0)
            distanceBillboard.StudsOffset = Vector3.new(0, 3.5, 0)
            distanceBillboard.Adornee = gen.Main
            distanceBillboard.Parent = gen.Main
            distanceBillboard.AlwaysOnTop = true
            distanceBillboard.Enabled = true
            
            local distanceLabel = Instance.new("TextLabel")
            distanceLabel.Size = UDim2.new(1, 0, 0.5, 0)
            distanceLabel.BackgroundTransparency = 1
            distanceLabel.TextScaled = false
            distanceLabel.Text = "计算距离中..."
            distanceLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
            distanceLabel.Font = Enum.Font.Arcade
            distanceLabel.TextStrokeTransparency = 0
            distanceLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            distanceLabel.TextSize = 8
            distanceLabel.Parent = distanceBillboard
            
            local highlight = Instance.new("Highlight")
            highlight.Name = "RealGeneratorHighlight"
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Enabled = true
            highlight.OutlineColor = Color3.fromRGB(0, 255, 0)
            highlight.FillColor = Color3.fromRGB(0, 255, 0)
            highlight.FillTransparency = 0.9
            highlight.OutlineTransparency = 0
            highlight.Parent = gen
            
            _G.RealGeneratorESP.Data[gen] = {
                Billboard = billboard,
                DistanceBillboard = distanceBillboard,
                TextLabel = textLabel,
                DistanceLabel = distanceLabel,
                Highlight = highlight
            }
            
            local destroyConnection
            destroyConnection = gen.Destroying:Connect(function()
                if _G.RealGeneratorESP.Data[gen] then
                    if _G.RealGeneratorESP.Data[gen].Billboard then 
                        _G.RealGeneratorESP.Data[gen].Billboard:Destroy() 
                    end
                    if _G.RealGeneratorESP.Data[gen].DistanceBillboard then 
                        _G.RealGeneratorESP.Data[gen].DistanceBillboard:Destroy() 
                    end
                    if _G.RealGeneratorESP.Data[gen].Highlight then 
                        _G.RealGeneratorESP.Data[gen].Highlight:Destroy() 
                    end
                    _G.RealGeneratorESP.Data[gen] = nil
                end
                if destroyConnection then
                    destroyConnection:Disconnect()
                end
            end)
            
            table.insert(_G.RealGeneratorESP.Connections, destroyConnection)
        end
        
        local function scanGenerators()
            local mapFolder = workspace:FindFirstChild("Map")
            if mapFolder then
                local ingameFolder = mapFolder:FindFirstChild("Ingame")
                if ingameFolder then
                    local mapSubFolder = ingameFolder:FindFirstChild("Map")
                    if mapSubFolder then
                        for gen in mapSubFolder:GetDescendants() do
                            if gen:IsA("Model") and gen:FindFirstChild("Main") and gen.Name == "Generator" then
                                createGeneratorESP(gen)
                            end
                        end
                    end
                end
            end
        end
        
        local mainConnection
        local mapFolder = workspace:FindFirstChild("Map")
        if mapFolder then
            local ingameFolder = mapFolder:FindFirstChild("Ingame")
            if ingameFolder then
                local mapSubFolder = ingameFolder:FindFirstChild("Map")
                if mapSubFolder then
                    mainConnection = mapSubFolder.DescendantAdded:Connect(function(v)
                        if v:IsA("Model") and v:FindFirstChild("Main") and v.Name == "Generator" then
                            createGeneratorESP(v)
                        end
                    end)
                end
            end
        end
        
        if mainConnection then
            table.insert(_G.RealGeneratorESP.Connections, mainConnection)
        end
        
        local heartbeatConnection = game:GetService("RunService").Heartbeat:Connect(function(deltaTime)
            lastScanTime = lastScanTime + deltaTime
            if lastScanTime >= scanInterval then
                lastScanTime = 0
                scanGenerators()
            end
            
            local gensToRemove = {}
            for gen, data in _G.RealGeneratorESP.Data do
                if not gen or not gen.Parent then
                    table.insert(gensToRemove, gen)
                else
                    if not updateGeneratorESP(gen, data) then
                        table.insert(gensToRemove, gen)
                    end
                end
            end
            
            for _, gen in gensToRemove do
                if _G.RealGeneratorESP.Data[gen] then
                    if _G.RealGeneratorESP.Data[gen].Billboard then 
                        _G.RealGeneratorESP.Data[gen].Billboard:Destroy() 
                    end
                    if _G.RealGeneratorESP.Data[gen].DistanceBillboard then 
                        _G.RealGeneratorESP.Data[gen].DistanceBillboard:Destroy() 
                    end
                    if _G.RealGeneratorESP.Data[gen].Highlight then 
                        _G.RealGeneratorESP.Data[gen].Highlight:Destroy() 
                    end
                    _G.RealGeneratorESP.Data[gen] = nil
                end
            end
        end)
        
        table.insert(_G.RealGeneratorESP.Connections, heartbeatConnection)
        scanGenerators()
    end
})

-- 绘制假电机
TabHandles.Visual:Toggle({
    Title = "绘制假电机",
    Default = false,
    Callback = function(enabled)
        if not enabled then
            if _G.FakeGeneratorESP.Active then
                for _, connection in _G.FakeGeneratorESP.Connections do
                    if connection and connection.Connected then
                        connection:Disconnect()
                    end
                end
                
                for gen, data in _G.FakeGeneratorESP.Data do
                    if type(data) == "table" then
                        if data.Highlight and data.Highlight.Parent then
                            data.Highlight:Destroy()
                        end
                        if data.NameLabel and data.NameLabel.Parent then
                            data.NameLabel:Destroy()
                        end
                    end
                end
                
                _G.FakeGeneratorESP.Data = {}
                _G.FakeGeneratorESP.Connections = {}
                _G.FakeGeneratorESP.Active = false
            end
            return
        end
        
        if _G.FakeGeneratorESP.Active then
            return
        end
        
        _G.FakeGeneratorESP.Active = true
        
        local scanInterval = 1.0
        local lastScanTime = 0
        
        local function createFakeGeneratorESP(gen)
            if not gen or not gen:FindFirstChild("Main") or _G.FakeGeneratorESP.Data[gen] then 
                return 
            end
            
            local highlight = Instance.new("Highlight")
            highlight.Name = "FakeGeneratorHighlight"
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Enabled = true
            highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
            highlight.FillColor = Color3.fromRGB(255, 0, 0)
            highlight.FillTransparency = 0.9
            highlight.OutlineTransparency = 0
            highlight.Parent = gen
            
            local nameBillboard = Instance.new("BillboardGui")
            nameBillboard.Name = "FakeGeneratorNameESP"
            nameBillboard.Size = UDim2.new(4, 0, 1, 0)
            nameBillboard.StudsOffset = Vector3.new(0, 2.5, 0)
            nameBillboard.Adornee = gen.Main
            nameBillboard.Parent = gen.Main
            nameBillboard.AlwaysOnTop = true
            nameBillboard.Enabled = true
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(1, 0, 1, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.TextScaled = false
            nameLabel.Text = "假电机"
            nameLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
            nameLabel.Font = Enum.Font.Arcade
            nameLabel.TextStrokeTransparency = 0
            nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            nameLabel.TextSize = 12
            nameLabel.Parent = nameBillboard
            
            _G.FakeGeneratorESP.Data[gen] = {
                Highlight = highlight,
                NameLabel = nameLabel,
                NameBillboard = nameBillboard
            }
            
            local destroyConnection
            destroyConnection = gen.Destroying:Connect(function()
                if _G.FakeGeneratorESP.Data[gen] then
                    if _G.FakeGeneratorESP.Data[gen].Highlight then 
                        _G.FakeGeneratorESP.Data[gen].Highlight:Destroy() 
                    end
                    if _G.FakeGeneratorESP.Data[gen].NameLabel then 
                        _G.FakeGeneratorESP.Data[gen].NameLabel:Destroy() 
                    end
                    if _G.FakeGeneratorESP.Data[gen].NameBillboard then 
                        _G.FakeGeneratorESP.Data[gen].NameBillboard:Destroy() 
                    end
                    _G.FakeGeneratorESP.Data[gen] = nil
                end
                if destroyConnection then
                    destroyConnection:Disconnect()
                end
            end)
            
            table.insert(_G.FakeGeneratorESP.Connections, destroyConnection)
        end
        
        local function scanGenerators()
            local mapFolder = workspace:FindFirstChild("Map")
            if mapFolder then
                local ingameFolder = mapFolder:FindFirstChild("Ingame")
                if ingameFolder then
                    local mapSubFolder = ingameFolder:FindFirstChild("Map")
                    if mapSubFolder then
                        for gen in mapSubFolder:GetDescendants() do
                            if gen:IsA("Model") and gen:FindFirstChild("Main") and gen.Name == "FakeGenerator" then
                                createFakeGeneratorESP(gen)
                            end
                        end
                    end
                end
            end
        end
        
        local mainConnection
        local mapFolder = workspace:FindFirstChild("Map")
        if mapFolder then
            local ingameFolder = mapFolder:FindFirstChild("Ingame")
            if ingameFolder then
                local mapSubFolder = ingameFolder:FindFirstChild("Map")
                if mapSubFolder then
                    mainConnection = mapSubFolder.DescendantAdded:Connect(function(v)
                        if v:IsA("Model") and v:FindFirstChild("Main") and v.Name == "FakeGenerator" then
                            createFakeGeneratorESP(v)
                        end
                    end)
                end
            end
        end
        
        if mainConnection then
            table.insert(_G.FakeGeneratorESP.Connections, mainConnection)
        end
        
        local heartbeatConnection = game:GetService("RunService").Heartbeat:Connect(function(deltaTime)
            lastScanTime = lastScanTime + deltaTime
            if lastScanTime >= scanInterval then
                lastScanTime = 0
                scanGenerators()
            end
            
            local gensToRemove = {}
            for gen, data in _G.FakeGeneratorESP.Data do
                if not gen or not gen.Parent then
                    table.insert(gensToRemove, gen)
                end
            end
            
            for _, gen in gensToRemove do
                if _G.FakeGeneratorESP.Data[gen] then
                    if _G.FakeGeneratorESP.Data[gen].Highlight then 
                        _G.FakeGeneratorESP.Data[gen].Highlight:Destroy() 
                    end
                    if _G.FakeGeneratorESP.Data[gen].NameLabel then 
                        _G.FakeGeneratorESP.Data[gen].NameLabel:Destroy() 
                    end
                    if _G.FakeGeneratorESP.Data[gen].NameBillboard then 
                        _G.FakeGeneratorESP.Data[gen].NameBillboard:Destroy() 
                    end
                    _G.FakeGeneratorESP.Data[gen] = nil
                end
            end
        end)
        
        table.insert(_G.FakeGeneratorESP.Connections, heartbeatConnection)
        scanGenerators()
    end
})

-- Noli传送电机警告TabHandles.Visual:Toggle({
    Title = "绘制Noli传送电机",
    Default = false,
    Callback = function(enabled)
        if not enabled then
            if _G.NoliWarningESP.Active then
                for _, connection in _G.NoliWarningESP.Connections do
                    if connection and connection.Connected then
                        connection:Disconnect()
                    end
                end
                
                for gen, data in _G.NoliWarningESP.Data do
                    if type(data) == "table" then
                        if data.Highlight and data.Highlight.Parent then
                            data.Highlight:Destroy()
                        end
                        if data.Label and data.Label.Parent then
                            data.Label:Destroy()
                        end
                    end
                end
                
                _G.NoliWarningESP.Data = {}
                _G.NoliWarningESP.Connections = {}
                _G.NoliWarningESP.Active = false
            end
            return
        end
        
        if _G.NoliWarningESP.Active then
            return
        end
        
        _G.NoliWarningESP.Active = true
        
        local scanInterval = 1.0
        local lastScanTime = 0
        
        local function hasNoliWarning(gen)
            if string.find(gen.Name, "NoliWarningIncoming") then
                return true
            end
            
            for child in gen:GetDescendants() do
                if (child:IsA("StringValue") or child:IsA("ObjectValue")) and 
                   string.find(tostring(child.Value), "NoliWarningIncoming") then
                    return true
                elseif child:IsA("BasePart") and string.find(child.Name, "NoliWarningIncoming") then
                    return true
                end
            end
            
            return false
        end
        
        local function createNoliWarningESP(gen)
            if not gen or not gen:FindFirstChild("Main") or _G.NoliWarningESP.Data[gen] then 
                return 
            end
            
            if not hasNoliWarning(gen) then
                return
            end
            
            local highlight = Instance.new("Highlight")
            highlight.Name = "NoliWarningHighlight"
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Enabled = true
            highlight.OutlineColor = Color3.fromRGB(255, 0, 255)
            highlight.FillColor = Color3.fromRGB(255, 0, 255)
            highlight.FillTransparency = 0.7
            highlight.OutlineTransparency = 0
            highlight.Parent = gen
            
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "NoliWarningBillboard"
            billboard.Size = UDim2.new(6, 0, 2, 0)
            billboard.StudsOffset = Vector3.new(0, 3, 0)
            billboard.Adornee = gen.Main
            billboard.Parent = gen.Main
            billboard.AlwaysOnTop = true
            
            local label = Instance.new("TextLabel")
            label.Size = UDim2.new(1, 0, 1, 0)
            label.BackgroundTransparency = 1
            label.Text = "[Noli即将传送]"
            label.TextColor3 = Color3.fromRGB(255, 0, 255)
            label.Font = Enum.Font.Arcade
            label.TextSize = 14
            label.TextStrokeTransparency = 0
            label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            label.Parent = billboard
            
            _G.NoliWarningESP.Data[gen] = {
                Highlight = highlight,
                Label = label,
                Billboard = billboard,
                LastCheck = os.time()
            }
            
            local destroyConnection
            destroyConnection = gen.Destroying:Connect(function()
                if _G.NoliWarningESP.Data[gen] then
                    if _G.NoliWarningESP.Data[gen].Highlight then 
                        _G.NoliWarningESP.Data[gen].Highlight:Destroy() 
                    end
                    if _G.NoliWarningESP.Data[gen].Label then 
                        _G.NoliWarningESP.Data[gen].Label:Destroy() 
                    end
                    if _G.NoliWarningESP.Data[gen].Billboard then 
                        _G.NoliWarningESP.Data[gen].Billboard:Destroy() 
                    end
                    _G.NoliWarningESP.Data[gen] = nil
                end
                if destroyConnection then
                    destroyConnection:Disconnect()
                end
            end)
            
            table.insert(_G.NoliWarningESP.Connections, destroyConnection)
        end
        
        local function scanGenerators()
            for gen in workspace:GetDescendants() do
                if gen:IsA("Model") and gen:FindFirstChild("Main") and 
                   (gen.Name == "Generator" or gen.Name == "FakeGenerator") then
                    createNoliWarningESP(gen)
                end
            end
        end
        
        local function updateExistingGenerators()
            local gensToRemove = {}
            for gen, data in _G.NoliWarningESP.Data do
                if not gen or not gen.Parent then
                    table.insert(gensToRemove, gen)
                else
                    if os.time() - data.LastCheck > 5 then
                        if not hasNoliWarning(gen) then
                            table.insert(gensToRemove, gen)
                        else
                            data.LastCheck = os.time()
                        end
                    end
                end
            end
            
            for _, gen in gensToRemove do
                if _G.NoliWarningESP.Data[gen] then
                    if _G.NoliWarningESP.Data[gen].Highlight then 
                        _G.NoliWarningESP.Data[gen].Highlight:Destroy() 
                    end
                    if _G.NoliWarningESP.Data[gen].Label then 
                        _G.NoliWarningESP.Data[gen].Label:Destroy() 
                    end
                    if _G.NoliWarningESP.Data[gen].Billboard then 
                        _G.NoliWarningESP.Data[gen].Billboard:Destroy() 
                    end
                    _G.NoliWarningESP.Data[gen] = nil
                end
            end
        end
        
        local mainConnection = workspace.DescendantAdded:Connect(function(v)
            if v:IsA("Model") and v:FindFirstChild("Main") and 
               (v.Name == "Generator" or v.Name == "FakeGenerator") then
                createNoliWarningESP(v)
            end
        end)
        
        table.insert(_G.NoliWarningESP.Connections, mainConnection)
        
        local heartbeatConnection = game:GetService("RunService").Heartbeat:Connect(function(deltaTime)
            lastScanTime = lastScanTime + deltaTime
            if lastScanTime >= scanInterval then
                lastScanTime = 0
                scanGenerators()
                updateExistingGenerators()
            end
        end)
        
        table.insert(_G.NoliWarningESP.Connections, heartbeatConnection)
        scanGenerators()
    end
})

-- John Doe陷阱绘制TabHandles.Visual:Section({ Title = "陷阱绘制" })

TabHandles.Visual:Button({
    Title = "绘制John Doe陷阱",
    Callback = function()
        local function findShadowInFolder(folder)
            for _, child in ipairs(folder:GetChildren()) do
                if child.Name == "Shadow" then
                    return child
                elseif child:IsA("Folder") or child:IsA("Model") then
                    local found = findShadowInFolder(child)
                    if found then return found end
                end
            end
            return nil
        end

        local shadow = findShadowInFolder(workspace.Map.Ingame)

        if shadow then
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
            
            local function getObjectSize(obj)
                if obj:IsA("BasePart") then
                    return obj.Size
                elseif obj:IsA("Model") and obj.PrimaryPart then
                    local cf = obj:GetBoundingBox()
                    return (cf[2] - cf[1]).Magnitude
                else
                    return Vector3.new(5, 5, 5) 
                end
            end
            
            local objectSize = getObjectSize(shadow)
            
            local highlight = Instance.new("Highlight")
            highlight.Name = "ShadowRangeIndicator"
            highlight.FillColor = Color3.fromRGB(255, 0, 0)
            highlight.FillTransparency = 0.8
            highlight.OutlineColor = Color3.fromRGB(255, 100, 100)
            highlight.OutlineTransparency = 0.5
            highlight.Parent = shadow
            
            local billboard = Instance.new("BillboardGui")
            billboard.Name = "ShadowNameDisplay"
            billboard.AlwaysOnTop = true
            billboard.Size = UDim2.new(0, 180, 0, 60) 
            billboard.StudsOffset = Vector3.new(0, objectSize.Y/2 + 2, 0) 
            billboard.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
            
            local textLabel = Instance.new("TextLabel")
            textLabel.Name = "TrapLabel"
            textLabel.Text = "TRAP"
            textLabel.Size = UDim2.new(1, 0, 0.5, 0)
            textLabel.Position = UDim2.new(0, 0, 0, 0) 
            textLabel.Font = Enum.Font.Arcade 
            textLabel.TextSize = 18  
            textLabel.TextColor3 = Color3.fromRGB(255, 0, 0) 
            textLabel.BackgroundTransparency = 1 
            textLabel.TextStrokeTransparency = 0 
            textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0) 
            textLabel.TextXAlignment = Enum.TextXAlignment.Center
            textLabel.TextYAlignment = Enum.TextYAlignment.Center
            
            local distanceLabel = Instance.new("TextLabel")
            distanceLabel.Name = "DistanceLabel"
            distanceLabel.Text = "Distance: Calculating..."
            distanceLabel.Size = UDim2.new(1, 0, 0.5, 0) 
            distanceLabel.Position = UDim2.new(0, 0, 0.5, 0) 
            distanceLabel.Font = Enum.Font.Arcade 
            distanceLabel.TextSize = 14  
            distanceLabel.TextColor3 = Color3.fromRGB(0, 255, 255) 
            distanceLabel.BackgroundTransparency = 1 
            distanceLabel.TextStrokeTransparency = 0 
            distanceLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0) 
            distanceLabel.TextXAlignment = Enum.TextXAlignment.Center
            distanceLabel.TextYAlignment = Enum.TextYAlignment.Center
            
            textLabel.Parent = billboard
            distanceLabel.Parent = billboard
            billboard.Parent = shadow
            
            if shadow:IsA("BasePart") then
                local boxHandleAdornment = Instance.new("BoxHandleAdornment")
                boxHandleAdornment.Name = "SizeIndicator"
                boxHandleAdornment.Adornee = shadow
                boxHandleAdornment.AlwaysOnTop = true
                boxHandleAdornment.Size = shadow.Size
                boxHandleAdornment.Transparency = 0.7
                boxHandleAdornment.Color3 = Color3.fromRGB(255, 50, 50)
                boxHandleAdornment.ZIndex = 10
                boxHandleAdornment.Parent = shadow
            end
            
            game:GetService("RunService").Heartbeat:Connect(function()
                if not shadow or not shadow.Parent then return end
                if not humanoidRootPart or not humanoidRootPart.Parent then return end
                
                local distance = (humanoidRootPart.Position - shadow.Position).Magnitude
                distanceLabel.Text = string.format("Distance: %.1f m", distance)  
                
                local baseScale = math.clamp(40 / math.max(1, distance), 0.4, 1.8) 
                
                textLabel.TextSize = 18 * baseScale
                distanceLabel.TextSize = 14 * baseScale
                
                local overallTransparency = math.clamp(distance / 80, 0.1, 0.4)
                
                local strokeTransparency = overallTransparency * 0.1 
                textLabel.TextStrokeTransparency = strokeTransparency
                distanceLabel.TextStrokeTransparency = strokeTransparency
                
                highlight.FillTransparency = math.clamp(distance/70, 0.3, 0.8)
            end)
        end
    end
})

-- Taph空间绊线绘制TabHandles.Visual:Button({
    Title = "绘制Taph空间绊线",
    Callback = function()
        local DEEP_PURPLE = Color3.fromRGB(102, 0, 153)

        local HIGHLIGHT_SETTINGS = {
            FillColor = DEEP_PURPLE,
            OutlineColor = DEEP_PURPLE,
            FillTransparency = 0.2,
            OutlineTransparency = 0,
            DepthMode = Enum.HighlightDepthMode.AlwaysOnTop,
            OutlineThickness = 2,
        }

        local function clearExistingHighlights()
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("Highlight") and obj.Name == "TaphTripwire_DeepPurpleHighlight" then
                    obj:Destroy()
                end
            end
        end

        local function getTargetFolder()
            local map = workspace:FindFirstChild("Map")
            if not map then return nil end
            local ingame = map:FindFirstChild("Ingame")
            if not ingame then return nil end
            return ingame
        end

        local function applyDeepPurpleHighlight(obj)
            local highlight = Instance.new("Highlight")
            highlight.Name = "TaphTripwire_DeepPurpleHighlight"
            highlight.Parent = obj
            for setting, value in pairs(HIGHLIGHT_SETTINGS) do
                highlight[setting] = value
            end
            if obj:IsA("BasePart") then
                local glow = Instance.new("SurfaceAppearance")
                glow.ColorMap = DEEP_PURPLE
                glow.Parent = obj
            end
        end

        local function highlightTaphTripwireObjects()
            clearExistingHighlights()
            local targetFolder = getTargetFolder()
            if not targetFolder then return end
            local count = 0
            for _, obj in pairs(targetFolder:GetDescendants()) do
                if obj.Name:find("TaphTripwire") then
                    applyDeepPurpleHighlight(obj)
                    count += 1
                end
            end
            print("高亮完成 数量: "..count)
            targetFolder.DescendantAdded:Connect(function(newObj)
                if newObj.Name:find("TaphTripwire") then
                    applyDeepPurpleHighlight(newObj)
                end
            end)
        end

        highlightTaphTripwireObjects()
        game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
            Text = "输入 /highlighttaph 重新执行高亮",
            Color = DEEP_PURPLE,
            Font = Enum.Font.SourceSansBold
        })
    end
})

-- Noli自身的星星绘制TabHandles.Visual:Button({
    Title = "绘制Noli星星",
    Callback = function()
        local Workspace = game:GetService("Workspace")
        local HIGHLIGHT_COLOR = Color3.fromRGB(255, 0, 255)
        local voidstar = Workspace:FindFirstChild("Voidstar", true)

        if voidstar then
            local highlight = Instance.new("Highlight")
            highlight.Name = "VoidstarHighlight"
            highlight.FillColor = HIGHLIGHT_COLOR
            highlight.OutlineColor = HIGHLIGHT_COLOR
            highlight.FillTransparency = 0.3
            highlight.OutlineTransparency = 0
            highlight.Parent = voidstar
        end
    end
})

-- ========== 玩家功能部分 ==========
TabHandles.Player:Section({ Title = "角色美化" })

-- Jason美化TabHandles.Player:Button({
    Title = "Jason角色美化",
    Callback = function()
        local player = game.Players.LocalPlayer  
        local character = player.Character or player.CharacterAdded:Wait()

        local bodyColors = character:FindFirstChild("Body Colors")
        if bodyColors then
            bodyColors.HeadColor = BrickColor.new("Really black")
            bodyColors.LeftArmColor = BrickColor.new("Really black")
            bodyColors.RightArmColor = BrickColor.new("Really black")
            bodyColors.LeftLegColor = BrickColor.new("Really black")
            bodyColors.RightLegColor = BrickColor.new("Really black")
            bodyColors.TorsoColor = BrickColor.new("Really black")
        end

        for _, obj in pairs(character:GetDescendants()) do
            if obj.Name == "Mask" then
                obj:Destroy()
            end
        end

        local shirt = character:FindFirstChildOfClass("Shirt") or Instance.new("Shirt", character)
        shirt.ShirtTemplate = "http://www.roblox.com/asset/?id=130919586308395"

        local pants = character:FindFirstChildOfClass("Pants") or Instance.new("Pants", character)
        pants.PantsTemplate = "http://www.roblox.com/asset/?id=83940216283729"

        local function addAccessory(name, meshId, textureId, parentPartName, handleSize, position, rotation, customWeldC0, customWeldC1, meshScale, meshOffset)
            local accessory = Instance.new("Accessory")
            accessory.Name = name
            local handle = Instance.new("Part")
            handle.Name = "Handle"
            handle.Size = Vector3.new(unpack(handleSize))
            handle.CanCollide = false
            handle.Anchored = false
            handle.Parent = accessory
            local mesh = Instance.new("SpecialMesh", handle)
            mesh.MeshId = meshId
            if textureId ~= "" then
                mesh.TextureId = textureId
            end
            if meshScale then
                mesh.Scale = Vector3.new(unpack(meshScale))
            else
                mesh.Scale = Vector3.new(1, 1, 1)
            end
            if meshOffset then
                mesh.Offset = Vector3.new(unpack(meshOffset))
            else
                mesh.Offset = Vector3.new(0, 0, 0)
            end
            local targetPart = character:FindFirstChild(parentPartName)
            if not targetPart then
                warn("Target part '" .. parentPartName .. "' not found in character!")
                return
            end
            local weld = Instance.new("Weld")
            weld.Part0 = targetPart
            weld.Part1 = handle
            if customWeldC0 and customWeldC1 then
                weld.C0 = CFrame.new(unpack(customWeldC0))
                weld.C1 = CFrame.new(unpack(customWeldC1))
            else
                weld.C0 = CFrame.new(unpack(position)) * CFrame.Angles(math.rad(rotation[1]), math.rad(rotation[2]), math.rad(rotation[3]))
            end
            weld.Parent = handle
            accessory.Parent = character
        end

        addAccessory(
            "Scarf", 
            "rbxassetid://131343945015522", 
            "rbxassetid://78916578261510", 
            "Scarf", 
            {1, 0.5, 1},
            {0, 0, 0},
            {0, 0, 0}
        )

        addAccessory(
            "NewMask", 
            "rbxassetid://119998744532277", 
            "rbxassetid://85762572283089", 
            "Head", 
            {1, 1, 1},
            {0, 0, 0},
            {0, 260, 0}
        )
    end
})

-- ========== 阻止功能部分 ==========TabHandles.Anti:Section({ Title = "1x弹窗阻止" })

local AutoPopup = {
    Enabled = false,
    Task = nil,
    Connections = {},
    Interval = 0.5
}

local function deletePopups()
    if not player or not player:FindFirstChild("PlayerGui") then
        return false
    end
    
    local tempUI = player.PlayerGui:FindFirstChild("TemporaryUI")
    if not tempUI then
        return false
    end
    
    local deleted = false
    for _, popup in ipairs(tempUI:GetChildren()) do
        if popup.Name == "1x1x1x1Popup" then
            popup:Destroy()
            deleted = true
        end
    end
    return deleted
end

local function triggerEntangled()
    local args = { [1] = "Entangled" }
    pcall(function()
        game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
    end)
end

local function setupPopupListener()
    if not player or not player:FindFirstChild("PlayerGui") then return end
    
    local tempUI = player.PlayerGui:FindFirstChild("TemporaryUI")
    if not tempUI then
        tempUI = Instance.new("Folder")
        tempUI.Name = "TemporaryUI"
        tempUI.Parent = player.PlayerGui
    end
    
    if AutoPopup.Connections.ChildAdded then
        AutoPopup.Connections.ChildAdded:Disconnect()
    end
    
    AutoPopup.Connections.ChildAdded = tempUI.ChildAdded:Connect(function(child)
        if AutoPopup.Enabled and child.Name == "1x1x1x1Popup" then
            task.defer(function()
                child:Destroy()
            end)
        end
    end)
end

local function runMainTask()
    while AutoPopup.Enabled do
        deletePopups()
        triggerEntangled()
        task.wait(AutoPopup.Interval)
    end
end

local function startAutoPopup()
    if AutoPopup.Enabled then return end
    
    AutoPopup.Enabled = true
    setupPopupListener()
    
    if AutoPopup.Task then
        task.cancel(AutoPopup.Task)
    end
    AutoPopup.Task = task.spawn(runMainTask)
end

local function stopAutoPopup()
    if not AutoPopup.Enabled then return end
    
    AutoPopup.Enabled = false
    
    if AutoPopup.Task then
        task.cancel(AutoPopup.Task)
        AutoPopup.Task = nil
    end
    
    for _, connection in pairs(AutoPopup.Connections) do
        connection:Disconnect()
    end
    AutoPopup.Connections = {}
end

local popupToggle = TabHandles.Anti:Toggle({
    Title = "阻止1x弹窗",
    Default = false,
    Callback = function(state)
        if state then
            startAutoPopup()
        else
            stopAutoPopup()
        end
    end
})

local intervalSlider = TabHandles.Anti:Slider({
    Title = "延迟",
    Step = 0.1,
    Value = { Min = 0.1, Max = 1, Default = AutoPopup.Interval },
    Callback = function(value)
        AutoPopup.Interval = value
    end
})

if player then
    player:GetPropertyChangedSignal("Parent"):Connect(function()
        if not player.Parent then
            stopAutoPopup()
        end
    end)
end

-- ========== John Doe自动404功能 ==========TabHandles.Survivors:Section({ Title = "John Doe自动404" })

local JohnDoePlayers = game:GetService("Players")
local JohnDoeRunService = game:GetService("RunService")
local JohnDoeReplicatedStorage = game:GetService("ReplicatedStorage")

local JohnDoelp = JohnDoePlayers.LocalPlayer
local JohnDoeRANGE = 19
local JohnDoeSPAM_DURATION = 3
local JohnDoeCOOLDOWN_TIME = 5
local JohnDoeactiveCooldowns = {}

local JohnDoeanimsToDetect = {
    ["116618003477002"] = true,
    ["119462383658044"] = true,
    ["131696603025265"] = true,
    ["121255898612475"] = true,
    ["133491532453922"] = true,
    ["103601716322988"] = true,
    ["86371356500204"] = true,
    ["72722244508749"] = false,
    ["87259391926321"] = true,
    ["96959123077498"] = false,
    ["86709774283672"] = true,
    ["77448521277146"] = true,
}

local function JohnDoefire404Error()
    local args = { "UseActorAbility", "404Error" }
    JohnDoeReplicatedStorage:WaitForChild("Modules")
        :WaitForChild("Network")
        :WaitForChild("RemoteEvent")
        :FireServer(unpack(args))
end

local function JohnDoeisAnimationMatching(anim)
    local id = tostring(anim.Animation and anim.Animation.AnimationId or "")
    local numId = id:match("%d+")
    return JohnDoeanimsToDetect[numId] or false
end

local JohnDoeAuto404Enabled = false
local JohnDoeConnection = nil

TabHandles.Survivors:Toggle({
    Title = "John Doe自动404",
    Default = false,
    Callback = function(state)
        JohnDoeAuto404Enabled = state
        
        if state then
            JohnDoeConnection = JohnDoeRunService.Heartbeat:Connect(function()
                for _, player in ipairs(JohnDoePlayers:GetPlayers()) do
                    if player ~= JohnDoelp and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local targetHRP = player.Character.HumanoidRootPart
                        local myChar = JohnDoelp.Character
                        if myChar and myChar:FindFirstChild("HumanoidRootPart") then
                            local dist = (targetHRP.Position - myChar.HumanoidRootPart.Position).Magnitude
                            if dist <= JohnDoeRANGE and (not JohnDoeactiveCooldowns[player] or tick() - JohnDoeactiveCooldowns[player] >= JohnDoeCOOLDOWN_TIME) then
                                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                                if humanoid then
                                    for _, track in pairs(humanoid:GetPlayingAnimationTracks()) do
                                        if JohnDoeisAnimationMatching(track) then
                                            JohnDoeactiveCooldowns[player] = tick()
                                            task.spawn(function()
                                                local startTime = tick()
                                                while tick() - startTime < JohnDoeSPAM_DURATION do
                                                    JohnDoefire404Error()
                                                    task.wait(0.05)
                                                end
                                            end)
                                            break
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end)
        elseif JohnDoeConnection then
            JohnDoeConnection:Disconnect()
            JohnDoeConnection = nil
        end
    end
})

-- ========== Noli VoidRush无视碰撞 ==========TabHandles.Survivors:Section({ Title = "Noli高级功能" })

TabHandles.Survivors:Button({
    Title = "Noli VoidRush无视碰撞",
    Callback = function()
        game:GetService("Players").LocalPlayer.Character:SetAttribute("VoidRushState", "Crashed")
    end
})

-- ========== 脚本初始化完成 ==========
WindUI:Notify({
    Title = "HuK高级版",
    Content = "所有功能加载完成！",
    Duration = 3
})
