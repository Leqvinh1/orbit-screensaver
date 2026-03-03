cmake_minimum_required(VERSION 3.16)
project(orbit_screensaver)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS_RELEASE "-O2 -s")
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)

find_package(SDL2 REQUIRED)
find_package(SDL2_image QUIET)

if(EXISTS "${CMAKE_SOURCE_DIR}/extern/box2d/CMakeLists.txt")
    set(BOX2D_BUILD_TESTBED OFF CACHE BOOL "" FORCE)
    set(BOX2D_BUILD_UNIT_TESTS OFF CACHE BOOL "" FORCE)
    add_subdirectory(extern/box2d)
else()
    find_package(box2d REQUIRED)
endif()

# main screensaver
add_executable(orbit_screensaver WIN32 main.cpp)
if(EXISTS "${CMAKE_SOURCE_DIR}/icon.rc")
    target_sources(orbit_screensaver PRIVATE icon.rc)
endif()
target_include_directories(orbit_screensaver PRIVATE ${SDL2_INCLUDE_DIRS})
target_link_libraries(orbit_screensaver PRIVATE
    ${SDL2_LIBRARIES}
    SDL2_image::SDL2_image
    box2d
    opengl32
    shell32
    comdlg32
    comctl32
    gdi32
    winmm
    advapi32
)

# stub scr
add_executable(orbit_stub WIN32 stub_scr.cpp)
target_link_libraries(orbit_stub PRIVATE shell32 advapi32)
set_target_properties(orbit_stub PROPERTIES OUTPUT_NAME "orbit_stub")

# copy assets
set(ASSETS
    orb1.png orb2.png orb3.png orb4.png orb5.png
    orb6.png orb7.png orb8.png orb9.png orb10.png
    cube.png
)
foreach(ASSET ${ASSETS})
    if(EXISTS "${CMAKE_SOURCE_DIR}/${ASSET}")
        add_custom_command(TARGET orbit_screensaver POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${CMAKE_SOURCE_DIR}/${ASSET}"
                "$<TARGET_FILE_DIR:orbit_screensaver>/${ASSET}"
        )
    endif()
endforeach()
