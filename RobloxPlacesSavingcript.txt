local Players = game:GetService("Players")
local plr = Players.LocalPlayer

local savedPos = nil
local clickCount = 0
local maxClicks = 10
local resetTime = 3
local lastClick = 0

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = plr:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 120, 0, 60)
Frame.Position = UDim2.new(0.4,0,0.4,0)
Frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0,5)
UICorner.Parent = Frame

local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 1
UIStroke.Color = Color3.fromRGB(80,80,80)
UIStroke.Parent = Frame

-- بار ↑
local DragBar = Instance.new("TextLabel")
DragBar.Size = UDim2.new(1,0,0.3,0)
DragBar.BackgroundColor3 = Color3.fromRGB(45,45,45)
DragBar.Position = UDim2.new(0,0,0,0)
DragBar.Text = "↑"
DragBar.TextColor3 = Color3.fromRGB(255,255,255)
DragBar.TextScaled = true
DragBar.Active = true
DragBar.Parent = Frame

-- عداد الضغطات يمين السهم
local PressCounter = Instance.new("TextLabel")
PressCounter.Size = UDim2.new(0.5,0,0.3,0)
PressCounter.Position = UDim2.new(0.5,0,0,0)
PressCounter.BackgroundTransparency = 1
PressCounter.TextColor3 = Color3.fromRGB(255,200,0)
PressCounter.TextScaled = true
PressCounter.Text = tostring(maxClicks)
PressCounter.TextXAlignment = Enum.TextXAlignment.Right
PressCounter.Parent = DragBar

-- زر القفل شمال السهم
local LockBtn = Instance.new("TextButton")
LockBtn.Size = UDim2.new(0.3,0,0.3,0)
LockBtn.Position = UDim2.new(0,0,0,0)
LockBtn.Text = "🔒"
LockBtn.TextScaled = true
LockBtn.BackgroundColor3 = Color3.fromRGB(70,70,70)
LockBtn.TextColor3 = Color3.fromRGB(255,255,255)
LockBtn.Parent = DragBar

local locked = false
LockBtn.MouseButton1Click:Connect(function()
    locked = not locked
    if locked then
        LockBtn.Text = "🔓" -- مفتوح معناها لا يتحرك
    else
        LockBtn.Text = "🔒"
    end
end)

-- أزرار Set و TP
local SetBtn = Instance.new("TextButton")
SetBtn.Size = UDim2.new(0.9,0,0.3,0)
SetBtn.Position = UDim2.new(0.05,0,0.35,0)
SetBtn.Text = "Set"
SetBtn.TextScaled = true
SetBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
SetBtn.TextColor3 = Color3.fromRGB(255,255,255)
SetBtn.Parent = Frame

local TpBtn = Instance.new("TextButton")
TpBtn.Size = UDim2.new(0.9,0,0.3,0)
TpBtn.Position = UDim2.new(0.05,0,0.7,0)
TpBtn.Text = "TP"
TpBtn.TextScaled = true
TpBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
TpBtn.TextColor3 = Color3.fromRGB(255,255,255)
TpBtn.Parent = Frame

-- حماية الأزرار من الضغط السريع
local SetBusy = false
local TpBusy = false

local normalTransparency = 0
local fadedTransparency = 0.8
local lastInteraction = tick()

-- دالة لإعادة الشفافية عند لمس السهم
local function onDragInteraction()
    lastInteraction = tick()
end

-- وظائف الأزرار
SetBtn.MouseButton1Click:Connect(function()
    if SetBusy then return end
    if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
        SetBusy = true
        savedPos = plr.Character.HumanoidRootPart.CFrame
        local originalColor = SetBtn.BackgroundColor3
        SetBtn.BackgroundColor3 = Color3.fromRGB(0,255,0)
        SetBtn.Text = "Saved!"
        task.wait(0.5)
        SetBtn.BackgroundColor3 = originalColor
        SetBtn.Text = "Set"
        SetBusy = false
    end
end)

TpBtn.MouseButton1Click:Connect(function()
    if TpBusy then return end
    if savedPos and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
        TpBusy = true
        plr.Character.HumanoidRootPart.CFrame = savedPos
        local originalColor = TpBtn.BackgroundColor3
        TpBtn.BackgroundColor3 = Color3.fromRGB(0,255,255)
        task.wait(0.125)
        TpBtn.BackgroundColor3 = originalColor
        TpBusy = false
    else
        TpBtn.Text = "No Point!"
        task.wait(1)
        TpBtn.Text = "TP"
    end
end)

-- تحديث العداد تلقائيًا بعد مرور resetTime
task.spawn(function()
    while true do
        task.wait(0.1)
        if tick() - lastClick > resetTime then
            clickCount = 0
            PressCounter.Text = tostring(maxClicks)
        end
    end
end)

-- السحب باللمس مع الشفافية
local dragging = false
local dragStartPos, frameStartPos

local function updateTransparency()
    while true do
        task.wait(0.1)
        local t = (tick() - lastInteraction > 3) and fadedTransparency or normalTransparency
        Frame.BackgroundTransparency = t
        DragBar.BackgroundTransparency = t
        SetBtn.BackgroundTransparency = t
        TpBtn.BackgroundTransparency = t
        DragBar.TextTransparency = t
        PressCounter.TextTransparency = t
        SetBtn.TextTransparency = t
        TpBtn.TextTransparency = t
        LockBtn.BackgroundTransparency = t
        LockBtn.TextTransparency = t
    end
end

task.spawn(updateTransparency)

DragBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        if locked then return end -- إذا القفل مفعل، لا يتحرك
        dragging = true
        dragStartPos = input.Position
        frameStartPos = Frame.Position
        onDragInteraction()

        local currentTime = tick()
        if currentTime - lastClick > resetTime then
            clickCount = 0
        end
        clickCount = clickCount + 1
        lastClick = currentTime

        local remaining = maxClicks - clickCount
        if remaining < 0 then remaining = 0 end
        PressCounter.Text = tostring(remaining)

        if clickCount >= maxClicks then
            Frame.Visible = false
        end

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
                onDragInteraction()
            end
        end)
    end
end)

DragBar.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStartPos
        Frame.Position = UDim2.new(
            frameStartPos.X.Scale, frameStartPos.X.Offset + delta.X,
            frameStartPos.Y.Scale, frameStartPos.Y.Offset + delta.Y
        )
        onDragInteraction()
    end
end)
