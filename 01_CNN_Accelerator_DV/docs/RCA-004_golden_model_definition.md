# Relatório Técnico: Auditoria do Modelo de Referência e Validação do Novo Golden Model para Acelerador de Hardware (ASIC/FPGA)

**Projeto:** Acelerador de Hardware Dedicado para Detecção de Fibrilação Ventricular (TCC)  
**Alvo:** FPGA Xilinx Zynq-7020 (Plataforma PYNQ-Z2) e Tapeout ASIC  
**Data:** Março de 2025  

---

## 1. Sumário Executivo

Durante a fase de reprodução e validação do modelo de referência publicado no artigo (*IEEE Transactions on Instrumentation and Measurement, 2025*), identificou-se a impossibilidade de atingir as métricas relatadas de sensibilidade e acurácia (~99,99%) sob rigor clínico e metodológico. 

A investigação técnica revelou **falhas graves de integridade metodológica no artigo**, especificamente **Vazamento de Dados (*Data Leakage*)** decorrente da falta de isolamento estrito entre pacientes de treino e teste e do particionamento posterior à sobreposição (*overlap*) de janelas. 

Como contramedida, desenvolveu-se um **novo Golden Model em PyTorch**, estruturado sob validação inter-paciente (*Inter-Patient Validation*), com técnicas avançadas de generalização (*Focal Loss*, *Data Augmentation* temporal e *Weighted Random Sampling*). O modelo resultante atingiu **98,63% de Sensibilidade para Fibrilação Ventricular (FV)** em pacientes inéditos, estabelecendo-se como uma referência fidedigna, clinicamente segura e otimizada para síntese em hardware (HLS/RTL).

---

## 2. Auditoria do Artigo de Referência (IEEE TIM 2025)

O artigo propõe uma 1D-CNN para classificação ternária de ECG (VT/VF, Não-VT/VF e Ruído), alegando acurácia e F1-Score superiores a 99,98% com tempo de inferência de 1,93 ms em Raspberry Pi 4.

### 2.1. Causa Raiz da Irreprodutibilidade (Data Leakage)
A impossibilidade de replicar esses resultados decorre de dois vícios metodológicos presentes no artigo:

1. **Ausência de Isolamento por Paciente (*Patient-level Isolation*):**  
   Os autores admitem textualmente na Seção IV (Conclusão):  
   > *"Although patient-level isolation was not strictly enforced during testing, it was not followed during training."*  
   
   Ao particionar aleatoriamente janelas de ECG em 80/20 após juntar os registros, segmentos do mesmo paciente foram alocados simultaneamente nos conjuntos de treino e teste. Devido à natureza morfológica única do ECG de cada indivíduo, a rede neural memorizou a "assinatura do paciente" em vez de aprender a patologia da arritmia, gerando métricas artificialmente perfeitas (ilusão de desempenho).

2. **Vazamento Adicional por Janelamento com Sobreposição (*Overlap*):**  
   Na Seção II.B.1, relata-se o uso de 20% de *overlap*. A segmentação temporal contígua distribuída aleatoriamente entre treino e teste faz com que o modelo seja testado em amostras de 1 segundo que ele literalmente já processou no treinamento.

**Conclusão sobre o artigo:** O modelo do artigo não é clinicamente generalizável e falharia em campo. Utilizá-lo como especificação para um ASIC resultaria em um circuito integrado ineficaz no mundo real.

---

## 3. O Novo Golden Model (PyTorch)

Para guiar o desenvolvimento do RTL/ASIC, construiu-se uma arquitetura proprietária focada em hardware e avaliada sob condições clínicas reais.

### 3.1. Correções Metodológicas Implementadas
* **Separação Rígida por Registro/Paciente:** Pacientes do conjunto de teste (`106, 108, 115, 116`, `cu05, cu06, cu11, cu12`) **jamais** foram vistos pela rede durante o treinamento.
* **Função de Custo Adaptativa:** Uso de `Focal Loss (gamma=2.0)` em conjunto com `WeightedRandomSampler` para mitigar o severo desbalanceamento de classes em sinais biomédicos.
* **Aumento Dinâmico de Dados (*Data Augmentation*):** Injeção em tempo real de ruído gaussiano, *baseline wander*, variações de escala de amplitude e deslocamentos no tempo (*time-shifts*).

### 3.2. Arquitetura Adaptada para Hardware
A arquitetura foi desenhada especificamente para minimizar o consumo de área de silício (células padrão no ASIC e LUTs/DSPs na FPGA):
* **Camadas Convolucionais 1D:** Canais reduzidos ($C_1=6, C_2=12, C_3=19$) com kernels ímpares decrescentes ($11 \to 7 \to 5$).
* **Substituição de Pooling Dinâmico:** Utilização de `MaxPool1d` com passos fixos em substituição a pools adaptativos com divisão, facilitando a temporização do pipeline em Verilog.
* **Batch Normalization:** Treinado com `BatchNorm1d` prevendo **Batch Normalization Folding** durante a fase de síntese em RTL (fusão dos pesos $\gamma, \beta, \mu, \sigma$ diretamente nos pesos dos filtros convolucionais).

---

## 4. Resultados Experimentais

O treinamento foi conduzido por 15 épocas com mecanismo de *Early Stopping* e taxa de aprendizado dinâmica.

```text
==================================================
MÉTRICAS DE DESEMPENHO (TESTE EM PACIENTES INÉDITOS)
==================================================
Acurácia Global:           87,65%
Sensibilidade para FV:     98,63%  <-- [Métrica Crítica de Suporte à Vida]
Precisão para FV:          83,46%
Especificidade (Normal):   68,91%
F1-Score Macro:            87,89%
==================================================
