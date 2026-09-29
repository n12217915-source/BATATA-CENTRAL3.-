--=============================================================
--  PLAYER — Float / Speed / Jump / InfJump
--=============================================================
local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local UserInputService  = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer       = Players.LocalPlayer

local Config = {
	Float        = false,
	FloatHeight  = 5,
	FloatSpeed   = 20,
	Speed        = false,
	SpeedValue   = 32,
	Jump         = false,
	JumpValue    = 100,
	InfJump      = false,
}

local function getHum()
	return LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
end

-- Float
local floatBody, floatAttach
local function setupFloat()
	local char = LocalPlayer.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	floatAttach = Instance.new("Attachment")
	floatAttach.Parent = hrp

	floatBody = Instance.new("BodyPosition")
	floatBody.MaxForce = Vector3.new(1e5,1e5,1e5)
	floatBody.D = 600
	floatBody.P = 12000
	floatBody.Position = hrp.Position
	floatBody.Parent = hrp
end

local function destroyFloat()
	if floatBody then floatBody:Destroy() floatBody = nil end
	if floatAttach then floatAttach:Destroy() floatAttach = nil end
end

RunService.Heartbeat:Connect(function(dt)
	-- Float
	if Config.Float then
		if not floatBody then setupFloat() end
		if floatBody then
			local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				local target = hrp.Position
				target = Vector3.new(target.X, target.Y + Config.FloatHeight*0.02, target.Z)
				floatBody.Position = target
			end
		end
	else
		destroyFloat()
	end

	-- Speed
	local hum = getHum()
	if hum then
		hum.WalkSpeed = Config.Speed and Config.SpeedValue or 16
		hum.JumpPower = Config.Jump and Config.JumpValue or 50
		hum.UseJumpPower = true
	end
end)

-- InfJump
UserInputService.JumpRequest:Connect(function()
	if Config.InfJump then
		local hum = getHum()
		if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
	end
end)

--=============================================================
--  Registro (delay 5s)
--=============================================================
task.wait(5)
local api = ReplicatedStorage:WaitForChild("BatataHub_RegisterTab")

local ok, err = api:Invoke("Batata001", {
	Name = "PLAYER",
	BuildContent = function(page, ctx)
		local function makeToggle(text, y, initial, onChange)
			local btn = Instance.new("TextButton")
			btn.Position = UDim2.fromOffset(9,y)
			btn.Size = UDim2.new(1,-18,0,22)
			btn.BackgroundColor3 = ctx.colors.BUTTON or Color3.fromRGB(35,35,35)
			btn.Text = ""
			btn.BorderSizePixel = 0
			btn.Parent = page
			local c = Instance.new("UICorner") c.CornerRadius = UDim.new(0,5) c.Parent = btn
			local lbl = Instance.new("TextLabel")
			lbl.BackgroundTransparency = 1
			lbl.Position = UDim2.fromOffset(8,0)
			lbl.Size = UDim2.new(1,-50,1,0)
			lbl.Font = Enum.Font.Gotham
			lbl.TextSize = 11
			lbl.TextColor3 = ctx.colors.TEXT
			lbl.TextXAlignment = Enum.TextXAlignment.Left
			lbl.Text = text
			lbl.Parent = btn
			local st = Instance.new("TextLabel")
			st.BackgroundTransparency = 1
			st.Position = UDim2.new(1,-45,0,0)
			st.Size = UDim2.fromOffset(40,22)
			st.Font = Enum.Font.GothamBold
			st.TextSize = 11
			st.TextColor3 = initial and Color3.fromRGB(90,220,120) or Color3.fromRGB(220,90,90)
			st.Text = initial and "ON" or "OFF"
			st.Parent = btn
			btn.MouseButton1Click:Connect(function()
				initial = not initial
				st.Text = initial and "ON" or "OFF"
				st.TextColor3 = initial and Color3.fromRGB(90,220,120) or Color3.fromRGB(220,90,90)
				onChange(initial)
			end)
		end

		local function makeSlider(text, y, min, max, initial, onChange)
			local frame = Instance.new("Frame")
			frame.Position = UDim2.fromOffset(9,y)
			frame.Size = UDim2.new(1,-18,0,30)
			frame.BackgroundTransparency = 1
			frame.Parent = page
			local lbl = Instance.new("TextLabel")
			lbl.BackgroundTransparency = 1
			lbl.Size = UDim2.new(1,0,0,14)
			lbl.Font = Enum.Font.Gotham
			lbl.TextSize = 11
			lbl.TextColor3 = ctx.colors.SUBTEXT
			lbl.TextXAlignment = Enum.TextXAlignment.Left
			lbl.Text = text..": "..tostring(initial)
			lbl.Parent = frame
			local bar = Instance.new("Frame")
			bar.Position = UDim2.fromOffset(0,18)
			bar.Size = UDim2.new(1,0,0,6)
			bar.BackgroundColor3 = Color3.fromRGB(50,50,50)
			bar.BorderSizePixel = 0
			bar.Parent = frame
			local bc = Instance.new("UICorner") bc.CornerRadius = UDim.new(1,0) bc.Parent = bar
			local fill = Instance.new("Frame")
			fill.Size = UDim2.new((initial-min)/(max-min),0,1,0)
			fill.BackgroundColor3 = ctx.colors.ACCENT or Color3.fromRGB(120,180,255)
			fill.BorderSizePixel = 0
			fill.Parent = bar
			local fc = Instance.new("UICorner") fc.CornerRadius = UDim.new(1,0) fc.Parent = fill
			local dragging = false
			local function update(input)
				local pos = math.clamp((input.Position.X - bar.AbsolutePosition.X)/bar.AbsoluteSize.X,0,1)
				fill.Size = UDim2.new(pos,0,1,0)
				local val = math.floor(min+(max-min)*pos+0.5)
				lbl.Text = text..": "..tostring(val)
				onChange(val)
			end
			bar.InputBegan:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1
				or input.UserInputType == Enum.UserInputType.Touch then
					dragging = true update(input)
				end
			end)
			UserInputService.InputChanged:Connect(function(input)
				if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
				or input.UserInputType == Enum.UserInputType.Touch) then update(input) end
			end)
			UserInputService.InputEnded:Connect(function(input)
				if input.UserInputType == Enum.UserInputType.MouseButton1
				or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
			end)
		end

		local t = Instance.new("TextLabel")
		t.BackgroundTransparency = 1
		t.Position = UDim2.fromOffset(9,8)
		t.Size = UDim2.new(1,-18,0,17)
		t.Font = Enum.Font.GothamBlack
		t.Text = "PLAYER"
		t.TextSize = 14
		t.TextColor3 = ctx.colors.TEXT
		t.TextXAlignment = Enum.TextXAlignment.Left
		t.Parent = page

		local scroll = Instance.new("ScrollingFrame")
		scroll.Position = UDim2.fromOffset(0,32)
		scroll.Size = UDim2.new(1,0,1,-32)
		scroll.BackgroundTransparency = 1
		scroll.BorderSizePixel = 0
		scroll.ScrollBarThickness = 4
		scroll.CanvasSize = UDim2.new(0,0,0,420)
		scroll.Parent = page

		makeToggle("Float",    8,  Config.Float,   function(v) Config.Float=v end)
		makeSlider("Float H", 36, 1, 20, Config.FloatHeight, function(v) Config.FloatHeight=v end)

		makeToggle("Speed",   76, Config.Speed,   function(v) Config.Speed=v end)
		makeSlider("Speed",   104, 16, 300, Config.SpeedValue, function(v) Config.SpeedValue=v end)

		makeToggle("Jump",    144, Config.Jump,   function(v) Config.Jump=v end)
		makeSlider("JumpPower",172, 50, 500, Config.JumpValue, function(v) Config.JumpValue=v end)

		makeToggle("InfJump", 212, Config.InfJump, function(v) Config.InfJump=v end)
	end
})

if not ok then warn("[Player] registro falhou:", err) end
