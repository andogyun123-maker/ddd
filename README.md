-- 로블록스 비행 스크립트 (LocalScript - StarterCharacterScripts에 배치)
-- 플레이어가 F키를 눌러 비행 활성화
-- WASD로 방향 조절

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local character = script.Parent
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

-- 비행 설정
local isFlying = false
local flySpeed = 50 -- 비행 속도 (조절 가능)
local acceleration = 1.5
local currentSpeed = 0

-- 입력 방향
local moveDirection = Vector3.new(0, 0, 0)

-- BodyVelocity 생성
local bodyVelocity
local originalGravity

-- 비행 시작 함수
local function startFlying()
    if isFlying then return end
    isFlying = true
    
    -- 중력 제거
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new(0, 0, 0)
    bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bodyVelocity.Parent = humanoidRootPart
    
    humanoid.PlatformStand = true
    print("비행 시작! WASD로 조종하세요")
end

-- 비행 종료 함수
local function stopFlying()
    if not isFlying then return end
    isFlying = false
    
    if bodyVelocity then
        bodyVelocity:Destroy()
        bodyVelocity = nil
    end
    
    humanoid.PlatformStand = false
    moveDirection = Vector3.new(0, 0, 0)
    currentSpeed = 0
    print("비행 종료!")
end

-- 키 입력 감지
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- F키: 비행 토글
    if input.KeyCode == Enum.KeyCode.F then
        if isFlying then
            stopFlying()
        else
            startFlying()
        end
    end
    
    -- WASD 입력
    if isFlying then
        if input.KeyCode == Enum.KeyCode.W then
            moveDirection = moveDirection + humanoidRootPart.CFrame.LookVector
        elseif input.KeyCode == Enum.KeyCode.S then
            moveDirection = moveDirection - humanoidRootPart.CFrame.LookVector
        elseif input.KeyCode == Enum.KeyCode.A then
            moveDirection = moveDirection - humanoidRootPart.CFrame.RightVector
        elseif input.KeyCode == Enum.KeyCode.D then
            moveDirection = moveDirection + humanoidRootPart.CFrame.RightVector
        end
    end
end)

-- 키 떼기 감지
UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if not isFlying then return end
    
    -- WASD 키 떼기 시 감속
    if input.KeyCode == Enum.KeyCode.W or 
       input.KeyCode == Enum.KeyCode.S or 
       input.KeyCode == Enum.KeyCode.A or 
       input.KeyCode == Enum.KeyCode.D then
        moveDirection = moveDirection * 0.8 -- 감속
    end
end)

-- 비행 루프
RunService.RenderStepped:Connect(function()
    if isFlying and bodyVelocity then
        -- 속도 계산
        if moveDirection.Magnitude > 0 then
            currentSpeed = math.min(currentSpeed + acceleration, flySpeed)
            moveDirection = moveDirection.Unit * currentSpeed
        else
            currentSpeed = math.max(currentSpeed - acceleration, 0)
            moveDirection = moveDirection * 0.95
        end
        
        -- 속도 적용
        bodyVelocity.Velocity = moveDirection
        
        -- 캐릭터 방향 업데이트 (마우스 방향 보기)
        local mouse = player:GetMouse()
        local targetPosition = mouse.Hit.Position
        humanoidRootPart.CFrame = CFrame.lookAt(humanoidRootPart.Position, targetPosition)
    end
end)

-- 스크립트 정리 (캐릭터 제거 시)
humanoid.Died:Connect(function()
    if isFlying then
        stopFlying()
    end
end)
