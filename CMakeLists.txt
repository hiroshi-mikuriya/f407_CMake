cmake_minimum_required(VERSION 3.6)

set(CMAKE_SYSTEM_NAME Generic)

project(F407_CMake)

# product
set(ELF_FILE ${PROJECT_NAME}.elf)
set(HEX_FILE ${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_NAME}.bin)
set(MAP_FILE ${PROJECT_NAME}.map)

# compiler
set(CMAKE_C_COMPILER   arm-none-eabi-gcc)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_EXE_LINKER   arm-none-eabi-g++)
set(OBJCOPY            arm-none-eabi-objcopy)
set(SIZE               arm-none-eabi-size)

# compiler options
set(CPU       cortex-m4)
set(FPU       -mfpu=fpv4-sp-d16)
set(FLOAT_ABI -mfloat-abi=hard)
set(LDSCRIPT  STM32F407VGTx_FLASH.ld)

set(CMAKE_C_FLAGS "-mcpu=${CPU} -std=gnu11 -Os -ffunction-sections -fdata-sections -Wall -fstack-usage ${FPU} ${FLOAT_ABI} -mthumb")
set(CMAKE_CXX_FLAGS "-mcpu=${CPU} -std=gnu++14 -Os -ffunction-sections -fdata-sections -fno-exceptions -fno-rtti -fno-use-cxa-atexit -Wall -fstack-usage ${FPU} ${FLOAT_ABI} -mthumb")
set(CMAKE_AS_FLAGS "-mcpu=${CPU} -x assembler-with-cpp ${FPU} ${FLOAT_ABI} -mthumb")
set(CMAKE_EXE_LINKER_FLAGS "-mcpu=${CPU} -T${CMAKE_CURRENT_SOURCE_DIR}/${LDSCRIPT} --specs=nosys.specs -Wl,-Map=${MAP_FILE} -Wl,--gc-sections -static --specs=nano.specs ${FPU} ${FLOAT_ABI} -mthumb -Wl,--start-group -lc -lm -lstdc++ -lsupc++ -Wl,--end-group")

enable_language(ASM)

# define
add_definitions(-DUSE_FULL_LL_DRIVER)
add_definitions(-DPREFETCH_ENABLE=1)
add_definitions(-DINSTRUCTION_CACHE_ENABLE=1)
add_definitions(-DDATA_CACHE_ENABLE=1)
add_definitions(-DSTM32F407xx)

# includes
include_directories(
	Core/Inc
	Drivers/STM32F4xx_HAL_Driver/Inc
	Drivers/CMSIS/Device/ST/STM32F4xx/Include
	Drivers/CMSIS/Include
)

# sources
file(GLOB SRCS
	Core/Src/main.c
	Core/Src/stm32f4xx_it.c
	Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_gpio.c
	Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_rcc.c
	Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_utils.c
	Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_ll_exti.c
	Core/Src/system_stm32f4xx.c
	startup_stm32f407xx.s
)

# executables
add_executable(${ELF_FILE} ${SRCS})

add_custom_command(TARGET ${ELF_FILE} POST_BUILD
	COMMAND ${OBJCOPY} -Obinary ${ELF_FILE} ${BIN_FILE}
	COMMAND ${OBJCOPY} -Oihex   ${ELF_FILE} ${HEX_FILE}
	COMMAND ${SIZE} --format=berkeley ${ELF_FILE} ${HEX_FILE}
	COMMENT "Invoking: Cross ARM GNU Print Size"
)
