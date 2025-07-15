-- Dentro da função fly(), logo após local desiredY = ...

local reachedHeight = false
local reachedSpawn = false

-- Ponto de spawn original
local spawnPosition = Workspace:FindFirstChild("SpawnLocation") and Workspace.SpawnLocation.Position or character:GetPivot().Position

flyConnection = RunService.RenderStepped:Connect(function()
    if not flying then return end

    local rayOrigin = humanoidRootPart.Position
    local rayDirection = Vector3.new(0, -200, 0)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {character}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    local groundY = raycastResult and raycastResult.Position.Y or 0
    local desiredY = groundY + fixedHeight

    -- Movimentação para altura fixa
    if not reachedHeight then
        bodyPosition.Position = Vector3.new(
            humanoidRootPart.Position.X,
            desiredY,
            humanoidRootPart.Position.Z
        )
        if math.abs(humanoidRootPart.Position.Y - desiredY) < 2 then
            reachedHeight = true
        end
    elseif not reachedSpawn then
        -- Voa em direção ao ponto de spawn
        local currentPos = humanoidRootPart.Position
        local targetPos = Vector3.new(spawnPosition.X, desiredY, spawnPosition.Z)
        local direction = (targetPos - currentPos).Unit
        local distance = (targetPos - currentPos).Magnitude

        if distance < 3 then
            reachedSpawn = true
        else
            bodyPosition.Position = currentPos + direction * math.min(speed/10, distance)
        end
    else
        -- Modo livre (WASD)
        local move = Vector3.new(control.L + control.R, 0, control.F + control.B)
        local camCFrame = Workspace.CurrentCamera.CFrame
        local direction = move.Magnitude > 0 and camCFrame:VectorToWorldSpace(move.Unit) * speed or Vector3.zero

        bodyPosition.Position = Vector3.new(
            humanoidRootPart.Position.X + direction.X,
            desiredY,
            humanoidRootPart.Position.Z + direction.Z
        )
        bodyGyro.CFrame = camCFrame
    end
end)
