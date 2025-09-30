local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local TweenService = game:GetService("TweenService")

local states = {
autoAim = false,
esp = false,
wallhack = false,
noclip = false,
}

local gui = Instance.new("ScreenGui")
gui.Name = "PerfectCheatUI"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 300, 0, 500)
main.Position = UDim2.new(0, 20, 0.2, 0)
main.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
main.BorderSizePixel = 0
main.BackgroundTransparency = 0.1
main.Active = true
main.Draggable = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

local border = Instance.new("UIStroke", main)
border.Color = Color3.fromRGB(180, 0, 255)
border.Thickness = 2
border.Transparency = 0.3

local title = Instance.new("TextButton", main)
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.Text = "▼項目を表示▼"
title.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.BorderSizePixel = 0
Instance.new("UICorner", title).CornerRadius = UDim.new(0, 12)

local content = Instance.new("Frame", main)
content.Position = UDim2.new(0, 0, 0, 40)
content.Size = UDim2.new(1, 0, 1, -40)
content.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", content)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 8)

local open = true
title.MouseButton1Click:Connect(function()
open = not open
content.Visible = open
main.Size = open and UDim2.new(0, 300, 0, 500) or UDim2.new(0, 300, 0, 40)
end)

local function createToggleButton(name, key, callback)
local button = Instance.new("TextButton")
button.Size = UDim2.new(1, -20, 0, 36)
button.Position = UDim2.new(0, 10, 0, 0)
button.Text = name .. ": OFF"
button.Font = Enum.Font.GothamMedium
button.TextSize = 16
button.TextColor3 = Color3.new(1, 1, 1)
button.BackgroundColor3 = Color3.fromRGB(30, 0, 50)
button.BorderSizePixel = 0
button.Parent = content
Instance.new("UICorner", button).CornerRadius = UDim.new(0, 10)

button.MouseButton1Click:Connect(function()  
    states[key] = not states[key]  
    button.Text = name .. ": " .. (states[key] and "ON" or "OFF")  
    if callback then callback(states[key]) end  
end)  


end

createToggleButton("AutoAim", "autoAim")
createToggleButton("ESP", "esp")
createToggleButton("WallHack", "wallhack", function(enabled)
local transparencyValue = enabled and 0.5 or 0
for _, obj in pairs(workspace:GetDescendants()) do
if obj:IsA("BasePart") then
obj.LocalTransparencyModifier = transparencyValue
end
end
end)
createToggleButton("NoClip", "noclip")

local function createLabeledInput(labelText, defaultText, callback)
local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, -20, 0, 20)
label.Position = UDim2.new(0, 10, 0, 0)
label.Text = labelText
label.Font = Enum.Font.GothamBold
label.TextSize = 14
label.TextColor3 = Color3.fromRGB(255, 255, 255)
label.BackgroundTransparency = 1
label.TextXAlignment = Enum.TextXAlignment.Left
label.Parent = content

local box = Instance.new("TextBox")  
box.Size = UDim2.new(1, -20, 0, 30)  
box.Position = UDim2.new(0, 10, 0, 0)  
box.Text = defaultText  
box.Font = Enum.Font.GothamMedium  
box.TextSize = 16  
box.TextColor3 = Color3.new(1, 1, 1)  
box.BackgroundColor3 = Color3.fromRGB(50, 0, 70)  
box.BorderSizePixel = 0  
box.Parent = content  
Instance.new("UICorner", box).CornerRadius = UDim.new(0, 8)  

box.FocusLost:Connect(function()  
    local value = tonumber(box.Text)  
    if value then  
        callback(value)  
    end  
end)  


end

createLabeledInput("移動速度倍率（0.0001〜1億）", "16", function(value)
local char = LocalPlayer.Character
if char then
local humanoid = char:FindFirstChildOfClass("Humanoid")
if humanoid then
humanoid.WalkSpeed = value
end
end
end)

createLabeledInput("ジャンプ力倍率（0.0001〜1億）", "50", function(value)
local char = LocalPlayer.Character
if char then
local humanoid = char:FindFirstChildOfClass("Humanoid")
if humanoid then
humanoid.JumpPower = value
end
end
end)
local aimCircle = Drawing.new("Circle")
aimCircle.Visible = false
aimCircle.Color = Color3.new(1, 0, 0)
aimCircle.Thickness = 2
aimCircle.Radius = 120
aimCircle.Transparency = 0.6
aimCircle.Filled = false

local aimLine = Drawing.new("Line")
aimLine.Visible = false
aimLine.Color = Color3.new(1, 0, 0)
aimLine.Thickness = 2
aimLine.Transparency = 0.7

local function isEnemy(player)
if player.Team == nil or LocalPlayer.Team == nil then return true end
return player.Team ~= LocalPlayer.Team
end

local function getClosestEnemy()
local closest = nil
local shortest = math.huge
local cam = workspace.CurrentCamera
local center = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)

for _, p in pairs(Players:GetPlayers()) do  
    if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") and isEnemy(p) then  
        local head = p.Character.Head  
        local pos, onScreen = cam:WorldToViewportPoint(head.Position)  
        if onScreen then  
            local dist = (Vector2.new(pos.X, pos.Y) - center).Magnitude  
            if dist < shortest then  
                shortest = dist  
                closest = head  
            end  
        end  
    end  
end  

return closest  


end

local activeESP = {}

local function drawESPBox(character, player)
if not character:FindFirstChild("HumanoidRootPart") then return nil end

local box = Drawing.new("Square")  
box.Color = Color3.new(1, 0, 0)  
box.Thickness = 2  
box.Transparency = 0.7  
box.Filled = false  
box.Visible = false  

local text = Drawing.new("Text")  
text.Size = 14  
text.Center = true  
text.Outline = true  
text.Visible = false  
text.Color = Color3.new(1, 1, 1)  

local function update()  
    if not character or not character:FindFirstChild("HumanoidRootPart") then  
        box.Visible = false  
        text.Visible = false  
        return  
    end  

    local hrp = character.HumanoidRootPart  
    local size = Vector3.new(4, 6, 1)  
    local corners = {  
        hrp.CFrame * CFrame.new(-size.X,  size.Y, 0),  
        hrp.CFrame * CFrame.new( size.X,  size.Y, 0),  
        hrp.CFrame * CFrame.new( size.X, -size.Y, 0),  
        hrp.CFrame * CFrame.new(-size.X, -size.Y, 0)  
    }  

    local minX, minY, maxX, maxY = math.huge, math.huge, -math.huge, -math.huge  
    local visible = true  

    for _, corner in ipairs(corners) do  
        local screen, onScreen = Camera:WorldToViewportPoint(corner.Position)  
        if not onScreen then  
            visible = false  
            break  
        end  
        minX = math.min(minX, screen.X)  
        minY = math.min(minY, screen.Y)  
        maxX = math.max(maxX, screen.X)  
        maxY = math.max(maxY, screen.Y)  
    end  

    if visible then  
        box.Size = Vector2.new(maxX - minX, maxY - minY)  
        box.Position = Vector2.new(minX, minY)  
        box.Visible = true  

        local head = character:FindFirstChild("Head")  
        local humanoid = character:FindFirstChildOfClass("Humanoid")  
        if head and humanoid then  
            local headPos, onScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))  
            if onScreen then  
                local distance = (Camera.CFrame.Position - head.Position).Magnitude  
                text.Position = Vector2.new(headPos.X, headPos.Y - 20)  
                text.Text = string.format("%s | HP: %d | %.1fm", player.Name, humanoid.Health, distance)  
                text.Visible = true  
            else  
                text.Visible = false  
            end  
        end  
    else  
        box.Visible = false  
        text.Visible = false  
    end  
end  

return {box = box, text = text, update = update}  


end

RunService.RenderStepped:Connect(function()
local character = LocalPlayer.Character
local humanoid = character and character:FindFirstChildOfClass("Humanoid")
local hrp = character and character:FindFirstChild("HumanoidRootPart")

local cam = workspace.CurrentCamera  
local center = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)  
aimCircle.Position = center  

if states.autoAim then  
    aimCircle.Visible = true  
    local target = getClosestEnemy()  

    if target then  
        local bodyPart = target.Parent:FindFirstChild("HumanoidRootPart") or target  
        local screenPos, visible = cam:WorldToViewportPoint(bodyPart.Position)  

        if visible then  
            aimLine.Visible = true  
            aimLine.From = center  
            aimLine.To = Vector2.new(screenPos.X, screenPos.Y)  

            local origin = cam.CFrame.Position  
            local direction = (bodyPart.Position - origin).Unit  
            cam.CFrame = CFrame.new(origin, origin + direction)  

            if hrp then  
                local lookVector = Vector3.new(direction.X, 0, direction.Z).Unit  
                hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + lookVector)  
                if humanoid then humanoid.AutoRotate = false end  
            end  
        else  
            aimLine.Visible = false  
        end  
    else  
        aimLine.Visible = false  
        if hrp and humanoid then  
            local camLook = Vector3.new(cam.CFrame.LookVector.X, 0, cam.CFrame.LookVector.Z).Unit  
            hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + camLook)  
            humanoid.AutoRotate = false  
        end  
    end  
else  
    aimCircle.Visible = false  
    aimLine.Visible = false  
    if humanoid then humanoid.AutoRotate = true end  
end  

if states.noclip and character then  
    for _, part in pairs(character:GetDescendants()) do  
        if part:IsA("BasePart") then  
            part.CanCollide = false  
        end  
    end  
end  

if states.esp then  
    for _, player in pairs(Players:GetPlayers()) do  
        if player ~= LocalPlayer and player.Character and isEnemy(player) then  
            if not activeESP[player] then  
                local espData = drawESPBox(player.Character, player)  
                activeESP[player] = espData  
            end  
        end  
    end  

    for player, data in pairs(activeESP) do  
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then  
            if data.box then data.box:Remove() end  
            if data.text then data.text:Remove() end  
            activeESP[player] = nil  
        else  
            data.update()  
        end  
    end  
else  
    for _, data in pairs(activeESP) do  
        if data.box then data.box:Remove() end  
        if data.text then data.text:Remove() end  
    end  
    activeESP = {}  
end  


end)

