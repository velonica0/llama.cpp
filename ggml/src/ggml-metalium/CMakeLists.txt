if("$ENV{TT_METAL_HOME}" STREQUAL "")
    message(FATAL_ERROR "TT_METAL_HOME is not set")
endif()
if("$ENV{ARCH_NAME}" STREQUAL "")
    message(FATAL_ERROR "ARCH_NAME is not set")
endif()

message(STATUS "Metalium found.")
# N stands for Normalized
set(ARCH_NAME_N "$ENV{ARCH_NAME}")
if("$ENV{ARCH_NAME}" STREQUAL "wormhole_b0")
    set(ARCH_NAME_N "wormhole")
endif()

set(METALIUM_EXTRA_INCLUDES "")
if(ARCH_NAME_N STREQUAL "wormhole")
    set(METALIUM_EXTRA_INCLUDES  $ENV{TT_METAL_HOME}/tt_metal/hw/inc/wormhole/wormhole_b0_defines)
endif()

# TODO: In the future TTNN can optionally not depend on Python
find_package(Python REQUIRED Development)

ggml_add_backend_library(ggml-metalium
    ggml-metalium.cpp
)


target_include_directories(ggml-metalium PRIVATE
    # Metalium
    $ENV{TT_METAL_HOME}
    $ENV{TT_METAL_HOME}/tt_metal
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api/umd
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api/umd/device
    $ENV{TT_METAL_HOME}/tt_metal/third_party/fmt
    $ENV{TT_METAL_HOME}/tt_metal/hw/inc/${ARCH_NAME_N}
    $ENV{TT_METAL_HOME}/tt_metal/hw/inc/
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/src/firmware/riscv/${ARCH_NAME_N}
    $ENV{TT_METAL_HOME}/tt_metal/third_party/tracy/public/tracy
    $ENV{TT_METAL_HOME}/tt_metal/third_party/tracy/public/
    $ENV{TT_METAL_HOME}/tt_metal/hostdevcommon/api/
    $ENV{TT_METAL_HOME}/tt_metal/include

    # TTNN
    $ENV{TT_METAL_HOME}/ttnn/cpp
    $ENV{TT_METAL_HOME}/ttnn/cpp/ttnn/deprecated/
    $ENV{TT_METAL_HOME}/tt_eager
    $ENV{TT_METAL_HOME}/tt_metal/third_party/magic_enum

    # Metalium extra includes
    ${METALIUM_EXTRA_INCLUDES}

    ..
)

target_link_directories(ggml-metalium PRIVATE
    $ENV{TT_METAL_HOME}/build/lib
)

target_link_libraries(ggml-metalium PRIVATE
    # Metalium
    tt_metal
    yaml-cpp

    # TTNN
    Python::Python
    $ENV{TT_METAL_HOME}/build/lib/_ttnn.so

    ggml-base
)


target_compile_definitions(
    ggml-metalium PRIVATE
    FMT_HEADER_ONLY
)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsized-deallocation")
endif()

target_precompile_headers(ggml-metalium PRIVATE metalium-pch.hpp)
# FIXME: Metalium shoud should only require C++20 but needs C++23 on GCC
# see: https://github.com/tenstorrent/tt-metal/issues/10652
target_compile_features(ggml-metalium PRIVATE cxx_std_23)
