#
# Copyright (C) 2023, Advanced Micro Devices, Inc. All rights reserved.
# SPDX-License-Identifier: MIT
#
# Author: Mark Rollins

ECHO                               = @echo

export TARGET                      = hw_emu

RELEASE=2024.1

TOP_DIR                   ?= $(shell readlink -f .)
PLATFORM_NAME              = xilinx_vck190_base_202410_1
PLATFORM_PATH              = ${TOP_DIR}/platform/platform_repo

export PLATFORM            = ${PLATFORM_PATH}/${PLATFORM_NAME}/export/${PLATFORM_NAME}/${PLATFORM_NAME}.xpfm
export SYSROOT             = ${COMMON_IMAGE_VERSAL}/sysroots/cortexa72-cortexa53-xilinx-linux
export KERNEL_IMAGE        = ${COMMON_IMAGE_VERSAL}/Image
export ROOTFS              = ${COMMON_IMAGE_VERSAL}/rootfs.ext4
export PREBUILT_LINUX_PATH = ${COMMON_IMAGE_VERSAL}

##############
# Build Flow #
##############
.PHONY: clean help

help::
	$(ECHO) "Makefile Usage:"
	$(ECHO) "  make all"
	$(ECHO) "      Command to generate everything for this design"
	$(ECHO) ""
	$(ECHO) "  make clean"
	$(ECHO) "      Command to remove all the generated files."

all:	channelizer

channelizer:
	$(MAKE) all -C platform
	$(MAKE) all -C aie/m16_ssr8
	$(MAKE) all -C hls/m16_ssr8_permute_fb_i
	$(MAKE) all -C hls/m16_ssr8_permute_fb_o
	$(MAKE) all -C hls/m16_ssr8_cyclic_shift
	$(MAKE) all -C hls/dma_stream_src
	$(MAKE) all -C hls/dma_stream_snk
	$(MAKE) all -C ps_apps/hw_emu
	$(MAKE) all -C vitis/final

power:
	$(MAKE) vivado_xpe -C vitis/final
	$(MAKE) vitis_xpe -C aie/m16_ssr8

clean:
	$(MAKE) clean -C platform
	$(MAKE) clean -C aie/m16_ssr8
	$(MAKE) clean -C aie/m16_ssr8_dft
	$(MAKE) clean -C aie/m16_ssr8_filterbank
	$(MAKE) clean -C hls/m16_ssr8_permute_fb_i
	$(MAKE) clean -C hls/m16_ssr8_permute_fb_o
	$(MAKE) clean -C hls/m16_ssr8_cyclic_shift
	$(MAKE) clean -C hls/dma_stream_src
	$(MAKE) clean -C hls/dma_stream_snk
	$(MAKE) clean -C ps_apps/hw_emu
	$(MAKE) clean -C vitis/final
	@/bin/rm -rf .crashReporter*


