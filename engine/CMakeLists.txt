cmake_minimum_required(VERSION 2.8)

set (CMAKE_CXX_STANDARD 17)

option(USE_PROFILING             "Build with profiling"   OFF)
option(USE_RL                    "Build with reinforcement learning support"  OFF)
option(BACKEND_TENSORRT          "Build with TensorRT support"  ON)
option(BACKEND_MXNET             "Build with MXNet backend (Blas/IntelMKL/CUDA/TensorRT) support"  OFF)
option(BACKEND_TORCH             "Build with Torch backend (CPU/GPU) support" OFF)
option(USE_960                   "Build with 960 variant support"  OFF)
option(BUILD_TESTS               "Build and run tests"  OFF)
# enable a single mode for different model input / outputs
option(MODE_CRAZYHOUSE           "Build with crazyhouse only support"  ON)
option(MODE_CHESS                "Build with chess + chess960 only support"  OFF)
option(MODE_LICHESS              "Build with lichess variants support"  OFF)
option(MODE_OPEN_SPIEL           "Build with open_spiel environment support"  OFF)
option(SEARCH_UCT                "Build with UCT instead of PUCT search"  OFF)

add_definitions(-DIS_64BIT)
add_definitions(-DCRAZYHOUSE)

if (MODE_CRAZYHOUSE)
    project(CrazyAra CXX)
    add_definitions(-DMODE_CRAZYHOUSE)
endif()

if (MODE_CHESS)
    project(ClassicAra CXX)
    add_definitions(-DMODE_CHESS)
    add_definitions(-DMCTS_TB_SUPPORT)
endif()

if (MODE_LICHESS)
    project(MultiAra CXX)
    add_definitions(-DMODE_LICHESS)
    add_definitions(-DCRAZYHOUSE)
    add_definitions(-DKOTH)
    add_definitions(-DTHREECHECK)
    add_definitions(-DANTI)
    add_definitions(-DATOMIC)
    add_definitions(-DHORDE)
    add_definitions(-DRACE)
    add_definitions(-DMCTS_TB_SUPPORT)
endif()

if (MODE_OPEN_SPIEL)
    project(OpenSpielAra CXX)
    add_definitions(-DMODE_OPEN_SPIEL)
    add_definitions(-DACTION_64_BIT)
endif()

if (BUILD_TESTS)
    add_definitions(-DBUILD_TESTS)
endif()

# -pg performance profiling flags
if (USE_PROFILING)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg")
    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pg")
    SET(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -pg")
endif()

if(DEFINED ENV{BLAZE_PATH})
    MESSAGE(STATUS "BLAZE_PATH set to: $ENV{BLAZE_PATH}")
    include_directories("$ENV{BLAZE_PATH}/include")
    include_directories("$ENV{BLAZE_PATH}")
else()
    MESSAGE(STATUS "BLAZE_PATH not set")
endif()

find_package( blaze )
if( blaze_FOUND )
     add_library( blaze_target INTERFACE )
     target_link_libraries( blaze_target INTERFACE blaze::blaze )
endif()

if (SEARCH_UCT)
    add_definitions(-DSEARCH_UCT)
endif()

file(GLOB source_files
    "3rdparty/catch-2.91/*.hpp"
    "*.h"
    "*.cpp"
    "*.md"
    "src/*.h"
    "src/*.cpp"
    "src/util/*.h"
    "src/util/*.cpp"
    "src/agents/*.cpp"
    "src/agents/*.h"
    "src/agents/config/*.cpp"
    "src/agents/config/*.h"
    "src/agents/util/*.cpp"
    "src/agents/util/*.h"
    "src/nn/*.cpp"
    "src/nn/*.h"
    "src/nn/util/*.cpp"
    "src/nn/util/*.h"
    "src/manager/*.cpp"
    "src/manager/*.h"
    "src/rl/*.cpp"
    "src/rl/*.h"
    "tests/*.h"
    "tests/*.cpp"
)

set(CPP_PACKAGE_INCLUDE_DIR $ENV{MXNET_PATH}/cpp-package/include)
file(GLOB_RECURSE CPP_PACKAGE_HEADERS
    "${CPP_PACKAGE_INCLUDE_DIR}/*.h"
    "${CPP_PACKAGE_INCLUDE_DIR}/*.hpp"
    )

find_package (Threads)
include_directories("3rdparty/Stockfish/src")
include_directories("src")

file(GLOB sf_related_files
    "3rdparty/Stockfish/src/syzygy/tbprobe.h"
    "3rdparty/Stockfish/src/syzygy/tbprobe.cpp"
    "3rdparty/Stockfish/src/types.h"
    "3rdparty/Stockfish/src/types.cpp"
    "3rdparty/Stockfish/src/position.h"
    "3rdparty/Stockfish/src/position.cpp"
    "3rdparty/Stockfish/src/uci.h"
    "3rdparty/Stockfish/src/uci.cpp"
    "3rdparty/Stockfish/src/bitboard.h"
    "3rdparty/Stockfish/src/bitboard.cpp"
    "3rdparty/Stockfish/src/*.cpp"
    )
file(GLOB uci_files
    "src/uci/*.h"
    "src/uci/*.cpp"
    )
file(GLOB chess_related_files
    "src/environments/chess_related/*.h"
    "src/environments/chess_related/*.cpp"
    )

set(source_files
    ${source_files}
    ${sf_related_files}
    ${uci_files}
    )

if (NOT MODE_OPEN_SPIEL)
    set(source_files
        ${source_files}
        ${chess_related_files}
        )
endif()

if (MODE_OPEN_SPIEL)

    set (OPEN_SPIEL_CORE_FILES
      action_view.h
      action_view.cc
      canonical_game_strings.cc
      canonical_game_strings.h
      fog/fog_constants.h
      fog/observation_history.cc
      fog/observation_history.h
      game_parameters.cc
      game_parameters.h
      matrix_game.cc
      matrix_game.h
      normal_form_game.h
      observer.cc
      observer.h
      policy.cc
      policy.h
      simultaneous_move_game.cc
      simultaneous_move_game.h
      spiel.cc
      spiel.h
      spiel_bots.cc
      spiel_bots.h
      spiel_globals.h
      spiel_utils.cc
      spiel_utils.h
      tensor_game.cc
      tensor_game.h
    )

    file(GLOB open_spiel_abseil_files
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/numeric/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/numeric/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/random/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/random/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/base/internal/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/base/internal/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/base/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/base/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/meta/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/meta/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/random/internal/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/random/internal/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/strings/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/strings/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/strings/internal/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/strings/internal/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/strings/internal/str_format/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/strings/internal/str_format/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/synchronization/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/synchronization/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/synchronization/internal/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/synchronization/internal/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/types/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/types/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/time/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/time/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/time/internal/cctz/src/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/time/internal/cctz/src/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/debugging/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/debugging/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/debugging/internal/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/debugging/internal/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/hash/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/hash/*.cc"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/hash/internal/*.h"
        "3rdparty/open_spiel/open_spiel/abseil-cpp/absl/hash/internal/*.cc"
        )

    file(GLOB open_spiel_related_files
        "src/environments/open_spiel/*.h"
        "src/environments/open_spiel/*.cpp"
        "3rdparty/open_spiel/open_spiel/games/chess.h"
        "3rdparty/open_spiel/open_spiel/games/chess.cc"
        "3rdparty/open_spiel/open_spiel/games/chess/*.h"
        "3rdparty/open_spiel/open_spiel/games/chess/*.cc"
        "3rdparty/open_spiel/open_spiel/games/hex.h"
        "3rdparty/open_spiel/open_spiel/games/hex.cc"
        )

    list(TRANSFORM OPEN_SPIEL_CORE_FILES PREPEND "3rdparty/open_spiel/open_spiel/")
    list(FILTER open_spiel_abseil_files EXCLUDE REGEX "^.*test.*$")
    list(FILTER open_spiel_abseil_files EXCLUDE REGEX "^.*benchmark.*$")
    list(FILTER open_spiel_abseil_files EXCLUDE REGEX "^.*gentables.*$")
    list(FILTER open_spiel_abseil_files EXCLUDE REGEX "^.*mutex_nonprod.*$")
    list(FILTER open_spiel_abseil_files EXCLUDE REGEX "^.*chi_square.*$")
    list(FILTER open_spiel_abseil_files EXCLUDE REGEX "^.*print_hash_of.*$")

    set (BUILD_TESTING OFF)
    include_directories("3rdparty/open_spiel")
    include_directories("3rdparty/open_spiel/open_spiel/abseil-cpp")
    message(STATUS "MODE_OPEN_SPIEL")
    set(source_files
        ${source_files}
        ${open_spiel_related_files}
        ${open_spiel_abseil_files}
        ${OPEN_SPIEL_CORE_FILES}
        )
endif()

include_directories("3rdparty/catch-2.9.1")

# incude dirent library seperately because it's missing in the stdlib
if(MSVC OR MSYS OR MINGW)
    include_directories("3rdparty/dirent")
endif()

include_directories("src/domain/crazyhouse")
include_directories("src/agents")
include_directories("src/agents/config")
include_directories("src/nn")


if (BACKEND_MXNET)
    IF(DEFINED ENV{MXNET_PATH})
    MESSAGE(STATUS "MXNET_PATH set to: $ENV{MXNET_PATH}")
    ELSE()
    MESSAGE(STATUS "MXNET_PATH not set")
    ENDIF()

    include_directories("$ENV{MXNET_PATH}/cpp-package/include")
    include_directories("$ENV{MXNET_PATH}/include/")
    include_directories("$ENV{MXNET_PATH}/3rdparty/tvm/nnvm/include")
    include_directories("$ENV{MXNET_PATH}/3rdparty/dmlc-core/include")

    link_directories("$ENV{MXNET_PATH}/lib")
    link_directories("$ENV{MXNET_PATH}/Release/lib")
    link_directories("$ENV{MXNET_PATH}/build/lib")
    link_directories("$ENV{MXNET_PATH}/build/Release")
    link_directories("$ENV{MXNET_PATH}/build")

    set(CPP_EXAMPLE_LIBS
        ${BEGIN_WHOLE_ARCHIVE} mxnet_static ${END_WHOLE_ARCHIVE}
        ${BEGIN_WHOLE_ARCHIVE} dmlc ${END_WHOLE_ARCHIVE}
        ${mxnet_LINKER_LIBS}
        )

    add_definitions(-DMXNET)
endif()

if (BACKEND_TORCH)
    message(STATUS "Enabled Torch Backend")
    find_package(Torch REQUIRED)
    add_definitions(-DTORCH)
endif()

if (USE_RL)
    message(STATUS "Enabled Reinforcement Learning functionality")
    if(DEFINED ENV{Z5_PATH})
        MESSAGE(STATUS "Z5_PATH set to: $ENV{Z5_PATH}")
    else()
        MESSAGE(STATUS "Z5_PATH not set")
    endif()
    include_directories($ENV{Z5_PATH}/include)
    include_directories("3rdparty/json-3.7.0")
    include_directories($ENV{XTL_PATH}/include)
    include_directories($ENV{XTENSOR_PATH}/include)
    add_definitions(-DUSE_RL)
    add_definitions(-DDISABLE_UCI_INFO)
endif()

if (BACKEND_TENSORRT)
    # build CrazyAra with TensorRT support, requires a working TensorRT-MXNet library package
    message(STATUS "Enabled TensorRT support")
    message(STATUS "TensorRT path: $ENV{TENSORRT_PATH}")
    # add additional source file
    set(source_files
        ${source_files}
        $ENV{TENSORRT_PATH}/samples/common/logger.cpp
        )
    find_package(CUDA REQUIRED)
    message(STATUS "CUDA path: $ENV{CUDA_PATH}")
    include_directories("$ENV{CUDA_PATH}/include")
    link_directories("$ENV{CUDA_PATH}/lib64")
    link_directories("$ENV{CUDA_PATH}/lib/x64")
    link_directories("$ENV{TENSORRT_PATH}/lib")
if(WIN32)
  find_library(TENSORRT_LIBRARY_MYELIN myelin64_1
    HINTS  ${TENSORRT_PATH}
    PATH_SUFFIXES lib lib64 lib/x64)
endif()
    include_directories("$ENV{TENSORRT_PATH}/include")
    include_directories("$ENV{TENSORRT_PATH}/samples/common/")
    add_definitions(-DTENSORRT)
endif()

if (USE_960)
    add_definitions(-DSUPPORT960)
endif()

add_executable(${PROJECT_NAME} ${source_files})

if (BACKEND_TENSORRT)
    target_link_libraries(${PROJECT_NAME} nvonnxparser nvinfer cudart myelin ${CUDART_LIB} ${CUBLAS_LIB} ${CUDNN_LIB})
endif()

if (USE_RL)
    # include filesystem (needed for z5)
    target_link_libraries(${PROJECT_NAME} stdc++fs)
endif()

find_package(Threads REQUIRED)

if(THREADS_HAVE_PTHREAD_ARG)
    target_compile_options(${PROJECT_NAME} PUBLIC "-pthread")
endif()
if(CMAKE_THREAD_LIBS_INIT)
    target_link_libraries(${PROJECT_NAME} "${CMAKE_THREAD_LIBS_INIT}")
endif()

# add target directory as library run path for unix systems
if(UNIX)
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "-Wl,-rpath,./")
endif()

if (BACKEND_MXNET)
    if(UNIX)
        target_link_libraries(${PROJECT_NAME} mxnet)
    else()
        target_link_libraries(${PROJECT_NAME} libmxnet)
    endif()
endif()

if (BACKEND_TORCH)
    set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)
    target_link_libraries(${PROJECT_NAME} "${TORCH_LIBRARIES}")
endif()
