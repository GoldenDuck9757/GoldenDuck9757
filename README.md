# GoldenChalk9757's GitHub

## Sobre Mim

Olá, sou **GoldenChalk9757**. Desenvolvedor com experiência em **Python**, **Java**, **HTML** e crio e desenvolvo **Inteligências Artificiais**. Meus pronomes são **ele/dele**. Gosto de programar coisas bem malucas e ate mesmo sem nexo, ou por pura curiosidade ou por diversão...

[![GoldenChalk9757 Little secret ](https://img.shields.io/badge/Botão%20Secreto-Click%20Aqui!-blue?style=for-the-badge)](https://youareanidiot.cc/)

## Tecnologias e Especialidades

- **Python**: Criação de scripts, automação de tarefas e desenvolvimento de IAs.
- **Java**: Desenvolvimento de aplicações robustas e escaláveis.
- **HTML**: Construção de interfaces web intuitivas e acessíveis.
- **IA**: Implementação de modelos de aprendizado de máquina e redes neurais com o uso da API do Google generative AI (IA studio)

## Projetos em Destaque

### 1. **Tamagotchi com Pygame**
- **Descrição**: Um jogo inspirado no Tamagotchi, desenvolvido com Pygame, onde você cuida de uma criatura virtual. Este projeto explora diferentes mecânicas de interação e gerenciamento de estados, interação com o uso de IA.
- **Tecnologias**: Python, Pygame, Google Generative AI

### 2. **Bot de Discord com Memória Avançada**
- **Descrição**: um bot de Discord capaz de manter o contexto de conversas por longos períodos, respondendo de forma inteligente e natural. Ele utiliza o Google Generative AI para garantir respostas precisas e adaptadas ao contexto.
- **Tecnologias**: Python, Google Generative AI

### 3. **Interface de Conversa com IA**
- **Descrição**: Desenvolvi uma interface em Python utilizando PyQt5, que permite ao usuário interagir de maneira natural com uma IA. A interface pode executar comandos no terminal, manipular arquivos, e ainda salva o histórico das conversas para consultas futuras.
- **Tecnologias**: Python, PyQt5

## Contato

Se quiser trocar ideias, discutir projetos ou colaborar, fique à vontade para me contatar:

- **GitHub**: [GoldenChalk9757](https://github.com/GoldenChalk9757)
- **Discord**: golden4484




## Código do Último projeto de IA

```import os
import discord
import google.generativeai as genai
import json
import re
import pytz  
from discord.ext import commands
from datetime import datetime

# Configura o bot do Discord com Intents
intents = discord.Intents.default()
intents.members = True
intents.messages = True
intents.message_content = True
bot = commands.Bot(command_prefix=commands.when_mentioned, intents=intents)

# Configuração da IA da Google
genai.configure(api_key=os.environ["GEMINI_API_KEY"])

# Configuração do modelo da IA
generation_config = {
    "temperature": 0.7,
    "top_p": 0.95,
    "top_k": 64,
    "max_output_tokens": 100,  # Limita o tamanho da resposta
    "stop_sequences": ["100", "200", "300"],
    "response_mime_type": "text/plain",
}

# Função para carregar o histórico de conversa (memória)
def load_memory(file_path):
    if os.path.exists(file_path):
        with open(file_path, 'r', encoding='utf-8') as file:
            raw_memory = json.load(file)
            return [{"text": item["message"]} for item in raw_memory]
    return []

# Função para salvar o histórico de conversa (memória)
def save_memory(file_path, memory):
    with open(file_path, 'w', encoding='utf-8') as file:
        json.dump(memory, file, ensure_ascii=False, indent=4)

# Definindo o arquivo de memória
memory_file = 'chat_memory.json'
chat_history = load_memory(memory_file)

# Cria a sessão do modelo, agora passando o histórico no formato correto
model = genai.GenerativeModel(
    model_name="gemini-1.5-flash",
    generation_config=generation_config,
    system_instruction=(
        "Você se chama Astelown, seu modo de falar é direto, não utiliza emojis, e utiliza '...' "
        "não usa ponto final, e normalmente fala de modo mais direto e simples. "
        "Você interage com membros do servidor Discord quando necessário. "
        "Não repita o que o usuário diz e só responda diretamente ou aja quando solicitado. "
        "Você vai às vezes soltar comentários engraçados, mas de forma breve. "
        "Você não é uma IA, e quando te chamam assim você fica meio brava. "
        "Você tem 22 anos e é uma garota."
    ),
)

chat_session = model.start_chat(history=chat_history)

# Função para obter a data e a hora atuais em formato legível
def get_current_datetime():
    now = datetime.now()
    formatted_date = now.strftime("%A, %d de %B de %Y")  # Exemplo: Segunda-feira, 07 de Outubro de 2024
    formatted_time = now.strftime("%H:%M:%S")  # Exemplo: 14:35:22
    return formatted_date, formatted_time

# Função para obter a hora em diferentes países e cidades
def get_time_in_timezone(timezone_name):
    try:
        tz = pytz.timezone(timezone_name)
        now = datetime.now(tz)
        return now.strftime("%A, %d de %B de %Y - %H:%M:%S")
    except pytz.UnknownTimeZoneError:
        return "Fuso horário desconhecido"

# Função principal de interação com a IA
async def interact_with_ai(message):
    user_name = message.author.name
    user_id = message.author.id
    server_name = message.guild.name

    # Informações detalhadas do membro e do servidor
    members_info = [f"{member.name} (ID: <@{member.id}>)" for member in message.guild.members]
    channels_info = [f"{channel.name} (ID: <#{channel.id}>, Tipo: {'Texto' if isinstance(channel, discord.TextChannel) else 'Voz'})" for channel in message.guild.channels]
    
    # Obtém a data e a hora atuais
    current_date, current_time = get_current_datetime()

    prompt = (f"Servidor: {server_name}\n"
              f"Quem está falando: {user_name} (ID: <@{user_id}>)\n"
              f"Membros: {', '.join(members_info)}\n"
              f"Canais: {', '.join(channels_info)}\n"
              f"Data: {current_date}\n"
              f"Hora: {current_time}\n"
              f"Mensagem: {message.content}")

    response = chat_session.send_message(prompt)
    
    chat_history.append({"user": user_name, "user_id": user_id, "message": message.content, "response": response.text})
    save_memory(memory_file, chat_history)

    return response.text

# Função para verificar se a mensagem menciona um canal e enviar resposta para ele
async def check_and_send_to_channel(message):
    # Modificação para verificar até 8 canais
    matches = re.findall(r'<#(\d+)>', message.content)
    if matches:
        channels_to_send = matches[:8]  # Limita a 8 canais
        response_text = await interact_with_ai(message)

        # Separa a mensagem de forma que cada canal receba uma mensagem natural, sem repetição feia
        for idx, canal_id in enumerate(channels_to_send):
            canal_destino = message.guild.get_channel(int(canal_id))
            
            if canal_destino and isinstance(canal_destino, discord.TextChannel):
                # Divide o texto de resposta para cada canal de maneira simples e natural
                individual_response = f"{response_text}" if idx == 0 else f"{response_text.split()[0]}..."
                await canal_destino.send(individual_response)  # Envia a mensagem diretamente para o canal
        
        return True
    return False

# Função para enviar mensagem direta (DM)
async def send_dm(user_id, message_content):
    user = await bot.fetch_user(user_id)
    if user:
        await user.send(message_content)

# Função para verificar se a mensagem menciona um usuário e enviar DM
async def check_and_send_dm(message):
    # Envia DM sempre que a mensagem é sobre DM
    if "dm" in message.content.lower() or "direct" in message.content.lower():
        response_text = await interact_with_ai(message)
        await send_dm(message.author.id, response_text)  # Envia a mensagem diretamente para o DM
        return True
    return False

# Evento para quando o bot for mencionado ou quando uma mensagem for enviada
@bot.event
async def on_message(message):
    if message.author == bot.user:
        return

    # Verifica se o bot foi mencionado
    if bot.user in message.mentions:
        response_text = await interact_with_ai(message)
        await message.reply(response_text)

    # Verifica se alguém mencionou um canal
    if await check_and_send_to_channel(message):
        return  # Não envia mensagem de confirmação

    # Verifica se alguém mencionou a DM ou algo relacionado
    if await check_and_send_dm(message):
        return  # Não envia mensagem de confirmação

    # Se houver uma imagem, a IA pode ver a imagem
    if message.attachments:
        image_urls = [attachment.url for attachment in message.attachments if attachment.url.lower().endswith(('png', 'jpg', 'jpeg', 'gif'))]
        if image_urls:
            response_text = await interact_with_ai(message)
            await message.channel.send(f"Recebi sua imagem! {image_urls}")

    await bot.process_commands(message)

# Comando para listar canais disponíveis
@bot.command(name="canais")
async def listar_canais(ctx):
    canais_texto = [f"{c.name} (ID: <#{c.id}>)" for c in ctx.guild.channels if isinstance(c, discord.TextChannel)]
    canais_voz = [f"{c.name} (ID: <#{c.id}>)" for c in ctx.guild.channels if isinstance(c, discord.VoiceChannel)]
    
    texto = (
        "Canais disponíveis:\n"
        "Canais de Texto:\n" + "\n".join(canais_texto) + "\n\n"
        "Canais de Voz:\n" + "\n".join(canais_voz)
    )
    await ctx.send(texto)

# Comando para mostrar o horário em um país ou cidade
@bot.command(name="horario")
async def mostrar_horario(ctx, *, location):
    time_in_location = get_time_in_timezone(location)
    await ctx.send(f"Horário atual em {location}: {time_in_location}")

# Inicia o bot com o token do Discord
bot.run(os.environ["DISCORD_TOKEN"])```
