# Explicação do Código "MIAUPOWWWWW"

Este código é um jogo de Pedra, Papel e Tesoura com elementos especiais, desenvolvido em Python usando a biblioteca Pygame. Vamos analisá-lo por partes:

## 1. Estrutura Geral

O jogo possui:
- Um sistema de menu inicial com opções para jogar contra CPU ou em modo local (2 jogadores)
- Um sistema de cartas especiais que adicionam habilidades aos jogadores
- Um sistema de vidas (máximo de 7 para cada jogador)
- Rodadas que avançam até um máximo de 13 partidas

## 2. Componentes Principais

### a) Inicialização e Configuração
- Importa bibliotecas necessárias (random, pygame, os, threading)
- Configura a tela em modo redimensionável usando o tamanho total do monitor
- Carrega imagens e fontes necessárias

### b) Variáveis de Estado
- `estado_jogo`: Dicionário que armazena todo o estado atual do jogo (vidas, rodada, jogadas, cartas, etc.)
- `teclas_jogador1` e `teclas_jogador2`: Mapeamento das teclas para cada jogador
- `CARTAS_PROPRIEDADES`: Define os efeitos especiais de cada carta

### c) Cartas Especiais
O jogo inclui cartas especiais com diferentes efeitos:
- Jogada do Hacker
- A Oitava Vida (recupera vida)
- Miaudição
- Garra Feroz (causa dano duplo)
- Roubo Felino
- Arranhão da Sorte

### d) Threads e Sincronização
- Usa threads para processar as jogadas dos jogadores simultaneamente
- Utiliza semáforos (`estado_jogo_lock`) para proteger o estado do jogo contra condições de corrida
- Eventos (`escolha_jogador1_event`, `escolha_jogador2_event`) para sincronizar as escolhas dos jogadores

## 3. Fluxo do Jogo

1. **Tela Inicial**: Mostra opções para jogar contra CPU ou em modo local
2. **Durante o Jogo**:
   - Cada jogador escolhe sua jogada (Pedra, Papel ou Tesoura) ou ativa uma carta especial
   - O resultado da rodada é calculado baseado nas regras clássicas de Pedra-Papel-Tesoura
   - Cartas especiais podem modificar o resultado ou dar habilidades temporárias
3. **Fim de Jogo**: Quando um jogador perde todas as vidas (chega a 0)

## 4. Funções Principais

- `desenhar_tela_inicial()`: Renderiza o menu inicial com botões
- `processar_jogadas_thread()`: Processa as escolhas dos jogadores em threads separadas
- `calcular_resultado_rodada()`: Determina o vencedor da rodada
- `aplicar_efeito_carta()`: Ativa os efeitos das cartas especiais
- `exibir_popup_carta()`: Mostra um pop-up quando uma carta especial é ativada

## 5. Como Jogar

- **Jogador 1** usa teclas: 
  - A/S/D para Tesoura/Pedra/Papel
  - Q/W/E para cartas especiais
- **Jogador 2** usa teclas: 
  - J/K/L para Tesoura/Pedra/Papel
  - U/I/O para cartas especiais

O jogo tem um estilo visual colorido com elementos felinos (como sugere o nome "MIAUPOWWWWW") e foi projetado para ser jogado em tela cheia.

Este é um jogo completo que combina o clássico Pedra-Papel-Tesoura com elementos de estratégia através das cartas especiais, tornando-o mais complexo e interessante que a versão tradicional.
