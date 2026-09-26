# ==============================================================================
# Makefile para Simulação e Debug Synopsys (VCS & Verdi)
# Projeto: Asynchronous FIFO (CDC - UCIe/D2D Readiness)
# ==============================================================================

# Definição dos caminhos
RTL_DIR = ../rtl
TB_DIR  = ../tb

# Lista de arquivos RTL (Arquitetura modular de Cummings)
RTL_SRC = $(RTL_DIR)/fifomem.sv \
          $(RTL_DIR)/sync_ptr.sv \
          $(RTL_DIR)/rptr_empty.sv \
          $(RTL_DIR)/wptr_full.sv \
          $(RTL_DIR)/async_fifo.sv

# Lista de arquivos do Testbench
TB_SRC  = $(TB_DIR)/async_fifo_if.sv \
          $(TB_DIR)/tb_top.sv

# Flags de compilação VCS
# -kdb: Gera a base para o Verdi
# -debug_access+all: Permite visibilidade total de sinais no dump FSDB/Verdi
VCS_FLAGS = -sverilog -full64 -kdb -debug_access+all \
            -timescale=1ns/1ps \
            +vcs+fsdbon \
            -l vcs_compile.log

seed = 1
run_opts =

.PHONY: default compile run test verdi clean help

default: test

# Compila RTL e TB
compile:
	vcs $(VCS_FLAGS) $(RTL_SRC) $(TB_SRC)

# Executa simulação com geração de log e seed configurável
run:
	./simv -l simv.log +ntb_random_seed=$(seed) $(run_opts)

# Compila e roda sequencialmente
test: compile run

# Abre o visualizador Verdi diretamente carregando o design e FSDB
verdi:
	verdi -dbdir simv.daidir -ssf nova.fsdb &

# Executa simulação com GUI interativa do Verdi
verdi_gui:
	./simv -gui=verdi -l simv_interactive.log +ntb_random_seed=$(seed) $(run_opts) &

# Limpeza de logs, binários intermediários e dumps
clean:
	rm -rf simv simv.daidir csrc *.log *.vpd *.fsdb verdiLog novas.* ucli.key *.dat vc_hdrs.h DVEfiles

help:
	@echo "=================================================================="
	@echo " Makefile Synopsys VCS/Verdi - Async FIFO                         "
	@echo "=================================================================="
	@echo " make test      : Compila e executa o teste em modo batch"
	@echo " make compile   : Apenas compila o RTL e TB com VCS"
	@echo " make run       : Executa o binário simv gerado"
	@echo " make verdi     : Abre a GUI do Verdi pós-simulação"
	@echo " make verdi_gui : Roda o simv de forma interativa no Verdi"
	@echo " make clean     : Remove arquivos temporários de compilação"
	@echo "=================================================================="
