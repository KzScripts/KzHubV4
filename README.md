local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "KzHub " .. Fluent.Version,
    SubTitle = "by kz",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "layout-dashboard" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

-- Infinite Jump
Tabs.Main:AddButton({
    Title = "Infinite Jump",
    Callback = function()
        game:GetService("UserInputService").JumpRequest:Connect(function()
            if game.Players.LocalPlayer.Character then
                game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
            end
        end)
        Fluent:Notify({ Title = "KzHub", Content = "Infinite Jump ativado!", Duration = 5 })
    end
})

-- Fly
Tabs.Main:AddButton({
    Title = "Fly",
    Callback = function()
        loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Gui-Fly-v3-37111"))()
        Fluent:Notify({ Title = "KzHub", Content = "Fly ativado!", Duration = 5 })
    end
})

-- Speed Slider
local Slider = Tabs.Main:AddSlider("Slider", {
    Title = "Speed",
    Description = "Aumentar a velocidade",
    Default = 16,
    Min = 1,
    Max = 450,
    Rounding = 1,
    Callback = function(Value)
        local hum = game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = Value end
    end
})
Slider:SetValue(16)

-- ESP com Highlight e Cor
local highlightESP
local colorMap = {
    ["Vermelho"] = Color3.fromRGB(255, 0, 0),
    ["Azul"] = Color3.fromRGB(0, 0, 255),
    ["Amarelo"] = Color3.fromRGB(255, 255, 0),
    ["Roxo"] = Color3.fromRGB(128, 0, 128)
}
local selectedColor = Color3.fromRGB(255, 0, 0)

local espToggle = Tabs.Main:AddToggle("ESP", { Title = "Ativar ESP", Default = false })
espToggle:OnChanged(function(state)
    if state then
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer and player.Character then
                local highlight = Instance.new("Highlight")
                highlight.FillColor = selectedColor
                highlight.OutlineColor = Color3.new(1, 1, 1)
                highlight.OutlineTransparency = 0
                highlight.FillTransparency = 0.5
                highlight.Adornee = player.Character
                highlight.Parent = player.Character
            end
        end
        Fluent:Notify({ Title = "KzHub", Content = "ESP ativado!", Duration = 5 })
    else
        for _, player in pairs(game.Players:GetPlayers()) do
            if player.Character then
                for _, v in pairs(player.Character:GetChildren()) do
                    if v:IsA("Highlight") then
                        v:Destroy()
                    end
                end
            end
        end
        Fluent:Notify({ Title = "KzHub", Content = "ESP desativado!", Duration = 5 })
    end
end)

local colorDropdown = Tabs.Main:AddDropdown("ESPColor", {
    Title = "Cor do ESP",
    Values = { "Vermelho", "Azul", "Amarelo", "Roxo" },
    Default = "Vermelho"
})
colorDropdown:OnChanged(function(value)
    selectedColor = colorMap[value]
    -- Atualiza a cor dos highlights ativos
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character then
            for _, v in pairs(player.Character:GetChildren()) do
                if v:IsA("Highlight") then
                    v.FillColor = selectedColor
                end
            end
        end
    end
end)


--FpsBoost
Tabs.Settings:AddButton({
    Title = "FPS Boost",
    Description = "Melhora o desempenho no jogo",
    Callback = function()
        -- Reduz qualidade gráfica
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01

        -- Remove efeitos e luzes desnecessárias
        for _, obj in ipairs(game:GetDescendants()) do
            if obj:IsA("BlurEffect") or obj:IsA("SunRaysEffect") or obj:IsA("BloomEffect") or obj:IsA("DepthOfFieldEffect") then
                obj.Enabled = false
            elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") then
                obj.Enabled = false
            elseif obj:IsA("Explosion") then
                obj.Visible = false
            elseif obj:IsA("Decal") then
                obj.Transparency = 1
            elseif obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
                obj.Enabled = false
            end
        end

        -- Diminui a renderização de água
        if workspace:FindFirstChildOfClass("Terrain") then
            workspace:FindFirstChildOfClass("Terrain").WaterWaveSize = 0
            workspace:FindFirstChildOfClass("Terrain").WaterWaveSpeed = 0
            workspace:FindFirstChildOfClass("Terrain").WaterReflectance = 0
            workspace:FindFirstChildOfClass("Terrain").WaterTransparency = 1
        end

        Fluent:Notify({
            Title = "FPS Boost",
            Content = "Desempenho melhorado com sucesso!",
            Duration = 5
        })
    end
})

local antiAfkConnection

local antiAfkToggle = Tabs.Settings:AddToggle("AntiAFK", {
    Title = "Ativar Anti-AFK",
    Default = false
})

antiAfkToggle:OnChanged(function(state)
    if state then
        antiAfkConnection = game:GetService("Players").LocalPlayer.Idled:Connect(function()
            -- Simula uma interação com o jogo
            VirtualUser = game:GetService("VirtualUser")
            VirtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            task.wait(1)
            VirtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        end)

        Fluent:Notify({
            Title = "Anti-AFK Ativado",
            Content = "Você não será mais desconectado por inatividade.",
            Duration = 4
        })
    else
        if antiAfkConnection then
            antiAfkConnection:Disconnect()
            antiAfkConnection = nil
        end

        Fluent:Notify({
            Title = "Anti-AFK Desativado",
            Content = "Modo de inatividade normal restaurado.",
            Duration = 4
        })
    end
end)


local dashSlider = Tabs.Main:AddSlider("DashLength", {
    Title = "Dash Length",
    Description = "Ajusta o comprimento do Dash",
    Min = 10,
    Max = 1000,
    Default = 10,
    Suffix = " Length",
    Rounding = 0,
    Callback = function(Value)
        local char = game.Players.LocalPlayer.Character
        if char then
            char:SetAttribute("DashLength", Value)
        end
    end
})

dashSlider:OnChanged(function(Value)
    local char = game.Players.LocalPlayer.Character
    if char then
        char:SetAttribute("DashLength", Value)
    end
end)

local jumpSlider = Tabs.Main:AddSlider("JumpHeight", {
    Title = "Jump Height",
    Description = "Ajusta a altura do pulo",
    Min = 10,
    Max = 500,
    Default = 10,
    Suffix = " Height",
    Rounding = 0,
    Callback = function(Value)
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.JumpPower = Value
        end
    end
})

jumpSlider:OnChanged(function(Value)
    local char = game.Players.LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.JumpPower = Value
    end
end)



-- SaveManager e InterfaceManager
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)

SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})

InterfaceManager:SetFolder("KzHub")
SaveManager:SetFolder("KzHub/Saves")

InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

SaveManager:LoadAutoloadConfig()

Fluent:Notify({
    Title = "KzHub",
    Content = "Script carregado com sucesso!",
    Duration = 5
})
