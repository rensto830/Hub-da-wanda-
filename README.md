


local TweenService = game:GetService(\"TweenService\")
local UserInputService = game:GetService(\"UserInputService\")
local RunService = game:GetService(\"RunService\")
local Players = game:GetService(\"Players\")
local Debris = game:GetService(\"Debris\")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Mouse = LocalPlayer:GetMouse()

--═══════════════════════════════════════════════════════
--  CONFIGURAÇÕES DE CORES E MODOS
--═══════════════════════════════════════════════════════

local Modes = {
    Normal = {
        MainColor = Color3.fromRGB(170, 0, 0),
        GlowColor = Color3.fromRGB(255, 0, 0),
        SecondaryColor = Color3.fromRGB(255, 100, 100),
        Name = \"SCARLET WITCH\"
    },
    Darkhold = {
        MainColor = Color3.fromRGB(80, 0, 80),
        GlowColor = Color3.fromRGB(150, 0, 150),
        SecondaryColor = Color3.fromRGB(200, 0, 200),
        Name = \"DARKHOLD CORRUPTED\"
    }
}

local CurrentMode = \"Normal\"
local function GetColor() return Modes[CurrentMode].MainColor end
local function GetGlow() return Modes[CurrentMode].GlowColor end
local function GetSecondary() return Modes[CurrentMode].SecondaryColor end

local DarkBg = Color3.fromRGB(15, 15, 15)
local CooldownTable = {}

--═══════════════════════════════════════════════════════
--  FUNÇÕES UTILITÁRIAS
--═══════════════════════════════════════════════════════

-- Verificar Cooldown
local function CheckCooldown(name, time)
    if CooldownTable[name] and tick() - CooldownTable[name] < time then
        return false
    end
    CooldownTable[name] = tick()
    return true
end

-- Obter Mãos (R6 & R15)
local function GetHands()
    local char = LocalPlayer.Character
    if not char then return {}, {} end
    local right = char:FindFirstChild(\"RightHand\") or char:FindFirstChild(\"Right Arm\")
    local left = char:FindFirstChild(\"LeftHand\") or char:FindFirstChild(\"Left Arm\")
    return right, left
end

-- Obter Root e Humanoid
local function GetCharParts()
    local char = LocalPlayer.Character
    if not char then return nil, nil end
    return char:FindFirstChild(\"HumanoidRootPart\"), char:FindFirstChild(\"Humanoid\")
end

-- Som Customizado
local function PlaySound(id, volume)
    local root = GetCharParts()
    if not root then return end
    local sound = Instance.new(\"Sound\", root)
    sound.SoundId = \"rbxassetid://\" .. tostring(id)
    sound.Volume = volume or 0.5
    sound:Play()
    Debris:AddItem(sound, 3)
end

-- Criar Efeito de Partículas
local function CreateParticleEffect(part, color, lifetime)
    local p = Instance.new(\"ParticleEmitter\", part)
    p.Color = ColorSequence.new(color)
    p.LightEmission = 1
    p.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.5, 2),
        NumberSequenceKeypoint.new(1, 0)
    })
    p.Texture = \"rbxassetid://244221440\"
    p.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1)
    })
    p.Lifetime = NumberRange.new(lifetime or 1)
    p.Rate = 80
    p.Speed = NumberRange.new(8)
    p.SpreadAngle = Vector2.new(180, 180)
    delay(2, function() 
        p.Enabled = false 
        wait(2) 
        p:Destroy() 
    end)
    return p
end

-- Criar Aura Contínua
local function CreateAura(part, color, size)
    local aura = Instance.new(\"ParticleEmitter\", part)
    aura.Color = ColorSequence.new(color)
    aura.LightEmission = 1
    aura.Size = NumberSequence.new(size or 2)
    aura.Texture = \"rbxassetid://244221440\"
    aura.Transparency = NumberSequence.new(0.3, 1)
    aura.Lifetime = NumberRange.new(1, 2)
    aura.Rate = 50
    aura.Speed = NumberRange.new(3)
    aura.SpreadAngle = Vector2.new(360, 360)
    return aura
end

-- Criar Projétil
local function CreateProjectile(startPos, targetPos, color, callback)
    local projectile = Instance.new(\"Part\", workspace)
    projectile.Size = Vector3.new(2, 2, 2)
    projectile.CFrame = CFrame.new(startPos)
    projectile.Color = color
    projectile.Material = Enum.Material.Neon
    projectile.Shape = Enum.PartType.Ball
    projectile.CanCollide = false
    projectile.Anchored = true
    
    CreateParticleEffect(projectile, color, 0.8)
    
    local distance = (targetPos - startPos).Magnitude
    local duration = distance / 100
    
    TweenService:Create(projectile, TweenInfo.new(duration, Enum.EasingStyle.Linear), {
        CFrame = CFrame.new(targetPos)
    }):Play()
    
    delay(duration, function()
        if callback then callback(projectile) end
        projectile:Destroy()
    end)
    
    return projectile
end

--═══════════════════════════════════════════════════════
--  CRIAR INTERFACE GUI
--═══════════════════════════════════════════════════════

local ScreenGui = Instance.new(\"ScreenGui\")
ScreenGui.Name = \"WandaHub_VR8_Enhanced\"
ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local isOpen = true

-- Função de Arrastar
local function MakeDraggable(topbar, object)
    local dragging, dragInput, dragStart, startPos
    topbar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = object.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then 
                    dragging = false 
                end
            end)
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                local delta = input.Position - dragStart
                object.Position = UDim2.new(
                    startPos.X.Scale, startPos.X.Offset + delta.X,
                    startPos.Y.Scale, startPos.Y.Offset + delta.Y
                )
            end
        end
    end)
end

-- Painel Principal
local MainFrame = Instance.new(\"Frame\")
MainFrame.Name = \"MainFrame\"
MainFrame.Size = UDim2.new(0, 420, 0, 500)
MainFrame.Position = UDim2.new(0.5, -210, 0.5, -250)
MainFrame.BackgroundColor3 = DarkBg
MainFrame.BackgroundTransparency = 0.1
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new(\"UICorner\", MainFrame)
MainCorner.CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new(\"UIStroke\", MainFrame)
MainStroke.Color = GetColor()
MainStroke.Thickness = 3

-- TopBar
local TopBar = Instance.new(\"Frame\")
TopBar.Size = UDim2.new(1, 0, 0, 50)
TopBar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TopCorner = Instance.new(\"UICorner\", TopBar)
TopCorner.CornerRadius = UDim.new(0, 12)

MakeDraggable(TopBar, MainFrame)

local Title = Instance.new(\"TextLabel\")
Title.Text = \"✨ \" .. Modes[CurrentMode].Name .. \" [VR8] ✨\"
Title.Size = UDim2.new(1, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.TextColor3 = GetColor()
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.Parent = TopBar

-- Botão de Modo
local ModeBtn = Instance.new(\"TextButton\")
ModeBtn.Size = UDim2.new(0, 120, 0, 30)
ModeBtn.Position = UDim2.new(0.5, -60, 0, 55)
ModeBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ModeBtn.Text = \"🔄 TROCAR MODO\"
ModeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ModeBtn.Font = Enum.Font.GothamBold
ModeBtn.TextSize = 12
ModeBtn.Parent = MainFrame

local ModeCorner = Instance.new(\"UICorner\", ModeBtn)
ModeCorner.CornerRadius = UDim.new(0, 8)

local ModeStroke = Instance.new(\"UIStroke\", ModeBtn)
ModeStroke.Color = GetColor()
ModeStroke.Thickness = 2

ModeBtn.MouseButton1Click:Connect(function()
    CurrentMode = (CurrentMode == \"Normal\") and \"Darkhold\" or \"Normal\"
    Title.Text = \"✨ \" .. Modes[CurrentMode].Name .. \" [VR8] ✨\"
    Title.TextColor3 = GetColor()
    MainStroke.Color = GetColor()
    ModeStroke.Color = GetColor()
    PlaySound(6518380871, 0.3)
end)

-- Abas de Categorias
local CategoriesFrame = Instance.new(\"Frame\")
CategoriesFrame.Size = UDim2.new(1, -20, 0, 40)
CategoriesFrame.Position = UDim2.new(0, 10, 0, 95)
CategoriesFrame.BackgroundTransparency = 1
CategoriesFrame.Parent = MainFrame

local CatLayout = Instance.new(\"UIListLayout\", CategoriesFrame)
CatLayout.FillDirection = Enum.FillDirection.Horizontal
CatLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
CatLayout.Padding = UDim.new(0, 8)

local CurrentCategory = \"Ataque\"
local CategoryButtons = {}

-- Scrolling Frame
local ScrollFrame = Instance.new(\"ScrollingFrame\")
ScrollFrame.Size = UDim2.new(1, -20, 1, -155)
ScrollFrame.Position = UDim2.new(0, 10, 0, 145)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel = 0
ScrollFrame.ScrollBarThickness = 4
ScrollFrame.ScrollBarImageColor3 = GetColor()
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
ScrollFrame.Parent = MainFrame

local PowersList = Instance.new(\"UIListLayout\", ScrollFrame)
PowersList.Padding = UDim.new(0, 8)
PowersList.HorizontalAlignment = Enum.HorizontalAlignment.Center
PowersList:GetPropertyChangedSignal(\"AbsoluteContentSize\"):Connect(function()
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, PowersList.AbsoluteContentSize.Y + 10)
end)

-- Botão de Toggle (Mobile)
local ToggleBtn = Instance.new(\"TextButton\")
ToggleBtn.Size = UDim2.new(0, 55, 0, 55)
ToggleBtn.Position = UDim2.new(0, 20, 0.5, -27)
ToggleBtn.BackgroundColor3 = DarkBg
ToggleBtn.Text = \"🔴\"
ToggleBtn.TextSize = 30
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.Parent = ScreenGui

local ToggleCorner = Instance.new(\"UICorner\", ToggleBtn)
ToggleCorner.CornerRadius = UDim.new(1, 0)

local ToggleStroke = Instance.new(\"UIStroke\", ToggleBtn)
ToggleStroke.Color = GetColor()
ToggleStroke.Thickness = 3

ToggleBtn.MouseButton1Click:Connect(function()
    isOpen = not isOpen
    local targetSize = isOpen and UDim2.new(0, 420, 0, 500) or UDim2.new(0, 0, 0, 0)
    TweenService:Create(MainFrame, TweenInfo.new(0.4, Enum.EasingStyle.Quart), {Size = targetSize}):Play()
    MainFrame.Visible = true
    if not isOpen then 
        wait(0.4) 
        if not isOpen then MainFrame.Visible = false end 
    end
end)

--═══════════════════════════════════════════════════════
--  SISTEMA DE PODERES E CATEGORIAS
--═══════════════════════════════════════════════════════

local Powers = {
    [\"Ataque\"] = {},
    [\"Defesa\"] = {},
    [\"Controle Mental\"] = {},
    [\"Realidade\"] = {},
    [\"Movimento\"] = {},
    [\"Especial\"] = {}
}

-- Função para criar botão de categoria
local function CreateCategoryButton(name)
    local btn = Instance.new(\"TextButton\")
    btn.Name = name
    btn.Size = UDim2.new(0, 90, 0, 35)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(150, 150, 150)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 11
    btn.AutoButtonColor = false
    btn.Parent = CategoriesFrame
    
    local corner = Instance.new(\"UICorner\", btn)
    corner.CornerRadius = UDim.new(0, 6)
    
    local stroke = Instance.new(\"UIStroke\", btn)
    stroke.Color = Color3.fromRGB(50, 50, 50)
    stroke.Thickness = 1
    
    CategoryButtons[name] = {Button = btn, Stroke = stroke}
    
    btn.MouseButton1Click:Connect(function()
        CurrentCategory = name
        for catName, data in pairs(CategoryButtons) do
            if catName == name then
                data.Button.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
                data.Button.TextColor3 = Color3.fromRGB(255, 255, 255)
                data.Stroke.Color = GetColor()
            else
                data.Button.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
                data.Button.TextColor3 = Color3.fromRGB(150, 150, 150)
                data.Stroke.Color = Color3.fromRGB(50, 50, 50)
            end
        end
        UpdatePowersDisplay()
    end)
    
    return btn
end

-- Criar botões de categoria
for _, cat in ipairs({\"Ataque\", \"Defesa\", \"Controle Mental\", \"Realidade\", \"Movimento\", \"Especial\"}) do
    CreateCategoryButton(cat)
end
CategoryButtons[\"Ataque\"].Button.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
CategoryButtons[\"Ataque\"].Button.TextColor3 = Color3.fromRGB(255, 255, 255)
CategoryButtons[\"Ataque\"].Stroke.Color = GetColor()

-- Função para criar botão de poder
local function CreatePowerButton(category, name, description, cooldown, callback)
    table.insert(Powers[category], {
        Name = name,
        Description = description,
        Cooldown = cooldown,
        Callback = callback
    })
end

-- Atualizar display de poderes
function UpdatePowersDisplay()
    for _, child in ipairs(ScrollFrame:GetChildren()) do
        if child:IsA(\"TextButton\") then
            child:Destroy()
        end
    end
    
    for _, power in ipairs(Powers[CurrentCategory]) do
        local btn = Instance.new(\"TextButton\")
        btn.Name = power.Name
        btn.Size = UDim2.new(0.95, 0, 0, 50)
        btn.BackgroundColor3 = Color3.fromRGB(28, 28, 28)
        btn.Text = \"\"
        btn.AutoButtonColor = false
        btn.Parent = ScrollFrame
        
        local corner = Instance.new(\"UICorner\", btn)
        corner.CornerRadius = UDim.new(0, 8)
        
        local stroke = Instance.new(\"UIStroke\", btn)
        stroke.Color = Color3.fromRGB(60, 0, 0)
        stroke.Thickness = 1.5
        
        local title = Instance.new(\"TextLabel\")
        title.Size = UDim2.new(1, -10, 0, 20)
        title.Position = UDim2.new(0, 5, 0, 5)
        title.BackgroundTransparency = 1
        title.Text = \"⚡ \" .. power.Name
        title.TextColor3 = Color3.fromRGB(255, 200, 200)
        title.Font = Enum.Font.GothamBold
        title.TextSize = 13
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.Parent = btn
        
        local desc = Instance.new(\"TextLabel\")
        desc.Size = UDim2.new(1, -10, 0, 15)
        desc.Position = UDim2.new(0, 5, 0, 27)
        desc.BackgroundTransparency = 1
        desc.Text = power.Description
        desc.TextColor3 = Color3.fromRGB(150, 150, 150)
        desc.Font = Enum.Font.Gotham
        desc.TextSize = 10
        desc.TextXAlignment = Enum.TextXAlignment.Left
        desc.TextWrapped = true
        desc.Parent = btn
        
        btn.MouseEnter:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.2), {
                BackgroundColor3 = Color3.fromRGB(50, 10, 10)
            }):Play()
            TweenService:Create(stroke, TweenInfo.new(0.2), {
                Color = GetColor()
            }):Play()
        end)
        
        btn.MouseLeave:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.2), {
                BackgroundColor3 = Color3.fromRGB(28, 28, 28)
            }):Play()
            TweenService:Create(stroke, TweenInfo.new(0.2), {
                Color = Color3.fromRGB(60, 0, 0)
            }):Play()
        end)
        
        btn.MouseButton1Click:Connect(function()
            if CheckCooldown(power.Name, power.Cooldown) then
                power.Callback()
                PlaySound(6518380871, 0.2)
            else
                warn(\"[Cooldown] \" .. power.Name .. \" ainda em cooldown!\")
            end
        end)
    end
end

--═══════════════════════════════════════════════════════
--  IMPLEMENTAÇÃO DAS 43 HABILIDADES DA WANDA
--═══════════════════════════════════════════════════════

--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
--  🔴 CATEGORIA: ATAQUE
--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

-- 1. Hex Bolts (Clássico dos Comics)
CreatePowerButton(\"Ataque\", \"Hex Bolts\", \"Projéteis de energia do caos\", 1, function()
    local root = GetCharParts()
    if not root then return end
    
    for i = 1, 5 do
        delay(i * 0.1, function()
            local targetPos = Mouse.Hit.p
            CreateProjectile(root.Position, targetPos, GetColor(), function(proj)
                local explosion = Instance.new(\"Part\", workspace)
                explosion.Shape = Enum.PartType.Ball
                explosion.Size = Vector3.new(8, 8, 8)
                explosion.CFrame = proj.CFrame
                explosion.Color = GetColor()
                explosion.Material = Enum.Material.Neon
                explosion.Anchored = true
                explosion.CanCollide = false
                
                CreateParticleEffect(explosion, GetColor(), 1)
                TweenService:Create(explosion, TweenInfo.new(0.5), {
                    Size = Vector3.new(15, 15, 15),
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(explosion, 0.5)
            end)
        end)
    end
end)

-- 2. Chaos Blast (Age of Ultron)
CreatePowerButton(\"Ataque\", \"Chaos Blast\", \"Explosão massiva de caos\", 3, function()
    local root = GetCharParts()
    if not root then return end
    
    local blast = Instance.new(\"Part\", workspace)
    blast.Shape = Enum.PartType.Ball
    blast.Size = Vector3.new(5, 5, 5)
    blast.CFrame = root.CFrame * CFrame.new(0, 0, -5)
    blast.CanCollide = false
    blast.Anchored = true
    blast.Color = GetColor()
    blast.Material = Enum.Material.Neon
    
    CreateParticleEffect(blast, GetColor(), 2)
    PlaySound(9125402735, 0.8)
    
    TweenService:Create(blast, TweenInfo.new(1), {
        Size = Vector3.new(40, 40, 40),
        Transparency = 1
    }):Play()
    
    -- Empurrar inimigos
    for _, v in pairs(workspace:GetChildren()) do
        if v:IsA(\"Model\") and v:FindFirstChild(\"HumanoidRootPart\") and v ~= Character then
            if (v.HumanoidRootPart.Position - root.Position).Magnitude < 25 then
                local bv = Instance.new(\"BodyVelocity\", v.HumanoidRootPart)
                bv.Velocity = (v.HumanoidRootPart.Position - root.Position).Unit * 150
                bv.MaxForce = Vector3.new(1e6, 1e6, 1e6)
                Debris:AddItem(bv, 0.3)
            end
        end
    end
    
    Debris:AddItem(blast, 1)
end)

-- 3. Energy Burst (Civil War)
CreatePowerButton(\"Ataque\", \"Energy Burst\", \"Explosão de energia em todas direções\", 2, function()
    local root = GetCharParts()
    if not root then return end
    
    PlaySound(9114397505, 0.6)
    
    for i = 1, 12 do
        local angle = (i / 12) * math.pi * 2
        local burst = Instance.new(\"Part\", workspace)
        burst.Size = Vector3.new(3, 3, 3)
        burst.CFrame = root.CFrame
        burst.Color = GetColor()
        burst.Material = Enum.Material.Neon
        burst.Shape = Enum.PartType.Ball
        burst.Anchored = true
        burst.CanCollide = false
        
        CreateParticleEffect(burst, GetColor(), 1)
        
        local targetCFrame = root.CFrame * CFrame.new(
            math.cos(angle) * 30,
            math.sin(angle) * 30,
            0
        )
        
        TweenService:Create(burst, TweenInfo.new(1.2), {
            CFrame = targetCFrame,
            Size = Vector3.new(8, 8, 8),
            Transparency = 1
        }):Play()
        
        Debris:AddItem(burst, 1.2)
    end
end)

-- 4. Chaos Wave (Comics)
CreatePowerButton(\"Ataque\", \"Chaos Wave\", \"Onda destrutiva de caos\", 4, function()
    local root = GetCharParts()
    if not root then return end
    
    for i = 1, 5 do
        delay(i * 0.2, function()
            local wave = Instance.new(\"Part\", workspace)
            wave.Size = Vector3.new(50, 1, 5)
            wave.CFrame = root.CFrame * CFrame.new(0, 0, -10 - (i * 8))
            wave.Color = GetColor()
            wave.Material = Enum.Material.Neon
            wave.Anchored = true
            wave.CanCollide = false
            wave.Transparency = 0.3
            
            CreateParticleEffect(wave, GetColor(), 0.8)
            
            TweenService:Create(wave, TweenInfo.new(0.8), {
                Size = Vector3.new(60, 8, 8),
                Transparency = 1
            }):Play()
            
            Debris:AddItem(wave, 0.8)
        end)
    end
end)

-- 5. Molecular Disintegration (Multiverse of Madness)
CreatePowerButton(\"Ataque\", \"Molecular Disintegration\", \"Desintegra objetos em nível molecular\", 5, function()
    local target = Mouse.Target
    if target and not target.Anchored and not Players:GetPlayerFromCharacter(target.Parent) then
        CreateParticleEffect(target, GetColor(), 2)
        PlaySound(9112836991, 0.5)
        
        for i = 1, 10 do
            local fragment = Instance.new(\"Part\", workspace)
            fragment.Size = target.Size / 5
            fragment.CFrame = target.CFrame * CFrame.new(
                math.random(-2, 2),
                math.random(-2, 2),
                math.random(-2, 2)
            )
            fragment.Color = GetColor()
            fragment.Material = Enum.Material.Neon
            fragment.Anchored = false
            fragment.CanCollide = false
            
            TweenService:Create(fragment, TweenInfo.new(1.5), {
                Transparency = 1,
                Size = Vector3.new(0, 0, 0)
            }):Play()
            
            Debris:AddItem(fragment, 1.5)
        end
        
        target:Destroy()
    end
end)

-- 6. Chaos Lightning (Comics)
CreatePowerButton(\"Ataque\", \"Chaos Lightning\", \"Raios de energia do caos\", 2, function()
    local root = GetCharParts()
    if not root then return end
    
    for i = 1, 3 do
        delay(i * 0.15, function()
            local lightning = Instance.new(\"Part\", workspace)
            lightning.Size = Vector3.new(1, 50, 1)
            lightning.CFrame = CFrame.new(Mouse.Hit.p + Vector3.new(0, 25, 0))
            lightning.Color = GetColor()
            lightning.Material = Enum.Material.Neon
            lightning.Anchored = true
            lightning.CanCollide = false
            
            CreateParticleEffect(lightning, GetColor(), 1)
            PlaySound(9113800142, 0.4)
            
            delay(0.3, function()
                TweenService:Create(lightning, TweenInfo.new(0.3), {Transparency = 1}):Play()
                Debris:AddItem(lightning, 0.3)
            end)
        end)
    end
end)

-- 7. Necrotic Bolts (Darkhold)
CreatePowerButton(\"Ataque\", \"Necrotic Bolts\", \"Energia necrótica corrompida\", 3, function()
    local root = GetCharParts()
    if not root then return end
    
    for i = 1, 8 do
        delay(i * 0.08, function()
            local bolt = Instance.new(\"Part\", workspace)
            bolt.Size = Vector3.new(2, 2, 6)
            bolt.CFrame = CFrame.new(root.Position, Mouse.Hit.p)
            bolt.Color = CurrentMode == \"Darkhold\" and Color3.fromRGB(100, 0, 100) or GetColor()
            bolt.Material = Enum.Material.Neon
            bolt.Anchored = true
            bolt.CanCollide = false
            
            CreateParticleEffect(bolt, bolt.Color, 0.8)
            
            TweenService:Create(bolt, TweenInfo.new(0.5), {
                CFrame = CFrame.new(Mouse.Hit.p),
                Transparency = 1
            }):Play()
            
            Debris:AddItem(bolt, 0.5)
        end)
    end
end)

-- 8. Soul Extraction (Darkhold)
CreatePowerButton(\"Ataque\", \"Soul Extraction\", \"Extrai a essência vital\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    local soul = Instance.new(\"Part\", workspace)
    soul.Size = Vector3.new(2, 4, 2)
    soul.CFrame = CFrame.new(Mouse.Hit.p + Vector3.new(0, 2, 0))
    soul.Color = Color3.fromRGB(200, 200, 255)
    soul.Material = Enum.Material.Neon
    soul.Transparency = 0.5
    soul.Anchored = true
    soul.CanCollide = false
    
    CreateAura(soul, Color3.fromRGB(200, 200, 255), 3)
    PlaySound(9116933190, 0.6)
    
    TweenService:Create(soul, TweenInfo.new(2), {
        CFrame = root.CFrame + Vector3.new(0, 5, 0),
        Transparency = 1
    }):Play()
    
    Debris:AddItem(soul, 2)
end)

-- 9. Probability Hex (Comics - Manipulação de Probabilidade)
CreatePowerButton(\"Ataque\", \"Probability Hex\", \"Altera probabilidades causando caos\", 4, function()
    local root = GetCharParts()
    if not root then return end
    
    local hexField = Instance.new(\"Part\", workspace)
    hexField.Shape = Enum.PartType.Cylinder
    hexField.Size = Vector3.new(1, 30, 30)
    hexField.CFrame = root.CFrame * CFrame.Angles(0, 0, math.pi/2)
    hexField.Color = GetColor()
    hexField.Material = Enum.Material.Neon
    hexField.Transparency = 0.7
    hexField.Anchored = true
    hexField.CanCollide = false
    
    -- Criar efeito de glitch
    for i = 1, 20 do
        delay(i * 0.1, function()
            local glitch = Instance.new(\"Part\", workspace)
            glitch.Size = Vector3.new(math.random(1, 3), math.random(1, 3), math.random(1, 3))
            glitch.CFrame = hexField.CFrame * CFrame.new(
                math.random(-15, 15),
                math.random(-15, 15),
                0
            )
            glitch.Color = GetColor()
            glitch.Material = Enum.Material.Neon
            glitch.Anchored = true
            glitch.CanCollide = false
            
            TweenService:Create(glitch, TweenInfo.new(0.2), {
                Transparency = 1,
                Size = Vector3.new(0, 0, 0)
            }):Play()
            
            Debris:AddItem(glitch, 0.2)
        end)
    end
    
    Debris:AddItem(hexField, 2)
end)

-- 10. Chaos Spear (WandaVision)
CreatePowerButton(\"Ataque\", \"Chaos Spear\", \"Lança penetrante de energia pura\", 2.5, function()
    local root = GetCharParts()
    if not root then return end
    
    local spear = Instance.new(\"Part\", workspace)
    spear.Size = Vector3.new(1, 1, 12)
    spear.CFrame = CFrame.new(root.Position, Mouse.Hit.p)
    spear.Color = GetColor()
    spear.Material = Enum.Material.Neon
    spear.Anchored = true
    spear.CanCollide = false
    
    CreateParticleEffect(spear, GetColor(), 1)
    PlaySound(9114221327, 0.5)
    
    local distance = (Mouse.Hit.p - root.Position).Magnitude
    TweenService:Create(spear, TweenInfo.new(distance / 150, Enum.EasingStyle.Linear), {
        CFrame = CFrame.new(Mouse.Hit.p) * spear.CFrame.Rotation
    }):Play()
    
    delay(distance / 150, function()
        local impact = Instance.new(\"Part\", workspace)
        impact.Shape = Enum.PartType.Ball
        impact.Size = Vector3.new(1, 1, 1)
        impact.CFrame = spear.CFrame
        impact.Color = GetColor()
        impact.Material = Enum.Material.Neon
        impact.Anchored = true
        impact.CanCollide = false
        
        TweenService:Create(impact, TweenInfo.new(0.5), {
            Size = Vector3.new(20, 20, 20),
            Transparency = 1
        }):Play()
        
        Debris:AddItem(impact, 0.5)
        spear:Destroy()
    end)
end)

--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
--  🛡️ CATEGORIA: DEFESA
--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

-- 11. Scarlet Shield (Age of Ultron)
CreatePowerButton(\"Defesa\", \"Scarlet Shield\", \"Escudo protetor escarlate\", 3, function()
    local root = GetCharParts()
    if not root then return end
    
    local shield = Instance.new(\"Part\", workspace)
    shield.Size = Vector3.new(18, 18, 1)
    shield.CFrame = root.CFrame * CFrame.new(0, 0, -6)
    shield.Color = GetColor()
    shield.Material = Enum.Material.ForceField
    shield.Transparency = 0.3
    shield.Anchored = true
    shield.CanCollide = true
    
    CreateAura(shield, GetColor(), 2)
    PlaySound(9113494869, 0.4)
    
    Debris:AddItem(shield, 6)
end)

-- 12. Chaos Barrier (Comics)
CreatePowerButton(\"Defesa\", \"Chaos Barrier\", \"Barreira esférica de proteção\", 4, function()
    local root = GetCharParts()
    if not root then return end
    
    local barrier = Instance.new(\"Part\", workspace)
    barrier.Shape = Enum.PartType.Ball
    barrier.Size = Vector3.new(25, 25, 25)
    barrier.CFrame = root.CFrame
    barrier.Color = GetColor()
    barrier.Material = Enum.Material.ForceField
    barrier.Transparency = 0.5
    barrier.Anchored = true
    barrier.CanCollide = true
    
    CreateAura(barrier, GetColor(), 3)
    
    -- Pulsar
    for i = 1, 8 do
        delay(i * 0.8, function()
            if barrier.Parent then
                local pulse = barrier:Clone()
                pulse.Parent = workspace
                pulse.CanCollide = false
                pulse.Transparency = 0.7
                
                TweenService:Create(pulse, TweenInfo.new(0.6), {
                    Size = Vector3.new(35, 35, 35),
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(pulse, 0.6)
            end
        end)
    end
    
    Debris:AddItem(barrier, 7)
end)

-- 13. Force Field Generation (Comics)
CreatePowerButton(\"Defesa\", \"Force Field Dome\", \"Cúpula de campo de força\", 5, function()
    local root = GetCharParts()
    if not root then return end
    
    local dome = Instance.new(\"Part\", workspace)
    dome.Shape = Enum.PartType.Ball
    dome.Size = Vector3.new(35, 35, 35)
    dome.CFrame = root.CFrame
    dome.Color = GetColor()
    dome.Material = Enum.Material.ForceField
    dome.Transparency = 0.6
    dome.Anchored = true
    dome.CanCollide = true
    
    -- Cortar metade inferior para parecer cúpula
    local mesh = Instance.new(\"SpecialMesh\", dome)
    mesh.MeshType = Enum.MeshType.Sphere
    mesh.Scale = Vector3.new(1, 0.5, 1)
    
    CreateAura(dome, GetColor(), 4)
    PlaySound(9113869537, 0.5)
    
    Debris:AddItem(dome, 10)
end)

-- 14. Hex Armor (WandaVision)
CreatePowerButton(\"Defesa\", \"Hex Armor\", \"Armadura de energia hexagonal\", 4, function()
    local root, humanoid = GetCharParts()
    if not root then return end
    
    -- Criar hexágonos ao redor do corpo
    for i = 1, 12 do
        local angle = (i / 12) * math.pi * 2
        local hex = Instance.new(\"Part\", Character)
        hex.Size = Vector3.new(2, 3, 0.2)
        hex.CFrame = root.CFrame * CFrame.new(
            math.cos(angle) * 4,
            0,
            math.sin(angle) * 4
        ) * CFrame.Angles(0, -angle, 0)
        hex.Color = GetColor()
        hex.Material = Enum.Material.Neon
        hex.Transparency = 0.4
        hex.Anchored = true
        hex.CanCollide = false
        
        local weld = Instance.new(\"Weld\", hex)
        weld.Part0 = root
        weld.Part1 = hex
        weld.C0 = root.CFrame:Inverse() * hex.CFrame
        hex.Anchored = false
        
        CreateAura(hex, GetColor(), 1)
        
        delay(5, function()
            TweenService:Create(hex, TweenInfo.new(0.5), {Transparency = 1}):Play()
            Debris:AddItem(hex, 0.5)
        end)
    end
end)

-- 15. Reality Shield (Multiverse of Madness)
CreatePowerButton(\"Defesa\", \"Reality Shield\", \"Escudo que dobra a realidade\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    local shield = Instance.new(\"Part\", workspace)
    shield.Size = Vector3.new(20, 20, 0.5)
    shield.CFrame = root.CFrame * CFrame.new(0, 0, -5)
    shield.Color = GetColor()
    shield.Material = Enum.Material.Glass
    shield.Transparency = 0.2
    shield.Anchored = true
    shield.CanCollide = true
    shield.Reflectance = 0.5
    
    CreateAura(shield, GetColor(), 2.5)
    
    -- Efeito de distorção
    for i = 1, 10 do
        delay(i * 0.3, function()
            if shield.Parent then
                local ripple = Instance.new(\"Part\", workspace)
                ripple.Size = Vector3.new(22, 22, 0.1)
                ripple.CFrame = shield.CFrame
                ripple.Color = GetSecondary()
                ripple.Material = Enum.Material.Neon
                ripple.Transparency = 0.5
                ripple.Anchored = true
                ripple.CanCollide = false
                
                TweenService:Create(ripple, TweenInfo.new(0.4), {
                    Size = Vector3.new(25, 25, 0.1),
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(ripple, 0.4)
            end
        end)
    end
    
    Debris:AddItem(shield, 8)
end)

-- 16. Absorption Field (Comics)
CreatePowerButton(\"Defesa\", \"Energy Absorption\", \"Absorve energia de ataques\", 5, function()
    local root = GetCharParts()
    if not root then return end
    
    local field = Instance.new(\"Part\", workspace)
    field.Shape = Enum.PartType.Ball
    field.Size = Vector3.new(15, 15, 15)
    field.CFrame = root.CFrame
    field.Color = GetColor()
    field.Material = Enum.Material.Neon
    field.Transparency = 0.7
    field.Anchored = true
    field.CanCollide = false
    
    -- Efeito de absorção
    local absorb = CreateAura(field, GetColor(), 3)
    absorb.Speed = NumberRange.new(-10) -- Partículas indo para dentro
    
    PlaySound(9116421983, 0.4)
    
    Debris:AddItem(field, 5)
end)

--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
--  🧠 CATEGORIA: CONTROLE MENTAL
--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

-- 17. Mind Control (Age of Ultron)
CreatePowerButton(\"Controle Mental\", \"Mind Control\", \"Controla a mente do alvo\", 5, function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local theirRoot = player.Character:FindFirstChild(\"HumanoidRootPart\")
            local myRoot = GetCharParts()
            if theirRoot and myRoot and (theirRoot.Position - myRoot.Position).Magnitude < 40 then
                CreateParticleEffect(theirRoot, GetColor(), 2)
                PlaySound(9113027201, 0.5)
                
                -- Levitar alvo
                local bp = Instance.new(\"BodyPosition\", theirRoot)
                bp.Position = theirRoot.Position + Vector3.new(0, 15, 0)
                bp.MaxForce = Vector3.new(1e6, 1e6, 1e6)
                
                -- Criar aura mental
                local aura = CreateAura(theirRoot, Color3.fromRGB(255, 100, 100), 3)
                
                delay(5, function()
                    bp:Destroy()
                    aura:Destroy()
                end)
                
                break
            end
        end
    end
end)

-- 18. Telepathy (Comics)
CreatePowerButton(\"Controle Mental\", \"Telepathic Link\", \"Conexão telepática visual\", 3, function()
    local root = GetCharParts()
    if not root then return end
    
    -- Criar ondas telepáticas
    for i = 1, 8 do
        delay(i * 0.2, function()
            local wave = Instance.new(\"Part\", workspace)
            wave.Shape = Enum.PartType.Ball
            wave.Size = Vector3.new(5, 5, 5)
            wave.CFrame = root.CFrame
            wave.Color = Color3.fromRGB(255, 100, 255)
            wave.Material = Enum.Material.Neon
            wave.Transparency = 0.5
            wave.Anchored = true
            wave.CanCollide = false
            
            TweenService:Create(wave, TweenInfo.new(1), {
                Size = Vector3.new(40, 40, 40),
                Transparency = 1
            }):Play()
            
            Debris:AddItem(wave, 1)
        end)
    end
    
    PlaySound(9113388094, 0.4)
end)

-- 19. Memory Manipulation (WandaVision)
CreatePowerButton(\"Controle Mental\", \"Memory Wipe\", \"Apaga memórias recentes\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    local memory = Instance.new(\"Part\", workspace)
    memory.Shape = Enum.PartType.Ball
    memory.Size = Vector3.new(30, 30, 30)
    memory.CFrame = root.CFrame
    memory.Color = Color3.fromRGB(200, 100, 255)
    memory.Material = Enum.Material.Neon
    memory.Transparency = 0.8
    memory.Anchored = true
    memory.CanCollide = false
    
    CreateParticleEffect(memory, Color3.fromRGB(200, 100, 255), 2)
    PlaySound(9116754535, 0.5)
    
    -- Pulsos de memória
    for i = 1, 5 do
        delay(i * 0.4, function()
            if memory.Parent then
                TweenService:Create(memory, TweenInfo.new(0.3), {
                    Transparency = 0.5
                }):Play()
                wait(0.3)
                TweenService:Create(memory, TweenInfo.new(0.3), {
                    Transparency = 0.8
                }):Play()
            end
        end)
    end
    
    Debris:AddItem(memory, 3)
end)

-- 20. Fear Projection (Age of Ultron)
CreatePowerButton(\"Controle Mental\", \"Fear Visions\", \"Projeta medos na mente\", 4, function()
    local root = GetCharParts()
    if not root then return end
    
    -- Criar sombras de medo
    for i = 1, 6 do
        local shadow = Instance.new(\"Part\", workspace)
        shadow.Size = Vector3.new(4, 8, 0.5)
        shadow.CFrame = root.CFrame * CFrame.new(
            math.random(-10, 10),
            0,
            math.random(-10, 10)
        ) * CFrame.Angles(0, math.random() * math.pi * 2, 0)
        shadow.Color = Color3.fromRGB(20, 0, 0)
        shadow.Material = Enum.Material.Neon
        shadow.Transparency = 0.3
        shadow.Anchored = true
        shadow.CanCollide = false
        
        CreateParticleEffect(shadow, Color3.fromRGB(100, 0, 0), 1.5)
        
        TweenService:Create(shadow, TweenInfo.new(3), {
            Transparency = 1,
            Size = Vector3.new(6, 10, 0.5)
        }):Play()
        
        Debris:AddItem(shadow, 3)
    end
    
    PlaySound(9114835335, 0.6)
end)

-- 21. Possession (Darkhold)
CreatePowerButton(\"Controle Mental\", \"Dark Possession\", \"Possessão através do Darkhold\", 7, function()
    local root = GetCharParts()
    if not root then return end
    
    local portal = Instance.new(\"Part\", workspace)
    portal.Size = Vector3.new(8, 10, 0.1)
    portal.CFrame = CFrame.new(Mouse.Hit.p) * CFrame.Angles(0, 0, 0)
    portal.Color = Color3.fromRGB(80, 0, 80)
    portal.Material = Enum.Material.Neon
    portal.Transparency = 0.3
    portal.Anchored = true
    portal.CanCollide = false
    
    CreateAura(portal, Color3.fromRGB(100, 0, 100), 3)
    PlaySound(9125862175, 0.7)
    
    -- Criar tentáculos escuros
    for i = 1, 4 do
        delay(i * 0.3, function()
            local tentacle = Instance.new(\"Part\", workspace)
            tentacle.Size = Vector3.new(1, 15, 1)
            tentacle.CFrame = portal.CFrame * CFrame.new(
                math.random(-3, 3),
                0,
                math.random(-3, 3)
            )
            tentacle.Color = Color3.fromRGB(60, 0, 60)
            tentacle.Material = Enum.Material.Neon
            tentacle.Anchored = true
            tentacle.CanCollide = false
            
            TweenService:Create(tentacle, TweenInfo.new(2), {
                Transparency = 1,
                Size = Vector3.new(1, 20, 1)
            }):Play()
            
            Debris:AddItem(tentacle, 2)
        end)
    end
    
    Debris:AddItem(portal, 4)
end)

-- 22. Hypnotic Gaze (Comics)
CreatePowerButton(\"Controle Mental\", \"Hypnotic Gaze\", \"Olhar hipnótico paralisante\", 3, function()
    local root = GetCharParts()
    if not root then return end
    
    -- Criar raio visual
    local gaze = Instance.new(\"Part\", workspace)
    gaze.Size = Vector3.new(2, 2, 50)
    gaze.CFrame = CFrame.new(root.Position, Mouse.Hit.p) * CFrame.new(0, 0, -25)
    gaze.Color = Color3.fromRGB(255, 150, 150)
    gaze.Material = Enum.Material.Neon
    gaze.Transparency = 0.5
    gaze.Anchored = true
    gaze.CanCollide = false
    
    CreateParticleEffect(gaze, Color3.fromRGB(255, 150, 150), 1)
    
    delay(1.5, function()
        TweenService:Create(gaze, TweenInfo.new(0.5), {Transparency = 1}):Play()
        Debris:AddItem(gaze, 0.5)
    end)
end)

--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
--  🌀 CATEGORIA: REALIDADE
--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

-- 23. Reality Warp (WandaVision)
CreatePowerButton(\"Realidade\", \"Reality Warp\", \"Distorce a realidade local\", 5, function()
    local platform = Instance.new(\"Part\", workspace)
    platform.Size = Vector3.new(25, 1, 25)
    platform.CFrame = CFrame.new(Mouse.Hit.p)
    platform.Color = GetColor()
    platform.Material = Enum.Material.Neon
    platform.Anchored = true
    platform.CanCollide = true
    
    CreateAura(platform, GetColor(), 3)
    PlaySound(9114997626, 0.6)
    
    -- Criar objetos flutuantes
    for i = 1, 8 do
        delay(i * 0.2, function()
            local obj = Instance.new(\"Part\", workspace)
            obj.Size = Vector3.new(2, 2, 2)
            obj.CFrame = platform.CFrame * CFrame.new(
                math.random(-12, 12),
                math.random(3, 8),
                math.random(-12, 12)
            )
            obj.Color = GetSecondary()
            obj.Material = Enum.Material.Neon
            obj.Anchored = true
            obj.CanCollide = false
            
            TweenService:Create(obj, TweenInfo.new(2, Enum.EasingStyle.Sine), {
                CFrame = obj.CFrame * CFrame.new(0, 5, 0),
                Orientation = Vector3.new(math.random(0, 360), math.random(0, 360), math.random(0, 360))
            }):Play()
            
            Debris:AddItem(obj, 5)
        end)
    end
    
    Debris:AddItem(platform, 8)
end)

-- 24. Westview Hex (WandaVision)
CreatePowerButton(\"Realidade\", \"Westview Hex\", \"Cria uma bolha de realidade alterada\", 8, function()
    local root = GetCharParts()
    if not root then return end
    
    local hex = Instance.new(\"Part\", workspace)
    hex.Shape = Enum.PartType.Ball
    hex.Size = Vector3.new(10, 10, 10)
    hex.CFrame = root.CFrame
    hex.Color = GetColor()
    hex.Material = Enum.Material.ForceField
    hex.Transparency = 0.4
    hex.Anchored = true
    hex.CanCollide = false
    
    CreateAura(hex, GetColor(), 4)
    PlaySound(9116539222, 0.8)
    
    -- Expandir hex
    TweenService:Create(hex, TweenInfo.new(2, Enum.EasingStyle.Quad), {
        Size = Vector3.new(60, 60, 60)
    }):Play()
    
    -- Criar padrão hexagonal
    for i = 1, 20 do
        delay(i * 0.15, function()
            if hex.Parent then
                local hexPattern = Instance.new(\"Part\", workspace)
                hexPattern.Size = Vector3.new(3, 3, 0.2)
                hexPattern.CFrame = hex.CFrame * CFrame.new(
                    math.random(-30, 30),
                    math.random(-30, 30),
                    math.random(-30, 30)
                )
                hexPattern.Color = GetSecondary()
                hexPattern.Material = Enum.Material.Neon
                hexPattern.Transparency = 0.6
                hexPattern.Anchored = true
                hexPattern.CanCollide = false
                
                TweenService:Create(hexPattern, TweenInfo.new(1), {
                    Transparency = 1,
                    Size = Vector3.new(5, 5, 0.2)
                }):Play()
                
                Debris:AddItem(hexPattern, 1)
            end
        end)
    end
    
    Debris:AddItem(hex, 12)
end)

-- 25. Transmutation (Comics)
CreatePowerButton(\"Realidade\", \"Transmutation\", \"Transforma matéria\", 4, function()
    local target = Mouse.Target
    if target and not target.Anchored and target.Parent ~= Character then
        CreateParticleEffect(target, GetColor(), 2)
        PlaySound(9113976209, 0.5)
        
        local originalColor = target.Color
        local originalMaterial = target.Material
        
        -- Transformar
        TweenService:Create(target, TweenInfo.new(1), {
            Color = GetColor(),
            Material = Enum.Material.Neon
        }):Play()
        
        delay(5, function()
            TweenService:Create(target, TweenInfo.new(1), {
                Color = originalColor,
                Material = originalMaterial
            }):Play()
        end)
    end
end)

-- 26. Pocket Dimension (Comics)
CreatePowerButton(\"Realidade\", \"Pocket Dimension\", \"Cria dimensão de bolso\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    local dimension = Instance.new(\"Part\", workspace)
    dimension.Shape = Enum.PartType.Ball
    dimension.Size = Vector3.new(45, 45, 45)
    dimension.CFrame = root.CFrame
    dimension.Color = Color3.fromRGB(100, 0, 100)
    dimension.Material = Enum.Material.Glass
    dimension.Transparency = 0.6
    dimension.Anchored = true
    dimension.CanCollide = false
    
    CreateAura(dimension, Color3.fromRGB(150, 0, 150), 5)
    PlaySound(9125428387, 0.7)
    
    -- Criar estrelas dentro
    for i = 1, 30 do
        local star = Instance.new(\"Part\", dimension)
        star.Size = Vector3.new(0.5, 0.5, 0.5)
        star.CFrame = dimension.CFrame * CFrame.new(
            math.random(-20, 20),
            math.random(-20, 20),
            math.random(-20, 20)
        )
        star.Color = Color3.fromRGB(255, 255, 255)
        star.Material = Enum.Material.Neon
        star.Shape = Enum.PartType.Ball
        star.Anchored = true
        star.CanCollide = false
        
        -- Fazer estrelas brilharem
        spawn(function()
            while star.Parent do
                TweenService:Create(star, TweenInfo.new(0.5), {
                    Transparency = math.random(0, 7) / 10
                }):Play()
                wait(0.5)
            end
        end)
    end
    
    Debris:AddItem(dimension, 10)
end)

-- 27. No More Mutants (House of M)
CreatePowerButton(\"Realidade\", \"Reality Rewrite\", \"Reescreve a realidade local\", 10, function()
    local root = GetCharParts()
    if not root then return end
    
    PlaySound(9116890699, 1)
    
    -- Criar onda de realidade
    for radius = 10, 100, 10 do
        delay(radius / 50, function()
            local wave = Instance.new(\"Part\", workspace)
            wave.Shape = Enum.PartType.Ball
            wave.Size = Vector3.new(radius, radius, radius)
            wave.CFrame = root.CFrame
            wave.Color = GetColor()
            wave.Material = Enum.Material.Neon
            wave.Transparency = 0.7
            wave.Anchored = true
            wave.CanCollide = false
            
            TweenService:Create(wave, TweenInfo.new(0.5), {
                Transparency = 1,
                Size = Vector3.new(radius + 10, radius + 10, radius + 10)
            }):Play()
            
            Debris:AddItem(wave, 0.5)
        end)
    end
    
    -- Efeito de glitch na tela
    local glitchGui = Instance.new(\"ScreenGui\", LocalPlayer.PlayerGui)
    local glitchFrame = Instance.new(\"Frame\", glitchGui)
    glitchFrame.Size = UDim2.new(1, 0, 1, 0)
    glitchFrame.BackgroundColor3 = GetColor()
    glitchFrame.BackgroundTransparency = 0.5
    glitchFrame.BorderSizePixel = 0
    
    for i = 1, 10 do
        delay(i * 0.1, function()
            glitchFrame.BackgroundTransparency = math.random(3, 9) / 10
        end)
    end
    
    delay(1.5, function()
        glitchGui:Destroy()
    end)
end)

-- 28. Chaos Domain (Comics)
CreatePowerButton(\"Realidade\", \"Chaos Domain\", \"Domínio do caos absoluto\", 7, function()
    local root = GetCharParts()
    if not root then return end
    
    local domain = Instance.new(\"Part\", workspace)
    domain.Shape = Enum.PartType.Ball
    domain.Size = Vector3.new(50, 50, 50)
    domain.CFrame = root.CFrame
    domain.Color = GetColor()
    domain.Material = Enum.Material.ForceField
    domain.Transparency = 0.5
    domain.Anchored = true
    domain.CanCollide = false
    
    CreateAura(domain, GetColor(), 6)
    PlaySound(9114608326, 0.8)
    
    -- Criar fragmentos caóticos
    for i = 1, 40 do
        delay(i * 0.1, function()
            if domain.Parent then
                local fragment = Instance.new(\"Part\", workspace)
                fragment.Size = Vector3.new(
                    math.random(1, 4),
                    math.random(1, 4),
                    math.random(1, 4)
                )
                fragment.CFrame = domain.CFrame * CFrame.new(
                    math.random(-25, 25),
                    math.random(-25, 25),
                    math.random(-25, 25)
                )
                fragment.Color = GetSecondary()
                fragment.Material = Enum.Material.Neon
                fragment.Anchored = true
                fragment.CanCollide = false
                
                -- Rotação caótica
                local spin = TweenService:Create(fragment, TweenInfo.new(2, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1), {
                    Orientation = Vector3.new(360, 360, 360)
                })
                spin:Play()
                
                TweenService:Create(fragment, TweenInfo.new(3), {
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(fragment, 3)
            end
        end)
    end
    
    Debris:AddItem(domain, 15)
end)

-- 29. Sitcom Reality (WandaVision)
CreatePowerButton(\"Realidade\", \"Sitcom Reality\", \"Transforma área em cenário vintage\", 5, function()
    local area = Instance.new(\"Part\", workspace)
    area.Size = Vector3.new(40, 1, 40)
    area.CFrame = CFrame.new(Mouse.Hit.p)
    area.Color = Color3.fromRGB(200, 200, 200)
    area.Material = Enum.Material.SmoothPlastic
    area.Anchored = true
    area.CanCollide = true
    
    -- Criar \"paredes\" de TV
    local wall1 = Instance.new(\"Part\", workspace)
    wall1.Size = Vector3.new(40, 20, 1)
    wall1.CFrame = area.CFrame * CFrame.new(0, 10, -20)
    wall1.Color = Color3.fromRGB(220, 220, 220)
    wall1.Material = Enum.Material.SmoothPlastic
    wall1.Anchored = true
    
    local wall2 = wall1:Clone()
    wall2.Parent = workspace
    wall2.CFrame = area.CFrame * CFrame.new(0, 10, 20)
    
    -- Efeito de TV antiga
    for i = 1, 15 do
        delay(i * 0.3, function()
            if area.Parent then
                local static = Instance.new(\"Part\", workspace)
                static.Size = Vector3.new(math.random(2, 5), math.random(2, 5), 0.1)
                static.CFrame = wall1.CFrame * CFrame.new(
                    math.random(-18, 18),
                    math.random(-8, 8),
                    1
                )
                static.Color = Color3.fromRGB(math.random(200, 255), math.random(200, 255), math.random(200, 255))
                static.Material = Enum.Material.Neon
                static.Anchored = true
                static.CanCollide = false
                
                TweenService:Create(static, TweenInfo.new(0.2), {Transparency = 1}):Play()
                Debris:AddItem(static, 0.2)
            end
        end)
    end
    
    PlaySound(9118277872, 0.5)
    
    Debris:AddItem(area, 10)
    Debris:AddItem(wall1, 10)
    Debris:AddItem(wall2, 10)
end)

--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
--  ✈️ CATEGORIA: MOVIMENTO
--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

local isFlying = false
local isTeleporting = false

-- 30. Scarlet Flight (Age of Ultron)
CreatePowerButton(\"Movimento\", \"Scarlet Flight\", \"Ativa/Desativa voo\", 0, function()
    isFlying = not isFlying
    local root = GetCharParts()
    if not root then return end
    
    if isFlying then
        local bv = Instance.new(\"BodyVelocity\", root)
        bv.Name = \"WandaFlight\"
        bv.Velocity = Vector3.new(0, 0, 0)
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        
        local aura = CreateAura(root, GetColor(), 3)
        aura.Name = \"FlightAura\"
        
        PlaySound(9114090595, 0.3)
        
        RunService:BindToRenderStep(\"WandaFlight\", Enum.RenderPriority.Character.Value, function()
            if isFlying and root:FindFirstChild(\"WandaFlight\") then
                local moveDir = Vector3.new()
                
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then
                    moveDir = moveDir + workspace.CurrentCamera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then
                    moveDir = moveDir - workspace.CurrentCamera.CFrame.LookVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then
                    moveDir = moveDir - workspace.CurrentCamera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then
                    moveDir = moveDir + workspace.CurrentCamera.CFrame.RightVector
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
                    moveDir = moveDir + Vector3.new(0, 1, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
                    moveDir = moveDir - Vector3.new(0, 1, 0)
                end
                
                bv.Velocity = moveDir.Unit * 80
            end
        end)
    else
        RunService:UnbindFromRenderStep(\"WandaFlight\")
        if root:FindFirstChild(\"WandaFlight\") then 
            root.WandaFlight:Destroy() 
        end
        if root:FindFirstChild(\"FlightAura\") then 
            root.FlightAura:Destroy() 
        end
    end
end)

-- 31. Teleportation (Comics)
CreatePowerButton(\"Movimento\", \"Chaos Teleport\", \"Teletransporte instantâneo\", 2, function()
    if isTeleporting then return end
    isTeleporting = true
    
    local root = GetCharParts()
    if not root then 
        isTeleporting = false
        return 
    end
    
    local startPos = root.CFrame
    local endPos = Mouse.Hit + Vector3.new(0, 3, 0)
    
    -- Efeito de saída
    local exitPortal = Instance.new(\"Part\", workspace)
    exitPortal.Shape = Enum.PartType.Cylinder
    exitPortal.Size = Vector3.new(1, 8, 8)
    exitPortal.CFrame = startPos * CFrame.Angles(0, 0, math.pi/2)
    exitPortal.Color = GetColor()
    exitPortal.Material = Enum.Material.Neon
    exitPortal.Anchored = true
    exitPortal.CanCollide = false
    
    CreateParticleEffect(exitPortal, GetColor(), 1)
    PlaySound(9119301044, 0.5)
    
    -- Efeito de entrada
    local entryPortal = exitPortal:Clone()
    entryPortal.Parent = workspace
    entryPortal.CFrame = CFrame.new(endPos.p) * CFrame.Angles(0, 0, math.pi/2)
    
    CreateParticleEffect(entryPortal, GetColor(), 1)
    
    -- Teleportar
    delay(0.3, function()
        root.CFrame = CFrame.new(endPos.p)
        
        Debris:AddItem(exitPortal, 1)
        Debris:AddItem(entryPortal, 1)
        
        wait(0.5)
        isTeleporting = false
    end)
end)

-- 32. Speed Enhancement (Comics)
CreatePowerButton(\"Movimento\", \"Speed Boost\", \"Aumenta velocidade temporariamente\", 4, function()
    local root, humanoid = GetCharParts()
    if not root or not humanoid then return end
    
    local originalSpeed = humanoid.WalkSpeed
    humanoid.WalkSpeed = originalSpeed * 2.5
    
    local trail = CreateAura(root, GetColor(), 2)
    trail.Name = \"SpeedTrail\"
    
    PlaySound(9114178935, 0.4)
    
    delay(5, function()
        humanoid.WalkSpeed = originalSpeed
        if trail.Parent then
            trail:Destroy()
        end
    end)
end)

-- 33. Levitation (WandaVision)
CreatePowerButton(\"Movimento\", \"Self Levitation\", \"Levita suavemente\", 1, function()
    local root = GetCharParts()
    if not root then return end
    
    if not root:FindFirstChild(\"LevitationBP\") then
        local bp = Instance.new(\"BodyPosition\", root)
        bp.Name = \"LevitationBP\"
        bp.Position = root.Position + Vector3.new(0, 10, 0)
        bp.MaxForce = Vector3.new(1e6, 1e6, 1e6)
        bp.D = 500
        bp.P = 5000
        
        local aura = CreateAura(root, GetColor(), 2)
        aura.Name = \"LevitationAura\"
        
        delay(5, function()
            bp.Position = root.Position
            wait(1)
            if bp.Parent then bp:Destroy() end
            if aura.Parent then aura:Destroy() end
        end)
    end
end)

-- 34. Dimensional Portal (Multiverse of Madness)
CreatePowerButton(\"Movimento\", \"Dimensional Rift\", \"Abre portal dimensional\", 5, function()
    local root = GetCharParts()
    if not root then return end
    
    local portal1 = Instance.new(\"Part\", workspace)
    portal1.Size = Vector3.new(8, 12, 0.5)
    portal1.CFrame = root.CFrame * CFrame.new(0, 0, -5)
    portal1.Color = Color3.fromRGB(100, 0, 150)
    portal1.Material = Enum.Material.Neon
    portal1.Transparency = 0.3
    portal1.Anchored = true
    portal1.CanCollide = false
    
    local portal2 = portal1:Clone()
    portal2.Parent = workspace
    portal2.CFrame = CFrame.new(Mouse.Hit.p) * CFrame.Angles(0, math.pi, 0)
    
    CreateAura(portal1, Color3.fromRGB(150, 0, 200), 3)
    CreateAura(portal2, Color3.fromRGB(150, 0, 200), 3)
    
    PlaySound(9125559515, 0.7)
    
    -- Efeito de borda portal
    for i = 1, 20 do
        delay(i * 0.1, function()
            if portal1.Parent then
                local spark = Instance.new(\"Part\", workspace)
                spark.Size = Vector3.new(0.5, 0.5, 0.5)
                spark.CFrame = portal1.CFrame * CFrame.new(
                    math.random(-4, 4),
                    math.random(-6, 6),
                    0
                )
                spark.Color = Color3.fromRGB(200, 100, 255)
                spark.Material = Enum.Material.Neon
                spark.Shape = Enum.PartType.Ball
                spark.Anchored = true
                spark.CanCollide = false
                
                TweenService:Create(spark, TweenInfo.new(0.3), {
                    Transparency = 1,
                    Size = Vector3.new(1.5, 1.5, 1.5)
                }):Play()
                
                Debris:AddItem(spark, 0.3)
            end
        end)
    end
    
    Debris:AddItem(portal1, 8)
    Debris:AddItem(portal2, 8)
end)

--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
--  ⭐ CATEGORIA: ESPECIAL
--━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

-- 35. Hex Telekinesis (Age of Ultron)
CreatePowerButton(\"Especial\", \"Hex Telekinesis\", \"Controla objetos com a mente\", 2, function()
    local target = Mouse.Target
    if target and not target.Anchored and target.Parent ~= Character then
        CreateParticleEffect(target, GetColor(), 2)
        PlaySound(9113545666, 0.4)
        
        local bp = Instance.new(\"BodyPosition\", target)
        bp.Position = target.Position + Vector3.new(0, 15, 0)
        bp.MaxForce = Vector3.new(1e6, 1e6, 1e6)
        bp.D = 500
        
        -- Criar aura telecinética
        local aura = CreateAura(target, GetColor(), 2)
        
        delay(2, function()
            bp.Position = Mouse.Hit.p
            wait(0.5)
            bp:Destroy()
            aura:Destroy()
        end)
    end
end)

-- 36. Astral Projection (Comics)
CreatePowerButton(\"Especial\", \"Astral Form\", \"Projeta forma astral\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    -- Criar clone astral
    local astral = Instance.new(\"Part\", workspace)
    astral.Size = Vector3.new(2, 5, 1)
    astral.CFrame = root.CFrame + Vector3.new(5, 0, 0)
    astral.Color = GetColor()
    astral.Material = Enum.Material.Neon
    astral.Transparency = 0.5
    astral.Anchored = true
    astral.CanCollide = false
    
    CreateAura(astral, GetColor(), 3)
    PlaySound(9116235074, 0.5)
    
    -- Criar \"cabeça\" astral
    local head = Instance.new(\"Part\", astral)
    head.Size = Vector3.new(1.5, 1.5, 1.5)
    head.Shape = Enum.PartType.Ball
    head.CFrame = astral.CFrame + Vector3.new(0, 3, 0)
    head.Color = GetColor()
    head.Material = Enum.Material.Neon
    head.Transparency = 0.5
    head.Anchored = true
    head.CanCollide = false
    
    -- Fazer forma astral flutuar
    local float = TweenService:Create(astral, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
        CFrame = astral.CFrame + Vector3.new(0, 3, 0)
    })
    float:Play()
    
    Debris:AddItem(astral, 8)
end)

-- 37. Rune Casting (WandaVision)
CreatePowerButton(\"Especial\", \"Mystic Runes\", \"Cria runas místicas protetoras\", 5, function()
    local root = GetCharParts()
    if not root then return end
    
    PlaySound(9114456108, 0.6)
    
    -- Criar 6 runas em círculo
    for i = 1, 6 do
        local angle = (i / 6) * math.pi * 2
        local rune = Instance.new(\"Part\", workspace)
        rune.Size = Vector3.new(4, 4, 0.2)
        rune.CFrame = root.CFrame * CFrame.new(
            math.cos(angle) * 10,
            5,
            math.sin(angle) * 10
        ) * CFrame.Angles(0, -angle + math.pi/2, 0)
        rune.Color = GetColor()
        rune.Material = Enum.Material.Neon
        rune.Transparency = 0.3
        rune.Anchored = true
        rune.CanCollide = false
        
        -- Adicionar símbolo de runa
        local decal = Instance.new(\"Decal\", rune)
        decal.Texture = \"rbxassetid://6847005813\"
        decal.Face = Enum.NormalId.Front
        
        CreateAura(rune, GetColor(), 2)
        
        -- Rotação constante
        local spin = TweenService:Create(rune, TweenInfo.new(4, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1), {
            Orientation = rune.Orientation + Vector3.new(0, 360, 0)
        })
        spin:Play()
        
        Debris:AddItem(rune, 10)
    end
end)

-- 38. Life Force Absorption (Darkhold)
CreatePowerButton(\"Especial\", \"Life Drain\", \"Drena energia vital\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    -- Criar ondas de drenagem
    for i = 1, 10 do
        delay(i * 0.2, function()
            local drain = Instance.new(\"Part\", workspace)
            drain.Shape = Enum.PartType.Ball
            drain.Size = Vector3.new(30, 30, 30)
            drain.CFrame = root.CFrame
            drain.Color = Color3.fromRGB(100, 0, 0)
            drain.Material = Enum.Material.Neon
            drain.Transparency = 0.8
            drain.Anchored = true
            drain.CanCollide = false
            
            -- Partículas sendo absorvidas
            local absorb = CreateAura(drain, Color3.fromRGB(255, 0, 0), 2)
            absorb.Speed = NumberRange.new(-15)
            
            TweenService:Create(drain, TweenInfo.new(0.5), {
                Size = Vector3.new(5, 5, 5),
                Transparency = 1
            }):Play()
            
            Debris:AddItem(drain, 0.5)
        end)
    end
    
    PlaySound(9116688583, 0.7)
end)

-- 39. Darkhold Corruption (Multiverse of Madness)
CreatePowerButton(\"Especial\", \"Darkhold Power\", \"Libera poder corrompido do Darkhold\", 8, function()
    local root = GetCharParts()
    if not root then return end
    
    -- Criar livro Darkhold visual
    local book = Instance.new(\"Part\", workspace)
    book.Size = Vector3.new(3, 4, 0.5)
    book.CFrame = root.CFrame + Vector3.new(0, 8, 0)
    book.Color = Color3.fromRGB(60, 0, 60)
    book.Material = Enum.Material.Neon
    book.Anchored = true
    book.CanCollide = false
    
    CreateAura(book, Color3.fromRGB(100, 0, 100), 4)
    PlaySound(9125988817, 0.9)
    
    -- Fazer livro girar e liberar energia
    local spin = TweenService:Create(book, TweenInfo.new(3, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1), {
        Orientation = Vector3.new(0, 360, 0)
    })
    spin:Play()
    
    -- Liberar raios corrompidos
    for i = 1, 15 do
        delay(i * 0.2, function()
            if book.Parent then
                local ray = Instance.new(\"Part\", workspace)
                ray.Size = Vector3.new(1, 1, 20)
                ray.CFrame = book.CFrame * CFrame.Angles(
                    math.random() * math.pi,
                    math.random() * math.pi,
                    math.random() * math.pi
                )
                ray.Color = Color3.fromRGB(80, 0, 80)
                ray.Material = Enum.Material.Neon
                ray.Anchored = true
                ray.CanCollide = false
                
                CreateParticleEffect(ray, Color3.fromRGB(120, 0, 120), 1)
                
                TweenService:Create(ray, TweenInfo.new(1), {
                    Size = Vector3.new(1, 1, 40),
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(ray, 1)
            end
        end)
    end
    
    Debris:AddItem(book, 6)
end)

-- 40. Scarlet Witch Crown (Full Power)
CreatePowerButton(\"Especial\", \"Scarlet Crown\", \"Manifesta a coroa da Bruxa Escarlate\", 7, function()
    local root = GetCharParts()
    if not root then return end
    
    local head = Character:FindFirstChild(\"Head\")
    if not head then return end
    
    -- Criar coroa
    local crown = Instance.new(\"Part\", Character)
    crown.Size = Vector3.new(2, 0.5, 2)
    crown.CFrame = head.CFrame + Vector3.new(0, 1, 0)
    crown.Color = GetColor()
    crown.Material = Enum.Material.Neon
    crown.Transparency = 0.3
    crown.CanCollide = false
    
    -- Fazer coroa seguir cabeça
    local weld = Instance.new(\"Weld\", crown)
    weld.Part0 = head
    weld.Part1 = crown
    weld.C0 = CFrame.new(0, 1, 0)
    
    -- Criar pontas da coroa
    for i = 1, 5 do
        local spike = Instance.new(\"Part\", crown)
        spike.Size = Vector3.new(0.3, 1.5, 0.3)
        spike.CFrame = crown.CFrame * CFrame.new((i - 3) * 0.5, 1, 0)
        spike.Color = GetColor()
        spike.Material = Enum.Material.Neon
        spike.CanCollide = false
        
        local spikeWeld = Instance.new(\"Weld\", spike)
        spikeWeld.Part0 = crown
        spikeWeld.Part1 = spike
        spikeWeld.C0 = CFrame.new((i - 3) * 0.5, 1, 0)
        
        CreateAura(spike, GetColor(), 1)
    end
    
    CreateAura(crown, GetColor(), 2)
    PlaySound(9116316451, 0.8)
    
    -- Criar aura poderosa ao redor do corpo
    local powerAura = Instance.new(\"Part\", workspace)
    powerAura.Shape = Enum.PartType.Ball
    powerAura.Size = Vector3.new(15, 15, 15)
    powerAura.CFrame = root.CFrame
    powerAura.Color = GetColor()
    powerAura.Material = Enum.Material.Neon
    powerAura.Transparency = 0.7
    powerAura.Anchored = true
    powerAura.CanCollide = false
    
    -- Fazer aura seguir jogador
    spawn(function()
        while powerAura.Parent and crown.Parent do
            powerAura.CFrame = root.CFrame
            wait()
        end
    end)
    
    delay(15, function()
        TweenService:Create(crown, TweenInfo.new(1), {Transparency = 1}):Play()
        TweenService:Create(powerAura, TweenInfo.new(1), {Transparency = 1}):Play()
        
        for _, child in pairs(crown:GetChildren()) do
            if child:IsA(\"BasePart\") then
                TweenService:Create(child, TweenInfo.new(1), {Transparency = 1}):Play()
            end
        end
        
        wait(1)
        crown:Destroy()
        powerAura:Destroy()
    end)
end)

-- 41. Necromancy (Darkhold)
CreatePowerButton(\"Especial\", \"Raise Dead\", \"Invoca forças necromânticas\", 7, function()
    local root = GetCharParts()
    if not root then return end
    
    PlaySound(9125142495, 0.7)
    
    -- Criar círculo necromântico
    local circle = Instance.new(\"Part\", workspace)
    circle.Size = Vector3.new(25, 0.5, 25)
    circle.CFrame = CFrame.new(Mouse.Hit.p)
    circle.Color = Color3.fromRGB(80, 0, 80)
    circle.Material = Enum.Material.Neon
    circle.Transparency = 0.5
    circle.Anchored = true
    circle.CanCollide = false
    
    -- Adicionar pentagrama
    local decal = Instance.new(\"Decal\", circle)
    decal.Texture = \"rbxassetid://6864086333\"
    decal.Face = Enum.NormalId.Top
    
    CreateAura(circle, Color3.fromRGB(100, 0, 100), 3)
    
    -- Invocar \"almas\"
    for i = 1, 8 do
        delay(i * 0.3, function()
            if circle.Parent then
                local soul = Instance.new(\"Part\", workspace)
                soul.Size = Vector3.new(1, 3, 1)
                soul.CFrame = circle.CFrame * CFrame.new(
                    math.random(-10, 10),
                    -5,
                    math.random(-10, 10)
                )
                soul.Color = Color3.fromRGB(150, 150, 200)
                soul.Material = Enum.Material.Neon
                soul.Transparency = 0.5
                soul.Anchored = true
                soul.CanCollide = false
                
                CreateAura(soul, Color3.fromRGB(200, 200, 255), 2)
                
                -- Fazer alma subir
                TweenService:Create(soul, TweenInfo.new(2), {
                    CFrame = soul.CFrame + Vector3.new(0, 15, 0),
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(soul, 2)
            end
        end)
    end
    
    Debris:AddItem(circle, 6)
end)

-- 42. Power Negation (Comics)
CreatePowerButton(\"Especial\", \"Power Nullify\", \"Anula poderes em área\", 6, function()
    local root = GetCharParts()
    if not root then return end
    
    local nullField = Instance.new(\"Part\", workspace)
    nullField.Shape = Enum.PartType.Ball
    nullField.Size = Vector3.new(35, 35, 35)
    nullField.CFrame = root.CFrame
    nullField.Color = Color3.fromRGB(255, 255, 0)
    nullField.Material = Enum.Material.Neon
    nullField.Transparency = 0.7
    nullField.Anchored = true
    nullField.CanCollide = false
    
    PlaySound(9114746213, 0.6)
    
    -- Criar ondas de anulação
    for i = 1, 8 do
        delay(i * 0.4, function()
            if nullField.Parent then
                local wave = Instance.new(\"Part\", workspace)
                wave.Shape = Enum.PartType.Ball
                wave.Size = Vector3.new(20, 20, 20)
                wave.CFrame = nullField.CFrame
                wave.Color = Color3.fromRGB(255, 255, 100)
                wave.Material = Enum.Material.Neon
                wave.Transparency = 0.8
                wave.Anchored = true
                wave.CanCollide = false
                
                TweenService:Create(wave, TweenInfo.new(0.6), {
                    Size = Vector3.new(40, 40, 40),
                    Transparency = 1
                }):Play()
                
                Debris:AddItem(wave, 0.6)
            end
        end)
    end
    
    Debris:AddItem(nullField, 5)
end)

-- 43. Chaos Embodiment (Ultimate Power)
CreatePowerButton(\"Especial\", \"Chaos Incarnate\", \"Forma final - Encarnação do Caos\", 15, function()
    local root, humanoid = GetCharParts()
    if not root then return end
    
    PlaySound(9125789456, 1)
    
    -- Transformação épica
    local transformation = Instance.new(\"Part\", workspace)
    transformation.Shape = Enum.PartType.Ball
    transformation.Size = Vector3.new(10, 10, 10)
    transformation.CFrame = root.CFrame
    transformation.Color = GetColor()
    transformation.Material = Enum.Material.Neon
    transformation.Transparency = 0.3
    transformation.Anchored = true
    transformation.CanCollide = false
    
    CreateAura(transformation, GetColor(), 8)
    
    -- Expandir explosão de poder
    TweenService:Create(transformation, TweenInfo.new(3, Enum.EasingStyle.Quad), {
        Size = Vector3.new(100, 100, 100),
        Transparency = 1
    }):Play()
    
    -- Criar múltiplas camadas de energia
    for layer = 1, 5 do
        delay(layer * 0.3, function()
            local energyLayer = Instance.new(\"Part\", workspace)
            energyLayer.Shape = Enum.PartType.Ball
            energyLayer.Size = Vector3.new(20, 20, 20)
            energyLayer.CFrame = root.CFrame
            energyLayer.Color = layer % 2 == 0 and GetColor() or GetSecondary()
            energyLayer.Material = Enum.Material.Neon
            energyLayer.Transparency = 0.6
            energyLayer.Anchored = true
            energyLayer.CanCollide = false
            
            TweenService:Create(energyLayer, TweenInfo.new(2), {
                Size = Vector3.new(80, 80, 80),
                Transparency = 1
            }):Play()
            
            Debris:AddItem(energyLayer, 2)
        end)
    end
    
    -- Criar pilares de energia
    for i = 1, 12 do
        delay(i * 0.15, function()
            local angle = (i / 12) * math.pi * 2
            local pillar = Instance.new(\"Part\", workspace)
            pillar.Size = Vector3.new(3, 60, 3)
            pillar.CFrame = root.CFrame * CFrame.new(
                math.cos(angle) * 20,
                0,
                math.sin(angle) * 20
            )
            pillar.Color = GetColor()
            pillar.Material = Enum.Material.Neon
            pillar.Transparency = 0.4
            pillar.Anchored = true
            pillar.CanCollide = false
            
            CreateParticleEffect(pillar, GetColor(), 2)
            
            TweenService:Create(pillar, TweenInfo.new(3), {
                Size = Vector3.new(5, 80, 5),
                Transparency = 1
            }):Play()
            
            Debris:AddItem(pillar, 3)
        end)
    end
    
    -- Criar aura permanente temporária
    local ultimateAura = CreateAura(root, GetColor(), 6)
    ultimateAura.Name = \"UltimateAura\"
    
    -- Aumentar stats temporariamente
    local originalWalkSpeed = humanoid.WalkSpeed
    humanoid.WalkSpeed = originalWalkSpeed * 1.5
    
    delay(20, function()
        humanoid.WalkSpeed = originalWalkSpeed
        if ultimateAura.Parent then
            ultimateAura:Destroy()
        end
    end)
    
    Debris:AddItem(transformation, 3)
end)

--═══════════════════════════════════════════════════════
--  EFEITO CONSTANTE DE FOGO NAS MÃOS
--═══════════════════════════════════════════════════════

local function CreateHandEffects()
    local rightHand, leftHand = GetHands()
    
    for _, hand in pairs({rightHand, leftHand}) do
        if hand and not hand:FindFirstChild(\"WandaMagicFire\") then
            local container = Instance.new(\"Folder\")
            container.Name = \"WandaMagicFire\"
            container.Parent = hand
            
            -- Fogo principal
            local fire = Instance.new(\"Fire\", hand)
            fire.Name = \"MainFire\"
            fire.Size = 4
            fire.Heat = 6
            fire.Color = GetColor()
            fire.SecondaryColor = GetSecondary()
            
            -- Partículas extras
            local particles = CreateAura(hand, GetColor(), 1.5)
            particles.Name = \"HandAura\"
        end
    end
end

-- Criar efeitos iniciais
task.spawn(CreateHandEffects)

-- Recriar quando personagem respawnar
LocalPlayer.CharacterAdded:Connect(function(newChar)
    Character = newChar
    task.wait(1)
    CreateHandEffects()
end)

-- Atualizar cores das mãos quando trocar modo
local originalModeBtn = ModeBtn.MouseButton1Click
ModeBtn.MouseButton1Click:Connect(function()
    task.wait(0.1)
    local rightHand, leftHand = GetHands()
    for _, hand in pairs({rightHand, leftHand}) do
        if hand then
            local fire = hand:FindFirstChild(\"MainFire\")
            if fire then
                fire.Color = GetColor()
                fire.SecondaryColor = GetSecondary()
            end
            local aura = hand:FindFirstChild(\"HandAura\")
            if aura then
                aura.Color = ColorSequence.new(GetColor())
            end
        end
    end
end)
