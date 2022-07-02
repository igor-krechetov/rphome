cmake_minimum_required(VERSION 3.18)
project(RPHomeApp LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)

set(BINARY_NAME RPHomeApp)

# NOTE: Requires Qt 6 compiled for RP4 with correct toolchain. Make sure to configure environment and use qt-cmake

# ----------------------------------------------------------------------
# HSMCPP
# Configure and install HSMCPP library
SET(HSMBUILD_VERBOSE OFF CACHE BOOL "Disable traces")
SET(HSMBUILD_EXAMPLES OFF CACHE BOOL "Disable HSM examples")
SET(HSMBUILD_TESTS OFF CACHE BOOL "Disable HSM tests")
SET(HSMBUILD_DISPATCHER_GLIB OFF CACHE BOOL "Disable Glib dispatcher")
SET(HSMBUILD_DISPATCHER_GLIBMM OFF CACHE BOOL "Disable Glibmm dispatcher")
SET(HSMBUILD_DISPATCHER_QT ON CACHE BOOL "Enable Qt dispatcher")
SET(HSMBUILD_DISPATCHER_STD OFF CACHE BOOL "Disable STD dispatcher")

add_subdirectory(./modules/hsmcpp)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${HSMCPP_CXX_FLAGS}")

# generate state machine code
set(GEN_DIR ${CMAKE_BINARY_DIR}/gen)
file(MAKE_DIRECTORY ${GEN_DIR})
generateHsm(GEN_HSM ./rphome.scxml "RPHome" ${GEN_DIR} "GEN_HSM_SRC")

# # link with HSMCPP library (based on used dispatcher)
# target_link_libraries(${BINARY_NAME} PRIVATE ${HSMCPP_STD_LIB} )

# ----------------------------------------------------------------------
# Qt application
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

find_package(Qt6 COMPONENTS Gui Qml Quick)

qt_add_executable(${BINARY_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
                            ${GEN_HSM_SRC})

qt_add_resources(${BINARY_NAME} "configuration"
    PREFIX "/"
    FILES
        qtquickcontrols2.conf
)

add_dependencies(${BINARY_NAME} GEN_HSM)
# -DLOGGING_MODE_STRICT_VERBOSE
# target_compile_definitions(${BINARY_NAME} PUBLIC ${HSM_DEFINITIONS_STD})

target_link_libraries(${BINARY_NAME} PRIVATE
    Qt${QT_VERSION_MAJOR}::Core
    Qt${QT_VERSION_MAJOR}::Gui
    Qt${QT_VERSION_MAJOR}::Quick
    Qt${QT_VERSION_MAJOR}::Qml
    ${HSMCPP_QT_LIB}
)

target_include_directories(${BINARY_NAME}
    PRIVATE
        ${HSMCPP_QT_INCLUDE}
        ${CMAKE_BINARY_DIR}
        ./modules/hwiocpp/include
        ./src
)

include(${CMAKE_CURRENT_SOURCE_DIR}/qmlmodules)
