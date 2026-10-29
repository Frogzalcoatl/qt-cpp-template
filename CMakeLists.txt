cmake_minimum_required(VERSION 3.16)

project(MyQtProject VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Required for .qrc files (QT resource files for icons, images, etc)
set(CMAKE_AUTORCC ON)

# Suggestion from AI to make this compatible with Visual Studio
# Auto-detect default Qt 6 MSVC installations on Windows
if(NOT CMAKE_PREFIX_PATH AND WIN32 AND MSVC)
    file(GLOB _QT_CANDIDATES "C:/Qt/6.*/msvc2022_64")
    if(_QT_CANDIDATES)
        list(REVERSE _QT_CANDIDATES) # Pick newest if multiple exist
        list(GET _QT_CANDIDATES 0 _QT_PATH)
        list(APPEND CMAKE_PREFIX_PATH "${_QT_PATH}")
        message(STATUS "Auto-detected Qt at: ${_QT_PATH}")
    endif()
endif()

find_package(Qt6 REQUIRED COMPONENTS Widgets Network)

qt_standard_project_setup()

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
)

qt_add_executable(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE include)

target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt6::Widgets
    Qt6::Network
)

# Another suggestion from AI for Visual Studio
# Automatically bundes Qt DLLs on Windows
if(WIN32)
    # Locate windeployqt.exe inside the detected Qt path
    find_program(WINDEPLOYQT_EXE windeployqt HINTS "${_QT_PATH}/bin")
    
    # Automatically copy all necessary Qt DLLs and plugins next to the .exe
    if(WINDEPLOYQT_EXE)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND "${WINDEPLOYQT_EXE}" "$<TARGET_FILE:${PROJECT_NAME}>"
            COMMENT "Running windeployqt to bundle DLLs..."
        )
    endif()
endif()