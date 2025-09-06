--// PhuLib.lua - Ultra Short UI Lib by Phú
local PhuLib = {}

function PhuLib:Win(title)
    local sg = Instance.new("ScreenGui", game:GetService("CoreGui"))
    local frame = Instance.new("Frame", sg)
    frame.Size = UDim2.new(0, 400, 0, 300)
    frame.Position = UDim2.new(0.5, -200, 0.5, -150)
    frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    frame.Active = true frame.Draggable = true

    local tl = Instance.new("TextLabel", frame)
    tl.Size = UDim2.new(1,0,0,30)
    tl.BackgroundColor3 = Color3.fromRGB(40,40,40)
    tl.Text = title tl.TextColor3 = Color3.fromRGB(255,255,255)
    tl.Font = Enum.Font.SourceSansBold tl.TextSize = 20

    local cont = Instance.new("Frame", frame)
    cont.Size = UDim2.new(1,-10,1,-40)
    cont.Position = UDim2.new(0,5,0,35)
    cont.BackgroundTransparency = 1

    Instance.new("UIListLayout", cont).Padding = UDim.new(0,5)

    local win = {}
    function win:Tab(name)
        local tab = {}

        function tab:Btn(txt, cb)
            local b = Instance.new("TextButton", cont)
            b.Size = UDim2.new(1,0,0,30)
            b.Text = txt b.BackgroundColor3 = Color3.fromRGB(60,60,60)
            b.TextColor3 = Color3.fromRGB(255,255,255)
            b.MouseButton1Click:Connect(cb)
        end

        function tab:Tog(txt, state, cb)
            local b = Instance.new("TextButton", cont)
            b.Size = UDim2.new(1,0,0,30)
            b.Text = txt.." : "..tostring(state)
            b.BackgroundColor3 = Color3.fromRGB(60,60,60)
            b.TextColor3 = Color3.fromRGB(255,255,255)
            b.MouseButton1Click:Connect(function()
                state = not state
                b.Text = txt.." : "..tostring(state)
                cb(state)
            end)
        end

        function tab:Sld(txt, min, max, cb)
            local slider = Instance.new("TextButton", cont)
            slider.Size = UDim2.new(1,0,0,30)
            slider.BackgroundColor3 = Color3.fromRGB(60,60,60)
            slider.TextColor3 = Color3.fromRGB(255,255,255)
            slider.Text = txt..": "..min
            local val = min
            slider.MouseButton1Click:Connect(function()
                val = (val + 1 > max) and min or val + 1
                slider.Text = txt..": "..val
                cb(val)
            end)
        end

        return tab
    end

    return win
end

return PhuLib
