local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LP = Players.LocalPlayer
local COREGUI = game.CoreGui

-- Очистка старых ESP
for _, v in pairs(COREGUI:GetChildren()) do
    if v.Name == "Boat_ESP" then
        v:Destroy()
    end
end

-- Создаём папку для ESP
local BoatESPfolder = Instance.new("Folder")
BoatESPfolder.Name = "Boat_ESP"
BoatESPfolder.Parent = COREGUI

-- Находим все лодки
local function findAllBoats()
    local boatWorkspace = Workspace:FindFirstChild("Game Systems")
    if not boatWorkspace then return {} end
    
    boatWorkspace = boatWorkspace:FindFirstChild("Boat Workspace")
    if not boatWorkspace then return {} end
    
    local foundBoats = {}
    
    for _, boatModel in ipairs(boatWorkspace:GetChildren()) do
        if boatModel:IsA("Model") then
            local primaryPart = boatModel.PrimaryPart or 
                               boatModel:FindFirstChild("HumanoidRootPart") or
                               boatModel:FindFirstChildWhichIsA("BasePart")
            
            if primaryPart then
                table.insert(foundBoats, {
                    model = boatModel,
                    name = boatModel.Name,
                    primaryPart = primaryPart
                })
            end
        end
    end
    
    return foundBoats
end

-- Создаём ESP для одной лодки
local function createBoatESP(boatData)
    -- Highlight (подсветка)
    local highlight = Instance.new("Highlight")
    highlight.Name = "Boat_Highlight"
    highlight.Adornee = boatData.model
    highlight.FillColor = Color3.fromRGB(0, 150, 255)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.35
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = boatData.model
    
    -- BillboardGui (текст над лодкой)
    local billboard = Instance.new("BillboardGui")
    billboard.Name = boatData.name .. "_ESP"
    billboard.Adornee = boatData.primaryPart
    billboard.Size = UDim2.new(0, 300, 0, 120)
    billboard.StudsOffset = Vector3.new(0, 10, 0)
    billboard.AlwaysOnTop = true
    billboard.MaxDistance = 10000
    billboard.Parent = BoatESPfolder
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = boatData.name .. "\nЗагрузка..."
    textLabel.TextColor3 = Color3.fromRGB(0, 255, 255)
    textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.TextStrokeTransparency = 0.3
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 18
    textLabel.TextWrapped = true
    textLabel.TextYAlignment = Enum.TextYAlignment.Top
    textLabel.Parent = billboard
    
    return {
        highlight = highlight,
        billboard = billboard,
        textLabel = textLabel,
        model = boatData.model,
        primaryPart = boatData.primaryPart,
        name = boatData.name
    }
end

-- Функция поиска здоровья лодки
local function findBoatHealth(boatModel)
    local healthValue = 0
    local maxHealthValue = 0
    
    -- Ищем в разных местах
    local function checkHealth(obj)
        if obj:IsA("Humanoid") then
            return obj.Health, obj.MaxHealth
        elseif obj:IsA("NumberValue") and obj.Name == "Health" then
            return obj.Value, 100
        elseif obj:IsA("IntValue") and obj.Name == "Health" then
            return obj.Value, 100
        end
        return nil, nil
    end
    
    -- Проверяем прямые дочерние объекты
    for _, child in ipairs(boatModel:GetChildren()) do
        local hp, maxHP = checkHealth(child)
        if hp then
            healthValue = hp
            maxHealthValue = maxHP or 100
            break
        end
        
        -- Проверяем вложенные объекты (например, в Stats)
        if child:IsA("Folder") or child:IsA("Model") then
            for _, subChild in ipairs(child:GetChildren()) do
                local hp2, maxHP2 = checkHealth(subChild)
                if hp2 then
                    healthValue = hp2
                    maxHealthValue = maxHP2 or 100
                    break
                end
            end
        end
    end
    
    -- Если здоровье не найдено, ищем MaxHealth отдельно
    if healthValue > 0 and maxHealthValue == 0 then
        local maxHealth = boatModel:FindFirstChild("MaxHealth")
        if maxHealth and (maxHealth:IsA("NumberValue") or maxHealth:IsA("IntValue")) then
            maxHealthValue = maxHealth.Value
        else
            maxHealthValue = 100 -- Значение по умолчанию
        end
    end
    
    return healthValue, maxHealthValue
end

-- Обновляем ESP
local function updateBoatESP(espData)
    if not espData.model or not espData.model.Parent then
        if espData.highlight then espData.highlight:Destroy() end
        if espData.billboard then espData.billboard:Destroy() end
        return false
    end
    
    -- Ищем здоровье
    local healthValue, maxHealthValue = findBoatHealth(espData.model)
    
    -- Дистанция
    local distance = 0
    if LP.Character then
        local charRoot = LP.Character:FindFirstChild("HumanoidRootPart") or LP.Character.PrimaryPart
        if charRoot and espData.primaryPart then
            distance = (charRoot.Position - espData.primaryPart.Position).Magnitude
        end
    end
    
    -- Форматируем текст
    local displayText = ""
    
    if healthValue > 0 and maxHealthValue > 0 then
        local healthPercent = math.floor((healthValue / maxHealthValue) * 100)
        displayText = string.format("🚢 %s\n❤ HP: %d/%d (%d%%)\n📏 Дистанция: %d studs", 
            espData.name,
            math.floor(healthValue), 
            math.floor(maxHealthValue),
            healthPercent,
            math.floor(distance)
        )
        
        -- Цвет по здоровью
        if healthPercent < 30 then
            espData.textLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
            espData.highlight.FillColor = Color3.fromRGB(255, 50, 50)
        elseif healthPercent < 60 then
            espData.textLabel.TextColor3 = Color3.fromRGB(255, 255, 50)
            espData.highlight.FillColor = Color3.fromRGB(255, 255, 50)
        else
            espData.textLabel.TextColor3 = Color3.fromRGB(50, 255, 100)
            espData.highlight.FillColor = Color3.fromRGB(0, 150, 255)
        end
    else
        -- Если здоровье не найдено
        displayText = string.format("🚢 %s\n📏 Дистанция: %d studs\nℹ️ HP: Не найдено", 
            espData.name,
            math.floor(distance)
        )
        espData.textLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
        espData.highlight.FillColor = Color3.fromRGB(100, 100, 100)
    end
    
    espData.textLabel.Text = displayText
    
    return true
end

-- Основной цикл
local trackedBoats = {}
local initialized = false

local function mainESP()
    -- Ищем лодки
    local foundBoats = findAllBoats()
    
    -- Добавляем новые
    for _, boatData in ipairs(foundBoats) do
        if not trackedBoats[boatData.model] then
            local espData = createBoatESP(boatData)
            if espData then
                trackedBoats[boatData.model] = espData
                if not initialized then
                    print("[Boat ESP] Активирован. Отслеживается: " .. boatData.name)
                end
            end
        end
    end
    
    initialized = true
    
    -- Обновляем и удаляем старые
    for model, espData in pairs(trackedBoats) do
        if not updateBoatESP(espData) then
            trackedBoats[model] = nil
        end
    end
end

-- Запуск
local connection
local function startESP()
    if connection then
        connection:Disconnect()
    end
    
    connection = RunService.Heartbeat:Connect(function()
        pcall(mainESP)
    end)
end

local function stopESP()
    if connection then
        connection:Disconnect()
    end
    
    -- Очищаем всё
    for model, espData in pairs(trackedBoats) do
        if espData.highlight then 
            pcall(function() espData.highlight:Destroy() end) 
        end
        if espData.billboard then 
            pcall(function() espData.billboard:Destroy() end) 
        end
    end
    
    trackedBoats = {}
    
    if BoatESPfolder then
        BoatESPfolder:Destroy()
    end
    
    print("[Boat ESP] Выключен")
end

-- Автостарт
wait(1)
startESP()
print("[Boat ESP] Запущен (Insert - перезапуск, Delete - выключить)")

-- Управление
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Insert then
        stopESP()
        wait(0.1)
        startESP()
        print("[Boat ESP] Перезапущен")
    elseif input.KeyCode == Enum.KeyCode.Delete then
        stopESP()
    end
end)
