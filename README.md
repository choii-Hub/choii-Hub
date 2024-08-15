local lp = game.Players.LocalPlayer
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Remotes = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("Quest")
local GetQuestInfo = Remotes:WaitForChild("GetQuestInfo")
local AcceptQuest = Remotes:WaitForChild("AcceptQuest")

-- Variables to store the target and quest information
local targetName = "High End"
local questNPC = "Mirko"
local questName = "Quest"
local questAccepted = false
local questCheckInterval = 25 -- Time in seconds to wait between quest checks
local moveSpeed = 550 -- Increase this value to move faster
local swingInterval = 5 -- Time in seconds to wait between swings

local function getNPC()
    local dist, thing = math.huge
    for _, v in pairs(game:GetService("Workspace").NPCs:GetChildren()) do
        if v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
            if string.find(string.lower(v.Name), string.lower(targetName)) then
                local mag = (lp.Character.HumanoidRootPart.Position - v.HumanoidRootPart.Position).magnitude
                if mag < dist then
                    dist = mag
                    thing = v
                end
            end
        end
    end
    return thing
end

local noclipE = nil
local antifall = nil

local function noclip()
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        for _, v in pairs(lp.Character:GetDescendants()) do
            if v:IsA("BasePart") and v.CanCollide then
                v.CanCollide = false
            end
        end
        lp.Character.HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
    end
end

local function moveto(obj, speed)
    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        local info = TweenInfo.new(((lp.Character.HumanoidRootPart.Position - obj.Position).Magnitude) / speed, Enum.EasingStyle.Linear)
        local tween = TweenService:Create(lp.Character.HumanoidRootPart, info, { CFrame = obj })

        if not lp.Character.HumanoidRootPart:FindFirstChild("BodyVelocity") then
            antifall = Instance.new("BodyVelocity", lp.Character.HumanoidRootPart)
            antifall.Velocity = Vector3.new(0, 0, 0)
            noclipE = RunService.Stepped:Connect(noclip)
            tween:Play()
        end

        tween.Completed:Connect(function()
            if antifall then
                antifall:Destroy()
            end
            if noclipE then
                noclipE:Disconnect()
            end
        end)
    end
end

-- Function to automatically accept the quest if available
local function autoAcceptQuest()
    local questInfoArgs = { [1] = questNPC }

    -- Fetch quest information
    local success, questInfo = pcall(function()
        return GetQuestInfo:InvokeServer(unpack(questInfoArgs))
    end)

    if success and questInfo then
        if not questAccepted then
            print("Quest Info: ", questInfo)

            local acceptArgs = { [1] = questNPC, [2] = questName }

            -- Accept the quest
            pcall(function()
                AcceptQuest:FireServer(unpack(acceptArgs))
                questAccepted = true -- Mark quest as accepted
            end)

            print("Quest Accepted")
        else
            print("Quest already accepted.")
        end
    else
        print("Failed to get quest info or quest not available")
    end
end

-- Function to automatically swing
local function autoSwing()
    local swingArgs = {
        [1] = Vector3.new(450.3987121582031, 325.8056640625, 3989.541259765625)
    }
    pcall(function()
        game:GetService("Players").LocalPlayer.Character.Main.Swing:FireServer(unpack(swingArgs))
    end)
end

-- Load Kavo UI Library
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Choii Hub | Buko no roblox", "BloodTheme")
local Tab = Window:NewTab("Main")
local Section = Tab:NewSection("Farming")

-- Input for target name
Section:NewTextBox("Target Name", "Enter target NPC name", function(value)
    targetName = value
end)

-- Button to toggle the script
Section:NewButton("Autofarm", "Enable or Disable Autofarm", function()
    getgenv().scriptEnabled = not getgenv().scriptEnabled
    if getgenv().scriptEnabled then
        print("Autofarm Enabled")
    else
        print("Autofarm Disabled")
    end
end)

-- Main loop
RunService.Heartbeat:Connect(function()
    if getgenv().scriptEnabled then
        pcall(function()
            local target = getNPC()
            if target and target.HumanoidRootPart then
                moveto(target.HumanoidRootPart.CFrame, moveSpeed)
                VirtualUser:ClickButton1(Vector2.new(9e9, 9e9))
                autoSwing() -- Call autoSwing function
            end
        end)
    end
end)

-- Check for quest availability every `questCheckInterval` seconds
spawn(function()
    while true do
        if getgenv().scriptEnabled then
            pcall(function()
                autoAcceptQuest()
            end)
        end
        wait(questCheckInterval)
    end
end)

-- Automatically swing at intervals
spawn(function()
    while true do
        if getgenv().scriptEnabled then
            pcall(function()
                autoSwing()
            end)
        end
        wait(swingInterval)
    end
end)
