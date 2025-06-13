local plr = game.Players.LocalPlayer
local mouse = plr:GetMouse()
local character = plr.Character or plr.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local accessory = nil
local npcTarget = nil
local connection = nil

-- Encontra o primeiro acessório do jogador local
for _, v in pairs(character:GetChildren()) do
    if v:IsA("Accessory") and v:FindFirstChild("Handle") then
        accessory = v.Handle
        break
    end
end

if not accessory then
    warn("Nenhum acessório encontrado!")
    return
end

-- Função para definir a gravidade do jogador local
local function setGravity(value)
    game:GetService("Workspace").Gravity = value
end

-- Função para verificar se um modelo é realmente um NPC
local function isNPC(model)
    if not model:IsA("Model") then return false end

    local humanoid = model:FindFirstChildOfClass("Humanoid")
    local hrp = model:FindFirstChild("HumanoidRootPart")

    -- Se não tiver humanoid ou HRP, não é um NPC válido
    if not humanoid or not hrp then
        return false
    end

    -- Verifica se pertence a um jogador (para evitar confusão)
    if game.Players:GetPlayerFromCharacter(model) then
        return false
    end

    -- Se passou por todos os testes, é um NPC
    return true
end

mouse.Button1Down:Connect(function()
    local target = mouse.Target
    if target and target.Parent and isNPC(target.Parent) then
        local npc = target.Parent
        local hrp = npc:FindFirstChild("HumanoidRootPart")

        if hrp then
            -- Se já há um NPC seguindo, para o antigo
            if connection then
                connection:Disconnect()
            end

            -- Define o novo NPC alvo
            npcTarget = hrp

            -- Desativa a colisão do acessório e zera a gravidade do jogador local
            accessory.CanCollide = false
            setGravity(0)

            -- Faz o acessório seguir o NPC continuamente
            connection = game:GetService("RunService").Heartbeat:Connect(function()
                if npcTarget and accessory then
                    accessory.Position = npcTarget.Position
                end
            end)
        end
    end
end)

-- Criar botão "Parar o seguimento"
local gui = Instance.new("ScreenGui", plr.PlayerGui)
local stopButton = Instance.new("TextButton", gui)
stopButton.Size = UDim2.new(0, 150, 0, 50)
stopButton.Position = UDim2.new(0.4, 0, 0.9, 0)
stopButton.Text = "Parar o seguimento"
stopButton.BackgroundColor3 = Color3.new(1, 0, 0)
stopButton.TextColor3 = Color3.new(1, 1, 1)

stopButton.MouseButton1Click:Connect(function()
    if connection then
        connection:Disconnect()
    end
    npcTarget = nil

    -- Restaura a gravidade do jogador local para o padrão (196.2)
    setGravity(196.2)
end)

if replicatesignal then
    replicatesignal(game.Players.LocalPlayer.ConnectDiedSignalBackend)
else
    print("Não foi possível ativar o Permadeath: replicatesignal não é suportado.")
end

wait(game.Players.RespawnTime + 0.1)
local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
if humanoid then
  
end

local fph = workspace.FallenPartsDestroyHeight  
  
local plr = game.Players.LocalPlayer  
local character = plr.Character  
local hrp = character:WaitForChild("HumanoidRootPart")  
local torso = character:FindFirstChild("UpperTorso") or character:FindFirstChild("Torso")  
local start = hrp.CFrame  

-- **Teleporta o jogador para 1500 studs de altura**
local teleportHeight = 1500
hrp.CFrame = CFrame.new(start.Position + Vector3.new(0, teleportHeight, 0))

-- **Espera 0.2 segundos antes de continuar**
task.wait(0.2)

local campart = Instance.new("Part", character)  
campart.Transparency = 1  
campart.CanCollide = false  
campart.Size = Vector3.one  
campart.Position = start.Position  
campart.Anchored = true  
  
local function updatestate(hat,state)  
    if sethiddenproperty then  
        sethiddenproperty(hat,"BackendAccoutrementState",state)  
    elseif setscriptable then  
        setscriptable(hat,"BackendAccoutrementState",true)  
        hat.BackendAccoutrementState = state  
    else  
        local success = pcall(function()  
            hat.BackendAccoutrementState = state  
        end)  
        if not success then  
            error("executor not supported, sorry!")  
        end  
    end  
end  
  
local allhats = {}  
for i,v in pairs(character:GetChildren()) do  
    if v:IsA("Accessory") then  
        table.insert(allhats,v)  
    end  
end  
  
local locks = {}  
for i,v in pairs(allhats) do  
    table.insert(locks,v.Changed:Connect(function(p)  
        if p == "BackendAccoutrementState" then  
            updatestate(v,0)  
        end  
    end))  
    updatestate(v,2)  
end  
  
workspace.FallenPartsDestroyHeight = 0/0  
  
local function play(id,speed,prio,weight)  
    local Anim = Instance.new("Animation")  
    Anim.AnimationId = "https"..tostring(math.random(1000000,9999999)).."="..tostring(id)  
    local track = character.Humanoid:LoadAnimation(Anim)  
    track.Priority = prio  
    track:Play()  
    track:AdjustSpeed(speed)  
    track:AdjustWeight(weight)  
    return track  
end  
  
local r6fall = 180436148  
local r15fall = 507767968  
  
local dropcf = CFrame.new(character.HumanoidRootPart.Position.x,fph-.25,character.HumanoidRootPart.Position.z)  
if character.Humanoid.RigType == Enum.HumanoidRigType.R15 then  
    dropcf =  dropcf * CFrame.Angles(math.rad(20),0,0)  
    character.Humanoid:ChangeState(16)  
    play(r15fall,1,5,1).TimePosition = .1  
else  
    play(r6fall,1,5,1).TimePosition = .1  
end  
  
spawn(function()  
    while hrp.Parent ~= nil do  
        hrp.CFrame = dropcf  
        hrp.Velocity = Vector3.new(0,0,0)  
        hrp.RotVelocity = Vector3.new(0,0,0)  
        game:GetService("RunService").Heartbeat:wait()  
    end  
end)  
  
task.wait(.506)  

-- **Quebra todas as juntas do personagem**
for _, v in pairs(character:GetDescendants()) do  
    if v:IsA("Motor6D") then  
        v:Destroy()  
    end  
end  
  
torso.AncestryChanged:wait()  
  
for i,v in pairs(locks) do  
    v:Disconnect()  
end  
for i,v in pairs(allhats) do  
    updatestate(v,4)  
end  

-- **Teleporta os acessórios para longe do void imediatamente**
local safePosition = Vector3.new(0, 1000, 0)  
for _, hat in pairs(allhats) do  
    if hat:FindFirstChild("Handle") then  
        hat.Handle.Position = safePosition  
    end  
end  
  
spawn(function()  
    plr.CharacterAdded:wait():WaitForChild("HumanoidRootPart",10).CFrame = start  
    workspace.FallenPartsDestroyHeight = fph  
end)  
  
local dropped = false  
repeat  
    local foundhandle = false  
    for i,v in pairs(allhats) do  
        if v:FindFirstChild("Handle") then  
            foundhandle = true  
            if v.Handle.CanCollide then  
                dropped = true  
                break  
            end  
        end  
    end  
    if not foundhandle then  
        break  
    end  
    task.wait()  
until plr.Character ~= character or dropped  
  
if dropped then  
    print("dropped")  
    workspace.CurrentCamera.CameraSubject = campart  
    for i,v in pairs(character:GetChildren()) do  
        if v:IsA("Accessory") and v:FindFirstChild("Handle") and v.Handle.CanCollide then  
            spawn(function()  
                for i = 1,10 do  
                    v.Handle.CFrame = start  
                    v.Handle.Velocity = Vector3.new(0,50,0)  
                    task.wait()  
                end  
            end)  
        end  
    end  
else  
    print("failed to drop")  
end

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local localPlayer = Players.LocalPlayer
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local flingForce = Vector3.new(0, 35, 0)

-- Lista para armazenar acessórios do jogador local
local playerHats = {}

-- Atualiza a lista de acessórios do jogador local
local function updatePlayerHats()
    table.clear(playerHats)

    for _, accessory in ipairs(character:GetChildren()) do  
        if accessory:IsA("Accessory") and accessory:FindFirstChild("Handle") then  
            playerHats[accessory.Handle] = accessory.Handle  
        end  
    end
end

-- Aplica os flings nos acessórios do jogador local
local function applyFling()
    for handle in pairs(playerHats) do
        if handle and handle.Parent then
            handle.CanCollide = false

            -- Fling 1 e 2 mantidos  
            handle.Velocity = Vector3.new(0, 35, 0) + flingForce  

            -- Otimização do BodyVelocity  
            local bodyVelocity = handle:FindFirstChild("BodyVelocity")  
            if not bodyVelocity then  
                bodyVelocity = Instance.new("BodyVelocity")  
                bodyVelocity.MaxForce = Vector3.new(8000, 8000, 8000)  
                bodyVelocity.Velocity = flingForce  
                bodyVelocity.Parent = handle  

                -- Conexão única para remover o BodyVelocity caso o hat seja deletado  
                handle.Destroying:Once(function()  
                    bodyVelocity:Destroy()  
                end)  
            else  
                bodyVelocity.Velocity = flingForce  
            end  
        end  
    end
end

-- Atualiza a lista de hats do jogador local a cada 2 segundos
task.spawn(function()
    while true do
        updatePlayerHats()
        task.wait(2)
    end
end)

-- Aplica o fling a cada frame
RunService.Heartbeat:Connect(applyFling)
-- Variáveis principais  
local Players = game:GetService("Players")  
local RunService = game:GetService("RunService")  
local Workspace = game:GetService("Workspace")  

local localPlayer = Players.LocalPlayer  
local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local rootPart = char:WaitForChild("HumanoidRootPart")

-- Tabela para armazenar partes não ancoradas válidas
local unanchoredParts = {}

-- Função para atualizar a lista de partes não ancoradas válidas
local function updateUnanchoredParts()  
    table.clear(unanchoredParts)
    for _, part in ipairs(Workspace:GetDescendants()) do  
        if part:IsA("BasePart") and not part.Anchored then  
            local parent = part.Parent
            
            -- Ignora partes que pertencem a personagens de jogadores
            if not (parent and Players:GetPlayerFromCharacter(parent)) then
                -- Ignora partes que contêm um Motor6D
                if not part:FindFirstChildWhichIsA("Motor6D", true) then  
                    unanchoredParts[part] = part -- Usa um dicionário para otimização
                    part.Destroying:Connect(function()
                        unanchoredParts[part] = nil -- Remove a parte da lista ao ser deletada
                    end)
                end
            end
        end  
    end  
end  

-- Função para aplicar SimulationRadius  
local function enforceSimulationRadius()  
    for _, player in ipairs(Players:GetPlayers()) do  
        if player == localPlayer then  
            sethiddenproperty(player, "SimulationRadius", 1000)  
        else  
            sethiddenproperty(player, "SimulationRadius", 0)  
        end  
    end  
end  

-- Função para desativar NetworkSleeping  
local function disableNetworkSleeping()  
    for part, _ in pairs(unanchoredParts) do  
        if part and part.Parent then  
            -- Desativa NetworkSleeping apenas para partes próximas (raio de 1000 unidades)  
            if (part.Position - rootPart.Position).Magnitude then  
                sethiddenproperty(part, "NetworkIsSleeping", false)  
            end  
        else
            unanchoredParts[part] = nil -- Remove a parte caso tenha sido deletada
        end
    end  
end  

-- Atualiza a lista de partes não ancoradas a cada 3 segundos  
task.spawn(function()  
    while true do  
        updateUnanchoredParts()  
        task.wait(3)  
    end  
end)  

-- Aplica as configurações constantemente  
RunService.Heartbeat:Connect(function()  
    enforceSimulationRadius()  
    disableNetworkSleeping()  
end)

task.wait(1)
