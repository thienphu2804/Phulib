-- PhuLibVN v1.0
-- Thư viện GUI tiếng Việt (phong cách Rayfield)

local PhuLibVN = {}
PhuLibVN.__index = PhuLibVN

-- Hàm tạo object nhanh
local function tao(loai, props)
    local obj = Instance.new(loai)
    for k,v in pairs(props or {}) do obj[k] = v end
    return obj
end

-- ScreenGui chính
local CoreGui = game:GetService("CoreGui")
local manhinh = CoreGui:FindFirstChild("PhuLibVN_ScreenGui")
if not manhinh then
    manhinh = tao("ScreenGui", {Name="PhuLibVN_ScreenGui", ResetOnSpawn=false})
    manhinh.Parent = CoreGui
end

-- Cấu hình cơ bản
local MAU = {
    Nen = Color3.fromRGB(25,25,25),
    Nhan = Color3.fromRGB(230,230,230),
    Cham = Color3.fromRGB(0,170,255)
}

-- Lớp cửa sổ
local CuaSo = {}
CuaSo.__index = CuaSo

function PhuLibVN:TạoCửaSổ(tuychon)
    tuychon = tuychon or {}
    local ten = tuychon.Tên or "Cửa Sổ"

    local frame = tao("Frame", {
        Parent = manhinh,
        Size = UDim2.new(0,500,0,350),
        Position = UDim2.new(0.25,0,0.25,0),
        BackgroundColor3 = MAU.Nen,
        BorderSizePixel = 0
    })
    tao("UICorner",{Parent=frame, CornerRadius=UDim.new(0,10)})

    local tieuDe = tao("TextLabel",{
        Parent = frame,
        Text = ten,
        Size = UDim2.new(1,0,0,35),
        BackgroundTransparency = 1,
        TextColor3 = MAU.Nhan,
        Font = Enum.Font.GothamBold,
        TextSize = 18
    })

    local than = tao("Frame", {
        Parent = frame,
        Size = UDim2.new(1,-10,1,-45),
        Position = UDim2.new(0,5,0,40),
        BackgroundTransparency = 1
    })

    local cuaso = setmetatable({
        Goc = frame,
        Than = than,
        Tab = {}
    }, CuaSo)

    function cuaso:TạoTab(ten)
        local tabFrame = tao("Frame", {
            Parent = than,
            Size = UDim2.new(1,0,1,0),
            BackgroundTransparency = 1
        })
        local layout = tao("UIListLayout",{Parent=tabFrame,SortOrder=Enum.SortOrder.LayoutOrder,Padding=UDim.new(0,6)})

        local tab = {
            Goc = tabFrame
        }

        -- API tiếng Việt
        function tab:TạoNút(tuychon)
            local nut = tao("TextButton", {
                Parent = tabFrame,
                Size = UDim2.new(1,-20,0,30),
                Text = tuychon.Tên or "Nút",
                BackgroundColor3 = Color3.fromRGB(40,40,40),
                TextColor3 = MAU.Nhan,
                Font = Enum.Font.Gotham,
                TextSize = 14,
                BorderSizePixel = 0
            })
            tao("UICorner",{Parent=nut,CornerRadius=UDim.new(0,6)})
            nut.MouseButton1Click:Connect(function()
                if tuychon.Hàm then pcall(tuychon.Hàm) end
            end)
            return nut
        end

        function tab:TạoGạt(tuychon)
            local khung = tao("Frame", {Parent=tabFrame,Size=UDim2.new(1,-20,0,30),BackgroundTransparency=1})
            local nhan = tao("TextLabel",{Parent=khung,Text=tuychon.Tên or "Gạt",Size=UDim2.new(0.6,0,1,0),BackgroundTransparency=1,TextColor3=MAU.Nhan,Font=Enum.Font.Gotham,TextSize=14,TextXAlignment=Enum.TextXAlignment.Left})
            local nut = tao("TextButton",{Parent=khung,Size=UDim2.new(0,60,0,26),Position=UDim2.new(1,-70,0.5,-13),BackgroundColor3=Color3.fromRGB(50,50,50),Text="TẮT",TextColor3=MAU.Nhan,Font=Enum.Font.Gotham,TextSize=13})
            tao("UICorner",{Parent=nut,CornerRadius=UDim.new(0,6)})
            local trangthai = false
            nut.MouseButton1Click:Connect(function()
                trangthai = not trangthai
                nut.Text = trangthai and "BẬT" or "TẮT"
                nut.BackgroundColor3 = trangthai and MAU.Cham or Color3.fromRGB(50,50,50)
                if tuychon.Hàm then pcall(function() tuychon.Hàm(trangthai) end) end
            end)
            return nut
        end

        function tab:TạoNhãn(tuychon)
            local nhan = tao("TextLabel",{
                Parent=tabFrame,
                Size=UDim2.new(1,-20,0,25),
                Text = tuychon.Tên or "Nhãn",
                BackgroundTransparency=1,
                TextColor3=MAU.Nhan,
                Font=Enum.Font.Gotham,
                TextSize=14,
                TextXAlignment=Enum.TextXAlignment.Left
            })
            return nhan
        end

        return tab
    end

    return cuaso
end

return PhuLibVN
