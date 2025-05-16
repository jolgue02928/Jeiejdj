local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")

local player = Players.LocalPlayer
local PLACE_ID = 116495829188952 -- Dead Rails Alpha
local JOB_ID = game.JobId

-- Função para procurar um servidor diferente
local function findNewServer()
    local cursor = nil

    for _ = 1, 5 do -- tenta até 5 páginas
        local url = "https://games.roblox.com/v1/games/"..PLACE_ID.."/servers/Public?sortOrder=Asc&limit=100"
        if cursor then
            url = url .. "&cursor=" .. cursor
        end

        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)

        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.playing < server.maxPlayers and server.id ~= JOB_ID then
                    return server.id
                end
            end
            cursor = result.nextPageCursor
            if not cursor then break end
        else
            warn("Falha ao buscar servidores.")
            break
        end
    end

    return nil
end

-- Loop infinito com espera de 5 minutos
while true do
    task.wait(300) -- 5 minutos (300 segundos)

    local serverId = findNewServer()
    if serverId then
        TeleportService:TeleportToPlaceInstance(PLACE_ID, serverId, player)
    else
        warn("Não encontrou outro servidor disponível.")
    end
end
