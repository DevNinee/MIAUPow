# Explicação Técnica: Sistema de Threads e Sincronização no MIAUPOWWWWW

##  Arquitetura de Threads

O MIAUPOWWWWW utiliza um sistema multi-thread para processamento paralelo das jogadas, garantindo uma experiência fluida aos jogadores. A implementação atual emprega:

- **Threads dedicadas** para processamento de jogadas
- **Mecanismos de sincronização** para coordenação entre threads
- **Acesso thread-safe** ao estado compartilhado do jogo

##  Componentes de Sincronização

### `estado_jogo_lock` (Semáforo)
```python
estado_jogo_lock = threading.Lock()
```
Protege o acesso concorrente ao dicionário `estado_jogo` que armazena todas as variáveis críticas do jogo.

### Eventos de Jogada
```python
escolha_jogador1_event = threading.Event()
escolha_jogador2_event = threading.Event()
```
Sinalizam quando cada jogador completou sua jogada, permitindo que o jogo prossiga apenas quando ambos estiverem prontos.

##  Fluxo de Processamento

1. **Início da Jogada**:
   - Tecla é pressionada por um jogador
   - Thread dedicada é criada para processar a jogada:
     ```python
     threading.Thread(target=processar_jogadas_thread, args=(jogador_num, key)).start()
     ```

2. **Processamento Seguro**:
   - A thread adquire o lock antes de acessar o estado:
     ```python
     with estado_jogo_lock:
         # Atualiza estado_jogo de forma segura
     ```

3. **Sinalização**:
   - Após processar, a thread sinaliza conclusão:
     ```python
     escolha_jogadorX_event.set()
     ```

4. **Sincronização**:
   - Loop principal verifica eventos antes de avançar:
     ```python
     if escolha_jogador1_event.is_set() and escolha_jogador2_event.is_set():
         # Calcula resultado e avança rodada
     ```

##  Garantias de Segurança

- **Exclusão mútua**: O lock previte condições de corrida no estado do jogo
- **Coordenação**: Os eventos garantem que o jogo só avance quando ambos jogadores estiverem prontos
- **Isolamento**: Cada jogada é processada em seu próprio contexto

##  Benefícios da Implementação

1. **Responsividade**: Interface permanece fluida durante processamento
2. **Justiça**: Ambos jogadores podem jogar simultaneamente
3. **Extensibilidade**: Fácil adição de novas mecânicas thread-safe
4. **Manutenibilidade**: Separação clara de responsabilidades

Esta arquitetura foi cuidadosamente projetada para balancear desempenho paralelo com segurança no acesso a dados compartilhados, seguindo boas práticas de programação concorrente.
