-- Macro Recording System for Educational Purposes Only
-- This script creates a UI for recording and playing back player movements

local player = game.Players.LocalPlayer
local userInputService = game:GetService("UserInputService")
local tweenService = game:GetService("TweenService")
local runService = game:GetService("RunService")

-- Setup storage for recordings
local recordings = {}
local recordingCount = 0
local currentRecording = nil
local isRecording = false
local isPlaying = false
local currentFilter = "Oldest to Youngest"

-- UI Creation
local gui = Instance.new("ScreenGui")
gui.Name = "MacroRecorder"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = player.PlayerGui

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0.45, 0, 0.45, 0)
mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.Parent = gui

-- Round corners
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0.02, 0)
uiCorner.Parent = mainFrame

-- Left Panel (Navigation)
local leftPanel = Instance.new("Frame")
leftPanel.Name = "NavPanel"
leftPanel.Size = UDim2.new(0.2, 0, 1, 0)
leftPanel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
leftPanel.BorderSizePixel = 0
leftPanel.Parent = mainFrame

-- Round corners for left panel
local leftPanelCorner = Instance.new("UICorner")
leftPanelCorner.CornerRadius = UDim.new(0.02, 0)
leftPanelCorner.Parent = leftPanel

-- Right Panel (Content)
local rightPanel = Instance.new("Frame")
rightPanel.Name = "ContentPanel"
rightPanel.Size = UDim2.new(0.8, 0, 1, 0)
rightPanel.Position = UDim2.new(0.2, 0, 0, 0)
rightPanel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
rightPanel.BorderSizePixel = 0
rightPanel.Parent = mainFrame

-- Navigation Buttons
local function createNavButton(text, position)
    local button = Instance.new("TextButton")
    button.Name = text.."Button"
    button.Size = UDim2.new(0.9, 0, 0.1, 0)
    button.Position = position
    button.AnchorPoint = Vector2.new(0.5, 0)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.BorderSizePixel = 0
    button.Text = text
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.SourceSansBold
    button.TextSize = 14
    button.Parent = leftPanel
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0.2, 0)
    buttonCorner.Parent = button
    
    return button
end

local macroStartButton = createNavButton("Macro Start", UDim2.new(0.5, 0, 0.05, 0))
local recordsButton = createNavButton("Records", UDim2.new(0.5, 0, 0.18, 0))

-- Macro Start Panel
local macroStartPanel = Instance.new("Frame")
macroStartPanel.Name = "MacroStartPanel"
macroStartPanel.Size = UDim2.new(1, 0, 1, 0)
macroStartPanel.BackgroundTransparency = 1
macroStartPanel.Visible = true
macroStartPanel.Parent = rightPanel

-- Record Name Input
local recordNameBox = Instance.new("TextBox")
recordNameBox.Name = "RecordNameBox"
recordNameBox.Size = UDim2.new(0.8, 0, 0.08, 0)
recordNameBox.Position = UDim2.new(0.5, 0, 0.1, 0)
recordNameBox.AnchorPoint = Vector2.new(0.5, 0)
recordNameBox.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
recordNameBox.BorderSizePixel = 0
recordNameBox.Text = "Enter Record Name..."
recordNameBox.TextColor3 = Color3.fromRGB(200, 200, 200)
recordNameBox.Font = Enum.Font.SourceSans
recordNameBox.TextSize = 18
recordNameBox.ClearTextOnFocus = true
recordNameBox.Parent = macroStartPanel

local recordNameCorner = Instance.new("UICorner")
recordNameCorner.CornerRadius = UDim.new(0.1, 0)
recordNameCorner.Parent = recordNameBox

-- Status Label
local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Size = UDim2.new(0.8, 0, 0.08, 0)
statusLabel.Position = UDim2.new(0.5, 0, 0.25, 0)
statusLabel.AnchorPoint = Vector2.new(0.5, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "⏸️ Paused"
statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
statusLabel.Font = Enum.Font.SourceSansBold
statusLabel.TextSize = 24
statusLabel.Parent = macroStartPanel

-- Record Button
local recordButton = Instance.new("TextButton")
recordButton.Name = "RecordButton"
recordButton.Size = UDim2.new(0.6, 0, 0.12, 0)
recordButton.Position = UDim2.new(0.5, 0, 0.4, 0)
recordButton.AnchorPoint = Vector2.new(0.5, 0)
recordButton.BackgroundColor3 = Color3.fromRGB(80, 180, 80)
recordButton.BorderSizePixel = 0
recordButton.Text = "Start Record"
recordButton.TextColor3 = Color3.fromRGB(255, 255, 255)
recordButton.Font = Enum.Font.SourceSansBold
recordButton.TextSize = 20
recordButton.Parent = macroStartPanel

local recordButtonCorner = Instance.new("UICorner")
recordButtonCorner.CornerRadius = UDim.new(0.1, 0)
recordButtonCorner.Parent = recordButton

-- Records Panel
local recordsPanel = Instance.new("Frame")
recordsPanel.Name = "RecordsPanel"
recordsPanel.Size = UDim2.new(1, 0, 1, 0)
recordsPanel.BackgroundTransparency = 1
recordsPanel.Visible = false
recordsPanel.Parent = rightPanel

-- Filter Button
local filterButton = Instance.new("TextButton")
filterButton.Name = "FilterButton"
filterButton.Size = UDim2.new(0.15, 0, 0.06, 0)
filterButton.Position = UDim2.new(0.95, 0, 0.02, 0)
filterButton.AnchorPoint = Vector2.new(1, 0)
filterButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
filterButton.BorderSizePixel = 0
filterButton.Text = "Filter"
filterButton.TextColor3 = Color3.fromRGB(255, 255, 255)
filterButton.Font = Enum.Font.SourceSans
filterButton.TextSize = 14
filterButton.Parent = recordsPanel

local filterCorner = Instance.new("UICorner")
filterCorner.CornerRadius = UDim.new(0.2, 0)
filterCorner.Parent = filterButton

-- Current Filter Label
local currentFilterLabel = Instance.new("TextLabel")
currentFilterLabel.Name = "CurrentFilterLabel"
currentFilterLabel.Size = UDim2.new(0.4, 0, 0.05, 0)
currentFilterLabel.Position = UDim2.new(0.25, 0, 0.02, 0)
currentFilterLabel.AnchorPoint = Vector2.new(0.5, 0)
currentFilterLabel.BackgroundTransparency = 1
currentFilterLabel.Text = "Filter: Oldest to Youngest"
currentFilterLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
currentFilterLabel.Font = Enum.Font.SourceSans
currentFilterLabel.TextSize = 14
currentFilterLabel.Parent = recordsPanel

-- Recordings ScrollingFrame
local recordingsFrame = Instance.new("ScrollingFrame")
recordingsFrame.Name = "RecordingsFrame"
recordingsFrame.Size = UDim2.new(0.95, 0, 0.85, 0)
recordingsFrame.Position = UDim2.new(0.5, 0, 0.12, 0)
recordingsFrame.AnchorPoint = Vector2.new(0.5, 0)
recordingsFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
recordingsFrame.BorderSizePixel = 0
recordingsFrame.ScrollBarThickness = 6
recordingsFrame.ScrollingDirection = Enum.ScrollingDirection.Y
recordingsFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
recordingsFrame.Parent = recordsPanel

local recordingsCorner = Instance.new("UICorner")
recordingsCorner.CornerRadius = UDim.new(0.02, 0)
recordingsCorner.Parent = recordingsFrame

-- Filter Menu
local filterMenu = Instance.new("Frame")
filterMenu.Name = "FilterMenu"
filterMenu.Size = UDim2.new(0.25, 0, 0.4, 0)
filterMenu.Position = UDim2.new(0.95, 0, 0.09, 0)
filterMenu.AnchorPoint = Vector2.new(1, 0)
filterMenu.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
filterMenu.BorderSizePixel = 0
filterMenu.Visible = false
filterMenu.ZIndex = 10
filterMenu.Parent = recordsPanel

local filterMenuCorner = Instance.new("UICorner")
filterMenuCorner.CornerRadius = UDim.new(0.05, 0)
filterMenuCorner.Parent = filterMenu

local filterOptions = {
    "Alphabetically",
    "Oldest to Youngest",
    "Youngest to Oldest",
    "Favorites"
}

for i, option in ipairs(filterOptions) do
    local optionButton = Instance.new("TextButton")
    optionButton.Name = option.."Button"
    optionButton.Size = UDim2.new(0.9, 0, 0.2, 0)
    optionButton.Position = UDim2.new(0.5, 0, 0.05 + (i-1)*0.23, 0)
    optionButton.AnchorPoint = Vector2.new(0.5, 0)
    optionButton.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    optionButton.BorderSizePixel = 0
    optionButton.Text = option
    optionButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    optionButton.Font = Enum.Font.SourceSans
    optionButton.TextSize = 14
    optionButton.ZIndex = 11
    optionButton.Parent = filterMenu
    
    local optionCorner = Instance.new("UICorner")
    optionCorner.CornerRadius = UDim.new(0.1, 0)
    optionCorner.Parent = optionButton
    
    optionButton.MouseButton1Click:Connect(function()
        currentFilter = option
        currentFilterLabel.Text = "Filter: " .. option
        filterMenu.Visible = false
        refreshRecordingsList()
    end)
end

-- Function to create a recording entry
local function createRecordingEntry(recording, index)
    local entry = Instance.new("Frame")
    entry.Name = "Recording_" .. index
    entry.Size = UDim2.new(0.95, 0, 0, 40)
    entry.Position = UDim2.new(0.5, 0, 0, (index-1) * 45 + 5)
    entry.AnchorPoint = Vector2.new(0.5, 0)
    entry.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    entry.BorderSizePixel = 0
    entry.Parent = recordingsFrame
    
    local entryCorner = Instance.new("UICorner")
    entryCorner.CornerRadius = UDim.new(0.1, 0)
    entryCorner.Parent = entry
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "NameLabel"
    nameLabel.Size = UDim2.new(0.6, 0, 1, 0)
    nameLabel.Position = UDim2.new(0.05, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = recording.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.Font = Enum.Font.SourceSans
    nameLabel.TextSize = 16
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Parent = entry
    
    local starButton = Instance.new("TextButton")
    starButton.Name = "StarButton"
    starButton.Size = UDim2.new(0.1, 0, 1, 0)
    starButton.Position = UDim2.new(0.7, 0, 0, 0)
    starButton.BackgroundTransparency = 1
    starButton.Text = recording.favorite and "⭐" or "☆"
    starButton.TextColor3 = Color3.fromRGB(255, 255, 0)
    starButton.Font = Enum.Font.SourceSans
    starButton.TextSize = 24
    starButton.Parent = entry
    
    local playButton = Instance.new("TextButton")
    playButton.Name = "PlayButton"
    playButton.Size = UDim2.new(0.15, 0, 0.8, 0)
    playButton.Position = UDim2.new(0.9, 0, 0.5, 0)
    playButton.AnchorPoint = Vector2.new(1, 0.5)
    playButton.BackgroundColor3 = Color3.fromRGB(80, 180, 80)
    playButton.BorderSizePixel = 0
    playButton.Text = "Play"
    playButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    playButton.Font = Enum.Font.SourceSansBold
    playButton.TextSize = 14
    playButton.Parent = entry
    
    local playCorner = Instance.new("UICorner")
    playCorner.CornerRadius = UDim.new(0.2, 0)
    playCorner.Parent = playButton
    
    starButton.MouseButton1Click:Connect(function()
        recording.favorite = not recording.favorite
        starButton.Text = recording.favorite and "⭐" or "☆"
        saveRecordings()
    end)
    
    playButton.MouseButton1Click:Connect(function()
        if not isPlaying and not isRecording then
            playRecording(recording)
        end
    end)
    
    return entry
end

-- Function to refresh the recordings list
function refreshRecordingsList()
    for _, child in pairs(recordingsFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    local sortedRecordings = {}
    for _, recording in pairs(recordings) do
        table.insert(sortedRecordings, recording)
    end
    
    if currentFilter == "Alphabetically" then
        table.sort(sortedRecordings, function(a, b)
            return a.name < b.name
        end)
    elseif currentFilter == "Oldest to Youngest" then
        table.sort(sortedRecordings, function(a, b)
            return a.timestamp < b.timestamp
        end)
    elseif currentFilter == "Youngest to Oldest" then
        table.sort(sortedRecordings, function(a, b)
            return a.timestamp > b.timestamp
        end)
    elseif currentFilter == "Favorites" then
        table.sort(sortedRecordings, function(a, b)
            if a.favorite and not b.favorite then
                return true
            elseif not a.favorite and b.favorite then
                return false
            else
                return a.timestamp < b.timestamp
            end
        end)
    end
    
    for i, recording in ipairs(sortedRecordings) do
        createRecordingEntry(recording, i)
    end
    
    recordingsFrame.CanvasSize = UDim2.new(0, 0, 0, #sortedRecordings * 45 + 10)
end

-- Functions to save and load recordings
local function loadRecordings()
    local success, result = pcall(function()
        return player:FindFirstChild("MacroRecordings")
    end)
    
    if success and result then
        local jsonData = result.Value
        local success, decodedData = pcall(function()
            return game:GetService("HttpService"):JSONDecode(jsonData)
        end)
        
        if success then
            recordings = decodedData.recordings
            recordingCount = decodedData.count
        end
    end
end

function saveRecordings()
    local data = {
        recordings = recordings,
        count = recordingCount
    }
    
    local jsonData = game:GetService("HttpService"):JSONEncode(data)
    
    local existingData = player:FindFirstChild("MacroRecordings")
    if not existingData then
        local stringValue = Instance.new("StringValue")
        stringValue.Name = "MacroRecordings"
        stringValue.Value = jsonData
        stringValue.Parent = player
    else
        existingData.Value = jsonData
    end
end

-- Recording functionality
local function startRecording()
    if isPlaying then return end
    
    local recordName = recordNameBox.Text
    if recordName == "" or recordName == "Enter Record Name..." then
        recordingCount = recordingCount + 1
        recordName = "Record-" .. recordingCount
    end
    
    currentRecording = {
        name = recordName,
        actions = {},
        timestamp = os.time(),
        favorite = false
    }
    
    isRecording = true
    statusLabel.Text = "▶️ Recording"
    recordButton.Text = "Stop Record"
    recordButton.BackgroundColor3 = Color3.fromRGB(180, 80, 80)
    
    -- Start recording user input
    local startTime = tick()
    
    local function recordAction(actionType, data)
        if isRecording and currentRecording then
            table.insert(currentRecording.actions, {
                type = actionType,
                time = tick() - startTime,
                data = data
            })
        end
    end
    
    -- Record player movement
    local lastPosition = player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character.HumanoidRootPart.Position
    local lastCameraRotation = workspace.CurrentCamera.CFrame.Rotation
    
    local movementConnection = runService.Heartbeat:Connect(function()
        if not isRecording or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
        
        local currentPosition = player.Character.HumanoidRootPart.Position
        local currentCameraRotation = workspace.CurrentCamera.CFrame.Rotation
        
        if lastPosition and (currentPosition - lastPosition).Magnitude > 0.1 then
            recordAction("move", {
                position = currentPosition
            })
            lastPosition = currentPosition
        end
        
        if lastCameraRotation and math.abs((currentCameraRotation.X - lastCameraRotation.X) + (currentCameraRotation.Y - lastCameraRotation.Y)) > 0.05 then
            recordAction("camera", {
                rotation = currentCameraRotation
            })
            lastCameraRotation = currentCameraRotation
        end
    end)
    
    -- Record input actions
    local inputConnection = userInputService.InputBegan:Connect(function(input, gameProcessed)
        if not isRecording then return end
        
        if input.UserInputType == Enum.UserInputType.Keyboard then
            recordAction("keypress", {
                keyCode = input.KeyCode,
                processed = gameProcessed
            })
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
            recordAction("click", {
                position = userInputService:GetMouseLocation(),
                processed = gameProcessed
            })
        elseif input.UserInputType == Enum.UserInputType.Touch then
            recordAction("touch", {
                position = input.Position,
                processed = gameProcessed
            })
        end
    end)
    
    recordButton.MouseButton1Click:Connect(function()
        if isRecording then
            isRecording = false
            movementConnection:Disconnect()
            inputConnection:Disconnect()
            
            recordings[recordName] = currentRecording
            saveRecordings()
            refreshRecordingsList()
            
            statusLabel.Text = "⏸️ Paused"
            recordButton.Text = "Start Record"
            recordButton.BackgroundColor3 = Color3.fromRGB(80, 180, 80)
            recordNameBox.Text = "Enter Record Name..."
        else
            startRecording()
        end
    end)
end

-- Play back a recording
function playRecording(recording)
    if isRecording or isPlaying then return end
    
    isPlaying = true
    statusLabel.Text = "▶️ Playing"
    
    local startTime = tick()
    local lastActionTime = 0
    
    local function playAction(action)
        if action.type == "move" and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid:MoveTo(action.data.position)
            end
        elseif action.type == "camera" then
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position) * action.data.rotation
        elseif action.type == "keypress" then
            -- Simulate key press (limited by Roblox security)
        elseif action.type == "click" or action.type == "touch" then
            -- Simulate mouse click or touch (limited by Roblox security)
        end
    end
    
    -- Sort actions by time
    local sortedActions = {}
    for _, action in pairs(recording.actions) do
        table.insert(sortedActions, action)
    end
    
    table.sort(sortedActions, function(a, b)
        return a.time < b.time
    end)
    
    -- Play each action at the right time
    local actionIndex = 1
    local connection = runService.Heartbeat:Connect(function()
        local currentTime = tick() - startTime
        
        while actionIndex <= #sortedActions and currentTime >= sortedActions[actionIndex].time do
            playAction(sortedActions[actionIndex])
            actionIndex = actionIndex + 1
        end
        
        if actionIndex > #sortedActions then
            isPlaying = false
            connection:Disconnect()
            statusLabel.Text = "⏸️ Paused"
        end
    end)
end

-- UI Navigation
macroStartButton.MouseButton1Click:Connect(function()
    macroStartPanel.Visible = true
    recordsPanel.Visible = false
})

recordsButton.MouseButton1Click:Connect(function()
    macroStartPanel.Visible = false
    recordsPanel.Visible = true
    refreshRecordingsList()
})

recordButton.MouseButton1Click:Connect(function()
    if not isRecording then
        startRecording()
    else
        isRecording = false
        statusLabel.Text = "⏸️ Paused"
        recordButton.Text = "Start Record"
        recordButton.BackgroundColor3 = Color3.fromRGB(80, 180, 80)
    end
})

filterButton.MouseButton1Click:Connect(function()
    filterMenu.Visible = not filterMenu.Visible
end)

-- Initialize
loadRecordings()

-- Make UI draggable
local isDragging = false
local dragInput
local dragStart
local startPos

local function updateDrag(input)
    local delta = input.Position - dragStart
    mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                isDragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

userInputService.InputChanged:Connect(function(input)
    if input == dragInput and isDragging then
        updateDrag(input)
    end
end)

-- Toggle visibility of the UI
local isVisible = true
userInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightShift then
        isVisible = not isVisible
        gui.Enabled = isVisible
    end
end)
