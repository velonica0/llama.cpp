if("$ENV{TT_METAL_HOME}" STREQUAL "")
    message(FATAL_ERROR "TT_METAL_HOME is not set")
endif()

message(STATUS "Metalium found.")

ggml_add_backend_library(ggml-metalium
    ggml-metalium.cpp
)

target_include_directories(ggml-metalium PRIVATE
    # Metalium
    $ENV{TT_METAL_HOME}
    $ENV{TT_METAL_HOME}/tt_metal
    $ENV{TT_METAL_HOME}/tt_metal/api
    $ENV{TT_METAL_HOME}/tt_metal/tt_stl
    $ENV{TT_METAL_HOME}/tt_stl
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api/umd
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api/umd/device
    $ENV{TT_METAL_HOME}/tt_metal/third_party/fmt
    $ENV{TT_METAL_HOME}/tt_metal/hw/inc/
    $ENV{TT_METAL_HOME}/tt_metal/third_party/tracy/public/tracy
    $ENV{TT_METAL_HOME}/tt_metal/third_party/tracy/public/
    $ENV{TT_METAL_HOME}/tt_metal/hostdevcommon/api/
    $ENV{TT_METAL_HOME}/tt_metal/include

    # TTNN
    $ENV{TT_METAL_HOME}/ttnn
    $ENV{TT_METAL_HOME}/ttnn/cpp
    $ENV{TT_METAL_HOME}/tt_metal/third_party/magic_enum

    # Parent GGML stuff
    ..
)

target_link_directories(ggml-metalium PRIVATE
    $ENV{TT_METAL_HOME}/build/lib
    $ENV{TT_METAL_HOME}/build/ttnn
)

target_link_libraries(ggml-metalium PRIVATE
    # Metalium
    tt_metal
    yaml-cpp

    # TTNN
    ttnncpp

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
target_compile_features(ggml-metalium PRIVATE cxx_std_20)
