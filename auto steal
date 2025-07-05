-- GLOBAL PET LIST + AUTO BUY (SECRET ONLY)
-- Author: GPT-Plus (July 2025)
-- ================================================================
-- CONFIG ---------------------------------------------------------
local PET_LIST_REFRESH = 1            -- seconds between rescans
local CLICK_TIMEOUT     = 20          -- hard stop per prompt
local CLICK_RETRY_DELAY = 0.10        -- delay between re‑tries
local DEBUG             = true        -- spam console?

local function log(...)
    if DEBUG then
        print(string.format("[DEBUG %s]", os.date("%H:%M:%S")), ...)
    end
end

-- SERVICES -------------------------------------------------------
local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local UserInput    = game:GetService("UserInputService")
local player       = Players.LocalPlayer

-- SPEED BOOST ----------------------------------------------------
local SPEED_MULTIPLIER = 2.5
local function applySpeedBoost(char)
    local hum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid")
    if math.abs(hum.WalkSpeed - 16) < 0.01 then
        hum.WalkSpeed *= SPEED_MULTIPLIER
    end
end
if player.Character then applySpeedBoost(player.Character) end
player.CharacterAdded:Connect(applySpeedBoost)

-- WAYPOINT (safe anchor‑point while teleporting around) ----------
local WAYPOINT = Vector3.new(-412.153, -6.5, 178.425)

-- ANIMAL DATA (editable Lua table) -------------------------------
local animalsSource = [[ return {
    ["La Vacca Saturno Saturnita"] = {Rarity = "Secret", Price = 50_000_000},
    ["Los Tralaleritos"]          = {Rarity = "Secret", Price =100_000_000},
    ["Graipuss Medussi"]          = {Rarity = "Secret", Price =250_000_000},
    ["La Grande Combinasion"]     = {Rarity = "Secret", Price =1_000_000_000}
} ]]
local Animals = loadstring(animalsSource)()
log("Loaded", (#Animals), "animals")

-- UTILITIES ------------------------------------------------------
local function waitChar()  return player.Character or player.CharacterAdded:Wait() end

local function walkTo(pos)
    local char = waitChar()
    local root = char:WaitForChild("HumanoidRootPart")
    local hum  = char:WaitForChild("Humanoid")
    local last = root.Position
    local stuck = 0
    while (root.Position - pos).Magnitude > 5 do
        hum:MoveTo(pos)
        task.wait(0.15)
        if (root.Position - last).Magnitude < 0.5 then stuck += 0.15 else stuck = 0; last = root.Position end
        if stuck > 4 then
            log("Stuck >4 s ⇒ respawn")
            char:BreakJoints()
            return false
        end
    end
    return true
end

local function findMyPlot(waitSpawn)
    local deadline = tick() + (waitSpawn and 10 or 0)
    repeat
        for _, plot in ipairs(workspace.Plots:GetChildren()) do
            local owner = plot:FindFirstChild("Owner")
            if owner and owner.Value == player then return plot end
            local sign = plot:FindFirstChild("PlotSign")
            local lbl  = sign and sign.SurfaceGui.Frame:FindFirstChild("TextLabel")
            if lbl and (lbl.Text:lower():find(player.Name:lower())
                     or lbl.Text:lower():find(player.DisplayName:lower())) then
                return plot
            end
        end
        task.wait(0.25)
    until tick() > deadline
end

local function getPetDataFromSpawn(sp)
    local at      = sp and sp:FindFirstChild("Attachment")
    local overhead= at and at:FindFirstChild("AnimalOverhead")
    local lbl     = overhead and overhead:FindFirstChild("DisplayName")
    local name    = lbl and lbl.Text
    if not name or name == "" then return nil end
    local d = Animals[name] or {}
    return {name = name, rar = d.Rarity or "?", price = d.Price or 0}
end

local function rarityColorHex(r)
    if     r == "Secret"        then return "#ff3c3c"
    elseif r == "Brainrot God"  then return "#ffa500"
    elseif r == "Mythic"        then return "#e35bff"
    elseif r == "Legendary"     then return "#fdd835"
    elseif r == "Epic"          then return "#6ab7ff"
    elseif r == "Rare"          then return "#7cfc00"
    else                             return "#ffffff" end
end

local function stepOn(part, label)
    if not part then log("Missing part:", label); return false end
    local root   = waitChar():WaitForChild("HumanoidRootPart")
    local target = Vector3.new(part.Position.X, root.Position.Y, part.Position.Z)
    log("StepOn →", label)
    if not walkTo(target) then return false end
    root.CFrame = root.CFrame - Vector3.new(0,2,0) -- slight push‑through
    task.wait(0.05)
    return true
end

-- GUI ------------------------------------------------------------
local guiPetList, petListContainer
local allButtons, isBusy = {}, false
local function createGUI()
    if player.PlayerGui:FindFirstChild("AllPetsHUD") then
        player.PlayerGui.AllPetsHUD:Destroy()
    end
    guiPetList                 = Instance.new("ScreenGui")
    guiPetList.Name            = "AllPetsHUD"
    guiPetList.IgnoreGuiInset  = true
    guiPetList.ResetOnSpawn    = false
    guiPetList.ZIndexBehavior  = Enum.ZIndexBehavior.Global
    guiPetList.Parent          = player.PlayerGui

    petListContainer           = Instance.new("ScrollingFrame")
    petListContainer.Name      = "List"
    petListContainer.Size      = UDim2.new(0.3, 0, 0.9, 0)
    petListContainer.Position  = UDim2.new(0.68, 0, 0.05, 0)
    petListContainer.AutomaticCanvasSize = Enum.AutomaticSize.Y
    petListContainer.ScrollBarThickness   = 6
    petListContainer.BackgroundTransparency = 0.25
    petListContainer.BackgroundColor3       = Color3.fromRGB(20,20,20)
    petListContainer.BorderSizePixel        = 0
    petListContainer.Parent = guiPetList

    local layout = Instance.new("UIListLayout")
    layout.FillDirection = Enum.FillDirection.Vertical
    layout.SortOrder     = Enum.SortOrder.LayoutOrder
    layout.Padding       = UDim.new(0,4)
    layout.Parent        = petListContainer
end
createGUI()

-- SCAN ALL OTHER PLOTS FOR SECRET PETS --------------------------
local function scanPets()
    local mine = findMyPlot()
    local list = {}
    for _, plot in ipairs(workspace.Plots:GetChildren()) do
        if plot == mine then continue end
        local podFolder = plot:FindFirstChild("AnimalPodiums")
        if not podFolder then continue end
        for _, pod in ipairs(podFolder:GetChildren()) do
            local sp   = pod:FindFirstChild("Base") and pod.Base:FindFirstChild("Spawn")
            local info = getPetDataFromSpawn(sp)
            if info and info.rar == "Secret" then
                table.insert(list, {plot = plot, spawn = sp, info = info})
            end
        end
    end
    return list
end

-- RENDER BUTTONS -------------------------------------------------
local function rebuildButtons()
    for _, rec in ipairs(allButtons) do
        if rec.button then rec.button:Destroy() end
    end
    allButtons = {}

    for _, pet in ipairs(scanPets()) do
        local btn           = Instance.new("TextButton")
        btn.Size            = UDim2.new(1,-8,0,28)
        btn.BackgroundColor3= Color3.fromRGB(35,35,35)
        btn.TextColor3      = Color3.new(1,1,1)
        btn.TextSize        = 20
        btn.TextXAlignment  = Enum.TextXAlignment.Left
        btn.RichText        = true
        btn.BorderSizePixel = 0
        btn.Text = string.format("<font color='%s'>%s</font>  [%s]", rarityColorHex(pet.info.rar), pet.info.name, pet.info.rar)
        btn.Parent = petListContainer
        table.insert(allButtons, {button = btn, pet = pet})
    end
end

-- INTERACT PROMPT (KEEP "HOLDING E" UNTIL SUCCESS) --------------
local function interactPrompt(prompt)
    if not prompt or not prompt:IsA("ProximityPrompt") then return false end

    local completed = false
    local conn = prompt.Triggered:Connect(function()
        completed = true
    end)

    local deadline = tick() + CLICK_TIMEOUT
    while not completed and tick() < deadline do
        -- If the prompt requires holding, supply its HoldDuration to fireproximityprompt
        local hold = prompt.HoldDuration or 0
        if hold > 0 then
            fireproximityprompt(prompt, hold + 0.05) -- hold slightly longer
            task.wait(hold + 0.1)
        else
            fireproximityprompt(prompt)
            task.wait(CLICK_RETRY_DELAY)
        end
    end

    conn:Disconnect()
    return completed
end

-- HANDLE BUTTON CLICK -------------------------------------------
local function handleClick(petObj)
    if isBusy then log("Busy, click ignored"); return end
    isBusy = true
    log("========== NEW TASK ==========", petObj.info.name, "("..petObj.info.rar..")")

    local destPlot = petObj.plot
    local destBox  = destPlot:FindFirstChild("DeliveryHitbox") or destPlot:FindFirstChild("DeliveryBox")
    local petSpawn = petObj.spawn
    local myPlot   = findMyPlot(true)
    local myBox    = myPlot and (myPlot:FindFirstChild("DeliveryHitbox") or myPlot:FindFirstChild("DeliveryBox"))
    local hrpY     = waitChar().HumanoidRootPart.Position.Y

    if not walkTo(Vector3.new(WAYPOINT.X, hrpY, WAYPOINT.Z)) then isBusy = false return end
    if not stepOn(destBox, "DestBox‑1") then isBusy = false return end
    if not walkTo(Vector3.new(petSpawn.Position.X, hrpY, petSpawn.Position.Z)) then isBusy = false return end

    local prompt = (petSpawn:FindFirstChild("PromptAttachment") and petSpawn.PromptAttachment:FindFirstChildWhichIsA("ProximityPrompt"))
                or petSpawn.Parent:FindFirstChildWhichIsA("ProximityPrompt", true)

    if prompt then
        log("Interacting prompt …")
        if interactPrompt(prompt) then
            log("Prompt completed ✔")
        else
            log("❌ Prompt failed or timed out")
            isBusy = false; return
        end
    else
        log("❌ Prompt not found.")
        isBusy = false; return
    end
    task.wait(0.2)

    if not stepOn(destBox, "DestBox‑2") then isBusy = false; return end
    if not walkTo(Vector3.new(WAYPOINT.X, hrpY, WAYPOINT.Z)) then isBusy = false; return end
    if myBox then stepOn(myBox, "MyBox") end

    log("========== TASK FINISHED ==========")
    isBusy = false
end

-- CONNECT GUI BUTTONS -------------------------------------------
local function connectClicks()
    for _, rec in ipairs(allButtons) do
        rec.button.MouseButton1Click:Connect(function()
            handleClick(rec.pet)
        end)
    end
end

-- AUTO‑REFRESH LOOP ---------------------------------------------
RunService.Heartbeat:Connect(function()
    -- prevent potential duplicate loops if multiple copies running
    if not guiPetList then return end
end)

task.spawn(function()
    while true do
        rebuildButtons()
        connectClicks()
        task.wait(PET_LIST_REFRESH)
    end
end)

-- CINEMATIC CAMERA SPIN (fun) -----------------------------------
local function spinCam()
    local camera = workspace.CurrentCamera
    local radius, height, speed = 10, 5, 0.5
    local angle = 0
    RunService.RenderStepped:Connect(function(dt)
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local hrp = char.HumanoidRootPart
            camera.CameraType = Enum.CameraType.Scriptable
            angle += speed * dt
            camera.CFrame = CFrame.new(
                hrp.Position + Vector3.new(math.cos(angle) * radius, height, math.sin(angle) * radius),
                hrp.Position
            )
        end
    end)
end
spinCam()
local noclipEnabled = true
UserInput.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.N then
        noclipEnabled = not noclipEnabled
    end
end)

RunService.Stepped:Connect(function()
    if not noclipEnabled then return end
    local char = player.Character or player.CharacterAdded:Wait()
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end)

log("GlobalPetList Secret‑Only script loaded ✓")
