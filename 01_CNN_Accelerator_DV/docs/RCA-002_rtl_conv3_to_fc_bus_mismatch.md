# Relatório de Verificação e Arquitetura: Gargalo de Cobertura e Serialização da Camada FC

**Data:** $(data_de_hoje)  
**Módulos Impactados:** `cnn_top.sv`, `gen_bn_pool3`, `fully_connected_layer.sv`  
**Ferramentas Utilizadas:** Cadence IMC (Integrated Metrics Center), SimVision / Xcelium  

---

## 1. Contexto e Problema Identificado

Durante a análise das métricas de cobertura de código (*Code Coverage*) através do **Cadence IMC**, observou-se uma discrepância nos índices de cobertura das instâncias da terceira camada convolucional (`gen_bn_pool3`):

* **Média Geral de Cobertura:** Abaixo do esperado (~30% a 48%).
* **Toggle Coverage:** Atingindo valores em torno de **1.11%**.
* **Comportamento Incomum:** As instâncias de `0` a `18` da terceira camada convolucional apresentavam grande inatividade em seus barramentos de sinal, indicando ausência de estímulos ou estagnação lógica (*dead logic*).

---

## 2. Diagnóstico da Causa Raiz

Aprofundando a inspeção no RTL (`cnn_top.sv`) e no módulo do classificador (`fully_connected_layer.sv`), constatou-se um **descasamento temporal e dimensional de dados** (*Data Width / Rate Mismatch*):

1. **Assincronia Dimensional / Estrutural:**
   - A saída da Camada 3 (`c3_avg`) gera **19 canais simultâneos** de 16 bits cada, resultando em um barramento plano (*flat*) de **304 bits** em uma única transação paralela.
   - A camada subsequente, `fully_connected_layer`, possui porta de entrada `s_axis_tdata` de apenas **16 bits**, pois foi projetada para receber os dados de forma serializada (uma feature por ciclo de clock, acumulando os pesos via `feature_idx` até atingir 19 ciclos).

2. **Truncamento Indevido no Topo:**
   - No `cnn_top.sv`, a conexão foi feita diretamente como:
     ```systemverilog
     .s_axis_tdata(c3_avg[15:0]) // Envia apenas o Canal 0 repetidamente
     ```
   - Isso acarretou o **descarte completo dos canais 1 a 18**, fazendo com que o simulador/sintetizador tratasse esses canais como sinais ociosos, derrubando a cobertura de Toggle da Camada 3.
   - Além disso, a rede calculava as predições de forma incorreta, avaliando 19 vezes a mesma feature em vez de usar as 19 características distintas da convolução.

---

## 3. Solução Proposta: Implementação de Módulo Serializador / Buffer

Para reestabelecer a integridade do fluxo de dados e recuperar a cobertura das instâncias da CNN, determinou-se a criação de um bloco intermediário entre a Convolução 3 e a Fully Connected:

* **Conversor Paralelo-Serial (Unpacker):**
  - Trava (*latch*) o vetor de 304 bits vindo da Camada 3 quando `c3_avg_v` estiver alto.
  - Distribui em fatias de 16 bits para a `fully_connected_layer`, um canal por ciclo ao longo de 19 ciclos de clock.
* **Avaliação de Buffer / FIFO AXI-Stream:**
  - Estuda-se o acréscimo de uma pequena FIFO/Buffer assimétrico (Write: 304 bits, Read: 16 bits) para desacoplar a taxa de transferência (*throughput*) e absorver o *backpressure* (`tready`), evitando travamentos no pipeline da CNN.

---

## 4. Próximos Passos
- [ ] Implementar o módulo RTL do Serializador / Buffer AXI-Stream.
- [ ] Atualizar o `cnn_top.sv` com as novas conexões de controle de fluxo (`tvalid`/`tready`).
- [ ] Criar nova sequência UVM com estímulos aleatórios (`fc_random_seq`) para forçar o Toggle Coverage dos bits mais significativos.
- [ ] Re-executar a simulação e verificar o salto na cobertura via Cadence IMC.
