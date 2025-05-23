
# 🐾 MIAUPÔ

Um jogo de **pedra, papel e tesoura com cartas especiais**, onde dois jogadores (ou você contra o computador) se enfrentam para zerar a vida do adversário. Totalmente felinizado, divertido e com efeitos únicos para cada carta!

## 🎮 Como jogar

### Modo de jogo
- **1** — Jogar contra CPU
- **2** — Jogar local com outro jogador

### Controles
#### Jogador 1:
- **A** — Pedra
- **S** — Papel
- **D** — Tesoura
- **Q** — Usar carta especial

#### Jogador 2 (somente no modo local):
- **←** — Pedra
- **↑** — Papel
- **→** — Tesoura
- **M** — Usar carta especial

### Cartas especiais
- **Jogada Hacker**: força o adversário a fazer uma jogada aleatória
- **A Oitava Vida**: bloqueia a última jogada feita pelo oponente na próxima rodada
- **Garra Feroz**: o dano da próxima vitória será 2 vidas

🃏 Cartas são sorteadas nas rodadas 3, 6 e 9!

## 🐱 Regras
- Cada jogador começa com 7 vidas
- Cada rodada, os jogadores fazem uma jogada
- O vencedor faz o oponente perder 1 vida (ou 2 se estiver com Garra Feroz)
- Empates não causam dano
- O jogo termina quando um jogador zera suas vidas ou ao final da 13ª rodada

## 🖼️ Pré-requisitos

- [Python 3](https://www.python.org/)
- [Pygame](https://pypi.org/project/pygame/)

Instale o Pygame com:

```bash
pip install pygame
