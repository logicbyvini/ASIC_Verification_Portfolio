Abaixo está o conteúdo formatado em bloco único pronto para você copiar e colar:

# BUG-001: Ausência de Cobertura de Backpressure (Implicit Else) na Interface AXI-Stream

| Metadado | Detalhe |
| :--- | :--- |
| **ID do Registro** | BUG-001 |
| **Módulo Impactado** | `rtl/conv_layer.sv` (Instâncias: `conv1`, `conv2`, `conv3`) |
| **Ambiente de Teste** | UVM Testbench (`uvm_tb/`) |
| **Ferramenta de Diagnóstico** | Cadence IMC (Integrated Metrics Center) / Xcelium |
| **Métrica Afetada** | Code Coverage: Block Coverage / Branch Coverage |
| **Severidade** | Alta (Risco de congelamento ou perda de dados em silício) |
| **Status** | Causa-Raiz Identificada / Em Correção |

---

## 1. Descrição do Problema

Durante a análise de cobertura de código (*Code Coverage*) executada no **Cadence IMC**, identificou-se um furo de cobertura crítico na máquina de estados finitos (FSM) do controlador da camada convolucional (`conv_layer.sv`), especificamente na transição do estado `DONE`.

Trecho do código analisado:
```systemverilog
91   DONE: begin
92       m_axis_tvalid = 1'b1;
93       if (m_axis_tready) next_state = IDLE;
94   end
```

O analisador do compilador inferiu um bloco condicional implícito (**Block 30: implicit else**) referente à linha 93, o qual obteve **0% de cobertura** durante as regressões funcionais.

---

## 2. Evidências Coletadas (Cadence IMC)

### 2.1 Análise de Bloco na Instância `cnn_top.conv1`
O estado `DONE` foi atingido (Block 28, score 1) e a condição verdadeira (`m_axis_tready == 1'b1`) foi exercitada (Block 29, score 1). Entretanto, o escape implícito da condicional permaneceu sem nenhum estímulo (Block 30, score 0).

![Cadence IMC - Implicit Else Failure](https://github.com/user-attachments/assets/9ea13564-f3c4-45bc-9f35-1d45ee412bb0)

### 2.2 Propagação Multi-Instância (`conv1`, `conv2`, `conv3`)
A análise hierárquica demonstrou que o comportamento se repete em todos os núcleos convolucionais do pipeline, apontando que a deficiência não é um caso isolado de roteamento interno, mas sim uma deficiência sistêmica do ambiente de estímulos.

![Cadence IMC - Multi-instance Coverage Analysis](https://github.com/user-attachments/assets/712bc4df-c4c7-45ff-9a37-5f876292f38e)

---

## 3. Análise de Causa-Raiz (Root Cause Analysis - RCA)

1. **Hipótese de Receptor Ideal no Testbench:** O driver/receptor AXI-Stream instanciado no ambiente UVM foi implementado mantendo o sinal `m_axis_tready` permanentemente em nível lógico alto (`1'b1`).
2. **Ausência de Estresse de Barramento:** O design sob teste (DUT) nunca operou sob condição de retenção de fluxo (*backpressure*). Consequentemente, o DUT jamais permaneceu no estado `DONE` sob a condição `m_axis_tready == 1'b0`.
3. **Mapeamento pelo EDA:** Na ausência de uma cláusula explícita `else` para a linha 93, o sintetizador e o simulador da Cadence geram uma ramificação lógica implícita para manter o estado atual caso a condição seja falsa. Essa ramificação nunca foi exercitada.

---

## 4. Avaliação de Risco para Tape-out

* **Comportamento Não Validado:** Em silício, caso o circuito receptor (e.g., DMA, FIFO de saída ou barramento externo) desative o sinal `tready` por contenção, a FSM deve sustentar o dado estável e reter `m_axis_tvalid = 1'b1` sem corrupção de estado.
* **Impacto:** Assumir receptor ideal durante a verificação mascara *deadlocks* em potencial e viola o protocolo AMBA AXI-Stream, inviabilizando a garantia de conformidade para fabricação.

---

## 5. Ação Corretiva Proposta

Rejeita-se o uso de exclusão de cobertura (*waiver* via `.vRefine`), optando-se pela correção direta no ambiente de verificação:

1. **Injeção de Backpressure Aleatório:** Modificar o UVM Sequence/Driver responsável por consumir o fluxo AXI-Stream para aplicar quedas periódicas e pseudoaleatórias no sinal `tready`.
   ```systemverilog
   // Controle de atraso reativo no driver escravo UVM
   constraint c_axi_backpressure {
       tready_delay dist { 0 := 70, [1:4] := 30 };
   }
   ```
2. **Critério de Aceitação:** Fechamento de 100% de cobertura no Block 30 (transição para verde no IMC) em todas as instâncias convolucionais após reexecução da regressão.

---

## 6. Apêndice: Nota Técnica de Operação do Xcelium (`*E,NOSTUP`)

Durante a tentativa de recarga da base de cobertura, o simulador retornou:
```text
xrun: *E,NOSTUP: A problem was detected in the setup for simulation. 
Simulation can be done only after successfully completing design file parsing and elaboration.
```
* **Causa:** O comando `xrun -R` (restart/run from snapshot) foi executado após uma falha anterior de compilação/elaboração. Quando a fase de análise sintática falha, nenhum snapshot executável válido é gerado na base `xcelium.d/`.
* **Resolução:** Garantir a resolução integral dos erros sintáticos antes de invocar a etapa de simulação.
```
