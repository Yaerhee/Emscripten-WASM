cmake_minimum_required(VERSION 3.27)
project(EmscriptenWASM CXX)
set(CMAKE_CXX_STANDARD 17)

set(Emscripten_WASM_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Build Cryptopp

add_library(Cryptopp)

FILE(GLOB CRYPTOPP_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/external/cryptopp/*.cpp)

target_sources(
    Cryptopp
        PRIVATE
            ${CRYPTOPP_SRCS}
)

# Build EmscriptenWASM_Exec

add_executable(EmscriptenWASM_Exec)

FILE(GLOB CRYPTOPP_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/cryptopp/*.cpp)

target_sources(
    EmscriptenWASM_Exec
        PRIVATE
            ${Emscripten_WASM_SOURCE_DIR}/WASM/EmscriptenWASM.cpp
)

target_include_directories(
    EmscriptenWASM_Exec
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/include
            ${CMAKE_CURRENT_SOURCE_DIR}/external
)

target_link_libraries(
    EmscriptenWASM_Exec
        PRIVATE
            Cryptopp
)

target_link_options(
    EmscriptenWASM_Exec
        PRIVATE
            -sWASM=1 -sEXPORTED_RUNTIME_METHODS=ccall,cwrap,UTF8ToString -sSAFE_HEAP=1 -sNO_DISABLE_EXCEPTION_CATCHING -sEXPORTED_FUNCTIONS=['_test']
)

add_custom_command(TARGET EmscriptenWASM_Exec POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_BINARY_DIR}/EmscriptenWASM_Exec.js ${CMAKE_CURRENT_SOURCE_DIR}/result
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_BINARY_DIR}/EmscriptenWASM_Exec.wasm ${CMAKE_CURRENT_SOURCE_DIR}/result
)
