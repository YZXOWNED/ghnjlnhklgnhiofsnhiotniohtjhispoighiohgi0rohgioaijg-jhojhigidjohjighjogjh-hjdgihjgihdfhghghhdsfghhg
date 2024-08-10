_G.HeadSize = 3.5
_G.Disabled = true

local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Create GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.CoreGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 200, 0, 150)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
Frame.Visible = false
Frame.Parent = ScreenGui

local ToggleESPButton = Instance.new("TextButton")
ToggleESPButton.Size = UDim2.new(0, 180, 0, 50)
ToggleESPButton.Position = UDim2.new(0, 10, 0, 10)
ToggleESPButton.Text = "Toggle head"
ToggleESPButton.Parent = Frame

local HeadSizeLabel = Instance.new("TextLabel")
HeadSizeLabel.Size = UDim2.new(0, 180, 0, 30)
HeadSizeLabel.Position = UDim2.new(0, 10, 0, 70)
HeadSizeLabel.Text = "Head Size: " .. _G.HeadSize
HeadSizeLabel.Parent = Frame

local HeadSizeSlider = Instance.new("TextButton")
HeadSizeSlider.Size = UDim2.new(0, 180, 0, 30)
HeadSizeSlider.Position = UDim2.new(0, 10, 0, 110)
HeadSizeSlider.Text = "Adjust Head Size"
HeadSizeSlider.Parent = Frame

-- Toggle GUI visibility with 'K' key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
        Frame.Visible = not Frame.Visible
    end
end)

-- Toggle ESP
ToggleESPButton.MouseButton1Click:Connect(function()
    _G.Disabled = not _G.Disabled
end)

-- Adjust Head Size
HeadSizeSlider.MouseButton1Click:Connect(function()
    if _G.HeadSize < 25 then
        _G.HeadSize = _G.HeadSize + 0.5
    else
        _G.HeadSize = 0.5
    end
    HeadSizeLabel.Text = "Head Size: " .. _G.HeadSize
end)

-- ESP Functionality
RunService.RenderStepped:Connect(function()
    if not _G.Disabled then
        for _, v in next, Players:GetPlayers() do
            if v ~= LocalPlayer then  -- Ensure ESP is not applied to the local player
                pcall(function()
                    if v.Character and v.Character:FindFirstChild("Head") then
                        v.Character.Head.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
                        v.Character.Head.Transparency = 0.4  -- Set to 0.3 for semi-transparency
                        v.Character.Head.BrickColor = BrickColor.new("Red")
                        v.Character.Head.Material = "Neon"
                        v.Character.Head.CanCollide = false
                        v.Character.Head.Massless = true
                    end
                end)
            end
        end
    end
end)

local BoxESP = {}

function BoxESP.Create(Player)
    if Player == LocalPlayer then return end  -- Skip creating ESP box for the local player
    if BoxESP[Player] then return end

    local character = Player.Character
    if character and character:IsDescendantOf(workspace) and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
        local Box = Drawing.new("Square")
        Box.Visible = false
        Box.Color = Color3.fromRGB(255, 255, 255)
        Box.Filled = false
        Box.Thickness = 1

        local Connection
        Connection = RunService.RenderStepped:Connect(function()
            if character and character:IsDescendantOf(workspace) and character:FindFirstChild("Humanoid") and character.Humanoid.Health > 0 then
                local Pos, Visible = workspace.CurrentCamera:WorldToViewportPoint(character.HumanoidRootPart.Position)
                local scale_factor = 1 / (Pos.Z * math.tan(math.rad(workspace.CurrentCamera.FieldOfView * 0.5)) * 2) * 100
                local width, height = math.floor(40 * scale_factor), math.floor(62 * scale_factor)

                if Visible then
                    Box.Size = Vector2.new(width, height)
                    Box.Position = Vector2.new(Pos.X - Box.Size.X / 2, Pos.Y - Box.Size.Y / 2)
                    Box.Visible = true
                else
                    Box.Visible = false
                end
            else
                Box:Destroy()
                Connection:Disconnect()
                BoxESP[Player] = nil
            end
        end)

        BoxESP[Player] = {
            Box = Box,
            Connection = Connection
        }
    end
end

local function updateESP()
    for _, Player in pairs(Players:GetPlayers()) do
        if Player.Character then
            BoxESP.Create(Player)
        end
    end
end

RunService.RenderStepped:Connect(updateESP)

-- Ensure ESP boxes for the local player are removed if needed
Players.PlayerRemoving:Connect(function(player)
    if BoxESP[player] then
        BoxESP[player].Box:Destroy()
        BoxESP[player].Connection:Disconnect()
        BoxESP[player] = nil
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    if BoxESP[LocalPlayer] then
        BoxESP[LocalPlayer].Box:Destroy()
        BoxESP[LocalPlayer].Connection:Disconnect()
        BoxESP[LocalPlayer] = nil
    end
end)
