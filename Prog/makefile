#****************************************************************************************
#|  Description: Makefile for GNU ARM Embedded toolchain.
#|    File Name: makefile
#|
#|---------------------------------------------------------------------------------------
#|                          C O P Y R I G H T
#|---------------------------------------------------------------------------------------
#|   Copyright (c) 2017  by Feaser    http://www.feaser.com    All rights reserved
#|
#|---------------------------------------------------------------------------------------
#|                            L I C E N S E
#|---------------------------------------------------------------------------------------
#| This file is part of OpenBLT. OpenBLT is free software: you can redistribute it and/or
#| modify it under the terms of the GNU General Public License as published by the Free
#| Software Foundation, either version 3 of the License, or (at your option) any later
#| version.
#|
#| OpenBLT is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
#| without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
#| PURPOSE. See the GNU General Public License for more details.
#|
#| You have received a copy of the GNU General Public License along with OpenBLT. It 
#| should be located in ".\Doc\license.html". If not, contact Feaser to obtain a copy.
#|
#****************************************************************************************
SHELL = sh

#|--------------------------------------------------------------------------------------|
#| Configure project name                                                               |
#|--------------------------------------------------------------------------------------|
PROJ_NAME=demoprog_stm32f103


#|--------------------------------------------------------------------------------------|
#| Configure tool path                                                                  |
#|--------------------------------------------------------------------------------------|
# Configure the path to where the arm-none-eabi-gcc program is located. If the program
# is available on the path, then the TOOL_PATH variable can be left empty.
# Make sure to add a fordward slash at the end. Note that on Windows it should be in the
# 8.3 short pathname format with forward slashes. To obtain the pathname in the 8.3
# format, open the directory in the Windows command prompt and run the following command:
#  cmd /c for %A in ("%cd%") do @echo %~sA 
#TOOL_PATH=/opt/gcc-arm-none-eabi-8-2018-q4-major/bin/
#TOOL_PATH=C:/PROGRA~2/GNUTOO~1/82018-~1/bin/
TOOL_PATH=/opt/gcc-arm-none-eabi-10.3-2021.10/bin/

#|--------------------------------------------------------------------------------------|
#| Collect project files                                                                |
#|--------------------------------------------------------------------------------------|
# Recursive wildcard function implementation. Example usages:
#   $(call rwildcard, , *.c *.h)   
#     --> Returns all *.c and *.h files in the current directory and below
#   $(call rwildcard, /lib/, *.c)
#     --> Returns all *.c files in the /lib directory and below
rwildcard = $(strip $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2) $(filter $(subst *,%,$2),$d)))

# Collect all application files in the current directory and its subdirectories
PROJ_FILES = $(call rwildcard, , *.c *.h *.s)


#|--------------------------------------------------------------------------------------|
#| Toolchain binaries                                                                   |
#|--------------------------------------------------------------------------------------|
RM = rm
CC = $(TOOL_PATH)arm-none-eabi-gcc
LN = $(TOOL_PATH)arm-none-eabi-gcc
OC = $(TOOL_PATH)arm-none-eabi-objcopy
OD = $(TOOL_PATH)arm-none-eabi-objdump
AS = $(TOOL_PATH)arm-none-eabi-gcc
SZ = $(TOOL_PATH)arm-none-eabi-size


#|--------------------------------------------------------------------------------------|
#| Filter project files
#|--------------------------------------------------------------------------------------|
PROJ_ASRCS  = $(filter %.s,$(foreach file,$(PROJ_FILES),$(notdir $(file))))
PROJ_CSRCS  = $(filter %.c,$(foreach file,$(PROJ_FILES),$(notdir $(file))))


#|--------------------------------------------------------------------------------------|
#| Set important path variables                                                         |
#|--------------------------------------------------------------------------------------|
VPATH    = $(foreach path,$(sort $(foreach file,$(PROJ_FILES),$(dir $(file)))) $(subst \,/,$(OBJ_PATH)),$(path) :)
OBJ_PATH = obj
BIN_PATH = bin
INC_PATH = $(patsubst %/,%,$(patsubst %,-I%,$(sort $(foreach file,$(filter %.h,$(PROJ_FILES)),$(dir $(file))))))
LIB_PATH  = 


#|--------------------------------------------------------------------------------------|
#| Options for toolchain binaries                                                       |
#|--------------------------------------------------------------------------------------|
STDFLAGS    = -mcpu=cortex-m3 -mthumb -std=gnu11 -fstack-usage -Wall -specs=nano.specs
STDFLAGS   += -fdata-sections -ffunction-sections -Wall -g -Wno-strict-aliasing
OPTFLAGS    = -Og
DEPFLAGS  = -MT $@ -MMD -MP -MF $(OBJ_PATH)/$*.d
CFLAGS      = $(STDFLAGS) $(OPTFLAGS)
CFLAGS     += -DUSE_HAL_DRIVER -DSTM32F103xB
CFLAGS     += -D__weak="__attribute__((weak))" -D__packed="__attribute__((__packed__))"
CFLAGS     += $(INC_PATH)
AFLAGS      = $(CFLAGS)
LFLAGS      = $(STDFLAGS) $(OPTFLAGS)
LFLAGS     += -T"STM32F103C8TX_FLASH.ld" -Wl,-Map=$(BIN_PATH)/$(PROJ_NAME).map
LFLAGS     += -Wl,--gc-sections $(LIB_PATH)
OFLAGS      = -O srec
ODFLAGS     = -x
SZFLAGS     = -B -d
RMFLAGS     = -f


#|--------------------------------------------------------------------------------------|
#| Specify library files                                                                |
#|--------------------------------------------------------------------------------------|
LIBS = 


#|--------------------------------------------------------------------------------------|
#| Define targets                                                                       |
#|--------------------------------------------------------------------------------------|
AOBJS = $(patsubst %.s,%.o,$(PROJ_ASRCS))
COBJS = $(patsubst %.c,%.o,$(PROJ_CSRCS))


#|--------------------------------------------------------------------------------------|
#| Make ALL                                                                             |
#|--------------------------------------------------------------------------------------|
.PHONY: all
all: $(BIN_PATH)/$(PROJ_NAME).srec

$(BIN_PATH)/$(PROJ_NAME).srec : $(BIN_PATH)/$(PROJ_NAME).elf
	@$(OC) $< $(OFLAGS) $@
	@$(OD) $(ODFLAGS) $< > $(BIN_PATH)/$(PROJ_NAME).map
	@echo +++ Summary of memory consumption:
	@$(SZ) $(SZFLAGS) $<
	@echo +++ Build complete [$(notdir $@)]

$(BIN_PATH)/$(PROJ_NAME).elf : $(AOBJS) $(COBJS) 
	@echo +++ Linking [$(notdir $@)]
	@$(LN) $(LFLAGS) -o $@ $(patsubst %.o,$(OBJ_PATH)/%.o,$(^F)) $(LIBS)


#|--------------------------------------------------------------------------------------|
#| Compile and assemble                                                                 |
#|--------------------------------------------------------------------------------------|
$(AOBJS): %.o: %.s
	@echo +++ Assembling [$(notdir $<)]
	@$(AS) $(AFLAGS) -c $< -o $(OBJ_PATH)/$(@F)

$(COBJS): %.o: %.c $(OBJ_PATH)/%.d
	@echo +++ Compiling [$(notdir $<)]
	@$(CC) $(DEPFLAGS) $(CFLAGS) -c $< -o $(OBJ_PATH)/$(@F)


#|--------------------------------------------------------------------------------------|
#| Make CLEAN                                                                           |
#|--------------------------------------------------------------------------------------|
.PHONY: clean
clean: 
	@echo +++ Cleaning build environment
	@$(RM) $(RMFLAGS) $(foreach file,$(AOBJS),$(OBJ_PATH)/$(file))
	@$(RM) $(RMFLAGS) $(foreach file,$(COBJS),$(OBJ_PATH)/$(file))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.lst,$(foreach file,$(AOBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.lst,$(foreach file,$(COBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.d,$(foreach file,$(COBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(BIN_PATH)/$(PROJ_NAME).elf $(BIN_PATH)/$(PROJ_NAME).map
	@$(RM) $(RMFLAGS) $(BIN_PATH)/$(PROJ_NAME).srec
	@echo +++ Clean complete

	
#|--------------------------------------------------------------------------------------|
#| Dependency generation                                                                |
#|--------------------------------------------------------------------------------------|
DEPFILES := $(PROJ_CSRCS:%.c=$(OBJ_PATH)/%.d)
$(DEPFILES):
include $(wildcard $(DEPFILES))	

