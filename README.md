--[[
    guguhub - Versão Otimizada (menos travamento)
    - ESP atualizado a cada 3 frames
    - Spam com delay de 1 segundo
    - Limitador de distância no ESP (1000 studs)
    - Menos desenhos (apenas caixa por padrão)
    - Opção de reduzir qualidade
]]

-- Configurações de otimização
local OTIMIZACAO = {
    FRAME_SKIP = 3,           -- Atualiza ESP a cada 3 frames
    MAX_DISTANCIA = 1000,      -- Não desenha além dessa distância
    SPAM_INTERVALO = 1,        -- Segundos entre mensagens de spam
    USAR_NOMES = false,        -- Desativar nomes para melhor performance
    USAR_DISTANCIA = false,    -- Desativar distância
    USAR_LINHA = false         -- Desativar linha de visão
}

-- (cole o resto do script aqui, mas com as modificações abaixo)

-- Variável para controle de frames
local frameCount = 0

-- No loop RenderStepped:
runService.RenderStepped:Connect(function()
    frameCount = frameCount + 1
    if frameCount % OTIMIZACAO.FRAME_SKIP == 0 then
        -- Atualiza ESP apenas a cada FRAME_SKIP frames
        if config.esp.ativo then
            desenharESPJogadoresOtimizado()
        end
    end
    -- Outras funções que precisam de alta taxa (aimbot, fly) continuam normais
    aimbotFunction()
    flyFunction()
    noclipFunction()
    nightVisionFunction()
    wallhackFunction()
end)

-- Função ESP otimizada com limite de distância
local function desenharESPJogadoresOtimizado()
    for _, jogador in pairs(players:GetPlayers()) do
        if jogador ~= player and jogador.Character and jogador.Character:FindFirstChild("HumanoidRootPart") then
            local root = jogador.Character.HumanoidRootPart
            local distancia = (camera.CFrame.Position - root.Position).Magnitude
            if distancia < OTIMIZACAO.MAX_DISTANCIA then
                local pos, visivel = camera:WorldToViewportPoint(root.Position)
                if visivel then
                    local cor = getPapelCor(jogador)
                    local scale = 200 / distancia
                    local size = Vector2.new(scale * 2, scale * 3)

                    -- Apenas caixa (mais leve)
                    if config.esp.ativo and config.esp.caixa then
                        if not espDrawings[jogador] then espDrawings[jogador] = {} end
                        if not espDrawings[jogador].caixa then
                            local d = Drawing.new("Square")
                            d.Thickness = 2
                            d.Filled = false
                            espDrawings[jogador].caixa = d
                        end
                        local d = espDrawings[jogador].caixa
                        d.Color = cor
                        d.Position = Vector2.new(pos.X - size.X/2, pos.Y - size.Y/2)
                        d.Size = size
                        d.Visible = true
                    end
                    -- Outros elementos só se ativados
                end
            end
        end
    end
end

-- Função de spam com timer
local ultimoSpam = 0
local function spamFunction()
    if not config.troll.spamAtivo then return end
    local agora = tick()
    if agora - ultimoSpam > OTIMIZACAO.SPAM_INTERVALO then
        ultimoSpam = agora
        local mensagem = config.troll.spamMensagem
        game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("SayMessageRequest"):FireServer(mensagem, "All")
    end
end

-- No Heartbeat, chame spamFunction
