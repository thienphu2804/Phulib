-- PhuLib v1.0
-- Minimal, mobile-friendly UI library (Rayfield-like)
-- Usage:
-- local PhuLib = loadstring(game:HttpGet("raw-link-to-PhuLib.lua"))()
-- local win = PhuLib:CreateWindow({Name="Phú Hub"})
-- local tab = win:CreateTab("Main")
-- tab:CreateToggle({Name="Inf Jump", CurrentValue=false, Callback=function(v) print(v) end})

local PhuLib = {}
PhuLib.__index = PhuLib

-- Utilities
local function new(cls, props)
    local obj = Instance.new(cls)
    if props then
        for k,v in pairs(props) do obj[k] = v end
    end
    return obj
end

local function isTouch()
    local UserInputService = game:GetService("UserInputService")
    return UserInputService.TouchEnabled or UserInputService:GetPlatform() == Enum.Platform.Android or UserInputService:GetPlatform() == Enum.Platform.IOS
end

-- Core Gui container (safe fallback)
local CoreGui = (game:GetService("CoreGui"))
local PhuScreen = CoreGui:FindFirstChild("PhuLib_ScreenGui")
if not PhuScreen then
    PhuScreen = new("ScreenGui", {Name = "PhuLib_ScreenGui", ResetOnSpawn = false, ZIndexBehavior = Enum.ZIndexBehavior.Sibling})
    PhuScreen.Parent = CoreGui
end

-- Basic style
local STYLE = {
    WindowSize = UDim2.new(0, 540, 0, 380),
    WindowColor = Color3.fromRGB(24,24,24),
    AccentColor = Color3.fromRGB(0,153,255),
    TextColor = Color3.fromRGB(240,240,240),
    Font = Enum.Font.GothamSemibold,
    CornerRadius = UDim.new(0,10),
}

-- helper: create rounded frame with borderless look
local function createFrame(parent, size, pos)
    local f = new("Frame", {
        Parent = parent,
        BackgroundColor3 = STYLE.WindowColor,
        Size = size,
        Position = pos,
        BorderSizePixel = 0,
        Active = true
    })
    local uc = new("UICorner", {Parent = f, CornerRadius = STYLE.CornerRadius})
    return f
end

-- draggable util
local function makeDraggable(frame, handle)
    local UserInputService = game:GetService("UserInputService")
    local dragging, dragInput, dragStart, startPos
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            local screen = frame.Parent
            local absolute = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            frame.Position = absolute
        end
    end)
end

-- Window class
local Window = {}
Window.__index = Window

function PhuLib:CreateWindow(opts)
    opts = opts or {}
    local title = opts.Name or "PhuLib Window"

    local container = createFrame(PhuScreen, opts.Size or STYLE.WindowSize, opts.Position or UDim2.new(0.2,0,0.2,0))
    container.Name = "PhuLib_Window"
    container.AnchorPoint = Vector2.new(0,0)

    -- Header
    local header = new("Frame", {
        Parent = container,
        BackgroundTransparency = 1,
        Size = UDim2.new(1,0,0,38),
        Position = UDim2.new(0,0,0,0)
    })
    local titleLabel = new("TextLabel", {
        Parent = header,
        Text = title,
        Size = UDim2.new(1,-80,1,0),
        Position = UDim2.new(0,10,0,0),
        BackgroundTransparency = 1,
        TextColor3 = STYLE.TextColor,
        Font = STYLE.Font,
        TextSize = 18,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    -- close/minimize
    local btnClose = new("TextButton", {
        Parent = header,
        Size = UDim2.new(0,28,0,28),
        Position = UDim2.new(1,-36,0,5),
        Text = "✕",
        BackgroundTransparency = 0,
        BackgroundColor3 = Color3.fromRGB(40,40,40),
        TextColor3 = STYLE.TextColor,
        BorderSizePixel = 0,
        AutoButtonColor = true
    })
    new("UICorner", {Parent = btnClose, CornerRadius = UDim.new(0,6)})

    local btnMin = new("TextButton", {
        Parent = header,
        Size = UDim2.new(0,28,0,28),
        Position = UDim2.new(1,-72,0,5),
        Text = "—",
        BackgroundTransparency = 0,
        BackgroundColor3 = Color3.fromRGB(40,40,40),
        TextColor3 = STYLE.TextColor,
        BorderSizePixel = 0,
        AutoButtonColor = true
    })
    new("UICorner", {Parent = btnMin, CornerRadius = UDim.new(0,6)})

    -- body: left tablist & right content
    local left = new("Frame", {Parent = container, Size = UDim2.new(0,140,1,-50), Position = UDim2.new(0,0,0,38), BackgroundTransparency = 1})
    local right = new("Frame", {Parent = container, Size = UDim2.new(1,-150,1,-50), Position = UDim2.new(0,150,0,38), BackgroundTransparency = 1})

    -- left scroll for tabs
    local tabList = new("ScrollingFrame", {
        Parent = left,
        Size = UDim2.new(1, -10, 1, 0),
        Position = UDim2.new(0,5,0,0),
        BackgroundTransparency = 1,
        ScrollBarImageColor3 = Color3.fromRGB(60,60,60),
        CanvasSize = UDim2.new(0,0,0,0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y
    })
    tabList.VerticalScrollBarInset = Enum.ScrollBarInset.ScrollBar

    -- right content scrolling
    local content = new("ScrollingFrame", {
        Parent = right,
        Size = UDim2.new(1, -10, 1, 0),
        Position = UDim2.new(0,5,0,0),
        BackgroundTransparency = 1,
        ScrollBarImageColor3 = Color3.fromRGB(60,60,60),
        CanvasSize = UDim2.new(0,0,0,0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y
    })
    content.VerticalScrollBarInset = Enum.ScrollBarInset.ScrollBar

    -- store tabs and content frames
    local win = setmetatable({
        Container = container,
        Header = header,
        TabList = tabList,
        Content = content,
        Tabs = {},
        ActiveTab = nil,
        Minimized = false
    }, Window)

    -- behavior
    btnClose.MouseButton1Click:Connect(function()
        container:Destroy()
    end)
    btnMin.MouseButton1Click:Connect(function()
        win.Minimized = not win.Minimized
        if win.Minimized then
            container.Size = UDim2.new(container.Size.X.Scale, container.Size.X.Offset, 0, 38)
            content.Visible = false
            left.Visible = false
        else
            container.Size = opts.Size or STYLE.WindowSize
            content.Visible = true
            left.Visible = true
        end
    end)

    -- draggable
    makeDraggable(container, header)

    -- API: CreateTab
    function win:CreateTab(name, icon)
        local tabBtn = new("TextButton", {
            Parent = tabList,
            Size = UDim2.new(1,0,0,40),
            BackgroundTransparency = 0,
            BackgroundColor3 = Color3.fromRGB(35,35,35),
            BorderSizePixel = 0,
            Text = "  "..tostring(name),
            TextColor3 = STYLE.TextColor,
            Font = STYLE.Font,
            TextSize = 14,
            AutoButtonColor = true
        })
        new("UICorner", {Parent = tabBtn, CornerRadius = UDim.new(0,6)})

        local tabContent = new("Frame", {
            Parent = content,
            Size = UDim2.new(1,0,0,10),
            BackgroundTransparency = 1,
            LayoutOrder = #self.Tabs + 1
        })
        -- layout inside content
        local layout = new("UIListLayout", {Parent = tabContent, SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0,8)})
        tabContent.BackgroundTransparency = 1

        local tabObj = {
            Name = name,
            Button = tabBtn,
            Content = tabContent,
            Elements = {}
        }

        -- switching
        tabBtn.MouseButton1Click:Connect(function()
            -- hide others
            for i,v in pairs(self.Tabs) do
                v.Button.BackgroundColor3 = Color3.fromRGB(35,35,35)
                v.Content.Visible = false
            end
            tabBtn.BackgroundColor3 = STYLE.AccentColor
            tabContent.Visible = true
            self.ActiveTab = tabObj
        end)

        -- if first tab -> activate
        if #self.Tabs == 0 then
            tabBtn.BackgroundColor3 = STYLE.AccentColor
            tabContent.Visible = true
            self.ActiveTab = tabObj
        else
            tabContent.Visible = false
        end

        -- helper to create element wrappers
        local function elementTemplate(nameText)
            local frame = new("Frame", {Parent = tabContent, Size = UDim2.new(1, -20, 0, 40), BackgroundTransparency = 1})
            local label = new("TextLabel", {
                Parent = frame,
                Text = tostring(nameText),
                Size = UDim2.new(0.5,0,1,0),
                Position = UDim2.new(0,8,0,0),
                BackgroundTransparency = 1,
                TextColor3 = STYLE.TextColor,
                Font = STYLE.Font,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })
            return frame, label
        end

        -- API on tabObj
        function tabObj:CreateLabel(opt)
            opt = opt or {}
            local frame,label = elementTemplate(opt.Name or "")
            label.Font = STYLE.Font
            label.TextSize = opt.TextSize or 14
            local value = new("TextLabel", {Parent = frame, Size = UDim2.new(0.45, -10, 1, 0), Position = UDim2.new(0.5, 0, 0, 0), BackgroundTransparency = 1, Text = opt.Value or "", TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Right})
            return {Frame = frame, Label = label, Value = value}
        end

        function tabObj:CreateButton(opt)
            opt = opt or {}
            local frame,label = elementTemplate(opt.Name or "Button")
            label.Text = ""
            local btn = new("TextButton", {Parent = frame, Size = UDim2.new(1, -16, 1, 0), Position = UDim2.new(0,8,0,0), Text = opt.Name or "Button", BackgroundColor3 = Color3.fromRGB(45,45,45), TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 14, BorderSizePixel = 0})
            new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,8)})
            btn.MouseButton1Click:Connect(function()
                pcall(function() if opt.Callback then opt.Callback() end end)
            end)
            table.insert(self.Elements, btn)
            return btn
        end

        function tabObj:CreateToggle(opt)
            opt = opt or {}
            local frame,label = elementTemplate(opt.Name or "Toggle")
            label.Text = opt.Name or "Toggle"
            local toggle = new("TextButton", {Parent = frame, Size = UDim2.new(0,60,0,26), Position = UDim2.new(1, -70, 0.5, -13), BackgroundColor3 = Color3.fromRGB(45,45,45), Text = (opt.CurrentValue and "ON") or "OFF", TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 13, BorderSizePixel = 0})
            new("UICorner", {Parent = toggle, CornerRadius = UDim.new(0,8)})
            local state = opt.CurrentValue and true or false
            toggle.MouseButton1Click:Connect(function()
                state = not state
                toggle.Text = state and "ON" or "OFF"
                toggle.BackgroundColor3 = state and STYLE.AccentColor or Color3.fromRGB(45,45,45)
                pcall(function() if opt.Callback then opt.Callback(state) end end)
            end)
            -- init color
            toggle.BackgroundColor3 = state and STYLE.AccentColor or Color3.fromRGB(45,45,45)
            table.insert(self.Elements, toggle)
            return {Toggle = toggle, GetValue = function() return state end, SetValue = function(v) state=v; toggle.Text = v and "ON" or "OFF"; toggle.BackgroundColor3 = v and STYLE.AccentColor or Color3.fromRGB(45,45,45) end}
        end

        function tabObj:CreateSlider(opt)
            opt = opt or {}
            local min = opt.Range and opt.Range[1] or 0
            local max = opt.Range and opt.Range[2] or 100
            local inc = opt.Increment or 1
            local value = opt.CurrentValue or min

            local frame,label = elementTemplate(opt.Name or "Slider")
            label.Text = opt.Name or "Slider"

            local sliderBg = new("Frame", {Parent = frame, Size = UDim2.new(0.55,0,0,16), Position = UDim2.new(0.42,0,0.5,-8), BackgroundColor3 = Color3.fromRGB(40,40,40), BorderSizePixel = 0})
            new("UICorner", {Parent = sliderBg, CornerRadius = UDim.new(0,8)})
            local fill = new("Frame", {Parent = sliderBg, Size = UDim2.new( (value - min)/(max - min), 0, 1, 0), BackgroundColor3 = STYLE.AccentColor, BorderSizePixel = 0})
            new("UICorner", {Parent = fill, CornerRadius = UDim.new(0,8)})
            local dragging = false
            local uis = game:GetService("UserInputService")

            sliderBg.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    dragging = true
                end
            end)
            sliderBg.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                    dragging = false
                end
            end)
            uis.InputChanged:Connect(function(input)
                if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                    local rel = math.clamp((input.Position.X - sliderBg.AbsolutePosition.X) / sliderBg.AbsoluteSize.X, 0, 1)
                    local raw = min + rel * (max - min)
                    -- quantize to increment
                    raw = math.floor(raw / inc + 0.5) * inc
                    value = raw
                    fill.Size = UDim2.new((value - min)/(max - min), 0, 1, 0)
                    if opt.Callback then
                        pcall(function() opt.Callback(value) end)
                    end
                end
            end)

            -- label value
            local valLabel = new("TextLabel", {Parent = frame, Size = UDim2.new(0,60,1,0), Position = UDim2.new(1,-70,0,0), BackgroundTransparency = 1, Text = tostring(value), TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Right})
            -- update function
            local function updateVal(v)
                value = v
                valLabel.Text = tostring(value)
                fill.Size = UDim2.new((value - min)/(max - min), 0, 1, 0)
            end
            updateVal(value)
            table.insert(self.Elements, {Slider = sliderBg, GetValue = function() return value end, SetValue = updateVal})
            return {GetValue = function() return value end, SetValue = updateVal}
        end

        function tabObj:CreateDropdown(opt)
            opt = opt or {}
            local items = opt.Options or {}
            local frame,label = elementTemplate(opt.Name or "Dropdown")
            label.Text = opt.Name or "Dropdown"
            local button = new("TextButton", {Parent = frame, Size = UDim2.new(0,120,0,26), Position = UDim2.new(1,-130,0.5,-13), BackgroundColor3 = Color3.fromRGB(45,45,45), Text = tostring(opt.Current or "Select"), TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 13, BorderSizePixel = 0})
            new("UICorner", {Parent = button, CornerRadius = UDim.new(0,8)})
            local open = false
            local listFrame = new("Frame", {Parent = tabContent, Size = UDim2.new(0,120,0,#items*30), Position = UDim2.new(1,-130,0,0), BackgroundColor3 = Color3.fromRGB(40,40,40), Visible = false})
            new("UICorner", {Parent = listFrame, CornerRadius = UDim.new(0,8)})
            local layout = new("UIListLayout", {Parent = listFrame, SortOrder = Enum.SortOrder.LayoutOrder})
            for i,item in ipairs(items) do
                local itbtn = new("TextButton", {Parent = listFrame, Size = UDim2.new(1,0,0,30), BackgroundTransparency = 0, BackgroundColor3 = Color3.fromRGB(45,45,45), Text = item, TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 13, BorderSizePixel = 0})
                itbtn.MouseButton1Click:Connect(function()
                    button.Text = item
                    if opt.Callback then pcall(function() opt.Callback(item) end) end
                    listFrame.Visible = false
                    open = false
                end)
            end
            button.MouseButton1Click:Connect(function()
                open = not open
                listFrame.Visible = open
            end)
            table.insert(self.Elements, button)
            return {Set = function(v) button.Text = v end, Get = function() return button.Text end}
        end

        function tabObj:CreateInput(opt)
            opt = opt or {}
            local frame,label = elementTemplate(opt.Name or "Input")
            label.Text = opt.Name or "Input"
            local inputBox = new("TextBox", {Parent = frame, Size = UDim2.new(0.44,0,1,0), Position = UDim2.new(0.5, -4, 0, 0), BackgroundColor3 = Color3.fromRGB(40,40,40), Text = opt.Placeholder or "", TextColor3 = STYLE.TextColor, Font = STYLE.Font, TextSize = 14, ClearTextOnFocus = false})
            new("UICorner", {Parent = inputBox, CornerRadius = UDim.new(0,6)})
            inputBox.FocusLost:Connect(function(enter)
                if enter and opt.Callback then pcall(function() opt.Callback(inputBox.Text) end) end
            end)
            table.insert(self.Elements, inputBox)
            return inputBox
        end

        table.insert(self.Tabs, tabObj)
        return tabObj
    end

    return win
end

-- expose library
return PhuLib
