function thisCode()
    repeat task.wait() until game:IsLoaded()
    game:GetService('ReplicatedStorage').Remotes.CommF_:InvokeServer("SetTeam", "Pirates")
    local PlaceID = game.PlaceId
    local AllIDs = {}
    local foundAnything = ""
    local actualHour = os.date("!*t").hour
    local Deleted = false
    local File = pcall(function()
        AllIDs = game:GetService('HttpService'):JSONDecode(readfile("NotSameServers.json"))
    end)
    if not File then
        table.insert(AllIDs, actualHour)
        writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
    end

    local function TPReturner()
        local Site
        if foundAnything == "" then
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
        else
            Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
        end
        local ID = ""
        if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
            foundAnything = Site.nextPageCursor
        end
        local num = 0
        for i, v in pairs(Site.data) do
            local Possible = true
            ID = tostring(v.id)
            if tonumber(v.maxPlayers) > tonumber(v.playing) then
                for _, Existing in pairs(AllIDs) do
                    if num ~= 0 then
                        if ID == tostring(Existing) then
                            Possible = false
                        end
                    else
                        if tonumber(actualHour) ~= tonumber(Existing) then
                            local delFile = pcall(function()
                                delfile("NotSameServers.json")
                                AllIDs = {}
                                table.insert(AllIDs, actualHour)
                            end)
                        end
                    end
                    num = num + 1
                end
                if Possible == true then
                    table.insert(AllIDs, ID)
                    wait()
                    pcall(function()
                        writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
                        wait()
                        game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
                    end)
                    wait(4)
                end
            end
        end
    end

    local function Teleport()
        while wait() do
            pcall(function()
                TPReturner()
                if foundAnything ~= "" then
                    TPReturner()
                end
            end)
        end
    end

    local veryImportantWaitTime = 0.5

    task.spawn(function()
        while task.wait(veryImportantWaitTime) do
            pcall(function()
                for i, v in pairs(game.CoreGui:GetDescendants()) do
                    pcall(function()
                        if string.find(v.Name, "ErrorMessage") then
                            if string.find(v.Text, "Security kick") then
                                veryImportantWaitTime = 1e9
                                Teleport()
                            end
                        end
                    end)
                end
            end)
        end
    end)

    game:GetService("TeleportService").OnTeleport:Connect(function()
        
        Teleport()
    end)

    while true do
        local hasChar = game.Players.LocalPlayer:FindFirstChild("Character")

        if not game.Players.LocalPlayer.Character then
        else
            local hasCrewTag = game.Players.LocalPlayer.Character:FindFirstChild("CrewBBG", true)
            if hasCrewTag then
                hasCrewTag:Destroy()
            end
            local hasHumanoid = game.Players.LocalPlayer.Character:FindFirstChild("Humanoid")
            if hasHumanoid then
                local Chest = game.Workspace:FindFirstChild("Chest4") or game.Workspace:FindFirstChild("Chest3") or game.Workspace:FindFirstChild("Chest2") or game.Workspace:FindFirstChild("Chest1") or game.Workspace:FindFirstChild("Chest")

                if Chest then
                    game.Players.LocalPlayer.Character:PivotTo(Chest:GetPivot())
                    firesignal(Chest.Touched, game.Players.LocalPlayer.Character.HumanoidRootPart)
                else
                    Teleport()
                    break
                end
            end
        end
        task.wait()
    end
end

thisCode()
