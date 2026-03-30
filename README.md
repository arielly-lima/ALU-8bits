# ALU de 8 Bits — Digital Simulator

> Unidade Lógica e Aritmética com seleção de operação por enable e decodificador 3:8

---

## 1. Visão Geral

Este projeto implementa uma **Unidade Lógica e Aritmética (ALU) de 8 bits** no simulador [Digital](https://github.com/hneemann/Digital). A ALU executa 7 operações distintas sobre os operandos **AC** e **N**, com seleção via decodificador 3:8 e habilitação individual de cada bloco operacional (enable).

O projeto é composto por subcircuitos independentes em formato `.dig`, integrados em um circuito principal que gerencia a seleção e a composição da saída final.

---

## 2. Operações Suportadas

| Operação | Expressão | Entradas | Saídas |
|---|---|---|---|
| Soma | AC + N | AC, N (8b) | AC (8b) |
| Subtração | AC − N | AC, N (8b) | AC (8b) |
| Multiplicação | AC × N | AC, N (8b) | AC (8 LSB), MQ (8 MSB) |
| Divisão | AC ÷ N | AC, N (8b) | AC (Resto), MQ (Quociente) |
| Shift Lógico | shift(AC) | AC (8b) | AC (8b) |
| NAND | AC NAND N | AC, N (8b) | AC (8b) |
| XOR | AC XOR N | AC, N (8b) | AC (8b) |

> **AC** = Acumulador (registrador principal de 8 bits)  
> **MQ** = Registrador auxiliar de 8 bits (usado em multiplicação e divisão)

---

## 3. Arquitetura do Circuito

### 3.1 Entradas Globais

| Sinal | Largura | Descrição |
|---|---|---|
| `AC[7..0]` | 8 bits | Acumulador — operando principal |
| `N[7..0]` | 8 bits | Segundo operando |
| `S[2..0]` | 3 bits | Código de seleção da operação |

### 3.2 Decodificador 3:8

Os 3 bits de seleção ativam exatamente **um** sinal de enable por vez:

| S2 | S1 | S0 | Enable | Operação |
|---|---|---|---|---|
| 0 | 0 | 0 | EN0 | Soma |
| 0 | 0 | 1 | EN1 | Subtração |
| 0 | 1 | 0 | EN2 | Multiplicação |
| 0 | 1 | 1 | EN3 | Divisão |
| 1 | 0 | 0 | EN4 | Shift Lógico |
| 1 | 0 | 1 | EN5 | NAND |
| 1 | 1 | 0 | EN6 | XOR |
| 1 | 1 | 1 | — | *(reservado)* |

No Digital: **Componentes → Plexers → Decoder** (3 entradas, 8 saídas).

### 3.3 Blocos Operacionais com Enable

Cada operação reside em um subcircuito `.dig` separado. Todos os blocos recebem `AC`, `N` e um pino `EN`. Quando `EN = 0`, a saída é mascarada para `00000000` via portas AND de 8 bits:

```
AC, N ──► [ Lógica da operação ] ──► resultado[7..0]
                                            │
EN    ──► [ Replicator 1→8 ]    ──► [ AND 8b ] ──► saída[7..0]
```

> O componente **Replicator** (Componentes → Wiring) expande EN de 1 bit para 8 bits, permitindo o AND bit a bit com o resultado.

### 3.4 Composição da Saída

Como apenas um EN está ativo por vez, todas as saídas de 8 bits são combinadas por uma **porta OR de 8 bits com 7 entradas**. O resultado é encaminhado para:

- **AC** — soma, subtração, shift, NAND, XOR, resto da divisão, 8 LSB da multiplicação
- **MQ** — 8 MSB da multiplicação, quociente da divisão

---

## 4. Subcircuitos

### 4.1 `divisor-4bits.dig`

Implementa o algoritmo de **divisão por restauração**, expandido de 3 para **4 bits no dividendo**:

- **Dividendo:** 4 bits (AC)
- **Divisor:** 3 bits (N)
- **4 estágios** de processamento, cada um com 4 PUs (Processing Units)
- **Saídas:** `R_F` (Resto Final) → AC · `Q0, Q1, Q2` (Quociente) → MQ

A expansão de 3 para 4 bits foi feita por edição direta do XML do arquivo `.dig`:

1. Cópia do Stage 2 com deslocamento **+480 unidades** no eixo Y
2. Replicação de todos os wires internos do Stage 2 para o Stage 3
3. Extensão dos barramentos verticais (`x = 1480, 1520, 1560`) até o novo estágio
4. Adição dos wires de conexão Stage 2 → Stage 3 (espelhando o padrão Stage 1 → Stage 2)
5. Remoção de wires duplicados gerados pela cópia

### 4.2 Demais Subcircuitos

Os blocos de soma, subtração, multiplicação, shift, NAND e XOR devem seguir o mesmo padrão de interface:

- **Entradas:** `AC[7..0]`, `N[7..0]`, `EN`
- **Saída:** `resultado[7..0]` (mascarado por AND quando `EN = 0`)

Blocos já existentes podem ser adaptados adicionando a porta AND de mascaramento e o Replicator do sinal EN.

---

## 5. Como Usar no Digital

1. Abra o arquivo principal `ula_8bits.dig` no Digital
2. Certifique-se de que todos os subcircuitos `.dig` estejam na **mesma pasta**
3. Configure `AC[7..0]` e `N[7..0]` com os operandos desejados
4. Configure `S[2..0]` conforme a tabela de seleção (Seção 3.2)
5. Leia o resultado em **AC** e, se necessário, em **MQ**

---

## 6. Estrutura de Arquivos

```
projeto/
├── ula_8bits.dig          # Circuito principal (a criar)
├── divisor-4bits.dig      # Bloco de divisão (4 bits) ✅
├── soma.dig               # Bloco de soma (a adaptar)
├── subtracao.dig          # Bloco de subtração (a adaptar)
├── multiplicacao.dig      # Bloco de multiplicação (a adaptar)
├── shift.dig              # Bloco de shift lógico (a adaptar)
├── nand.dig               # Bloco NAND (a adaptar)
├── xor.dig                # Bloco XOR (a adaptar)
└── README.md              # Este documento
```

---

## 7. Decisões de Projeto

**Por que enable em vez de MUX?**  
A abordagem com enable e OR final é mais modular: cada bloco é independente e pode ser desenvolvido, testado e substituído sem alterar o circuito principal. Um MUX exigiria ajustar a largura do barramento de entrada sempre que uma operação fosse adicionada.

**Por que 3 bits de seleção e não 7 chaves?**  
Três bits permitem selecionar até 8 operações com apenas 3 entradas, reduzindo o número de pinos no circuito principal. O decodificador 3:8 é um componente nativo do Digital.

**Por que o divisor usa 4 bits no dividendo e 3 no divisor?**  
A estrutura original do circuito possuía 3 estágios com 4 PUs cada, operando sobre um divisor de 3 bits. Adicionar um quarto estágio estendeu o dividendo para 4 bits mantendo a largura do divisor. Para um divisor de 4 bits seria necessário adicionar um quinto PU por estágio — uma mudança estrutural maior.

---

*Projeto ULA 8 bits — Digital Simulator*
