cmake_minimum_required(VERSION 3.20)
project(pcpd CXX)

# Prefix
if (NOT PREFIX)
    set(PREFIX ${CMAKE_CURRENT_BINARY_DIR})
endif (NOT PREFIX)
set(CMAKE_INSTALL_PREFIX ${PREFIX} CACHE STRING "Installation prefix." FORCE)

if (CXX)
    set(CMAKE_CXX_COMPILER ${CXX})
endif (CXX)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

# Release or Debug
if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Debug' as none was specified.")
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build." FORCE)
endif ()


# Detect PNL
find_package(Pnl REQUIRED)
set(LIBS ${LIBS} ${PNL_LIBRARIES})
include_directories(${PNL_INCLUDE_DIRS})
if(${CMAKE_BUILD_TYPE} STREQUAL "Release")
    add_definitions(-DPNL_RANGE_CHECK_OFF)
endif()

# Find nlohmann_json to parse json files
# Add nlohmann_json::nlohmann_json to target_link_libraries
find_package(nlohmann_json)
if (NOT nlohmann_json_FOUND)
    if (${CMAKE_VERSION} VERSION_GREATER "3.24.0")
        CMAKE_POLICY(SET CMP0135 NEW)
    endif()
    include(FetchContent)
    FetchContent_Declare(json URL https://github.com/nlohmann/json/releases/download/v3.11.2/json.tar.xz)
    FetchContent_MakeAvailable(json)
endif()

### Ne rien modifier dans la partie au dessus de cette ligne.

# Test the parser
add_executable(test_json_reader src/json_helper.cpp src/test_json_reader.cpp)
target_link_libraries(test_json_reader
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
)
pnl_add_postbuild(test_json_reader)

add_executable(test_compute_lastdate src/test_compute_lastdate.cpp)


# my_lib 
# Utiliser GLOB pour inclure tous les fichiers source
file(GLOB HELPER_FILE src/json_helper.cpp src/pcpd_helper.cpp)
file(GLOB OPTION_FILES src/Option/*.cpp)
file(GLOB BSM_FILES src/black_scholes_model/*.cpp)
file(GLOB MONTCARLO_FILES src/monte_carlo/*.cpp)
file(GLOB UTILS_FILES src/Utils/*.cpp)

# Créer la bibliothèque avec tous les fichiers .cpp trouvés
add_library(my_lib ${SRC_FILES} ${OPTION_FILES} ${BSM_FILES} ${MONTCARLO_FILES} ${UTILS_FILES} ${HELPER_FILE})


# Pour compiler vos exécutables, compléter les lignes ci-dessous.

# add_executable(price0 _liste_des_fichiers_cpp)
# target_link_libraries(price0
#     ${PNL_LIBRARIES}
#     nlohmann_json::nlohmann_json
# )

# add_executable(hedge _liste_des_fichiers_cpp)
# target_link_libraries(hedge
#     ${PNL_LIBRARIES}
#     nlohmann_json::nlohmann_json
# )



# ============================== executable pour les tests :GTEST ===================
# Ajoute les sous-répertoires pour Google Test si installé depuis GitHub
find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})


# Test Convert : src/Utils/convert.cpp
add_executable(test_convert src/tests/test_convert.cpp)
target_link_libraries(test_convert
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)



# Test Pricer à t = 0: 
add_executable(test_pricer_0 src/tests/test_pricer_0.cpp)
target_link_libraries(test_pricer_0
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)

# Test Pricer à t  
add_executable(test_pricer_t src/tests/test_price_t.cpp)
target_link_libraries(test_pricer_t
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)

# Test Deltas : 
add_executable(test_deltas_0 src/tests/test_deltas_0.cpp)
target_link_libraries(test_deltas_0
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)


# Test Deltas : 
add_executable(test_deltas src/tests/test_deltas.cpp)
target_link_libraries(test_deltas
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)

# Test hedge : 
add_executable(test_hedge src/tests/test_hedge.cpp)
target_link_libraries(test_hedge
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)

# ========================================================== End Testing ==============================

# price0: 
add_executable(price0 src/price.cpp)
target_link_libraries(price0
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json

)

# hedge :  
add_executable(hedge src/hedge.cpp)
target_link_libraries(hedge
    my_lib
    ${PNL_LIBRARIES}
    nlohmann_json::nlohmann_json
    ${GTEST_LIBRARIES}
    gtest_main
    pthread
)



# Print compiler flags
get_directory_property(DirDefs COMPILE_DEFINITIONS)
message(STATUS "COMPILE_DEFINITIONS = ${DirDefs}")
if (${CMAKE_BUILD_TYPE} STREQUAL "Release")
    message(STATUS "C++ Flags: ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_RELEASE} ${CMAKE_LINKER_EXE_FLAGS}" )
else()
    message(STATUS "C++ Flags: ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_DEBUG} ${CMAKE_LINKER_EXE_FLAGS}" )
endif()
