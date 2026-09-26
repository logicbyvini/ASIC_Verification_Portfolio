# Bug Report 01: Ghost Transaction no UVM Driver AXI4-Stream

## 1. Descrição do Problema
Durante o teste isolado da `fully_connected_layer` (que requer 19 features de entrada para calcular 3 logits de saída), o ambiente UVM finalizou a simulação com sucesso (`0 UVM_ERROR`), mas o Scoreboard não realizou a predição e comparação matemática.
Ao analisar os logs, foi constatado que o Monitor UVM capturou apenas **18 transações**, deixando a máquina de estados (FSM) do RTL Verilog travada aguardando a 19ª amostra.

## 2. Análise de Causa Raiz (Root Cause Analysis - RCA)
O erro ocorreu devido a um problema de sincronismo na injeção de dados do Driver Ativo (`fc_driver.sv`).
- O `uvm_test` iniciou a sequência em um tempo fixo (`#50ns`).
- O UVM Driver recebeu o pacote e imediatamente forçou os pinos virtuais (`tvalid <= 1`) de forma assíncrona.
- Na borda de subida do clock seguinte (`55ns`), o driver completou o handshake e abaixou o `tvalid`. 
- Como o RTL e o Monitor são síncronos e amostram os dados logo após a borda do clock, o sinal `tvalid` piscou muito rápido no barramento físico, resultando na perda da primeira amostra (Ghost Transaction).

## 3. Solução Implementada (Fix)
Para alinhar o UVM Driver ao domínio de clock do RTL e respeitar o protocolo AXI4-Stream, foi adicionado um alinhamento explícito ao *clocking block* antes do início da transação na tarefa `run_phase` do Driver.

**Código Corrigido (`fc_driver.sv`):**
```systemverilog
virtual task run_phase(uvm_phase phase);
    vif.cb.tvalid <= 1'b0;
    vif.cb.tdata  <= '0; 
    
    // FIX: Alinhamento síncrono obrigatório antes de iniciar a injeção
    @(vif.cb); 

    forever begin
        seq_item_port.get_next_item(req);
        repeat(req.delay_cycles) @(vif.cb);
        
        vif.cb.tdata  <= req.tdata;
        vif.cb.tvalid <= 1'b1;

        do begin
            @(vif.cb);
        end while (vif.cb.tready !== 1'b1);

        vif.cb.tvalid <= 1'b0;
        vif.cb.tdata  <= 'x; 
        seq_item_port.item_done();
    end
endtask
```
## 4. Resultado da Validação
Após a correção, o Monitor registrou perfeitamente as 19 transações. O DUT realizou o cálculo das 3 classes e o Preditor Matemático (Golden Model local do Scoreboard) confirmou o Match bit-a-bit com sucesso absoluto.
