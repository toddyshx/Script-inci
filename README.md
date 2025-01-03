local Library = loadstring(game:HttpGetAsync("https://github.com/ActualMasterOogway/Fluent-Renewed/releases/latest/download/Fluent.luau"))()
local Window = Library:CreateWindow{
    Title = "Toddy Script made by Anna",
    SubTitle = "https://discord.gg/KQukDAvC3f",
    TabWidth = 160,
    Size = UDim2.fromOffset(830, 525),
    Resize = true,
    MinSize = Vector2.new(470, 380),
    Acrylic = true,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.RightControl
}

local Tabs = {
    Main = Window:CreateTab{
        Title = "Main",
        Icon = "circle-user-round"
    },
    Shop = Window:CreateTab{
        Title = "Shop",
        Icon = "bag-shopping"
    },
    Misc = Window:CreateTab{
        Title = "Misc",
        Icon = "gear"
    }
}

-- Toggle 1 - Auto Cast
local Toggle1 = Tabs.Main:CreateToggle("AutoCast", {Title = "Auto Cast", Default = false})
Toggle1:OnChanged(function()
    local Players = game:GetService("Players")
    local player = Players.LocalPlayer
    local isActive = Toggle1.Value

    local function castFishingLine()
        local character = player.Character
        if character then
            for _, tool in ipairs(character:GetChildren()) do
                if tool:IsA("Tool") and tool:FindFirstChild("values") then
                    local values = tool.values
                    local castedValue = values:FindFirstChild("casted")
                    local bobberzoneValue = values:FindFirstChild("bobberzone")
                    local shaky = player.PlayerGui:FindFirstChild("shakeui")

                    if (castedValue and not castedValue.Value) or 
                       (bobberzoneValue and (bobberzoneValue.Value == "" or bobberzoneValue.Value == nil)) or (shaky == nil) then
                        wait(0.4689999)
                        tool.events.reset:FireServer(math.random(35, 100), true)
                        wait(0.1)
                        tool.events.cast:FireServer(math.random(35, 100), true)
                        local bobber = tool:FindFirstChild("bobber")
                        if bobber then
                            bobber:WaitForChild("RopeConstraint").Length = 999999999999999
                            bobber:WaitForChild("gyro").Enabled = false
                            bobber.Anchored = false
                            bobber.CanCollide = false
                            wait(0.869420)
                            bobber.Anchored = true
                        end
                        wait(1.269420)
                    end
                end
            end
        end
    end

    local function checkCasting()
        while isActive do
            castFishingLine()
            task.wait()
        end
    end

    if isActive then
        task.spawn(checkCasting)
    end
end)

-- Toggle 2 - Auto Shake
local Toggle2 = Tabs.Main:CreateToggle("AutoShake", {Title = "Auto Shake", Default = false})
Toggle2:OnChanged(function()
    local Players = game:GetService("Players")
    local UserInputService = game:GetService("UserInputService")
    local GuiService = game:GetService("GuiService")
    local VirtualInputManager = game:GetService("VirtualInputManager")

    local player = Players.LocalPlayer
    local playerGui = player:WaitForChild("PlayerGui")

    local buttonClicked = Toggle2.Value
    local firingCoroutine = nil

    local function checkButton()
        local shakeui = playerGui:FindFirstChild("shakeui")
        if shakeui then
            local safezone = shakeui:FindFirstChild("safezone")
            if safezone then
                return safezone:FindFirstChild("button")
            end
        end
        return nil
    end

    local function ensureUINavigation()
        if not UserInputService.ModalEnabled then
            UserInputService.ModalEnabled = true
        end
    end

    local function simulateButtonPress()
        local button = checkButton()
        GuiService.SelectedObject = button
        VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Return, false, game)
        task.wait()
        VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Return, false, game)
    end

    if buttonClicked then
        ensureUINavigation()
        firingCoroutine = task.spawn(function()
            while buttonClicked do
                local button = checkButton()
                if button and button.Parent then
                    GuiService.SelectedObject = button
                    simulateButtonPress()
                end
                task.wait()
            end
        end)
    else
        if firingCoroutine then
            task.cancel(firingCoroutine)
            firingCoroutine = nil
        end
    end
end)

-- Toggle 3 - Auto Reel
local Toggle3 = Tabs.Main:CreateToggle("Toggle3", {Title = "Auto Reel", Default = false})
Toggle3:OnChanged(function()
    local isActive = Toggle3.Value
    local coroutineReel

    local function autoReel()
        while isActive do
            local args = {
                [1] = 100,
                [2] = false
            }
            game:GetService("ReplicatedStorage").events.reelfinished:FireServer(unpack(args))
            task.wait(0.1)
        end
    end

    if isActive then
        coroutineReel = task.spawn(autoReel)
    else
        if coroutineReel then
            task.cancel(coroutineReel)
            coroutineReel = nil
        end
    end
end)

-- Toggle - Sell ALL no Tab "Shop"
local SellAllToggle = Tabs.Shop:CreateToggle("SellAll", {Title = "Sell ALL", Default = false})
SellAllToggle:OnChanged(function()
    local isActive = SellAllToggle.Value
    local coroutineSellAll

    local function sellAllItems()
        while isActive do
            workspace.world.npcs:FindFirstChild("Marc Merchant").merchant.sellall:InvokeServer()
            task.wait(0.1)
        end
    end

    if isActive then
        coroutineSellAll = task.spawn(sellAllItems)
    else
        if coroutineSellAll then
            task.cancel(coroutineSellAll)
            coroutineSellAll = nil
        end
    end
end)

-- Slider - Walkspeed no Tab "Misc"
local WalkspeedSlider = Tabs.Misc:CreateSlider("Walkspeed", {Title = "Walkspeed", Default = 16, Min = 16, Max = 100, Decimals = 0})
WalkspeedSlider:OnChanged(function(value)
    local player = game:GetService("Players").LocalPlayer
    if player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid.WalkSpeed = value
    end
end)

-- Toggle - Infinite Jump no Tab "Misc"
local InfiniteJumpToggle = Tabs.Misc:CreateToggle("InfiniteJump", {Title = "Infinite Jump", Default = false})
InfiniteJumpToggle:OnChanged(function()
    local UserInputService = game:GetService("UserInputService")
    local isEnabled = InfiniteJumpToggle.Value

    local jumpConnection
    if isEnabled then
        jumpConnection = UserInputService.JumpRequest:Connect(function()
            local player = game:GetService("Players").LocalPlayer
            if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
                player.Character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        if jumpConnection then
            jumpConnection:Disconnect()
            jumpConnection = nil
        end
    end
end)
