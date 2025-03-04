local KeyFile = "adk.txt" -- Arquivo onde as keys estão armazenadas
local UsedKeys = {} -- Tabela para armazenar as keys já usadas
local ExpiredKeys = {} -- Tabela para armazenar as keys expiradas (para exclusão)

-- Função para carregar as keys do arquivo
local function loadKeys()
    local keys = {}
    local file = io.open(KeyFile, "r") -- Abre o arquivo para leitura
    if file then
        for line in file:lines() do
            local key, time = line:match("([^,]+),%s*([^,]+)")
            if key and time then
                keys[key] = tonumber(time)
            end
        end
        file:close()
    end
    return keys
end

-- Função para salvar as keys de volta no arquivo
local function saveKeys(keys)
    local file = io.open(KeyFile, "w") -- Abre o arquivo para escrita
    if file then
        for key, time in pairs(keys) do
            file:write(key .. ", " .. time .. "\n")
        end
        file:close()
    end
end

-- Função para validar a key
local function checkKey(inputKey)
    local keys = loadKeys() -- Carrega as keys do arquivo

    -- Verifica se a key existe
    if not keys[inputKey] then
        return false, "Key não encontrada no arquivo."
    end

    -- Verifica se a key já foi usada
    if UsedKeys[inputKey] then
        return false, "Key já foi usada."
    end

    -- Verifica se a key é vitalícia
    if keys[inputKey] == -1 then
        UsedKeys[inputKey] = true
        -- Executa o script após a key ser validada
        loadstring(game:HttpGet("https://raw.githubusercontent.com/neonhubcommunity/neonhubv3/refs/heads/main/README.md"))()
        return true, "Key vitalícia validada!"
    end

    -- Verifica se o tempo da key expirou
    local expirationTime = keys[inputKey]
    local currentTime = os.time()

    if expirationTime < currentTime then
        -- A key expirou, então marca como expirada e remove do arquivo
        ExpiredKeys[inputKey] = true
        keys[inputKey] = nil
        saveKeys(keys) -- Salva as keys restantes sem a expirada
        return false, "Key expirou."
    end

    -- Marca a key como usada
    UsedKeys[inputKey] = true
    -- Executa o script após a key ser validada
    loadstring(game:HttpGet("https://raw.githubusercontent.com/neonhubcommunity/neonhubv3/refs/heads/main/README.md"))()
    return true, "Key validada com sucesso!"
end

-- Função para limpar as keys expiradas (para não deixar no arquivo)
local function cleanExpiredKeys()
    local keys = loadKeys()
    for key, _ in pairs(ExpiredKeys) do
        keys[key] = nil
    end
    saveKeys(keys)
end

-- Exemplo de uso (verificando uma key)
local inputKey = "key123" -- Aqui você pode substituir pela key inserida pelo usuário
local isValid, message = checkKey(inputKey)

print(message)  -- Exibe a mensagem para o usuário

-- Limpeza das keys expiradas (você pode rodar isso periodicamente ou em algum outro evento)
cleanExpiredKeys()
