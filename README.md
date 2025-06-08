local DrawingLib = {}

local Camera = game:GetService("Workspace"):FindFirstChild("Camera")
local RunService = game:GetService("RunService")
local CoreGui = (RunService:IsStudio() and game:GetService("Players")["LocalPlayer"]:WaitForChild("PlayerGui") or game:GetService("CoreGui"))

local BaseDrawingProperties = setmetatable({
	Visible = true,
	Color = Color3.new(),
	Transparency = 0,
    Position, Vector2.new(),
	Remove = function()
	end
}, {
	__add = function(tbl1, tbl2)
		local new = {}
		for i, v in next, tbl1 do
			new[i] = v
		end
		for i, v in next, tbl2 do
			new[i] = v
		end
		return new
	end
})

local DrawingUI = nil;

DrawingLib.new = function(Type)
	if DrawingUI == nil then
		DrawingUI = Instance.new("ScreenGui");
		DrawingUI.Parent = CoreGui;
		DrawingUI.Name = "DrawingLib"
		DrawingUI.DisplayOrder = 1999999999
		DrawingUI.IgnoreGuiInset = true
	end

	if (Type == "Line") then
		local LineProperties = ({
			To = Vector2.new(),
			From = Vector2.new(),
			Thickness = 1,
		} + BaseDrawingProperties)

		local LineFrame = Instance.new("Frame");
		LineFrame.AnchorPoint = Vector2.new(0.5, 0.5);
		LineFrame.BorderSizePixel = 0

		LineFrame.BackgroundColor3 = LineProperties.Color
		LineFrame.Visible = LineProperties.Visible
		LineFrame.BackgroundTransparency =  LineProperties.Transparency
		LineFrame.ZIndex=3000

		LineFrame.Parent = DrawingUI

		return setmetatable({}, {
					__newindex = (function(self, Property, Value)
				if (Property == "To") then
					local To = Value
					local Direction = (To - LineProperties.From);
					local Center = (To + LineProperties.From) / 2
					local Distance = Direction.Magnitude
					local Theta = math.atan2(Direction.Y, Direction.X);

					LineFrame.Position = UDim2.fromOffset(Center.X, Center.Y);
					LineFrame.Rotation = math.deg(Theta);
					LineFrame.Size = UDim2.fromOffset(Distance, LineProperties.Thickness);

					LineProperties.To = To
				end
				if (Property == "From") then
					local From = Value
					local Direction = (LineProperties.To - From);
					local Center = (LineProperties.To + From) / 2
					local Distance = Direction.Magnitude
					local Theta = math.atan2(Direction.Y, Direction.X);

					LineFrame.Position = UDim2.fromOffset(Center.X, Center.Y);
					LineFrame.Rotation = math.deg(Theta);
					LineFrame.Size = UDim2.fromOffset(Distance, LineProperties.Thickness);


					LineProperties.From = From
				end
				if (Property == "Visible") then
					LineFrame.Visible = Value
					LineProperties.Visible = Value
				end
				if (Property == "Thickness") then
					Value = Value < 1 and 1 or Value

					local Direction = (LineProperties.To - LineProperties.From);
					local Distance = Direction.Magnitude

					LineFrame.Size = UDim2.fromOffset(Distance, Value);

					LineProperties.Thickness = Value
				end
				if (Property == "Transparency") then
					LineFrame.BackgroundTransparency = 1 - Value
					LineProperties.Transparency = 1 - Value
				end
				if (Property == "Color") then
					LineFrame.BackgroundColor3 = Value
					LineProperties.Color = Value 
				end
				if (Property == "ZIndex") then
					LineFrame.ZIndex = Value
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						LineFrame:Destroy();
					end)
				end
                if Property == "Destroy" then
                 return (function()
						LineFrame:Destroy();
					end)
                end
				return LineProperties[Property]
			end)
		})
	end

	if (Type == "Circle") then
		local CircleProperties = ({
			Radius = 150,
			Filled = false,
			Thickness = 0,
			Position = Vector2.new()
		} + BaseDrawingProperties)

		local CircleFrame = Instance.new("Frame");

		CircleFrame.AnchorPoint = Vector2.new(0.5, 0.5);
		CircleFrame.BorderSizePixel = 0

		CircleFrame.BackgroundColor3 = CircleProperties.Color
		CircleFrame.Visible = CircleProperties.Visible
		CircleFrame.BackgroundTransparency = CircleProperties.Transparency

		local Corner = Instance.new("UICorner", CircleFrame);
		Corner.CornerRadius = UDim.new(1, 0);
		CircleFrame.Size = UDim2.new(0, CircleProperties.Radius, 0, CircleProperties.Radius);

		CircleFrame.Parent = DrawingUI

		local Stroke = Instance.new("UIStroke", CircleFrame)
		Stroke.Thickness = CircleProperties.Thickness
		Stroke.Enabled = true
        Stroke.Transparency = 0

		return setmetatable({}, {
			__newindex = (function(self, Property, Value)
				if (Property == "Radius") then
					CircleFrame.Size = UDim2.new(0,Value*2,0,Value*2)
					CircleProperties.Radius = Value
				end
				if (Property == "Position") then
					CircleFrame.Position = UDim2.new(0, Value.X, 0, Value.Y);
					CircleProperties.Position = Value
				end
				if (Property == "Filled") then
					if Value == true then	
						CircleFrame.BackgroundTransparency = CircleProperties.Transparency
						Stroke.Enabled = not Value
						CircleProperties.Filled = Value
					else
					    CircleFrame.BackgroundTransparency = (Value == true and 0 or 1)
					    Stroke.Enabled = not Value
					    CircleProperties.Filled = Value
					end
				end
				if (Property == "Color") then
					CircleFrame.BackgroundColor3 = Value
					Stroke.Color = Value
					CircleProperties.Color = Value
				end
				if (Property == "Thickness") then
					Stroke.Thickness = Value
					CircleProperties.Thickness = Value
				end
				if (Property == "Transparency") then
					CircleFrame.BackgroundTransparency = Value
					CircleProperties.Transparency = Value
				end
				if (Property == "Visible") then
					CircleFrame.Visible = Value
					CircleProperties.Visible = Value
				end
				if (Property == "ZIndex") then
					CircleFrame.ZIndex = Value
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						CircleFrame:Destroy();
					end)
				end
                if Property ==  "Destroy" then
                return (function()
						CircleFrame:Destroy();
					end)
                end
				return CircleProperties[Property]
			end)
		})
	end

	if (Type == "Text") then
		local TextProperties = ({
			Text = "",
			Center = false,
			Outline = false,
			OutlineColor = Color3.new(),
			Position = Vector2.new(),
            TextBounds = Vector2.new(),
		} + BaseDrawingProperties)

		local TextLabel = Instance.new("TextLabel");
		TextLabel.AnchorPoint = Vector2.new(0.5,0.5)
		TextLabel.BorderSizePixel = 0
		TextLabel.Font = Enum.Font.SourceSans
		TextLabel.TextSize = 14
		TextLabel.TextXAlignment = Enum.TextXAlignment.Left or Enum.TextXAlignment.Right
		TextLabel.TextYAlignment = Enum.TextYAlignment.Top

		TextLabel.TextColor3 = TextProperties.Color
		TextLabel.Visible = true
		TextLabel.BackgroundTransparency = 1
		TextLabel.TextTransparency = 1 - TextProperties.Transparency
		
		local Stroke = Instance.new("UIStroke", TextLabel)
		Stroke.Thickness = 0
		Stroke.Enabled = false
		Stroke.Color = TextProperties.OutlineColor
		TextLabel.Parent = DrawingUI

		return setmetatable({}, {
			__newindex = (function(self, Property, Value)
				if (Property == "Text") then
					TextLabel.Text = Value
					TextProperties.Text = Value
				end
				if (Property == "Position") then
						TextLabel.Position = UDim2.fromOffset(Value.X, Value.Y);
					    TextProperties.Position = Vector2.new(Value.X, Value.Y);
				end
				if (Property == "Size") then
					TextLabel.TextSize = Value
					TextProperties.TextSize = Value
				end
				if (Property == "Color") then
					TextLabel.TextColor3 = Value
				--	Stroke.Color = Value
					TextProperties.Color = Value
				end
				if (Property == "Transparency") then
					TextLabel.TextTransparency = 1 - Value
                     Stroke.Transparency = 1 - Value
					TextProperties.Transparency = 1 - Value
				end
				if (Property == "OutlineOpacity") then
					TextLabel.TextStrokeTransparency = Value
					--Stroke.Transparency = Value
				end
				if (Property == "OutlineColor") then
					Stroke.Color = Value
					TextProperties.OutlineColor = Value
				end
				if (Property == "Visible") then
					TextLabel.Visible = Value
					TextProperties.Visible = Value
				end
				if (Property == "Outline") then
					if Value == true then
						Stroke.Thickness = 1
						Stroke.Enabled = Value
					else
						Stroke.Thickness = 0
						Stroke.Enabled = Value
					end
				end
				if (Property == "TextBounds") then
					TextLabel.TextBounds = Vector2.new(TextProperties.Position , Value);
				end
				if (Property == "Center") then
					if Value == true then
						TextLabel.TextXAlignment = Enum.TextXAlignment.Center;
						TextLabel.TextYAlignment = Enum.TextYAlignment.Center;
						TextProperties.Center = Enum.TextYAlignment.Center;
					else
						TextProperties.Center = Value
					end
				end
				if (Property == "ZIndex") then
					TextLabel.ZIndex = Value
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						TextLabel:Destroy();
					end)
				end
                 if Property == "Destroy" then
                return (function()
						TextLabel:Destroy();
					end)
                end
				return TextProperties[Property]
			end)
		})
	end

	if (Type == "Square") then
		local SquareProperties = ({
			Thickness = 1,
			Size = Vector2.new(),
			Position = Vector2.new(),
			Filled = false,
		} + BaseDrawingProperties);
		local SquareFrame = Instance.new("Frame");

		--SquareFrame.AnchorPoint = Vector2.new(0.5, 0.5);
		SquareFrame.BorderSizePixel = 0

		SquareFrame.Visible = SquareProperties.Visible
		SquareFrame.Parent = DrawingUI

		local Stroke = Instance.new("UIStroke", SquareFrame)
		Stroke.Thickness = 2
		Stroke.Enabled = true
		SquareFrame.BackgroundTransparency = 0
		Stroke.Transparency = 0

		return setmetatable({}, {
			__newindex = (function(self, Property, Value)
				if (Property == "Position") then
					SquareFrame.Position = UDim2.fromOffset(Value.X, Value.Y);
					SquareProperties.Position = Value
				end
				if (Property == "Size") then
					SquareFrame.Size = UDim2.new(0, Value.X, 0, Value.Y);
					SquareProperties.Size = Value
				end
                if (Property == "Thickness") then
                    Stroke.Thickness = Value
                    SquareProperties.Thickness = Value
				end
				if (Property == "Color") then
					SquareFrame.BackgroundColor3 = Value
					Stroke.Color = Value
					SquareProperties.Color = Value
				end
				if (Property == "Transparency") then
					--SquareFrame.BackgroundTransparency = Value
				--	Stroke.Transparency = Value
					SquareProperties.Transparency = Value
				end
				if (Property == "Visible") then
					SquareFrame.Visible = Value
					SquareProperties.Visible = Value
				end
				if (Property == "Filled") then -- requires beta
					if Value == true then	
						SquareFrame.BackgroundTransparency = SquareProperties.Transparency
						Stroke.Transparency = 1
						Stroke.Enabled = not Value
						SquareProperties.Filled = Value
					else
					    SquareFrame.BackgroundTransparency = (Value == true and 0 or 1)
					    Stroke.Enabled = not Value
					    SquareProperties.Filled = Value
					end
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						SquareFrame:Destroy();
					end)
				end
               if Property == "Destroy" then				
				return (function()
						SquareFrame:Destroy();
					end)
				end
				return SquareProperties[Property]
			end)
		})
	end

 if (Type == "Image") then
		local ImageProperties = ({
			Data = "rbxassetid://848623155", -- roblox assets only rn
			Size = Vector2.new(),
			Position = Vector2.new(),
			Rounding = 0,
			Color = Color3.new(),
		});

		local ImageLabel = Instance.new("ImageLabel");

		ImageLabel.BorderSizePixel = 0
		ImageLabel.ScaleType = Enum.ScaleType.Stretch
		ImageLabel.Transparency = 1

		ImageLabel.ImageColor3 = ImageProperties.Color
		ImageLabel.Visible = false
		ImageLabel.Parent = DrawingUI

		return setmetatable({}, {
			__newindex = (function(self, Property, Value)
				if (Property == "Size") then
					ImageLabel.Size = UDim2.new(0, Value.X, 0, Value.Y);
					ImageProperties.Text = Value
				end
				if (Property == "Position") then
					ImageLabel.Position = UDim2.new(0, Value.X, 0, Value.Y);
					ImageProperties.Position = Value
				end
				if (Property == "Size") then
					ImageLabel.Size = UDim2.new(0, Value.X, 0, Value.Y);
					ImageProperties.Size = Value
				end
				if (Property == "Transparency") then
					ImageLabel.ImageTransparency = math.clamp(1-Value,0,1)
					ImageProperties.Transparency = math.clamp(1-Value,0,1)
				end
				if (Property == "Visible") then
					ImageLabel.Visible = Value
					ImageProperties.Visible = Value
				end
				if (Property == "Color") then
					ImageLabel.ImageColor3 = Value
					ImageProperties.Color = Value
				end
				if (Property == "Data") then
					ImageLabel.Image = Value
					ImageProperties.Data = Value
				end
				if (Property == "ZIndex") then
					ImageLabel.ZIndex = Value
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						ImageLabel:Destroy();
					end)
				end
                if Property ==  "Destroy" then
                return (function()
						ImageLabel:Destroy();
					end)
                end
				return ImageProperties[Property]
			end)
		})
	end

	if (Type == "Quad") then -- idk if this will work lmao
		local QuadProperties = ({
			Thickness = 1,
			Transparency = 1,	
			Color = Color3.new(),
			PointA = Vector2.new();
			PointB = Vector2.new();
			PointC = Vector2.new();
			PointD = Vector2.new();
			Filled = false;
		}  + BaseDrawingProperties);

		local PointA = DrawingLib.new("Line")
		local PointB = DrawingLib.new("Line")
		local PointC = DrawingLib.new("Line")
		local PointD = DrawingLib.new("Line")

		return setmetatable({}, {
			__newindex = (function(self, Property, Value)
				if Property == "Thickness" then
					PointA.Thickness = Value
					PointB.Thickness = Value
					PointC.Thickness = Value
					PointD.Thickness = Value
					QuadProperties.Thickness = Value
				end
				if Property == "PointA" then
					PointA.From = Value
					PointB.To = Value
				end
				if Property == "PointB" then
					PointB.From = Value
					PointC.To = Value
				end
				if Property == "PointC" then
					PointC.From = Value
					PointD.To = Value
				end
				if Property == "PointD" then
					PointD.From = Value
					PointA.To = Value
				end
				if Property == "Filled" then
					-- i'll do this later
				end
				if Property == "Color" then
					PointA.Color = Value
					PointB.Color = Value
					PointC.Color = Value
					PointD.Color = Value
					QuadProperties.Color = Value
				end
				if Property == "Transparency" then
					PointA.Transparency = Value
					PointB.Transparency = Value
					PointC.Transparency = Value
					PointD.Transparency = Value
					QuadProperties.Transparency = Value
				end
				if Property == "Visible" then
					PointA.Visible = Value
					PointB.Visible = Value
					PointC.Visible = Value
					PointD.Visible = Value
					QuadProperties.Visible = Value
				end
				if (Property == "ZIndex") then
					PointA.ZIndex = Value
					PointB.ZIndex = Value
					PointC.ZIndex = Value
					PointD.ZIndex = Value
					QuadProperties.ZIndex = Value
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						PointA:Remove();
						PointB:Remove();
						PointC:Remove();
						PointD:Remove();
					end)
				end
                if Property ==  "Destroy" then
                       return (function()
						PointA:Remove();
						PointB:Remove();
						PointC:Remove();
						PointD:Remove();
					end)
                end
				return QuadProperties[Property]
			end)
		});
	end

	if (Type == "Triangle") then  -- idk if this will work lmao
		local TriangleProperties = ({
			Thickness = 1,
			Transparency = 1,	
			Color = Color3.new(),
			PointA = Vector2.new();
			PointB = Vector2.new();
			PointC = Vector2.new();
			PointD = Vector2.new();
			Filled = false;
		}  + BaseDrawingProperties);

		local PointA = DrawingLib.new("Line")
		local PointB = DrawingLib.new("Line")
		local PointC = DrawingLib.new("Line")

		return setmetatable({}, {
			__newindex = (function(self, Property, Value)
				if Property == "Thickness" then
					PointA.Thickness = Value
					PointB.Thickness = Value
					PointC.Thickness = Value
					PointD.Thickness = Value
					TriangleProperties.Thickness = Value
				end
				if Property == "PointA" then
					PointA.From = Value
					PointB.To = Value
				end
				if Property == "PointB" then
					PointB.From = Value
					PointC.To = Value
				end
				if Property == "PointC" then
					PointC.From = Value
					PointD.To = Value
				end
				if Property == "PointD" then
					PointD.From = Value
					PointA.To = Value
				end
				if Property == "Filled" then
					-- i'll do this later
				end
				if Property == "Color" then
					PointA.Color = Value
					PointB.Color = Value
					PointC.Color = Value
					PointD.Color = Value
					TriangleProperties.Color = Value
				end
				if Property == "Transparency" then
					PointA.Transparency = Value
					PointB.Transparency = Value
					PointC.Transparency = Value
					PointD.Transparency = Value
					TriangleProperties.Transparency = Value
				end
				if Property == "Visible" then
					PointA.Visible = Value
					PointB.Visible = Value
					PointC.Visible = Value
					PointD.Visible = Value
					TriangleProperties.Visible = Value
				end
				if (Property == "ZIndex") then
					PointA.ZIndex = Value
					PointB.ZIndex = Value
					PointC.ZIndex = Value
					PointD.ZIndex = Value
					TriangleProperties.ZIndex = Value
				end
			end),
			__index = (function(self, Property)
				if (string.lower(tostring(Property)) == "remove") then
					return (function()
						PointA:Remove();
						PointB:Remove();
						PointC:Remove();
					end)
				end
                if Property ==  "Destroy" then
                return (function()
						PointA:Remove();
						PointB:Remove();
						PointC:Remove();
					end)
                end
				return TriangleProperties[Property]
			end)
		});
	end
end


DrawingLib.clear = function() 
	DrawingUI:ClearAllChildren();
end

if RunService:IsStudio() then
	return DrawingLib
else
	if getgenv then
		getgenv()["Drawing"] = DrawingLib
		getgenv()["clear_drawing_lib"] = DrawingLib.clear
		Drawing = drawing
	else
		Drawing = DrawingLib
	end
end
getgenv()["Drawing"] = DrawingLib
getgenv()["clear_drawing_lib"] = DrawingLib.clear
getgenv().Drawing = DrawingLib
getgenv().clear_drawing_lib = DrawingLib.clear

Drawing.Fonts = {}

Drawing.Fonts.UI = 0
Drawing.Fonts.System = 1
Drawing.Fonts.Plex = 2
Drawing.Fonts.Monospace = 3

getgenv().cleardrawcache = DrawingLib.clear
Drawing.cleardrawcache = DrawingLib.clear



local rendered = false
getgenv().isrenderobj = function(a, drawing2)
if rendered == true then
return false
end

rendered = true
return true
end

getgenv().setrenderproperty = function(drawing, object, value)
drawing[object] = value
return object
end

getgenv().getrenderproperty = function(drawing, object)
local value = drawing[object]
return value
end

getgenv().debug.info = getrenv().debug.info


local function register(i, v)
    getgenv()[i] = v
    return v
end

if not WebSocket then
    local WebSocket = {
        connect = function()
            return {
                Send = function() end,
                Close = function() end,
                OnMessage = { Connect = function() end },
                OnClose = { Connect = function() end }
            }
        end
    }
    register('WebSocket', WebSocket)
end

 
