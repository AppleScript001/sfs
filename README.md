local Material = loadstring(game:HttpGet("https://raw.githubusercontent.com/Kinlei/MaterialLua/master/Module.lua"))()
local X = Material.Load({
    Title = "🌌 Strong Ninja Simulator 🌌",
    Style = 2,
    SizeX = 320,
    SizeY = 320,
    Theme = "Dark",
    ColorOverrides = {
        MainFrame = Color3.fromRGB(0, 9, 55)
    }
})

local ESPEnabled = false -- Track if ESP is enabled
local ESPDrawing = {} -- Table to store ESP drawings

local Y = X.New({
    Title = "Main"
})

local function EnableESP()
    ESPEnabled = true
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game:GetService("Players").LocalPlayer then
            ConnectESP(player)
        end
    end
    game:GetService("Players").PlayerAdded:Connect(function(player)
        if player ~= game:GetService("Players").LocalPlayer then
            ConnectESP(player)
        end
    end)
end

local function DisableESP()
    ESPEnabled = false
    for _, drawing in ipairs(ESPDrawing) do
        drawing.Visible = false
        drawing:Remove()
    end
    ESPDrawing = {}
end

local function ConnectESP(player)
    local characterAddedConn
    characterAddedConn = player.CharacterAdded:Connect(function(character)
        esp(player, character)
    end)
    table.insert(ESPDrawing, characterAddedConn)
end

local function esp(player, character)
    local humanoid = character:WaitForChild("Humanoid")
    local head = character:WaitForChild("Head")

    local text = Drawing.new("Text")
    text.Visible = false
    text.Center = true
    text.Outline = false
    text.Font = Enum.Font.SourceSans
    text.Size = 16
    text.Color = Color3.fromRGB(170, 170, 170)

    local function disconnect()
        text.Visible = false
        text:Remove()
    end

    local ancestryChangedConn = character.AncestryChanged:Connect(function(_, parent)
        if not parent then
            disconnect()
        end
    end)

    local healthChangedConn
    healthChangedConn = humanoid.HealthChanged:Connect(function(health)
        if health <= 0 or humanoid:GetState() == Enum.HumanoidStateType.Dead then
            disconnect()
            ancestryChangedConn:Disconnect()
            table.remove(ESPDrawing, table.find(ESPDrawing, healthChangedConn))
        end
    end)

    local renderSteppedConn
    renderSteppedConn = game:GetService("RunService").RenderStepped:Connect(function()
        local headPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(head.Position)
        if onScreen then
            text.Position = Vector2.new(headPos.X, headPos.Y - 27)
            text.Text = "[ " .. player.Name .. " ]"
            text.Visible = true
        else
            text.Visible = false
        end
    end)

    table.insert(ESPDrawing, text)
end

Y.Toggle({
    Text = "ESP [Player]",
    Callback = function(Value)
        if Value then
            EnableESP()
        else
            DisableESP()
        end
    end,
    Enabled = false -- Initial state is disabled
})
