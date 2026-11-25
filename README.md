# Brick Game Car

> Um jogo de corrida no estilo dos minigames Brick Game, em C, no terminal: você desvia o carro dos outros veículos que descem pela pista.

**Contexto:** UFSM · Sistemas de Informação · Lógica e Algoritmo · Prof. Dr. Joaquim Assunção · 2026/1

## Sobre

O enunciado é um trabalho prático em duplas que entrega o jogo pela metade, com interface, variáveis globais e algumas funções prontas, e pede que o aluno escreva o corpo de três funções:

- `InicializaObjetos()`: coloca o carro no centro inferior da tela usando só as variáveis globais;
- `Timer()`: chamada a cada fração de segundo, faz os obstáculos descerem, verifica se o carro continua inteiro (se não estiver, houve colisão) e gera novos veículos;
- `keyPress()`: move o carro para a esquerda com `A` e para a direita com `D`.

No original o jogo roda em Delphi, com uma `TStringGrid` chamada `mat` e `TTimer`. Este repositório traz o jogo inteiro portado para C e rodando no terminal, mantendo os mesmos nomes (`mat`, `tamX`, `tamY`, `noBorder`, `timer`, `keyPress`, `inicializaObjetos`) para que a correspondência com o enunciado fique clara. O exercício pratica matrizes, varredura de matriz de baixo para cima, detecção de colisão por inspeção de células e o laço de um jogo em tempo real.

## Funcionalidades

- Pista de 30 linhas por 60 colunas guardada em `char mat[30][60]`, com bordas `|` e `-`.
- O carro do jogador (`C`) tem 7 pixels:

  ```
   C
  CCC
   C
  C C
  ```

- Os outros veículos (`X`) têm o mesmo formato. A cada tique há 1 chance em 4 de surgir um no topo, em coluna aleatória e só se o espaço estiver livre.
- A cada 200 ms todos os obstáculos descem uma linha. A matriz é varrida de baixo para cima, para que nenhum pixel seja movido duas vezes no mesmo tique.
- **Colisão:** a cada tique o programa confere se os 7 pixels do carro continuam na matriz. Se um obstáculo ocupou algum deles, é *game over*.
- **Pontuação:** 1 ponto por pixel de obstáculo que sai pelo fim da pista (um veículo inteiro vale 7 pontos).
- `noBorder('l')` e `noBorder('r')` impedem o carro de sair pelas laterais.
- No fim aparece `*** GAME OVER ***  Score: N`, no lugar da `MessageBox('Game Over', ':/')` do enunciado.

### Diferença entre as versões

| Arquivo | Como sabe onde está o carro |
|---|---|
| `brickgamecar.c` (v1) | guarda a posição em uma variável global extra, `Carro carro` (linha e coluna) |
| `brickgamecar_v2.c` (v2) | não usa variável extra: procura na matriz a linha com três `C` seguidos (o meio do carro), cumprindo a regra do enunciado de usar só as variáveis globais |

A jogabilidade das duas é a mesma.

## Tecnologias

- C99 com GCC
- Windows: `conio.h` (`_kbhit`, `_getch`) para ler o teclado sem bloquear e `windows.h` (`Sleep`)
- Linux: `unistd.h` (`usleep`)
- A tela é redesenhada com `system("cls")` ou `system("clear")` a cada quadro

## Estrutura do projeto

| Caminho | Descrição |
|---|---|
| `brickgamecar_v2.c` | Versão 2, a que segue a restrição de variáveis do enunciado |
| `brickgamecar.c` | Versão 1, com a posição do carro em uma struct global |
| `enunciado/` | Fotos do enunciado impresso: descrição do jogo, funções pedidas, variáveis globais e tabela de sintaxe pseudocódigo para Delphi |
| `referencias/Unit1.pas` | Implementação do jogo em Delphi (Object Pascal/VCL, com `TStringGrid` e `TTimer`), usada como base da versão em C. O formulário `.dfm` não está no repositório, então ela não compila sozinha. |
| `referencias/xadrezteste.c` | "Mini chess" em matriz 5x6, outro exercício da disciplina, usado como referência de manipulação de matriz em C |

## Como rodar

### Pré-requisitos

- GCC
  - Windows: [MSYS2](https://www.msys2.org/) com MinGW-w64 (inclui `conio.h`)
  - Linux: `sudo apt install gcc` (Debian/Ubuntu) ou equivalente
- **Para jogar de fato, use o Windows.** No Linux o programa compila e roda, mas não lê o teclado (veja [Observações](#observações)).

### Passo a passo

```bash
git clone https://github.com/itskelme/ufsm-si-brick-game-car.git
cd ufsm-si-brick-game-car
```

**Windows** (rode no Prompt de Comando, no PowerShell ou no Windows Terminal. O terminal do Git Bash, mintty, não é um console nativo e o `_kbhit` pode não funcionar nele):

```powershell
gcc -Wall -std=c99 -o brickgamecar.exe brickgamecar_v2.c
.\brickgamecar.exe
```

**Linux:**

```bash
gcc -Wall -o brickgamecar brickgamecar_v2.c
./brickgamecar
```

No Linux, não use `-std=c99`: nesse modo o `usleep` não é declarado, e o GCC 14 ou mais novo trata isso como erro. Sem a opção, o padrão do GCC (`gnu17`) já inclui o `usleep`.

Para a versão 1, troque `brickgamecar_v2.c` por `brickgamecar.c`.

## Como usar

| Tecla | Ação |
|---|---|
| `A` ou `a` | move o carro uma coluna para a esquerda |
| `D` ou `d` | move o carro uma coluna para a direita |
| Ctrl+C | encerra o jogo |

A pontuação aparece no topo da tela. O jogo termina quando um veículo atinge o carro.

```
Pontuacao: 14
------------------------------------------------------------
|                 X                                        |
|                XXX                                       |
|                 X                                        |
|                X X                                       |
|                                                          |
...
|                             C                            |
|                            CCC                           |
|                             C                            |
|                            C C                           |
------------------------------------------------------------
```

## Observações

Pontos encontrados ao revisar o código para este README:

- **Linux sem teclado:** fora do Windows, `TECLA_DISP()` é definido como `0`, então nenhuma tecla é lida e o carro fica parado até a primeira colisão. Para jogar no Linux seria preciso colocar o terminal em modo cru (`termios`) e ler sem bloquear.
- **Tamanho da pista:** o enunciado fala em 30 linhas por 65 colunas. O código, como a versão em Delphi, usa 30 por 60.
- **Colisão lateral:** quando o carro anda de lado para cima de um obstáculo, `keyPress` sobrescreve os pixels `X` com `C`. A batida só é detectada quando algum obstáculo desce sobre o carro.
- **v2, acesso fora da matriz:** se uma colisão apagar a linha do meio do carro, a busca pelos três `C` falha e fica com `carroLin = 0`, e a verificação de integridade lê `mat[-1][...]`. Na prática o jogo termina em *game over* do mesmo jeito, mas é comportamento indefinido em C.
- O redesenho com `system("cls")`/`system("clear")` a cada 200 ms faz a tela piscar.

## Referências

- Enunciado do trabalho (fotos em [`enunciado/`](enunciado)), Prof. Dr. Joaquim Assunção, UFSM, Centro de Tecnologia
- Implementação em Delphi usada como base: [`referencias/Unit1.pas`](referencias/Unit1.pas)
- [Brick Game, Wikipedia](https://en.wikipedia.org/wiki/Brick_Game): os minigames que inspiraram o trabalho
- [`conio.h`: `_kbhit` e `_getch`, documentação Microsoft](https://learn.microsoft.com/cpp/c-runtime-library/reference/kbhit)

## Autores

- Diego Kelme Santos da Silva ([@itskelme](https://github.com/itskelme))
- Prof. Dr. Joaquim Assunção, docente da disciplina e autor do enunciado

## Licença

Este projeto está sob a licença [CC BY-NC 4.0](LICENSE): uso pessoal e modificação permitidos com atribuição; uso comercial proibido. Material de terceiros (enunciados, código fornecido pelo professor) pertence aos respectivos autores.
