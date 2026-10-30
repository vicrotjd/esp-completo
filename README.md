--[[
    Universal ESP Panel for Roblox
    Compatible with Vortex and other major executors.
    Features: Box ESP, Tracers, Skeleton, and Toggle Panel.
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações de Estado
local ESP_Settings = {
    Enabled = true,
    Boxes = true,
    Tracers = true,
    Skeletons = true,
    TeamCheck = false,
    BoxColor = Color3.fromRGB(255, 0, 0),
    TracerColor = Color3.fromRGB(255, 255, 255)
}

-- Armazenamento de objetos de ESP
local ESP_Objects = {}

-- Função para criar a interface (Painel)
local function CreatePanel()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ESP_Panel"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = game:GetService("CoreGui")

    local MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 200, 0, 250)
    MainFrame.Position = UDim2.new(0.1, 0, 0.1, 0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Text = "UNIVERSAL ESP PANEL"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    Title.Parent = MainFrame

    local function CreateToggle(name, settingKey, pos)
        local Button = Instance.new("TextButton")
        Button.Size = UDim2.new(0.9, 0, 0, 35)
        Button.Position = UDim2.new(0.05, 0, 0, pos)
        Button.Text = name .. ": " .. (ESP_Settings[settingKey] and "ON" or "OFF")
        Button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        Button.TextColor3 = Color3.fromRGB(255, 255, 255)
        Button.Parent = MainFrame

        Button.MouseButton1Click:Connect(function()
            ESP_Settings[settingKey] = not ESP_Settings[settingKey]
            Button.Text = name .. ": " .. (ESP_Settings[settingKey] and "ON" or "OFF")
        end)
    end

    CreateToggle("Master Switch", "Enabled", 40)
    CreateToggle("Box ESP", "Boxes", 80)
    CreateToggle("Tracers", "Tracers", 120)
    CreateToggle("Skeleton", "Skeletons", 160)
    CreateToggle("Team Check", "TeamCheck", 200)
end

-- Função para criar o ESP de um jogador
local function CreateESP(player)
    if player == LocalPlayer then return end

    local objects = {
        Box = Drawing.new("Square"),
        Tracer = Drawing.new("Line"),
        Skeletons = {}
    }

    -- Configuração inicial do Box
    objects.Box.Visible = false
    objects.Box.Color = ESP_Settings.BoxColor
    objects.Box.Thickness = 1
    objects.Box.Filled = false

    -- Configuração inicial do Tracer
    objects.Tracer.Visible = false
    objects.Tracer.Color = ESP_Settings.TracerColor
    objects.Tracer.Thickness = 1

    -- Configuração do Skeleton (Linhas entre juntas)
    local joints = {
        {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
        {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"},
        {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"},
        {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"},
        {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}
    }

    for i = 1, #joints do
        local line = Drawing.new("Line")
        line.Visible = false
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Thickness = 1
        table.insert(objects.Skeletons, line)
    end

    ESP_Objects[player] = objects
end

-- Loop de atualização (RenderStep)
RunService.RenderStepped:Connect(function()
    for player, obj in pairs(ESP_Objects) do
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
            obj.Box.Visible = false
            obj.Tracer.Visible = false
            for _, s in pairs(obj.Skeletons) do s.Visible = false end
            continue
        end

        if not ESP_Settings.Enabled then
            obj.Box.Visible = false
            obj.Tracer.Visible = false
            for _, s in pairs(obj.Skeletons) do s.Visible = false end
            continue
        end

        if ESP_Settings.TeamCheck and player.Team == LocalPlayer.Team then
            obj.Box.Visible = false
            obj.Tracer.Visible = false
            for _, s in pairs(obj.Skeletons) do s.Visible = false end
            continue
        end

        local hrp = player.Character.HumanoidRootPart
        local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

        if onScreen then
            -- Atualiza Box
            if ESP_Settings.Boxes then
                obj.Box.Size = Vector2.new(2000 / pos.Z, 3000 / pos.Z)
                obj.Box.Position = Vector2.new(pos.X - obj.Box.Size.X / 2, pos.Y - obj.Box.Size.Y / 2)
                obj.Box.Visible = true
            else
                obj.Box.Visible = false
            end

            -- Atualiza Tracer
            if ESP_Settings.Tracers then
                obj.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                obj.Tracer.To = Vector2.new(pos.X, pos.Y)
                obj.Tracer.Visible = true
            else
                obj.Tracer.Visible = false
            end

            -- Atualiza Skeleton
            if ESP_Settings.Skeletons then
                local char = player.Character
                local joints = {
                    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
                    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"},
                    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"},
                    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"},
                    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}
                }

                for i, joint in pairs(joints) do
                    local p1 = char:FindFirstChild(joint[1])
                    local p2 = char:FindFirstChild(joint[2])
                    if p1 and p2 then
                        local v1, s1 = Camera:WorldToViewportPoint(p1.Position)
                        local v2, s2 = Camera:WorldToViewportPoint(p2.Position)
                        if s1 and s2 then
                            obj.Skeletons[i].From = Vector2.new(v1.X, v1.Y)
                            obj.Skeletons[i].To = Vector2.new(v2.X, v2.Y)
                            obj.Skeletons[i].Visible = true
                        else
                            obj.Skeletons[i].Visible = false
                        end
                    else
                        obj.Skeletons[i].Visible = false
                    end
                end
            else
                for _, s in pairs(obj.Skeletons) do s.Visible = false end
            end
        else
            obj.Box.Visible = false
            obj.Tracer.Visible = false
            for _, s in pairs(obj.Skeletons) do s.Visible = false end
        end
    end
end)

-- Gerenciamento de Jogadores
Players.PlayerAdded:Connect(CreateESP)
for _, player in pairs(Players:GetPlayers()) do
    CreateESP(player)
end

Players.PlayerRemoving:Connect(function(player)
    if ESP_Objects[player] then
        ESP_Objects[player].Box:Remove()
        ESP_Objects[player].Tracer:Remove()
        for _, s in pairs(ESP_Objects[player].Skeletons) do s:Remove() end
        ESP_Objects[player] = nil
    end
end)

CreatePanel()
