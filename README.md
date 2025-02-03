-- تعريف الخدمات
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- اللاعب المحلي
local player = Players.LocalPlayer

-- متغيرات لتتبع الحالات
local aimbotEnabled = false
local espEnabled = false
local aimTarget = "Head" -- الهدف الافتراضي للتصويب

-- دالة لإنشاء واجهة المستخدم (GUI)
local function createGUI()
    -- إنشاء ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = player.PlayerGui

    -- زر "A" لفتح القائمة
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0, 50, 0, 50)
    toggleButton.Position = UDim2.new(0, 10, 0, 10)
    toggleButton.Text = "A"
    toggleButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
    toggleButton.TextColor3 = Color3.new(1, 1, 1)
    toggleButton.Parent = screenGui

    -- القائمة
    local menuFrame = Instance.new("Frame")
    menuFrame.Size = UDim2.new(0, 150, 0, 200)
    menuFrame.Position = UDim2.new(0, 70, 0, 10)
    menuFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    menuFrame.Visible = false
    menuFrame.Parent = screenGui

    -- زر Aimbot داخل القائمة
    local aimbotButton = Instance.new("TextButton")
    aimbotButton.Size = UDim2.new(0, 140, 0, 40)
    aimbotButton.Position = UDim2.new(0, 5, 0, 5)
    aimbotButton.Text = "Aimbot: OFF"
    aimbotButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
    aimbotButton.TextColor3 = Color3.new(1, 1, 1)
    aimbotButton.Parent = menuFrame

    -- زر ESP داخل القائمة
    local espButton = Instance.new("TextButton")
    espButton.Size = UDim2.new(0, 140, 0, 40)
    espButton.Position = UDim2.new(0, 5, 0, 50)
    espButton.Text = "ESP: OFF"
    espButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
    espButton.TextColor3 = Color3.new(1, 1, 1)
    espButton.Parent = menuFrame

    -- زر اختيار الهدف للتصويب
    local aimTargetButton = Instance.new("TextButton")
    aimTargetButton.Size = UDim2.new(0, 140, 0, 40)
    aimTargetButton.Position = UDim2.new(0, 5, 0, 95)
    aimTargetButton.Text = "Aim Target"
    aimTargetButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
    aimTargetButton.TextColor3 = Color3.new(1, 1, 1)
    aimTargetButton.Parent = menuFrame

    -- القائمة الصغيرة لاختيار الهدف
    local targetMenuFrame = Instance.new("Frame")
    targetMenuFrame.Size = UDim2.new(0, 100, 0, 100)
    targetMenuFrame.Position = UDim2.new(0, 160, 0, 95)
    targetMenuFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    targetMenuFrame.Visible = false
    targetMenuFrame.Parent = menuFrame

    -- زر التصويب نحو الرأس
    local headButton = Instance.new("TextButton")
    headButton.Size = UDim2.new(1, 0, 0.5, 0)
    headButton.Position = UDim2.new(0, 0, 0, 0)
    headButton.Text = "Head"
    headButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
    headButton.TextColor3 = Color3.new(1, 1, 1)
    headButton.Parent = targetMenuFrame

    -- زر التصويب نحو الجسم
    local bodyButton = Instance.new("TextButton")
    bodyButton.Size = UDim2.new(1, 0, 0.5, 0)
    bodyButton.Position = UDim2.new(0, 0, 0.5, 0)
    bodyButton.Text = "Body"
    bodyButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.8)
    bodyButton.TextColor3 = Color3.new(1, 1, 1)
    bodyButton.Parent = targetMenuFrame

    -- تفعيل/إيقاف القائمة عند النقر على زر "A"
    toggleButton.MouseButton1Click:Connect(function()
        menuFrame.Visible = not menuFrame.Visible
    end)

    -- تفعيل/إيقاف الـ Aimbot عند النقر على زر Aimbot
    aimbotButton.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        aimbotButton.Text = aimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
        aimbotButton.BackgroundColor3 = aimbotEnabled and Color3.new(0, 1, 0) or Color3.new(0.2, 0.2, 0.8)
        print("Aimbot:", aimbotEnabled and "ON" or "OFF")
    end)

    -- تفعيل/إيقاف الـ ESP عند النقر على زر ESP
    espButton.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        espButton.Text = espEnabled and "ESP: ON" or "ESP: OFF"
        espButton.BackgroundColor3 = espEnabled and Color3.new(0, 1, 0) or Color3.new(0.2, 0.2, 0.8)
        print("ESP:", espEnabled and "ON" or "OFF")
        updateESP()
    end)

    -- فتح/إغلاق القائمة الصغيرة عند النقر على زر Aim Target
    aimTargetButton.MouseButton1Click:Connect(function()
        targetMenuFrame.Visible = not targetMenuFrame.Visible
    end)

    -- اختيار التصويب نحو الرأس
    headButton.MouseButton1Click:Connect(function()
        aimTarget = "Head"
        targetMenuFrame.Visible = false
        print("Aim Target set to: Head")
    end)

    -- اختيار التصويب نحو الجسم
    bodyButton.MouseButton1Click:Connect(function()
        aimTarget = "Body"
        targetMenuFrame.Visible = false
        print("Aim Target set to: Body")
    end)
end

-- دالة لإيجاد أقرب عدو
local function findClosestEnemy()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        -- تجاهل اللاعب المحلي
        if otherPlayer ~= player and otherPlayer.Character then
            local character = otherPlayer.Character
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")

            if humanoidRootPart then
                -- حساب المسافة بين اللاعب المحلي والعدو
                local distance = (humanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude

                -- التحقق من أقرب عدو
                if distance < shortestDistance then
                    shortestDistance = distance
                    closestPlayer = otherPlayer
                end
            end
        end
    end

    return closestPlayer
end

-- دالة لتوجيه الكاميرا نحو العدو
local function aimAtEnemy()
    if aimbotEnabled then
        local closestEnemy = findClosestEnemy()

        if closestEnemy and closestEnemy.Character then
            local enemyRoot = closestEnemy.Character:FindFirstChild("HumanoidRootPart")

            if enemyRoot then
                -- توجيه الكاميرا نحو العدو حسب الهدف المختار
                local camera = workspace.CurrentCamera
                if aimTarget == "Head" then
                    local head = closestEnemy.Character:FindFirstChild("Head")
                    if head then
                        camera.CFrame = CFrame.new(camera.CFrame.Position, head.Position)
                    end
                else
                    camera.CFrame = CFrame.new(camera.CFrame.Position, enemyRoot.Position)
                end
            end
        end
    end
end

-- دالة لعرض ضوء أزرق فوق رؤوس اللاعبين (ESP)
local function updateESP()
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local character = otherPlayer.Character
            local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")

            if humanoidRootPart then
                -- إنشاء أو تحديث BillboardGui
                local espBillboard = humanoidRootPart:FindFirstChild("ESPBillboard")
                if not espBillboard then
                    espBillboard = Instance.new("BillboardGui")
                    espBillboard.Name = "ESPBillboard"
                    espBillboard.Size = UDim2.new(0, 20, 0, 20)
                    espBillboard.AlwaysOnTop = true
                    espBillboard.Adornee = humanoidRootPart
                    espBillboard.LightInfluence = 0
                    espBillboard.Parent = humanoidRootPart

                    -- إنشاء دائرة زرقاء
                    local circle = Instance.new("Frame")
                    circle.Size = UDim2.new(1, 0, 1, 0)
                    circle.BackgroundColor3 = Color3.new(0, 0, 1)
                    circle.BackgroundTransparency = 0.5
                    circle.BorderSizePixel = 0
                    circle.Parent = espBillboard

                    -- جعل الدائرة دائرية
                    local corner = Instance.new("UICorner")
                    corner.CornerRadius = UDim.new(1, 0)
                    corner.Parent = circle
                end

                espBillboard.Enabled = espEnabled
            end
        end
    end
end

-- إنشاء واجهة المستخدم عند ولادة اللاعب
player.CharacterAdded:Connect(function()
    createGUI()
end)

-- إنشاء واجهة المستخدم لأول مرة
createGUI()

-- تحديث الـ Aimbot في كل إطار
RunService.RenderStepped:Connect(aimAtEnemy)

-- تحديث الـ ESP في كل إطار
RunService.RenderStepped:Connect(updateESP)
