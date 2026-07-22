cmake_minimum_required(VERSION 3.16)
project(mapflns2 LANGUAGES CXX)

option(MAPFLNS2_BUILD_DRIVER "Build the MAPFLNS2 command-line driver" ON)

# --- Sources (exclude driver) -------------------------------------------------
file(GLOB MAPFLNS2_ALL_SOURCES CONFIGURE_DEPENDS
        src/*.cpp
        src/CBS/*.cpp
        src/PIBT/*.cpp)
set(MAPFLNS2_DRIVER_SRC src/driver.cpp)
list(REMOVE_ITEM MAPFLNS2_ALL_SOURCES ${MAPFLNS2_DRIVER_SRC})

# --- Core library -------------------------------------------------------------
add_library(mapflns2_core STATIC ${MAPFLNS2_ALL_SOURCES})
add_library(mapflns2::core ALIAS mapflns2_core)

# IMPORTANT: export all header roots that headers assume in #include "..."
# (ConstraintTable.h includes "CBSNode.h" -> needs inc/CBS in the search path)
target_include_directories(mapflns2_core
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc/CBS>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc/PIBT>
        $<INSTALL_INTERFACE:include>
        $<INSTALL_INTERFACE:include/CBS>
        $<INSTALL_INTERFACE:include/PIBT>
)

target_compile_features(mapflns2_core PUBLIC cxx_std_17)
set_target_properties(mapflns2_core PROPERTIES POSITION_INDEPENDENT_CODE ON)

# Deps used in core
find_package(Boost CONFIG REQUIRED)     # header-only is fine for core
target_link_libraries(mapflns2_core PUBLIC Boost::boost)

find_package(Eigen3 3.3 REQUIRED NO_MODULE)
target_link_libraries(mapflns2_core PUBLIC Eigen3::Eigen)

# --- Optional CLI driver ------------------------------------------------------
if(MAPFLNS2_BUILD_DRIVER)
    add_executable(mapflns2 ${MAPFLNS2_DRIVER_SRC})
    target_compile_features(mapflns2 PRIVATE cxx_std_17)
    target_link_libraries(mapflns2
            PRIVATE
            mapflns2_core
            Boost::program_options
            Boost::filesystem
            Boost::system
    )
    # If your Boost doesn’t provide the imported targets above, fall back to:
    # find_package(Boost REQUIRED COMPONENTS program_options filesystem system)
    # target_include_directories(mapflns2 PRIVATE ${Boost_INCLUDE_DIRS})
    # target_link_libraries(mapflns2 PRIVATE ${Boost_LIBRARIES})
endif()
