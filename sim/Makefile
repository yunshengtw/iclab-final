# CAD-Tools & shell-commands
VLOG	= ncverilog
LINT	= nLint
DC	= dc_shell
RM	= rm -rf

# Working Directories
HDL_DIR = ../hdl/
SYN_DIR = ../syn/
SIM_DIR	= ./

# Testbench and File
TESTBENCH_TOP = test_top.v
TESTBENCH     = test_$(basename $@).v

# Misc
# TEST_DEFINE_FILE = $(basename $@)_test_define.v
# DEFINE_FILE = ../hdl/define.v
TOP     = top
INCLIST = ../hdl/$(basename $@).f
NETLIST = ../syn/netlist/$(basename $@)_syn.v
GATE    = /usr/cad/synopsys/SAED32_EDK/lib/stdcell_hvt/verilog/saed32nm_hvt.tv
LIB      = -v /usr/cad/synopsys/SAED32_EDK/synthesis/vG-2012.06-SP5/dw/dw02/src_ver/DW02_prod_sum1.v
CELLVLOG = /usr/cadtool/cad/synopsys/SAED32_EDK/lib/stdcell_hvt/verilog/saed32nm_hvt.v


# arguments
VLOGARG	= +access+r
OPT      = +notimingcheck
SAVELOG  = | tee

# temp files
SIM_TEMP =  INCA_libs \
						verdiLog \
						nLintDB \
						nLintLog \
						sim_nLint \
						sim_pre \
						sim_gate \
						sim_post \
						*.log \
						novas.rc \
						novas.conf \
						*.key \
						*.X \
						*~

SYN_TEMP =  alib-52 \
						$(TOP) \
						netlist \
						report \
						*.svf \
						*.log \
						*~

# Targets
lint:
	rm -rf *.log nLintDB nLintLog sim_nLint
	mkdir sim_nLint
	nLint -f $(HDL_DIR)hdl.f -out nLint.rep
	mv nLintDB sim_nLint
	mv nLintLog sim_nLint
	cat nLint.rep | less

# nwave:
#	verdi & -nWave -f *.fsdb

presim:
	rm -rf *.log INCA_libs sim_pre
	mkdir sim_pre
	ncverilog -f sim_pre.f
	mv INCA_libs sim_pre
	mv *.log sim_pre
# cat ./sim_pre/ncverilog.log | less

gatesim:
	rm -rf *.log INCA_libs sim_gate
	mkdir sim_gate
	ncverilog -f sim_gate.f -warnmax 0
	mv INCA_libs sim_gate
	mv *.log sim_gate

postsim:
	rm -rf *.log INCA_libs sim_post
	mkdir sim_post
	ncverilog -f sim_post.f -warnmax 0
	mv INCA_libs sim_post
	mv *.log sim_post


syn:
	cd $(SYN_DIR) && nohup dc_shell -f synthesis.tcl | tee da.log

# clean working directories
simclean:
	rm -rf $(SIM_TEMP)

synclean:
	cd $(SYN_DIR) && $(RM) $(SYN_TEMP)

%.preicc:
	cp -i ../syn/netlist/$(basename $@)_syn.sdc ../icc/pre_layout/dc
	cp -i ../syn/netlist/$(basename $@)_syn.v ../icc/pre_layout/dc
	cp -i ../icc/pre_layout/dc/$(basename $@)_syn.sdc \
	../icc/pre_layout/design_data/CHIP_syn.sdc 
	cat ../icc/pre_layout/dc/$(basename $@)_syn.v \
	../icc/pre_layout/design_data/CHIP_syn_app.v \
	> ../icc/pre_layout/design_data/CHIP_syn.v
	cd ../icc/icc_run && rm -rf CHIP *.log *.txt 
# Now modify CHIP_syn.sdc "set_operating_conditions" and "create_clock"
# set_operating_conditions -max ss0p95v125c -min ff1p16vn40c
# #set_wire_load_mode enclosed
# #set_max_fanout 0.82 [current_design]
# #set_max_area 0
# #set_wire_load_selection_group predcaps

###########################################
%.verdi :
	cd verdi
	verdi -autoalias -logfile ./verdi_compiler.log \
	+incdir + $(INCDIR) $(TEST_DEFINE_FILE) \
	$(DEFINE_FILE) $(LIB) $(TESTBENCH) &

%.lint:
	make simclean
	rm -rf sim_nLint
	mkdir sim_nLint
	nLint $(HDL_DIR)$(basename $@).v -out nLint.rep
	mv nLintDB sim_nLint
	mv nLintLog sim_nLint
	cat nLint.rep | less

%.com :
	ncverilog -c $(DEFINE_FILE) $(TESTBENCH) -F $(INCLIST)

%.sim :
	make simclean
	mkdir sim_pre
	ncverilog $(DEFINE_FILE) $(TESTBENCH) $(LIB) -F $(INCLIST)
	mv INCA_libs sim_pre
	mv *.log sim_pre

%.posim :
	make simclean
	mkdir sim_post
	ncverilog $(DEFINE_FILE) $(TESTBENCH) $(LIB) +vg $(GATE) $(NETLIST) +define+SDF
	mv INCA_libs sim_post
	mv *.log sim_post

# %.vcs :
#   make clean
#   vcs -R -full64 + v2k $(OPT) +define + $(DEFINE_FILE) +incdir + $(INCDIR) $(TEST_DEFINE_FILE)\
#   $(DEFINE_FILE) $(LIB) $(TESTBENCH) \
#   -l vcs.log -debug_pp +vcs_d + vpi + memcbk -P