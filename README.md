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

local ItemsTab = Window:CreateTab("Items", 4483362458) -- Title, Image

local Toggle = ItemsTab:CreateToggle({
	Name = "Get all items",
	CurrentValue = false,
	Flag = "Toggle1",
	Callback = function(Value)
		local Players = game:GetService("Players")
		local VirtualInputManager = game:GetService("VirtualInputManager")

		local player = Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		local hrp = character:WaitForChild("HumanoidRootPart")
		local humanoid = character:WaitForChild("Humanoid")
		local itemsFolder = workspace:WaitForChild("Items")

		local running = Value
		local stopId = 0

		if running then
			Rayfield:Notify({
				Title = "Warning",
				Content = "Please cancel when there are 2 seconds left before the round starts to avoid being kicked (in the lobby)",
				Duration = 6.5,
				Image = "rewind",
			})
		end

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
			return closest, shortest
		end

		local function waitUntilRemoved(item)
			while running and item.Parent == itemsFolder do
				task.wait(0.05)
			end
		end

		local function moveTowards(targetPos, id)
			humanoid.PlatformStand = true
			local startTime = tick()
			local initialDistance = (hrp.Position - targetPos).Magnitude

			if initialDistance <= 50 then
				hrp.CFrame = CFrame.new(targetPos + Vector3.new(0, 0.7, 0))
			else
				while running and stopId == id and (hrp.Position - targetPos).Magnitude > 1 do
					local dir = (targetPos - hrp.Position).Unit
					local step = dir * 12
					if step.Magnitude > (targetPos - hrp.Position).Magnitude then
						step = targetPos - hrp.Position
					end
					hrp.CFrame = CFrame.new(hrp.Position + step)
					task.wait(0.01)
					if tick() - startTime > 10 and (hrp.Position - targetPos).Magnitude > 350 then
						running = false
						break
					end
				end
			end
			humanoid.PlatformStand = false
		end

		local function pressF()
			VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
			task.wait(0.05)
			VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
		end

		if running then
			stopId += 1
			local id = stopId

			task.spawn(function()
				while running and #itemsFolder:GetChildren() > 0 do
					local item, dist = getClosestItem()
					if not item then break end
					local pos = getItemPosition(item)
					if not pos then break end

					moveTowards(pos, id)
					if not running then break end

					task.wait(0.15)
					if (hrp.Position - pos).Magnitude <= 6 then
						pressF()
					end

					waitUntilRemoved(item)
					task.wait(0.5)
				end
				running = false
				humanoid.PlatformStand = false
			end)
		end
	end,
})
