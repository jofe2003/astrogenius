# A S T R O G E N I U S

O **AstroGenius** é um jogo no estilo *arcade* de asteroides, projetado para ser sintetizado numa **placa FPGA**. O jogador controla uma nave no centro de um campo de asteroides e deve destruí-los com tiros (normais ou especiais) enquanto evita colisões.

Projeto desenvolvido na disciplina **PCS3635 – Laboratório Digital I** (Escola Politécnica da USP, turma T5BB5). O repositório original da equipe é [henriqueedu2001/poli-asteroids](https://github.com/henriqueedu2001/poli-asteroids).

<p align="center">
  <img src="docs/media/demo.gif" alt="AstroGenius rodando: controle do arcade e o jogo na tela" width="480">
  <br>
  <em>Jogo rodando na FPGA, com a interface gráfica no monitor (trecho do vídeo da entrega da semana 4).</em>
</p>

**Autores**
- [Felipe Luis Korbes](https://github.com/luis-fk)
- [Henrique S. Souza](https://github.com/henriqueedu2001)
- [João Felipe Souza Melo](https://github.com/jofe2003)

## Sumário

- [Descrição](#descrição)
- [Demonstração](#demonstração)
- [Requisitos](#requisitos)
- [Instalação](#instalação)
- [Uso](#uso)
- [Estrutura de pastas](#estrutura-de-pastas)
- [Evolução do projeto](#evolução-do-projeto)
- [Fitas binárias (modo de depuração)](#fitas-binárias-modo-de-depuração)
- [Protocolo de transmissão serial](#protocolo-de-transmissão-serial)
- [Licença](#licença)

## Descrição

O jogo foi pensado para funcionar como um arcade: um controle fixo na carcaça e um monitor que exibe os gráficos. O projeto tem três partes:

1. **Hardware (Verilog)**: toda a lógica do jogo (menu, movimento de asteroides e tiros, colisões, pontuação, vidas e dificuldade) roda numa FPGA **Cyclone V 5CEBA4F23C7N**. O módulo de topo é `astro_genius` (`src/AstroGenius/astro_genius.v`). Ele recebe os comandos por 6 chaves/botões e envia o estado do jogo pela saída `saida_serial`.
2. **Comunicação serial (UART)**: a placa envia ao computador, por UART a 9600 baud, um pacote de 51 bytes com o estado do jogo e, nas trocas de tela, um *byte de menu*.
3. **Interface gráfica (Python + pygame)**: lê os dados da porta serial, decodifica os pacotes e desenha as telas (menu inicial, gameplay, game over, pontuações) no monitor.

Para testar a interface sem a placa, existe um **modo de depuração**: os bytes vêm de um arquivo de texto (uma "fita binária") em vez da porta serial.

## Demonstração

O GIF no topo é um trecho de 12 segundos do vídeo `jogoFuncionando.mp4`, gravado no laboratório em 03/04/2024. Primeiro aparece o controle do arcade (a caixa com os botões, ligada à FPGA); depois, a tela com a pontuação subindo enquanto a nave atira nos asteroides.

O vídeo completo (41 s) está dentro de `src/Entregas/semana4/PCS3635-T5BB5-Relatorio-Semana4.zip`.

## Requisitos

### Interface gráfica
- Python **3.8 ou mais recente** (testado com 3.8 e 3.10)
- [pygame](https://www.pygame.org/) (testado com 2.5.2)
- [pyserial](https://pypi.org/project/pyserial/) (só é usado no modo `gameplay`, mas é importado sempre)

### Hardware (opcional, para sintetizar ou simular)
- Placa FPGA com Cyclone V **5CEBA4F23C7N** (ex.: DE0-CV)
- **Intel Quartus Prime**, para abrir os arquivos `.qar` e gravar a placa
- Um adaptador USB-serial ligado à saída `saida_serial` da placa
- [Icarus Verilog](https://steveicarus.github.io/iverilog/) e [GTKWave](https://gtkwave.sourceforge.net/), para simular os testbenches
- [Digital](https://github.com/hneemann/Digital), para abrir os circuitos `.dig`

## Instalação

```bash
git clone https://github.com/jofe2003/astrogenius.git
cd astrogenius

# (recomendado) ambiente virtual
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install pygame pyserial
```

Também é possível baixar uma versão específica (veja as [tags](https://github.com/jofe2003/astrogenius/tags)):

```bash
git clone --branch v2.0 --depth 1 https://github.com/jofe2003/astrogenius.git
```

## Uso

### 1. Interface gráfica

A configuração fica no dicionário `GAME_CONFIG` em `start_game.py`. Depois de ajustar, rode na raiz do repositório:

```bash
python start_game.py
```

Principais opções de `GAME_CONFIG`:

| Chave | Descrição |
|-------|-----------|
| `mode` | `'gameplay'` lê da porta serial (placa); `'debug'` lê de uma fita binária |
| `serial_port` | Porta serial da placa, ex.: `'COM4'` (Windows) ou `'/dev/ttyUSB0'` (Linux) |
| `byte_tape` | Nome da fita usada no modo `debug`, sem extensão (ex.: `'in_default'`, `'fita_01'`) |
| `screen_widht` / `screen_heigth` | Tamanho da janela em pixels (os nomes das chaves são escritos assim mesmo) |
| `delay` | Pausa, em segundos, entre leituras (útil para assistir uma fita em câmera lenta) |
| `print_*` | Ativam prints de depuração no terminal (buffer, chunk, UART, dados recebidos, tela atual) |

**Para testar sem a placa**, coloque `'mode': 'debug'` e escolha uma fita em `byte_tape`.

Para listar as portas seriais disponíveis:

```bash
python -c "import serial.tools.list_ports as p; [print(x.device) for x in p.comports()]"
```

No modo `gameplay`, se a porta não abrir, o programa tenta de novo a cada 2 segundos.

### 2. Hardware

- **Quartus**: abra `src/Quartus/astro_genius.qar` (*Project → Restore Archived Project*), compile e grave a placa.
- **Simulação com Icarus Verilog**: em `src/AstroGenius/` existe um `comp.txt` com a lista de arquivos a compilar:
  ```bash
  cd src/AstroGenius
  iverilog -c comp.txt -o sim && vvp sim
  gtkwave wave.vcd          # os arquivos h.gtkw e menu.gtkw guardam layouts de sinais
  ```
  Para simular outro testbench, troque o arquivo `*_tb.v` que está ativo no `comp.txt`.
- **Arquivo único**: `src/AstroGenius/comp.py` junta todos os módulos em `digital.v`, que serve para importar o projeto no simulador Digital. Esse arquivo é gerado e não fica versionado:
  ```bash
  cd src/AstroGenius && python comp.py
  ```

## Estrutura de pastas

```
poli-asteroids/
├── start_game.py            # Ponto de entrada da interface gráfica (configuração em GAME_CONFIG)
├── bytetapes/               # Fitas binárias (.bin) para o modo de depuração
├── docs/
│   ├── chunk_encoding.md    # Especificação do pacote serial FPGA -> PC
│   └── media/demo.gif       # Trecho do vídeo de demonstração
└── src/
    ├── AstroGenius/         # Versão atual do hardware (Verilog)
    │   ├── astro_genius.v   # Módulo de topo
    │   ├── jogo_base.v      # Fluxo de dados do jogo
    │   ├── asteroide.v, tiro.v
    │   ├── UC/              # Unidades de controle (máquinas de estado) + testbenches em UC/tb/
    │   ├── Modulos/         # Blocos básicos: contadores, registradores, muxes, UART TX, memórias...
    │   ├── tb/              # Testbenches de submódulos
    │   ├── comp.txt         # Lista de arquivos para o iverilog
    │   ├── comp.py          # Gera digital.v (todos os módulos num único arquivo, não versionado)
    │   └── *.gtkw           # Layouts de sinais do GTKWave
    ├── Interface/           # Interface gráfica em Python
    │   ├── main.py          # Classe Game: laço principal, recepção UART/fita, troca de telas
    │   ├── render/          # Telas (menu, gameplay, game over, pontuações, debug) e sprites em imgs/
    │   └── utilitary/       # UART, buffer, chunk e decodificador, leitor de fitas binárias
    ├── Quartus/             # Projeto Quartus arquivado (astro_genius.qar)
    ├── Digital/             # Circuitos do simulador Digital (.dig)
    ├── Verilog/             # Primeira versão do hardware (legado)
    ├── Entregas/            # Entregas semanais da disciplina (veja "Evolução do projeto")
    ├── Pitch/               # Apresentação inicial do projeto
    ├── Design/              # Documento de design
    └── Misc/                # Rascunhos e arquivos avulsos
```

## Evolução do projeto

O projeto foi feito em quatro semanas de laboratório, de março a abril de 2024. A cada semana a equipe entregava um planejamento em PDF e, junto, o projeto Quartus (`astro_genius.qar`) ou o Verilog daquele momento. Esses arquivos ficam em `src/Entregas/` e mostram como o jogo foi crescendo:

| Semana | Data | O que tem | Arquivos |
|--------|------|-----------|----------|
| 1 | 13/03/2024 | Pitch da ideia, primeira versão do fluxo de dados e das unidades de controle, simulações no Digital e testbenches | `src/Pitch/PCS3635-T5BB5-Pitch-Semana1.pdf`, `src/Entregas/PCS3635-T5BB5-Planejamento-Semana_1.{pdf,zip}`, `src/Verilog/` |
| 2 | 20/03/2024 | Fluxo de dados e unidades de controle de asteroides e tiros (mover, registrar tiro, comparar colisões) | `src/Entregas/semana2/` (PDF, ZIP e o Verilog entregue em `Verilog/`) |
| 3 | 27/03/2024 | Requisitos, primeira versão da serialização (UART) e projeto Quartus | `src/Entregas/semana3/` |
| 4 | 03/04/2024 | Hardware completo em Verilog, interface gráfica em pygame, serialização, maquete e controle do arcade. O relatório traz o vídeo do jogo funcionando | `src/Entregas/semana4/` |

Os `.zip` são os pacotes enviados em cada entrega (PDF + projeto) e foram mantidos como registro. A versão final do hardware está em `src/AstroGenius/`. As versões publicadas estão marcadas nas tags do git: `v1.0`, `v1.0.1`, `v1.0.2`, `v1.0.3` e `v2.0`.

## Fitas binárias (modo de depuração)

Uma fita binária é um arquivo `.bin` em `bytetapes/`, escrito em **texto**, que lista os bytes que a interface vai "receber". A fita padrão é `in_default.bin`, que mostra a navegação pelo menu e um trecho de gameplay.

```
# FITA PADRÃO DE TESTE
# comentários começam com # ou //

input f0 # byte de menu: menu inicial
input f4 # byte de menu: gameplay

input # bloco 01 (51 bytes)
01 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 41 41 41 41 41
41 41 41
```

Regras:
- Cada leitura começa com o token `input` (ou `INPUT`). `output`/`OUTPUT` também são aceitos.
- Os bytes vêm em hexadecimal, com dígitos maiúsculos ou minúsculos. Espaços e quebras de linha são ignorados, mas o **total de dígitos de cada bloco precisa ser par**.
- Comentários de uma linha com `#` ou `//`. O que fica depois do token é ignorado até o fim da linha.
- Um bloco com **um único byte** é tratado como *byte de menu* e troca a tela:

| Byte | Tela |
|------|------|
| `F0` | Menu inicial |
| `F1` | Pontuações |
| `F2` | Game over |
| `F3` | Registrar pontuação |
| `F4` | Gameplay |

## Protocolo de transmissão serial

A placa envia o estado do jogo em pacotes (*chunks*) de **51 bytes**: 43 bytes de dados e 8 bytes de fim de bloco (*break point*, `41 41 41 41 41 41 41 41`).

| Slice | Bytes | Conteúdo |
|-------|-------|----------|
| PT | 1 | Pontuação |
| G1 | 1 | Direção da nave (2 bits), vidas (3 bits), dificuldade (3 bits) |
| AS | 16 | Posições dos asteroides (4 bits para x e 4 bits para y) |
| DA | 4 | Direções dos asteroides (2 bits cada) |
| TI | 16 | Posições dos tiros |
| DT | 4 | Direções dos tiros |
| G2 | 1 | Flags: jogada especial, especial disponível, tiro, tiro disponível, fim das vidas |
| BP | 8 | Break point |

As direções são codificadas em 2 bits: `00` = RIGHT, `01` = LEFT, `10` = UP, `11` = DOWN.

A especificação completa está em [`docs/chunk_encoding.md`](docs/chunk_encoding.md).

## Licença

Distribuído sob a licença MIT. Veja [`LICENSE`](LICENSE).
