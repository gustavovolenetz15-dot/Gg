-- LocalScript em StarterPlayerScripts para teste no Roblox Studio
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local userInputService = game:GetService("UserInputService")
local defaultWalkSpeed = humanoid.WalkSpeed
local defaultJumpHeight = humanoid.JumpHeight
local speedEnabled = false
local jumpEnabled = false
local espEnabled = false
local highlights = {}

-- Funções
local function toggleSpeed()
    speedEnabled = not speedEnabled
    humanoid.WalkSpeed = speedEnabled and defaultWalkSpeed + 50 or defaultWalkSpeed
end

local function toggleJump()
    jumpEnabled = not jumpEnabled
    humanoid.JumpHeight = jumpEnabled and defaultJumpHeight + 50 or defaultJumpHeight
end

local function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        for _, p in ipairs(game.Players:GetPlayers()) do
            if p ~= player and p.Character then
                local hl = Instance.new("Highlight", p.Character)
                hl.FillColor = Color3.fromRGB(255, 0, 0)
                hl.OutlineColor = Color3.fromRGB(255, 255, 0)
                hl.FillTransparency = 0.5
                highlights[p] = hl
            end
        end
    else
        for _, hl in pairs(highlights) do
            if hl then hl:Destroy() end
        end
        highlights = {}
    end
end

-- Respawn do personagem
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    if speedEnabled then humanoid.WalkSpeed = defaultWalkSpeed + 50 end
    if jumpEnabled then humanoid.JumpHeight = defaultJumpHeight + 50 end
    if espEnabled then toggleESP() end
end)

-- GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.IgnoreGuiInset = true
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 150, 0, 150)
frame.Position = UDim2.new(0.5, -75, 0.5, -75)
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

-- Função para criar botões
local function createButton(name, y, toggleFunc)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.Position = UDim2.new(0, 0, 0, y)
    btn.Text = name .. ": OFF"
    btn.TextScaled = true
    btn.MouseButton1Click:Connect(function()
        toggleFunc()
        btn.Text = name .. ": " .. (name == "Speed" and (speedEnabled and "ON" or "OFF") or
                                   name == "Jump" and (jumpEnabled and "ON" or "OFF") or
                                   name == "ESP" and (espEnabled and "ON" or "OFF"))
    end)
end

-- Botões
createButton("Speed", 30, toggleSpeed)
createButton("Jump", 60, toggleJump)
createButton("ESP", 90, toggleESP)

-- Botão de arrastar
local dragButton = Instance.new("TextButton", frame)
dragButton.Size = UDim2.new(1, 0, 0, 30)
dragButton.Position = UDim2.new(0, 0, 0, 0)
dragButton.Text = "Mover"
dragButton.TextScaled = true
dragButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70)

-- Lógica de arrastar (mouse e toque)
local dragging, dragStart, startPos
userInputService.InputBegan:Connect(function(input)
    if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and
       input.Position.X >= frame.AbsolutePosition.X and input.Position.X <= frame.AbsolutePosition.X + frame.AbsoluteSize.X and
       input.Position.Y >= frame.AbsolutePosition.Y and input.Position.Y <= frame.AbsolutePosition.Y + dragButton.Size.Y.Offset then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

userInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

userInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ESP para jogadores entrando/saindo
game.Players.PlayerAdded:Connect(function(p)
    if espEnabled and p ~= player then
        p.CharacterAdded:Connect(function(char)
            local hl = Instance.new("Highlight", char)
            hl.FillColor = Color3.fromRGB(255, 0, 0)
            hl.OutlineColor = Color3.fromRGB(255, 255, 0)
            hl.FillTransparency = 0.5
            highlights[p] = hl
        end)
    end
end)

game.Players.PlayerRemoving:Connect(function(p)
    if highlights[p] then
        highlights[p]:Destroy()
        highlights[p] = nil
    end
end)
