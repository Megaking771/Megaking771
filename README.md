-- Script: put this inside a Model named "Dummy" that has a Humanoid and HumanoidRootPart
-- Coloque o Model no Workspace (por exemplo, em ServerStorage e depois clone para teste)

local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")

local model = script.Parent
local humanoid = model:WaitForChild("Humanoid")
local root = model:WaitForChild("HumanoidRootPart")

local ATTACK_RANGE = 4           -- alcance de ataque
local ATTACK_DAMAGE = 10         -- dano por ataque
local ATTACK_COOLDOWN = 1.2      -- segundos entre ataques

local function findNearestPlayer()
    local nearestPlayer = nil
    local minDist = math.huge
    for _, p in pairs(Players:GetPlayers()) do
        local ch = p.Character
        if ch and ch:FindFirstChild("Humanoid") and ch.Humanoid.Health > 0 and ch:FindFirstChild("HumanoidRootPart") then
            local dist = (ch.HumanoidRootPart.Position - root.Position).Magnitude
            if dist < minDist then
                minDist = dist
                nearestPlayer = p
            end
        end
    end
    return nearestPlayer, minDist
end

local lastAttack = 0

while humanoid and humanoid.Health > 0 do
    local targetPlayer, dist = findNearestPlayer()
    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetPos = targetPlayer.Character.HumanoidRootPart.Position
        local path = PathfindingService:CreatePath()
        local success, err = pcall(function()
            path:ComputeAsync(root.Position, targetPos)
        end)
        if success then
            local waypoints = path:GetWaypoints()
            for _, wp in ipairs(waypoints) do
                if humanoid.Health <= 0 then break end
                humanoid:MoveTo(wp.Position)
                local ok = humanoid.MoveToFinished:Wait()
                -- Checa ataque a cada passo
                if targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid") and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    local curDist = (targetPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude
                    if curDist <= ATTACK_RANGE and tick() - lastAttack >= ATTACK_COOLDOWN then
                        targetPlayer.Character.Humanoid:TakeDamage(ATTACK_DAMAGE)
                        lastAttack = tick()
                    end
                else
                    break
                end
            end
        else
            -- fallback: mover direto
            humanoid:MoveTo(targetPos)
            humanoid.MoveToFinished:Wait()
        end
    else
        wait(1) -- sem jogador perto, pausa
    end
    wait(0.1)
end
