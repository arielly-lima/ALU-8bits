# ALU de 8 Bits — Digital Simulator
**Unidade Lógica e Aritmética com Seleção por Barramento Tri-state e Decodificador 3:8**

## 1. Visão Geral
Este projeto implementa uma Unidade Lógica e Aritmética (ULA) de 8 bits desenvolvida no simulador **Digital**. A principal característica desta arquitetura é o uso de um **Barramento de Dados Compartilhado**, onde a seleção da operação ativa é feita através de um decodificador que habilita **Drivers Tri-state** individuais, evitando conflitos elétricos e garantindo modularidade.

## 2. Operações Suportadas
A ULA processa dois operandos de 8 bits (**AC** e **N**) e disponibiliza o resultado em um barramento comum.

| Opcode | Operação | Expressão | Descrição |
| :--- | :--- | :--- | :--- |
| `000` | **Soma** | $AC + N$ | Soma aritmética de 8 bits. |
| `001` | **Subtração** | $AC - N$ | Subtração via complemento de 2 (Somador com $Cin=1$ e $N$ invertido). |
| `010` | **Divisão** | $AC \div N$ | Quociente de 8 bits (Saída Q do divisor combinacional). |
| `011` | **NAND** | $AC \text{ NAND } N$ | Operação lógica bit a bit. |
| `100` | **XOR** | $AC \text{ XOR } N$ | Operação lógica bit a bit (OU Exclusivo). |
| `101` | **Shift Left** | $AC \ll 1$ | Deslocamento lógico para a esquerda (multiplicação por 2). |

## 3. Arquitetura do Circuito

### 3.1 Seleção de Operação (Opcode)
A escolha da função é feita por uma entrada de **3 bits**, conectada a um **Decodificador 3:8**. Este componente garante que apenas uma linha de controle (*Enable*) esteja ativa por vez, traduzindo o endereço binário em um sinal físico para o Driver correspondente.

### 3.2 Barramento Tri-state (Drivers)
Em vez de um Multiplexador (MUX) tradicional, utilizamos **Drivers Tri-state de 8 bits**. 
* Cada bloco operacional tem sua saída conectada a um Driver dedicado.
* O pino de controle (*Enable*) do Driver é ligado à saída correspondente do decodificador.
* Quando um bloco não está selecionado, sua saída entra em **Alta Impedância (Z)**, permitindo que múltiplos circuitos compartilhem o mesmo barramento sem causar curto-circuito (fio vermelho).

### 3.3 Entradas Compartilhadas
As entradas **AC** (Acumulador) e **N** são distribuídas para todos os blocos operacionais simultaneamente através de **Tunnels** (Túneis). Isso reduz a poluição visual do diagrama e simula o comportamento de um barramento de endereços/dados real.

## 4. Detalhes dos Subcircuitos

### 4.1 Divisor Combinacional (8 bits)
Utiliza uma estrutura de estágios em cascata (escada) para suportar a divisão de números de 8 bits.
* **Entrada:** Dividendo (AC) e Divisor (N).
* **Saída Q:** Quociente (encaminhado ao barramento via Driver).
* **Saída R:** Resto (visualizado de forma independente para depuração).

### 4.2 Shift Lógico (Esquerda)
Implementado via **Splitters**, realizando o deslocamento físico dos fios:
* O bit $A_n$ é mapeado para a posição $n+1$.
* O bit 0 recebe uma constante lógica `0`.
* O bit 7 (MSB) é extraído como **Cout** (Carry Out).

## 5. Como Utilizar
1. Abra o arquivo principal `ALU-completa.dig` no simulador Digital.
2. Certifique-se de que todos os subcircuitos (`.dig`) estejam na mesma pasta.
3. Configure os valores de entrada nos componentes **AC** e **N** (8 bits).
4. Altere o seletor de **Opcode** (3 bits) para observar a mudança de resultado no barramento.
5. O resultado final pode ser lido no **Hex Display** ou através de uma **Probe** decimal.

## 6. Decisões de Projeto
* **Modularidade:** Cada operação é um subcircuito independente, facilitando a manutenção.
* **Fidelidade à Hardware Real:** O uso de Drivers Tri-state e Decodificadores aproxima o projeto da arquitetura interna de processadores comerciais (como o 8085 ou 6502).
* **Escalabilidade:** Para adicionar uma 7ª ou 8ª operação, basta conectar o novo bloco a um novo Driver e utilizar as saídas 6 ou 7 do decodificador que já estão disponíveis.

---
*Projeto desenvolvido por Maria Arielly Lima - 2026*
