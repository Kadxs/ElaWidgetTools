﻿cmake_minimum_required(VERSION 3.5)

project(ElaWidgetTools VERSION 2.0.0 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

add_definitions(-DELAWIDGETTOOLS_LIBRARY)
option(ELAWIDGETTOOLS_BUILD_STATIC_LIB "Build static library." OFF)

FILE(GLOB ORIGIN *.h *.cpp)
FILE(GLOB INCLUDE include/*.h)
FILE(GLOB PRIVATE private/*.h private/*.cpp)
FILE(GLOB DEVELOPER DeveloperComponents/*.h DeveloperComponents/*.cpp)

source_group(include FILES ${INCLUDE})
source_group(private FILES ${PRIVATE})
source_group(DeveloperComponents FILES ${DEVELOPER})

set(PROJECT_SOURCES
    ${ORIGIN}
    ${INCLUDE}
    ${PRIVATE}
    ${DEVELOPER}
)

if (ELAWIDGETTOOLS_BUILD_STATIC_LIB)
    set(LIB_TYPE "STATIC")
else ()
    set(LIB_TYPE "SHARED")
endif ()

if (${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_library(${PROJECT_NAME} ${LIB_TYPE}
        ${PROJECT_SOURCES}
    )
else ()
    add_library(${PROJECT_NAME} ${LIB_TYPE}
        ${PROJECT_SOURCES}
    )
endif ()

FILE(GLOB EXPORT_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/*.qrc
)
target_include_directories(${PROJECT_NAME} PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/private
    ${CMAKE_CURRENT_SOURCE_DIR}/DeveloperComponents
)

if (${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    #遍历所有资源文件
    file(GLOB_RECURSE RES_PATHS *.png *.jpg *.svg *.ico *.ttf *.webp *.js)
    foreach (filepath ${RES_PATHS})
        string(REPLACE "${CMAKE_CURRENT_SOURCE_DIR}/" "" filename ${filepath})
        list(APPEND resource_files ${filename})
    endforeach (filepath)
    qt_add_resources(${PROJECT_NAME} "ElaWidgetTools"
        RESOURCES PREFIX "/"
        FILES
        ${resource_files}
    )
endif ()

if (MINGW)
    set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
endif ()
if (MSVC)
    set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX "d")
endif ()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${PROJECT_NAME})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${PROJECT_NAME})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${PROJECT_NAME})
if (WIN32)
    target_link_libraries(${PROJECT_NAME} PUBLIC
        Qt${QT_VERSION_MAJOR}::Widgets
        D3D11
        DXGI
    )
else ()
    target_link_libraries(${PROJECT_NAME} PUBLIC
        Qt${QT_VERSION_MAJOR}::Widgets
    )
endif ()


install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}
    ARCHIVE DESTINATION ${PROJECT_NAME}/lib
    LIBRARY DESTINATION ${PROJECT_NAME}/lib
    RUNTIME DESTINATION ${PROJECT_NAME}/bin
)
install(TARGETS ${PROJECT_NAME}
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/ElaWidgetToolsExample
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/ElaWidgetToolsExample
)
if (MSVC AND NOT ELAWIDGETTOOLS_BUILD_STATIC_LIB STREQUAL "STATIC")
    install(
        FILES $<TARGET_PDB_FILE:${PROJECT_NAME}>
        DESTINATION ${CMAKE_INSTALL_PREFIX}/ElaWidgetToolsExample OPTIONAL)
endif ()

install(FILES ${EXPORT_HEADERS} DESTINATION ${PROJECT_NAME}/include)

set(INCLUDE_DIRS include)
set(LIBRARIES ${PROJECT_NAME})
set(LIB_DIR lib)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    VERSION 2.0.0
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in
    ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    INSTALL_DESTINATION lib/cmake
    PATH_VARS INCLUDE_DIRS LIBRARIES LIB_DIR
    INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}
)

install(
    FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake ${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake DESTINATION ${PROJECT_NAME}/lib/cmake
)
