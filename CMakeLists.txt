# Copyright (c) 2024 LuckkMaker
# All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

if(${CONFIG_OMNI_FAMILY} STREQUAL "stm32f1")
    set(THREADX_ARCH "cortex_m3" CACHE STRING "ThreadX architecture" FORCE)
    set(THREADX_TOOLCHAIN "gnu" CACHE STRING "ThreadX toolchain" FORCE)
elseif(${CONFIG_OMNI_FAMILY} STREQUAL "stm32f4" OR ${CONFIG_OMNI_FAMILY} STREQUAL "apm32f4")
    set(THREADX_ARCH "cortex_m4" CACHE STRING "ThreadX architecture" FORCE)
    set(THREADX_TOOLCHAIN "gnu" CACHE STRING "ThreadX toolchain" FORCE)
elseif(${CONFIG_OMNI_FAMILY} STREQUAL "stm32h7")
    set(THREADX_ARCH "cortex_m7" CACHE STRING "ThreadX architecture" FORCE)
    set(THREADX_TOOLCHAIN "gnu" CACHE STRING "ThreadX toolchain" FORCE)
else()
    message(FATAL_ERROR "CONFIG_OMNI_FAMILY is not set")
endif()

add_library(omni-threadx INTERFACE)

add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/ports/${THREADX_ARCH}/${THREADX_TOOLCHAIN})
add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/common)

# Add the ThreadX low level configuration file
set(THREADX_LL_CFG_SOURCE_FILE ${CMAKE_CURRENT_LIST_DIR}/ports/${THREADX_ARCH}/${THREADX_TOOLCHAIN}/tx_initialize_low_level.S)
set(THREADX_LL_CFG_TARGET_FILE ${OMNI_RTOS_CFG_DIR}/tx_initialize_low_level.S)
omni_add_configure_file(${THREADX_LL_CFG_SOURCE_FILE} ${THREADX_LL_CFG_TARGET_FILE})

# Add the ThreadX configuration file
set(THREADX_CFG_SOURCE_FILE ${CMAKE_CURRENT_LIST_DIR}/common/inc/tx_user_sample.h)
set(THREADX_CFG_TARGET_FILE ${OMNI_RTOS_CFG_DIR}/tx_user.h)
omni_add_configure_file(${THREADX_CFG_SOURCE_FILE} ${THREADX_CFG_TARGET_FILE})

target_sources(omni-threadx INTERFACE
    ${THREADX_LL_CFG_TARGET_FILE}
)

omni_lib_src_ifdef(CONFIG_THREADX_EXECUTION_PROFILE 
    omni-threadx
    utility/execution_profile_kit/tx_execution_profile.c
)

omni_lib_inc_ifdef(CONFIG_THREADX_EXECUTION_PROFILE
    omni-threadx
    utility/execution_profile_kit
)