local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Slap Royale",
   Icon = 0, -- Icon in Topbar. Can use Lucide Icons (string) or Roblox Image (number). 0 to use no icon (default).
   LoadingTitle = "Loading yeahh",
   LoadingSubtitle = "by Not Hub",
   ShowText = "Slap Royale Script", -- for mobile users to unhide rayfield, change if you'd like
   Theme = "Default", -- Check https://docs.sirius.menu/rayfield/configuration/themes

   ToggleUIKeybind = "K", -- The keybind to toggle the UI visibility (string like "K" or Enum.KeyCode)

   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false, -- Prevents Rayfield from warning when the script has a version mismatch with the interface

   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil, -- Create a custom folder for your hub/game
      FileName = "Big Hub right"
   },

   Discord = {
      Enabled = false, -- Prompt the user to join your Discord server if their executor supports it
      Invite = "noinvitelink", -- The Discord invite code, do not include discord.gg/. E.g. discord.gg/ ABCD would be ABCD
      RememberJoins = true -- Set this to false to make them join the discord every time they load it up
   },

   KeySystem = false, -- Set this to true to use our key system
   KeySettings = {
      Title = "Untitled",
      Subtitle = "Key System",
      Note = "No method of obtaining the key is provided", -- Use this to tell the user how to get a key
      FileName = "Key", -- It is recommended to use something unique as other scripts using Rayfield may overwrite your key file
      SaveKey = true, -- The user's key will be saved, but if you change the key, they will be unable to use your script
      GrabKeyFromSite = false, -- If this is true, set Key below to the RAW site you would like Rayfield to get the key from
      Key = {"Hello"} -- List of keys that will be accepted by the system, can be RAW file links (pastebin, github etc) or simple strings ("hello","key22")
   }
})


local MainTab = Window:CreateTab("Main", 4483362458) -- Title, Image

local Section = MainTab:CreateSection("Welcome! This is a Slap royale game exploit. It's very OP!")
local Section = MainTab:CreateSection("Remember, don't equip items if you're in the lobby. Or you'll get kicked out, silly little thing")


local ItemsTab = Window:CreateTab("Items", 4483362458) -- Title, Image

local Button = ItemsTab:CreateButton({
	Name = "Get all bombs",
	Callback = function()
		local Players = game:GetService("Players")
		local VirtualInputManager = game:GetService("VirtualInputManager")

		local player = Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		local hrp = character:WaitForChild("HumanoidRootPart")
		local itemsFolder = workspace:WaitForChild("Items")

		-- Toggle
		if _G.GetBombsRunning then
			_G.GetBombsStopId += 1
			_G.GetBombsRunning = false
			return
		end

		_G.GetBombsRunning = true
		_G.GetBombsStopId = (_G.GetBombsStopId or 0) + 1
		local id = _G.GetBombsStopId

		Rayfield:Notify({
			Title = "Warning",
			Content = "Please cancel when there are 2 seconds left before the round starts to avoid being kicked (in the lobby)",
			Duration = 6.5,
			Image = "rewind",
		})

		local function getItemPos(item)
			if item:IsA("BasePart") then
				return item.Position
			elseif item:IsA("Model") then
				if item.PrimaryPart then
					return item.PrimaryPart.Position
				end
				local p = item:FindFirstChildWhichIsA("BasePart")
				if p then return p.Position end
			end
		end

		local function pressF()
			VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
			task.wait(0.05)
			VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
		end

		local function teleportAndPickup(item)
			local pos = getItemPos(item)
			if not pos then return end

			hrp.CFrame = CFrame.new(pos + Vector3.new(0, 0.7, 0))
			task.wait(0.25)

			if _G.GetBombsStopId ~= id then return end
			pressF()

			while _G.GetBombsStopId == id and item.Parent == itemsFolder do
				task.wait(0.05)
			end
		end

		task.spawn(function()
			-- 1️⃣ Forcefield Crystal
			for _, item in ipairs(itemsFolder:GetChildren()) do
				if item.Name:lower():find("forcefield crystal") then
					teleportAndPickup(item)
					break
				end
			end

			-- 2️⃣ Todas as Bombs
			while _G.GetBombsStopId == id do
				local bombFound = false

				for _, item in ipairs(itemsFolder:GetChildren()) do
					if item.Name:lower():find("bomb") then
						bombFound = true
						teleportAndPickup(item)
						task.wait(0.1)
						break
					end
				end

				if not bombFound then break end
			end

			_G.GetBombsRunning = false
		end)
	end,
})

local Button = ItemsTab:CreateButton({
	Name = "Bomb Bus",
	Callback = function()
		local Players = game:GetService("Players")
		local player = Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		local humanoid = character:WaitForChild("Humanoid")
		local backpack = player:WaitForChild("Backpack")

		local crystal = backpack:FindFirstChild("Forcefield Crystal")
		if crystal then
			humanoid:EquipTool(crystal)
			task.wait(0.01)
			crystal:Activate()
			task.wait(0.01)
		else
			Rayfield:Notify({
				Title = "Warning",
				Content = "That guy is so poor he doesn't even have a Forcefield Crystal, LOL!",
				Duration = 5,
				Image = "rewind",
			})
			return
		end

		local bombs = {}
		for _, tool in ipairs(backpack:GetChildren()) do
			if tool:IsA("Tool") and tool.Name == "Bomb" then
				table.insert(bombs, tool)
			end
		end

		if #bombs < 1 then
			Rayfield:Notify({
				Title = "Warning",
				Content = "You don't have bombs. 🥀",
				Duration = 5,
				Image = "rewind",
			})
			return
		end

		for _, bomb in ipairs(bombs) do
			bomb.Parent = character
		end

		for _, bomb in ipairs(bombs) do
			humanoid:EquipTool(bomb)
			task.wait(0.01)
			bomb:Activate()
		end
	end,
})

local Button = ItemsTab:CreateButton({
	Name = "Get all items (May kick)",
	Callback = function()
		local Players = game:GetService("Players")
		local VirtualInputManager = game:GetService("VirtualInputManager")
		local player = Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		local hrp = character:WaitForChild("HumanoidRootPart")
		local humanoid = character:WaitForChild("Humanoid")
		local itemsFolder = workspace:WaitForChild("Items")

		if _G.GetItemsRunning then
			_G.GetItemsStopId = (_G.GetItemsStopId or 0) + 1
			_G.GetItemsRunning = false
			humanoid.PlatformStand = false
			return
		end

		_G.GetItemsRunning = true
		_G.GetItemsStopId = (_G.GetItemsStopId or 0) + 1
		local id = _G.GetItemsStopId

		Rayfield:Notify({
			Title = "Warning",
			Content = "When there are 2 seconds left before the match starts, cancel the Get all items option.",
			Duration = 6.5,
			Image = "rewind",
		})

		local function getItemPosition(item)
			if item:IsA("BasePart") then return item.Position end
			if item:IsA("Model") then
				if item.PrimaryPart then return item.PrimaryPart.Position end
				local part = item:FindFirstChildWhichIsA("BasePart")
				if part then return part.Position end
			end
		end

		local function getClosestItem()
			local closest
			local shortest = math.huge
			for _, item in ipairs(itemsFolder:GetChildren()) do
				local pos = getItemPosition(item)
				if pos then
					local dist = (hrp.Position - pos).Magnitude
					if dist < shortest then
						shortest = dist
						closest = item
					end
				end
			end
			return closest
		end

		local function pressF()
			VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
			task.wait(0.05)
			VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
		end

		task.spawn(function()
			while _G.GetItemsStopId == id and #itemsFolder:GetChildren() > 0 do
				local item = getClosestItem()
				if not item then break end
				local pos = getItemPosition(item)
				if not pos then break end

				hrp.CFrame = CFrame.new(pos + Vector3.new(0, 0.7, 0))
				if _G.GetItemsStopId ~= id then break end

				task.wait(0.25)
				if (hrp.Position - pos).Magnitude <= 6 then pressF() end

				while _G.GetItemsStopId == id and item.Parent == itemsFolder do
					task.wait(0.05)
				end

				if _G.GetItemsStopId ~= id then break end
				task.wait(0.25)
			end
			_G.GetItemsRunning = false
			humanoid.PlatformStand = false
		end)
	end,
})

local Button = ItemsTab:CreateButton({
	Name = "Check Bombs",
	Callback = function()
		local itemsFolder = workspace:WaitForChild("Items")
		local bombsCount = 0

		for _, item in ipairs(itemsFolder:GetChildren()) do
			if item.Name:lower():find("bomb") then
				bombsCount += 1
			end
		end

		if bombsCount == 7 then
			Rayfield:Notify({
				Title = "Bomb Info",
				Content = "There are 7 bombs scattered across the map; this will kill everyone",
				Duration = 5,
				Image = "rewind",
			})
		else
			Rayfield:Notify({
				Title = "Bomb Info",
				Content = "There are "..bombsCount.." bombs on the map",
				Duration = 5,
				Image = "rewind",
			})
		end
	end,
})



local PlayerTab = Window:CreateTab("Player", 4483362458) -- Title, Image

local Button = PlayerTab:CreateButton({
	Name = "Jump off the bus early",
	Callback = function()
		if _G.BusJumped then
			Rayfield:Notify({
				Title = "Info",
				Content = "You've already gotten off the bus. Or You just clicked, not on the bus, lol",
				Duration = 5,
				Image = "rewind",
			})
			return
		end

		_G.BusJumped = true
		local ReplicatedStorage = game:GetService("ReplicatedStorage")
		local busEvent = ReplicatedStorage:WaitForChild("Events"):WaitForChild("BusJumping")
		busEvent:FireServer()
	end,
})

local Button = PlayerTab:CreateButton({
	Name = "stronger FOV",
	Callback = function()
		local Players = game:GetService("Players")
		local player = Players.LocalPlayer
		local camera = workspace.CurrentCamera

		if not _G.FOVToggled then
			camera.FieldOfView = 120
			_G.FOVToggled = true
		else
			camera.FieldOfView = 70
			_G.FOVToggled = false
		end
	end,
})

local Button = PlayerTab:CreateButton({
	Name = "Debug everything from the player",
	Callback = function()
		local Players = game:GetService("Players")
		local player = Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		local humanoid = character:WaitForChild("Humanoid")

		if _G.WalkSpeedOriginal == nil then
			_G.WalkSpeedOriginal = humanoid.WalkSpeed
		end

		if not _G.WalkSpeedToggled then
			humanoid.WalkSpeed = 16
			_G.WalkSpeedToggled = true
		else
			humanoid.WalkSpeed = _G.WalkSpeedOriginal
			_G.WalkSpeedToggled = false
		end
	end,
})
