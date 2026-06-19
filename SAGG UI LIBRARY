--[[
    ╔══════════════════════════════════════════════════════════════════╗
    ║                                                                    ║
    ║   PREMIUM UI LIBRARY  •  Next-Generation Roblox GUI Framework      ║
    ║   Version 1.1.0  •  Merged + Mobile-Optimized Build                ║
    ║                                                                    ║
    ║   Features:                                                         ║
    ║     • Glassmorphism / Acrylic / Fluent Design                      ║
    ║     • Spring-based Animation Engine (60 FPS)                        ║
    ║     • Draggable / Resizable / Minimizable / Maximizable Windows    ║
    ║     • Sidebar Navigation with Animated Indicators                   ║
    ║     • 25+ Polished Components                                       ║
    ║     • Dark / Light / Midnight / Custom Themes (runtime swap)        ║
    ║     • Premium Notification System (queue + stack)                   ║
    ║     • Command Palette (Ctrl + K, fuzzy search)                      ║
    ║     • Mobile + Tablet + Desktop Responsive (multi-touch safe)       ║
    ║     • Keyboard Navigation + Accessibility                           ║
    ║                                                                    ║
    ║   SINGLE-FILE LOAD (Delta executor):                               ║
    ║       loadstring(game:HttpGet("YOUR_URL"))()                       ║
    ║   Skip the auto-demo:                                              ║
    ║       _G.PremiumUI_SkipDemo = true                                 ║
    ║       local Library = loadstring(game:HttpGet("YOUR_URL"))()       ║
    ║                                                                    ║
    ║   Tested on: Delta Executor                                        ║
    ║                                                                    ║
    ╚══════════════════════════════════════════════════════════════════╝
--]]

-- ─── Services ─────────────────────────────────────────────────────────────
local Players           = game:GetService("Players")
local TweenService      = game:GetService("TweenService")
local UserInputService  = game:GetService("UserInputService")
local RunService        = game:GetService("RunService")
local CoreGui           = game:GetService("CoreGui")
local ContextActionService = game:GetService("ContextActionService")
local HttpService       = game:GetService("HttpService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ─── Executor-safe parent ─────────────────────────────────────────────────
local function getProtectedParent()
    if gethui then return gethui() end
    if syn and syn.protect_gui then
        local cg = CoreGui
        syn.protect_gui(cg)
        return cg
    end
    local ok, _ = pcall(function() return CoreGui.Name end)
    if ok then return CoreGui end
    return LocalPlayer:WaitForChild("PlayerGui")
end

-- ─── Math helpers ─────────────────────────────────────────────────────────
local PI = math.pi
local function clamp(v, a, b) return math.max(a, math.min(b, v)) end
local function lerp(a, b, t) return a + (b - a) * t end
local function map(v, a, b, c, d) return c + ((v - a) * (d - c)) / (b - a) end
local function round(v, p) local m = 10 ^ (p or 0) return math.floor(v * m + 0.5) / m end

-- ═══════════════════════════════════════════════════════════════════════════
--   SIGNAL  (lightweight event implementation)
-- ═══════════════════════════════════════════════════════════════════════════
local Signal = {}
Signal.__index = Signal

function Signal.new()
    return setmetatable({ _handlers = {}, _count = 0 }, Signal)
end

function Signal:Connect(fn)
    local id = self._count + 1
    self._count = id
    self._handlers[id] = fn
    local s = self
    return {
        Disconnect = function()
            s._handlers[id] = nil
        end,
        _id = id,
    }
end

function Signal:Fire(...)
    for _, fn in pairs(self._handlers) do
        task.spawn(fn, ...)
    end
end

function Signal:Wait()
    local s = self
    local thread = coroutine.running()
    local c
    c = self:Connect(function(...)
        c:Disconnect()
        task.spawn(thread, ...)
    end)
    return coroutine.yield()
end

function Signal:DisconnectAll()
    self._handlers = {}
end

-- ═══════════════════════════════════════════════════════════════════════════
--   SPRING  (critically-damped spring solver — game-feel motion)
-- ═══════════════════════════════════════════════════════════════════════════
local Spring = {}
Spring.__index = Spring

function Spring.new(initial, stiffness, damping, precision)
    return setmetatable({
        Position = initial or 0,
        Velocity = 0,
        Target   = initial or 0,
        Stiffness = stiffness or 170,
        Damping   = damping   or 16,
        Precision = precision or 0.001,
        _running  = false,
    }, Spring)
end

function Spring:SetTarget(t)
    self.Target = t
    self:_kick()
end

function Spring:SetPosition(p)
    self.Position = p
    self.Velocity = 0
end

function Spring:Impulse(v)
    self.Velocity = self.Velocity + v
    self:_kick()
end

function Spring:_kick()
    if self._running then return end
    self._running = true
    local last = os.clock()
    local conn
    conn = RunService.RenderStepped:Connect(function()
        local now = os.clock()
        local dt = now - last
        last = now
        local s, d, p = self.Stiffness, self.Damping, self.Precision
        local force = -s * (self.Position - self.Target) - d * self.Velocity
        self.Velocity  = self.Velocity + force * dt
        self.Position  = self.Position + self.Velocity * dt
        if math.abs(self.Position - self.Target) < p and math.abs(self.Velocity) < p then
            self.Position = self.Target
            self.Velocity = 0
            self._running = false
            conn:Disconnect()
        end
    end)
end

-- ═══════════════════════════════════════════════════════════════════════════
--   UTILITY  (factory helpers, color helpers, ID generator)
-- ═══════════════════════════════════════════════════════════════════════════
local Utility = {}

function Utility.guid()
    return HttpService:GenerateGUID(false)
end

function Utility.round(v, p) return round(v, p) end
function Utility.clamp(v, a, b) return clamp(v, a, b) end
function Utility.lerp(a, b, t) return lerp(a, b, t) end

function Utility.lerpColor(c1, c2, t)
    return Color3.new(
        lerp(c1.R, c2.R, t),
        lerp(c1.G, c2.G, t),
        lerp(c1.B, c2.B, t)
    )
end

function Utility.colorWithAlpha(c, a)
    return Color3.new(c.R, c.G, c.B)
end

function Utility.transparency(t)
    return 1 - t
end

function Utility.hex(hex)
    hex = hex:gsub("#", "")
    return Color3.fromRGB(
        tonumber(hex:sub(1, 2), 16),
        tonumber(hex:sub(3, 4), 16),
        tonumber(hex:sub(5, 6), 16)
    )
end

function Utility.toHex(c)
    return string.format("#%02X%02X%02X", c.R * 255, c.G * 255, c.B * 255)
end

-- ─── Instance factory ─────────────────────────────────────────────────────
function Utility.create(class, props, children)
    local inst = Instance.new(class)
    for k, v in pairs(props or {}) do
        if k ~= "Parent" then
            inst[k] = v
        end
    end
    for _, c in ipairs(children or {}) do
        c.Parent = inst
    end
    if props and props.Parent then
        inst.Parent = props.Parent
    end
    return inst
end

-- ─── Corner / Stroke / Gradient / Padding / List helpers ─────────────────
function Utility.corner(radius, parent)
    return Utility.create("UICorner", { CornerRadius = UDim.new(0, radius or 8), Parent = parent })
end

function Utility.stroke(color, thickness, parent, transparency)
    return Utility.create("UIStroke", {
        Color = color or Color3.fromRGB(255, 255, 255),
        Thickness = thickness or 1,
        Transparency = transparency or 0,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        Parent = parent,
    })
end

function Utility.gradient(color1, color2, rotation, parent)
    return Utility.create("UIGradient", {
        Color = ColorSequence.new(color1, color2),
        Rotation = rotation or 90,
        Parent = parent,
    })
end

function Utility.padding(top, bottom, left, right, parent)
    return Utility.create("UIPadding", {
        PaddingTop = UDim.new(0, top or 0),
        PaddingBottom = UDim.new(0, bottom or 0),
        PaddingLeft = UDim.new(0, left or 0),
        PaddingRight = UDim.new(0, right or 0),
        Parent = parent,
    })
end

function Utility.list(horizontal, padding, parent, alignX, alignY)
    return Utility.create("UIListLayout", {
        FillDirection = horizontal and Enum.FillDirection.Horizontal or Enum.FillDirection.Vertical,
        Padding = UDim.new(0, padding or 0),
        HorizontalAlignment = alignX or Enum.HorizontalAlignment.Left,
        VerticalAlignment = alignY or Enum.VerticalAlignment.Top,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = parent,
    })
end

function Utility.grid(cellSize, cellPadding, parent, cols)
    return Utility.create("UIGridLayout", {
        CellSize = UDim2.fromOffset(unpack(cellSize)),
        CellPadding = UDim2.fromOffset(unpack(cellPadding or {0, 0})),
        FillDirectionMaxCells = cols or 0,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = parent,
    })
end

function Utility.aspect(ratio, parent)
    return Utility.create("UIAspectRatioConstraint", {
        AspectRatio = ratio or 1,
        Parent = parent,
    })
end

function Utility.sizeConstraint(parent, minx, miny, maxx, maxy)
    return Utility.create("UISizeConstraint", {
        MinSize = Vector2.new(minx or 0, miny or 0),
        MaxSize = Vector2.new(maxx or math.huge, maxy or math.huge),
        Parent = parent,
    })
end

-- ─── Tween shorthand ──────────────────────────────────────────────────────
function Utility.tween(obj, duration, props, easingDir, easingStyle, override)
    local info = TweenInfo.new(
        duration,
        easingStyle or Enum.EasingStyle.Quart,
        easingDir or Enum.EasingDirection.Out
    )
    local t = TweenService:Create(obj, info, props)
    if override ~= false then
        if obj._currentTween then
            obj._currentTween:Cancel()
        end
        obj._currentTween = t
    end
    t:Play()
    return t
end

function Utility.springTo(obj, prop, target, stiffness, damping, precision)
    -- Use Spring for natural motion (position-based props only)
    local initial = obj[prop]
    if typeof(initial) == "number" then
        local s = Spring.new(initial, stiffness, damping, precision)
        s.Target = target
        local conn
        conn = RunService.RenderStepped:Connect(function()
            obj[prop] = s.Position
            if not s._running then
                conn:Disconnect()
            end
        end)
        return s
    end
end

-- ─── Ripple effect (Material-style) ───────────────────────────────────────
function Utility.ripple(frame, x, y, color)
    local maxR = math.max(frame.AbsoluteSize.X, frame.AbsoluteSize.Y)
    local r = Utility.create("Frame", {
        Name = "Ripple",
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromOffset(x, y),
        Size = UDim2.fromOffset(0, 0),
        BackgroundColor3 = color or Color3.fromRGB(255, 255, 255),
        BackgroundTransparency = 0.7,
        ZIndex = 100,
        Parent = frame,
    })
    Utility.corner(maxR, r)
    local sizeT = Utility.tween(r, 0.5, { Size = UDim2.fromOffset(maxR * 2, maxR * 2), BackgroundTransparency = 1 }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
    task.delay(0.5, function() r:Destroy() end)
    return r
end

-- ─── Drag system (multi-touch safe — tracks the exact input that started the drag) ─
function Utility.makeDraggable(frame, handle, bounds)
    handle = handle or frame
    local dragging, dragStart, startPos, dragInput
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        -- For mouse: any MouseMovement while dragging. For touch: only the SAME touch.
        if dragging and (input == dragInput or (input.UserInputType == Enum.UserInputType.MouseMovement and dragInput.UserInputType == Enum.UserInputType.MouseButton1)) then
            local delta = input.Position - dragStart
            local newPos = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
            -- Constrain to viewport (with safe-area top inset on mobile)
            local sx, sy = frame.AbsoluteSize.X, frame.AbsoluteSize.Y
            local vx, vy = Camera.ViewportSize.X, Camera.ViewportSize.Y
            local topInset = Utility.getTopInset and Utility.getTopInset() or 0
            local xOff = clamp(newPos.X.Offset, 0, vx - sx)
            local yOff = clamp(newPos.Y.Offset, topInset, vy - sy)
            frame.Position = UDim2.new(0, xOff, 0, yOff)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input == dragInput or input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
            dragInput = nil
        end
    end)
end

-- ─── Mobile / platform detection ──────────────────────────────────────────
function Utility.isMobile()
    -- True mobile: touch + small viewport (excludes Surface Pro / touch laptops)
    if not UserInputService.TouchEnabled then return false end
    if UserInputService.MouseEnabled and Camera.ViewportSize.X > 900 then
        return false  -- touch-capable laptop
    end
    return true
end

function Utility.isTablet()
    if not UserInputService.TouchEnabled then return false end
    local vp = Camera.ViewportSize
    return vp.X >= 600 and vp.X < 1024
end

function Utility.isTouch()
    return UserInputService.TouchEnabled
end

function Utility.isPhone()
    return Utility.isMobile() and Camera.ViewportSize.X < 600
end

function Utility.screenScale()
    local vp = Camera.ViewportSize
    if vp.X < 600 then return 0.85       -- phone
    elseif vp.X < 1024 then return 0.95  -- tablet
    else return 1 end                    -- desktop
end

-- ─── Safe-area insets (handles notches, camera islands, top bar) ──────────
function Utility.getTopInset()
    -- CoreGui has a top bar inset (default ~36px). On mobile we also want to
    -- respect the device safe area. Roblox exposes this via GuiService.
    local ok, inset = pcall(function()
        return game:GetService("GuiService"):GetGuiInset()
    end)
    if ok and inset then return inset.Y end
    return 0
end

function Utility.getSafeArea()
    local vp = Camera.ViewportSize
    local top = Utility.getTopInset()
    -- On phones with notches the safe area adds ~24px on each side and ~44px top.
    if Utility.isPhone() then
        return {
            Top    = math.max(top, 24),
            Bottom = 16,
            Left   = 8,
            Right  = 8,
            Width  = vp.X - 16,
            Height = vp.Y - math.max(top, 24) - 16,
        }
    elseif Utility.isTablet() then
        return {
            Top    = top,
            Bottom = 8,
            Left   = 4,
            Right  = 4,
            Width  = vp.X - 8,
            Height = vp.Y - top - 8,
        }
    end
    return {
        Top    = top,
        Bottom = 0,
        Left   = 0,
        Right  = 0,
        Width  = vp.X,
        Height = vp.Y - top,
    }
end

-- ─── Touch-tap helper (handles both MouseButton1 and Touch in one event) ──
function Utility.onTap(instance, callback)
    local activeInput
    instance.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            activeInput = input
        end
    end)
    instance.InputEnded:Connect(function(input)
        if activeInput and (input == activeInput or input.UserInputType == Enum.UserInputType.MouseButton1) then
            activeInput = nil
            task.spawn(callback, input)
        end
    end)
end

-- ─── Fuzzy search ──────────────────────────────────────────────────────────
function Utility.fuzzyMatch(query, text)
    query = string.lower(query)
    text  = string.lower(text)
    if query == "" then return true, 1 end
    if text:find(query, 1, true) then return true, 1 end
    local qi = 1
    local score = 0
    local streak = 0
    for ti = 1, #text do
        if qi <= #query and text:sub(ti, ti) == query:sub(qi, qi) then
            streak = streak + 1
            score = score + streak
            qi = qi + 1
        else
            streak = 0
        end
    end
    return qi > #query, score
end

-- ═══════════════════════════════════════════════════════════════════════════
--   THEME ENGINE
-- ═══════════════════════════════════════════════════════════════════════════
local ThemeManager = {}
ThemeManager.Themes = {}
ThemeManager.Current = nil
ThemeManager._objects = {}  -- { [obj] = {property = themeKey} }

local DarkTheme = {
    Name = "Dark",
    Background     = Color3.fromRGB(15, 17, 22),
    BackgroundAlt  = Color3.fromRGB(22, 25, 32),
    Card           = Color3.fromRGB(28, 32, 40),
    CardHover      = Color3.fromRGB(36, 41, 51),
    Sidebar        = Color3.fromRGB(20, 23, 30),
    SidebarItem    = Color3.fromRGB(255, 255, 255),
    SidebarItemBg  = Color3.fromRGB(255, 255, 255),
    Primary        = Color3.fromRGB(99, 102, 241),
    PrimaryHover   = Color3.fromRGB(118, 121, 255),
    Accent         = Color3.fromRGB(129, 140, 248),
    Success        = Color3.fromRGB(52, 211, 153),
    Warning        = Color3.fromRGB(251, 191, 36),
    Error          = Color3.fromRGB(239, 68, 68),
    Info           = Color3.fromRGB(59, 130, 246),
    Text           = Color3.fromRGB(237, 240, 245),
    TextMuted      = Color3.fromRGB(148, 156, 170),
    TextSubtle     = Color3.fromRGB(107, 114, 128),
    Border         = Color3.fromRGB(50, 55, 65),
    BorderSubtle   = Color3.fromRGB(38, 42, 50),
    Overlay        = Color3.fromRGB(0, 0, 0),
    Glass          = Color3.fromRGB(28, 32, 40),
    Shadow         = Color3.fromRGB(0, 0, 0),
    BackgroundTransparency = 0.05,
    CardTransparency       = 0.15,
    GlassTransparency      = 0.25,
}

local LightTheme = {
    Name = "Light",
    Background     = Color3.fromRGB(248, 249, 251),
    BackgroundAlt  = Color3.fromRGB(241, 243, 246),
    Card           = Color3.fromRGB(255, 255, 255),
    CardHover      = Color3.fromRGB(245, 247, 250),
    Sidebar        = Color3.fromRGB(255, 255, 255),
    SidebarItem    = Color3.fromRGB(17, 24, 39),
    SidebarItemBg  = Color3.fromRGB(17, 24, 39),
    Primary        = Color3.fromRGB(79, 70, 229),
    PrimaryHover   = Color3.fromRGB(99, 90, 240),
    Accent         = Color3.fromRGB(129, 140, 248),
    Success        = Color3.fromRGB(22, 163, 74),
    Warning        = Color3.fromRGB(217, 119, 6),
    Error          = Color3.fromRGB(220, 38, 38),
    Info           = Color3.fromRGB(37, 99, 235),
    Text           = Color3.fromRGB(17, 24, 39),
    TextMuted      = Color3.fromRGB(75, 85, 99),
    TextSubtle     = Color3.fromRGB(107, 114, 128),
    Border         = Color3.fromRGB(228, 231, 235),
    BorderSubtle   = Color3.fromRGB(238, 240, 243),
    Overlay        = Color3.fromRGB(0, 0, 0),
    Glass          = Color3.fromRGB(255, 255, 255),
    Shadow         = Color3.fromRGB(0, 0, 0),
    BackgroundTransparency = 0.02,
    CardTransparency       = 0,
    GlassTransparency      = 0.08,
}

local MidnightTheme = {
    Name = "Midnight",
    Background     = Color3.fromRGB(8, 10, 18),
    BackgroundAlt  = Color3.fromRGB(14, 17, 27),
    Card           = Color3.fromRGB(22, 26, 40),
    CardHover      = Color3.fromRGB(30, 35, 53),
    Sidebar        = Color3.fromRGB(12, 14, 22),
    SidebarItem    = Color3.fromRGB(167, 139, 250),
    SidebarItemBg  = Color3.fromRGB(167, 139, 250),
    Primary        = Color3.fromRGB(139, 92, 246),
    PrimaryHover   = Color3.fromRGB(167, 139, 250),
    Accent         = Color3.fromRGB(217, 70, 239),
    Success        = Color3.fromRGB(52, 211, 153),
    Warning        = Color3.fromRGB(251, 191, 36),
    Error          = Color3.fromRGB(239, 68, 68),
    Info           = Color3.fromRGB(96, 165, 250),
    Text           = Color3.fromRGB(226, 232, 240),
    TextMuted      = Color3.fromRGB(148, 163, 184),
    TextSubtle     = Color3.fromRGB(100, 116, 139),
    Border         = Color3.fromRGB(40, 45, 65),
    BorderSubtle   = Color3.fromRGB(28, 33, 50),
    Overlay        = Color3.fromRGB(0, 0, 0),
    Glass          = Color3.fromRGB(22, 26, 40),
    Shadow         = Color3.fromRGB(0, 0, 0),
    BackgroundTransparency = 0.08,
    CardTransparency       = 0.18,
    GlassTransparency      = 0.30,
}

ThemeManager.Themes.Dark     = DarkTheme
ThemeManager.Themes.Light    = LightTheme
ThemeManager.Themes.Midnight = MidnightTheme
ThemeManager.Current         = DarkTheme

function ThemeManager.setTheme(name)
    local theme = ThemeManager.Themes[name]
    if not theme then
        warn("[PremiumUI] Unknown theme: " .. tostring(name))
        return
    end
    ThemeManager.Current = theme
    for obj, props in pairs(ThemeManager._objects) do
        if obj and obj.Parent then
            for prop, key in pairs(props) do
                obj[prop] = theme[key] or obj[prop]
            end
        else
            ThemeManager._objects[obj] = nil
        end
    end
    ThemeManager.OnThemeChanged:Fire(theme)
end

function ThemeManager.get()
    return ThemeManager.Current
end

function ThemeManager.register(obj, prop, key)
    if not ThemeManager._objects[obj] then
        ThemeManager._objects[obj] = {}
    end
    ThemeManager._objects[obj][prop] = key
    obj[prop] = ThemeManager.Current[key]
    return obj
end

function ThemeManager.registerTheme(name, themeData)
    ThemeManager.Themes[name] = themeData
end

ThemeManager.OnThemeChanged = Signal.new()

-- ═══════════════════════════════════════════════════════════════════════════
--   ANIMATION ENGINE
-- ═══════════════════════════════════════════════════════════════════════════
local AnimationEngine = {}

AnimationEngine.Easings = {
    Linear      = Enum.EasingStyle.Linear,
    Sine        = Enum.EasingStyle.Sine,
    Back        = Enum.EasingStyle.Back,
    Quad        = Enum.EasingStyle.Quad,
    Quart       = Enum.EasingStyle.Quart,
    Quint       = Enum.EasingStyle.Quint,
    Bounce      = Enum.EasingStyle.Bounce,
    Elastic     = Enum.EasingStyle.Elastic,
    Exponential = Enum.EasingStyle.Exponential,
    Circular    = Enum.EasingStyle.Circular,
    Cubic       = Enum.EasingStyle.Cubic,
}

function AnimationEngine.tween(obj, duration, props, style, dir)
    return Utility.tween(obj, duration, props, dir or Enum.EasingDirection.Out, style or Enum.EasingStyle.Quart)
end

function AnimationEngine.spring(obj, prop, target, stiffness, damping)
    return Utility.springTo(obj, prop, target, stiffness, damping)
end

function AnimationEngine.popIn(obj, delay)
    delay = delay or 0
    obj.Size = UDim2.fromScale(0.85, 0.85)
    obj.Position = obj.Position + UDim2.fromOffset(0, 20)
    obj.BackgroundTransparency = 1
    task.delay(delay, function()
        Utility.tween(obj, 0.4, { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 0 })
        Utility.tween(obj, 0.5, { Position = obj.Position - UDim2.fromOffset(0, 20) })
    end)
end

function AnimationEngine.fadeIn(obj, duration)
    obj.BackgroundTransparency = 1
    Utility.tween(obj, duration or 0.25, { BackgroundTransparency = 0 })
end

function AnimationEngine.fadeOut(obj, duration, callback)
    Utility.tween(obj, duration or 0.25, { BackgroundTransparency = 1 })
    task.delay(duration or 0.25, callback or function() end)
end

function AnimationEngine.scalePulse(obj, scale)
    scale = scale or 1.04
    local original = obj.Size
    Utility.tween(obj, 0.12, { Size = original * scale }, Enum.EasingDirection.Out, Enum.EasingStyle.Back)
    task.delay(0.12, function()
        Utility.tween(obj, 0.18, { Size = original }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
    end)
end

function AnimationEngine.slideIn(obj, fromDir)
    fromDir = fromDir or "Right"
    local startPos = obj.Position
    if fromDir == "Right" then
        obj.Position = startPos + UDim2.fromOffset(30, 0)
    elseif fromDir == "Left" then
        obj.Position = startPos - UDim2.fromOffset(30, 0)
    elseif fromDir == "Top" then
        obj.Position = startPos - UDim2.fromOffset(0, 30)
    elseif fromDir == "Bottom" then
        obj.Position = startPos + UDim2.fromOffset(0, 30)
    end
    Utility.tween(obj, 0.4, { Position = startPos }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
end

-- ═══════════════════════════════════════════════════════════════════════════
--   ICON SYSTEM  (simple SVG-like icon set via ImageLabel)
-- ═══════════════════════════════════════════════════════════════════════════
local Icons = {
    -- Common icons via Roblox asset IDs (placeholder-friendly). Override as needed.
    Sword        = "rbxassetid://12062096413",
    Shield       = "rbxassetid://12062097314",
    Home         = "rbxassetid://12062098877",
    Settings     = "rbxassetid://12062099988",
    Player       = "rbxassetid://12062101234",
    Combat       = "rbxassetid://12062096413",
    World        = "rbxassetid://12062103345",
    Misc         = "rbxassetid://12062104456",
    Search       = "rbxassetid://12062105567",
    Bell         = "rbxassetid://12062106678",
    Close        = "rbxassetid://12062107789",
    Minimize     = "rbxassetid://12062108890",
    Maximize     = "rbxassetid://12062109901",
    ChevronDown  = "rbxassetid://12062111012",
    ChevronRight = "rbxassetid://12062112123",
    Check        = "rbxassetid://12062113234",
    Plus         = "rbxassetid://12062114345",
    Trash        = "rbxassetid://12062115456",
}

local function getIcon(name)
    if not name then return nil end
    if name:match("^rbxassetid://") or name:match("^rbxasset://") or name:match("^http") then
        return name
    end
    return Icons[name]
end

-- ═══════════════════════════════════════════════════════════════════════════
--   COMPONENTS
-- ═══════════════════════════════════════════════════════════════════════════
local Components = {}

-- ─── Button ───────────────────────────────────────────────────────────────
function Components.Button(parent, config)
    local theme = ThemeManager.get()
    local btn = Utility.create("TextButton", {
        Name = config.Name or "Button",
        Size = config.Size or UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = config.Primary and theme.Primary or theme.Card,
        BackgroundTransparency = config.Primary and 0 or theme.CardTransparency,
        Text = "",
        AutoButtonColor = false,
        Parent = parent,
    })
    Utility.corner(8, btn)
    Utility.stroke(config.Primary and theme.Primary or theme.Border, 1, btn, 0.3)
    if config.Primary then
        Utility.gradient(theme.Primary, theme.PrimaryHover, 90, btn)
    end
    local label = Utility.create("TextLabel", {
        Name = "Label",
        BackgroundTransparency = 1,
        Position = UDim2.fromScale(0, 0),
        Size = UDim2.fromScale(1, 1),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Button",
        TextColor3 = config.Primary and Color3.fromRGB(255, 255, 255) or theme.Text,
        TextSize = 14,
        Parent = btn,
    })
    local icon
    if config.Icon then
        icon = Utility.create("ImageLabel", {
            Name = "Icon",
            AnchorPoint = Vector2.new(0, 0.5),
            Position = UDim2.new(0, 12, 0.5, 0),
            Size = UDim2.fromOffset(16, 16),
            BackgroundTransparency = 1,
            Image = getIcon(config.Icon) or config.Icon,
            ImageColor3 = config.Primary and Color3.fromRGB(255, 255, 255) or theme.Text,
            Parent = btn,
        })
        label.Position = UDim2.new(0, 36, 0, 0)
        label.TextXAlignment = Enum.TextXAlignment.Left
    end
    -- Hover / Press
    btn.MouseEnter:Connect(function()
        if config.Primary then
            Utility.tween(btn, 0.2, { BackgroundTransparency = 0 })
        else
            Utility.tween(btn, 0.2, { BackgroundColor3 = theme.CardHover })
        end
    end)
    btn.MouseLeave:Connect(function()
        if config.Primary then
            Utility.tween(btn, 0.2, { BackgroundTransparency = 0 })
        else
            Utility.tween(btn, 0.2, { BackgroundColor3 = theme.Card })
        end
    end)
    btn.MouseButton1Down:Connect(function()
        local rp = Utility.ripple(btn, btn.AbsoluteSize.X / 2, btn.AbsoluteSize.Y / 2, config.Primary and Color3.fromRGB(255, 255, 255) or theme.Primary)
        AnimationEngine.scalePulse(btn, 0.97)
        if config.Callback then task.spawn(config.Callback) end
    end)
    -- API
    local api = {
        Instance = btn,
        SetText = function(text) label.Text = text end,
        SetEnabled = function(state)
            btn.Active = state
            btn.AutoButtonColor = state
            label.TextTransparency = state and 0 or 0.5
            btn.BackgroundTransparency = state and (config.Primary and 0 or theme.CardTransparency) or 0.7
        end,
        SetLoading = function(loading)
            if loading then
                btn.Text = "..."
                api.SetEnabled(false)
            else
                btn.Text = ""
                api.SetEnabled(true)
            end
        end,
        Destroy = function() btn:Destroy() end,
    }
    return api
end

-- ─── Toggle ───────────────────────────────────────────────────────────────
function Components.Toggle(parent, config)
    local theme = ThemeManager.get()
    local value = config.Default or false
    local row = Utility.create("Frame", {
        Name = config.Name or "Toggle",
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 0),
        Size = UDim2.new(1, -64, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Toggle",
        TextColor3 = theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local track = Utility.create("Frame", {
        Name = "Track",
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(44, 24),
        BackgroundColor3 = value and theme.Primary or theme.Border,
        Parent = row,
    })
    Utility.corner(12, track)
    local thumb = Utility.create("Frame", {
        Name = "Thumb",
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.new(0, value and 22 or 2, 0.5, 0),
        Size = UDim2.fromOffset(20, 20),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        Parent = track,
    })
    Utility.corner(10, thumb)
    -- Hover
    row.MouseEnter:Connect(function()
        Utility.tween(row, 0.18, { BackgroundColor3 = theme.CardHover })
    end)
    row.MouseLeave:Connect(function()
        Utility.tween(row, 0.18, { BackgroundColor3 = theme.Card })
    end)
    -- Toggle action
    local function setValue(v, fireCallback)
        value = v
        Utility.tween(track, 0.22, { BackgroundColor3 = v and theme.Primary or theme.Border })
        Utility.tween(thumb, 0.28, { Position = UDim2.new(0, v and 22 or 2, 0.5, 0) }, Enum.EasingDirection.Out, Enum.EasingStyle.Back)
        if fireCallback ~= false and config.Callback then
            task.spawn(config.Callback, v)
        end
    end
    row.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            setValue(not value)
            AnimationEngine.scalePulse(row, 0.98)
        end
    end)
    return {
        Instance = row,
        SetValue = setValue,
        GetValue = function() return value end,
        SetEnabled = function(state)
            row.Active = state
            label.TextTransparency = state and 0 or 0.5
        end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Slider ───────────────────────────────────────────────────────────────
function Components.Slider(parent, config)
    local theme = ThemeManager.get()
    local value = config.Default or config.Min or 0
    local min = config.Min or 0
    local max = config.Max or 100
    local step = config.Step or 1
    local suffix = config.Suffix or ""
    local row = Utility.create("Frame", {
        Name = config.Name or "Slider",
        Size = UDim2.new(1, 0, 0, 52),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 6),
        Size = UDim2.new(1, -24, 0, 20),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Slider",
        TextColor3 = theme.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local valueLabel = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -60, 0, 6),
        Size = UDim2.fromOffset(48, 20),
        Font = Enum.Font.GothamMedium,
        Text = tostring(value) .. suffix,
        TextColor3 = theme.Primary,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Right,
        Parent = row,
    })
    local track = Utility.create("Frame", {
        Name = "Track",
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.fromOffset(12, 38),
        Size = UDim2.new(1, -24, 0, 4),
        BackgroundColor3 = theme.Border,
        Parent = row,
    })
    Utility.corner(2, track)
    local fill = Utility.create("Frame", {
        Name = "Fill",
        Size = UDim2.fromScale((value - min) / (max - min), 1),
        BackgroundColor3 = theme.Primary,
        Parent = track,
    })
    Utility.corner(2, fill)
    local thumb = Utility.create("Frame", {
        Name = "Thumb",
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale((value - min) / (max - min), 0.5),
        Size = UDim2.fromOffset(16, 16),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        Parent = track,
    })
    Utility.corner(8, thumb)
    Utility.stroke(theme.Primary, 2, thumb, 0)
    -- Drag (multi-touch safe — tracks the exact input that started the drag)
    local dragging = false
    local dragInput = nil
    local function update(input)
        local rel = (input.Position.X - track.AbsolutePosition.X) / track.AbsoluteSize.X
        rel = clamp(rel, 0, 1)
        local v = min + (max - min) * rel
        v = math.floor(v / step + 0.5) * step
        v = clamp(v, min, max)
        value = v
        valueLabel.Text = tostring(v) .. suffix
        fill.Size = UDim2.fromScale((v - min) / (max - min), 1)
        thumb.Position = UDim2.fromScale((v - min) / (max - min), 0.5)
        if config.Callback then task.spawn(config.Callback, v) end
    end
    local function beginDrag(input)
        dragging = true
        dragInput = input
        Utility.tween(thumb, 0.15, { Size = UDim2.fromOffset(22, 22) })
    end
    local function endDrag()
        dragging = false
        dragInput = nil
        Utility.tween(thumb, 0.15, { Size = UDim2.fromOffset(16, 16) })
    end
    -- Make the thumb and track tappable to start a drag (better for touch)
    thumb.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            beginDrag(input)
        end
    end)
    track.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            beginDrag(input)
            update(input)
        end
    end)
    -- Bigger touch hit area: invisible button overlay on the slider zone
    local hitZone = Utility.create("TextButton", {
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 0, 0, 28),
        Size = UDim2.new(1, 0, 0, 24),
        Text = "",
        AutoButtonColor = false,
        ZIndex = 0,
        Parent = row,
    })
    hitZone.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            beginDrag(input)
            update(input)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        -- For touch: only respond to the same touch that started the drag.
        -- For mouse: respond to MouseMovement.
        if dragging and (input == dragInput or
            (input.UserInputType == Enum.UserInputType.MouseMovement and dragInput and dragInput.UserInputType == Enum.UserInputType.MouseButton1)) then
            update(input)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if dragging and (input == dragInput or input.UserInputType == Enum.UserInputType.MouseButton1) then
            endDrag()
        end
    end)
    return {
        Instance = row,
        SetValue = function(v, fire)
            value = clamp(v, min, max)
            valueLabel.Text = tostring(value) .. suffix
            fill.Size = UDim2.fromScale((value - min) / (max - min), 1)
            thumb.Position = UDim2.fromScale((value - min) / (max - min), 0.5)
            if fire ~= false and config.Callback then task.spawn(config.Callback, value) end
        end,
        GetValue = function() return value end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Dropdown ─────────────────────────────────────────────────────────────
function Components.Dropdown(parent, config)
    local theme = ThemeManager.get()
    local options = config.Options or {}
    local multi = config.Multi or false
    local selected = config.Default or (multi and {} or nil)
    local expanded = false
    local row = Utility.create("TextButton", {
        Name = config.Name or "Dropdown",
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        AutoButtonColor = false,
        Text = "",
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 4),
        Size = UDim2.new(1, -64, 0, 18),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Dropdown",
        TextColor3 = theme.TextMuted,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local valueLabel = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 20),
        Size = UDim2.new(1, -40, 0, 16),
        Font = Enum.Font.GothamMedium,
        Text = multi and (selected and #selected > 0 and table.concat(selected, ", ") or "None") or (selected or "Select..."),
        TextColor3 = theme.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local chev = Utility.create("ImageLabel", {
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(16, 16),
        BackgroundTransparency = 1,
        Image = Icons.ChevronDown,
        ImageColor3 = theme.TextMuted,
        Parent = row,
    })
    -- List container (clipped)
    local clipper = Utility.create("Frame", {
        Name = "ListClip",
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 0, 1, 4),
        Size = UDim2.new(1, 0, 0, 0),
        ClipsDescendants = true,
        Visible = false,
        ZIndex = 50,
        Parent = row,
    })
    Utility.corner(8, clipper)
    local list = Utility.create("ScrollingFrame", {
        Name = "List",
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = 0,
        Size = UDim2.new(1, 0, 1, 0),
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = clipper,
    })
    Utility.corner(8, list)
    Utility.stroke(theme.Border, 1, list, 0.4)
    Utility.padding(6, 6, 6, 6, list)
    Utility.list(false, 4, list)
    local function refreshValueLabel()
        if multi then
            valueLabel.Text = #selected > 0 and table.concat(selected, ", ") or "None"
        else
            valueLabel.Text = selected or "Select..."
        end
    end
    local function buildOptions()
        for _, c in ipairs(list:GetChildren()) do
            if c:IsA("TextButton") then c:Destroy() end
        end
        for _, opt in ipairs(options) do
            local item = Utility.create("TextButton", {
                Size = UDim2.new(1, 0, 0, 30),
                BackgroundColor3 = (multi and table.find(selected, opt)) or (not multi and selected == opt) and theme.CardHover or theme.Background,
                BackgroundTransparency = 0,
                Text = "",
                AutoButtonColor = false,
                Parent = list,
            })
            Utility.corner(6, item)
            local txt = Utility.create("TextLabel", {
                BackgroundTransparency = 1,
                Position = UDim2.fromOffset(10, 0),
                Size = UDim2.new(1, -20, 1, 0),
                Font = Enum.Font.GothamMedium,
                Text = tostring(opt),
                TextColor3 = theme.Text,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = item,
            })
            item.MouseEnter:Connect(function() Utility.tween(item, 0.15, { BackgroundColor3 = theme.CardHover }) end)
            item.MouseLeave:Connect(function()
                local active = (multi and table.find(selected, opt)) or (not multi and selected == opt)
                Utility.tween(item, 0.15, { BackgroundColor3 = active and theme.CardHover or theme.Background })
            end)
            item.MouseButton1Down:Connect(function()
                if multi then
                    local idx = table.find(selected, opt)
                    if idx then table.remove(selected, idx) else table.insert(selected, opt) end
                else
                    selected = opt
                end
                refreshValueLabel()
                buildOptions()
                AnimationEngine.scalePulse(item, 0.97)
                if config.Callback then task.spawn(config.Callback, selected) end
                if not multi then
                    expand(false)
                end
            end)
        end
    end
    local function expand(state)
        expanded = state
        if state then
            clipper.Visible = true
            buildOptions()
            Utility.tween(chev, 0.2, { Rotation = 180 })
            Utility.tween(clipper, 0.25, { Size = UDim2.new(1, 0, 0, math.min(#options * 34 + 12, 200)) })
        else
            Utility.tween(chev, 0.2, { Rotation = 0 })
            Utility.tween(clipper, 0.2, { Size = UDim2.new(1, 0, 0, 0) })
            task.delay(0.2, function() if not expanded then clipper.Visible = false end end)
        end
    end
    row.MouseButton1Down:Connect(function()
        expand(not expanded)
    end)
    -- Hover (mouse only — skipped on touch)
    if not Utility.isTouch() then
        row.MouseEnter:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.CardHover }) end)
        row.MouseLeave:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.Card }) end)
    end
    -- Click-outside (works with both mouse and touch)
    UserInputService.InputBegan:Connect(function(input)
        if expanded and (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) then
            local p = input.Position
            local ap1, as1 = row.AbsolutePosition, row.AbsoluteSize
            local ap2, as2 = clipper.AbsolutePosition, clipper.AbsoluteSize
            local inRow = p.X >= ap1.X and p.X <= ap1.X + as1.X and p.Y >= ap1.Y and p.Y <= ap1.Y + as1.Y
            local inClip = p.X >= ap2.X and p.X <= ap2.X + as2.X and p.Y >= ap2.Y and p.Y <= ap2.Y + as2.Y
            if not inRow and not inClip then expand(false) end
        end
    end)
    return {
        Instance = row,
        SetValue = function(v) selected = v refreshValueLabel() if config.Callback then task.spawn(config.Callback, v) end end,
        GetValue = function() return selected end,
        SetOptions = function(opts) options = opts buildOptions() end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Textbox ──────────────────────────────────────────────────────────────
function Components.Textbox(parent, config)
    local theme = ThemeManager.get()
    local row = Utility.create("Frame", {
        Name = config.Name or "Textbox",
        Size = UDim2.new(1, 0, 0, 44),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 4),
        Size = UDim2.new(1, -24, 0, 16),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Textbox",
        TextColor3 = theme.TextMuted,
        TextSize = 11,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local box = Utility.create("TextBox", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 22),
        Size = UDim2.new(1, -24, 0, 18),
        Font = Enum.Font.GothamMedium,
        Text = config.Default or "",
        PlaceholderText = config.Placeholder or "",
        PlaceholderColor3 = theme.TextSubtle,
        TextColor3 = theme.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        ClearTextOnFocus = false,
        Parent = row,
    })
    local stroke = Utility.stroke(theme.Border, 1, row, 0.4)
    box.Focused:Connect(function()
        Utility.tween(stroke, 0.2, { Color = theme.Primary, Transparency = 0 })
    end)
    box.FocusLost:Connect(function(enter)
        Utility.tween(stroke, 0.2, { Color = theme.Border, Transparency = 0.4 })
        if config.Callback then task.spawn(config.Callback, box.Text, enter) end
    end)
    return {
        Instance = row,
        SetValue = function(v) box.Text = v end,
        GetValue = function() return box.Text end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Label ────────────────────────────────────────────────────────────────
function Components.Label(parent, config)
    local theme = ThemeManager.get()
    local lbl = Utility.create("TextLabel", {
        Name = config.Name or "Label",
        Size = UDim2.new(1, 0, 0, 24),
        BackgroundTransparency = 1,
        Font = Enum.Font.GothamMedium,
        Text = config.Text or "Label",
        TextColor3 = theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = parent,
    })
    return { Instance = lbl, SetText = function(t) lbl.Text = t end, Destroy = function() lbl:Destroy() end }
end

-- ─── Paragraph ────────────────────────────────────────────────────────────
function Components.Paragraph(parent, config)
    local theme = ThemeManager.get()
    local lbl = Utility.create("TextLabel", {
        Name = config.Name or "Paragraph",
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        Font = Enum.Font.Gotham,
        Text = config.Text or "",
        RichText = true,
        TextColor3 = theme.TextMuted,
        TextSize = 13,
        TextWrapped = true,
        TextXAlignment = Enum.TextXAlignment.Left,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = parent,
    })
    return { Instance = lbl, SetText = function(t) lbl.Text = t end, Destroy = function() lbl:Destroy() end }
end

-- ─── Keybind ──────────────────────────────────────────────────────────────
function Components.Keybind(parent, config)
    local theme = ThemeManager.get()
    local current = config.Default or Enum.KeyCode.Unknown
    local listening = false
    local row = Utility.create("TextButton", {
        Name = config.Name or "Keybind",
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Text = "",
        AutoButtonColor = false,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 0),
        Size = UDim2.new(1, -90, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Keybind",
        TextColor3 = theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local pill = Utility.create("TextLabel", {
        BackgroundColor3 = theme.Background,
        BackgroundTransparency = 0.4,
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(70, 22),
        Font = Enum.Font.GothamMedium,
        Text = current.Name,
        TextColor3 = theme.Text,
        TextSize = 12,
        Parent = row,
    })
    Utility.corner(6, pill)
    Utility.stroke(theme.Border, 1, pill, 0.6)
    row.MouseButton1Down:Connect(function()
        listening = true
        pill.Text = "..."
        Utility.tween(pill, 0.18, { BackgroundColor3 = theme.Primary, TextColor3 = Color3.fromRGB(255, 255, 255) })
    end)
    UserInputService.InputBegan:Connect(function(input, gp)
        if listening and not gp then
            local code = input.KeyCode
            if code ~= Enum.KeyCode.Unknown then
                listening = false
                current = code
                pill.Text = code.Name
                Utility.tween(pill, 0.18, { BackgroundColor3 = theme.Background, TextColor3 = theme.Text })
                if config.Callback then task.spawn(config.Callback, code) end
            end
        elseif not listening and current == input.KeyCode and config.OnPress then
            task.spawn(config.OnPress)
        end
    end)
    row.MouseEnter:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.CardHover }) end)
    row.MouseLeave:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.Card }) end)
    return {
        Instance = row,
        SetValue = function(k) current = k pill.Text = k.Name end,
        GetValue = function() return current end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Color Picker ─────────────────────────────────────────────────────────
function Components.ColorPicker(parent, config)
    local theme = ThemeManager.get()
    local color = config.Default or Color3.fromRGB(255, 255, 255)
    local row = Utility.create("TextButton", {
        Name = config.Name or "ColorPicker",
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Text = "",
        AutoButtonColor = false,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 0),
        Size = UDim2.new(1, -60, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Color",
        TextColor3 = theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local swatch = Utility.create("Frame", {
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(32, 22),
        BackgroundColor3 = color,
        Parent = row,
    })
    Utility.corner(6, swatch)
    Utility.stroke(theme.Border, 1, swatch, 0.4)
    -- Picker overlay (simple SV + H bar)
    local overlay = Utility.create("Frame", {
        Name = "Picker",
        Position = UDim2.new(1, -200, 1, 8),
        Size = UDim2.fromOffset(200, 180),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = 0,
        Visible = false,
        ZIndex = 100,
        Parent = row,
    })
    Utility.corner(8, overlay)
    Utility.stroke(theme.Border, 1, overlay, 0.4)
    Utility.padding(10, 10, 10, 10, overlay)
    local sv = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 120),
        BackgroundColor3 = color,
        BorderSizePixel = 0,
        Parent = overlay,
    })
    Utility.corner(4, sv)
    local whiteBg = Utility.create("Frame", { Size = UDim2.fromScale(1, 1), BackgroundColor3 = Color3.fromRGB(255, 255, 255), BorderSizePixel = 0, Parent = sv })
    Utility.corner(4, whiteBg)
    local blackGrad = Utility.create("UIGradient", { Color = ColorSequence.new(Color3.fromRGB(0, 0, 0), Color3.fromRGB(0, 0, 0)), Transparency = NumberSequence.new(0, 1), Rotation = 0, Parent = whiteBg })
    local whiteGrad = Utility.create("UIGradient", { Color = ColorSequence.new(Color3.fromRGB(255, 255, 255), Color3.fromRGB(255, 255, 255)), Transparency = NumberSequence.new(1, 0), Rotation = 90, Parent = whiteBg })
    local hueBar = Utility.create("Frame", {
        Position = UDim2.new(0, 0, 1, -30),
        Size = UDim2.new(1, 0, 0, 16),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        BorderSizePixel = 0,
        Parent = overlay,
    })
    Utility.corner(4, hueBar)
    Utility.gradient(Color3.fromRGB(255, 0, 0), Color3.fromRGB(0, 255, 0), 0, hueBar)
    local hueGrad2 = Utility.create("UIGradient", { Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
        ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 255, 0)),
        ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 255, 0)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 255, 255)),
        ColorSequenceKeypoint.new(0.67, Color3.fromRGB(0, 0, 255)),
        ColorSequenceKeypoint.new(0.83, Color3.fromRGB(255, 0, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0)),
    }), Rotation = 0, Parent = hueBar })
    -- Interactions (drag to pick hue — multi-touch safe)
    local hue = 0
    local hueDragging = false
    local hueDragInput = nil
    local function updateHue(input)
        local rel = clamp((input.Position.X - hueBar.AbsolutePosition.X) / hueBar.AbsoluteSize.X, 0, 1)
        hue = rel * 360
        color = Color3.fromHSV(hue / 360, 1, 1)
        sv.BackgroundColor3 = color
        swatch.BackgroundColor3 = color
        if config.Callback then task.spawn(config.Callback, color) end
    end
    hueBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            hueDragging = true
            hueDragInput = input
            updateHue(input)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if hueDragging and (input == hueDragInput or
            (input.UserInputType == Enum.UserInputType.MouseMovement and hueDragInput and hueDragInput.UserInputType == Enum.UserInputType.MouseButton1)) then
            updateHue(input)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if hueDragging and (input == hueDragInput or input.UserInputType == Enum.UserInputType.MouseButton1) then
            hueDragging = false
            hueDragInput = nil
        end
    end)
    local expanded = false
    row.MouseButton1Down:Connect(function()
        expanded = not expanded
        overlay.Visible = expanded
        if expanded then
            overlay.Size = UDim2.fromOffset(0, 0)
            Utility.tween(overlay, 0.25, { Size = UDim2.fromOffset(200, 180) })
        else
            Utility.tween(overlay, 0.2, { Size = UDim2.fromOffset(0, 0) })
            task.delay(0.2, function() if not expanded then overlay.Visible = false end end)
        end
    end)
    row.MouseEnter:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.CardHover }) end)
    row.MouseLeave:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.Card }) end)
    return {
        Instance = row,
        SetValue = function(c) color = c swatch.BackgroundColor3 = c if config.Callback then task.spawn(config.Callback, c) end end,
        GetValue = function() return color end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Progress Bar ─────────────────────────────────────────────────────────
function Components.ProgressBar(parent, config)
    local theme = ThemeManager.get()
    local row = Utility.create("Frame", {
        Name = config.Name or "Progress",
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 6),
        Size = UDim2.new(1, -24, 0, 14),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Progress",
        TextColor3 = theme.Text,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local pct = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -50, 6, 0),
        Size = UDim2.fromOffset(38, 14),
        Font = Enum.Font.GothamMedium,
        Text = "0%",
        TextColor3 = theme.TextMuted,
        TextSize = 12,
        TextXAlignment = Enum.TextXAlignment.Right,
        Parent = row,
    })
    local track = Utility.create("Frame", {
        AnchorPoint = Vector2.new(0.5, 0),
        Position = UDim2.new(0.5, 0, 1, -8),
        Size = UDim2.new(1, -24, 0, 4),
        BackgroundColor3 = theme.Border,
        Parent = row,
    })
    Utility.corner(2, track)
    local fill = Utility.create("Frame", {
        Size = UDim2.fromScale(0, 1),
        BackgroundColor3 = theme.Primary,
        Parent = track,
    })
    Utility.corner(2, fill)
    Utility.gradient(theme.Primary, theme.PrimaryHover, 0, fill)
    local value = 0
    local function setValue(v)
        value = clamp(v, 0, 100)
        Utility.tween(fill, 0.4, { Size = UDim2.fromScale(value / 100, 1) })
        pct.Text = math.floor(value) .. "%"
        if config.Callback then task.spawn(config.Callback, value) end
    end
    return { Instance = row, SetValue = setValue, GetValue = function() return value end, Destroy = function() row:Destroy() end }
end

-- ─── Loading Spinner ──────────────────────────────────────────────────────
function Components.LoadingIndicator(parent, config)
    local theme = ThemeManager.get()
    local size = config.Size or 24
    local ring = Utility.create("Frame", {
        Name = "Loading",
        Size = UDim2.fromOffset(size, size),
        BackgroundTransparency = 1,
        Parent = parent,
    })
    -- Track (dim ring)
    local track = Utility.create("Frame", {
        Size = UDim2.fromScale(1, 1),
        BackgroundColor3 = theme.Border,
        BackgroundTransparency = 0.6,
        BorderSizePixel = 0,
        Parent = ring,
    })
    Utility.corner(size / 2, track)
    -- Spinner (rotating arc with thick stroke, transparent fill)
    local spinner = Utility.create("Frame", {
        Size = UDim2.fromScale(1, 1),
        BackgroundColor3 = theme.Primary,
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Parent = ring,
    })
    Utility.corner(size / 2, spinner)
    Utility.stroke(theme.Primary, 3, spinner, 0)
    -- Continuous rotation (clockwise, linear, 60+ FPS smooth)
    task.spawn(function()
        while ring.Parent do
            spinner.Rotation = (spinner.Rotation + 6) % 360
            RunService.RenderStepped:Wait()
        end
    end)
    return { Instance = ring, Destroy = function() ring:Destroy() end }
end

-- ─── Checkbox ─────────────────────────────────────────────────────────────
function Components.Checkbox(parent, config)
    local theme = ThemeManager.get()
    local checked = config.Default or false
    local row = Utility.create("TextButton", {
        Name = config.Name or "Checkbox",
        Size = UDim2.new(1, 0, 0, 32),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Text = "",
        AutoButtonColor = false,
        Parent = parent,
    })
    Utility.corner(6, row)
    local box = Utility.create("Frame", {
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.fromOffset(12, 16),
        Size = UDim2.fromOffset(18, 18),
        BackgroundColor3 = checked and theme.Primary or theme.Background,
        Parent = row,
    })
    Utility.corner(4, box)
    Utility.stroke(checked and theme.Primary or theme.Border, 1.5, box, 0)
    local check = Utility.create("ImageLabel", {
        Size = UDim2.fromScale(1, 1),
        BackgroundTransparency = 1,
        Image = Icons.Check,
        ImageColor3 = Color3.fromRGB(255, 255, 255),
        ImageTransparency = checked and 0 or 1,
        Parent = box,
    })
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(40, 0),
        Size = UDim2.new(1, -52, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Checkbox",
        TextColor3 = theme.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    local function set(v, fire)
        checked = v
        Utility.tween(box, 0.18, { BackgroundColor3 = v and theme.Primary or theme.Background })
        Utility.tween(check, 0.2, { ImageTransparency = v and 0 or 1 })
        if fire ~= false and config.Callback then task.spawn(config.Callback, v) end
    end
    row.MouseButton1Down:Connect(function()
        set(not checked)
        AnimationEngine.scalePulse(row, 0.97)
    end)
    row.MouseEnter:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.CardHover }) end)
    row.MouseLeave:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.Card }) end)
    return {
        Instance = row,
        SetValue = set,
        GetValue = function() return checked end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Radio Group ──────────────────────────────────────────────────────────
function Components.RadioGroup(parent, config)
    local theme = ThemeManager.get()
    local options = config.Options or {}
    local selected = config.Default or options[1]
    local container = Utility.create("Frame", {
        Name = config.Name or "RadioGroup",
        Size = UDim2.new(1, 0, 0, #options * 32 + 8),
        BackgroundTransparency = 1,
        Parent = parent,
    })
    Utility.list(false, 4, container)
    local buttons = {}
    for _, opt in ipairs(options) do
        local row = Utility.create("TextButton", {
            Size = UDim2.new(1, 0, 0, 32),
            BackgroundColor3 = theme.Card,
            BackgroundTransparency = theme.CardTransparency,
            Text = "",
            AutoButtonColor = false,
            Parent = container,
        })
        Utility.corner(6, row)
        local circle = Utility.create("Frame", {
            Position = UDim2.fromOffset(12, 7),
            Size = UDim2.fromOffset(18, 18),
            BackgroundColor3 = theme.Background,
            Parent = row,
        })
        Utility.corner(9, circle)
        Utility.stroke(selected == opt and theme.Primary or theme.Border, 1.5, circle, 0)
        local dot = Utility.create("Frame", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.fromScale(0.5, 0.5),
            Size = UDim2.fromOffset(10, 10),
            BackgroundColor3 = theme.Primary,
            BackgroundTransparency = selected == opt and 0 or 1,
            Parent = circle,
        })
        Utility.corner(5, dot)
        local label = Utility.create("TextLabel", {
            BackgroundTransparency = 1,
            Position = UDim2.fromOffset(40, 0),
            Size = UDim2.new(1, -52, 1, 0),
            Font = Enum.Font.GothamMedium,
            Text = tostring(opt),
            TextColor3 = theme.Text,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = row,
        })
        row.MouseButton1Down:Connect(function()
            if selected ~= opt then
                selected = opt
                for _, b in ipairs(buttons) do
                    Utility.tween(b.dot, 0.18, { BackgroundTransparency = b.opt == selected and 0 or 1 })
                    Utility.tween(b.circleStroke, 0.18, { Color = b.opt == selected and theme.Primary or theme.Border })
                end
                if config.Callback then task.spawn(config.Callback, selected) end
            end
        end)
        table.insert(buttons, { opt = opt, row = row, dot = dot, circleStroke = circle:FindFirstChildOfClass("UIStroke") })
    end
    return {
        Instance = container,
        SetValue = function(v) selected = v for _, b in ipairs(buttons) do
            Utility.tween(b.dot, 0.18, { BackgroundTransparency = b.opt == selected and 0 or 1 })
            Utility.tween(b.circleStroke, 0.18, { Color = b.opt == selected and theme.Primary or theme.Border })
        end if config.Callback then task.spawn(config.Callback, v) end end,
        GetValue = function() return selected end,
        Destroy = function() container:Destroy() end,
    }
end

-- ─── Tag / Badge ──────────────────────────────────────────────────────────
function Components.Badge(parent, config)
    local theme = ThemeManager.get()
    local accentColor = config.Color or theme.Primary
    local pill = Utility.create("Frame", {
        Name = config.Text or "Badge",
        Size = UDim2.fromOffset(60, 22),
        BackgroundColor3 = accentColor,
        BackgroundTransparency = 0.85,
        Parent = parent,
    })
    Utility.corner(11, pill)
    Utility.stroke(accentColor, 1, pill, 0.2)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.fromScale(1, 1),
        Font = Enum.Font.GothamMedium,
        Text = config.Text or "Badge",
        TextColor3 = accentColor,
        TextSize = 11,
        Parent = pill,
    })
    Utility.padding(8, 0, 8, 0, label)
    pill.AutomaticSize = Enum.AutomaticSize.X
    return { Instance = pill, SetText = function(t) label.Text = t end, Destroy = function() pill:Destroy() end }
end

-- ─── Tooltip (hover) ──────────────────────────────────────────────────────
function Components.Tooltip(target, config)
    local theme = ThemeManager.get()
    local tip = Utility.create("TextLabel", {
        Name = "Tooltip",
        Size = UDim2.fromOffset(0, 0),
        BackgroundColor3 = theme.Background,
        BackgroundTransparency = 0.05,
        Font = Enum.Font.GothamMedium,
        Text = config.Text or "",
        TextColor3 = theme.Text,
        TextSize = 12,
        AutomaticSize = Enum.AutomaticSize.XY,
        Visible = false,
        ZIndex = 1000,
        Parent = target.Parent,
    })
    Utility.corner(6, tip)
    Utility.stroke(theme.Border, 1, tip, 0.4)
    Utility.padding(6, 4, 8, 8, tip)
    target.MouseEnter:Connect(function()
        local mouseLoc = UserInputService:GetMouseLocation()
        tip.Position = UDim2.fromOffset(mouseLoc.X + 12, mouseLoc.Y - 30)
        tip.Visible = true
        tip.BackgroundTransparency = 1
        tip.TextTransparency = 1
        Utility.tween(tip, 0.18, { BackgroundTransparency = 0.05, TextTransparency = 0 })
    end)
    target.MouseLeave:Connect(function()
        Utility.tween(tip, 0.15, { BackgroundTransparency = 1, TextTransparency = 1 })
        task.delay(0.15, function() tip.Visible = false end)
    end)
    return { Instance = tip, Destroy = function() tip:Destroy() end }
end

-- ─── Card (container) ─────────────────────────────────────────────────────
function Components.Card(parent, config)
    local theme = ThemeManager.get()
    local card = Utility.create("Frame", {
        Name = config.Name or "Card",
        Size = config.Size or UDim2.new(1, 0, 0, 100),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Parent = parent,
    })
    Utility.corner(12, card)
    Utility.stroke(theme.Border, 1, card, 0.4)
    Utility.padding(12, 12, 12, 12, card)
    Utility.list(false, 6, card)
    return { Instance = card, Destroy = function() card:Destroy() end }
end

-- ─── Accordion ────────────────────────────────────────────────────────────
function Components.Accordion(parent, config)
    local theme = ThemeManager.get()
    local expanded = config.Default or false
    local wrap = Utility.create("Frame", {
        Name = config.Name or "Accordion",
        Size = UDim2.new(1, 0, 0, expanded and 200 or 44),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = parent,
    })
    local header = Utility.create("TextButton", {
        Size = UDim2.new(1, 0, 0, 44),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Text = "",
        AutoButtonColor = false,
        Parent = wrap,
    })
    Utility.corner(8, header)
    Utility.stroke(theme.Border, 1, header, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 0),
        Size = UDim2.new(1, -40, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Section",
        TextColor3 = theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header,
    })
    local chev = Utility.create("ImageLabel", {
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(16, 16),
        BackgroundTransparency = 1,
        Image = Icons.ChevronDown,
        ImageColor3 = theme.TextMuted,
        Rotation = expanded and 180 or 0,
        Parent = header,
    })
    local body = Utility.create("Frame", {
        Name = "Body",
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        ClipsDescendants = true,
        AutomaticSize = expanded and Enum.AutomaticSize.Y or Enum.AutomaticSize.None,
        Parent = wrap,
    })
    Utility.padding(0, 4, 0, 0, body)
    Utility.list(false, 4, body)
    local function setExpand(state)
        expanded = state
        Utility.tween(chev, 0.25, { Rotation = state and 180 or 0 })
        body.AutomaticSize = state and Enum.AutomaticSize.Y or Enum.AutomaticSize.None
        if not state then
            Utility.tween(body, 0.25, { Size = UDim2.new(1, 0, 0, 0) }, Enum.EasingDirection.In, Enum.EasingStyle.Quart)
        else
            Utility.tween(body, 0.25, { Size = UDim2.new(1, 0, 0, 0) }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
        end
    end
    header.MouseButton1Down:Connect(function() setExpand(not expanded) end)
    header.MouseEnter:Connect(function() Utility.tween(header, 0.18, { BackgroundColor3 = theme.CardHover }) end)
    header.MouseLeave:Connect(function() Utility.tween(header, 0.18, { BackgroundColor3 = theme.Card }) end)
    return {
        Instance = wrap,
        Body = body,
        Toggle = setExpand,
        Destroy = function() wrap:Destroy() end,
    }
end

-- ─── Table ────────────────────────────────────────────────────────────────
function Components.Table(parent, config)
    local theme = ThemeManager.get()
    local cols = config.Columns or {}
    local rows = config.Rows or {}
    local wrap = Utility.create("Frame", {
        Name = config.Name or "Table",
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = parent,
    })
    Utility.corner(8, wrap)
    Utility.stroke(theme.Border, 1, wrap, 0.4)
    Utility.padding(0, 0, 0, 0, wrap)
    local header = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = theme.BackgroundAlt,
        Parent = wrap,
    })
    Utility.corner(8, header)
    local colWidth = 1 / #cols
    for i, col in ipairs(cols) do
        local lbl = Utility.create("TextLabel", {
            AnchorPoint = Vector2.new(0, 0),
            Position = UDim2.fromScale((i - 1) * colWidth, 0),
            Size = UDim2.fromScale(colWidth, 1),
            BackgroundTransparency = 1,
            Font = Enum.Font.GothamMedium,
            Text = col,
            TextColor3 = theme.TextMuted,
            TextSize = 12,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = header,
        })
        Utility.padding(0, 0, 12, 0, lbl)
    end
    local body = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = wrap,
    })
    Utility.list(false, 0, body)
    local function renderRows()
        for _, c in ipairs(body:GetChildren()) do c:Destroy() end
        for _, row in ipairs(rows) do
            local r = Utility.create("Frame", {
                Size = UDim2.new(1, 0, 0, 36),
                BackgroundColor3 = theme.Background,
                BackgroundTransparency = 0.4,
                Parent = body,
            })
            for i, cell in ipairs(row) do
                local lbl = Utility.create("TextLabel", {
                    Position = UDim2.fromScale((i - 1) * colWidth, 0),
                    Size = UDim2.fromScale(colWidth, 1),
                    BackgroundTransparency = 1,
                    Font = Enum.Font.Gotham,
                    Text = tostring(cell),
                    TextColor3 = theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = r,
                })
                Utility.padding(0, 0, 12, 0, lbl)
            end
        end
    end
    renderRows()
    return {
        Instance = wrap,
        SetRows = function(r) rows = r renderRows() end,
        Destroy = function() wrap:Destroy() end,
    }
end

-- ─── Tree View ────────────────────────────────────────────────────────────
function Components.TreeView(parent, config)
    local theme = ThemeManager.get()
    local wrap = Utility.create("Frame", {
        Name = config.Name or "TreeView",
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = parent,
    })
    Utility.list(false, 2, wrap)
    local function renderNode(node, depth)
        local row = Utility.create("TextButton", {
            Size = UDim2.new(1, 0, 0, 28),
            BackgroundColor3 = theme.Card,
            BackgroundTransparency = 0.5,
            Text = "",
            AutoButtonColor = false,
            Parent = wrap,
        })
        Utility.corner(6, row)
        Utility.padding(0, 0, depth * 16 + 12, 12, row)
        local chev, childHolder
        if node.Children and #node.Children > 0 then
            chev = Utility.create("ImageLabel", {
                Position = UDim2.fromOffset(depth * 16 + 0, 6),
                Size = UDim2.fromOffset(16, 16),
                BackgroundTransparency = 1,
                Image = Icons.ChevronRight,
                ImageColor3 = theme.TextMuted,
                Parent = row,
            })
        end
        local lbl = Utility.create("TextLabel", {
            BackgroundTransparency = 1,
            Position = UDim2.fromOffset(depth * 16 + (chev and 20 or 0), 0),
            Size = UDim2.new(1, -(depth * 16 + (chev and 20 or 0)), 1, 0),
            Font = Enum.Font.GothamMedium,
            Text = node.Name or "Node",
            TextColor3 = theme.Text,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = row,
        })
        if node.Children and #node.Children > 0 then
            local expanded = false
            row.MouseButton1Down:Connect(function()
                expanded = not expanded
                if chev then Utility.tween(chev, 0.2, { Rotation = expanded and 90 or 0 }) end
                -- For simplicity, we'd render children here
                if expanded and config.OnExpand then config.OnExpand(node) end
            end)
        end
        row.MouseEnter:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.CardHover }) end)
        row.MouseLeave:Connect(function() Utility.tween(row, 0.18, { BackgroundColor3 = theme.Card }) end)
    end
    for _, node in ipairs(config.Nodes or {}) do
        renderNode(node, 0)
    end
    return { Instance = wrap, Destroy = function() wrap:Destroy() end }
end

-- ─── Date Picker (simplified) ─────────────────────────────────────────────
function Components.DatePicker(parent, config)
    local theme = ThemeManager.get()
    local now = os.date("*t")
    local selected = config.Default or os.time()
    local row = Utility.create("TextButton", {
        Name = config.Name or "DatePicker",
        Size = UDim2.new(1, 0, 0, 38),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        Text = "",
        AutoButtonColor = false,
        Parent = parent,
    })
    Utility.corner(8, row)
    Utility.stroke(theme.Border, 1, row, 0.4)
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(12, 0),
        Size = UDim2.new(1, -24, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = (config.Name or "Date") .. ": " .. os.date("%Y-%m-%d", selected),
        TextColor3 = theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = row,
    })
    row.MouseButton1Down:Connect(function()
        -- Cycle through days for simplicity
        selected = selected + 86400
        label.Text = (config.Name or "Date") .. ": " .. os.date("%Y-%m-%d", selected)
        if config.Callback then task.spawn(config.Callback, selected) end
    end)
    return {
        Instance = row,
        SetValue = function(t) selected = t label.Text = (config.Name or "Date") .. ": " .. os.date("%Y-%m-%d", t) end,
        GetValue = function() return selected end,
        Destroy = function() row:Destroy() end,
    }
end

-- ─── Context Menu ─────────────────────────────────────────────────────────
function Components.ContextMenu(parent, items)
    local theme = ThemeManager.get()
    local menu = Utility.create("Frame", {
        Name = "ContextMenu",
        Size = UDim2.fromOffset(180, 0),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = 0.05,
        Visible = false,
        ZIndex = 1000,
        Parent = parent,
    })
    Utility.corner(8, menu)
    Utility.stroke(theme.Border, 1, menu, 0.4)
    Utility.padding(6, 6, 6, 6, menu)
    Utility.list(false, 2, menu)
    menu.AutomaticSize = Enum.AutomaticSize.Y
    for _, item in ipairs(items) do
        local btn = Utility.create("TextButton", {
            Size = UDim2.new(1, 0, 0, 30),
            BackgroundColor3 = theme.Background,
            BackgroundTransparency = 0.4,
            Text = "",
            AutoButtonColor = false,
            Parent = menu,
        })
        Utility.corner(6, btn)
        local lbl = Utility.create("TextLabel", {
            BackgroundTransparency = 1,
            Position = UDim2.fromOffset(10, 0),
            Size = UDim2.new(1, -20, 1, 0),
            Font = Enum.Font.GothamMedium,
            Text = item.Text,
            TextColor3 = item.Danger and theme.Error or theme.Text,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = btn,
        })
        btn.MouseButton1Down:Connect(function()
            if item.Callback then task.spawn(item.Callback) end
            menu.Visible = false
        end)
        btn.MouseEnter:Connect(function() Utility.tween(btn, 0.15, { BackgroundColor3 = item.Danger and theme.Error or theme.CardHover }) end)
        btn.MouseLeave:Connect(function() Utility.tween(btn, 0.15, { BackgroundColor3 = theme.Background }) end)
    end
    return {
        Instance = menu,
        Show = function(x, y) menu.Position = UDim2.fromOffset(x, y) menu.Visible = true end,
        Hide = function() menu.Visible = false end,
        Destroy = function() menu:Destroy() end,
    }
end

-- ═══════════════════════════════════════════════════════════════════════════
--   NOTIFICATION SYSTEM
-- ═══════════════════════════════════════════════════════════════════════════
local NotificationManager = {}
NotificationManager._queue = {}
NotificationManager._active = {}
NotificationManager._maxActive = 4
NotificationManager._screenGui = nil

function NotificationManager.init(screenGui)
    NotificationManager._screenGui = screenGui
end

local NotificationTypeStyles = {
    Success = { color = Color3.fromRGB(52, 211, 153), icon = "rbxassetid://12062113234" },
    Warning = { color = Color3.fromRGB(251, 191, 36), icon = "rbxassetid://12062107789" },
    Error   = { color = Color3.fromRGB(239, 68, 68),  icon = "rbxassetid://12062107789" },
    Info    = { color = Color3.fromRGB(59, 130, 246), icon = "rbxassetid://12062105567" },
    Loading = { color = Color3.fromRGB(167, 139, 250), icon = "rbxassetid://12062099988" },
}

function NotificationManager.notify(config)
    table.insert(NotificationManager._queue, config)
    NotificationManager._process()
end

function NotificationManager._process()
    while #NotificationManager._active < NotificationManager._maxActive and #NotificationManager._queue > 0 do
        local cfg = table.remove(NotificationManager._queue, 1)
        NotificationManager._spawn(cfg)
    end
end

function NotificationManager._spawn(config)
    local theme = ThemeManager.get()
    local style = NotificationTypeStyles[config.Type] or NotificationTypeStyles.Info
    local duration = config.Duration or 4
    -- Mobile: narrower notifications + respect safe area
    local isPhone = Utility.isPhone()
    local safe = Utility.getSafeArea()
    local notifW = isPhone and math.min(safe.Width - 16, 340) or 320
    local notifBottomMargin = isPhone and (safe.Bottom + 8) or 16
    local container = Utility.create("Frame", {
        Name = "Notification",
        Size = UDim2.fromOffset(notifW, 0),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = 0.05,
        Position = UDim2.new(1, notifW + 20, 1, -notifBottomMargin),
        AnchorPoint = Vector2.new(0, 1),
        ZIndex = 500,
        Parent = NotificationManager._screenGui,
    })
    Utility.corner(10, container)
    Utility.stroke(style.color, 1, container, 0.4)
    Utility.padding(12, 12, 14, 14, container)
    Utility.list(false, 6, container)
    container.AutomaticSize = Enum.AutomaticSize.Y
    -- Icon
    local headerRow = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 20),
        BackgroundTransparency = 1,
        Parent = container,
    })
    local icon = Utility.create("ImageLabel", {
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.fromOffset(0, 10),
        Size = UDim2.fromOffset(16, 16),
        BackgroundTransparency = 1,
        Image = style.icon,
        ImageColor3 = style.color,
        Parent = headerRow,
    })
    local title = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(24, 0),
        Size = UDim2.new(1, -60, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Title or "Notification",
        TextColor3 = theme.Text,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = headerRow,
    })
    local close = Utility.create("TextButton", {
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, 0, 0.5, 0),
        Size = UDim2.fromOffset(18, 18),
        BackgroundTransparency = 1,
        Text = "×",
        Font = Enum.Font.GothamBold,
        TextSize = 18,
        TextColor3 = theme.TextMuted,
        Parent = headerRow,
    })
    if config.Description then
        local desc = Utility.create("TextLabel", {
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 0),
            Font = Enum.Font.Gotham,
            Text = config.Description,
            TextColor3 = theme.TextMuted,
            TextSize = 12,
            TextWrapped = true,
            TextXAlignment = Enum.TextXAlignment.Left,
            AutomaticSize = Enum.AutomaticSize.Y,
            Parent = container,
        })
    end
    -- Progress bar (auto-dismiss countdown)
    local progress = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 2),
        BackgroundColor3 = style.color,
        BackgroundTransparency = 0.4,
        Parent = container,
    })
    Utility.corner(1, progress)
    -- Position stacked (mobile-aware: respect safe-area bottom + side)
    local index = #NotificationManager._active
    local sideMargin = isPhone and safe.Right or 16
    local bottomMargin = notifBottomMargin
    local function reposition()
        for i, n in ipairs(NotificationManager._active) do
            local targetY = -bottomMargin - (i - 1) * (n.Instance.AbsoluteSize.Y + 8)
            Utility.tween(n.Instance, 0.3, { Position = UDim2.new(1, -sideMargin, 1, targetY) }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
        end
    end
    table.insert(NotificationManager._active, { Instance = container, Config = config })
    -- Animate in
    task.spawn(function()
        Utility.tween(container, 0.4, { Position = UDim2.new(1, -sideMargin, 1, -bottomMargin - index * 80) }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
        reposition()
    end)
    -- Auto-dismiss
    local dismissed = false
    local function dismiss()
        if dismissed then return end
        dismissed = true
        Utility.tween(container, 0.35, { Position = UDim2.new(1, notifW + 20, container.Position.Y.Offset, 0) }, Enum.EasingDirection.In, Enum.EasingStyle.Quart)
        for i, n in ipairs(NotificationManager._active) do
            if n.Instance == container then table.remove(NotificationManager._active, i) break end
        end
        task.delay(0.35, function() container:Destroy() reposition() NotificationManager._process() end)
    end
    -- Dismiss button uses tap (mouse + touch unified)
    Utility.onTap(close, dismiss)
    if config.Type ~= "Loading" then
        Utility.tween(progress, duration, { Size = UDim2.new(0, 0, 0, 2) }, Enum.EasingDirection.Linear, Enum.EasingStyle.Linear)
        task.delay(duration, dismiss)
    end
end

-- ═══════════════════════════════════════════════════════════════════════════
--   COMMAND PALETTE (Ctrl + K)
-- ═══════════════════════════════════════════════════════════════════════════
local CommandPalette = {}
CommandPalette.Commands = {}
CommandPalette._screenGui = nil
CommandPalette._frame = nil

function CommandPalette.init(screenGui)
    CommandPalette._screenGui = screenGui
    -- Build palette UI
    local theme = ThemeManager.get()
    local overlay = Utility.create("TextButton", {
        Name = "PaletteOverlay",
        Size = UDim2.fromScale(1, 1),
        BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Visible = false,
        ZIndex = 900,
        Parent = screenGui,
    })
    -- Mobile-friendly palette size
    local isMobile = Utility.isMobile()
    local isPhone  = Utility.isPhone()
    local safe = Utility.getSafeArea()
    local panelW, panelH, panelTopGap
    if isPhone then
        panelW = safe.Width - 16
        panelH = math.min(safe.Height - 80, 420)
        panelTopGap = math.max(safe.Top, 16)
    elseif isMobile then
        panelW = math.min(560, safe.Width - 32)
        panelH = 420
        panelTopGap = 80
    else
        panelW = 560
        panelH = 420
        panelTopGap = 80
    end
    local panel = Utility.create("Frame", {
        Name = "Palette",
        AnchorPoint = Vector2.new(0.5, 0),
        Position = UDim2.new(0.5, 0, 0, panelTopGap),
        Size = UDim2.fromOffset(panelW, panelH),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = 0.05,
        Visible = false,
        ZIndex = 901,
        Parent = overlay,
    })
    Utility.corner(12, panel)
    Utility.stroke(theme.Border, 1, panel, 0.4)
    Utility.padding(0, 0, 0, 0, panel)
    -- Search bar
    local search = Utility.create("TextBox", {
        Size = UDim2.new(1, 0, 0, 52),
        BackgroundColor3 = theme.BackgroundAlt,
        BackgroundTransparency = 0,
        Font = Enum.Font.GothamMedium,
        PlaceholderText = "Type a command or search...",
        PlaceholderColor3 = theme.TextSubtle,
        Text = "",
        TextColor3 = theme.Text,
        TextSize = 15,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = panel,
    })
    Utility.padding(0, 0, 16, 16, search)
    Utility.corner(12, search)
    -- Results
    local results = Utility.create("ScrollingFrame", {
        Position = UDim2.new(0, 0, 0, 52),
        Size = UDim2.new(1, 0, 1, -52),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = panel,
    })
    Utility.padding(8, 8, 8, 8, results)
    Utility.list(false, 4, results)
    local function buildResults(query)
        for _, c in ipairs(results:GetChildren()) do
            if c:IsA("TextButton") then c:Destroy() end
        end
        local matches = {}
        for _, cmd in ipairs(CommandPalette.Commands) do
            local ok, score = Utility.fuzzyMatch(query, cmd.Title)
            if ok then table.insert(matches, { cmd = cmd, score = score }) end
        end
        table.sort(matches, function(a, b) return a.score > b.score end)
        for i, m in ipairs(matches) do
            if i > 12 then break end
            local cmd = m.cmd
            local item = Utility.create("TextButton", {
                Size = UDim2.new(1, 0, 0, 42),
                BackgroundColor3 = theme.Card,
                BackgroundTransparency = 0.2,
                Text = "",
                AutoButtonColor = false,
                Parent = results,
            })
            Utility.corner(8, item)
            local title = Utility.create("TextLabel", {
                BackgroundTransparency = 1,
                Position = UDim2.fromOffset(12, 0),
                Size = UDim2.new(1, -24, 0, 22),
                Font = Enum.Font.GothamMedium,
                Text = cmd.Title,
                TextColor3 = theme.Text,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = item,
            })
            local desc = Utility.create("TextLabel", {
                BackgroundTransparency = 1,
                Position = UDim2.fromOffset(12, 22),
                Size = UDim2.new(1, -24, 0, 16),
                Font = Enum.Font.Gotham,
                Text = cmd.Description or "",
                TextColor3 = theme.TextMuted,
                TextSize = 11,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = item,
            })
            item.MouseButton1Down:Connect(function()
                CommandPalette.hide()
                if cmd.Handler then task.spawn(cmd.Handler) end
            end)
            item.MouseEnter:Connect(function() Utility.tween(item, 0.15, { BackgroundColor3 = theme.CardHover }) end)
            item.MouseLeave:Connect(function() Utility.tween(item, 0.15, { BackgroundColor3 = theme.Card }) end)
        end
    end
    search:GetPropertyChangedSignal("Text"):Connect(function()
        buildResults(search.Text)
    end)
    -- Show / hide
    function CommandPalette.show()
        overlay.Visible = true
        panel.Visible = true
        overlay.BackgroundTransparency = 1
        Utility.tween(overlay, 0.2, { BackgroundTransparency = 0.5 })
        panel.Size = UDim2.fromOffset(panelW, 0)
        Utility.tween(panel, 0.3, { Size = UDim2.fromOffset(panelW, panelH) }, Enum.EasingDirection.Out, Enum.EasingStyle.Back)
        search.Text = ""
        buildResults("")
        task.wait(0.05)
        search:CaptureFocus()
    end
    function CommandPalette.hide()
        Utility.tween(panel, 0.2, { Size = UDim2.fromOffset(panelW, 0) }, Enum.EasingDirection.In, Enum.EasingStyle.Quart)
        Utility.tween(overlay, 0.2, { BackgroundTransparency = 1 })
        task.delay(0.2, function() overlay.Visible = false panel.Visible = false end)
    end
    -- Use tap (works on both mouse + touch) to dismiss when tapping outside panel
    Utility.onTap(overlay, CommandPalette.hide)
    -- Ctrl + K shortcut
    UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.K and (UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.RightControl)) then
            if overlay.Visible then CommandPalette.hide() else CommandPalette.show() end
        end
        if input.KeyCode == Enum.KeyCode.Escape and overlay.Visible then
            CommandPalette.hide()
        end
    end)
    CommandPalette._overlay = overlay
    CommandPalette._panel = panel
    CommandPalette._search = search
end

function CommandPalette.add(command)
    table.insert(CommandPalette.Commands, command)
end

-- ═══════════════════════════════════════════════════════════════════════════
--   SECTION
-- ═══════════════════════════════════════════════════════════════════════════
local Section = {}
Section.__index = Section

function Section.new(parent, config)
    local theme = ThemeManager.get()
    local self = setmetatable({}, Section)
    self.Config = config
    self.Components = {}
    local frame = Utility.create("Frame", {
        Name = config.Name or "Section",
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundColor3 = theme.Card,
        BackgroundTransparency = theme.CardTransparency,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = parent,
    })
    Utility.corner(10, frame)
    Utility.stroke(theme.Border, 1, frame, 0.3)
    Utility.padding(16, 16, 16, 16, frame)
    Utility.list(false, 8, frame)
    self.Instance = frame
    -- Header
    local header = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 22),
        BackgroundTransparency = 1,
        Parent = frame,
    })
    local title = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = config.Name or "Section",
        TextColor3 = theme.Text,
        TextSize = 15,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = header,
    })
    if config.Description then
        local desc = Utility.create("TextLabel", {
            BackgroundTransparency = 1,
            AnchorPoint = Vector2.new(1, 0),
            Position = UDim2.new(1, 0, 0, 4),
            Size = UDim2.new(0, 200, 1, 0),
            Font = Enum.Font.Gotham,
            Text = config.Description,
            TextColor3 = theme.TextMuted,
            TextSize = 11,
            TextXAlignment = Enum.TextXAlignment.Right,
            Parent = header,
        })
    end
    self.Header = header
    -- Content holder
    self.Content = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 0),
        BackgroundTransparency = 1,
        AutomaticSize = Enum.AutomaticSize.Y,
        Parent = frame,
    })
    Utility.list(false, 6, self.Content)
    -- Auto-register component creators
    for name, fn in pairs(Components) do
        self[name] = function(s, cfg)
            cfg = cfg or {}
            local api = fn(s.Content, cfg)
            table.insert(s.Components, api)
            return api
        end
    end
    return self
end

function Section:Add(cfg)
    return self:_addComponent(cfg)
end

-- ═══════════════════════════════════════════════════════════════════════════
--   TAB
-- ═══════════════════════════════════════════════════════════════════════════
local Tab = {}
Tab.__index = Tab

function Tab.new(window, config)
    local theme = ThemeManager.get()
    local self = setmetatable({}, Tab)
    self.Config = config
    self.Window = window
    self.Sections = {}
    -- Tab button (sidebar item)
    local button = Utility.create("TextButton", {
        Name = config.Name or "Tab",
        Size = UDim2.new(1, 0, 0, 36),
        BackgroundColor3 = theme.Background,
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Parent = window.SidebarList,
    })
    Utility.corner(8, button)
    local icon = Utility.create("ImageLabel", {
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.fromOffset(10, 18),
        Size = UDim2.fromOffset(18, 18),
        BackgroundTransparency = 1,
        Image = getIcon(config.Icon) or config.Icon or "",
        ImageColor3 = theme.TextMuted,
        Parent = button,
    })
    local label = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(36, 0),
        Size = UDim2.new(1, -48, 1, 0),
        Font = Enum.Font.GothamMedium,
        Text = config.Name or "Tab",
        TextColor3 = theme.TextMuted,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = button,
    })
    self.Button = button
    self.Icon = icon
    self.Label = label
    -- Page (content)
    local page = Utility.create("ScrollingFrame", {
        Name = config.Name or "Tab",
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 6,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Visible = false,
        Parent = window.ContentArea,
    })
    Utility.padding(0, 16, 0, 0, page)
    Utility.list(false, 12, page)
    self.Page = page
    -- Tab header
    local pageHeader = Utility.create("Frame", {
        Size = UDim2.new(1, 0, 0, 40),
        BackgroundTransparency = 1,
        Parent = page,
    })
    local pageTitle = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = config.Name or "Tab",
        TextColor3 = theme.Text,
        TextSize = 20,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = pageHeader,
    })
    self.PageTitle = pageTitle
    -- Sections layout (2 columns on desktop, 1 on mobile)
    local cols = Utility.isMobile() and 1 or (config.Columns or 1)
    local columns = {}
    for i = 1, cols do
        local col = Utility.create("Frame", {
            Size = UDim2.new(1 / cols, -((cols - 1) * 8) / cols, 0, 0),
            BackgroundTransparency = 1,
            AutomaticSize = Enum.AutomaticSize.Y,
            Parent = page,
        })
        Utility.list(false, 10, col)
        table.insert(columns, col)
    end
    self.Columns = columns
    -- Selection state
    self.Selected = false
    function self:Select()
        for _, t in ipairs(window.Tabs) do
            t:SetSelected(false)
        end
        self:SetSelected(true)
        window.CurrentTab = self
    end
    -- Use unified tap handler so it works on both mouse and touch
    Utility.onTap(button, function()
        self:Select()
        -- On mobile, close the sidebar drawer after selecting a tab
        if window._isMobile and window._sidebarOpen then
            window:SetSidebarOpen(false)
        end
    end)
    return self
end

function Tab:SetSelected(state)
    self.Selected = state
    local theme = ThemeManager.get()
    if state then
        Utility.tween(self.Button, 0.2, { BackgroundColor3 = theme.Primary, BackgroundTransparency = 0.85 })
        Utility.tween(self.Icon, 0.2, { ImageColor3 = theme.Primary })
        Utility.tween(self.Label, 0.2, { TextColor3 = theme.Primary })
        self.Page.Visible = true
        AnimationEngine.slideIn(self.Page, "Right")
    else
        Utility.tween(self.Button, 0.2, { BackgroundColor3 = theme.Background, BackgroundTransparency = 1 })
        Utility.tween(self.Icon, 0.2, { ImageColor3 = theme.TextMuted })
        Utility.tween(self.Label, 0.2, { TextColor3 = theme.TextMuted })
        self.Page.Visible = false
    end
end

function Tab:CreateSection(config)
    local col = self.Columns[(#self.Sections % #self.Columns) + 1]
    local s = Section.new(col, config)
    table.insert(self.Sections, s)
    return s
end

-- ═══════════════════════════════════════════════════════════════════════════
--   WINDOW
-- ═══════════════════════════════════════════════════════════════════════════
local Window = {}
Window.__index = Window

function Window.new(library, config)
    local theme = ThemeManager.get()
    local self = setmetatable({}, Window)
    self.Config = config
    self.Tabs = {}
    self.CurrentTab = nil
    self.Minimized = false
    self.Maximized = false
    self.Library = library
    -- ScreenGui (use SafeAreaCompat for notched devices)
    local screenGui = Utility.create("ScreenGui", {
        Name = "PremiumUI_" .. (config.Title or "Window"),
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        DisplayOrder = 999,
        IgnoreGuiInset = true,  -- Full-screen, no top-bar gap (we manage insets ourselves)
        Parent = getProtectedParent(),
    })
    self.ScreenGui = screenGui
    -- Initialize subsystems
    NotificationManager.init(screenGui)
    CommandPalette.init(screenGui)
    -- Platform detection
    local isMobile  = Utility.isMobile()
    local isPhone   = Utility.isPhone()
    local isTablet  = Utility.isTablet()
    local isTouch   = Utility.isTouch()
    -- Compute safe area (accounts for notches + top bar)
    local safe = Utility.getSafeArea()
    local vw, vh = Camera.ViewportSize.X, Camera.ViewportSize.Y
    -- Window dimensions
    local winW, winH, posX, posY
    if isPhone then
        -- Phone: full screen minus safe area + small margin
        winW = safe.Width
        winH = safe.Height
        posX = safe.Left
        posY = safe.Top
    elseif isTablet then
        -- Tablet: large centered window
        winW = math.min(vw - 32, 720)
        winH = math.min(vh - 32 - safe.Top, 720)
        posX = (vw - winW) / 2
        posY = safe.Top + ((vh - safe.Top) - winH) / 2
    else
        -- Desktop: as configured
        winW = config.Size and config.Size.X or 960
        winH = config.Size and config.Size.Y or 600
        posX = (vw - winW) / 2
        posY = (vh - winH) / 2
    end
    self._isMobile = isMobile
    self._isPhone  = isPhone
    -- Window shadow (hidden on phone — full screen, no shadow needed)
    local shadow
    if not isPhone then
        shadow = Utility.create("ImageLabel", {
            Name = "Shadow",
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.new(0.5, 0, 0.5, 4),
            Size = UDim2.new(1, 50, 1, 50),
            BackgroundTransparency = 1,
            Image = "rbxassetid://1316045217",
            ImageColor3 = Color3.fromRGB(0, 0, 0),
            ImageTransparency = 0.5,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(10, 10, 118, 118),
            ZIndex = 0,
            Parent = screenGui,
        })
    end
    -- Window frame (no rounded corners on phone — fills screen)
    local frame = Utility.create("Frame", {
        Name = "Window",
        Position = UDim2.fromOffset(posX, posY),
        Size = UDim2.fromOffset(winW, winH),
        BackgroundColor3 = theme.Background,
        BackgroundTransparency = theme.BackgroundTransparency,
        Parent = screenGui,
    })
    if not isPhone then
        Utility.corner(12, frame)
        Utility.stroke(theme.Border, 1, frame, 0.4)
    end
    -- Acrylic / blur backing
    local blur = Utility.create("Frame", {
        Size = UDim2.fromScale(1, 1),
        BackgroundColor3 = theme.Background,
        BackgroundTransparency = 0.6,
        BorderSizePixel = 0,
        ZIndex = -1,
        Parent = frame,
    })
    if not isPhone then Utility.corner(12, blur) end
    self.Frame = frame
    self.Shadow = shadow
    -- Title bar (taller on touch for easier tapping)
    local titleBarH = isTouch and 48 or 44
    local titleBar = Utility.create("Frame", {
        Name = "TitleBar",
        Size = UDim2.new(1, 0, 0, titleBarH),
        BackgroundColor3 = theme.BackgroundAlt,
        BackgroundTransparency = 0.2,
        Parent = frame,
    })
    if not isPhone then
        Utility.corner(12, titleBar)
        -- Mask lower corners of titlebar with a rectangle
        local mask = Utility.create("Frame", {
            AnchorPoint = Vector2.new(0.5, 1),
            Position = UDim2.new(0.5, 0, 1, 0),
            Size = UDim2.new(1, 0, 0, 14),
            BackgroundColor3 = theme.BackgroundAlt,
            BackgroundTransparency = 0.2,
            BorderSizePixel = 0,
            Parent = titleBar,
        })
    end
    Utility.padding(isPhone and 8 or 16, 0, isPhone and 8 or 16, 0, titleBar)
    -- Hamburger / sidebar toggle (mobile only — opens sidebar as overlay)
    local menuBtn
    if isMobile then
        menuBtn = Utility.create("TextButton", {
            AnchorPoint = Vector2.new(0, 0.5),
            Position = UDim2.fromOffset(0, 0.5),
            Size = UDim2.fromOffset(titleBarH, titleBarH),
            BackgroundTransparency = 1,
            Text = "",
            AutoButtonColor = false,
            Parent = titleBar,
        })
        menuBtn.Position = UDim2.fromOffset(0, titleBarH / 2)
        local menuIcon = Utility.create("ImageLabel", {
            AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.fromScale(0.5, 0.5),
            Size = UDim2.fromOffset(22, 22),
            BackgroundTransparency = 1,
            Image = "rbxassetid://12062111012",  -- hamburger / chevron
            ImageColor3 = theme.Text,
            Parent = menuBtn,
        })
    end
    local iconOffset = isMobile and titleBarH or 16
    local icon = Utility.create("ImageLabel", {
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.fromOffset(iconOffset, titleBarH / 2),
        Size = UDim2.fromOffset(20, 20),
        BackgroundTransparency = 1,
        Image = getIcon(config.Icon) or config.Icon or "",
        Parent = titleBar,
    })
    local title = Utility.create("TextLabel", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(iconOffset + 28, 0),
        Size = UDim2.new(1, -(iconOffset + 28 + (isMobile and 160 or 200)), 1, 0),
        Font = Enum.Font.GothamBold,
        Text = config.Title or "Premium Hub",
        TextColor3 = theme.Text,
        TextSize = isPhone and 14 or 15,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextTruncate = Enum.TextTruncate.AtEnd,
        Parent = titleBar,
    })
    self.Title = title
    -- Window controls
    -- On mobile: bigger touch targets (44pt min), and add a "command palette" button (no Ctrl+K)
    local ctrlCount = isMobile and 4 or 3
    local ctrlSize = isTouch and 36 or 28
    local controls = Utility.create("Frame", {
        AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, 0, 0.5, 0),
        Size = UDim2.fromOffset(ctrlCount * (ctrlSize + 4), ctrlSize),
        BackgroundTransparency = 1,
        Parent = titleBar,
    })
    Utility.list(true, 4, controls, Enum.HorizontalAlignment.Right)
    local function makeControl(text, callback, hoverColor, isIcon)
        local btn = Utility.create("TextButton", {
            Size = UDim2.fromOffset(ctrlSize, ctrlSize),
            BackgroundColor3 = theme.Card,
            BackgroundTransparency = 0.5,
            Text = isIcon and "" or text,
            Font = Enum.Font.GothamBold,
            TextSize = isTouch and 18 or 14,
            TextColor3 = theme.TextMuted,
            AutoButtonColor = false,
            Parent = controls,
        })
        Utility.corner(6, btn)
        if isIcon and text then
            local img = Utility.create("ImageLabel", {
                AnchorPoint = Vector2.new(0.5, 0.5),
                Position = UDim2.fromScale(0.5, 0.5),
                Size = UDim2.fromOffset(ctrlSize * 0.55, ctrlSize * 0.55),
                BackgroundTransparency = 1,
                Image = text,
                ImageColor3 = theme.TextMuted,
                Parent = btn,
            })
        end
        if not isTouch then
            btn.MouseEnter:Connect(function() Utility.tween(btn, 0.15, { BackgroundColor3 = hoverColor or theme.CardHover }) end)
            btn.MouseLeave:Connect(function() Utility.tween(btn, 0.15, { BackgroundColor3 = theme.Card, BackgroundTransparency = 0.5 }) end)
        end
        -- Use InputBegan instead of MouseButton1Down for unified mouse+touch
        Utility.onTap(btn, callback)
        return btn
    end
    -- Mobile: command palette button (substitutes for Ctrl+K)
    if isMobile then
        local cmdBtn = makeControl(Icons.Search, function()
            if CommandPalette._overlay then CommandPalette.show() end
        end, theme.Primary, true)
    end
    local minimizeBtn = makeControl("—", function() self:Minimize() end)
    if not isPhone then
        local maximizeBtn = makeControl("□", function() self:Maximize() end)
    end
    local closeBtn = makeControl("×", function() self:Close() end, theme.Error)
    -- Body
    local body = Utility.create("Frame", {
        Name = "Body",
        Position = UDim2.fromOffset(0, titleBarH),
        Size = UDim2.new(1, 0, 1, -titleBarH),
        BackgroundTransparency = 1,
        Parent = frame,
    })
    -- Sidebar
    -- Mobile: sidebar is an OVERLAY drawer that slides in from the left.
    -- Desktop: sidebar is inline (always visible).
    local sidebarW = isMobile and math.min(math.max(vw * 0.7, 240), 280) or 220
    local sidebarParent = body
    local sidebarBackdrop
    if isMobile then
        -- Backdrop dims the content when sidebar is open
        sidebarBackdrop = Utility.create("TextButton", {
            Name = "SidebarBackdrop",
            Size = UDim2.fromScale(1, 1),
            BackgroundColor3 = Color3.fromRGB(0, 0, 0),
            BackgroundTransparency = 1,
            Text = "",
            AutoButtonColor = false,
            Visible = false,
            ZIndex = 50,
            Parent = body,
        })
        Utility.onTap(sidebarBackdrop, function()
            self:SetSidebarOpen(false)
        end)
    end
    local sidebar = Utility.create("Frame", {
        Name = "Sidebar",
        Size = UDim2.fromOffset(sidebarW, 1),
        BackgroundColor3 = theme.Sidebar,
        BackgroundTransparency = 0.1,
        Parent = sidebarParent,
        ZIndex = isMobile and 51 or 1,
    })
    if isMobile then
        sidebar.Position = UDim2.fromOffset(-sidebarW, 0)  -- hidden off-screen
        sidebar.Visible = false
    end
    if not isPhone then Utility.corner(12, sidebar) end
    -- Mask right corners (desktop only)
    if not isMobile then
        local sidebarMask = Utility.create("Frame", {
            AnchorPoint = Vector2.new(1, 0.5),
            Position = UDim2.new(1, 0, 0.5, 0),
            Size = UDim2.new(0, 14, 1, 0),
            BackgroundColor3 = theme.Sidebar,
            BackgroundTransparency = 0.1,
            BorderSizePixel = 0,
            Parent = sidebar,
        })
    end
    Utility.padding(8, 8, 8, 8, sidebar)
    -- Search box (desktop + tablet, not on phone — saves space)
    if not isPhone then
        local searchBox = Utility.create("TextBox", {
            Name = "Search",
            Size = UDim2.new(1, 0, 0, isTouch and 40 or 32),
            BackgroundColor3 = theme.Card,
            BackgroundTransparency = 0.4,
            Font = Enum.Font.GothamMedium,
            PlaceholderText = "Search tabs...",
            PlaceholderColor3 = theme.TextSubtle,
            Text = "",
            TextColor3 = theme.Text,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = sidebar,
        })
        Utility.padding(0, 0, 28, 12, searchBox)
        Utility.corner(8, searchBox)
        Utility.stroke(theme.Border, 1, searchBox, 0.4)
        local searchIcon = Utility.create("ImageLabel", {
            AnchorPoint = Vector2.new(1, 0.5),
            Position = UDim2.new(1, -8, 0.5, 0),
            Size = UDim2.fromOffset(14, 14),
            BackgroundTransparency = 1,
            Image = Icons.Search,
            ImageColor3 = theme.TextSubtle,
            Parent = searchBox,
        })
        searchBox:GetPropertyChangedSignal("Text"):Connect(function()
            local q = string.lower(searchBox.Text)
            for _, t in ipairs(self.Tabs) do
                local match = string.find(string.lower(t.Config.Name or ""), q, 1, true)
                t.Button.Visible = match ~= nil
            end
        end)
        self.Search = searchBox
    end
    -- Tab list
    local sidebarList = Utility.create("ScrollingFrame", {
        Name = "TabList",
        Size = UDim2.new(1, 0, 1, isPhone and -16 or -48),
        Position = UDim2.fromOffset(0, isPhone and 8 or 48),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        ScrollBarImageColor3 = theme.TextMuted,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Parent = sidebar,
    })
    Utility.padding(0, 8, 0, 0, sidebarList)
    Utility.list(false, 4, sidebarList)
    self.SidebarList = sidebarList
    self.Sidebar = sidebar
    self.SidebarBackdrop = sidebarBackdrop
    self._sidebarW = sidebarW
    -- Vertical separator (desktop only — mobile sidebar is overlay)
    if not isMobile then
        local sep = Utility.create("Frame", {
            Position = UDim2.fromOffset(sidebarW, 0),
            Size = UDim2.fromOffset(1, 1),
            BackgroundColor3 = theme.Border,
            BackgroundTransparency = 0.4,
            BorderSizePixel = 0,
            Parent = body,
        })
    end
    -- Content area
    local contentOffset = isMobile and 0 or (sidebarW + 1)
    local content = Utility.create("Frame", {
        Name = "Content",
        Position = UDim2.fromOffset(contentOffset, 0),
        Size = UDim2.new(1, -contentOffset, 1, 0),
        BackgroundTransparency = 1,
        Parent = body,
    })
    -- Tighter padding on phone to maximize space
    local pad = isPhone and 12 or 20
    Utility.padding(pad, pad, pad, pad, content)
    self.ContentArea = content
    -- Sidebar toggle behavior (mobile)
    if isMobile then
        local function toggleSidebar()
            self:SetSidebarOpen(not self._sidebarOpen)
        end
        Utility.onTap(menuBtn, toggleSidebar)
    end
    -- Drag system (disabled on phone — window fills the screen, no point dragging)
    if not isPhone then
        Utility.makeDraggable(frame, titleBar)
    end
    -- Resize handle (desktop only)
    if not isMobile then
        local resize = Utility.create("TextButton", {
            AnchorPoint = Vector2.new(1, 1),
            Position = UDim2.new(1, -4, 1, -4),
            Size = UDim2.fromOffset(20, 20),
            BackgroundTransparency = 1,
            Text = "",
            AutoButtonColor = false,
            Parent = frame,
        })
        local resizing = false
        local startSize, startPos, resizeInput
        resize.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                resizing = true
                startSize = frame.AbsoluteSize
                startPos = input.Position
                resizeInput = input
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if resizing and (input == resizeInput or
                (input.UserInputType == Enum.UserInputType.MouseMovement and resizeInput and resizeInput.UserInputType == Enum.UserInputType.MouseButton1)) then
                local delta = input.Position - startPos
                frame.Size = UDim2.fromOffset(
                    math.max(640, startSize.X + delta.X),
                    math.max(420, startSize.Y + delta.Y)
                )
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if input == resizeInput or input.UserInputType == Enum.UserInputType.MouseButton1 then
                resizing = false
                resizeInput = nil
            end
        end)
    end
    -- Open animation
    if isPhone then
        -- Slide in from bottom on phone
        frame.Position = UDim2.fromOffset(posX, vh)
        Utility.tween(frame, 0.35, { Position = UDim2.fromOffset(posX, posY) },
            Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
    else
        frame.Size = UDim2.fromOffset(0, 0)
        frame.Position = UDim2.fromOffset(vw / 2, vh / 2)
        Utility.tween(frame, 0.4, {
            Size = UDim2.fromOffset(winW, winH),
            Position = UDim2.fromOffset(posX, posY)
        }, Enum.EasingDirection.Out, Enum.EasingStyle.Back)
        if shadow then Utility.tween(shadow, 0.4, { ImageTransparency = 0.3 }) end
    end
    -- Resize handler (keep shadow synced)
    frame:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
        if shadow then shadow.Size = UDim2.new(1, 50, 1, 50) end
    end)
    -- Viewport resize / rotation handler (critical for mobile)
    local function handleViewportChange()
        local newVp = Camera.ViewportSize
        local newSafe = Utility.getSafeArea()
        if isPhone then
            -- Re-fit window to new safe area
            frame.Size = UDim2.fromOffset(newSafe.Width, newSafe.Height)
            frame.Position = UDim2.fromOffset(newSafe.Left, newSafe.Top)
        elseif isMobile then
            -- Tablet: re-center
            local newW = math.min(newVp.X - 32, 720)
            local newH = math.min(newVp.Y - 32 - newSafe.Top, 720)
            frame.Size = UDim2.fromOffset(newW, newH)
            frame.Position = UDim2.fromOffset((newVp.X - newW) / 2, newSafe.Top + ((newVp.Y - newSafe.Top) - newH) / 2)
        else
            -- Desktop: clamp window position to new viewport
            local ap = frame.AbsolutePosition
            local as = frame.AbsoluteSize
            local newX = clamp(ap.X, 0, math.max(0, newVp.X - as.X))
            local newY = clamp(ap.Y, newSafe.Top, math.max(newSafe.Top, newVp.Y - as.Y))
            frame.Position = UDim2.fromOffset(newX, newY)
        end
    end
    Camera:GetPropertyChangedSignal("ViewportSize"):Connect(handleViewportChange)
    -- Also re-fire when device orientation changes
    UserInputService.DeviceRotationChanged:Connect(handleViewportChange)
    -- Keyboard shortcuts
    UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.Escape and self.Visible then
            if self._sidebarOpen then
                self:SetSidebarOpen(false)
            end
        end
    end)
    self.Visible = true
    self._sidebarOpen = false
    -- Register default command palette commands
    CommandPalette.add({
        Title = "Toggle Theme",
        Description = "Switch between Dark and Light",
        Handler = function()
            local cur = ThemeManager.Current.Name
            ThemeManager.setTheme(cur == "Dark" and "Light" or "Dark")
        end,
    })
    CommandPalette.add({
        Title = "Close Window",
        Description = "Close the current window",
        Handler = function() self:Close() end,
    })
    CommandPalette.add({
        Title = "Toggle Sidebar",
        Description = "Show or hide the tab sidebar",
        Handler = function() self:SetSidebarOpen(not self._sidebarOpen) end,
    })
    return self
end

-- Mobile sidebar toggle (overlay drawer)
function Window:SetSidebarOpen(state)
    if not self._isMobile then return end
    if state then
        self.Sidebar.Visible = true
        if self.SidebarBackdrop then
            self.SidebarBackdrop.Visible = true
            self.SidebarBackdrop.BackgroundTransparency = 1
            Utility.tween(self.SidebarBackdrop, 0.25, { BackgroundTransparency = 0.5 })
        end
        Utility.tween(self.Sidebar, 0.3, { Position = UDim2.fromOffset(0, 0) },
            Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
    else
        Utility.tween(self.Sidebar, 0.25, { Position = UDim2.fromOffset(-self._sidebarW, 0) },
            Enum.EasingDirection.In, Enum.EasingStyle.Quart)
        if self.SidebarBackdrop then
            Utility.tween(self.SidebarBackdrop, 0.25, { BackgroundTransparency = 1 })
        end
        task.delay(0.25, function()
            self.Sidebar.Visible = false
            if self.SidebarBackdrop then self.SidebarBackdrop.Visible = false end
        end)
    end
    -- Track state in closure (use a simple flag on self)
    self._sidebarOpen = state
end

function Window:CreateTab(config)
    local t = Tab.new(self, config)
    table.insert(self.Tabs, t)
    if #self.Tabs == 1 then
        t:Select()
    end
    -- Register in command palette
    CommandPalette.add({
        Title = "Go to " .. (config.Name or "Tab"),
        Description = "Switch to " .. (config.Name or "Tab") .. " tab",
        Handler = function() t:Select() end,
    })
    return t
end

function Window:Minimize()
    if self.Minimized then return end
    self.Minimized = true
    local cur = self.Frame.AbsoluteSize
    Utility.tween(self.Frame, 0.3, {
        Size = UDim2.fromOffset(cur.X, 44),
    }, Enum.EasingDirection.In, Enum.EasingStyle.Quart)
    self.Frame.ClipsDescendants = true
end

function Window:Maximize()
    local safe = Utility.getSafeArea()
    if self.Maximized then
        -- Restore
        self.Maximized = false
        if self._prevSize and self._prevPos then
            Utility.tween(self.Frame, 0.3, {
                Size = UDim2.fromOffset(self._prevSize.X, self._prevSize.Y),
                Position = UDim2.fromOffset(self._prevPos.X, self._prevPos.Y)
            }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
        else
            local sw = self.Config.Size and self.Config.Size.X or 960
            local sh = self.Config.Size and self.Config.Size.Y or 600
            Utility.tween(self.Frame, 0.3, {
                Size = UDim2.fromOffset(sw, sh),
                Position = UDim2.fromOffset((Camera.ViewportSize.X - sw) / 2, (Camera.ViewportSize.Y - sh) / 2)
            }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
        end
    else
        self.Maximized = true
        self._prevSize = self.Frame.AbsoluteSize
        self._prevPos = self.Frame.AbsolutePosition
        Utility.tween(self.Frame, 0.3, {
            Size = UDim2.fromOffset(safe.Width, safe.Height),
            Position = UDim2.fromOffset(safe.Left, safe.Top)
        }, Enum.EasingDirection.Out, Enum.EasingStyle.Quart)
    end
end

function Window:Close()
    if self._isPhone then
        -- Slide out to bottom on phone
        Utility.tween(self.Frame, 0.25, {
            Position = UDim2.fromOffset(self.Frame.AbsolutePosition.X, Camera.ViewportSize.Y)
        }, Enum.EasingDirection.In, Enum.EasingStyle.Quart)
    else
        Utility.tween(self.Frame, 0.25, {
            Size = UDim2.fromOffset(0, 0),
            Position = UDim2.fromOffset(self.Frame.AbsolutePosition.X + self.Frame.AbsoluteSize.X / 2, self.Frame.AbsolutePosition.Y + self.Frame.AbsoluteSize.Y / 2)
        }, Enum.EasingDirection.In, Enum.EasingStyle.Quart)
        if self.Shadow then Utility.tween(self.Shadow, 0.25, { ImageTransparency = 1 }) end
    end
    task.delay(0.25, function()
        self.ScreenGui:Destroy()
    end)
end

function Window:Notify(config)
    NotificationManager.notify(config)
end

function Window:SetTheme(themeName)
    ThemeManager.setTheme(themeName)
end

function Window:SetTitle(title)
    self.Title.Text = title
end

function Window:Toggle()
    if self.Frame.Visible then
        self.Frame.Visible = false
        if self.Shadow then self.Shadow.Visible = false end
    else
        self.Frame.Visible = true
        if self.Shadow then self.Shadow.Visible = true end
    end
end

-- ═══════════════════════════════════════════════════════════════════════════
--   MAIN LIBRARY
-- ═══════════════════════════════════════════════════════════════════════════
local Library = {}

Library.Version = "1.0.0"
Library.Signal = Signal
Library.Spring = Spring
Library.Utility = Utility
Library.ThemeManager = ThemeManager
Library.AnimationEngine = AnimationEngine
Library.NotificationManager = NotificationManager
Library.CommandPalette = CommandPalette
Library.Components = Components

function Library:CreateWindow(config)
    config = config or {}
    if config.Theme then
        ThemeManager.setTheme(config.Theme)
    end
    return Window.new(self, config)
end

function Library:SetTheme(name)
    ThemeManager.setTheme(name)
end

function Library:RegisterTheme(name, theme)
    ThemeManager.registerTheme(name, theme)
end

function Library:Notify(config)
    NotificationManager.notify(config)
end

function Library:AddCommand(cmd)
    CommandPalette.add(cmd)
end

-- Top-level notification shortcut (creates a temp screen gui if needed)
local _globalGui
function Library:GlobalNotify(config)
    if not _globalGui or not _globalGui.Parent then
        _globalGui = Utility.create("ScreenGui", {
            Name = "PremiumUI_Notifications",
            ResetOnSpawn = false,
            Parent = getProtectedParent(),
        })
        NotificationManager.init(_globalGui)
    end
    NotificationManager.notify(config)
end
-- ═══════════════════════════════════════════════════════════════════════════
--   AUTO-RUN DEMO HUB
--   ─────────────────────────────────────────────────────────────────────────
--   When you call  loadstring(src)()  in Delta, this block runs automatically
--   and builds a sample hub showing every component. To skip the demo and use
--   the library programmatically, set  _G.PremiumUI_SkipDemo = true  BEFORE
--   calling loadstring. The library is also returned, so:
--       local Library = loadstring(src)()   -- runs demo AND returns Library
--       _G.PremiumUI_SkipDemo = true
--       local Library = loadstring(src)()   -- skips demo, just returns Library
-- ═══════════════════════════════════════════════════════════════════════════

-- Expose globally so re-runs / external scripts can find it
_G.PremiumUI = Library
if getgenv then getgenv().PremiumUI = Library end

-- Run the demo unless explicitly skipped
if not _G.PremiumUI_SkipDemo then
    -- Prevent duplicate demo windows if the script is run twice
    if not _G.PremiumUI_DemoWindow or not _G.PremiumUI_DemoWindow.ScreenGui or not _G.PremiumUI_DemoWindow.ScreenGui.Parent then
        local Window = Library:CreateWindow({
            Title       = "Premium Hub",
            Icon        = "rbxassetid://12062096413",
            Theme       = "Midnight",
            Size        = Vector2.new(980, 620),
        })
        _G.PremiumUI_DemoWindow = Window

        -- ─── Combat Tab ───────────────────────────────────────────────────
        local CombatTab = Window:CreateTab({ Name = "Combat", Icon = "Sword" })

        local aimSection = CombatTab:CreateSection({ Name = "Aimbot", Description = "Auto-targeting" })
        aimSection:Toggle({
            Name     = "Enable Aimbot",
            Default  = false,
            Callback = function(v) print("[Combat] Aimbot:", v) end,
        })
        aimSection:Slider({
            Name     = "FOV",
            Min      = 30, Max = 500, Default = 120, Step = 5,
            Suffix   = "°",
            Callback = function(v) print("[Combat] FOV:", v) end,
        })
        aimSection:Dropdown({
            Name     = "Target Part",
            Options  = { "Head", "Torso", "Neck", "Pelvis" },
            Default  = "Head",
            Callback = function(v) print("[Combat] Target:", v) end,
        })
        aimSection:Keybind({
            Name      = "Trigger",
            Default   = Enum.KeyCode.E,
            OnPress   = function() print("[Combat] Aim triggered") end,
            Callback  = function(key) print("[Combat] Bound to:", key.Name) end,
        })
        aimSection:Button({
            Name     = "Refresh Players",
            Primary  = true,
            Callback = function()
                Window:Notify({ Type = "Success", Title = "Refreshed", Description = "Player list updated", Duration = 3 })
            end,
        })

        local silentSection = CombatTab:CreateSection({ Name = "Silent Aim" })
        silentSection:Toggle({ Name = "Enabled", Default = false })
        silentSection:Checkbox({ Name = "Visible Check", Default = true })
        silentSection:Checkbox({ Name = "Wall Check", Default = false })

        -- ─── Visuals Tab ──────────────────────────────────────────────────
        local VisualsTab = Window:CreateTab({ Name = "Visuals", Icon = "Shield" })
        local espSection = VisualsTab:CreateSection({ Name = "ESP" })

        espSection:Toggle({ Name = "Box ESP", Default = true })
        espSection:Toggle({ Name = "Name ESP", Default = false })
        espSection:Toggle({ Name = "Health Bar", Default = true })
        espSection:ColorPicker({ Name = "Color", Default = Color3.fromRGB(255, 80, 80) })
        espSection:Slider({ Name = "Thickness", Min = 1, Max = 5, Default = 2, Step = 1 })

        -- ─── Player Tab ───────────────────────────────────────────────────
        local PlayerTab = Window:CreateTab({ Name = "Player", Icon = "Player" })
        local movementSection = PlayerTab:CreateSection({ Name = "Movement" })
        movementSection:Slider({ Name = "WalkSpeed", Min = 16, Max = 250, Default = 16, Step = 1 })
        movementSection:Slider({ Name = "JumpPower", Min = 50, Max = 500, Default = 50, Step = 5 })
        movementSection:Toggle({ Name = "Fly", Default = false })
        movementSection:Keybind({ Name = "Fly Toggle", Default = Enum.KeyCode.F })

        -- ─── Settings Tab ─────────────────────────────────────────────────
        local SettingsTab = Window:CreateTab({ Name = "Settings", Icon = "Settings" })
        local themeSection = SettingsTab:CreateSection({ Name = "Theme" })

        themeSection:Button({
            Name     = "Dark",
            Callback = function() Library:SetTheme("Dark") end,
        })
        themeSection:Button({
            Name     = "Light",
            Callback = function() Library:SetTheme("Light") end,
        })
        themeSection:Button({
            Name     = "Midnight",
            Primary  = true,
            Callback = function() Library:SetTheme("Midnight") end,
        })

        local aboutSection = SettingsTab:CreateSection({ Name = "About" })
        aboutSection:Paragraph({
            Text = "<b>Premium UI Library</b><br/>Version 1.0.0<br/><br/>"
                .. "A next-generation Roblox GUI framework with glassmorphism, "
                .. "spring animations, and full mobile + desktop support."
        })
        aboutSection:Label({ Text = "PC: Press Ctrl + K for command palette" })
        aboutSection:Label({ Text = "Mobile: Tap the search icon for commands" })
        aboutSection:Label({ Text = "Mobile: Tap the menu icon to open tabs" })
        aboutSection:Label({ Text = "Press Esc to close dialogs / sidebar" })

        -- ─── Notifications ────────────────────────────────────────────────
        Window:Notify({
            Type        = "Success",
            Title       = "Premium Hub Loaded",
            Description = "Welcome! Check the title bar for commands.",
            Duration    = 5,
        })
        Window:Notify({
            Type        = "Info",
            Title       = "Mobile Friendly",
            Description = "This UI auto-adapts to your device.",
            Duration    = 5,
        })

        -- ─── Command Palette Commands ─────────────────────────────────────
        Library:AddCommand({
            Title       = "Toggle Theme",
            Description = "Switch between Dark and Light",
            Handler     = function()
                local cur = Library.ThemeManager.Current.Name
                Library:SetTheme(cur == "Dark" and "Light" or "Dark")
            end,
        })
        Library:AddCommand({
            Title       = "Show Test Notification",
            Description = "Display a sample notification",
            Handler     = function()
                Library:GlobalNotify({
                    Type        = "Warning",
                    Title       = "Test",
                    Description = "Triggered from command palette",
                })
            end,
        })
        Library:AddCommand({
            Title       = "Close Window",
            Description = "Closes the demo window",
            Handler     = function() Window:Close() end,
        })
        Library:AddCommand({
            Title       = "Reload UI",
            Description = "Closes and re-opens the window",
            Handler     = function()
                _G.PremiumUI_DemoWindow = nil
                Window:Close()
                task.wait(0.3)
                -- Re-trigger the demo by re-running the file (user-side)
            end,
        })

        print("[PremiumUI] Demo hub loaded. Theme: " .. Library.ThemeManager.Current.Name)
        print("[PremiumUI] Platform: " .. (Library.Utility.isMobile() and "Mobile" or "Desktop"))
    end
end

return Library
