if not game:IsLoaded() then game.Loaded:Wait() end
task.wait(1)

local GAMES = {
	[8304191830] = "TypeSoul",
	[80167904212435] = "StrikeBorn",
}

local BASE = "https://raw.githubusercontent.com/andrew221121-collab/berri-loader/refs/heads/main/"

local gameMode = GAMES[game.PlaceId]
if not gameMode then
	return warn("[Berri] Game not supported. Place ID: " .. game.PlaceId)
end

local cache = {}
getgenv().require = function(path)
	path = path:gsub("%.", "/")
	if cache[path] then return cache[path] end

	local url = BASE .. path .. ".lua.txt"
	local ok, result = pcall(function()
		return loadstring(game:HttpGet(url))()
	end)
	if not ok then
		local ok2, result2 = pcall(function()
			return loadstring(game:HttpGet(BASE .. path .. ".lua"))()
		end)
		if ok2 then result = result2 else warn("[Berri] Failed: " .. path) return nil end
	end

	cache[path] = result
	return result
end

getgenv().LPH_NO_VIRTUALIZE = function(...) return ... end
getgenv().PP_SCRAMBLE_NUM = function(...) return ... end
getgenv().PP_SCRAMBLE_STR = function(...) return ... end
getgenv().PP_SCRAMBLE_RE_NUM = function(...) return ... end

local GameConfig = require("Game/GameConfig")
local config = GameConfig[gameMode]

if not config then
	return warn("[Berri] No config found for: " .. gameMode)
end

local AutoParry = require("Modules/AutoParry")
AutoParry.init(config)

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BerriToggle"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")

local toggle = Instance.new("TextButton")
toggle.Name = "AutoParryToggle"
toggle.Size = UDim2.new(0, 100, 0, 40)
toggle.Position = UDim2.new(0.5, -50, 0, 10)
toggle.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
toggle.TextSize = 14
toggle.Font = Enum.Font.GothamBold
toggle.Text = "Parry: OFF"
toggle.Parent = screenGui

toggle.MouseButton1Click:Connect(function()
	AutoParry.enabled = not AutoParry.enabled
	toggle.Text = "Parry: " .. (AutoParry.enabled and "ON" or "OFF")
	toggle.BackgroundColor3 = AutoParry.enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
end)

game:GetService("RunService").Heartbeat:Connect(function()
	AutoParry:update()
end)

print("[Berri] Loaded for: " .. gameMode)
