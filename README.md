--[[
    🔥 FFH4X - PAINEL PROTEGIDO
    Valida a key e executa o aimbot
]]

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer

-- ============ CONFIG ============
local USUARIO_GITHUB = "socapa123ff2015-netizen"
local GIST_ID_KEYS = "a7db5a897ab877afccb596d3a93e5e39"
local GIST_ID_AIMBOT = "ca730abb33aec6f39f51443c6828c5ac"

local URL_KEYS = "https://gist.githubusercontent.com/" .. USUARIO_GITHUB .. "/" .. GIST_ID_KEYS .. "/raw/keys.json"
local URL_AIMBOT = "https://gist.githubusercontent.com/" .. USUARIO_GITHUB .. "/" .. GIST_ID_AIMBOT .. "/raw/aimbot.lua"

-- ============ GUI DA KEY ============
local guiKey = Instance.new("ScreenGui")
guiKey.Name = "FFH4X_Key"
guiKey.ResetOnSpawn = false
guiKey.IgnoreGuiInset = true
guiKey.DisplayOrder = 100
guiKey.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
guiKey.Parent = player:WaitForChild("PlayerGui")

local frameKey = Instance.new("Frame")
frameKey.Size = UDim2.fromOffset(320, 240)
frameKey.Position = UDim2.new(.5, -160, .5, -120)
frameKey.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
frameKey.BorderSizePixel = 0
frameKey.Active = true
frameKey.Visible = true
frameKey.Parent = guiKey
Instance.new("UICorner", frameKey).CornerRadius = UDim.new(0, 10)

local keyStroke = Instance.new("UIStroke")
keyStroke.Color = Color3.fromRGB(255, 0, 0)
keyStroke.Thickness = 2
keyStroke.Parent = frameKey

local keyTitulo = Instance.new("TextLabel")
keyTitulo.Size = UDim2.new(1, 0, 0, 45)
keyTitulo.BackgroundColor3 = Color3.fromRGB(220, 0, 0)
keyTitulo.Text = "🔥 FFH4X - INSIRA A KEY"
keyTitulo.TextColor3 = Color3.new(1, 1, 1)
keyTitulo.Font = Enum.Font.GothamBold
keyTitulo.TextSize = 15
keyTitulo.Parent = frameKey
Instance.new("UICorner", keyTitulo).CornerRadius = UDim.new(0, 10)

local keyInput = Instance.new("TextBox")
keyInput.Size = UDim2.new(1, -30, 0, 45)
keyInput.Position = UDim2.fromOffset(15, 70)
keyInput.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
keyInput.Text = ""
keyInput.PlaceholderText = "FFH4X-XXXX-XXXX-XXXX"
keyInput.TextColor3 = Color3.new(1, 1, 1)
keyInput.PlaceholderColor3 = Color3.fromRGB(120, 120, 120)
keyInput.Font = Enum.Font.Gotham
keyInput.TextSize = 12
keyInput.ClearTextOnFocus = false
keyInput.Parent = frameKey
Instance.new("UICorner", keyInput).CornerRadius = UDim.new(0, 6)

local btnOK = Instance.new("TextButton")
btnOK.Size = UDim2.new(1, -30, 0, 45)
btnOK.Position = UDim2.fromOffset(15, 125)
btnOK.BackgroundColor3 = Color3.fromRGB(220, 0, 0)
btnOK.Text = "OK"
btnOK.TextColor3 = Color3.new(1, 1, 1)
btnOK.Font = Enum.Font.GothamBold
btnOK.TextSize = 16
btnOK.Parent = frameKey
Instance.new("UICorner", btnOK).CornerRadius = UDim.new(0, 6)

local keyStatus = Instance.new("TextLabel")
keyStatus.Size = UDim2.new(1, -20, 0, 45)
keyStatus.Position = UDim2.fromOffset(10, 180)
keyStatus.BackgroundTransparency = 1
keyStatus.Text = ""
keyStatus.TextColor3 = Color3.fromRGB(255, 80, 80)
keyStatus.Font = Enum.Font.Gotham
keyStatus.TextSize = 11
keyStatus.TextWrapped = true
keyStatus.Parent = frameKey

-- Arrastar
do
    local dragging, dragStart, startPos
    keyTitulo.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frameKey.Position
        end
    end)
    keyTitulo.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            frameKey.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- ============ VALIDAÇÃO ============
local function baixarKeys()
    local ok, resposta = pcall(function()
        return game:HttpGet(URL_KEYS)
    end)
    if not ok then return nil end

    local parseOk, lista = pcall(function()
        return HttpService:JSONDecode(resposta)
    end)
    if not parseOk or not lista then return nil end

    return lista.keys
end

local function baixarAimbot()
    local ok, codigo = pcall(function()
        return game:HttpGet(URL_AIMBOT)
    end)
    if ok then return codigo end
    return nil
end

-- ============ BOTÃO OK ============
btnOK.Activated:Connect(function()
    local key = string.upper(string.gsub(keyInput.Text, "%s", ""))

    if key == "" then
        keyStatus.Text = "❌ Digite uma key!"
        keyStatus.TextColor3 = Color3.fromRGB(255, 80, 80)
        return
    end

    keyStatus.Text = "⏳ Validando..."
    keyStatus.TextColor3 = Color3.fromRGB(255, 200, 0)

    task.spawn(function()
        local keys = baixarKeys()
        if not keys then
            keyStatus.Text = "❌ Sem conexão. Tente de novo."
            keyStatus.TextColor3 = Color3.fromRGB(255, 80, 80)
            return
        end

        local encontrada = false
        for _, entrada in ipairs(keys) do
            if entrada.key == key then
                encontrada = true
                break
            end
        end

        if not encontrada then
            keyStatus.Text = "❌ Key inválida!"
            keyStatus.TextColor3 = Color3.fromRGB(255, 80, 80)
            return
        end

        keyStatus.Text = "✅ Key válida! Carregando painel..."
        keyStatus.TextColor3 = Color3.fromRGB(80, 255, 120)
        task.wait(0.8)

        local codigo = baixarAimbot()
        if not codigo then
            keyStatus.Text = "❌ Erro ao baixar o painel."
            keyStatus.TextColor3 = Color3.fromRGB(255, 80, 80)
            return
        end

        guiKey:Destroy()

        local func, erro = loadstring(codigo)
        if func then
            local sucesso, msg = pcall(func)
            if not sucesso then
                warn("Erro: " .. tostring(msg))
            end
        else
            warn("Erro: " .. tostring(erro))
        end
    end)
end)

print("🔑 Painel FFH4X carregado")
