-- IHades
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()
 
-- ============================================================
-- VARIÁVEIS GLOBAIS
-- ============================================================
-- Glitch (Lógica do Claudinei)
getgenv().GlideEnabled = false
getgenv().GlideSpeed = 350
getgenv().GlideTime = 0.20
local JumpForce = 300
local LastShootTime = 0
 
-- Aimbot (Legit)
getgenv().AimbotEnabled = false
getgenv().AimbotSmoothness = 0.1
getgenv().AimbotFOV = 150
getgenv().AimbotMaxDistance = 500
getgenv().AimbotTargetPart = "Head"

-- svAimbot (Brutal Silent Aim)
getgenv().svAimbotEnabled = false
getgenv().svAimbotTargetPart = "Head"
getgenv().svAimbotFOV = 200
getgenv().svAimbotHitChance = 100
getgenv().svAimbotShowFOV = false
local svTarget = nil
 
-- ESP
getgenv().ESPEnabled = false
getgenv().ESPNames = false
getgenv().ESPDistance = false
getgenv().ESPRGB = true
getgenv().ESPBoxes = false
getgenv().ESPTracers = false
 
-- Fly
local flyActive = false
local flySpeed = 400
local FlyVel = nil
local FlyGyro = nil
 
-- Hitbox
getgenv().HitboxStatus = false
getgenv().HitboxSize = 15
getgenv().HitboxTransparency = 0.9
 
-- Drawing FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Filled = false
FOVCircle.Visible = false

local svFOVCircle = Drawing.new("Circle")
svFOVCircle.Thickness = 1.5
svFOVCircle.Color = Color3.fromRGB(255, 255, 255)
svFOVCircle.Filled = false
svFOVCircle.Visible = false
 
-- ============================================================
-- PALETA E CONFIGURAÇÕES VISUAIS
-- ============================================================
local C = {
    BG          = Color3.fromRGB(10, 10, 10),
    PANEL       = Color3.fromRGB(15, 15, 15),
    SIDEBAR     = Color3.fromRGB(12, 12, 12),
    ACCENT      = Color3.fromRGB(255, 0, 0),
    ACCENT2     = Color3.fromRGB(180, 0, 0),
    ACCENT_DIM  = Color3.fromRGB(60, 0, 0),
    ON          = Color3.fromRGB(255, 0, 0),
    OFF         = Color3.fromRGB(30, 30, 30),
    TEXT        = Color3.fromRGB(255, 255, 255),
    SUBTEXT     = Color3.fromRGB(150, 150, 150),
    BORDER      = Color3.fromRGB(40, 40, 40),
    BORDER_GLOW = Color3.fromRGB(200, 0, 0),
    SLIDER_BG   = Color3.fromRGB(20, 20, 20),
    SLIDER_FG   = Color3.fromRGB(255, 0, 0),
    TOGGLE_KNOB = Color3.fromRGB(255, 255, 255),
    ROW_HOVER   = Color3.fromRGB(40, 40, 40),
    TITLE_BG    = Color3.fromRGB(12, 12, 12),
    DOT_RED     = Color3.fromRGB(255, 0, 0),
    DOT_YEL     = Color3.fromRGB(150, 0, 0),
    DOT_GRN     = Color3.fromRGB(80, 0, 0),
}
 
local MENU = {
    W = 640, H = 480,
    TOGGLE_KEY = Enum.KeyCode.K,
    RADIUS = 14,
}
 
-- ============================================================
-- UTILITÁRIOS
-- ============================================================
local function addCorner(parent, r)
    local c = Instance.new("UICorner", parent)
    c.CornerRadius = UDim.new(0, r or 8)
    return c
end
 
local function addStroke(parent, color, thickness)
    local s = Instance.new("UIStroke", parent)
    s.Color = color or C.BORDER
    s.Thickness = thickness or 1
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    return s
end
 
local function addPadding(parent, top, bottom, left, right)
    local p = Instance.new("UIPadding", parent)
    p.PaddingTop    = UDim.new(0, top    or 0)
    p.PaddingBottom = UDim.new(0, bottom or 0)
    p.PaddingLeft   = UDim.new(0, left   or 0)
    p.PaddingRight  = UDim.new(0, right  or 0)
    return p
end
 
local function newLabel(parent, text, size, color, font, xAlign)
    local l = Instance.new("TextLabel", parent)
    l.BackgroundTransparency = 1
    l.Size = UDim2.new(1, 0, 0, size + 4)
    l.Text = text
    l.TextSize = size
    l.TextColor3 = color or C.TEXT
    l.Font = font or Enum.Font.GothamMedium
    l.TextXAlignment = xAlign or Enum.TextXAlignment.Left
    l.TextTruncate = Enum.TextTruncate.AtEnd
    return l
end
 
-- ============================================================
-- INTERFACE PRINCIPAL
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "wClownMaster"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui
 
local Shadow = Instance.new("ImageLabel", ScreenGui)
Shadow.Name = "Shadow"
Shadow.AnchorPoint = Vector2.new(0.5, 0.5)
Shadow.BackgroundTransparency = 1
Shadow.Size = UDim2.new(0, MENU.W + 100, 0, MENU.H + 100)
Shadow.Position = UDim2.new(0.5, 0, -1.5, 0)
Shadow.Image = "rbxassetid://6014261993"
Shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
Shadow.ImageTransparency = 0.25
Shadow.ScaleType = Enum.ScaleType.Slice
Shadow.SliceCenter = Rect.new(49, 49, 450, 450)
Shadow.Visible = false
 
local ShadowGlow = Instance.new("ImageLabel", ScreenGui)
ShadowGlow.Name = "ShadowGlow"
ShadowGlow.AnchorPoint = Vector2.new(0.5, 0.5)
ShadowGlow.BackgroundTransparency = 1
ShadowGlow.Size = UDim2.new(0, MENU.W + 60, 0, MENU.H + 60)
ShadowGlow.Position = UDim2.new(0.5, 0, -1.5, 0)
ShadowGlow.Image = "rbxassetid://6014261993"
ShadowGlow.ImageColor3 = Color3.fromRGB(255, 0, 0)
ShadowGlow.ImageTransparency = 0.85
ShadowGlow.ScaleType = Enum.ScaleType.Slice
ShadowGlow.SliceCenter = Rect.new(49, 49, 450, 450)
ShadowGlow.Visible = false
 
local Main = Instance.new("Frame", ScreenGui)
Main.Name = "wMainGang"
Main.AnchorPoint = Vector2.new(0.5, 0.5)
Main.Size = UDim2.new(0, MENU.W, 0, MENU.H)
Main.Position = UDim2.new(0.5, 0, -1.5, 0)
Main.BackgroundColor3 = C.BG
Main.BorderSizePixel = 0
Main.Visible = false
addCorner(Main, MENU.RADIUS)
addStroke(Main, C.BORDER, 1)
 
local TitleBar = Instance.new("Frame", Main)
TitleBar.Size = UDim2.new(1, 0, 0, 56)
TitleBar.BackgroundColor3 = C.TITLE_BG
TitleBar.BorderSizePixel = 0
addCorner(TitleBar, MENU.RADIUS)
 
local TitleBarFix = Instance.new("Frame", TitleBar)
TitleBarFix.Size = UDim2.new(1, 0, 0, MENU.RADIUS)
TitleBarFix.Position = UDim2.new(0, 0, 1, -MENU.RADIUS)
TitleBarFix.BackgroundColor3 = C.TITLE_BG
TitleBarFix.BorderSizePixel = 0
 
local TitleLine = Instance.new("Frame", TitleBar)
TitleLine.Size = UDim2.new(1, 0, 0, 2)
TitleLine.Position = UDim2.new(0, 0, 1, -2)
TitleLine.BackgroundColor3 = C.ACCENT
TitleLine.BorderSizePixel = 0
local TitleLineGrad = Instance.new("UIGradient", TitleLine)
TitleLineGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0,   C.ACCENT2),
    ColorSequenceKeypoint.new(0.4, C.ACCENT),
    ColorSequenceKeypoint.new(0.6, C.ACCENT),
    ColorSequenceKeypoint.new(1,   C.ACCENT2),
})
 
local function makeDot(color, posX)
    local d = Instance.new("Frame", TitleBar)
    d.Size = UDim2.new(0, 11, 0, 11)
    d.Position = UDim2.new(0, posX, 0.5, -5)
    d.BackgroundColor3 = color
    d.BorderSizePixel = 0
    addCorner(d, 99)
    return d
end
makeDot(C.DOT_RED, 14)
makeDot(C.DOT_YEL, 30)
makeDot(C.DOT_GRN, 46)
 
local TitleLabel = Instance.new("TextLabel", TitleBar)
TitleLabel.Size = UDim2.new(0, 200, 1, 0)
TitleLabel.Position = UDim2.new(0, 68, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "wClownMaster"
TitleLabel.TextColor3 = C.TEXT
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 15
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
 
local TitleBadge = Instance.new("TextLabel", TitleBar)
TitleBadge.Size = UDim2.new(0, 28, 0, 18)
TitleBadge.Position = UDim2.new(0, 178, 0.5, -9)
TitleBadge.BackgroundColor3 = C.ACCENT_DIM
TitleBadge.BorderSizePixel = 0
TitleBadge.Text = "vBeta"
TitleBadge.TextColor3 = C.ACCENT
TitleBadge.Font = Enum.Font.GothamBold
TitleBadge.TextSize = 10
addCorner(TitleBadge, 4)
addStroke(TitleBadge, C.BORDER_GLOW, 1)
 
local KeyHintFrame = Instance.new("Frame", TitleBar)
KeyHintFrame.Size = UDim2.new(0, 90, 0, 22)
KeyHintFrame.AnchorPoint = Vector2.new(1, 0.5)
KeyHintFrame.Position = UDim2.new(1, -14, 0.5, 0)
KeyHintFrame.BackgroundTransparency = 1
 
local KeyBadge = Instance.new("TextLabel", KeyHintFrame)
KeyBadge.Size = UDim2.new(0, 22, 1, 0)
KeyBadge.Position = UDim2.new(1, -22, 0, 0)
KeyBadge.BackgroundColor3 = C.OFF
KeyBadge.BorderSizePixel = 0
KeyBadge.Text = "K"
KeyBadge.TextColor3 = C.TEXT
KeyBadge.Font = Enum.Font.GothamBold
KeyBadge.TextSize = 10
addCorner(KeyBadge, 4)
addStroke(KeyBadge, C.BORDER, 1)
 
local BodyFrame = Instance.new("Frame", Main)
BodyFrame.Size = UDim2.new(1, -20, 1, -68)
BodyFrame.Position = UDim2.new(0, 10, 0, 62)
BodyFrame.BackgroundTransparency = 1
 
local Sidebar = Instance.new("Frame", BodyFrame)
Sidebar.Size = UDim2.new(0, 136, 1, 0)
Sidebar.BackgroundColor3 = C.SIDEBAR
Sidebar.BorderSizePixel = 0
addCorner(Sidebar, 10)
addStroke(Sidebar, C.BORDER, 1)
 
local SidebarList = Instance.new("UIListLayout", Sidebar)
SidebarList.Padding = UDim.new(0, 3)
SidebarList.HorizontalAlignment = Enum.HorizontalAlignment.Center
SidebarList.VerticalAlignment = Enum.VerticalAlignment.Top
SidebarList.SortOrder = Enum.SortOrder.LayoutOrder
addPadding(Sidebar, 10, 10, 8, 8)
 
local ContentArea = Instance.new("Frame", BodyFrame)
ContentArea.Size = UDim2.new(1, -148, 1, 0)
ContentArea.Position = UDim2.new(0, 148, 0, 0)
ContentArea.BackgroundColor3 = C.PANEL
ContentArea.BorderSizePixel = 0
addCorner(ContentArea, 10)
addStroke(ContentArea, C.BORDER, 1)
 
-- ============================================================
-- SISTEMA DE ABAS
-- ============================================================
local TabContents = {}
local ActiveTab = nil
 
local TAB_ICONS = {
    ["Aimbot"]     = "🎯",
    ["svAimbot"]   = "🔮",
    ["ESP"]        = "👁️",
    ["Fly"]        = "✈️",
    ["Glitch"]     = "⚡",
    ["Otimiz"]     = "⚙️",
}
 
local function createTab(name)
    local Btn = Instance.new("TextButton", Sidebar)
    Btn.Size = UDim2.new(1, 0, 0, 38)
    Btn.BackgroundColor3 = C.SIDEBAR
    Btn.BorderSizePixel = 0
    Btn.AutoButtonColor = false
    addCorner(Btn, 6)
 
    local BtnLayout = Instance.new("Frame", Btn)
    BtnLayout.Size = UDim2.new(1, 0, 1, 0)
    BtnLayout.BackgroundTransparency = 1
 
    local IconLbl = Instance.new("TextLabel", BtnLayout)
    IconLbl.Size = UDim2.new(0, 22, 1, 0)
    IconLbl.Position = UDim2.new(0, 8, 0, 0)
    IconLbl.BackgroundTransparency = 1
    IconLbl.Text = TAB_ICONS[name] or "•"
    IconLbl.TextColor3 = C.SUBTEXT
    IconLbl.Font = Enum.Font.GothamBold
    IconLbl.TextSize = 14
 
    local NameLbl = Instance.new("TextLabel", BtnLayout)
    NameLbl.Size = UDim2.new(1, -34, 1, 0)
    NameLbl.Position = UDim2.new(0, 34, 0, 0)
    NameLbl.BackgroundTransparency = 1
    NameLbl.Text = name
    NameLbl.TextColor3 = C.SUBTEXT
    NameLbl.Font = Enum.Font.GothamMedium
    NameLbl.TextSize = 12
    NameLbl.TextXAlignment = Enum.TextXAlignment.Left
 
    local ActiveBar = Instance.new("Frame", Btn)
    ActiveBar.Size = UDim2.new(0, 3, 0.6, 0)
    ActiveBar.AnchorPoint = Vector2.new(0, 0.5)
    ActiveBar.Position = UDim2.new(0, 0, 0.5, 0)
    ActiveBar.BackgroundColor3 = C.ACCENT
    ActiveBar.BorderSizePixel = 0
    ActiveBar.Visible = false
    addCorner(ActiveBar, 99)
 
    local Scroll = Instance.new("ScrollingFrame", ContentArea)
    Scroll.Size = UDim2.new(1, -8, 1, -16)
    Scroll.Position = UDim2.new(0, 4, 0, 8)
    Scroll.BackgroundTransparency = 1
    Scroll.BorderSizePixel = 0
    Scroll.ScrollBarThickness = 3
    Scroll.ScrollBarImageColor3 = C.ACCENT
    Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    Scroll.Visible = false
    Scroll.ScrollingEnabled = true
 
    local List = Instance.new("UIListLayout", Scroll)
    List.Padding = UDim.new(0, 8)
    List.HorizontalAlignment = Enum.HorizontalAlignment.Center
    List.SortOrder = Enum.SortOrder.LayoutOrder
    addPadding(Scroll, 10, 10, 0, 0)
 
    List:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        Scroll.CanvasSize = UDim2.new(0, 0, 0, List.AbsoluteContentSize.Y + 24)
    end)
 
    TabContents[name] = {Button = Btn, Content = Scroll, Icon = IconLbl, Label = NameLbl, Bar = ActiveBar}
 
    local function activate()
        if ActiveTab and TabContents[ActiveTab] then
            local prev = TabContents[ActiveTab]
            prev.Content.Visible = false
            prev.Bar.Visible = false
            TweenService:Create(prev.Button, TweenInfo.new(0.15), {BackgroundColor3 = C.SIDEBAR}):Play()
            TweenService:Create(prev.Icon, TweenInfo.new(0.15), {TextColor3 = C.SUBTEXT}):Play()
            TweenService:Create(prev.Label, TweenInfo.new(0.15), {TextColor3 = C.SUBTEXT}):Play()
        end
        Scroll.Visible = true
        ActiveBar.Visible = true
        TweenService:Create(Btn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(40, 0, 0)}):Play()
        TweenService:Create(IconLbl, TweenInfo.new(0.15), {TextColor3 = C.ACCENT}):Play()
        TweenService:Create(NameLbl, TweenInfo.new(0.15), {TextColor3 = C.TEXT}):Play()
        ActiveTab = name
    end
 
    Btn.MouseButton1Click:Connect(activate)
end
 
-- ============================================================
-- COMPONENTES: TOGGLE
-- ============================================================
local function createToggle(parent, text, callback, default)
    local state = default or false
    local Row = Instance.new("Frame", parent)
    Row.Size = UDim2.new(0.96, 0, 0, 44)
    Row.BackgroundColor3 = C.OFF
    Row.BorderSizePixel = 0
    addCorner(Row, 8)
    addStroke(Row, C.BORDER, 1)
 
    local Label = Instance.new("TextLabel", Row)
    Label.Size = UDim2.new(1, -60, 1, 0)
    Label.Position = UDim2.new(0, 14, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = C.TEXT
    Label.Font = Enum.Font.GothamMedium
    Label.TextSize = 13
    Label.TextXAlignment = Enum.TextXAlignment.Left
 
    local Track = Instance.new("Frame", Row)
    Track.Size = UDim2.new(0, 38, 0, 20)
    Track.AnchorPoint = Vector2.new(1, 0.5)
    Track.Position = UDim2.new(1, -12, 0.5, 0)
    Track.BackgroundColor3 = state and C.ACCENT or Color3.fromRGB(50, 0, 0)
    Track.BorderSizePixel = 0
    addCorner(Track, 99)
 
    local Knob = Instance.new("Frame", Track)
    Knob.Size = UDim2.new(0, 14, 0, 14)
    Knob.AnchorPoint = Vector2.new(0, 0.5)
    Knob.Position = state and UDim2.new(1, -17, 0.5, 0) or UDim2.new(0, 3, 0.5, 0)
    Knob.BackgroundColor3 = C.TOGGLE_KNOB
    Knob.BorderSizePixel = 0
    addCorner(Knob, 99)
 
    local StatusLbl = Instance.new("TextLabel", Row)
    StatusLbl.Size = UDim2.new(0, 26, 0, 14)
    StatusLbl.AnchorPoint = Vector2.new(1, 0.5)
    StatusLbl.Position = UDim2.new(1, -58, 0.5, 0)
    StatusLbl.BackgroundTransparency = 1
    StatusLbl.Text = state and "ON" or "OFF"
    StatusLbl.TextColor3 = state and C.ACCENT or C.SUBTEXT
    StatusLbl.Font = Enum.Font.GothamBold
    StatusLbl.TextSize = 10
 
    local function update(s, silent)
        state = s
        local targetPos = state and UDim2.new(1, -17, 0.5, 0) or UDim2.new(0, 3, 0.5, 0)
        local targetCol = state and C.ACCENT or Color3.fromRGB(50, 0, 0)
        TweenService:Create(Knob, TweenInfo.new(0.2), {Position = targetPos}):Play()
        TweenService:Create(Track, TweenInfo.new(0.2), {BackgroundColor3 = targetCol}):Play()
        StatusLbl.Text = state and "ON" or "OFF"
        StatusLbl.TextColor3 = state and C.ACCENT or C.SUBTEXT
        if not silent then callback(state) end
    end
 
    local Btn = Instance.new("TextButton", Row)
    Btn.Size = UDim2.new(1, 0, 1, 0)
    Btn.BackgroundTransparency = 1
    Btn.Text = ""
    Btn.MouseButton1Click:Connect(function() update(not state) end)
 
    return Row, update
end
 
-- ============================================================
-- COMPONENTES: SLIDER
-- ============================================================
local function createSlider(parent, text, min, max, default, callback)
    local currentVal = default
    local Wrapper = Instance.new("Frame", parent)
    Wrapper.Size = UDim2.new(0.96, 0, 0, 60)
    Wrapper.BackgroundColor3 = C.OFF
    Wrapper.BorderSizePixel = 0
    addCorner(Wrapper, 8)
    addStroke(Wrapper, C.BORDER, 1)
 
    local TopRow = Instance.new("Frame", Wrapper)
    TopRow.Size = UDim2.new(1, -20, 0, 26)
    TopRow.Position = UDim2.new(0, 10, 0, 8)
    TopRow.BackgroundTransparency = 1
 
    local TextLbl = Instance.new("TextLabel", TopRow)
    TextLbl.Size = UDim2.new(1, -100, 1, 0)
    TextLbl.BackgroundTransparency = 1
    TextLbl.Text = text
    TextLbl.TextColor3 = C.TEXT
    TextLbl.Font = Enum.Font.GothamMedium
    TextLbl.TextSize = 12
    TextLbl.TextXAlignment = Enum.TextXAlignment.Left
 
    local ValBox = Instance.new("TextBox", TopRow)
    ValBox.Size = UDim2.new(0, 50, 0, 20)
    ValBox.Position = UDim2.new(1, -50, 0.5, -10)
    ValBox.BackgroundColor3 = C.SLIDER_BG
    ValBox.Text = tostring(default)
    ValBox.TextColor3 = C.ACCENT
    ValBox.Font = Enum.Font.GothamBold
    ValBox.TextSize = 11
    addCorner(ValBox, 4)
    addStroke(ValBox, C.BORDER, 1)
 
    local Track = Instance.new("Frame", Wrapper)
    Track.Size = UDim2.new(1, -80, 0, 4)
    Track.Position = UDim2.new(0, 40, 0, 44)
    Track.BackgroundColor3 = C.SLIDER_BG
    Track.BorderSizePixel = 0
    addCorner(Track, 99)
 
    local Fill = Instance.new("Frame", Track)
    Fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
    Fill.BackgroundColor3 = C.SLIDER_FG
    Fill.BorderSizePixel = 0
    addCorner(Fill, 99)
 
    local Knob = Instance.new("Frame", Track)
    Knob.Size = UDim2.new(0, 12, 0, 12)
    Knob.AnchorPoint = Vector2.new(0.5, 0.5)
    Knob.Position = UDim2.new((default - min) / (max - min), 0, 0.5, 0)
    Knob.BackgroundColor3 = Color3.new(1, 1, 1)
    Knob.BorderSizePixel = 0
    addCorner(Knob, 99)
    addStroke(Knob, C.ACCENT, 1)
 
    local MinusBtn = Instance.new("TextButton", Wrapper)
    MinusBtn.Size = UDim2.new(0, 24, 0, 24)
    MinusBtn.Position = UDim2.new(0, 8, 0, 34)
    MinusBtn.BackgroundColor3 = C.SLIDER_BG
    MinusBtn.Text = "-"
    MinusBtn.TextColor3 = C.TEXT
    MinusBtn.Font = Enum.Font.GothamBold
    MinusBtn.TextSize = 14
    addCorner(MinusBtn, 6)
 
    local PlusBtn = Instance.new("TextButton", Wrapper)
    PlusBtn.Size = UDim2.new(0, 24, 0, 24)
    PlusBtn.Position = UDim2.new(1, -32, 0, 34)
    PlusBtn.BackgroundColor3 = C.SLIDER_BG
    PlusBtn.Text = "+"
    PlusBtn.TextColor3 = C.TEXT
    PlusBtn.Font = Enum.Font.GothamBold
    PlusBtn.TextSize = 14
    addCorner(PlusBtn, 6)
 
    local function updateSlider(ratio)
        ratio = math.clamp(ratio, 0, 1)
        local val = math.floor(min + (max - min) * ratio)
        currentVal = val
        Fill.Size = UDim2.new(ratio, 0, 1, 0)
        Knob.Position = UDim2.new(ratio, 0, 0.5, 0)
        ValBox.Text = tostring(val)
        callback(val)
    end
 
    local dragging = false
    Knob.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
    RunService.RenderStepped:Connect(function()
        if dragging then
            local mouse = UserInputService:GetMouseLocation()
            local absPos = Track.AbsolutePosition
            local absSize = Track.AbsoluteSize
            local ratio = (mouse.X - absPos.X) / absSize.X
            updateSlider(ratio)
        end
    end)
 
    ValBox.FocusLost:Connect(function()
        local v = tonumber(ValBox.Text)
        if v then v = math.clamp(v, min, max) updateSlider((v - min) / (max - min)) else ValBox.Text = tostring(currentVal) end
    end)
 
    MinusBtn.MouseButton1Click:Connect(function() local newVal = math.clamp(currentVal - 1, min, max) updateSlider((newVal - min) / (max - min)) end)
    PlusBtn.MouseButton1Click:Connect(function() local newVal = math.clamp(currentVal + 1, min, max) updateSlider((newVal - min) / (max - min)) end)
end
 
-- ============================================================
-- CONSTRUÇÃO DAS ABAS
-- ============================================================
createTab("Aimbot")
createTab("svAimbot")
createTab("ESP")
createTab("Fly")
createTab("Glitch")
createTab("Otimiz")
 
-- Legitimate Aimbot
createToggle(TabContents["Aimbot"].Content, "Aimbot Ativado", function(v) getgenv().AimbotEnabled = v end, false)
createToggle(TabContents["Aimbot"].Content, "Mostrar FOV", function(v) FOVCircle.Visible = v end, false)
createSlider(TabContents["Aimbot"].Content, "Suavidade", 0, 100, 10, function(v) getgenv().AimbotSmoothness = v / 100 end)
createSlider(TabContents["Aimbot"].Content, "FOV", 10, 800, 150, function(v) getgenv().AimbotFOV = v end)
 
-- Brutal Silent Aim (svAimbot)
createToggle(TabContents["svAimbot"].Content, "svAimbot Brutal", function(v) getgenv().svAimbotEnabled = v end, false)
createToggle(TabContents["svAimbot"].Content, "Mostrar FOV Brutal", function(v) svFOVCircle.Visible = v end, false)
createSlider(TabContents["svAimbot"].Content, "FOV Brutal", 10, 800, 200, function(v) getgenv().svAimbotFOV = v end)
createSlider(TabContents["svAimbot"].Content, "Hit Chance (%)", 0, 100, 100, function(v) getgenv().svAimbotHitChance = v end)
newLabel(TabContents["svAimbot"].Content, "O svAimbot redireciona ataques brutalmente", 12, C.SUBTEXT)
 
-- ESP
createToggle(TabContents["ESP"].Content, "ESP Principal", function(v) getgenv().ESPEnabled = v end, false)
createToggle(TabContents["ESP"].Content, "Mostrar Nomes", function(v) getgenv().ESPNames = v end, false)
createToggle(TabContents["ESP"].Content, "Mostrar Distância", function(v) getgenv().ESPDistance = v end, false)
createToggle(TabContents["ESP"].Content, "Boxes", function(v) getgenv().ESPBoxes = v end, false)
createToggle(TabContents["ESP"].Content, "Tracers", function(v) getgenv().ESPTracers = v end, false)
 
-- Fly
createToggle(TabContents["Fly"].Content, "Modo Fly (H)", function(v)
    flyActive = v
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
    if hum then hum.PlatformStand = v end
end, false)
createSlider(TabContents["Fly"].Content, "Velocidade", 1, 1000, 400, function(v) flySpeed = v end)

-- GLITCH (LÓGICA DO CLAUDINEI)
createToggle(TabContents["Glitch"].Content, "Sistema de Glitch", function(v) getgenv().GlideEnabled = v end, false)
createSlider(TabContents["Glitch"].Content, "Glitch Speed", 1, 1000, 350, function(v) getgenv().GlideSpeed = v end)
createSlider(TabContents["Glitch"].Content, "Glitch Duration", 0, 500, 20, function(v) getgenv().GlideTime = v / 100 end)
createSlider(TabContents["Glitch"].Content, "Jump Force (G)", 1, 1000, 300, function(v) JumpForce = v end)

-- OTIMIZAÇÃO
createToggle(TabContents["Otimiz"].Content, "Remover Névoa (Fog)", function(v)
    if v then
        Lighting.FogEnd = 9e9
        for _, obj in pairs(Lighting:GetChildren()) do
            if obj:IsA("Atmosphere") then obj.Density = 0 end
        end
    else
        Lighting.FogEnd = 1000
        for _, obj in pairs(Lighting:GetChildren()) do
            if obj:IsA("Atmosphere") then obj.Density = 0.3 end
        end
    end
end, false)
createToggle(TabContents["Otimiz"].Content, "Remover Efeitos (Anti-Lag)", function(v)
    if v then 
        Lighting.GlobalShadows = false 
        Lighting.Brightness = 0 
        for _, ef in pairs(Lighting:GetChildren()) do 
            if ef:IsA("PostEffect") or ef:IsA("BloomEffect") or ef:IsA("BlurEffect") then ef.Enabled = false end 
        end 
    else 
        Lighting.GlobalShadows = true 
        Lighting.Brightness = 1 
        for _, ef in pairs(Lighting:GetChildren()) do 
            if ef:IsA("PostEffect") or ef:IsA("BloomEffect") or ef:IsA("BlurEffect") then ef.Enabled = true end 
        end
    end
end, false)
createToggle(TabContents["Otimiz"].Content, "Gráficos Baixos", function(v) if v then for _, obj in pairs(workspace:GetDescendants()) do if obj:IsA("BasePart") then obj.Material = Enum.Material.Plastic obj.Reflectance = 0 end if obj:IsA("Decal") then obj.Transparency = 1 end end end end, false)
 
-- ============================================================
-- LÓGICA DE TARGET
-- ============================================================
local function getClosest(fov)
    local closest, dist = nil, fov
    local mouseLoc = UserInputService:GetMouseLocation()
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("Head") then
            local pos, onScreen = Camera:WorldToViewportPoint(v.Character.Head.Position)
            if onScreen then
                local mag = (Vector2.new(pos.X, pos.Y) - mouseLoc).Magnitude
                if mag < dist then closest = v dist = mag end
            end
        end
    end
    return closest
end

-- ============================================================
-- BRUTAL SILENT AIM HOOK
-- ============================================================
local mt = getrawmetatable(game)
local oldIndex = mt.__index
setreadonly(mt, false)

mt.__index = newcclosure(function(t, k)
    if not checkcaller() and t == Mouse and (k == "Hit" or k == "Target") then
        if getgenv().svAimbotEnabled and svTarget and svTarget.Character and svTarget.Character:FindFirstChild(getgenv().svAimbotTargetPart) then
            if math.random(1, 100) <= getgenv().svAimbotHitChance then
                local part = svTarget.Character[getgenv().svAimbotTargetPart]
                return (k == "Hit" and part.CFrame or part)
            end
        end
    end
    return oldIndex(t, k)
end)
setreadonly(mt, true)
 
-- ============================================================
-- RENDERLOOP (FLY & AIMBOT)
-- ============================================================
RunService.RenderStepped:Connect(function()
    local mouseLoc = UserInputService:GetMouseLocation()
    FOVCircle.Position = mouseLoc
    FOVCircle.Radius = getgenv().AimbotFOV
    
    svFOVCircle.Position = mouseLoc
    svFOVCircle.Radius = getgenv().svAimbotFOV
 
    local myChar = LocalPlayer.Character
    if not myChar then return end
 
    -- Aimbot (Legit)
    if getgenv().AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosest(getgenv().AimbotFOV)
        if target and target.Character:FindFirstChild(getgenv().AimbotTargetPart) then
            local pos = Camera:WorldToViewportPoint(target.Character[getgenv().AimbotTargetPart].Position)
            if typeof(mousemoverel) == "function" then
                mousemoverel((pos.X - mouseLoc.X) * getgenv().AimbotSmoothness, (pos.Y - mouseLoc.Y) * getgenv().AimbotSmoothness)
            end
        end
    end

    -- svAimbot Target Update
    svTarget = getgenv().svAimbotEnabled and getClosest(getgenv().svAimbotFOV) or nil
 
    -- Fly Logic (FIXED)
    if flyActive then
        local root = myChar:FindFirstChild("HumanoidRootPart")
        local hum = myChar:FindFirstChild("Humanoid")
        if root and hum then
            hum.PlatformStand = true
            if not FlyVel or FlyVel.Parent ~= root then 
                if FlyVel then FlyVel:Destroy() end 
                FlyVel = Instance.new("BodyVelocity", root)
                FlyVel.MaxForce = Vector3.new(9e9, 9e9, 9e9) 
            end
            if not FlyGyro or FlyGyro.Parent ~= root then
                if FlyGyro then FlyGyro:Destroy() end
                FlyGyro = Instance.new("BodyGyro", root)
                FlyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
                FlyGyro.P = 3000
            end
            local dir = Vector3.zero
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir += Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir -= Camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir -= Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir += Camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0, 1, 0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0, 1, 0) end
            FlyVel.Velocity = (dir.Magnitude > 0 and dir.Unit * flySpeed) or Vector3.zero
            FlyGyro.CFrame = Camera.CFrame
        end
    else
        if FlyVel then FlyVel:Destroy() FlyVel = nil end
        if FlyGyro then FlyGyro:Destroy() FlyGyro = nil end
        local hum = myChar:FindFirstChild("Humanoid")
        if hum and hum.PlatformStand then hum.PlatformStand = false end
    end
end)

-- ============================================================
-- LÓGICA DE GLITCH (CLAUDINEI)
-- ============================================================
RunService.Heartbeat:Connect(function()
    if not getgenv().GlideEnabled then return end
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") then
        local hrp = char.HumanoidRootPart
        local hum = char.Humanoid
        if (tick() - LastShootTime < 1.5) and hrp.Velocity.Magnitude > 20 then
            if hrp:FindFirstChild("OmniGlide") then hrp.OmniGlide:Destroy() end
            local dir = hum.MoveDirection
            if dir.Magnitude == 0 then
                local look = Camera.CFrame.LookVector
                dir = Vector3.new(look.X, 0, look.Z).Unit
            else
                dir = dir.Unit
            end
            local bv = Instance.new("BodyVelocity")
            bv.Name = "OmniGlide"
            bv.MaxForce = Vector3.new(100000, 0, 100000)
            bv.Velocity = dir * getgenv().GlideSpeed
            bv.Parent = hrp
            Debris:AddItem(bv, getgenv().GlideTime)
            LastShootTime = 0
        end
    end
end)

local function SetupTool(char)
    char.ChildAdded:Connect(function(child)
        if child:IsA("Tool") and (child.Name:find("Soul") or child.Name:find("Guitar")) then
            child.Activated:Connect(function() 
                if getgenv().GlideEnabled then LastShootTime = tick() end 
            end)
        end
    end)
end
if LocalPlayer.Character then SetupTool(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(SetupTool)
 
-- ============================================================
-- ESP SYSTEM (ENHANCED)
-- ============================================================
local function createESP(plr)
    local bg = Instance.new("BillboardGui", ScreenGui) bg.AlwaysOnTop = true bg.Size = UDim2.new(0, 100, 0, 50)
    local lbl = Instance.new("TextLabel", bg) lbl.Size = UDim2.new(1, 0, 1, 0) lbl.BackgroundTransparency = 1 lbl.TextColor3 = Color3.new(1, 0, 0)
    lbl.Font = Enum.Font.GothamBold lbl.TextSize = 11
    
    local Box = Drawing.new("Square")
    Box.Thickness = 1
    Box.Filled = false
    Box.Visible = false
    
    local Tracer = Drawing.new("Line")
    Tracer.Thickness = 1
    Tracer.Visible = false

    RunService.RenderStepped:Connect(function()
        if getgenv().ESPEnabled and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Head") then
            local hrp = plr.Character.HumanoidRootPart
            local head = plr.Character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            local color = getgenv().ESPRGB and Color3.fromHSV(tick() % 5 / 5, 1, 1) or Color3.new(1, 0, 0)
            
            bg.Adornee = head bg.Enabled = true 
            lbl.TextColor3 = color
            local text = "" if getgenv().ESPNames then text = text .. plr.Name .. "\n" end
            if getgenv().ESPDistance and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Head") then text = text .. math.floor((LocalPlayer.Character.Head.Position - head.Position).Magnitude) .. "m" end
            lbl.Text = text
            
            if getgenv().ESPBoxes and onScreen then
                local headPos = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))
                local legPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
                Box.Size = Vector2.new(2000 / pos.Z, headPos.Y - legPos.Y)
                Box.Position = Vector2.new(pos.X - Box.Size.X / 2, pos.Y - Box.Size.Y / 2)
                Box.Color = color
                Box.Visible = true
            else Box.Visible = false end
            
            if getgenv().ESPTracers and onScreen then
                Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                Tracer.To = Vector2.new(pos.X, pos.Y)
                Tracer.Color = color
                Tracer.Visible = true
            else Tracer.Visible = false end
        else bg.Enabled = false Box.Visible = false Tracer.Visible = false end
    end)
end

for _, v in pairs(Players:GetPlayers()) do if v ~= LocalPlayer then createESP(v) end end
Players.PlayerAdded:Connect(function(plr) plr.CharacterAdded:Connect(function() createESP(plr) end) end)
 
-- ============================================================
-- INPUTS
-- ============================================================
local menuOpen = false
local function toggleMenu()
    menuOpen = not menuOpen
    local openPos, closePos = UDim2.new(0.5, 0, 0.5, 0), UDim2.new(0.5, 0, -1.5, 0)
    if menuOpen then 
        Main.Visible = true 
        Shadow.Visible = true 
        ShadowGlow.Visible = true
        TweenService:Create(Main, TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = openPos}):Play() 
        TweenService:Create(Shadow, TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = openPos}):Play() 
        TweenService:Create(ShadowGlow, TweenInfo.new(0.45, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Position = openPos}):Play()
    else 
        TweenService:Create(Main, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {Position = closePos}):Play() 
        TweenService:Create(Shadow, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {Position = closePos}):Play() 
        TweenService:Create(ShadowGlow, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {Position = closePos}):Play()
        task.delay(0.32, function() if not menuOpen then Main.Visible = false Shadow.Visible = false ShadowGlow.Visible = false end end) 
    end
end
 
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == MENU.TOGGLE_KEY then toggleMenu()
    elseif input.KeyCode == Enum.KeyCode.H then 
        flyActive = not flyActive 
        local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid")
        if hum then hum.PlatformStand = flyActive end
    elseif input.KeyCode == Enum.KeyCode.G then 
        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.Velocity = Vector3.new(hrp.Velocity.X, JumpForce, hrp.Velocity.Z)
        end
    end
end)
 
-- Inicialização
if TabContents["Aimbot"] then
    TabContents["Aimbot"].Content.Visible = true
    TabContents["Aimbot"].Bar.Visible = true
    TabContents["Aimbot"].Button.BackgroundColor3 = Color3.fromRGB(40, 0, 0)
    TabContents["Aimbot"].Icon.TextColor3 = C.ACCENT
    TabContents["Aimbot"].Label.TextColor3 = C.TEXT
    ActiveTab = "Aimbot"
end
