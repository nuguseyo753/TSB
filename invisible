if not game:IsLoaded() then
    game.Loaded:Wait()
end

local Players = cloneref(game:GetService("Players"))
local RunService = cloneref(game:GetService("RunService"))
local UserInputService = cloneref(game:GetService("UserInputService"))
local Workspace = cloneref(game:GetService("Workspace"))

local LocalPlayer = Players.LocalPlayer

local TOGGLE_KEY = Enum.KeyCode.V
local ANIMATION_ID = "rbxassetid://107114358965793"
local ANIMATION_TIME = 18
local FADED_TRANSPARENCY = 0.5

local state = {
    enabled = false,
    track = nil,
    trackHumanoid = nil,
    savedTransparency = {},
}

local function getCharacter()
    return LocalPlayer.Character
end

local function getHumanoid(character)
    if not character then
        return nil
    end

    return character:FindFirstChildOfClass("Humanoid")
end

local function getRootPart(character)
    if not character then
        return nil
    end

    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if rootPart and rootPart:IsA("BasePart") then
        return rootPart
    end

    return nil
end

local function getCamera()
    local camera = Workspace.CurrentCamera
    if camera and camera:IsA("Camera") then
        return camera
    end

    return nil
end

local function isHitbox(instance)
    return instance.Name:find("Hitbox_", 1, true) ~= nil
end

local function rememberTransparency(instance)
    if state.savedTransparency[instance] == nil then
        state.savedTransparency[instance] = instance.Transparency
    end
end

local function applyFade(instance)
    rememberTransparency(instance)

    local original = state.savedTransparency[instance]
    if original == nil then
        return
    end

    instance.Transparency = original == 0 and FADED_TRANSPARENCY or original
end

local function restoreTransparency(character)
    for instance, original in pairs(state.savedTransparency) do
        if instance:IsDescendantOf(character) then
            if instance:IsA("BasePart") or instance:IsA("Decal") or instance:IsA("Texture") then
                instance.Transparency = original
            end

            state.savedTransparency[instance] = nil
        end
    end
end

local function applyTransparency(character, rootPart)
    for _, descendant in character:GetDescendants() do
        if descendant:IsA("BasePart") then
            if descendant == rootPart then
                continue
            end

            if isHitbox(descendant) then
                rememberTransparency(descendant)
                descendant.Transparency = 1
            else
                applyFade(descendant)
            end
        elseif descendant:IsA("Decal") or descendant:IsA("Texture") then
            local parent = descendant.Parent
            if parent and isHitbox(parent) then
                continue
            end

            applyFade(descendant)
        end
    end
end

local function getAnimationTrack(humanoid)
    if state.track and state.trackHumanoid == humanoid then
        return state.track
    end

    if state.track then
        state.track:Destroy()
        state.track = nil
        state.trackHumanoid = nil
    end

    local animator = humanoid:FindFirstChildOfClass("Animator")
    if not animator then
        animator = Instance.new("Animator")
        animator.Parent = humanoid
    end

    local animation = Instance.new("Animation")
    animation.AnimationId = ANIMATION_ID

    local ok, track = pcall(function()
        return animator:LoadAnimation(animation)
    end)

    animation:Destroy()

    if not ok or not track then
        return nil
    end

    track.Priority = Enum.AnimationPriority.Action4
    state.track = track
    state.trackHumanoid = humanoid

    return track
end

local function setCameraSubject(subject)
    local camera = getCamera()
    if not camera or not subject then
        return
    end

    if camera.CameraSubject ~= subject then
        camera.CameraSubject = subject
    end
end

local function restoreCameraSubject(character)
    local camera = getCamera()
    local humanoid = getHumanoid(character)
    if not camera or not humanoid then
        return
    end

    camera.CameraSubject = humanoid
end

local function stopTrack()
    local track = state.track
    if track and track.IsPlaying then
        track:Stop(0)
    end
end

local function resetState()
    local character = getCharacter()
    if character then
        restoreTransparency(character)
        restoreCameraSubject(character)
    end

    stopTrack()
end

LocalPlayer.CharacterAdded:Connect(function()
    if state.track then
        state.track:Destroy()
        state.track = nil
        state.trackHumanoid = nil
    end

    table.clear(state.savedTransparency)
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed or input.KeyCode ~= TOGGLE_KEY then
        return
    end

    state.enabled = not state.enabled

    if not state.enabled then
        resetState()
    end
end)

RunService.Heartbeat:Connect(function()
    local character = getCharacter()
    local humanoid = getHumanoid(character)
    local rootPart = getRootPart(character)

    if not character or not humanoid or not rootPart then
        return
    end

    if not state.enabled then
        restoreTransparency(character)
        restoreCameraSubject(character)
        stopTrack()
        return
    end

    local track = getAnimationTrack(humanoid)
    if not track then
        return
    end

    local savedCFrame = rootPart.CFrame
    local savedVelocity = rootPart.AssemblyLinearVelocity
    local savedAngularVelocity = rootPart.AssemblyAngularVelocity

    setCameraSubject(rootPart)

    if not track.IsPlaying then
        track:Play(0)
    end

    track.TimePosition = ANIMATION_TIME
    track:AdjustSpeed(0)
    track:AdjustWeight(1, 0)

    applyTransparency(character, rootPart)

    RunService.RenderStepped:Wait()

    stopTrack()
    rootPart.CFrame = savedCFrame
    rootPart.AssemblyLinearVelocity = savedVelocity
    rootPart.AssemblyAngularVelocity = savedAngularVelocity
end)
