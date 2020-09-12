cmake_minimum_required(VERSION 3.10)
project(deus LANGUAGES C ASM VERSION 0.1)

include(vendor/CMakeLists.txt)

set(CMAKE_C_FLAGS "-mcpu=cortex-m4 -std=gnu11 -g3 -DUSE_HAL_DRIVER -DSTM32L433xx -c -Isrc/ -Ivendor/STM32L4xx_HAL_Driver -Ivendor/STM32L4xx_HAL_Driver/Legacy -Ivendor/CMSIS -O0 -ffunction-sections -fdata-sections -Wall -fstack-usage -MMD -MP --specs=nano.specs -mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb")
set(CMAKE_ASM_FLAGS "-mcpu=cortex-m4 -g3 -c -x assembler-with-cpp -MMD -MP --specs=nano.specs -mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb")

set(SOURCE
  src/main
  src/stm32l4xx_hal_msp
  src/stm32l4xx_it
  src/syscalls
  src/sysmem
  src/system_stm32l4xx
  src/boot/startup_stm32l433cctx.s
  ${VENDOR_MODULES}
)

add_executable(${PROJECT_NAME} ${SOURCE})
target_include_directories(${PROJECT_NAME} PRIVATE
  ${CMAKE_SOURCE_DIR}/src
  ${CMAKE_SOURCE_DIR}/vendor/STM32L4xx_HAL_Driver
  ${CMAKE_SOURCE_DIR}/vendor/STM32L4xx_HAL_Driver/Legacy
  ${CMAKE_SOURCE_DIR}/vendor/CMSIS
)

target_link_libraries(${PROJECT_NAME} "-mcpu=cortex-m4 -T${LINKER_SCRIPT} --specs=nosys.specs -Wl,--gc-sections -static --specs=nano.specs -mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb -Wl,--start-group -lc -lm -Wl,--end-group")

set_property(TARGET ${PROJECT_NAME} PROPERTY LINK_DEPENDS ${LINKER_SCRIPT})

add_custom_command(TARGET ${PROJECT_NAME}
  POST_BUILD
  COMMAND ${TOOLCHAIN}-objcopy
  ARGS -O binary ${PROJECT_NAME} ${PROJECT_NAME}.bin
  BYPRODUCTS ${PROJECT_NAME}.bin
  COMMENT "Creating flashable binary...")
