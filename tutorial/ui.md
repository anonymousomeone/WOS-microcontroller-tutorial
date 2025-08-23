# UI

In WOS, you can make ui elements using Instance.new, however it must be through a [`Screen`](https://arvidsilverlock.github.io/Pilot.lua-Luau-LSP/objects/Screen) or an [`ARController`](https://arvidsilverlock.github.io/Pilot.lua-Luau-LSP/objects/ARController), and you must parent the element to the _canvas_ of the part. For more info on each element, see [`Roblox's ui guide`](https://create.roblox.com/docs/ui)

Note: at the time of writing, TextBoxes do not sync their values to serverside (where micro code runs). Use alternative methods of input instead (keyboard, microphone)

![Screen demo](https://files.catbox.moe/o39706.png)

```lua
local screen = GetPart("Screen")
-- we need to parent ui instances to canvas (wont render otherwise)
-- if you wanted to use an arcontroller, simply replace screen with it
-- local canvas = arcontroller:GetCanvas("2D")
local canvas = screen:GetCanvas("2D")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromScale(1, 1)
frame.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
frame.Parent = canvas;
```

To make parts in 3D space (arcontroller), first Instance.new a Part then parent to the 3D canvas instead.

![ARController demo](https://files.catbox.moe/reatce.png)

```lua
local arc = GetPart("ARController")
local canvas = arc:GetCanvas("3D")

-- spawn a part at y = 1024 (this is the height of the top of the baseplate)
local part = Instance.new("Part")
part.Size = Vector3.new(8, 8, 8)
part.Position = Vector3.new(0, 1024, 0)
part.Color = Color3.fromRGB(255, 0, 242)
part.Shape = Enum.PartType.Ball
part.Parent = canvas
```

## Practical Usage

### 1. Player targetting ui

This project will demonstrate how to use a [`LifeSensor`](https://arvidsilverlock.github.io/Pilot.lua-Luau-LSP/objects/LifeSensor) along with a screen to display a targetting ui with player selection and a big red fire button.

![Targetting ui](https://files.catbox.moe/goyxpz.png)

```lua
local SCREEN = GetPart("Screen")
local CANVAS = SCREEN:GetCanvas("2D")
local LIFESENSOR = GetPart("LifeSensor")
local GUNS = GetParts("Gun")
local GYRO = GetPart("Gyro")

local function fire()
	for _, gun in GUNS do
		gun:Trigger()
	end
end

-- search table by key instead of value
local function contains(table, element)
	for key, _ in pairs(table) do
		if key == element then
			return true
		end
	end
	return false
end

local SCROLLING_FRAME
local TARGET_DISPLAY
local function buildUI(canvas) 
	local frame = Instance.new("Frame")
	frame.Size = UDim2.fromScale(1, 1)
	frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	frame.Parent = canvas

	local scrolling_frame = Instance.new("ScrollingFrame")
	scrolling_frame.Size = UDim2.fromScale(0.5, 1)
	scrolling_frame.Parent = frame

	local player_display = Instance.new("TextLabel")
	player_display.Size = UDim2.fromScale(0.5, 0.1)
	player_display.Position = UDim2.fromScale(0.5, 0.0)
	player_display.Text = "nil"
	player_display.Parent = frame

	local fire_button = Instance.new("TextButton")
	fire_button.Size = UDim2.fromScale(0.5, 0.2)
	fire_button.Position = UDim2.fromScale(0.5, 0.8)
	fire_button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
	fire_button.Text = "Fire"
	fire_button.Parent = frame

	fire_button.MouseButton1Click:Connect(function()
		-- nuke em
		fire()
	end)

	SCROLLING_FRAME = scrolling_frame
	TARGET_DISPLAY = player_display
end

local player_elements = {};
local function update_playerui()
	local players = LIFESENSOR:GetReading()

	-- update player list
	local index = 0;
	for player, _ in pairs(players) do
		if not contains(player_elements, player) then
			local element = Instance.new("TextButton")
			element.Size = UDim2.fromScale(1, 0.05)
			element.Position = UDim2.fromScale(0.0, index * 0.05)
			element.Text = player

			element.MouseButton1Click:Connect(function()
				-- on click, set gyro target
				GYRO.Seek = player;
				TARGET_DISPLAY.Text = player;
			end);

			player_elements[player] = element;

			element.Parent = SCROLLING_FRAME;

		end

		index += 1;
	end

	local index = 0;
	for player, element in pairs(player_elements) do
		-- this player left, destroy their corresponding ui element
		if not contains(players, player) then
			player_elements[player]:Destroy();
			player_elements[player] = nil;
			continue;
		end

		element.Position = UDim2.fromScale(0.0, index * 0.05);
		index += 1;
	end

	return players
end

buildUI(CANVAS)

while task.wait(1) do
	update_playerui()
end
```

Model code:

```json
{"m":null,"t":"buffer","zbase64":"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"}
```