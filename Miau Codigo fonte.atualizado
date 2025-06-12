import random
import pygame
import os
import threading

# Semáforo para proteger o estado do jogo
estado_jogo_lock = threading.Lock()

escolha_jogador1_event = threading.Event()
escolha_jogador2_event = threading.Event()

popup_carta = None  # Armazena a carta que deve exibir o pop-up
popup_resultado = None  # Armazena o resultado do pop-up (True ou False)
popup_carta_jogador = None

# --- 1. Inicialização e Configuração Básica do Pygame ---
pygame.init()

# --- Configuração da tela ---
info = pygame.display.Info()
LARGURA, ALTURA = info.current_w, info.current_h
TELA = pygame.display.set_mode((LARGURA, ALTURA), pygame.RESIZABLE)
pygame.display.set_caption("MIAUPOWWWWW")

# --- 2. Carregamento de Recursos (Imagens, Fontes) ---
base_path = os.path.dirname(os.path.abspath(__file__))
images_path = os.path.join(base_path, "images")

# Dicionário com nomes dos arquivos de imagem
arquivos_imagens_cartas = {
    "pedra_branco": "pedra.png",
    "papel_branco": "papel.png",
    "tesoura_branco": "tesoura.png",
    "pedra_cinza": "pedracinza.png",
    "papel_cinza": "papelcinza.png",
    "tesoura_cinza": "tesouracinza.png",
    "jogada_hacker": "jogada_hacker.png",
    "oitava_vida": "oitava_vida.png",
    "miaudicao": "miaudicao.png",
    "garra_feroz": "garra_feroz.png",
    "roubo_felino": "roubo_felino.png",
    "arranhao_sorte": "arranhao_sorte.png"
}

IMAGENS = {}
try:
    for chave, nome_arquivo in arquivos_imagens_cartas.items():
        caminho_imagem = os.path.join(images_path, nome_arquivo)
        if not os.path.exists(caminho_imagem):
            raise FileNotFoundError(f"Arquivo de imagem não encontrado: {caminho_imagem}")
        IMAGENS[chave] = pygame.image.load(caminho_imagem).convert_alpha()  # .convert_alpha() para melhor desempenho
except FileNotFoundError as e:
    print(f"Erro: {e}")
    pygame.quit()
    exit()
except pygame.error as e:
    print(f"Erro ao carregar imagens: {e}")
    pygame.quit()
    exit()

# Fontes
fonte = pygame.font.SysFont(None, 36)
fonte_pequena = pygame.font.SysFont(None, 24)
fonte_grande = pygame.font.SysFont(None, 48)  # Nova fonte para títulos maiores

# --- 3. Variáveis de Estado do Jogo (Globals, para simplificar por enquanto) ---
# Em um jogo maior, você poderia usar classes ou objetos para gerenciar o estado
opcoes = ["pedra", "papel", "tesoura"]

# Atualize o mapeamento de teclas para incluir as cartas
teclas_jogador1 = {
    pygame.K_q: 0,  # primeira carta sorteada
    pygame.K_w: 1,  # segunda carta sorteada
    pygame.K_e: 2,  # terceira carta sorteada
    pygame.K_s: "pedra_branco",
    pygame.K_d: "papel_branco",
    pygame.K_a: "tesoura_branco"
}

teclas_jogador2 = {
    pygame.K_u: 0,
    pygame.K_i: 1,
    pygame.K_o: 2,
    pygame.K_k: "pedra_cinza",
    pygame.K_l: "papel_cinza",
    pygame.K_j: "tesoura_cinza"
}

# Atualização do mapeamento de cartas amigáveis
NOME_CARTAS_AMIGAVEL = {
    "jogada_hacker": "Jogada do Hacker",
    "oitava_vida": "A Oitava Vida",
    "miaudicao": "Miaudição",  # Corrigido e adicionado
    "garra_feroz": "Garra Feroz",  # Adicionado
    "roubo_felino": "Roubo Felino",  # Adicionado
    "arranhao_sorte": "Arranhão da Sorte"
}

# Probabilidade da CPU usar uma carta especial em cada rodada (exemplo: 30%)
CHANCE_CPU_USAR_CARTA = 0.3

# Atualização das propriedades das cartas
CARTAS_PROPRIEDADES = {
    "jogada_hacker": {"duração": 1},
    "oitava_vida": {"efeito": "recupera_vida"},
    "miaudicao": {"duração": 2},  # Adicionado
    "garra_feroz": {"duração": 1},  # Adicionado
    "roubo_felino": {"duração": 0},  # Adicionado
    "arranhao_sorte": {"duração": 2}
}

cartas_disponiveis_para_sorteio = list(CARTAS_PROPRIEDADES.keys())  # Lista de todas as cartas

def sortear_cartas_unicas_para_jogador(cartas_possiveis, quantidade):
    cartas = []
    while len(cartas) < quantidade:
        carta = random.choice(cartas_possiveis)
        if carta not in cartas:
            cartas.append(carta)
    return cartas

# Estado global do jogo
estado_jogo = {
    "vidas": {"jogador1": 7, "jogador2": 7},
    "rodada": 1,
    "MAX_PARTIDAS": 13,
    "modo": None,  # 'menu', 'cpu', 'local', 'resultado_rodada', 'fim_jogo'
    "jogada_jogador1": None,  # Mão jogada pelo jogador 1
    "jogada_jogador2": None,  # Mão jogada pelo jogador 2
    "carta_ativada_jogador1": None,  # Carta especial ativada pelo jogador 1
    "carta_ativada_jogador2": None,  # Carta especial ativada pelo jogador 2
    "mostrar_jogadas": False,
    "mostrar_jogadas_tempo": 0,
    "mensagem_tela": "Escolha o modo: 1 - Jogar contra CPU | 2 - Jogar Local",
    "vencedor_final": None,
    "cartas_jogador1": [],
    "cartas_jogador2": [],
    "efeitos_ativos": {"jogador1": {}, "jogador2": {}},  # {carta: {duração: X, ...}}
    "mao_invencivel_j1": None,  # Para Arranhão da Sorte
    "mao_invencivel_j2": None,  # Para Arranhão da Sorte
    "carta_especial_usada_jogador1": False,
    "carta_especial_usada_jogador2": False,
    "vidas_perdidas_acumuladas_jogador1": 0,
    "vidas_perdidas_acumuladas_jogador2": 0,
    "empates_seguidos": 0,  # Novo campo para contar empates seguidos
}

# No início do jogo
estado_jogo["cartas_jogador1"] = sortear_cartas_unicas_para_jogador(cartas_disponiveis_para_sorteio, 3)
estado_jogo["cartas_jogador2"] = sortear_cartas_unicas_para_jogador(cartas_disponiveis_para_sorteio, 3)

# --- 4. Funções de Desenho (Renderização) ---

def desenhar_menu(estado):
    TELA.fill((255, 255, 255))
    titulo = fonte_grande.render("MIAUPOWWW", True, (0, 0, 0))
    instrucao = fonte.render(estado["mensagem_tela"], True, (0, 0, 0))
    TELA.blit(titulo, (LARGURA // 2 - titulo.get_width() // 2, ALTURA // 3))
    TELA.blit(instrucao, (LARGURA // 2 - instrucao.get_width() // 2, ALTURA // 3 + 70))
    pygame.display.update()

def desenhar_hud(estado):
    """Desenha 'Partida' centralizado no topo e o número da rodada logo abaixo."""
    try:
        fonte_londrina = pygame.font.Font(os.path.join(base_path, "LondrinaSolid-Regular.ttf"), 90)  # Fonte para "Partida"
    except:
        fonte_londrina = pygame.font.SysFont(None, 90)
    try:
        fonte_londrina_num = pygame.font.Font(os.path.join(base_path, "LondrinaSolid-Regular.ttf"), 96)
    except:
        fonte_londrina_num = pygame.font.SysFont(None, 96)

    texto_partida = fonte_londrina.render("Partida", True, (0, 0, 0))
    texto_num = fonte_londrina_num.render(str(estado["rodada"]), True, (0, 0, 0))

    margem_topo = 30  # Afasta da margem superior

    # Desenha "Partida" centralizado e afastado do topo
    TELA.blit(texto_partida, (LARGURA // 2 - texto_partida.get_width() // 2, margem_topo))

    # Desenha contorno preto para o número
    y_num = margem_topo + texto_partida.get_height() - 5  # Ajuste fino para aproximar
    for dx, dy in [(-2, 0), (2, 0), (0, -2), (0, 2)]:
        contorno = fonte_londrina_num.render(str(estado["rodada"]), True, (0, 0, 0))
        TELA.blit(contorno, (LARGURA // 2 - texto_num.get_width() // 2 + dx, y_num + dy))
    # Desenha o número da rodada logo abaixo de "Partida"
    TELA.blit(texto_num, (LARGURA // 2 - texto_num.get_width() // 2, y_num))

    # --- Adiciona a imagem VS centralizada no meio inferior ---
    vs_img_path = os.path.join(images_path, "vs.png")
    if os.path.exists(vs_img_path):
        vs_img = pygame.image.load(vs_img_path).convert_alpha()
        largura_vs, altura_vs = 80, 47  # Tamanho da imagem VS
        vs_img = pygame.transform.scale(vs_img, (largura_vs, altura_vs))
        vs_x = (LARGURA - largura_vs) // 2
        vs_y = ALTURA - altura_vs - 100  # Ajuste para ficar perto das cartas
        TELA.blit(vs_img, (vs_x, vs_y))

def desenhar_jogadas_no_jogo(estado):
    """
    Desenha as jogadas escolhidas pelos jogadores diretamente no lugar correto,
    com descrição para cartas especiais. Usa tamanho fixo e suavização.
    """
    try:
        # Tamanho das jogadas
        largura_jogada = 300
        altura_jogada = 200

        # Posição para o jogador 1
        x_j1 = 0
        y_j1 = ALTURA // 2 - altura_jogada // 2

        # Jogada do Jogador 1
        if estado["jogada_jogador1"]:
            img_j1 = pygame.image.load(os.path.join(images_path, arquivos_imagens_cartas[estado["jogada_jogador1"]])).convert_alpha()
            img_j1 = pygame.transform.smoothscale(img_j1, (largura_jogada, altura_jogada))  # SUAVIZAÇÃO
            TELA.blit(img_j1, (x_j1, y_j1))

        # Posição para o jogador 2
        x_j2 = LARGURA - largura_jogada
        y_j2 = ALTURA // 2 - altura_jogada // 2

        # Jogada do Jogador 2
        if estado["jogada_jogador2"]:
            img_j2 = pygame.image.load(os.path.join(images_path, arquivos_imagens_cartas[estado["jogada_jogador2"]])).convert_alpha()
            img_j2 = pygame.transform.smoothscale(img_j2, (largura_jogada, altura_jogada))  # SUAVIZAÇÃO
            TELA.blit(img_j2, (x_j2, y_j2))
    except FileNotFoundError as e:
        int(f"Erro ao carregar imagem: {e}")

def desenhar_tela_fim_jogo(estado):
    TELA.fill((160, 211, 243))
    vencedor = estado['vencedor_final']
    if vencedor == 'Jogador 1':
        jogador_texto = "JOGADOR 1"
        gato_branco_img_path = os.path.join(images_path, "feliz_branco.png")
        gato_cinza_img_path = os.path.join(images_path, "triste_cinza.png")
        coroa_lado = "esquerda"
    else:
        jogador_texto = "JOGADOR 2"
        gato_branco_img_path = os.path.join(images_path, "tristeza_branco.png")
        gato_cinza_img_path = os.path.join(images_path, "feliz_cinza.png")
        coroa_lado = "direita"

    def render_text_with_outline(text, font, text_color, outline_color, outline_width):
        base = font.render(text, True, text_color)
        size = (base.get_width() + 2*outline_width, base.get_height() + 2*outline_width)
        img = pygame.Surface(size, pygame.SRCALPHA)
        for dx in range(-outline_width, outline_width+1):
            for dy in range(-outline_width, outline_width+1):
                if dx != 0 or dy != 0:
                    img.blit(font.render(text, True, outline_color), (dx+outline_width, dy+outline_width))
        img.blit(base, (outline_width, outline_width))
        return img

    jogador_img = render_text_with_outline(jogador_texto, fonte_grande, (255,255,255), (0,0,0), 5)
    TELA.blit(jogador_img, (LARGURA // 2 - jogador_img.get_width() // 2, ALTURA // 2 - 300))

    fonte_maior = pygame.font.SysFont(None, 48)
    jogador1_texto = "JOGADOR 1"
    jogador1_img = render_text_with_outline(jogador1_texto, fonte_maior, (255,255,255), (0,0,0), 2)
    jogador1_x = LARGURA // 2 - 525
    jogador1_y = ALTURA // 2 - 150
    TELA.blit(jogador1_img, (jogador1_x - jogador1_img.get_width() // 2, jogador1_y))

    jogador2_texto = "JOGADOR 2"
    jogador2_img = render_text_with_outline(jogador2_texto, fonte_maior, (255,255,255), (0,0,0), 2)
    jogador2_x = LARGURA // 2 + 525
    jogador2_y = ALTURA // 2 - 150
    TELA.blit(jogador2_img, (jogador2_x - jogador2_img.get_width() // 2, jogador2_y))

    venceu_img = fonte_grande.render("VENCEU", True, (76, 199, 70))
    TELA.blit(venceu_img, (LARGURA // 2 - venceu_img.get_width() // 2, ALTURA // 2 - 230))

    # Prepara os retângulos dos botões para retornar
    revanche_rect = None
    sair_rect = None

    # Desenha o botão "Revanche"
    try:
        revanche_img = pygame.image.load(os.path.join(images_path, "revanche.png")).convert_alpha()
        revanche_img = pygame.transform.smoothscale(revanche_img, (363, 118))
        revanche_rect = revanche_img.get_rect(center=(LARGURA // 2, ALTURA // 2 + 80))
        TELA.blit(revanche_img, revanche_rect)
    except pygame.error:
        pass

    # Desenha o botão "Sair"
    try:
        sair_img = pygame.image.load(os.path.join(images_path, "sair_final.png")).convert_alpha()
        sair_img = pygame.transform.smoothscale(sair_img, (363, 118))
        sair_rect = sair_img.get_rect(center=(LARGURA // 2, ALTURA // 2 + 220))
        TELA.blit(sair_img, sair_rect)
    except pygame.error:
        pass

    # Gato branco (esquerda)
    gato_branco_rect = None
    try:
        gato_branco_img = pygame.image.load(gato_branco_img_path).convert_alpha()
        gato_branco_img = pygame.transform.smoothscale(gato_branco_img, (350, 350))
        revanche_x = LARGURA // 2 - 80
        revanche_y = ALTURA // 2 + 80
        revanche_width = 363
        gato_branco_rect = gato_branco_img.get_rect(midright=(revanche_x - revanche_width // 2 - 88, revanche_y))
        TELA.blit(gato_branco_img, gato_branco_rect)
    except pygame.error:
        pass

    # Gato cinza (direita)
    gato_cinza_rect = None
    try:
        gato_cinza_img = pygame.image.load(gato_cinza_img_path).convert_alpha()
        gato_cinza_img = pygame.transform.smoothscale(gato_cinza_img, (350, 350))
        revanche_x = LARGURA // 2 + 80
        revanche_y = ALTURA // 2 + 80
        revanche_width = 363
        gato_cinza_rect = gato_cinza_img.get_rect(midleft=(revanche_x + revanche_width // 2 + 88, revanche_y))
        TELA.blit(gato_cinza_img, gato_cinza_rect)
    except pygame.error:
        pass

    # Coroa acima do vencedor
    try:
        coroa_img = pygame.image.load(os.path.join(images_path, "coroa.png")).convert_alpha()
        coroa_img = pygame.transform.smoothscale(coroa_img, (257, 257))
        if coroa_lado == "esquerda" and gato_branco_rect:
            coroa_x = gato_branco_rect.centerx
            coroa_y = gato_branco_rect.top - 32 - 257 // 2
            coroa_rect = coroa_img.get_rect(center=(coroa_x, coroa_y))
            TELA.blit(coroa_img, coroa_rect)
        elif coroa_lado == "direita" and gato_cinza_rect:
            coroa_x = gato_cinza_rect.centerx
            coroa_y = gato_cinza_rect.top - 32 - 257 // 2
            coroa_rect = coroa_img.get_rect(center=(coroa_x, coroa_y))
            TELA.blit(coroa_img, coroa_rect)
    except pygame.error:
        pass

    try:
        miausky_img = pygame.image.load(os.path.join(images_path, "miausky.png")).convert_alpha()
        miausky_img = pygame.transform.smoothscale(miausky_img, (64, 64))
        miausky_rect = miausky_img.get_rect(bottomright=(LARGURA - 0, ALTURA - 0))
        TELA.blit(miausky_img, miausky_rect)
    except pygame.error:
        pass
    
    # Esta função NÃO tem mais um loop. Ela só desenha e retorna os retângulos.
    return revanche_rect, sair_rect

def desenhar_tela_inicial():
    """Desenha a tela inicial com as imagens e botões."""
    TELA.fill((0, 255, 255))  # Fundo ciano (RGB: 0, 255, 255)

    # Carregar as imagens
    logo = pygame.image.load(os.path.join(images_path, "logo.png")).convert_alpha()
    botao_cpu = pygame.image.load(os.path.join(images_path, "botao_cpu.png")).convert_alpha()
    botao_local = pygame.image.load(os.path.join(images_path, "botao_local.png")).convert_alpha()
    botao_sair = pygame.image.load(os.path.join(images_path, "botao_sair.png")).convert_alpha()
    mao_branca = pygame.image.load(os.path.join(images_path, "mao_branca.png")).convert_alpha()
    mao_cinza = pygame.image.load(os.path.join(images_path, "mao_cinza.png")).convert_alpha()

    # Redimensionar o logo para 900x600 (menor para caber melhor)
    largura_logo = 900
    altura_logo = 600
    logo = pygame.transform.scale(logo, (largura_logo, altura_logo))

    # Redimensionar os botões para tamanhos menores
    largura_botao_cpu_local = 300
    altura_botao_cpu_local = 100
    botao_cpu = pygame.transform.scale(botao_cpu, (largura_botao_cpu_local, altura_botao_cpu_local))
    botao_local = pygame.transform.scale(botao_local, (largura_botao_cpu_local, altura_botao_cpu_local))

    largura_botao_sair = 150
    altura_botao_sair = 80
    botao_sair = pygame.transform.scale(botao_sair, (largura_botao_sair, altura_botao_sair))

    # Desenhar o botão "Sair" afastado da borda superior e esquerda
    margem_superior = 20
    margem_esquerda = 20
    pos_botao_sair = (margem_esquerda, margem_superior)
    TELA.blit(botao_sair, pos_botao_sair)

    # Desenhar o logo um pouco mais à direita e mais próximo do topo
    deslocamento_direita = 25  # ajuste para mover mais à direita
    pos_logo = (LARGURA // 2 - logo.get_width() // 2 + deslocamento_direita, 20)  # topo mais próximo (20px)
    TELA.blit(logo, pos_logo)

    # Espaçamento entre os botões
    espaco_vertical = 22  # Espaçamento vertical entre os botões

    # Centralizar bloco dos botões verticalmente
    total_altura_botoes = botao_local.get_height() + botao_cpu.get_height() + espaco_vertical
    y_botoes_inicio = pos_logo[1] + logo.get_height() + ((ALTURA - (pos_logo[1] + logo.get_height()) - total_altura_botoes) // 2)

    pos_botao_local = (LARGURA // 2 - botao_local.get_width() // 2, y_botoes_inicio)
    pos_botao_cpu = (LARGURA // 2 - botao_cpu.get_width() // 2, y_botoes_inicio + botao_local.get_height() + espaco_vertical)

    TELA.blit(botao_local, pos_botao_local)
    TELA.blit(botao_cpu, pos_botao_cpu)

    # Adicionar a imagem da mão branca no canto inferior esquerdo, encostada na borda
    pos_mao_branca = (0, ALTURA - mao_branca.get_height())
    TELA.blit(mao_branca, pos_mao_branca)

    # Adicionar a imagem da mão cinza no canto inferior direito, encostada na borda
    pos_mao_cinza = (LARGURA - mao_cinza.get_width(), ALTURA - mao_cinza.get_height())
    TELA.blit(mao_cinza, pos_mao_cinza)

    pygame.display.update()

    # Retorna as posições e tamanhos dos botões para detecção de clique
    return {
        "cpu": (pos_botao_cpu, (botao_cpu.get_width(), botao_cpu.get_height())),
        "local": (pos_botao_local, (botao_local.get_width(), botao_local.get_height())),
        "sair": (pos_botao_sair, (botao_sair.get_width(), botao_sair.get_height()))
    }


def verificar_clique(pos_mouse, pos_botao, tamanho_botao):
    """Verifica se o clique do mouse está dentro de um botão."""
    x, y = pos_mouse
    bx, by = pos_botao
    bw, bh = tamanho_botao
    return bx <= x <= bx + bw and by <= y <= by + bh


def desenhar_vidas():
    """Desenha as vidas dos jogadores como corações vermelhos e brancos, e exibe 'Jogador 1' e 'Jogador 2' abaixo."""
    coracao_red = pygame.image.load(os.path.join(images_path, "coracao_red.png")).convert_alpha()
    coracao_branco = pygame.image.load(os.path.join(images_path, "coracao_branco.png")).convert_alpha()
    coracao_red = pygame.transform.scale(coracao_red, (60, 60))  # Redimensionar para 60x60
    coracao_branco = pygame.transform.scale(coracao_branco, (60, 60))  # Redimensionar para 60x60

    # Fonte para os textos "Jogador 1" e "Jogador 2"
    fonte_jogador = pygame.font.SysFont(None, 36)

    # Jogador 1
    for i in range(7):  # Máximo de 7 vidas
        if i < estado_jogo["vidas"]["jogador1"]:
            TELA.blit(coracao_red, (20 + i * 55, 60))  # Espaçamento de 55px entre corações
        else:
            TELA.blit(coracao_branco, (20 + i * 55, 60))

    # Texto "Jogador 1" abaixo dos corações
    texto_jogador1 = fonte_jogador.render("Jogador 1", True, (0, 0, 0))  # Texto preto
    TELA.blit(texto_jogador1, (20, 130))  # Posição abaixo dos corações

    # Jogador 2
    for i in range(7):  # Máximo de 7 vidas
        if i < estado_jogo["vidas"]["jogador2"]:
            TELA.blit(coracao_red, (LARGURA - (7 - i) * 55, 60))  # Espaçamento de 55px entre corações
        else:
            TELA.blit(coracao_branco, (LARGURA - (7 - i) * 55, 60))

    # Texto "Jogador 2" abaixo dos corações
    texto_jogador2 = fonte_jogador.render("Jogador 2", True, (0, 0, 0))  # Texto preto
    TELA.blit(texto_jogador2, (LARGURA - texto_jogador2.get_width() - 20, 130))  # Posição abaixo dos corações


def desenhar_opcoes_jogadores():
    """
    Desenha as opções de jogadas (ASD, JKS, QWE, UIO) para cada jogador,
    posicionadas simetricamente nos cantos inferiores da tela.
    QWE e UIO ficam mais centralizados e mais altos.
    """
    try:
        asd_img = pygame.image.load(os.path.join(images_path, "asd.png")).convert_alpha()
        jks_img = pygame.image.load(os.path.join(images_path, "jks.png")).convert_alpha()
        qwe_img = pygame.image.load(os.path.join(images_path, "qwe.png")).convert_alpha()
        uio_img = pygame.image.load(os.path.join(images_path, "uio.png")).convert_alpha()
    except pygame.error as e:
        print(f"Erro ao carregar imagem: {e}")
        return

    # Tamanhos das imagens
    largura_opcao, altura_opcao = 251, 162
    largura_extra, altura_extra = 300, 53  # Para QWE e UIO

    # Redimensionar as imagens principais
    asd_img = pygame.transform.scale(asd_img, (largura_opcao, altura_opcao))
    jks_img = pygame.transform.scale(jks_img, (largura_opcao, altura_opcao))
    # Redimensionar as imagens extras
    qwe_img = pygame.transform.scale(qwe_img, (largura_extra, altura_extra))
    uio_img = pygame.transform.scale(uio_img, (largura_extra, altura_extra))

    margem = 2  # Distância de 2 pixels das bordas inferior, esquerda e direita

    # Desenhar ASD no canto inferior esquerdo
    pos_asd = (margem, ALTURA - altura_opcao - margem)
    TELA.blit(asd_img, pos_asd)

    # Desenhar JKS no canto inferior direito
    pos_jks = (LARGURA - largura_opcao - margem, ALTURA - altura_opcao - margem)
    TELA.blit(jks_img, pos_jks)
    # Centralizar QWE e UIO horizontalmente acima dos botões ASD/JKS e mais alto
    # Posição da imagem vs.png (centralizada na parte inferior)
    largura_vs = 80
    altura_vs = 47
    vs_x = (LARGURA - largura_vs) // 2
    vs_y = ALTURA - altura_vs

    # QWE: 35px à esquerda da imagem vs.png, 97px acima da base da imagem vs.png
    pos_qwe = (
        vs_x - largura_extra - 100,
        vs_y - 200
    )
    TELA.blit(qwe_img, pos_qwe)

    # UIO: 40px à direita da imagem vs.png, 97px acima da base da imagem vs.png
    pos_uio = (
        vs_x + largura_vs + 100,
        vs_y - 200
    )
    TELA.blit(uio_img, pos_uio)


def desenhar_cartas_estaticas():
    margem_horizontal = 10
    largura_carta = 130
    altura_carta = 180

    # Use as cartas sorteadas
    cartas_jogador1 = estado_jogo["cartas_jogador1"]
    cartas_jogador2 = estado_jogo["cartas_jogador2"]

    # Posições fixas para Q, W, E (jogador 1)
    for idx, carta in enumerate(cartas_jogador1):
        x = 270 + idx * (largura_carta + margem_horizontal)
        y = ALTURA - altura_carta - 10
        if carta in arquivos_imagens_cartas:
            img_path = os.path.join(images_path, arquivos_imagens_cartas[carta])
            if os.path.exists(img_path):
                img_carta = pygame.image.load(img_path).convert_alpha()
                img_carta = pygame.transform.scale(img_carta, (largura_carta, altura_carta))
                TELA.blit(img_carta, (x, y))

    # Posições fixas para U, I, O (jogador 2)
    for idx, carta in enumerate(cartas_jogador2):
        x = LARGURA - 690 + idx * (largura_carta + margem_horizontal)
        y = ALTURA - altura_carta - 10
        if carta in arquivos_imagens_cartas:
            img_path = os.path.join(images_path, arquivos_imagens_cartas[carta])
            if os.path.exists(img_path):
                img_carta = pygame.image.load(img_path).convert_alpha()
                img_carta = pygame.transform.scale(img_carta, (largura_carta, altura_carta))
                TELA.blit(img_carta, (x, y))


# --- 5. Funções de Lógica do Jogo ---

def sortear_carta():
    """Retorna uma carta aleatória das disponíveis para sorteio."""
    carta = random.choice(cartas_disponiveis_para_sorteio)
    print(f"Carta sorteada: {carta}")  # Para debug
    return carta

def sortear_cartas_unicas_para_jogador(cartas_possiveis, quantidade):
    cartas = []
    while len(cartas) < quantidade:
        carta = random.choice(cartas_possiveis)
        if carta not in cartas:
            cartas.append(carta)
    return cartas


def aplicar_efeito_carta(jogador, carta_nome):
    """
    Aplica o efeito de uma carta.
    Retorna True se a carta deve ser removida da mão,
    False se o efeito da carta já tratou sua remoção/substituição.
    """
    global estado_jogo

    propriedades = CARTAS_PROPRIEDADES.get(carta_nome)
    if not propriedades:
        print(f"Erro: Carta '{carta_nome}' não encontrada nas propriedades.")
        return True  # Remove a carta com erro para evitar bugs

    oponente = "jogador2" if jogador == "jogador1" else "jogador1"

    # --- LÓGICA DAS CARTAS ---

    # Caso especial: Roubo Felino
    if carta_nome == "roubo_felino":
        cartas_oponente = estado_jogo[f"cartas_{oponente}"]
        cartas_jogador = estado_jogo[f"cartas_{jogador}"]
        if cartas_oponente:
            idx_aleatorio = random.randrange(len(cartas_oponente))
            carta_roubada = cartas_oponente.pop(idx_aleatorio)
            if carta_nome in cartas_jogador:
                idx_roubo = cartas_jogador.index(carta_nome)
                cartas_jogador[idx_roubo] = carta_roubada  # A carta é substituída aqui
                print(f"{jogador} usou Roubo Felino e roubou a carta '{NOME_CARTAS_AMIGAVEL.get(carta_roubada, carta_roubada)}' de {oponente}!")
        else:
            print(f"{jogador} usou Roubo Felino, mas {oponente} não tinha cartas para roubar.")
        # A carta foi substituída, então o loop principal NÃO deve tentar removê-la.
        return False

    # Lógica para outras cartas que são apenas "usadas e descartadas"
    if carta_nome == "jogada_hacker":
        if oponente == "jogador1":
            teclas_originais = { pygame.K_s: "pedra_branco", pygame.K_d: "papel_branco", pygame.K_a: "tesoura_branco" }
        else:
            teclas_originais = { pygame.K_k: "pedra_cinza", pygame.K_l: "papel_cinza", pygame.K_j: "tesoura_cinza" }
        jogadas_possiveis = list(teclas_originais.values())
        # Garante embaralhamento sem repetição na mesma tecla
        while True:
            random.shuffle(jogadas_possiveis)
            if all(orig != embar for orig, embar in zip(list(teclas_originais.values()), jogadas_possiveis)):
                break
        mapeamento_hackeado = dict(zip(teclas_originais.keys(), jogadas_possiveis))
        estado_jogo["efeitos_ativos"][oponente][carta_nome] = {
            "duracao_restante": propriedades["duração"],
            "mapeamento": mapeamento_hackeado
        }
        print(f"{jogador} usou a Jogada do Hacker! Controles de {oponente} embaralhados.")

    elif carta_nome == "miaudicao":
        mao_amaldiçoada = random.choice(["pedra", "papel", "tesoura"])
        estado_jogo["efeitos_ativos"][oponente][carta_nome] = {
            "duracao_restante": propriedades["duração"],
            "bloqueio": mao_amaldiçoada
        }
        print(f"{jogador} usou Miaudição! O oponente foi amaldiçoado.")

    elif carta_nome == "oitava_vida":
        estado_jogo["vidas"][jogador] = min(7, estado_jogo["vidas"][jogador] + 1)
        print(f"{jogador} usou A Oitava Vida e recuperou uma vida!")

    elif "duração" in propriedades: # Para Garra Feroz e Arranhão da Sorte
        estado_jogo["efeitos_ativos"][jogador][carta_nome] = {"duracao_restante": propriedades["duração"]}
        print(f"{jogador} ativou {NOME_CARTAS_AMIGAVEL[carta_nome]}!")
        if carta_nome == "arranhao_sorte":
            mao_invencivel = random.choice(opcoes)
            if jogador == "jogador1": estado_jogo["mao_invencivel_j1"] = mao_invencivel
            else: estado_jogo["mao_invencivel_j2"] = mao_invencivel
            print(f"Arranhão da Sorte ativado! Mão invencível secreta: {mao_invencivel}")

    # Para todas as cartas, exceto Roubo Felino, dizemos ao loop para removê-las da mão.
    return True

# SUBSTITUA ESTA FUNÇÃO NO SEU CÓDIGO
def processar_jogadas_thread(jogador_num, key):
    """Processa a entrada do teclado para jogadas e cartas."""
    global estado_jogo, popup_carta, popup_carta_jogador
    with estado_jogo_lock:
        jogador_str = f"jogador{jogador_num}"
        
        # --- VERIFICA SE O JOGADOR ESTÁ HACKEADO PRIMEIRO ---
        efeitos_jogador = estado_jogo["efeitos_ativos"][jogador_str]
        if "jogada_hacker" in efeitos_jogador:
            mapeamento_hackeado = efeitos_jogador["jogada_hacker"]["mapeamento"]
            if key in mapeamento_hackeado:
                jogada_hackeada = mapeamento_hackeado[key]
                estado_jogo[f"jogada_{jogador_str}"] = jogada_hackeada
                print(f"CONTROLES EMBARALHADOS! Jogador {jogador_num} apertou uma tecla, mas a jogada foi: {jogada_hackeada.split('_')[0].upper()}")
                if jogador_num == 1: escolha_jogador1_event.set()
                else: escolha_jogador2_event.set()
                return

        # --- LÓGICA NORMAL (SE NÃO ESTIVER HACKEADO) ---
        teclas = teclas_jogador1 if jogador_num == 1 else teclas_jogador2
        cartas = estado_jogo[f"cartas_{jogador_str}"]
        
        if key in teclas:
            valor = teclas[key]
            # Se for uma CARTA ESPECIAL (QWE/UIO)
            if isinstance(valor, int):
                # CORREÇÃO: Garante que está pegando a carta do jogador correto
                if valor < len(cartas) and not estado_jogo[f"carta_especial_usada_{jogador_str}"]:
                    carta_escolhida = cartas[valor]
                    popup_carta = carta_escolhida
                    popup_carta_jogador = jogador_num
            # Se for uma JOGADA NORMAL (Pedra, Papel, Tesoura)
            elif isinstance(valor, str):
                estado_jogo[f"jogada_{jogador_str}"] = valor
                print(f"Jogador {jogador_num} escolheu: {valor.split('_')[0].upper()}")
                if jogador_num == 1: escolha_jogador1_event.set()
                else: escolha_jogador2_event.set()

def remover_efeitos_expirados():
    """Remove efeitos de cartas que expiraram."""
    global estado_jogo
    for jogador in ["jogador1", "jogador2"]:
        chaves_remover = []
        for carta_nome, efeito_data in estado_jogo["efeitos_ativos"][jogador].items():
            if "duracao_restante" in efeito_data and isinstance(efeito_data["duracao_restante"], int):
                efeito_data["duracao_restante"] -= 1
                if efeito_data["duracao_restante"] <= 0:
                    print(f"Removendo efeito {carta_nome} de {jogador}")
                    chaves_remover.append(carta_nome)
                    # Limpa mao_invencivel se Arranhão da Sorte expirar
                    if carta_nome == "arranhao_sorte":
                        if jogador == "jogador1": estado_jogo["mao_invencivel_j1"] = None
                        else: estado_jogo["mao_invencivel_j2"] = None
        for chave in chaves_remover:
            print(f"Efeito de {NOME_CARTAS_AMIGAVEL.get(chave, chave)} de {jogador} expirou.")
            del estado_jogo["efeitos_ativos"][jogador][chave]

def pode_jogar_mao(jogador, mao):
    if not mao:
        return False

    tipo_mao = mao.split('_')[0] if isinstance(mao, str) else mao
    efeitos = estado_jogo["efeitos_ativos"][jogador]

    # NÃO verifica Miaudição aqui!
    # Só mantém Arranhão da Sorte ou outros efeitos que realmente bloqueiam

    mao_invencivel = estado_jogo["mao_invencivel_j1"] if jogador == "jogador1" else estado_jogo["mao_invencivel_j2"]
    if mao_invencivel and tipo_mao == mao_invencivel:
        print(f"{jogador} jogou a mão invencível {mao_invencivel.upper()}!")

    return True


def processar_jogada_cpu():
    """
    Thread para processar jogada da CPU.
    """
    global estado_jogo
    with estado_jogo_lock:  # Adquire o lock antes de modificar o estado
        if estado_jogo["jogada_jogador2"] is None:  # Garantir que a CPU só escolha uma vez
            estado_jogo["jogada_jogador2"] = random.choice(["pedra_cinza", "papel_cinza", "tesoura_cinza"])
            print(f"CPU escolheu: {estado_jogo['jogada_jogador2']}")

def decidir_jogada_cpu():
    """
    O "Cérebro" da CPU. Decide se usa uma carta especial ou faz uma jogada normal.
    """
    global estado_jogo
    
    with estado_jogo_lock:
        # 1. A CPU pode usar uma carta especial nesta rodada?
        pode_usar_carta = not estado_jogo["carta_especial_usada_jogador2"] and estado_jogo["cartas_jogador2"]
        
        usar_carta_nesta_rodada = False
        carta_a_usar = None

        # 2. Se for possível, joga um "dado" para ver se ela decide usar uma carta
        if pode_usar_carta and random.random() < CHANCE_CPU_USAR_CARTA:
            cartas_da_cpu = estado_jogo["cartas_jogador2"]
            
            # 3. Lógica Estratégica para escolher a MELHOR carta
            
            # Prioridade 1: Usar "Oitava Vida" se estiver com pouca vida
            if "oitava_vida" in cartas_da_cpu and estado_jogo["vidas"]["jogador2"] <= 3:
                carta_a_usar = "oitava_vida"
            
            # Prioridade 2: Usar "Garra Feroz" para atacar
            elif "garra_feroz" in cartas_da_cpu:
                carta_a_usar = "garra_feroz"

            # Prioridade 3: Usar "Roubo Felino" se o jogador tiver cartas
            elif "roubo_felino" in cartas_da_cpu and estado_jogo["cartas_jogador1"]:
                carta_a_usar = "roubo_felino"
                
            # Prioridade 4: Usar cartas de "bagunça" (Hacker ou Miaudição)
            elif "jogada_hacker" in cartas_da_cpu:
                carta_a_usar = "jogada_hacker"
            elif "miaudicao" in cartas_da_cpu:
                carta_a_usar = "miaudicao"

            # Se uma carta estratégica foi escolhida, aplica o efeito
            if carta_a_usar:
                print(f"CPU decidiu usar a carta: {NOME_CARTAS_AMIGAVEL.get(carta_a_usar, carta_a_usar)}!")
                deve_remover = aplicar_efeito_carta("jogador2", carta_a_usar)
                
                if deve_remover and carta_a_usar in estado_jogo["cartas_jogador2"]:
                    estado_jogo["cartas_jogador2"].remove(carta_a_usar)
                
                estado_jogo["carta_especial_usada_jogador2"] = True

        # 4. Se a CPU não usou uma carta, faz uma jogada normal
        estado_jogo["jogada_jogador2"] = random.choice(["pedra_cinza", "papel_cinza", "tesoura_cinza"])
        print(f"CPU escolheu: {estado_jogo['jogada_jogador2'].split('_')[0]}")
        escolha_jogador2_event.set()

def calcular_resultado_rodada():
    """
    Calcula o resultado da rodada, considerando jogadas inválidas, atualiza as vidas
    e retorna o vencedor ('jogador1', 'jogador2', 'empate').
    """
    global estado_jogo

    j1_jogada = estado_jogo["jogada_jogador1"]
    j2_jogada = estado_jogo["jogada_jogador2"]

    if not j1_jogada or not j2_jogada:
        return None

    vencedor = None
    vidas_perdidas = 1

    if "garra_feroz" in estado_jogo["efeitos_ativos"]["jogador1"] or "garra_feroz" in estado_jogo["efeitos_ativos"]["jogador2"]:
        vidas_perdidas = 2

    vidas_perdidas_j1 = vidas_perdidas
    vidas_perdidas_j2 = vidas_perdidas

    # 1. VERIFICA SE A JOGADA ERA VÁLIDA
    j1_valida = True
    j2_valida = True

    # --- MIAUDIÇÃO: verifica se algum jogador caiu na armadilha ---
    efeitos_j1 = estado_jogo["efeitos_ativos"]["jogador1"]
    efeitos_j2 = estado_jogo["efeitos_ativos"]["jogador2"]

    # Jogador 1 caiu na armadilha?
    if "miaudicao" in efeitos_j1:
        jogada_amaldiçoada = efeitos_j1["miaudicao"].get("bloqueio")
        if jogada_amaldiçoada and j1_jogada and j1_jogada.split('_')[0] == jogada_amaldiçoada:
            print("Jogador 1 caiu na armadilha da Miaudição! Perde 1 vida automaticamente.")
            estado_jogo["vidas"]["jogador1"] -= 1
            estado_jogo["vidas_perdidas_acumuladas_jogador1"] += 1

    # Jogador 2 caiu na armadilha?
    if "miaudicao" in efeitos_j2:
        jogada_amaldiçoada = efeitos_j2["miaudicao"].get("bloqueio")
        if jogada_amaldiçoada and j2_jogada and j2_jogada.split('_')[0] == jogada_amaldiçoada:
            print("Jogador 2 caiu na armadilha da Miaudição! Perde 1 vida automaticamente.")
            estado_jogo["vidas"]["jogador2"] -= 1
            estado_jogo["vidas_perdidas_acumuladas_jogador2"] += 1

    # --- Segue o fluxo normal do jogo ---
    # (mantém o restante do código igual, exceto que j1_valida/j2_valida sempre True)
    tipo_j1 = j1_jogada.split('_')[0]
    tipo_j2 = j2_jogada.split('_')[0]

    if tipo_j1 == tipo_j2:
        vencedor = "empate"
    elif (tipo_j1 == "pedra" and tipo_j2 == "tesoura") or \
         (tipo_j1 == "papel" and tipo_j2 == "pedra") or \
         (tipo_j1 == "tesoura" and tipo_j2 == "papel"):
        vencedor = "jogador1"
    else:
        vencedor = "jogador2"

    # Lógica de sobrescrita por "Arranhão da Sorte"
    if estado_jogo["mao_invencivel_j1"] and tipo_j1 == estado_jogo["mao_invencivel_j1"]:
        vencedor = "jogador1"
    if estado_jogo["mao_invencivel_j2"] and tipo_j2 == estado_jogo["mao_invencivel_j2"]:
        vencedor = "jogador2"

    # 3. APLICA A PERDA DE VIDAS NORMAL
    if vencedor == "jogador1":
        estado_jogo["vidas"]["jogador2"] -= vidas_perdidas_j2
        estado_jogo["vidas_perdidas_acumuladas_jogador2"] += vidas_perdidas_j2
    elif vencedor == "jogador2":
        estado_jogo["vidas"]["jogador1"] -= vidas_perdidas_j1
        estado_jogo["vidas_perdidas_acumuladas_jogador1"] += vidas_perdidas_j1
    elif vencedor == "empate":
        # Só o adversário de quem usou Garra Feroz perde 1 vida
        if "garra_feroz" in estado_jogo["efeitos_ativos"]["jogador1"]:
            estado_jogo["vidas"]["jogador2"] -= 1
            estado_jogo["vidas_perdidas_acumuladas_jogador2"] += 1
            print("Garra Feroz: Jogador 2 perdeu 1 vida mesmo no empate!")
        elif "garra_feroz" in estado_jogo["efeitos_ativos"]["jogador2"]:
            estado_jogo["vidas"]["jogador1"] -= 1
            estado_jogo["vidas_perdidas_acumuladas_jogador1"] += 1
            print("Garra Feroz: Jogador 1 perdeu 1 vida mesmo no empate!")

    if vencedor == "empate":
        estado_jogo["empates_seguidos"] += 1
    else:
        estado_jogo["empates_seguidos"] = 0

    return vencedor

def avancar_rodada():
    global estado_jogo

    estado_jogo["rodada"] += 1

    # Permite usar carta especial novamente a cada rodada
    estado_jogo["carta_especial_usada_jogador1"] = False
    estado_jogo["carta_especial_usada_jogador2"] = False

    # Zera jogadas para próxima rodada
    estado_jogo["jogada_jogador1"] = None
    estado_jogo["jogada_jogador2"] = None

    # Verifica condição de fim de jogo por vidas
    if estado_jogo["vidas"]["jogador1"] <= 0:
        estado_jogo["vencedor_final"] = "Jogador 2"
        estado_jogo["modo"] = "fim_jogo"
    elif estado_jogo["vidas"]["jogador2"] <= 0:
        estado_jogo["vencedor_final"] = "Jogador 1"
        estado_jogo["modo"] = "fim_jogo"

    # Sorteio por 2 vidas acumuladas (já deve existir, só mantenha)
    if estado_jogo["vidas_perdidas_acumuladas_jogador1"] >= 2:
        if len(estado_jogo["cartas_jogador1"]) < 3:
            nova_carta = sortear_carta()
            while nova_carta in estado_jogo["cartas_jogador1"]:
                nova_carta = sortear_carta()
            estado_jogo["cartas_jogador1"].append(nova_carta)
        estado_jogo["vidas_perdidas_acumuladas_jogador1"] = 0

    if estado_jogo["vidas_perdidas_acumuladas_jogador2"] >= 2:
        if len(estado_jogo["cartas_jogador2"]) < 3:
            nova_carta = sortear_carta()
            while nova_carta in estado_jogo["cartas_jogador2"]:
                nova_carta = sortear_carta()
            estado_jogo["cartas_jogador2"].append(nova_carta)
        estado_jogo["vidas_perdidas_acumuladas_jogador2"] = 0

    # Sorteio por 3 empates seguidos
    if estado_jogo["empates_seguidos"] >= 3:
        if len(estado_jogo["cartas_jogador1"]) < 3:
            nova_carta = sortear_carta()
            while nova_carta in estado_jogo["cartas_jogador1"]:
                nova_carta = sortear_carta()
            estado_jogo["cartas_jogador1"].append(nova_carta)
        if len(estado_jogo["cartas_jogador2"]) < 3:
            nova_carta = sortear_carta()
            while nova_carta in estado_jogo["cartas_jogador2"]:
                nova_carta = sortear_carta()
            estado_jogo["cartas_jogador2"].append(nova_carta)
        estado_jogo["empates_seguidos"] = 0


def resetar_jogo():
    """Reseta todas as variáveis de estado do jogo para o início."""
    global estado_jogo
    estado_jogo = {
        "vidas": {"jogador1": 7, "jogador2": 7},
        "rodada": 1,
        "MAX_PARTIDAS": 13,
        "modo": None,  # Começa no menu
        "jogada_jogador1": None,
        "jogada_jogador2": None,
        "carta_ativada_jogador1": None,  # Carta especial ativada pelo jogador 1
        "carta_ativada_jogador2": None,  # Carta especial ativada pelo jogador 2
        "mostrar_jogadas": False,
        "mostrar_jogadas_tempo": 0,
        "mensagem_tela": "Escolha o modo: 1 - Jogar contra CPU | 2 - Jogar Local",
        "vencedor_final": None,
        "cartas_jogador1": [],
        "cartas_jogador2": [],
        "efeitos_ativos": {"jogador1": {}, "jogador2": {}},
        "mao_invencivel_j1": None,
        "mao_invencivel_j2": None,
        "carta_especial_usada_jogador1": False,
        "carta_especial_usada_jogador2": False,
        "vidas_perdidas_acumuladas_jogador1": 0,
        "vidas_perdidas_acumuladas_jogador2": 0,
        "empates_seguidos": 0,  # Novo campo para contar empates seguidos
    }
    for _ in range(3):
        if len(estado_jogo["cartas_jogador1"]) < 3:
            nova_carta = sortear_carta()
            while nova_carta in estado_jogo["cartas_jogador1"]:
                nova_carta = sortear_carta()
            estado_jogo["cartas_jogador1"].append(nova_carta)

        if len(estado_jogo["cartas_jogador2"]) < 3:
            nova_carta2 = sortear_carta()
            while nova_carta2 in estado_jogo["cartas_jogador2"]:
                nova_carta2 = sortear_carta()
            estado_jogo["cartas_jogador2"].append(nova_carta2)


def exibir_popup_carta(carta_nome):
    """
    Exibe um pop-up centralizado apenas com a imagem grande da carta e dois botões laterais: "Usar" e "Não Usar".
    Retorna True se o jogador escolher "Usar" e False caso contrário.
    O fundo é a tela do jogo, não escurecido.
    """
    # Tamanho da carta grande
    largura_carta, altura_carta = 350, 500
    x_carta = (LARGURA - largura_carta) // 2
    y_carta = (ALTURA - altura_carta) // 2

    # Botões
    largura_botao = 160
    altura_botao = 60
    margem_horizontal = 40

    # Botão "Usar" à esquerda da carta
    botao_usar = pygame.Rect(
        x_carta - largura_botao - margem_horizontal,
        y_carta + altura_carta // 2 - altura_botao // 2,
        largura_botao,
        altura_botao
    )
    # Botão "Não Usar" à direita da carta
    botao_nao_usar = pygame.Rect(
        x_carta + largura_carta + margem_horizontal,
        y_carta + altura_carta // 2 - altura_botao // 2,
        largura_botao,
        altura_botao
    )

    # Fonte para os botões
    fonte_popup = pygame.font.SysFont(None, 36)

    # Carregar a imagem da carta
    img_path = os.path.join(images_path, arquivos_imagens_cartas.get(carta_nome, ""))
    if os.path.exists(img_path):
        img_carta = pygame.image.load(img_path).convert_alpha()
        img_carta = pygame.transform.scale(img_carta, (largura_carta, altura_carta))
    else:
        img_carta = None

    rodando_popup = True
    while rodando_popup:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif evento.type == pygame.MOUSEBUTTONDOWN and evento.button == 1:
                mouse_pos = pygame.mouse.get_pos()
                if botao_usar.collidepoint(mouse_pos):
                    return True
                elif botao_nao_usar.collidepoint(mouse_pos):
                    return False

        # Redesenha a tela do jogo como fundo
        TELA.fill((243, 240, 146))
        desenhar_hud(estado_jogo)
        desenhar_vidas()
        desenhar_opcoes_jogadores()
        desenhar_cartas_estaticas()

        # Desenha a carta grande centralizada
        if img_carta:
            TELA.blit(img_carta, (x_carta, y_carta))

        # Botão "Usar"
        pygame.draw.rect(TELA, (0, 180, 0), botao_usar, border_radius=12)
        texto_usar = fonte_popup.render("Usar", True, (255, 255, 255))
        TELA.blit(
            texto_usar,
            (botao_usar.x + (largura_botao - texto_usar.get_width()) // 2,
             botao_usar.y + (altura_botao - texto_usar.get_height()) // 2)
        )

        # Botão "Não Usar"
        pygame.draw.rect(TELA, (180, 0, 0), botao_nao_usar, border_radius=12)
        texto_nao_usar = fonte_popup.render("Não Usar", True, (255, 255, 255))
        TELA.blit(
            texto_nao_usar,
            (botao_nao_usar.x + (largura_botao - texto_nao_usar.get_width()) // 2,
             botao_nao_usar.y + (altura_botao - texto_nao_usar.get_height()) // 2)
        )

        pygame.display.update()


# --- 6. Loop Principal do Jogo ---
def game_loop():
    global estado_jogo, popup_carta, popup_resultado, popup_carta_jogador
    rodando = True
    clock = pygame.time.Clock()
    tela_inicial = True
    
    bloqueado = False
    tempo_inicio_bloqueio = 0
    tempo_de_exibicao = 3000

    # Variáveis para os botões que serão retornados pelas funções de desenho
    botoes_menu_inicial = {}
    revanche_rect = None
    sair_final_rect = None

    while rodando:
        delta_time = clock.tick(60) / 1000.0

        # --- Gerenciamento de Eventos Centralizado ---
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                rodando = False
            
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_ESCAPE:
                    rodando = False

                if not bloqueado and estado_jogo["modo"] in ["local", "cpu"]:
                    if estado_jogo["jogada_jogador1"] is None:
                        threading.Thread(target=processar_jogadas_thread, args=(1, evento.key)).start()
                    if estado_jogo["modo"] == "local" and estado_jogo["jogada_jogador2"] is None:
                        threading.Thread(target=processar_jogadas_thread, args=(2, evento.key)).start()
            
            if evento.type == pygame.MOUSEBUTTONDOWN and evento.button == 1:
                mouse_pos = pygame.mouse.get_pos()
                
                # Se estiver na tela inicial, verifica cliques nos botões do menu
                if tela_inicial:
                    if verificar_clique(mouse_pos, *botoes_menu_inicial["local"]):
                        estado_jogo["modo"] = "local"
                        tela_inicial = False
                    elif verificar_clique(mouse_pos, *botoes_menu_inicial["cpu"]):
                        estado_jogo["modo"] = "cpu"
                        tela_inicial = False
                    elif verificar_clique(mouse_pos, *botoes_menu_inicial["sair"]):
                        rodando = False
                
                # Se estiver na tela final, verifica cliques nos botões de revanche/sair
                elif estado_jogo["modo"] == "fim_jogo":
                    if revanche_rect and revanche_rect.collidepoint(mouse_pos):
                        resetar_jogo()
                        tela_inicial = True  # A chave para voltar ao menu!
                    elif sair_final_rect and sair_final_rect.collidepoint(mouse_pos):
                        rodando = False

        # --- Lógica de Pop-up de Cartas ---
        if popup_carta:
            pode_usar = (popup_carta_jogador == 1 and not estado_jogo["carta_especial_usada_jogador1"]) or \
                        (popup_carta_jogador == 2 and not estado_jogo["carta_especial_usada_jogador2"])
            
            if pode_usar:
                popup_resultado = exibir_popup_carta(popup_carta)
                if popup_resultado:
                    jogador_ativo = "jogador1" if popup_carta_jogador == 1 else "jogador2"
                    deve_remover = aplicar_efeito_carta(jogador_ativo, popup_carta)
                    if deve_remover:
                        if popup_carta_jogador == 1 and popup_carta in estado_jogo["cartas_jogador1"]:
                            estado_jogo["cartas_jogador1"].remove(popup_carta)
                        elif popup_carta_jogador == 2 and popup_carta in estado_jogo["cartas_jogador2"]:
                            estado_jogo["cartas_jogador2"].remove(popup_carta)
                    if popup_carta_jogador == 1:
                        estado_jogo["carta_especial_usada_jogador1"] = True
                    else:
                        estado_jogo["carta_especial_usada_jogador2"] = True
                else:
                    print(f"Jogador decidiu não usar a carta '{popup_carta}'.")
            else:
                print("Só é permitido usar uma carta especial por partida!")
            popup_carta = None

        # --- Lógica de Renderização ---
        if tela_inicial:
            botoes_menu_inicial = desenhar_tela_inicial()
        
        elif estado_jogo["modo"] == "fim_jogo" or estado_jogo["vidas"]["jogador1"] <= 0 or estado_jogo["vidas"]["jogador2"] <= 0:
            if estado_jogo["vidas"]["jogador1"] <= 0 and not estado_jogo["vencedor_final"]:
                estado_jogo["vencedor_final"] = "Jogador 2"
                estado_jogo["modo"] = "fim_jogo"
            elif estado_jogo["vidas"]["jogador2"] <= 0 and not estado_jogo["vencedor_final"]:
                estado_jogo["vencedor_final"] = "Jogador 1"
                estado_jogo["modo"] = "fim_jogo"
            
            # A função de desenho agora é chamada a cada frame e retorna os retângulos
            revanche_rect, sair_final_rect = desenhar_tela_fim_jogo(estado_jogo)
        
        else: # Tela principal do jogo
            TELA.fill((243, 240, 146))
            desenhar_hud(estado_jogo)
            desenhar_vidas()
            desenhar_opcoes_jogadores()
            desenhar_cartas_estaticas()

            if bloqueado:
                desenhar_jogadas_no_jogo(estado_jogo)
                if pygame.time.get_ticks() - tempo_inicio_bloqueio > tempo_de_exibicao:
                    vencedor = calcular_resultado_rodada()
                    remover_efeitos_expirados()
                    avancar_rodada()
                    escolha_jogador1_event.clear()
                    escolha_jogador2_event.clear()
                    bloqueado = False 
            else:
                jogada_local_pronta = (estado_jogo["modo"] == "local" and escolha_jogador1_event.is_set() and escolha_jogador2_event.is_set())
                jogada_cpu_pronta = (estado_jogo["modo"] == "cpu" and estado_jogo["jogada_jogador1"] is not None)

                if jogada_local_pronta or jogada_cpu_pronta:
                    if estado_jogo["modo"] == "cpu" and estado_jogo["jogada_jogador2"] is None:
                        decidir_jogada_cpu()
                    bloqueado = True
                    tempo_inicio_bloqueio = pygame.time.get_ticks()

        pygame.display.update()

    pygame.quit()
    exit()

# --- Ponto de Entrada Principal ---
if __name__ == "__main__":
    for _ in range(3):
        if len(estado_jogo["cartas_jogador1"]) < 3:
            nova_carta = sortear_carta()
            while nova_carta in estado_jogo["cartas_jogador1"]:
                nova_carta = sortear_carta()
            estado_jogo["cartas_jogador1"].append(nova_carta)

        if len(estado_jogo["cartas_jogador2"]) < 3:
            nova_carta2 = sortear_carta()
            while nova_carta2 in estado_jogo["cartas_jogador2"]:
                nova_carta2 = sortear_carta()
            estado_jogo["cartas_jogador2"].append(nova_carta2)

    game_loop()
